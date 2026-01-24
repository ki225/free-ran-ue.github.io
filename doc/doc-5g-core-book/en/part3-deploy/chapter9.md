# Chapter 9: Multi-Node / Multi-UPF Deployment

In the previous chapter, we completed a basic free5GC deployment in which “all NFs run on a single machine.” This single-node architecture is well suited for teaching and small-scale experiments. However, in real-world network design, one of the key characteristics of 5G is **its support for Control and User Plane Separation (CUPS), as well as the flexibility to deploy multiple UPFs—or even hierarchical UPF architectures**. This chapter starts from the conceptual level, introducing multi-node / multi-UPF architectures and the role of the ULCL (Uplink Classifier). We then briefly walk through how such deployments can be implemented in free5GC.

## 9.1 Why Do We Need Multi-UPF Deployments?

In the 5GC design, control-plane NFs (such as AMF, SMF, and PCF) and user-plane NF (UPF) are deliberately separated. This allows operators to flexibly determine the number and placement of UPFs based on different requirements and scenarios. For example:

- **Local traffic diversion**: Deploy one or more UPFs in different geographic regions so that UE traffic can “exit locally,” reducing backhaul latency.
- **Edge computing (MEC)**: Place UPFs at edge sites to steer selected traffic directly to edge application servers instead of routing everything back to the central data center.
- **Multi-tenant or service separation**: Assign different UPFs to different enterprise private networks or service types (such as video or IoT), and combine them with slicing and policy control for finer-grained traffic management.

To support these scenarios, 5G introduces “multi-node and multi-layer UPF” concept, where one of the key components is the **ULCL (Uplink Classifier)**.

## 9.2 What Is ULCL (Uplink Classifier)?

![ulcl](../../image/part3/ulcl.png)

ULCL can be viewed as a “UPF node with traffic diversion capabilities.” Its primary role is to **forward traffic to different downstream UPFs or data networks based on specific rules in uplink traffic (UE → network)**. Functionally, it is still a UPF, but in terms of topology placement and rule design, it acts as an “aggregation point and traffic splitter.”

From a 3GPP perspective, ULCL has the following characteristics:

- **Located upstream, close to the RAN or access network**: It is typically placed downstream of the gNB or N3IWF, serving as the first UPF that uplink traffic encounters when entering the 5GC.
- **Performs classification based on 5-tuple, DNN, S-NSSAI, or other fields**: Using PDRs (Packet Detection Rules) and related rules, ULCL determines which downstream UPF a packet should be forwarded to.
- **Can connect to multiple downstream UPFs simultaneously**: For example, one path may lead to “the normal Internet,” while another leads to an “enterprise private network” or an “edge application.”

Using an everyday analogy, ULCL is like “a highway merge area”: all vehicles (uplink packets) arrive at the merge area first, and are then directed to different exits based on their destination and type.

In practical free5GC deployments, ULCL is commonly used to:

- Steer selected application traffic to a local MEC UPF, while the remaining traffic goes through a central UPF
- Steer traffic to different enterprise private network UPFs based on DNNs / network slices

These architectures will be illustrated with diagrams and configuration examples in the following multi-node deployment sections.

## 9.3 Multi-Node / Multi-UPF Deployment in free5GC

In this section, we directly follow the official ULCL deployment as the basis for our introduction. The core idea is to configure the SMF with user-plane information that enables the activation of multiple UPF instances.

### 9.3.1 Changes to the SMF Configuration

In an ULCL-based SMF configuration, the most critical section is `userplaneInformation`. This section defines which nodes (`upNodes`) exist in the user-plane topology and how these nodes are interconnected (`links`). A simplified example is shown below, where three `upNodes` are defined:

```yaml
userplaneInformation:
  upNodes:
    gNB1:
      ...
    I-UPF:
      ...
    PSA-UPF:
      ...
```

Here:

- `gNB1` represents an access node (the gNB connected via N3)
- `I-UPF` is the upstream diversion node (the ULCL)
- `PSA-UPF` is the downstream anchor UPF (PDU Session Anchor)

With this declaration, the SMF learns “which user-plane nodes are available” and how to select paths when establishing PDU sessions.

Next, the `links` section in `userplaneInformation` describes connectivity between nodes using `A` / `B`:

```yaml
userplaneInformation:
  upNodes:
   ...
  links:
    - A: gNB1
      B: I-UPF
    - A: I-UPF
      B: PSA-UPF
```

This indicates that:

- `gNB1` connects to `I-UPF` over the N3 interface
- `I-UPF` connects downstream to `PSA-UPF`

After the SMF starts, it builds a “user-plane topology graph” based on the `upNodes` and `links` information. This topology is then used as the basis for delivering PFCP rules when handling PDU sessions and ULCL traffic steering.

> [!Caution]
> The node names used in `links` **must exactly** match the names declared in `upNodes` (including case sensitivity). Otherwise, the SMF will fail to resolve the nodes, resulting in PDU session establishment failures or incorrect traffic forwarding.

### 9.3.2 UE Path Configuration

During ULCL operation, predefined rules can be used to specify which UE traffic should be steered at the I-UPF without passing by the PSA-UPF. This is configured using `uerouting.yaml`, as shown below:

```yaml
info:
  version: 1.0.7
  description: Routing information for UE

ueRoutingInfo:
  UE1:
    members:
      - imsi-208930000000001
    topology:
      - A: gNB1
        B: I-UPF
      - A: I-UPF
        B: PSA-UPF
    specificPath:
      - dest: 1.0.0.1/32
        path: [I-UPF]
```

This configuration file performs several functions:

- Under `ueRoutingInfo`, the logical name `UE1` represents a group of UEs. The `members` field lists the actual UE identities (in this case, there is a single IMSI `imsi-208930000000001`). If additional UEs need to share the same routing and traffic-steering rules, their IMSIs can simply be added to this list.
- `topology` defines the “default user-plane topology” for this UE group—that is, the path from gNB1 through I-UPF to PSA-UPF. This corresponds to the connectivity declared earlier in the SMF `userplaneInformation.links` configuration.
- `specificPath` is the key ULCL feature: It specifies that when the destination address of this UE’s traffic is `1.0.0.1/32`, the data path traverses only `I-UPF`. As a result, the traffic “exits” at the I-UPF and is not forwarded to the PSA-UPF. In other words, this destination-specific traffic is therefore steered at the I-UPF (for example, toward a local MEC or a specific service), while all other traffic continues to follow the default topology and is forwarded through the PSA-UPF.

After the SMF starts and reads `uerouting.yaml`, it installs the corresponding PDR / FAR rules onto the ULCL-UPF based on these settings. These rules allow the I-UPF to distinguish “which packets should follow the default path toward the PSA-UPF and which packets can be locally steered and handled at the ULCL layer.”

## 9.4 Chapter Summary

Starting from the CUPS architecture of the 5GC, this chapter explained why multi-node and multi-UPF deployments are commonly required in real-world scenarios. Using ULCL as the entry point, we introduced the role of uplink traffic diversion in the 5G core network and its typical application scenarios. We then analyzed the official free5GC ULCL example, showing how `userplaneInformation.upNodes` and `links` in the SMF configuration describe the user-plane topology. We also examined how `uerouting.yaml` defines, for specific UEs and destination addresses, which traffic should be steered directly at the I-UPF and which should continue to be forwarded through the PSA-UPF.

With these concepts, you should now be able to understand the key configuration elements and path-design logic in multi-UPF / ULCL-based architectures, and recognize that “traffic-steering decisions are ultimately enforced by the SMF through PFCP UL/DL rules installed on each UPF.” When designing more complex topologies in the future (such as multi-layer UPF deployments, MEC scenarios, enterprise private networks, or multi-slice environments), keeping the design principles and configuration approaches introduced in this chapter in mind will allow you to progressively implement and validate various user-plane path designs using free5GC.

<div class="chapter-nav">
  <a href="../../part4-free-ran-ue/chapter10/" class="nav-btn nav-next" title="Next: Why Do We Need a RAN/UE Simulator?">
    <span class="arrow"></span>
  </a>
</div>
