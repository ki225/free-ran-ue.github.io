# Chapter 2：5G Core Network Overview from 3GPP Perspective

In Chapter 1, we started by discussing “why a core network is needed” and “the evolution and limitations of the 4G EPC.”
From this chapter onward, we will adopt a **3GPP-centric perspective** to examine the design of the entire 5G System (5GS), covering:
how standards are defined, the overall system architecture, and the meaning of commonly referenced 5G concepts such as SBA and interfaces (N1/N2/N3).

> [!Important]
> The goal of this chapter is not to make you memorize all the specification numbers, but to help you build a **mental model** of the system:  
>
> understanding “what the overall 5G system looks like” and “how the control and user planes are separated.” Detailed discussions of key concepts and the concrete roles of each Network Function (NF) will be covered later.

## 2.1 Overview of 3GPP and 5G System Standardization

To understand the 5G core network, the first step is to clarify two questions: **Who defines the specifications, and what do those specifications look like?**

### 2.1.1 What is 3GPP?

3GPP (3rd Generation Partnership Project) is an international standardization organization responsible for defining technical standards for 3G, 4G, and 5G systems.
It is not a single company, but a “collaborative partnership platform” whose members include network equipment vendors, mobile operators, chipset manufacturers, research institutions and so on.

Within 3GPP, the work is broadly divided into three major domains:

- **SA (Service & System Aspects)**

    Responsible for the design of the “overall architecture” and “service requirements.”
    This includes topics such as the overall architecture of the 5G System (5GS), how network functions are decomposed, what interfaces exist between them, and what types of services are supported.

- **RAN (Radio Access Network)**

    Focuses on “radio access technologies,” specifically the NR (New Radio) protocol stack, signaling design, and air-interface specifications.
    The commonly referenced **38.xxx series** (e.g., TS 38.300 and TS 38.331) are the specifications that define the RAN.

- **CT (Core Network & Terminals)**

    Responsible for “core network and terminal signaling protocols.”
    This includes how the UE interacts with the core network, the definition of NAS and NGAP signaling, and the detailed procedures for call and session handling.

> [!Tip]
> A simple way to remember this:
>
> - SA: defines what 5G looks like (architecture & services)
> - RAN: defines how radio transmission works
> - CT: defines how signaling flows and how procedures are executed

### 2.1.2 The Roles of TS 23.501 / 23.502 / 23.503

Among the specifications related to the 5G core network, the following three Technical Specifications (TS) are referenced most frequently:

- **TS 23.501 – System Architecture**

    This document defines the overall architecture of the 5G system, including:

    - The network functions (NFs), such as AMF, SMF, UPF, and others
    - The interfaces between them (N1, N2, N3, N4, N6, Nn, etc.)
    - How different service scenarios (eMBB / URLLC / mMTC) are supported architecturally

    TS 23.501 can be thought of as “the architecture blueprints of 5G.”

- **TS 23.502 – Procedures**

    It describes **how the system operates when events occur**, such as:

    - UE registration procedures
    - Establishment and release of PDU Sessions
    - High-level handover procedures

> [!Tip]
> A simple way to remember this:
>
> - 23.501 is “the architectural blueprint of 5G SA”
> - 23.502 is “the operational handbook describing how the system behaves in different scenarios”

### 2.1.3 The 38.x Series and RAN Signaling

The specifications discussed above are primarily core-network-oriented. On the radio access (RAN) side of 5G, you will frequently encounter documents starting with 38.xxx, for example:

- **TS 38.300**: Describes the overall architecture of NR and NG-RAN
- **TS 38.331**：Defines the RRC (Radio Resource Control) protocol, i.e., the control signaling between the UE and the gNB
- Additional 38.3xx / 38.4xx specifications cover details of the physical layer, MAC, RLC, PDCP, and related protocols

These documents primarily address “UE ↔ gNB (base station)” interactions. Since this book focuses on the **5G Core (5GC)**, RAN signaling will only be briefly referenced when necessary, mainly to help establish end-to-end context.

### 2.1.4 5GC + NR = 5GS

According to 3GPP definitions, the 5G System (5GS) consists of two main parts:

- **5GC (5G Core)**: the primary focus of this book
- **NR (New Radio)/ NG-RAN**: the new radio access network (including gNB)

Together, these form a complete 5G system.
Therefore, the 5G System (5GS) can be understood as “the end-to-end network formed by the 5G Core (5GC) and NR.”

## 2.2 5G System Overview (5GS Overview)

Before diving into details, let’s first look at the overall structure of the 5G system from a high-level perspective.

### 2.2.1 UE – RAN – 5GC – DN: The Path from Device to Service

From a user’s point of view, when a smartphone(UE) accesses the network, the data path typically goes through the following layers:

![5gc](../../image/part1/5gc.png)

- **UE (User Equipment)**: smartphones, CPEs, IoT devices, etc.
- **RAN (Radio Access Network, i.e., gNB / NG-RAN)**: converts radio signals into packets and connects them to the core network
- **5GC (5G Core)**: handles registration, identity management, routing, policy control, and charging
- **DN (Data Network)**: such as the Internet, enterprise private networks, public cloud VPCs, or IMS networks

### 2.2.2 Division of Responsibilities Between the RAN and the Core

- **The RAN acts like an “access station”**

    - Managing radio resources: who is allowed to transmit, how much data can be transmitted, and which modulation and coding schemes are used
    - Packaging each UE’s user data and forwarding it toward the core network

- **The Core acts like a “command center and traffic dispatcher”**

    - Determining the UE’s identity, whether it is authorized to access the network, and which path its traffic should follow
    - Deciding which Data Network (DN) the packets should be routed to, and what QoS and policy rules should be applied

This division of responsibilities is conceptually similar to that of the 4G era. However, in 5G, the separation between the control plane and the user plane is made much more explicit, enabling greater optimization and more flexible deployment options.

## 2.3 SBA (Service-Based Architecture)

In the 4G EPC, many network elements communicate with each other using **Diameter** or other traditional telecom protocols.
With the 5G core network, 3GPP introduced a major architectural shift: **the adoption of a Service-Based Architecture (SBA) built on HTTP and JSON**.

### 2.3.1 Why Move from Diameter to HTTP + JSON?

There are several key reasons for this transition:

- Bringing telecom networks closer to the IT and cloud technology stack, making it easier to integrate existing tools and ecosystems
- Making interactions between network elements more “microservices-like”: each network element exposing a clearly defined set of APIs
- Offering advantages over Diameter in terms of multiplexing, latency characteristics, and deployment flexibility

For developers and researchers, this also means that parts of the 5G core network logic can be understood and implemented using familiar web technologies.

### 2.3.2 What Is a Service-Based Interface (SBI)?

In an SBA, each Network Function (NF) exposes its capabilities as services, for example:

- AMF provides services related to UE registration and access management
- SMF provides services related to PDU session management
- PCF provides services related to policy control

These services communicate through a unified conceptual interface known as the **Service-Based Interface (SBI)**. In practice, an SBI is implemented as a set of HTTP + JSON APIs.

> [!Tip]
> You can think of SBA as follows: “Each network function in the 5G core acts as a service provider, and they invoke each other using REST-style APIs.”

### 2.3.3 Advantages of SBA

Compared to traditional point-to-point, interface-specific interactions between network elements, SBA offers several clear advantages:

- **Scalability**: When the load on a particular NF increases, additional instances can be deployed. As long as they are registered with the NRF, they can be used dynamically.
- **Replaceability**: Multiple implementations or versions of the same NF can coexist, potentially from different vendors, with compatibility achieved through standardized APIs.
- **Interoperability**: By adhering to common API specifications, different implementations can interoperate more easily, reducing vendor lock-in.
- **Testability**: Developers and testers can validate individual NFs or specific APIs in isolation, without having to deploy the entire system end to end.

## 2.4 5G Interfaces and Protocol Classification (N1 / N2 / N3 / N4 / N6 / SBA)

This section is one of the core parts of the chapter. We organize the commonly used 5G interfaces based on the distinction between the “control plane and the user plane.”

![5gc-n](../../image/part1/5gc-n.png)

### 2.4.1 Control Plane Interfaces

The control plane is primarily responsible for “signaling and coordination,” rather than carrying large volumes of user data. Common control-plane interfaces include the following:

- **N1：UE ↔ 5GC (NAS)**

    - This interface carries high-level control signaling between the UE and the 5GC, such as registration procedures and PDU session establishment requests.
    - The signaling content is referred to as **NAS (Non-Access Stratum)** and is transported via the RAN to the AMF.

- **N2：RAN ↔ 5GC (NGAP)**

    - This is the control interface between the gNB and the 5GC, specifically the AMF.
    - It uses the **NGAP** protocol, which handles functions such as UE context setup, handover control, and PDU Session–related control signaling.

- **N4：SMF ↔ UPF (PFCP)**

    - This interface is used by the SMF to instruct the UPF on how to establish or update forwarding rules, such as which Data Network (DN) packets should be routed to and which QoS parameters should be applied.

- **Nn：5GC NF ↔ NF (SBA / HTTP/2)**

    - This is a collective term representing service-based interactions between network functions within the 5GC.
    - These interactions are implemented using HTTP/2 and JSON, i.e., the Service-Based Interface (SBI) discussed earlier; typical examples include interactions between the AMF and SMF, or between the SMF and PCF.

### 2.4.2 User Plane Interfaces

The user plane represents the path that “actually carries user data, such as packets for video streaming, web browsing, and data transfer”:

- **N3：RAN ↔ UPF (GTP-U)**

    - This interface provides the user-plane tunnel between the gNB and the UPF.
    - It uses the GTP-U protocol to encapsulate UE IP packets, similar to the S1-U interface in 4G.

- **N6：UPF ↔ DN (Data Network)**

    - This interface connects the UPF to external data networks, such as the public Internet or enterprise private networks.

## 2.5 Overview of Key 5G Core Procedures

This section introduces several of the most important procedures in 5G at a “conceptual level.” Rather than diving into the detailed behavior of each network element, the goal here is to help you first see the overall storyline of how the system operates.

### 2.5.1 Registration Procedure (NAS / NGAP)

Registration can be viewed as the “registration / login procedure” in 5G. At a high level, it consists of the following key steps:

1. **UE initiates a Registration Request**: When the UE powers on or enters a 5G network, it sends a NAS Registration Request to the AMF via the RAN.
2. **UE context establishment**: After receiving the request, the AMF creates a “context” for the device, which includes information such as identity, location, and capabilities.
3. **Security-related procedures**: These include authentication and security mode control, ensuring that subsequent signaling and data transmission are protected and encrypted.
4. **Registration Complete**: After all required exchanges and configurations are completed, the UE responds with Registration Complete to indicate that it has successfully “logged in” to the 5G network.

> [!Tip]
> The **registration** procedure is the first access procedure initiated by the UE toward the core network when it powers on or exits airplane mode.

### 2.5.2 PDU Session Procedure

In 4G, we often refer to **bearers**. In 5G, the corresponding core concept is the **PDU session**, which you can think of as “a **logical connection** between the UE and a specific Data Network (DN).”

At a high level, the procedure works as follows:

1. **UE requests PDU session establishment**: When the UE wants to access the Internet or connect to a specific service, it sends a PDU Session Establishment Request.
2. **SM control-plane decisions**: The SMF in the core network determines which UPF and which DN the PDU session should be associated with, as well as the QoS parameters to be applied.
3. **User-plane path establishment**: The user-plane path of the PDU session is constructed through N3 (RAN ↔ UPF) and N6 (UPF ↔ DN).

Afterward, the UE’s data packets are forwarded along this PDU session. A single UE may have multiple PDU sessions simultaneously, each corresponding to a different service or network.

> [!Tip]
> Establishing a PDU session is essentially about creating a logical connection from the UE to a Data Network (DN)!

### 2.5.3 Handover Concept

Handover refers to the process by which a UE moves from one base station to another without interrupting the ongoing connection as it travels.

At this level, it is sufficient to understand two key aspects:

- **Control-plane relocation**: The system updates the UE’s location information so that the core network knows which RAN the UE is currently connected through.

- **User-plane path switching**: To ensure that packets continue to reach the UE, the forwarding path is switched from the old gNB to the new gNB. In some cases, the user-plane path at the UPF may also need to be updated.

<div class="chapter-nav">
  <a href="../../part2-free5gc/chapter3/" class="nav-btn nav-next" title="Next：free5GC Overall Architecture and Module Introduction">
    <span class="arrow"></span>
  </a>
</div>
