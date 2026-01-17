# Chapter 6: Other Network Functions

![sbi](../../image/part2/sbi.png)

In Chapters 4 and 5, we explored the three most fundamental 5G core network components: AMF / SMF / UPF. In this chapter, we will move on to introduce other network functions that are relatively simpler in functionality but still play important roles. This book does not aim to exhaustively describe every function within each network component. Instead, by focusing on their primary responsibilities, we help readers understand the roles these components play within the overall 5G system.

## 6.1 AUSf

AUSF (Authentication Server Function) is the core network function in the 5GC responsible for **user authentication**. When a UE initiates the registration procedure, the AMF forwards relevant identity information such as the SUCI and SUPI to the AUSF. The AUSF then works together with the UDM to complete authentication procedures such as 5G-AKA or EAP-AKA′, based on the user’s authentication vectors and key material.

In the free5GC implementation, the AUSF is mainly responsible for:

- Receiving authentication requests from the AMF and selecting an appropriate authentication method
- Querying the UDM and generating authentication vectors (AVs)
- Verifying whether the authentication response returned by the UE is valid and generating the security context information required for subsequent procedures

You can think of the AUSF as a backend service “dedicated to verifying identities and credentials,” while the AMF acts as the front desk that brings the UE in for verification.

> [!Note]
> 5G-AKA (Authentication and Key Agreement) is a mechanism used to authenticate the UE and to derive and distribute shared keys between the UE and the 5G operator.

## 6.2 CHF

CHF (Charging Function) is responsible for **charging logic and usage accounting**. It typically works together with network functions such as the SMF and UPF in the 5GC to record usage and events for different services and slices. In many implementations, the CHF integrates with existing billing systems or databases and provides interfaces for both online charging and offline charging.

In the free5GC architecture, the CHF is relatively simplified and is mainly used to:

- Receive usage information reported by the SMF / UPF via URRs
- Record and export usage data on an event basis, making it easier to perform further analysis or integrate with external systems

Although this book does not dive deeply into the implementation of every charging scenario, understanding the role of the CHF helps clarify where usage statistics ultimately go when you study PDU sessions and URR configurations.

## 6.3 NEF

NEF (Network Exposure Function) is a network function that **“exposes 5G core capabilities as APIs to external parties.”** Its main goal is to allow third-party applications or enterprise systems to access certain 5GC capabilities in a secure and controlled manner, such as:

- Querying or subscribing to specific events (for example changes in UE status or location)
- Adjusting certain policies or service behaviors through controlled interfaces

In free5GC, the NEF implementation is more simplified compared to commercial systems, but it provides a valuable framework for understanding how 5G transforms a “traditional telecom network” into “a set of services that can be invoked by applications.” For readers interested in integrating the 5G core with their own systems, the NEF serves as an important conceptual entry point.

## 6.4 NRF

NRF (Network Repository Function) is the **“service registration and directory service”** of the 5GC Service-Based Architecture (SBA). All network functions (including AMF, SMF, UPF, PCF, and AUSF) register their service types, addresses, and statuses with the NRF at startup. When a NF needs to invoke another NF, it first queries the NRF to discover available service instances.

In free5GC, the NRF mainly handles:

- Receiving and maintaining registration information for all network functions
- Providing service discovery capabilities so that network functions can communicate using service names rather than hard-coded IP addresses

If you think of the 5GC as a microservices system, the NRF plays a role similar to a “service registry combined with a lightweight service directory,” and it is one of the key components that enables the SBA to function smoothly.

> [!Note]
> For example, when the AMF initiates a PDU session establishment request to the SMF, the high-level flow is as follows:
>
> 1. The SMF registers its address and services with the NRF during startup
> 2. The NRF receives the SMF registration information
> 3. The AMF receives a PDU session establishment request from the UE
> 4. The AMF queries the NRF for the SMF address and a JWT authentication token
> 5. The AMF uses the obtained address and JWT to formally send the request to the SMF

## 6.5 NSSF

NSSF (Network Slice Selection Function) is responsible for **assisting in the selection of appropriate network slices**. When a UE registers or establishes a PDU session, the AMF and SMF determine which slice the UE should be associated with based on subscription information, service requirements, and network configuration. During this process, the NSSF provides recommendations and decision support.

In the free5GC implementation, the NSSF logic is also relatively simplified, but the overall concept is preserved:

- Determining which slices are available based on the UE’s requested S-NSSAI and subscription data
- Assisting the AMF and SMF in slice selection and corresponding network function routing

For readers dealing with multi-tenant or enterprise private network scenarios, understanding the role of the NSSF helps in designing architectures where “a single 5GC instance can serve multiple customers and service types.”

> [!Note]
> S-NSSAI (Single Network Slice Selection Assistance Information) logically consists of two fields:
>
> 1. **ST (Slice / Service Type)**: Indicates “the general category of the slice,” such as eMBB, URLLC, or mMTC. In 3GPP specifications, this is typically represented by an 8-bit value (for example 1 for eMBB, 2 for URLLC, and 3 for mMTC). Operators may also define additional values for specific service categories.
> 2. **SD / SSD (Slice Differentiator)**: Used to further distinguish different slice instances under “the same type of ST.” The standardized 3GPP term is SD (24 bits), although some implementations or documents refer to it as SSD. In this book, SSD is used consistently when describing the concept. Through the SSD, operators can create multiple logically isolated slices under the same ST, such as multiple enterprise private networks.

## 6.6 PCF

PCF (Policy Control Function) is the **“policy decision hub”** of the 5GC, responsible for managing and providing policies related to QoS, access control, routing selection, and charging. When the SMF, AMF, or other network functions need to decide “how much bandwidth a flow should receive, whether access to a specific DNN should be allowed, or which charging scheme should be applied,” they typically request policy decisions from the PCF via the SBI.

In free5GC, the PCF:

- Stores a relatively simplified set of policies and QoS configurations (such as default QoS profiles for specific DNNs or slices)
- Responds to queries from the SMF / AMF by returning the corresponding policy results (such as QoS profiles or policy rules)

You can think of the PCF as “the rule database and decision service of the entire 5GC.” Many of the PDR / FAR / QER / URR rules introduced in Chapter 5 are defined or influenced here and are then translated by the SMF into rules that can be directly executed by the UPF.

## 6.7 UDM

UDM (Unified Data Management) is the primary **management center for user subscription data and certain state information** in the 5GC. It roughly inherits the role of the HSS from the 4G era but is redesigned in 5G to better align with the SBA and cloud-native architectures.

The main responsibilities of the UDM include:

- Storing and providing basic user information (such as SUPI, service entitlements, and roaming permissions)
- Working with the AUSF to provide keys and authentication vector information required for authentication
- Providing subscription-related query services to network functions such as the AMF and SMF (for example available slices and allowed DNNs)

In free5GC, the UDM is often used together with a database (such as MongoDB). You will interact with it frequently during deployment and testing, especially when modifying user data or experimenting with different policy combinations.

## 6.8 UDR

UDR (Unified Data Repository) can be viewed as a **general-purpose data storage layer within the 5GC**, used to store various types of data managed by different NFs, such as subscription data, policy information, or application-related data. NFs like the UDM and PCF can persist their data in the UDR and access it through standardized interfaces.

In some implementations, the UDM is considered “a logical user data management service,” while the actual data resides in the UDR. Similarly, the PCF can use the UDR to store policy configurations. For readers, it is sufficient to remember that the UDR primarily addresses “how data is centrally stored and accessed,” rather than directly participating in signaling procedures.

> [!Caution]
> It is easy to confuse the UDM and the UDR. A simple way to distinguish them is:
>
> - **UDM is “a service that thinks”**: it provides APIs and implements 5GC logic related to user data and authentication (such as who is allowed to use the network, how authentication is performed, and which slices / DNNs are permitted).
> - **UDR is “a database that stores data”**: it focuses on unified storage for data from various network functions (including data used by the UDM) and does not itself handle 5GC signaling procedures or decision logic.

## 6.9 Web Console

According to 3GPP specifications, a 5G system should provide a unified OAM (Operations, Administration, and Maintenance) interface that allows operators to manage user data, monitor network status, and collect various statistics and charging information. free5GC does not implement a complete OAM platform that complies with all standards, but it provides a practical **Web Console** as a lightweight OAM solution.

In the free5GC implementation, this Web Console mainly offers two categories of functionality:

- **Data management and configuration**: Directly operating on the backend MongoDB through a frontend interface, including:

  - Adding, deleting, or modifying UE (user) data and related subscription settings
  - Managing static configurations related to user such as slices (S-NSSAI), DNNs, and policies

- **Status monitoring and report export**: While the core network is running, the Web Console and its backend components can:

  - Collect and export UE usage and charging-related reports by establishing FTP or other types of connections with the CHF, enabling further analysis or integration with external systems.

For readers who are learning or experimenting with free5GC, the key point of the Web Console lies in providing an intuitive graphical interface that makes it easier to manage user and policy data in MongoDB and to observe the statistics and charging information generated during actual system operation.

> [!Note]
> The fields displayed in the Web Console and their meanings will be explained in detail in [Chapter 11](../part4-free-ran-ue/chapter11).

## 6.10 Chapter Summary

This chapter briefly introduced AUSF, CHF, NEF, NRF, NSSF, PCF, UDM, UDR, and the free5GC Web Console. Although these components do not directly participate in every signaling exchange or data path in the same way as the AMF / SMF / UPF, they each play critical roles in areas such as authentication, security, policy, charging, slice selection, and data management. Understanding their responsibilities and interactions will help you move beyond focusing only on the “main NFs” and instead view the entire 5GC as a complete and extensible system when reading code or designing experimental topologies.

<div class="chapter-nav">
  <a href="../chapter7/" class="nav-btn nav-next" title="Next: Non-3GPP Access">
    <span class="arrow"></span>
  </a>
</div>
