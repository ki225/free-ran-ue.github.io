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

<div class="chapter-nav">
  <a href="../chapter2/" class="nav-btn nav-next" title="Next：5G Core Network Overview from 3GPP Perspective">
    <span class="arrow"></span>
  </a>
</div>
