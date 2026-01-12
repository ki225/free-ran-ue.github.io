# Chapter 1: From 4G to 5G — Evolution of Core Network Architecture

## 1.1 What is core network (CN)?
We can think of the core network as “the brain of a telecommunications system.” It is responsible for making decisions and coordinating actions between user equipment (smartphones, UE) () and the external world (e.g., the Internet, enterprise private networks, and other operators’ networks). When a phone powers on, registers, accesses the Internet, makes a call, or uses apps, most of the key control logic behind these actions is handled by the core network.

The functions of the core network can be broadly grouped into several aspects:

1. **User Registration**

    The core network identifies “who you are” and checks a set of subscriber-related conditions. For example, whether you are a legitimate subscriber, whether your phone number is valid, whether there are unpaid bills, and whether roaming is permitted, among others. In 4G/5G systems, this typically involves **authentication**, **registration/attach**, and **session/context** for each subscriber.
    
2. **Resource Management**

    When a user starts transmitting data (e.g., watching YouTube or joining a Teams meeting) or sets up a voice call, the core network allocates appropriate resources for the traffic. The Core Network is responsible for:

    - Establishing and releasing EPS bearers (4G) / PDU sessions (5G)
    - Configuring Quality of Service (QoS) parameters (e.g., latency, priority, and guaranteed bit rate)
    - Coordinating radio resource usage with the RAN (e.g., determining which users receive higher priority)

3. **Traffic Routing**

    The core network determines “where packets should go”. For example:

    - Which P-GW (4G) / UPF (5G) Internet traffic should be forwarded to
    - Whether specific services (e.g., VoLTE, VoNR, IMS) should be delivered to dedicated application servers

    In 5G architecture, this also involves designs such as the **Uplink Classifier (UL CL)** and **traffic steering**.

4. **Charging**

    Telecom operators need to know how much data each user consumes, how long calls last, and whether certain value-added services are used, in order to perform charging and settlement. The core network:

    - Collecting usage information (traffic, time, events)
    - Generating charging records
    - Interacting with charging systems to support online and offline charging

In summary, the core network serves the same essential purpose across 2G through 5G: it **identifies subscribers, allocates resources, determines traffic routing,** and **charging**. The difference across generations lies in the continuous evolution of protocols, interfaces, and functional decomposition, which allow the core network to support more diverse services, finer-grained QoS, and significantly higher levels of flexibility and programmability.

## 1.2 4G Evolved Packet Core (EPC)

### 1.2.1 EPC Architecture Overview
In 4G systems, the EPC (Evolved Packet Core) serves as the central component that connects the radio access network (eNodeB) with external networks such as the Internet and enterprise private networks. The EPC consists of several key network functions, including the HSS, MME, S-GW, P-GW, and the eNodeB, and we are going to introduce them in the following content.

![epc](../../image/part1/epc.png)

- HSS (Home Subscriber Server)

    - Storing subscriber information, such as International Mobile Subscriber Identity (IMSI), Mobile Station International Subscriber Directory Number (MSISDN), Service Permissions, and roaming permissions.
    - Maintaining authentication keys and security parameters used during UE authentication.
    - Providing subscription and roaming information to the MME to support mobility and access control decisions.

- MME (Mobility Management Entity)

    - Handling UE attach and detach procedures.
    - Managing mobility, including tracking the UE’s serving eNodeB and coordinating handovers.
    - Authenticating subscribers through interaction with the HSS and determining whether network access is permitted.
    - Coordinating with the S-GW and P-GW during data bearer establishment to create the appropriate EPS bearers.

- S-GW (Serving Gateway)

    - Forwarding user-plane traffic between the eNodeB and the P-GW
    - Serving as a stable mobility anchor during inter-eNodeB or inter-area mobility, reducing the need for frequent path re-establishment.
    - Supporting partial charging and traffic accounting functions for downstream billing systems.

- P-GW (Packet Data Network Gateway)

    - Assigning IP addresses to UEs to enable communication with the Internet or enterprise private networks.
    - Enforcing policy and charging control in coordination with the PCRF, including bandwidth limitations, service blocking or allowance, and traffic shaping.
    - Usually for enterprise private network, IMS system and entry for every application server.

- eNodeB (Evolved Node B)

    - The eNodeB represents the 4G base station
    - Managing radio resources, including scheduling, modulation and coding, and power control.
    - Converting radio signals from the UE into IP packets and forwarding them to the S-GW via the S1-U interface.
    - Signaling with the MME over the S1-MME interface, including RRC connections, attach procedures, and tracking area updates (TAU).
    - Supporting the X2 interface between neighboring eNodeBs for handover coordination and load balancing.

### 1.2.2 Limitations of 2G/3G/4G Core Networks

Although 2G, 3G, and 4G core networks successfully support voice, messaging, and mobile broadband services, their architectures and functional capabilities exhibit several fundamental limitations. These constraints make it difficult for legacy core networks to meet the **high flexibility**, **high efficiency**, and **service diversity requirements** targeted by 5G.

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

## 1.3 Requirements and Design Principles of the 5G Core (5GC)

### 1.3.1 Requirements of the 5G Core Network

5G is not simply a faster version of 4G. Instead, it is designed to simultaneously support three fundamentally different service categories, which directly shape the architectural design of the 5G Core (5GC).

- **eMBB (enhanced Mobile Broadband)**

    The primary objective of eMBB is to deliver higher data rates and significantly increased capacity, enabling applications such as 4K/8K video streaming, AR/VR, and cloud gaming.

    From a core network perspective, this requires the ability to handle large volumes of high-bandwidth traffic while maintaining stable QoS (e.g., throughput, latency, and packet loss performance) under heavy load conditions.

- **URLLC (Ultra-Reliable Low Latency Communications)**

    Targets use cases such as autonomous driving, remote surgery, and industrial control systems, which demand “millisecond-level or even sub-millisecond latency” combined with extremely high reliability.
    
    To meet these requirements, the core network must minimize end-to-end data paths and support the placement of user plane functions closer to the network edge (MEC), while also providing fine-grained prioritization and enhanced protection mechanisms.

- **mMTC (massive Machine Type Communications)**

    mMTC addresses scenarios involving massive numbers of IoT devices, such as smart meters, sensors, and wearable devices. Although each device typically generates low traffic volumes, the “device density is extremely high.”
    
    The core network must therefore support high-density device registration, low-cost connectivity, and connection models optimized for long idle periods and intermittent data transmission (e.g., energy-efficient and power-saving mechanisms introduced in 5G).

Beyond these three service categories, **the rapid proliferation of diverse IoT and vertical industry services** has introduced new requirements for the 5G Core (5GC):

- Different enterprises / industries (e.g., manufacturing, autonomous vehicles, healthcare) exhibit highly diverse requirements in terms of security, isolation, latency, and reliability
- The traditional “one network serving all users” model is no longer sufficient; the network must support rapid creation of customized network slices and policies tailored to specific customers and applications
- The core network must integrate more seamlessly with IT / Cloud environments, enabling automated deployment, elastic scaling, and DevOps-oriented operational models

Together, these requirements drive the evolution of the 5G Core from traditional telecom appliance-based systems toward a cloud-native, service-based architecture.

### 1.3.2 Design Principles of the 5G Core Network

To meet the requirements described above, 3GPP defined several fundamental design principles when specifying the 5GC, which can be broadly summarized into three key directions:

- **Introduction of SBA (Service-Based Architecture)**

    The 5GC is no longer centered around a “single, monolithic network function.” Instead, core functionalities are decomposed into multiple network functions (NFs), such as the AMF, SMF, UPF, UDM, and PCF, which interact with each other through **HTTP + JSON / REST-style** interfaces.  
    
    Each NF exposes clearly defined services, and other NFs can discover and invoke these services by name. This design:

    - Enables independent development, upgrade, and scaling of network functions
    - Facilitates the adoption of microservices and containerization technologies (e.g., Kubernetes)    
    - Brings telecom network architecture closer to conventional IT and cloud system design paradigms

- **Support for CUPS (Control and User Plane Separation)**

    Although CUPS was already introduced in the 4G EPC, the separation between the control plane (e.g., AMF, SMF) and the user plane (UPF) is more thoroughly realized in the 5GC. 
    
    As a result:

    - The control plane can be centrally deployed, enabling unified management of control logic and policies 
    - User plane (UPF) can be flexibly distributed toward the network edge or across different geographic locations, shortening data paths and reducing latency  
    - Control plane and user plane resources can be independently scaled in or out based on traffic characteristics and service requirements

- **Cloud-Native Design and Network Slicing**

    From the outset, the 5GC is designed to operate in cloud environments, with support for containerization, microservices, and automated orchestration. 
    
    Building on this foundation, the 5GC introduces the concept of **Network Slicing**, whereby multiple end-to-end logical “virtual networks” are created on top of a shared physical infrastructure. Each network slice can have:

    - Different combinations and configurations of network functions
    - Distinct QoS policies, security levels, and management logic
    - Alignment with specific customers or industry scenarios (e.g., one slice dedicated to mMTC and another to URLLC-based industrial control) 

Through the adoption of SBA, CUPS, and cloud-native network slicing, the 5G core network represents a significant evolution from previous generations. It transitions from a “single-purpose, inflexible telecom core network” toward a **“programmable, sliceable, and highly customizable general-purpose digital infrastructure,”** allowing for further innovative applications.

## 1.4 Chapter Summary

This chapter begins by addressing the question of “what the core network is” and explaining its role in mobile communication systems. The core network functions as the “brain” of the network, responsible for subscriber authentication, resource allocation and management, traffic routing decisions, and support for charging and accounting. Then the chapter reviews the overall architecture of the 4G EPC and the responsibilities of its key network functions, including the HSS, MME, S-GW, P-GW, and eNodeB. It also highlights the major limitations of 2G/3G/4G core networks, such as closed architectures, incomplete separation of control and user planes, limited service diversity, and constraints in areas such as scalability and cloud-native capabilities.

Building on this background, the chapter summarizes the motivations and design principles behind the emergence of the 5G Core (5GC). To support diverse service scenarios such as **eMBB**, **URLLC**, and **mMTC**, as well as the differentiated requirements of vertical industries and enterprise private networks, the 5GC adopts a service-based architecture (SBA), a more thorough separation of CUPS, and cloud-native design with network slicing. These design choices enable the core network to evolve from a traditional, closed telecom system into a programmable and highly customizable digital infrastructure. In the following chapters, this historical context serves as the foundation for a more systematic introduction to the functions and architecture of the 5G core network from a 3GPP perspective.


<div class="chapter-nav">
  <a href="../chapter2/" class="nav-btn nav-next" title="Next: 5G Core Network Overview from 3GPP Perspective">
    <span class="arrow"></span>
  </a>
</div>
