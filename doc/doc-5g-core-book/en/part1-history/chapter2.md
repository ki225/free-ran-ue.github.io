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

- **SA（Service & System Aspects）**

    Responsible for the design of the “overall architecture” and “service requirements.”
    This includes topics such as the overall architecture of the 5G System (5GS), how network functions are decomposed, what interfaces exist between them, and what types of services are supported.

- **RAN（Radio Access Network）**

    Focuses on “radio access technologies,” specifically the NR (New Radio) protocol stack, signaling design, and air-interface specifications.
    The commonly referenced **38.xxx series** (e.g., TS 38.300 and TS 38.331) are the specifications that define the RAN.

- **CT（Core Network & Terminals）**

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

- **5GC（5G Core）**: the primary focus of this book
- **NR（New Radio）/ NG-RAN**: the new radio access network (including gNB)

Together, these form a complete 5G system.
Therefore, the 5G System (5GS) can be understood as “the end-to-end network formed by the 5G Core (5GC) and NR.”

## 2.2 5G System Overview (5GS Overview)

Before diving into details, let’s first look at the overall structure of the 5G system from a high-level perspective.

### 2.2.1 UE – RAN – 5GC – DN: The Path from Device to Service

From a user’s point of view, when a smartphone(UE) accesses the network, the data path typically goes through the following layers:

![5gc](../../image/part1/5gc.png)

- **UE（User Equipment）**: smartphones, CPEs, IoT devices, etc.
- **RAN (Radio Access Network, i.e., gNB / NG-RAN)**: converts radio signals into packets and connects them to the core network
- **5GC（5G Core）**: handles registration, identity management, routing, policy control, and charging
- **DN（Data Network）**: such as the Internet, enterprise private networks, public cloud VPCs, or IMS networks

<div class="chapter-nav">
  <a href="../../part2-free5gc/chapter3/" class="nav-btn nav-next" title="Next：free5GC Overall Architecture and Module Introduction">
    <span class="arrow"></span>
  </a>
</div>
