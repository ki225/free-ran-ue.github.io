# Chapter 11: Architecture Design of free-ran-ue

## 11.1 Design Philosophy of free-ran-ue

![architecture](../../image/part4/free-ran-ue-architecture.png)

The core objective of free-ran-ue is to provide core network users with a **lightweight, reproducible, and stable** RAN / UE simulator for both control-plane and user-plane testing. By adopting a **decoupled** RAN / UE architecture, it more faithfully reproduces the interaction procedures and data forwarding behavior observed when a real UE connects to a gNB.

In addition, free-ran-ue provides a **visualized web-based management interface**, enabling users to more intuitively manage and monitor UE connection states within the simulator.

## 11.2 Software Architecture of free-ran-ue

The following structure shows the layout of the [free-ran-ue main repository](https://github.com/free-ran-ue/free-ran-ue):

```bash
.
├── cmd <= CLI control logic program
├── config <= User configuration files
├── console <= Visualized management interface
├── constant <= Global constants
├── gnb <= RAN (gNB) runtime program
├── logger <= Logging utilities
├── main.go <= Main program entry point
├── Makefile <= Build instructions
├── model <= Configuration structure models
├── script <= Scripts
│   ├── 10k-test-script <= Large-scale UE  scripts
│   └── namespace-script <= Namespace scripts
└── ue <= UE runtime implementation
```

## 11.3 RAN (gNB)

![ran](../../image/part4/free-ran.png)

### 11.3.1 Architectural Overview

In a 5G system, the role of the gNB can be summarized in one sentence: **correctly connect the UE’s control-plane signaling and user-plane traffic to the core network, and forward data to the appropriate destination when required.**

In free-ran-ue, the RAN (gNB) follows the same principle, but abstracts the “radio air interface” in a controllable manner, allowing typical procedures, for example registration, session establishment, data transmission, and traffic switching / offloading, to be reproduced without relying on physical base stations / real UEs.

You can think of the gNB as a combination of two functional roles:

- **Control-plane protocol endpoint**: Responsible for handshaking with the AMF, carrying NGAP signaling, and maintaining UE control-plane context on the gNB side.
- **User-plane forwarder**: Responsible for “encapsulating packets sent from UE into GTP-U” toward the UPF, and decapsulating GTP-U packets from the UPF back to the UE.

### 11.3.2 Basic Connectivity

To bring up both the control plane and user plane, the gNB maintains four fundamental connections / channels (from the perspective of “interaction targets,” it can be divided into the core network side and the UE side.):

#### To Core Network

- **Control Plane**: The gNB communicates with the **AMF** via **NGAP**, supporting NG Setup, UE registration signaling, and PDU Session resource management.
- **Data Plane**: The gNB establishes **GTP-U** tunnels with the **UPF** for user data encapsulation, decapsulation, and forwarding.

#### To UE (User Equipment)

- **Control Plane**: Carries the UE’s **NAS** signaling flows (such as registration and PDU Session control messages)
- **Data Plane**: Carries UE user-plane packets (it can be regarded as the transmission and reception of “raw IP packets”), which the gNB maps onto GTP-U tunnels toward the core network.

### 11.3.3 Typical Procedures (From “Working” to “Testable”)

This subsection uses “the procedures most frequently examined during core network testing” to illustrate what the gNB does. Message-level details are omitted, instead, each step is described in terms of its purpose and outcome.

#### (1) NG Setup: gNB “bootstrapping” with the AMF

After startup, the gNB must announce itself to the AMF as an available base station by performing NG Setup.

The goal of this step is to establish a basic cooperative relationship (such as supported capabilities, served PLMNs, and TACs) so that subsequent UE registration and session procedures have a valid foundation.

#### (2) UE Registration: establishing UE control-plane context

When a UE attaches, the gNB carries NAS signaling and facilitates the UE registration procedure.

From a testing perspective, once this step completes, you typically expect AMF / UDM / AUSF-related procedures to be triggered correctly, and the gNB to maintain the UE’s control-plane context.

#### (3) PDU Session Establishment: preparing the user-plane path

During PDU Session establishment, the core network allocates user-plane resources, and the gNB receives the corresponding tunnel information so that UE traffic can be correctly routed to the UPF.

The key concept here is that the user plane relies on **TEIDs** as identifiers for tunnel selection and forwarding.

#### (4) User-plane forwarding: encapsulation / decapsulation and TEID-based routing

Once tunnels are established, the gNB encapsulates UE uplink packets into GTP-U toward the UPF, and decapsulates downlink GTP-U packets from the UPF back to the UE.

This is the critical data path for stress testing, throughput / latency evaluation, and multi-UE concurrency testing.

### 11.3.4 Xn Interface

When implementing inter–base station interactions, gNBs must be able to exchange the necessary information that determines “how user-plane traffic should be steered or forwarded between them.”

In the current implementation of free-ran-ue, the Xn interface focuses on **exchanging TEIDs and essential forwarding information**, enabling correct handling of offloaded user-plane packets.

For each gNB, the Xn interface is bound to the address and port configured in the YAML file, for example:

```yaml
xnIp: "10.0.1.2"
xnPort: 31415
```

This listener uses **TCP**. From a user’s perspective, it is sufficient to know that “the Xn address / port can be specified in the configuration file.” When extending additional Xn interactions in the future, development can proceed directly from the Xn message exchange and forwarding logic.

### 11.3.5 How the gNB Records UE State

- **Regular UEs (served directly by this gNB)**: The gNB needs to maintain both control-plane identifiers (e.g., IDs used when coordinating with the AMF) and user-plane tunnel information (uplink / downlink TEIDs, UE data-plane addresses, etc.).
- **Xn UEs (offloaded from other gNBs)**: These entries are primarily for “user-plane forwarding purposes,” where the key information consists of TEIDs and data-plane addresses, enabling correct packet forwarding between the core network and the UE.

## 11.4 UE

![ue](../../image/part4/free-ue.png)

This section explains how the UE module in free-ran-ue interacts with the gNB, from the perspective of “what the UE side actually needs to do,” and how responsibilities are divided between the control plane / user plane.

### 11.4.1 Basic Connectivity (UE ↔ gNB)

After connecting to the gNB, the UE maintains two core connections (which can also be understood as two channels):

- **Control Plane**: Used for signaling exchange, authentication, and session management–related procedures.
- **Data Plane**: Used for user data transmission; traffic is exchanged through established PDU Sessions.

### 11.4.2 Typical Procedures

In the most common testing scenarios, the UE typically goes through the following sequence:

- **UE Registration**: The initial registration procedure, allowing the UE to attach to the 5G network and establish control-plane state.
- **PDU Session Establishment**: Establishment of the user-plane data connection (PDU Session) and the local 5G core network ingress interface (`ueTun0`), enabling user-plane communication.

### 11.4.3 GTP-U Responsibilities (UE Does Not Handle GTP)

In the free-ran-ue design, **the UE does not participate in any GTP procedures**.

All **GTP-U**–related encapsulation, decapsulation, and tunnel handling are performed entirely by the **gNB**. The UE only needs to focus on the generation and reception of “control-plane signaling” and “raw user data.”

## 11.5 Web Console

![console](../../image/part4/free-console.png)

The console provided by free-ran-ue is effectively a management interface that continuously retrieves information from gNB APIs. By configuring which gNBs are to be managed in the console, the console backend directly pulls status data from the target gNBs, including both gNB operational information and connected UE information, and presents this data in a visualized form—reducing observability overhead during integration testing.

From a “user-experience” perspective, the Web Console addresses a very practical pain point: when running multiple gNBs, multiple UEs, or even multiple scenario scripts simultaneously, relying solely on terminal logs makes it difficult to quickly answer questions such as “How many UEs are currently connected?”, “Which UE is stuck in which procedure?”, or “Is the user plane actually forwarding traffic?”. The console allows issues to be quickly localized via a “status dashboard,” after which logs and traces can be used for deeper analysis.

### 11.5.1 Operating Model (Polling / Pull-based)

The console adopts “a pull-based model,” where the console actively retrieves data, rather than having gNBs push data to it. The advantages of this design include:

- **Simpler deployment**: As long as the console can reach the gNB APIs, it can begin managing and displaying status.
- **Consistent multi-gNB management**: Adding a gNB to be managed is essentially equivalent to adding a new API data source.
- **Test-friendly behavior**: In CI or test environments, pull-based models are generally easier to deploy across network boundaries and NAT / firewall constraints.

### 11.5.2 Common Observability Data

- **gNB information**: Basic configuration, runtime state, and established core-network and user-plane connectivity.
- **UE lists and status**: Number of connected UEs, UE registration / session states, and key user-plane indicators (e.g., whether corresponding data paths have been successfully established).

## 11.6 Chapter Summary

This chapter decomposed free-ran-ue into three core components from an architectural perspective: RAN (gNB), UE, and the Web Console. Together, they can be understood as a complete “testable data path”: the UE generates control-plane / user-plane behavior; the gNB anchors control-plane signaling to the AMF and user-plane traffic to the UPF via GTP-U; and the console provides an accessible entry point for observing and managing the overall system state.

After completing this chapter, you should at least grasp the following key points:

- **Separation of control plane and user plane**: The UE focuses on signaling and user data, while GTP-U encapsulation / decapsulation, and tunnel handling are the responsibility of the gNB.
- **Observation points in typical procedures**: Successful UE registration indicates control-plane context establishment; only after PDU Session establishment does a stable user-plane data path (e.g., traffic via `ueTun0`) exist.
- **Role of the console**: By pulling status information from gNB APIs using a pull / polling model, the console enables rapid visibility into gNB / UE connectivity and shortens troubleshooting time.

The next chapter builds on this architecture and walks through connecting free-ran-ue to free5GC in practice, including configuration, startup order, and common integration issues and troubleshooting guidance, turning the simulator into a truly reusable system-level testing tool.

<div class="chapter-nav">
  <a href="../chapter12/" class="nav-btn nav-next" title="Next: How the Simulator Integrates with free5GC">
    <span class="arrow"></span>
  </a>
</div>
