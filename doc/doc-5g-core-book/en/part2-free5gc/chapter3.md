# Chapter 3: free5GC Overall Architecture and Module Introduction

## 3.1 What Is free5GC?

free5GC is an open-source 5G Core project primarily aimed at **education, research, and Proof of Concept (PoC)** use cases. It implements both the control plane and user plane functions of the 5G Core network, mainly based on 3GPP Release 15 (now upgraded to Release 17), enabling developers to quickly build a functional 5G core network testbed on general-purpose servers, virtual machines, or containerized environments.

Compared to commercial 5G core network products, free5GC places greater emphasis on **readability, modifiability, and ease of deployment**. As a result, it is particularly suitable for:

- Academic or enterprise-internal 5G research and teaching
- Validating new protocols, algorithms, or architectural concepts
- Conducting end-to-end (E2E) experiments in combination with open-source RAN and UE

On the other hand, free5GC does not aim to fully cover all 3GPP features, nor does it provide carrier-grade HA or scalability designs. Therefore, it is **not recommended for direct use in commercial production environments**. In the following chapters, we will approach free5GC more from the perspective of “understanding 5GC architecture and programming,” rather than treating it as a black box to operate.

Among the current open-source 5GC projects, free5GC is not the only option—there are several other open-source 5G core network projects as well. In comparison, free5GC:

- Is developed in Go, a language that is relatively easy to learn and offers strong concurrency support
- Fully adheres to the 3GPP Service-Based Architecture (SBA), with a clear, modular, and extensible implementation
- Provides documentation and example environments that are more teaching-oriented, making it easier for readers to follow step by step

The focus of this chapter is to first build a “map” of free5GC for the reader, including its overall architecture, source code repositories, and NF program structure concepts, so that you will not get lost when diving into deployment details and individual NF implementations later.

> [!Note]
> The official free5GC website: [free5gc.org](https://free5gc.org)

## 3.2 Architecture Overview of free5GC

In practice, free5GC is typically used together with:

- One or more UE instances (either real UEs or simulated UEs)
- A 5G RAN setup (either a real gNB or a simulator)
- A server running the free5GC control plane and user plane services

From a high-level perspective, free5GC mainly implements the **5G Core Network (5GC)**. Internally, it is split into multiple Network Functions (NFs) according to 3GPP definitions, and these NFs communicate with each other via the SBA architecture. All NFs can be deployed on a single machine, or distributed across multiple nodes depending on experimental needs.

In terms of deployment models, common free5GC usage patterns include:

- **Single-node deployment**: All NFs and databases run on the same server, making it convenient for quick trials and debugging
- **Multi-node deployment**: Databases, UPF, or certain NFs are separated to simulate a more realistic network topology
- **Containerized environments**: Deployment using Docker / Kubernetes, which facilitates integration with other systems or testing in cloud environments

The following section of this chapter will first introduce the main free5GC repositories on GitHub, and then explain the network component modules in the main repository, laying the groundwork for deeper discussions in later chapters. Detailed deployment methods will be covered in Part III of this book.

### 3.2.1 Main free5GC Repositories on GitHub

free5GC is not hosted as a single repository on GitHub. Instead, it consists of a [main repository](https://github.com/free5gc/free5gc) combined with multiple [auxiliary repositories (submodule)](https://github.com/free5gc/free5gc/tree/main/NFs), forming a complete ecosystem.

In the deployment chapters that follow, we will primarily focus on the main repository, covering the workflow from source code retrieval and build to NF startup. In this chapter, however, the focus is on “understanding the main repository from an architectural and program-structure perspective.”

### 3.2.2 Network Component Modules in the free5GC Main Repository

Within the main repository, free5GC follows the 3GPP 5GC definitions and splits different network functions into multiple independent Network Functions, as shown in the figure below:

![SBI](../../image/part2/sbi.png)

From a functional perspective, these can be roughly categorized as follows:

- **Access & Mobility Management**

    For example, the AMF is responsible for UE registration, connectivity, and mobility management, and serves as the primary entry point for UEs into the 5GC.

- **Session & User Plane Management**

    For example, SMF and UPF handle the establishment, modification, and release of PDU Sessions, as well as actual user data forwarding.

- **Session & User Plane Management**

    For example, AUSF and UDM manage UE authentication procedures and subscription data queries.

- **Policy & Charging**

    For example, PCF and CHF provide policy decisions and charging data feedback.

- **Network Slicing Management**

    For example, NSSF assists in selecting appropriate network slices and routing paths.

In later chapters, we will select key NFs and analyze them in depth from the perspectives of “functional role → signaling flow → program implementation.” To help readers more easily step into the codebase, the next section will first discuss what a typical NF program structure looks like in free5GC.

## 3.3 Program Structure of an NF

Based on the current free5GC implementation aligned with Release 17, the main structure of an NF repository on the SBI side is as follows:

```bash
.
├── cmd
│   └── main.go
├── go.mod
├── go.sum
├── internal
│   ├── context
│   ├── logger
│   ├── sbi
│   │   ├── api_xxx.go
│   │   ├── consumer
│   │   ├── processor
│   │   ├── router.go
│   │   └── server.go
│   └── util
├── LICENSE
└── pkg
```

From this directory structure, we can roughly see how an NF is composed at the code level:

- `cmd/main.go`: The entry point of the NF. It is mainly responsible for reading configurations, creating necessary contexts, initializing logging and various service components, and finally starting external interfaces such as the HTTP/SBI server.
- `go.mod` / `go.sum`: Go module dependency management files that record the external packages and versions used by the NF, ensuring a consistent build environment.
- `internal/context`: Typically contains structures related to the NF’s internal “execution context” and its core business logic.
- `internal/logger`: Encapsulates logging-related configurations and APIs, such as log levels and output formats, so that the entire NF uses a consistent logging interface.
- `internal/sbi`: The main implementation for interacting with other NFs over the 5GC SBA:

    - `api_xxx.go`: API interface definitions and handler skeletons generated from OpenAPI / Swagger or written manually, corresponding to service interfaces defined by 3GPP.
    - `consumer`: Acts as a “client” to call SBI services of other NFs, such as querying subscription data from UDM or obtaining policies from PCF.
    - `processor`: Handles the business logic of SBI messages, converting incoming requests into internal workflows, such as updating state machines or triggering internal context module functions.
    - `router.go` / `server.go`: Code for setting up the HTTP router and actually starting the SBI server, wiring URL paths and methods to their corresponding handlers.

- `internal/util`: Contains common utilities used by the NF, such as string processing, error wrapping, and shared constants, to avoid scattering them across multiple files.
- `pkg`: Contains factory functions for the NF, such as configuration loading and initialization of basic functionalities.
- `LICENSE`: The license file that specifies the open-source license used by the NF / project (free5GC uses Apache 2.0), which is crucial for the legality of subsequent modifications and reuse.

Understanding this directory layout and module responsibility split helps you quickly identify the following things, when reading any free5GC NF:

- Where the program starts (`cmd/main.go`)
- Where to look for internal processing logic and logging configuration (`context`、`logger`)
- Where to trace SBI message flows and behaviors (`sbi` 底下的 server / consumer / processor)
-  Which utilities or shared code can be reused (`util`、`pkg`)

## 3.4 Chapter Summary

This chapter establishes a “global map” of free5GC for the reader from three perspectives:

- First, we clarified the positioning of free5GC—as an open-source 5G Core platform focused on education, research, and PoC—highlighting that its strengths lie in readability and modifiability rather than carrier-grade high availability and full feature coverage.
- Next, centered on the 5GC architecture and the main free5GC GitHub repository, we introduced the roles and relationships of various NFs, helping readers place components such as AMF, SMF, UPF, and PCF back into the broader 3GPP SBA picture.
- Finally, using a typical NF directory structure as an example, we explained the division of responsibilities across directories such as `cmd/main.go`, `internal`, and `pkg`, providing a practical roadmap for “reading the code” in real projects.

<div class="chapter-nav">
  <a href="../chapter4/" class="nav-btn nav-next" title="Next: AMF Deep Dive">
    <span class="arrow"></span>
  </a>
</div>
