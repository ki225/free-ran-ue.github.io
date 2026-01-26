# Chapter 10: Why Do We Need a RAN/UE Simulator?

In R&D, testing, and validation workflows related to the 5G Core (5GC), the ability “to generate RAN / UE behavior in a stable, controllable, and reproducible manner” is often far more important than simply “being able to connect to a real base station.” The value of a RAN / UE simulator lies precisely in enabling **Registration**, **PDU Session establishment**, **user-plane data transmission**, **multi-UE concurrency**, and **abnormal or failure scenarios** to be exercised as fully automated test cases—even when real gNBs / UEs are unavailable or intentionally not used.

This chapter first explains why many teams eventually find simulators indispensable, then introduces commonly used mainstream simulators, along with their respective strengths, limitations, and selection considerations.

## 10.1 Why not just use “real devices”?

1. High cost of commercial simulators / test instruments

    Commercial simulators (or test instruments) typically provide comprehensive protocol support, strong observability, and consistent measurement capabilities, but they also come with several drawbacks:

    - **High cost**: Usually involves expensive licenses or full test equipment purchases, which can be a significant burden for early-stage PoCs or open-source–oriented projects.
    - **Licensing and binding constraints**: Limits on users, feature modules, throughput / number of UEs are common, and scaling often incurs additional cost.
    - **Delivery and maintenance**: Hardware delivery, calibration, maintenance, and environmental dependencies slow down iteration cycles.

2. Real gNB / UE testing is hard to make “controllable and reproducible”

    Even with access to real equipment, teams frequently encounter the following challenges:

    - **Uncontrollable behavior**: Certain UE behaviors are black-boxed, making abnormal scenarios difficult to trigger or precisely reproduce.
    - **Automation challenges**: It is difficult to integrate real RF / hardware into CI pipelines, preventing tests from running on every commit.
    - **Limited scalability**: Simulating large numbers of concurrent UEs (e.g., 100 or 1,000 UEs) quickly becomes costly in both hardware and operational effort.

As a result, most teams gradually introduce RAN / UE simulators across different phases— “feature development → regression testing → stress and concurrency testing → abnormal scenario validation”—to make testing faster, more cost-effective, and reproducible.

## 10.2 Mainstream simulators today: Integrated vs. Decoupled

Current common open-source approaches can generally be divided into two categories:

- **Integrated**: A single tool bundles major RAN and UE behaviors together, providing an out-of-the-box experience focused on “getting the flow running quickly.”
- **Decoupled / Modular**: RAN and UE (or their control-plane and user-plane components) are more modular, allowing components to be swapped for different scenarios and enabling more fine-grained testing and extensibility.

### 10.2.1 Open-source simulator examples

1. Integrated: packetrusher

    `packetrusher` is typically positioned as an integrated solution for “getting things running quickly and validating core network procedures.” Its main advantage is low entry cost and simple deployment, making it suitable for PoCs and educational use cases.

    - Advantages

        - **Quick to get started**: High integration and minimal dependencies make it suitable for rapidly validating basic 5GC procedures.
        - **Automation-friendly**: Relatively easy to integrate into CI / container-based environments for smoke testing.
        - **Lower resource requirements**: Generally lightweight for small-scale tests.

    - Limitations

        - **Limited extensibility and replaceability**: Integrated designs may lack flexibility when deep customization or large-scale concurrency is required.
        - **Protocol/feature coverage depends on version**: Support for detailed 3GPP features can vary across versions and projects, so required functionality must be verified in advance.

2. Decoupled: free-ran-ue

    `free-ran-ue` is positioned as a more “extensible and integrable” RAN / UE simulator, particularly well suited for system-level integration testing and multi-scenario regression. Typical use cases include validation with free5GC under different deployment architectures (single-node / multi-node / multi-UPF) and traffic patterns.

    - Advantages

        - **Extensible architecture**: Well suited for integration into a broader test platform (e.g., dashboards, scenario management, batch jobs).
        - **System integration testing–friendly**: Ideal for long-running tests, continuous regression, and frequent configuration switching.
        - **Aligned with “engineering-grade testing”**: Makes it easier to standardize “scenarios, parameters, and outputs,” forming maintainable test assets.
        - **free5GC integration**: Maintained directly by free5GC committers, ensuring tighter system-level compatibility.

    - Limitations

        - **No drawbacks**: Practically speaking, the only potential limitation is its relatively recent release and smaller user base.

## Simple comparison table (selection reference)

| Category | Tool | Primary use cases | Advantages | Limitations |
| - | - | - | - | - |
| Integrated | packetrusher | Rapid PoC, education, CI smoke tests | Fast setup, simple deployment, automation-friendly | Limited customization / scalability; feature coverage must be verified |
| Decoupled | free-ran-ue | System integration testing, multi-scenario regression, multi-node / multi-UPF validation | Engineering-oriented, extensible, suitable for long-term test assets | - |

## Summary

The core value of RAN / UE simulators is transforming an “uncontrollable real-world environment” into “controllable, reproducible, and automatable tests.” Tool selection should align with team maturity: integrated solutions are ideal for quick validation, while decoupled architectures are better suited for stable regression testing and system integration, where test scenarios can be progressively engineered into durable assets.

<div class="chapter-nav">
  <a href="../chapter11/" class="nav-btn nav-next" title="下一章: Architecture Design of free-ran-ue">
    <span class="arrow"></span>
  </a>
</div>
