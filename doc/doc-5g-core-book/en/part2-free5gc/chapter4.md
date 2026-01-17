# Chapter 4: AMF Deep Dive

## 4.1 The Role of AMF in the Core Network

![sbi](../../image/part2/sbi.png)

In the 5G Core Network, the AMF (**Access and Mobility Management Function**) is the “first stop” for all UEs entering the 5GC. When a mobile device powers on, initiates registration, establishes a PDU session, or moves into a new area, almost all related control signaling passes through the AMF first. The AMF then coordinates subsequent processing with other NFs (such as SMF, AUSF, UDM, PCF, and others).

From a functional perspective, the AMF is mainly responsible for the following:

- **Access control and registration management**: Handling UE registration, updates, and de-registration, and deciding whether a UE is allowed to access the network.
- **Mobility and connection management**: Tracking the UE’s Tracking Area (TA), managing RRC / NAS connection states, and assisting with handover and reselection procedures.
- **Security and authentication coordination**: Working with NFs such as AUSF and UDM to complete UE authentication and establish security contexts, ensuring protection of signaling and data across the radio and core network domains.
- **Session establishment coordination**: When a UE requests to establish a PDU session, the AMF selects an appropriate SMF based on policy and topology, and helps establish the context between the UE and the SMF / UPF.

In terms of interfaces, the AMF connects **N1 (NAS signaling between UE and AMF)** on one side and **N2 (NGAP signaling between gNB and AMF)** on the other, while also communicating with other core NFs via various Nn interfaces over the SBA. You can therefore think of the AMF as the “front desk and traffic control center of the 5GC control plane,” translating requests from the radio side into internal core network service invocations.

> [!Caution]
> Although NAS are messages exchanged between the UE and the AMF, they are actually encapsulated by the RAN inside NGAP messages and delivered to the AMF’s N2 interface over SCTP! The UE does not send NAS messages directly to the AMF!

## 4.2 The N2 Interface and Related Protocols

![protocol](../../image/part2/protocol.png)

## 4.2.1 N2

The N2 interface is the control plane interface between the 5G RAN (gNB) and the 5GC control plane (AMF). It mainly carries:

- Signaling coordination between the gNB and AMF, such as UE attach, tracking area updates, and handover procedures
- Bearing UE NAS messages, which are **encapsulated within NGAP messages and forwarded to the AMF for processing**

In the protocol stack, the N2 interface typically uses **SCTP** as the transport-layer protocol over IP, with **NGAP** running at the application layer. **NAS** messages between UE and core network are encapsulated inside NGAP messages and relayed to the AMF via N2, as illustrated in the protocol stack diagram above.

### 4.2.2 SCTP

SCTP (Stream Control Transmission Protocol) is a message-oriented transport-layer protocol that supports multi-homing and multi-streaming. In the 5G N2 interface, it is used to carry NGAP messages. Compared to the more commonly known TCP, SCTP has several characteristics that make it particularly suitable for telecom signaling:

- **Multi-streaming**: Multiple independent streams can be established within a single connection, reducing head-of-line blocking and preventing different types of signaling from blocking each other.
- **Multi-homing**: An endpoint can bind to multiple IP addresses, allowing rapid failover to a backup path in case one path fails, thereby improving reliability.
- **Message-oriented with explicit message boundaries**: This aligns well with the “one signal per message” usage model, making telecom protocol implementation more straightforward.

In practice, the gNB and the AMF each establish an SCTP connection on dedicated N2 IP / port pairs, and all subsequent NGAP messages are exchanged over this connection.

### 4.2.3 NGAP

NGAP (Next Generation Application Protocol) is the application-layer signaling protocol running on the N2 interface. It defines the message formats and procedures used by the gNB and AMF for various control-plane processes. Common NGAP procedures include:

- Initial UE Message / Initial Context Setup: Used for context establishment during initial UE registration or PDU session setup
- UE Context Release: Releasing UE context between the gNB and AMF
- Handover Preparation / Handover Resource Allocation: Coordinating signaling during UE handover between different gNBs

Within the free5GC AMF, the NGAP module is responsible for:

- Decoding NGAP PDUs received over SCTP
- Dispatching messages to the appropriate handlers based on PDU type (for example, handling Initial UE Message or UE Context Release)
- Extracting the NAS payload (N1 messages) from NGAP and passing it to the NAS processing logic

### 4.2.4 NAS

NAS (Non-Access Stratum) is the control-plane signaling layer between the UE and the 5GC. From the UE’s perspective, it is the layer that “talks directly to the core network.” NAS is responsible for:

- UE registration, update, and de-registration procedures
- Requests and responses for PDU session establishment, modification, and release
- Control procedures such as Security Mode Control and Identity management

In terms of the delivery path, NAS messages are transmitted as follows:

1. Between the UE and the gNB, NAS messages are encapsulated in RRC messages
2. Upon reaching the gNB, the NAS payload is wrapped inside appropriate NGAP messages
3. The messages are sent to the AMF over N2 / SCTP, where the AMF’s NAS logic decodes and processes them

> [!Note]
> As shown in the protocol stack diagram above, NAS messages can be divided into two types:
>
> - NAS-MM: Messages from the UE to the AMF
> - NAS-SM: Messages from the UE to the SMF, however, NAS-SM messages are still received by the AMF first and then forwarded to the SMF via the SBI

## 4.3 AMF Startup Procedure

> [!Note]
> The startup procedure described here applies to all SBI-based NFs except the NRF. When other NFs are introduced later, this procedure will not be repeated in detail.

In the free5GC implementation, the AMF startup procedure can be roughly divided into the following steps, which align closely with the generic NF structure introduced in Chapter 3:

1. **Load Configuration and Initialize the Logger**

    The startup program first loads the AMF configuration files (such as local IP / port, PLMN / TA lists, NRF address, SCTP parameters, etc.), and initializes the logging system accordingly to ensure consistent log output across all modules.

2. **Create the AMF Context**

    Next, the global AMF context is created, which includes:

    - Network topology and PLMN / TA configuration
    - SBI client settings required for communication with other NFs
    - Internal caches, state tables, and timers

3. **Initialize the SBI Server and Register with the NRF**

    The AMF starts its SBI HTTP server to expose services externally, and at the same time registers its service capabilities and endpoints with the NRF, allowing other NFs to discover and invoke the AMF via the SBA.

4. **Start N2 (SCTP / NGAP) Modules**

    Starts the SCTP listener on the N2 interface, initializes the NGAP protocol handling modules, and prepares to receive NGAP messages from the gNB along with the encapsulated NAS payloads.


5. **Enter the Event Loop to Handle Signaling and Errors**

    Finally, the AMF enters the main event loop, continuously handling:

    - NGAP / NAS messages from N2
    - SBI requests from other NFs
    - Internal timer events and state machine updates

When reading the AMF source code, you can follow these steps from the startup entry point to trace the code paths for configuration loading, context creation, SBI registration, and N2 listener initialization.

## 4.4 AMF FSM State Machine

In its implementation, the AMF makes extensive use of **FSMs (Finite State Machines)** to manage UE states and signaling flows. The reason is straightforward:

For each UE, the journey from “not yet connected” to “successfully registered, with PDU sessions established, and eventually de-registered” is essentially a sequence of transitions between different states. Each state only allows specific events and transitions. Without a clear state machine, the code would easily degrade into complex and hard-to-maintain chains of if-else statements.

The current state machine in free5GC’s AMF can be roughly illustrated by the diagram below:

![fms](../../image/part2/fsm.png)

The AMF updates the UE context state according to this state machine, starting from the initial de-registered state. In the diagram, the three colors represent:

- Orange: An event is triggered, transitioning to the next state
- Green: The event succeeds, transitioning to the next state
- Red: The event fails, reverting to a previous state (often back to the initial state)

> [!Tip]
> You don’t need to memorize the FSM in detail. A simple way to think about it is:
>
> 1. The phone powers on and starts registration
> 2. Basic information is verified on both sides
> 3. Security mode is confirmed
> 4. A UE record instance is created in the AMF
> 5. Registration succeeds!

## 4.5 AMF Operational Responsibilities

> [!Note]
> In this section, we walk through AMF behavior from the UE’s perspective, explaining how the AMF processes messages from the UE and the RAN. By using everyday analogies, we aim to make it easier for beginners to understand how the core network handles this information.

### 4.5.1 UE Registration

We start with the most common and critical procedure: **UE registration**. You can think of registration as “checking in at an airport counter.” The UE presents its identity and requirements to the AMF, and after validation, the AMF creates a “passenger profile” that serves as the basis for all subsequent actions.

From the UE’s perspective, registration roughly proceeds as follows:

1. When the UE powers on or enters a state requiring service, it establishes a connection with the gNB via RRC and sends a **Registration Request (NAS message)** to the gNB.  
2. The gNB encapsulates this NAS payload into an Initial UE Message (NGAP) and forwards it to the AMF over N2 / SCTP.  
3. Upon receiving the message, the AMF evaluates the registration cause, PLMN / TA information, and UE capabilities, decides whether to accept the UE, and triggers authentication and subscription data retrieval with AUSF / UDM.
4. Once security and subscription checks succeed, the AMF sends a Registration Accept to the UE and creates a corresponding UE context internally (including identity information, TA lists, and security parameters).

As long as the UE remains in the “registered” state, the AMF handles various update and mobility-related events according to the FSM design.

### 4.5.2 UE PDU Session Establishment

After registration, the UE must establish at least one **PDU Session** to actually access data services. This can be likened to “collecting a boarding pass and seat assignment after airport check-in.”  
The UE specifies its desired connectivity (such as DNN / slice and QoS), and the AMF is responsible for selecting suitable SMF / UPF instances and coordinating end-to-end path establishment.

From the UE’s perspective, PDU session establishment works roughly as follows:

1. The UE sends a PDU Session Establishment Request (NAS-SM message), containing information such as DNN, S-NSSAI, and requested SSC mode, to the AMF via N2.  
2. The AMF verifies the UE’s registration and security status, and selects an appropriate SMF based on policy and topology (possibly querying the NRF for available SMF instances).  
3. The AMF sends a PDU session creation request to the selected SMF via the SBI (for example, `Nsmf_PDUSession_Create`) and waits for the SMF to respond with the allocated IP address, UPF information, and QoS settings.  
4. Finally, the AMF responds to the UE via NAS (PDU Session Establishment Accept) and coordinates with the gNB to complete the corresponding N3 / N9 paths and QoS configuration.

In this process, the AMF acts as a “**proxy and coordinator**.” It does not directly handle user-plane packets, but instead translates UE service requirements into actual routing and resource allocation performed by the SMF / UPF.

### 4.5.3 UE De-registration

Every journey eventually comes to an end. When the UE powers off, explicitly disconnects, or is released by the network due to inactivity, it enters the **de-registration** procedure. The goal of this step is to gracefully clean up resources on both the UE and network sides.

From the UE’s perspective, de-registration generally involves:

1. The UE actively sends a Deregistration Request (NAS-MM message), or the network triggers de-registration under certain conditions.  
2. After receiving the request, the AMF:

    - Notifies the relevant SMF / UPF instances to release associated PDU sessions and resources  
    - Updates it own UE context state (from “registered” back to “de-registered”)  
    - Clears cached state in other NFs (such as PCF and UDM) if necessary

3. Finally, the AMF sends a De-registration Accept to the UE, marking the official end of this “journey.”

Through a complete de-registration procedure, the network ensures that resources are properly reclaimed, avoiding leftover contexts or QoS configurations caused by abnormal disconnections or timeouts.

### 4.5.4 Other Procedures

In addition to the core procedures described above, the AMF also handles many other mobility- and access-related operations, such as:

- Tracking Area Update (TAU) and related timer management  
- Paging (call for idle UEs) and service request procedures  
- Handover procedures  

These functions play important roles in day-to-day network operation. However, in this book we first focus on the three main flows—**registration, PDU session establishment, and de-registration**—to help readers clearly understand the core interactions between the AMF and the UE before diving deeper into other features.

## 4.6 Chapter Summary

This chapter explained how the AMF, as the “first stop” for UEs entering the 5GC, connects N1 / N2 signaling on one side and collaborates with NFs such as SMF, AUSF, UDM, and PCF over the SBA on the other, completing access control, mobility management, security coordination, and PDU session establishment.

We then analyzed the AMF from an implementation perspective through the N2 protocol stack (SCTP / NGAP / NAS), the startup procedure, and the FSM state machine, breaking down how the AMF processes messages and manages the UE lifecycle.

Finally, from the UE’s point of view, this chapter walked through the three main flows—registration, PDU session establishment, and de-registration—and briefly highlighted other procedures such as TAU, paging, and handover, providing the necessary background for subsequent deep dives into NFs such as the SMF and UPF.

<div class="chapter-nav">
  <a href="../chapter5/" class="nav-btn nav-next" title="Next: SMF, UPF and User Plane Procedures">
    <span class="arrow"></span>
  </a>
</div>
