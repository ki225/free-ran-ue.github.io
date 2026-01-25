# Chapter 12: How the Simulator Integrates with free5GC

> [!Note]
> This chapter assumes that the reader already knows how to start the free5GC core network. Readers who are not familiar with the core network can refer to [Chapter 8](../part3-deploy/chapter8).

## 12.1 UE Data in free5GC

Before starting the simulator, we must first add a subscriber record in the free5GC [Web Console](https://github.com/free5gc/webconsole). This is equivalent to declaring a new user equipment (UE) to the core network. Detailed steps can be found at: [Create Subscriber via Webconsole](https://free5gc.org/guide/Webconsole/Create-Subscriber-via-webconsole/).

Conceptually, adding a subscriber is similar to registering “a SIM card or user” with a mobile network operator: the operator records the subscriber’s identity, authentication credentials, and allowed service profiles. In free5GC, this subscriber record is the basis for determining “whether a UE exists, whether it is allowed to register, and which PDU Sessions it is permitted to establish.” If the subscriber is not created in advance, subsequent UE registration or authentication procedures will typically fail.

In practice, you must ensure that the parameters used by the simulator (UE side) are consistent with the subscriber data created in the free5GC Web Console, including:

- **IMSI / SUPI**: The unique subscriber identifier (carried by the UE during registration)
- **Authentication data (K, OP / OPc, etc.)**: Used for the AKA authentication procedure
- **Slices and services (S-NSSAI, DNN, allowed PDU Session settings)**: Define which networks / data services the UE is allowed to access

Only after the subscriber has been created do we proceed to the integration steps, where free-ran-ue is connected to free5GC with the correct configuration.

## 12.2 Integration Flow

The next section provides more detailed instructions for each startup mode. However, the “overall” integration flow can be summarized as follows:

1. **Start the core network** and verify that all NFs are running normally
2. **Add subscriber data via the Web Console** (UE subscription)
3. **Start the gNB simulator** and confirm that it is operating correctly, including:

    - Successful connectivity to the AMF (control plane) / UPF (user plane)
    - Control-plane and user-plane ports for UE access are open and listening

4. **Start the UE**, connect it to the gNB, and complete the UE registration and PDU Session establishment procedures

## 12.3 Startup Methods

free-ran-ue currently supports the following three startup methods:

1. Standalone host execution

    - **Advantages**

        - **Most intuitive and easiest to debug**: Running directly on the host provides clear visibility into network namespaces, interfaces, and routing.
        - **Best performance**: With one fewer layer of virtualization / isolation, performance is typically closer to bare metal.
        - **Ideal for quick validation**: Convenient for single-host PoCs, functional checks, and log inspection.

    - **Disadvantages**

        - **Weaker isolation**: Different test scenarios / experiments may interfere with each other (routing, interfaces, ports, residual state).
        - **Lower environment consistency**: Differences in system dependencies across users / machines can lead to “it works on my machine” issues.

    - **Best suited for**

        - Developers who want to get the flow working quickly and frequently modify code / inspect logs.

    > [!Note]
    > For standalone host startup instructions, refer to the [official documentation](https://free-ran-ue.github.io/doc-user-guide/02-free-ran-ue/).

2. Namespace-based startup

    - **Advantages**

        - **Good isolation while remaining close to the host network**: Different UEs / gNBs / scenarios can be placed in separate namespaces to reduce interference.
        - **Well suited for multi-scenario / multi-UE testing**: Enables more natural construction of complex topologies and network isolation on a single machine.
        - **Still relatively debug-friendly**: Fewer abstraction layers than containers, making network issues easier to trace.

    - **Disadvantages**

        - **Higher operational complexity**: Requires basic knowledge of Linux namespaces, veth, routing, and privileges.
        - **Cleanup overhead**: Interrupted tests require careful cleanup of namespaces / interfaces to avoid residual effects on subsequent runs.

    - **Best suited for**

        - Users who want more realistic network isolation for integration testing on a single host, or who need to run multiple experiments concurrently.

    > [!Note]
    > For namespace-based startup instructions, refer to the [official documentation](https://free-ran-ue.github.io/doc-user-guide/03-quickstart-free-ran-ue/).

3. Docker container–based virtualization

    - **Advantages**

        - **Best reproducibility**: Dependencies and versions are encapsulated in containers, making this ideal for team collaboration and CI automation.
        - **Easy deployment / portability**: The same container configuration can be started quickly on different machines.
        - **Straightforward integration with test stacks**: Can be composed together with free5GC, user-plane tools, and monitoring components.

    - **Disadvantages**

        - **More complex network debugging**: NAT, bridges, and port mappings increase troubleshooting complexity.
        - **Performance and privilege requirements**: Certain network and interface operations may require `--privileged` or additional capabilities. For free5GC specifically, the UPF relies on the kernel gtp5g module for packet forwarding, which must be handled with care.

    - **Best suited for**

        - Users who need reproducible integration tests, CI pipelines, or rapid deployment across multiple environments.

    > [!Note]
    > For Docker-based startup instructions, refer to the [official documentation](https://free-ran-ue.github.io/doc-user-guide/10-docker/).

> [!Tip]
> Quick selection guide (lazy decision mode)
>
> - **Developing / debugging and want logs immediately**: Choose **Standalone host**
> - **Running multiple scenarios on one machine with clearer network isolation**: Choose **Namespace**
> - **Need reproducibility, team sharing, or CI/automation**: Choose **Docker**

## 12.4 Configuration Files

The core concept of the configuration is simple: **ensure that free5GC (the core network) and the gNB / UE (the simulator) reside on mutually reachable networks, with consistent IP / port settings**.

When deploying free5GC in Chapter 8, you configured the service addresses in the AMF / SMF / UPF configuration files. Similarly, the gNB configuration file in free-ran-ue must specify the corresponding AMF / UPF addresses to successfully establish control-plane and user-plane connectivity.

1. Network and address planning (recommended approach)

    Using the free-ran-ue [gNB configuration](https://github.com/free-ran-ue/free-ran-ue/blob/main/config/gnb.yaml) and [UE configuration](https://github.com/free-ran-ue/free-ran-ue/blob/main/config/ue.yaml) as examples, the overall topology can be understood in two segments:

    - **Core network ↔ gNB (N2 / N3)**: `10.0.1.0/24`

        - Core network (AMF / UPF): `10.0.1.1`
        - gNB (RAN): `10.0.1.2`

    - **UE ↔ gNB (local access segment)**: `10.0.2.0/24`

        - gNB access address exposed to UEs: `10.0.2.1`
        - The UE establishes control-plane / user-plane connectivity via this address

    > [!Note]
    > Key point: In standalone host mode, “network isolation is not provided automatically.” You must ensure that these two subnets are correctly routed / bound on the host (e.g., via appropriate interfaces or routing rules). Otherwise, even with correct configuration files, connections will fail due to network reachability issues.

2. Key fields in the gNB configuration

    The following fields are most commonly required to be aligned (other fields such as `plmnId` / `tai` / `snssai` must also match UE and subscriber settings):

    - **Core network connectivity (N2 / N3)**

        - `amfN2Ip`: AMF N2 IP (e.g., `10.0.1.1`)
        - `ranN2Ip`: gNB N2 IP used to connect to the AMF (e.g., `10.0.1.2`)
        - `upfN3Ip`: UPF N3 IP (e.g., `10.0.1.1`)
        - `ranN3Ip`: gNB N3 IP used to connect to the UPF (e.g., `10.0.1.2`)
        - `amfN2Port`: AMF N2 port (commonly `38412`)
        - `upfN3Port`: UPF N3 GTP-U port (commonly `2152`)

    - **UE access (RAN-facing interfaces)**

        - `ranControlPlaneIp` / `ranControlPlanePort`: Control-plane address/port for UE access (e.g., `10.0.2.1:31413`)
        - `ranDataPlaneIp` / `ranDataPlanePort`: Data-plane address/port for UE access (e.g., `10.0.2.1:31414`)

    - **Console API (optional)**

        - `api.ip` / `api.port`: API address/port for the Web Console to pull status (e.g., `10.0.1.2:40104`)

3. Key fields in the UE configuration (`ue` section)

    The UE configuration focuses on “which gNB to connect to” and “who the UE is”:

    - **gNB access address**

        - `ranControlPlaneIp` / `ranControlPlanePort`: Must match the gNB’s `ranControlPlaneIp/Port` (e.g., `10.0.2.1:31413`)
        - `ranDataPlaneIp` / `ranDataPlanePort`: Must match the gNB’s `ranDataPlaneIp/Port` (e.g., `10.0.2.1:31414`)

    - **Identity and authentication (must match free5GC subscriber data)**

        - `plmnId (mcc/mnc)` + `msin`: Together form the UE identity (one of the sources of IMSI / SUPI)
        - `authenticationSubscription`: Authentication parameters (e.g., `encPermanentKey`, `encOpcKey`, `sequenceNumber`)

    - **User-plane services**

        - `pduSession.dnn`: Target DNN for connection (e.g., `internet`)
        - `pduSession.snssai`: Target slice (must match subscriber configuration)
        - `ueTunnelDevice`: Prefix for the UE local TUN interface name (e.g., `ueTun`, commonly resulting in `ueTun0`)

4. Startup order and verification

    It is recommended to start the components in the following order and validate them step by step (only proceed after each step passes):

    1. **Start free5GC**, ensuring that AMF, UPF, and other NFs are running normally, and that the AMF (N2) and UPF (N3) addresses are reachable from the gNB
    2. **Create the subscriber in the Web Console**, ensuring IMSI / authentication data / S-NSSAI / DNN match the UE configuration
    3. **Start the gNB**

        - Logs should show successful NGAP establishment with the AMF
        - If the user plane is enabled, logs should indicate successful user-plane connectivity / tunnel setup with the UPF

    4. **Start the UE**

        - Logs should show successful UE registration and PDU Session establishment
        - The UE TUN interface (e.g., `ueTun0`) should appear on the host

    > [!Caution]
    > If the process stalls: first check “IP reachability” and “IP / port alignment” in the configuration files. Only then verify authentication data and slice / DNN consistency.

## 12.5 Chapter Summary

This chapter emphasizes that, when integrating free-ran-ue with free5GC, the most critical factor is not “which commands to run,” but rather having a clear understanding of the dependency chain: the core network must contain the appropriate subscriber data before the gNB can establish control-plane connectivity and prepare the user-plane path. Only then can the UE successfully complete registration and PDU Session establishment, ultimately creating a local data channel such as `ueTun0`.

In practice, most issues can be diagnosed quickly by following two principles:

- **Connectivity first, authentication second, user plane last**: First verify IP / port reachability and interface connectivity, then validate IMSI / authentication data, and slice / DNN consistency, and finally inspect PDU Session establishment and that user-plane packets can flow end to end.
- **Configuration consistency**: PLMN, S-NSSAI, and DNN values in the gNB / UE configuration files must match the subscriber settings in the free5GC Web Console; otherwise, procedures will fail during registration or session establishment.

This chapter also compared three startup methods (standalone host, namespace-based, and Docker-based). You can select the appropriate deployment approach based on “debugging convenience, isolation requirements, reproducibility, and CI integration needs,” and progressively engineer the integration workflow into a reusable test environment.
