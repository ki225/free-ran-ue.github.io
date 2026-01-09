# Chapter 1：From 4G to 5G — Evolution of Core Network Architecture

## 1.1 What is core network(CN)?
We can think of the core network as “the brain of a telecommunications system.” It is responsible for making decisions and coordinating actions between user equipment (UE)—such as smartphones—and the external world (e.g., the Internet, enterprise private networks, and other operators’ networks). When a phone powers on, registers, accesses the Internet, makes a call, or uses apps, most of the key control logic behind these actions is handled by the core network.

The functions of the core network can be broadly grouped into several aspects:

1. **User Registration**

    The core network identifies “who you are” and checks a set of subscriber-related conditions. For example, whether you are a legitimate subscriber, whether your subscription (phone number/MSISDN) is valid, whether there are unpaid bills, and whether roaming is permitted, among others. In 4G/5G systems, this typically involves **authentication**, **registration/attach**, and establishing a corresponding **session/context** for each subscriber.
    
2. **Resource Management**

    When a user starts transmitting data (e.g., watching YouTube or joining a Teams meeting) or sets up a voice call, the core network allocates appropriate resources for the traffic. This includes:

    - Establishing and releasing EPS bearers (4G) / PDU sessions (5G)
    - Configuring Quality of Service(QoS) parameters (e.g., latency, priority, and guaranteed bit rate)
    - Coordinating radio resource usage with the RAN (e.g., determining which users receive higher priority)

3. **Traffic Steering**

    The core network determines “where packets should go”. For example:

    - Which P-GW (4G) / UPF (5G) Internet traffic should be forwarded to
    - Whether specific services (e.g., VoLTE, VoNR, IMS) should be delivered to dedicated application servers

    In 5G architecture, this also involves designs such as the **Uplink Classifier (UL CL)** and **traffic steering**.

4. **Charging**

    Telecom operators need to know how much data each user consumes, how long calls last, and whether certain value-added services are used, in order to perform charging and settlement. The core network:

    - Collects usage information (traffic, time, events)
    - Generates charging records
    - Interacts with charging systems to support online and offline charging

In summary, the core network serves the same essential purpose across 2G through 5G: it **identifies subscribers, allocates resources, determines routing, and enables charging**. The difference across generations lies in the continuous evolution of protocols, interfaces, and functional decomposition, which allow the core network to support more diverse services, finer-grained QoS, and greater flexibility and programmability.

## 1.2 4G Evolved Packet Core (EPC)

### 1.2.1 EPC Architecture Overview
In 4G systems, the Evolved Packet Core (EPC) serves as the central component that connects the radio access network (eNodeB) with external networks such as the Internet and enterprise private networks. The EPC consists of several key network functions, including the HSS, MME, S-GW, P-GW, and the eNodeB, we will introduce them in the following content.

![epc](../../image/part1/epc.png)

- HSS（Home Subscriber Server）

    - Storing subscriber information, such as International Mobile Subscriber Identity(IMSI), Mobile Station International Subscriber Directory Number(MSISDN), Service Permissions, and roaming permissions.
    - Maintaining authentication keys and security parameters used during UE authentication.
    - Providing subscription and roaming information to the MME to support mobility and access control decisions.

- MME（Mobility Management Entity）

    - Handling UE attach, detach, and registration procedures.
    - Managing mobility, including tracking the UE’s serving eNodeB and coordinating handovers.
    - Authenticating subscribers through interaction with the HSS and determining whether network access is permitted.
    - Coordinating with the S-GW and P-GW during bearer establishment to create the appropriate EPS bearers.

- S-GW（Serving Gateway）

    - Forwarding user-plane traffic between the eNodeB and the P-GW
    - Serving as a stable mobility anchor during.inter-eNodeB or inter-area mobility, reducing the need for frequent path re-establishment.
    - Supporting partial charging and traffic accounting functions for downstream billing systems.

- P-GW（Packet Data Network Gateway）

    - Assigning IP addresses to UEs to enable communication with the Internet or enterprise networks.
    - Enforcing policy and charging control in coordination with the PCRF, including bandwidth limitations, service blocking or allowance, and traffic shaping.
    - Usually for enterprise private network, IMS system and entry for every application server.

- Evolved Node B (eNodeB)

    - The eNodeB represents the 4G base station
    - Managing radio resources, including scheduling, modulation and coding, and power control.
    - Converting radio signals from the UE into IP packets and forwarding them to the S-GW via the S1-U interface.
    - Signaling with the MME over the S1-MME interface, including RRC connections, attach procedures, and tracking area updates (TAU).
    - Supporting the X2 interface between neighboring eNodeBs for handover coordination and load balancing.

### 1.2.2 Limitations of 2G/3G/4G Core Networks

Although 2G, 3G, and 4G core networks successfully support voice, messaging, and mobile broadband services, their architectures and functional capabilities exhibit several fundamental limitations. These constraints make it difficult for legacy core networks to meet the high flexibility, high efficiency, and service diversity requirements targeted by 5G.

- Closed Architecture and Tightly Coupled Network Functions

    - Core functions are implemented within a small number of large network functions (e.g., SGSN/GGSN or MME/S-GW/P-GW), resulting in tight coupling between components.
    - Introducing new features or optimizing data paths is often constrained by vendor-specific implementations and rigid interfaces, limiting architectural flexibility.

- Incomplete Separation of Control and User Planes

    - Although 4G EPC introduces a logical separation between the control plane (MME) and the user plane (S-GW/P-GW), implementation and deployment often remain based on integrated appliance style.
    - This limits support for advanced scenarios such as centralized control, distributed traffic breakout, and mobile edge computing (MEC)

- Limited Service Diversity and QoS Flexibility

    - Legacy core networks primarily support mobile broadband and traditional voice and messaging services, with insufficient native support for IoT, ultra-reliable low-latency communications (URLLC), and vertical industry private network scenarios.
    - While QoS differentiation exists, fine-grained and programmable policy control for different services and enterprise customers is difficult to achieve

- Hardware-Constrained Scalability and Deployment Pattern

    - Legacy core networks are commonly deployed on dedicated hardware or virtual machines, favoring vertical scaling (scale-up) over horizontal scaling (scale-out).
    - This makes it difficult to rapidly adapt to traffic surges or temporary events, such as concerts or large sporting events, and limits the feasibility of elastic, on-demand cloud-native deployments.

- Insufficient Support for Cloud-Native Design and Network Slicing

    - The design of 2G, 3G, and 4G core networks assumes a single shared network serving all users, rather than a multi-tenant, multi-slice environment.
    - Customizing core network behavior or allocating isolated resource slices for different enterprises or industries is costly, inflexible, and difficult to automate.

As a result of the limitations described above, when mobile networks evolved from a single, best-effort public service toward diverse vertical and industry-specific use cases, existing 2G/3G/4G core network architectures increasingly struggled to meet these demands. This evolution became a key motivation for the redesign of the 5G Core (5GC) and the introduction of service-based architecture (SBA), cloud-native principles, and network slicing.


<div class="chapter-nav">
  <a href="../chapter2/" class="nav-btn nav-next" title="Next：5G Core Network Overview from 3GPP Perspective">
    <span class="arrow"></span>
  </a>
</div>
