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
> 2. The NRF receives and stores the SMF registration information
> 3. The AMF receives a PDU session establishment request from the UE
> 4. The AMF queries the NRF for the SMF address and a JWT authentication token
> 5. The AMF uses the obtained address and JWT to formally send the request to the SMF

## 6.5 NSSF

NSSF（Network Slice Selection Function）負責 **協助選擇合適的網路切片（Slice）**。當 UE 註冊或建立 PDU 會話時，AMF 與 SMF 會根據 UE 的訂閱資訊、服務需求與網路配置，決定應該把它放到哪一個切片；在這個過程中，NSSF 會提供建議與決策支援。

在 free5GC 的實作中，NSSF 的邏輯同樣較簡化，但整體概念仍然保留：

- 根據 UE 的 S-NSSAI 請求與訂閱資料，判斷哪些切片可用
- 協助 AMF / SMF 進行切片選擇與對應的 NF 選路

對於需要多租戶或企業專網場景的讀者來說，理解 NSSF 的角色，有助於你設計「同一套 5GC，服務不同客戶與業務」的架構。

> [!Note]
> S-NSSAI（Single Network Slice Selection Assistance Information）邏輯上由兩個欄位組成：
>
> 1. **ST（Slice/Service Type）**：用來表示「這條切片的大類型」，例如 eMBB、URLLC、mMTC 等。3GPP 規範中通常以一個 8-bit 的數值來區分不同的服務類型（例如 1 = eMBB，2 = URLLC，3 = mMTC），營運商也可以自行定義其他數值代表特定業務類別。
> 2. **SD / SSD（Slice Differentiator）**：用來在「同一種 ST 類型」下面，再進一步區分不同的實際切片實例。3GPP 標準名稱為 SD（24-bit），有些實作或文件會寫成 SSD，本書在描述概念時統一稱為 SSD。透過 SSD，營運商可以在同一 ST 之下建立多個邏輯獨立的切片，例如多個不同企業專網。

## 6.6 PCF

PCF（Policy Control Function）是 5GC 中的 **「策略中樞」**，負責管理與下發與 QoS、接入控制、路由選擇與計費相關的策略。當 SMF、AMF 或其他 NF 在處理 UE 流量或會話時，如果需要決定「這條流量該給多少頻寬、允不允許進特定 DNN、應該用哪個計費方案」，通常都會透過 SBI 向 PCF 請求策略。

在 free5GC 中，PCF 會：

- 儲存一組相對簡化的策略與 QoS 配置（例如對特定 DNN / Slice 的預設 QoS）
- 接收 SMF/AMF 的查詢，回傳對應的策略結果（如 QoS Profile、Policy Rule）

你可以把 PCF 想像成「整個 5GC 的規則資料庫與決策服務」，前面第 5 章中介紹的 PDR/FAR/QER/URR，很多就是在這裡被定義或影響，再由 SMF 轉譯成 UPF 可執行的規則。

## 6.7 UDM

UDM（Unified Data Management）是 5GC 中主要的 **用戶訂閱資料與部份狀態的管理中心**。它大致承襲了 4G 時代 HSS 的角色，但在 5G 中被重新設計，以更符合 SBA 與雲原生架構。

UDM 的主要職責包括：

- 儲存與提供用戶的基本資料（SUPI、服務權限、漫遊權限等）
- 與 AUSF 合作，提供鑑權所需的金鑰與鑑權向量資訊
- 為 AMF、SMF 等網元提供與訂閱相關的查詢服務（例如可用的切片、允許的 DNN 等）

在 free5GC 中，UDM 常與資料庫（如 MongoDB）搭配使用，你在部署與測試時會經常與它打交道，特別是在修改用戶資料或測試不同策略組合時。

## 6.8 UDR

UDR（Unified Data Repository）可以視為 **5GC 中更通用的資料儲存層**，用來存放各種由不同 NF 管理的資料，如訂閱資料、策略資訊或應用相關資料。UDM、PCF 等 NF 可以把自己的資料持久化在 UDR 中，透過標準化的介面進行存取。

在某些實作中，UDM 會被視為「邏輯上的用戶資料管理服務」，而實際的資料則存放在 UDR；PCF 也可以利用 UDR 來儲存策略配置。對於讀者而言，只要記得：UDR 主要解決的是「資料如何統一儲存與存取」的問題，而不是直接參與訊號流程。

> [!Caution]
> 很容易把 UDM 和 UDR 搞混，可以用一句話來區分：
>
> - **UDM 是「會思考的服務」**：提供 API、實作 5GC 規範裡關於用戶資料與鑑權的邏輯（誰能用、怎麼驗證、允許哪些切片/DNN 等）。
> - **UDR 是「會存東西的資料庫」**：專注在統一儲存各種 NF 的資料（包含 UDM 的資料），本身不負責 5GC 訊號流程與決策邏輯。

## 6.9 Web console

根據 3GPP 的規定，5G 系統應提供統一的 OAM（Operations, Administration and Maintenance）介面，讓營運者可以管理用戶資料、監控網路狀態並收集各種統計與計費資訊。free5GC 並沒有實作一套完整、符合所有標準的 OAM 平台，但提供了一個實用的 **Web Console（網頁管理介面）** 作為輕量版的 OAM 解決方案。

在 free5GC 的實作中，這套 Web Console 主要具備兩類功能：

- **資料管理與配置**：透過前端頁面直接對後端的 MongoDB 進行操作，包含：

    - 新增、刪除或修改 UE（使用者）資料與相關訂閱設定
    - 管理切片（S-NSSAI）、DNN、策略等與用戶相關的靜態配置

- **狀態監控與報表輸出**：在核心網啟動並運作時，Web Console 及其後端元件可以：

    - 透過與 CHF 建立 FTP 或其他方式的連線，收集並匯出 UE 使用量與計費相關報告，方便後續分析或對接外部系統

對於正在學習或實驗 free5GC 的讀者來說，這套 Web Console 的重點是提供一個直覺的圖形介面，協助你更輕鬆地管理 MongoDB 中的用戶與策略資料，並觀察系統在實際運作時產生的各種統計與計費資訊。

> [!Note]
> Web console 中的欄位資料，與其代表的意義，本書將於[第11章](../part4-free-ran-ue/chapter12)做詳細的介紹！

## 6.10 本章小結

本章簡要介紹了 AUSF、CHF、NEF、NRF、NSSF、PCF、UDM、UDR 以及 free5GC 的 Web Console，這些元件雖然不像 AMF/SMF/UPF 那樣直接參與每一條訊號與資料路徑，但卻分別在鑑權、安全、策略、計費、切片選擇與資料管理等面向扮演關鍵角色。理解它們的職責與互動關係，有助於你在後續閱讀程式碼或設計實驗拓樸時，不會只看到「幾個主要 NF」，而能把整個 5GC 當成一個完整且可擴展的系統來思考。

<div class="chapter-nav">
  <a href="../chapter7/" class="nav-btn nav-next" title="Next: Non-3GPP Access">
    <span class="arrow"></span>
  </a>
</div>
