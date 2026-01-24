# Chapter 5: SMF, UPF and User Plane Procedures

![sbi](../../image/part2/sbi.png)

In the 5G core network, **SMF (Session Management Function)** and **UPF (User Plane Function)** form a complementary pair—often described as “the planner and the executor.” The SMF is responsible for designing and managing the “routes and rules” of each PDU session for the UE, while the UPF is the node that actually receives, forwards, and processes packets on the user plane. After the UE completes registration, the SMF collaborates with other NFs such as the PCF to establish or modify PDU sessions based on subscription data and policies, and then delivers the corresponding user-plane rules to one or more UPFs via PFCP over the N4 interface. This allows the network to correctly forward uplink and downlink traffic for the UE. This chapter begins by introducing the roles of the SMF and UPF and how they interact, and then progressively breaks down concrete user-plane procedures and implementation details in the following sections.

## 5.1 N3/N4 Interfaces and Related Protocols

### 5.1.1 N3

The N3 interface is the **user-plane interface between the RAN (gNB) and the UPF**, and it carries the UE’s actual data packets (such as web browsing, video streaming, or enterprise VPN traffic). From a topological perspective, it connects the “exit of the radio access network” to the “entry of the core network user plane”:

UE IP packets are first encapsulated into GTP-U packets at the gNB, transmitted over the N3 interface to the UPF, and then decapsulated and forwarded by the UPF either to another UPF or to an external data network according to PFCP rules.

It is worth noting that N3 is only the name of a “logical interface.” In practice, it runs GTP-U over UDP/IP in the protocol stack. The same technology was used in 4G on the S1-U interface between the eNodeB and the S-GW. Therefore, if you are already familiar with 4G, you can think of N3 as the “5G version of S1-U,” with the peer node changing from the S-GW to the UPF.

### 5.1.2 GTP

GTP (GPRS Tunneling Protocol) is a family of protocols “designed for packet encapsulation and control in mobile networks.” On the 5G user plane, the most common variant is **GTP-U (User Plane)**. Its core idea is straightforward:

An additional GTP-U header is added on top of the original IP packet, and the TEID (Tunnel Endpoint Identifier) is used to identify “which tunnel or PDU session the packet belongs to.”

> [!Note]
> A so-called GTP-U tunnel is essentially an agreed-upon set of packet header identifiers between the RAN and the UPF.

On the N3 interface, the gNB will:

- Select the appropriate TEID and destination UPF IP based on the UE’s RAN-side context.
- Encapsulate the UE’s IP packet into GTP-U and send it to the UPF.

After receiving the packet, the UPF will:

- Use the outer TEID to look up the corresponding PDR (thereby determining “which PDU session the packet belongs to”).
- Follow the PDR/FAR configuration to decide whether to decapsulate the packet, where to forward it, or whether to re-encapsulate it into another GTP-U tunnel.

From a troubleshooting perspective, understanding the GTP-U header and the concept of TEIDs is extremely helpful when analyzing Wireshark traces or determining why certain traffic is routed to a specific UPF interface.

### 5.1.3 N4

The N4 interface is the **control interface between the SMF and the UPF**, primarily used to deliver, update, and remove user-plane forwarding rules. From the SMF’s perspective, N4 acts like a “remote control channel”: through it, the SMF tells the UPF which packets belong to which PDU session, where they should be forwarded, and which QoS and charging policies should be applied.

### 5.1.4 PFCP

The protocol used on the N4 interface is PFCP (Packet Forwarding Control Protocol). PFCP is a control-plane signaling protocol responsible for the following between the SMF and the UPF:

- Establishing, modifying, and deleting **sessions**, which correspond to the user-plane state of a PDU session on the UPF.
- Delivering and updating **PDR/FAR/QER and other forwarding and QoS rules** (the detailed fields and behaviors will be explained in Section 5.3.3).
- Reporting usage statistics and events related to charging.

> [!Tip]
> At a high level, you can think of PFCP as the language “the SMF uses to tell the UPF how to handle packets.” The specific fields, encodings, and implementation details of each rule will be explained later in this chapter together with code examples.

From a procedural perspective, a PDU session typically goes through the following stages on N4/PFCP:

1. **Establishment phase**: After selecting the appropriate UPF, the SMF establishes a PFCP session and includes an initial set of PDR/FAR/QER rules, informing the UPF how to classify and forward certain GTP-U packets or internal traffic.
2. **Modification phase**: When the UE’s service requirements or network conditions change (for example, QoS adjustments, path changes, or adding/removing a UPF), the SMF sends a PFCP Session Modification to update the existing rules.
3. **Release phase**: When the UE deregisters or the PDU session is terminated, the SMF sends a PFCP Session Deletion to instruct the UPF to clear the corresponding session and rules, release resources, and stop accounting.

In practice, even a seemingly simple “Internet connection” may correspond to multiple PFCP rules. For example, ordinary data traffic, VoNR voice traffic, and specific enterprise private-network traffic may each map to different PDR/QER configurations. This makes it possible to precisely understand why each packet is handled in a particular way when you later inspect code or logs.

## 5.2 SMF (Session Management Function)

### 5.2.1 Main Functions of the SMF

The SMF is the core control-plane NF in the 5GC responsible for **PDU session management**. Based on UE requests (NAS-SM), subscription data (UDM), policies (PCF), and network topology (available UPFs and paths), it determines:

- Whether a PDU session can be established or modified.
- Which UPF or UPFs to select and which path to use.
- Which **QoS**, **charging**, and **slicing** policies should be applied to the session (these are obtained via SBI service requests to the PCF).

In free5GC, the SMF operates in two worlds simultaneously:

- On one side, it communicates with AMF, PCF, UDM, and other NFs via SBI, receiving PDU Session requests from the UE (NAS-SM forwarded by the AMF) and obtaining subscription and policy information.
- On the other side, it controls the UPF via N4/PFCP, translating the chosen routes and rules into concrete user-plane configurations such as PDRs, FARs, and QERs.

> [!Tip]
> Using a real-world analogy, you can think of the SMF as a “highway traffic planning center”:
>
> - The AMF comes with a request like “this car (UE) wants to go to a certain destination (DNN/S-NSSAI).”
> - The SMF decides which highways to take and which ramps to enter or exit, based on the car’s identity (subscription), pricing plan (policies), and current traffic conditions (topology and load).
> - It then tells the toll booths and ramp controllers along the way (the UPFs) via PFCP how to recognize the car and where to let it pass.

### 5.2.2 PDU Sessions

In the 5G core network, a **PDU session** can be thought of as a **logical dedicated connection** between the UE and a specific data network (DN, such as the Internet or an enterprise private network). As long as the session exists, the UE can send packets into the 5GC, and the SMF / UPF will forward them to the correct destination according to the rules. When the session is released, the associated IP address, QoS configuration, and user-plane paths are reclaimed.

A PDU session typically consists of several key elements:

- **PDN/DNN**: The data network the session connects to (for example, `internet`, an enterprise APN, or a private-network DNN).
- **S-NSSAI / Slice**: The network slice to which the session belongs (representing different customers or service scenarios).
- **IP address or other PDU types**: Such as IPv4, IPv6, IPv4v6, Ethernet, or Unstructured (currently, the official free5GC implementation supports IPv4 only).
- **QoS profile / QoS flows**: A session can be further divided into one or more QoS flows (each associated with a different QFI) to achieve finer-grained quality control.

Compared with the 4G EPC era, a PDU session plays a role similar to that of an EPS bearer. However, 5G introduces several design improvements:

- The separation of “a single session” from “multiple QoS flows,” allowing one PDU session to carry traffic with different QoS requirements (for example, data and voice sharing a session but using different QFIs).
- A tighter binding between “PDU sessions” and “slices / DNNs,” making it easier to establish different types of connections based on service needs (for example, a single UE simultaneously maintaining one PDU session for the public Internet and another for an enterprise private network).

## 5.3 UPF (User Plane Function)

### 5.3.1 Main Functions of the UPF

The UPF is the **data-plane worker** of the 5GC. Based on PFCP rules delivered by the SMF, it performs the following operations on passing packets:

- **Mmatching**: Determining which PDU session and which QFI / QoS flow a packet belongs to.
- **Forwarding**: Sending packets to the RAN, to another UPF, or to an external DN (such as the Internet or a private network).
- **Processing**: Such as usage measurement and accounting, packet mirroring, or limited header modification.

From an implementation perspective, the UPF is closer to a high-performance packet forwarding engine (for example, concepts like DPDK, eBPF, or P4). In this book, however, we focus on “how the UPF interprets PFCP rules from the SMF and applies them to packet paths,” rather than diving into the details of specific hardware or software acceleration technologies.

In the free5GC implementation, a UPF instance typically includes:

- An N3 interface (GTP-U) toward the RAN, responsible for sending and receiving user-plane packets from the gNB.
- Interfaces toward external data networks (DNs, such as the Internet or enterprise networks) for packet egress and ingress.
- An N9 interface toward other UPFs, used in hierarchical or multi-node UPF architectures (such as UL-CL or branching UPF designs).

The basic packet-handling flow in the UPF upon receiving a packet can be summarized as follows:

1. Look up the corresponding PDR based on TEID, 5-tuple, or other fields.
2. Use the FAR specified by the PDR to determine the outgoing interface or next hop, and whether encapsulation or decapsulation is required.
3. Apply QoS settings such as QERs / QFIs, and perform scheduling, shaping, or priority handling as required.
4. Update usage statistics and event information, which can later be reported to the SMF/CHF via PFCP for charging or monitoring purposes.

### 5.3.2 GTP5G

At the implementation level, the free5GC UPF does not “implement the entire GTP-U protocol and packet forwarding logic from scratch.” Instead, it relies heavily on a Linux kernel module developed by free5GC called **gtp5g** to handle low-level packet encapsulation and forwarding. You can think of gtp5g as a “GTP-U processing engine specifically designed for 5G,” integrated into the Linux kernel and exposed for use by the UPF.

The gtp5g module is mainly responsible for:

- Creating and managing GTP-U “tunnels” in the Linux kernel, along with their TEID / IP mappings.
- Performing GTP-U encapsulation and decapsulation on packets entering or leaving specific network interfaces, based on rules configured from user space (the UPF).
- Working together with Linux routing / qdisc / nftables, and related mechanisms to complete packet forwarding and partial QoS / filtering behaviors.

The free5GC UPF communicates with gtp5g via mechanisms such as netlink / ioctl:

- When new PFCP rules are received from the SMF, the UPF writes the corresponding TEIDs, remote IPs, and matching conditions into gtp5g.
- When traffic flows, the actual encapsulation/decapsulation and routing lookups are performed inside the Linux kernel by gtp5g.

This design brings two major benefits:

- The user-plane data path can fully leverage the existing packet-processing capabilities and optimizations of the Linux kernel (such as multi-core support and hardware offload).
- The UPF application can focus on PFCP rule management and 5GC logic, rather than reinventing a high-performance packet forwarding engine.

### 5.3.3 User-Plane Rules

| Rule | Full Name | Description |
| - | - | - |
| PDR | Packet Detection Rule | Defines **which packets belong to which PDU session or traffic flow**. Common matching fields include TEID, UE IP address, 5-tuple, and QFI. PDRs serve as the entry point of the entire user-plane rule chain. |
| FAR | Forwarding Action Rule | Specifies **how packets should be handled** once a PDR is matched—for example, whether to forward them, which interface or remote IP / TEID to send them to, whether to drop them, or whether to perform local processing. |
| QER | QoS Enforcement Rule | **Applies bandwidth, scheduling, and priority-related QoS settings**, and maps them to QFI / QoS flows. Through QERs, the UPF can enforce different rate limits, scheduling weights, or guarantees for different traffic types. |
| URR | Usage Reporting Rule | Responsible for **collecting usage statistics and reporting them to the SMF/CHF**, such as uplink / downlink byte and packet counts, or triggering reports when certain conditions are met (for example, thresholds exceeded or session termination), supporting charging and monitoring. |

These rules are derived in the SMF by first obtaining “policy and QoS configurations” from the PCF via SBI, then combining them with UE subscription data and the current network state. The SMF translates these inputs into concrete combinations of PDRs / FARs / QERs / URRs, which are finally delivered to the UPF via PFCP. In other words, the PCF provides “abstract policies,” while the SMF is responsible for turning them into executable user-plane rules on the UPF.

## Chapter Summary

This chapter provided a high-level overview of the division of responsibilities between the SMF and the UPF in the 5GC, and explained how mechanisms such as N3 / N4, GTP-U, PFCP, and gtp5g are used to concretely realize “PDU sessions and policies” as user-plane forwarding and charging behaviors.

In short: **the SMF designs the rules, and the UPF executes them**. The SMF combines subscription data and policies into PDR / FAR / QER / URR rules, and the UPF applies them on the N3 interface to actually receive, forward, and process packets.

With a clear understanding of this “planner and executor” pair, you will be better prepared to see how other NFs (such as the PCF, UDM, and AUSF) influence the overall control-plane and user-plane behavior from their respective perspectives.

<div class="chapter-nav">
  <a href="../chapter6/" class="nav-btn nav-next" title="Next: Other Network Functions">
    <span class="arrow"></span>
  </a>
</div>
