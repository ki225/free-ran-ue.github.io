# Chapter 7: Non-3GPP Access

In the 5G core network system, in addition to the standard 3GPP access, the specifications also define **non-3GPP access** in order to integrate with existing and common network architectures. Simply put, this allows devices to **access the 5G core network via Wi-Fi**.

Within 3GPP, Wi-Fi access is further categorized into **trusted (TNGF)** and **untrusted (N3IWF)**, as shown in the figure below. In the following sections, we will introduce these two access methods and their respective functions. In practice, both approaches add an additional layer between the UE and the 5GC, consisting of a **Non-3GPP Access Network (AN, such as Wi-Fi APs plus aggregation switches or IP networks) combined with TNGF / N3IWF**. You can think of this combination as “another type of RAN from the perspective of the 5GC.”

![n3](../../image/part2/n3.png)

## 7.1 TNGF

TNGF (Trusted Non-3GPP Gateway Function), as its name suggests, an access gateway designed for **“trusted non-3GPP networks.”** These networks are typically deployed and managed by the same operator, or by parties that have a close cooperative relationship with the operator, for example:

- Operator-deployed Wi-Fi hotspots
- Private Wi-Fi networks jointly planned by enterprises and telecom operators

In such scenarios, the operator can trust the deployment and management of the Wi-Fi network itself. As a result, the security architecture can be relatively simplified, and the combination of “non-3GPP AN + TNGF” can be treated as an extension of the 3GPP RAN, integrating non-3GPP access into the 5GC through the TNGF.

From a functional perspective, the TNGF is mainly responsible for:

- Establishing and maintaining user-plane and control-plane channels between the UE and the 5GC over non-3GPP networks
- Coordinate UE registration as well as PDU session establishment and management with the 5GC (especially the AMF / SMF)
- Cooperating with Wi-Fi-side AAA or authentication systems when required to complete UE access authentication

Under the assumption of a “trusted,” the security design around the TNGF typically focuses on several key aspects:

- **Access-layer protection**: The non-3GPP AN should still employ enterprise-grade Wi-Fi security mechanisms, such as 802.1X or WPA2/WPA3-Enterprise, and integrate with the operator’s AAA or authentication systems to prevent arbitrary devices from directly accessing the access network.
- **Network boundary protection**: The area where the TNGF is deployed is usually segmented from other IT networks using dedicated subnets and firewalls, preventing the non-3GPP AN from being treated as a general internal network.
- **Trust relationship with the 5GC**: Connections between the TNGF and the 5GC over SBI, N2, and N3 must still be secured with certificates and TLS or IPsec according to standard NF security requirements. This ensures that what is trusted is a controlled access network, rather than weakening the protection of the core network boundary.

> [!Tip]
> In short, when a Wi-Fi network can be regarded by the operator as “an extension of its own infrastructure,” the TNGF approach is typically used.

## 7.2 N3IWF

N3IWF (Non-3GPP InterWorking Function) is designed to support **“untrusted non-3GPP networks,”** with the most typical examples being **public Wi-Fi** and user-provided networks (such as home routers or coffee shop hotspots). In these cases, the 5GC cannot trust the Wi-Fi network itself, and therefore must:

- Treat the UE as accessing the network from “an untrusted IP network”
- Establish an encrypted tunnel between the UE and the N3IWF using **IPsec / IKEv2**
- Use this secure tunnel to carry NAS signaling and user-plane traffic into the 5GC

The overall process can be summarized as follows:

1. The UE first connects to a Wi-Fi or wired IP network and obtains an IP address (It is no different from ordinary Internet access).
2. The UE initiates IKEv2 / IPsec handshake with the N3IWF to establish an encrypted tunnel.
3. Within this IPsec tunnel, NAS messages begin to be exchanged between the UE and the AMF, which is equivalent to “remotely accessing the 5GC through a non-3GPP network.”
4. Subsequent registration procedures, PDU session establishment, and user-plane packets are all encapsulated and forwarded within this tunnel.

In the free5GC project, there is a corresponding N3IWF module and implementation, which is often used to demonstrate scenarios “where a UE connects to the 5GC via Wi-Fi / Internet VPN.” From the 5GC’s perspective, the combination of “non-3GPP AN + N3IWF” can be abstracted as another type of RAN, albeit a “virtual RAN” built on top of IPsec / VPN.

The security of the N3IWF relies on a “two-layer protection model”:

- **Outer IPsec / IKEv2 layer**: An encrypted tunnel is established between the UE and the N3IWF using certificates and keys. This ensures that even if the Wi-Fi or intermediate IP networks are completely untrusted, attackers cannot directly eavesdrop on or tamper with 5G signaling or user-plane traffic.
- **Inner 5G-AKA or EAP-AKA′ mechanisms**: Within the IPsec tunnel, the UE must still authenticate itself to the AUSF / UDM using authentication procedures in 5GC. This layer determines “who the user is and whether the user is authorized to access the network.”
- During implementation and experimentation, self-signed certificates and simplified configurations may be used for convenience. However, in environments closer to real deployments, it is strongly recommended to use a proper PKI and to strictly manage N3IWF certificates and keys to prevent the N3IWF itself from becoming an attack entry point.

> [!Tip]
> If you are familiar with traditional VPN architectures, you can think of the N3IWF as “a VPN gateway specifically designed for 5G UEs.” The difference is that the traffic carried inside the tunnel is not ordinary enterprise data, but 5G NAS signaling and user-plane traffic.

## 7.3 ATSSS

![atsss](../../image/part2/atsss.png)

ATSSS (Access Traffic Steering, Switching, and Splitting) is a mechanism introduced in 5G to support **“intelligent traffic control across multiple access paths.”** In simple terms, it allows a single UE to simultaneously use both 3GPP access (such as 5G NR) and non-3GPP access (such as Wi-Fi or N3IWF). The core network and the UE jointly decide:

- Which path a particular flow should use (Steering)
- Whether traffic should switch to another path when the quality of one path degrades (Switching)
- Or whether traffic should be split across multiple paths and transmitted in parallel (Splitting)

From an architectural perspective, ATSSS involves:

- Multi-path support and traffic classification on the UE side (usually requires specific UE stack capabilities)
- Joint path and policy decisions in the 5GC by the SMF / UPF / PCF (for example selecting 3GPP access, non-3GPP access, or both, based on application type, QoS requirements, and current network conditions)

In community experiments over the past few years, free5GC has also demonstrated “**dual-connectivity ATSSS examples combining N3IWF and 3GPP access**”:

In these scenarios, the UE connects to a gNB via 5G NR on one side, and establishes an IPsec tunnel with Wi-Fi and the N3IWF on the other. Both paths eventually converge at the same 5GC / UPF, then the core network directs some traffic over 3GPP access and other traffic over non-3GPP access, or splits traffic between the two according to policy. This combination provides an excellent experimental platform for experiencing how “multi-access and multi-path” operation works in the 5G core network.

## 7.4 Chapter Summary

This chapter briefly introduced “non-3GPP access” and multi-access traffic control mechanisms supported by the 5G core network: TNGF and N3IWF correspond to trusted and untrusted Wi-Fi / IP network access, respectively, while ATSSS further enables traffic steering, switching, and splitting between 3GPP and non-3GPP access. Understanding these mechanisms helps, when designing or experimenting with 5G networks, to consider “what additional paths beyond the 3GPP RAN can be used to securely and flexibly connect devices to the 5GC,” as well as how multiple paths can be leveraged to improve user experience and reliability.

<div class="chapter-nav">
  <a href="../../part3-deploy/chapter8/" class="nav-btn nav-next" title=Next: free5GC Deployment and Configuration">
    <span class="arrow"></span>
  </a>
</div>
