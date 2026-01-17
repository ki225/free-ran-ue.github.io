# 5G Core Easy Learning: free5GC and free-ran-ue

> [!Important]
> Welcome to the *5G Core Book*!
>
> This book is designed to help you understand the **5G Core (5GC)**, **deploy free5GC**, and build your own **RAN / UE simulator** in the most efficient way possible. Even if you’re completely new to 5G, you’ll be able to grow into a 5G core network builder who’s ready to work at the cutting edge!

## Why did we write this book?

When first stepped into the 5G world, everything felt extremely abstract. The standards were packed with interfaces, procedures, and acronyms—AMF, SMF, UPF, N1/N2/N3, PDU Session…

For many beginners, reading 3GPP specifications or trying to make sense of slide decks often comes with the same pain points:


- **Too many acronyms**: You see a long list of abbreviations, but can’t immediately map them to what they actually do.
- **Concepts feel too abstract**: You may know the names of network functions, but struggle to connect them to real packets and real procedures.
- **3GPP specs are massive**: The documents are extremely detailed and complex, making it hard to know where to start.

That’s why this book aims to guide you into the world of 5G in the simplest, most intuitive, and most hands-on way, helping you build your own 5G core network environment step by step.

---

## Who is this book for?

- **Students or engineers new to 5G** who need a structured learning path.
- **Readers who already understand 4G / EPC** and want to learn what 5GC adds and what changes.
- **Anyone interested in free5GC** who wants to understand its architecture and the role of each module.
- **Those who want a reproducible 5G lab environment**, including both the core network and a RAN / UE simulator.

You don’t need to be fluent in 3GPP specs from day one, but if you have basic networking knowledge (IP / TCP / UDP), Linux experience, and familiarity with Docker or Kubernetes, the book will feel much smoother to follow.

---

## Book structure and how to read it

This series is roughly divided into four parts (feel free to jump around based on your needs):

- [**Part I - Foundation and Evolution of 5G Core Network**](./part1-history/chapter1)

    Traces the evolution from 4G EPC to the 5G Core, and introduces the 3GPP’s overall view of 5GC, helping you build a solid big-picture understanding of the system architecture.

- [**Part II - In-Depth Analysis of free5GC Architecture**](./part2-free5gc/chapter3)

    Breaks down core network functions such as AMF, SMF, and UPF one by one, mapping them to real modules and procedures.

- [**Part III - free5GC Deployment and Common Issues**](./part3-deploy/chapter8)

    Covers deployment scenarios ranging from single-node setups to multi-node and multi-UPF architectures (Docker / Kubernetes), along with common pitfalls and practical troubleshooting strategies.

- [**Part IV - free-ran-ue Simulator**](./part4-free-ran-ue/chapter11)

    Explains why a RAN / UE simulator is needed, the design philosophy of free-ran-ue, and how to integrate it with free5GC for experiments.

**If this is your first time learning 5G:**

1. Start by reading **Part I** in full to establish a solid understanding of the overall architecture.
2. Continue with **Part II / Part II**I for deeper concepts or hands-on practice.
3. Finally, read **Part IV** and run an End-to-End test using free-ran-ue.

---

## Lab environment and open-source projects

This series heavily relies on the following open-source projects:

- **free5GC**: An open-source 5G Core project

    For the official site and source code, refer to the [free5GC Official Site](https://www.free5gc.org/) and its GitHub repository.

- **free-ran-ue**: The RAN / UE simulator provided by this project
    The source code and documentation are available on [GitHub](https://github.com/free-ran-ue/free-ran-ue)。

For all lab steps in this book, we aim to ensure:

- **Reproducibility**: With the same commands and configurations, you should be able to run everything in your own environment.
- **Step-by-step explanations**: We don’t just give commands, we explain why we do it and which network function or procedure it corresponds to.

<div class="chapter-nav">
  <a href="author/" class="nav-btn nav-next" title="Next：About the Author">
    <span class="arrow"></span>
  </a>
</div>
