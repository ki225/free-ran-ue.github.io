# 第 12 章：模擬器如何與 free5GC 整合

> [!Note]
> 本章節預設讀者都了解如何啟動 free5GC 核心網路，對於核心網路部分不熟的讀者可以參考[第8章](../part3-deploy/chapter8)。

## 12.1 free5GC 中的 UE 資料

在啟動模擬器之前，我們需要先在 free5GC 的 [Web Console](https://github.com/free5gc/webconsole) 中新增一筆訂閱者資料，相當於對核網宣告一個新的使用者設備（UE），具體如何新增可以參考：[Create Subscriber via Webconsole](https://free5gc.org/guide/Webconsole/Create-Subscriber-via-webconsole/).

新增訂閱者資料的意義，可以想成一般情況前往電信業者辦理門號時，需要先把「這張 SIM/這個用戶」登記到電信業者的系統裡：包含身分識別、鑑權用的密鑰、以及允許使用的服務方案。對 free5GC 而言，這筆訂閱者資料就是核心網用來判斷「這個 UE 是否存在、能不能註冊、可以建立哪些 PDU Session」的依據；如果沒有先建立，UE 後續的註冊或鑑權流程通常就會失敗。

實務上，你至少需要確保模擬器端（UE）使用的參數，能與 free5GC Web Console 中建立的訂閱者資料一致，例如：

- **IMSI / SUPI**：用戶的唯一識別（UE 會帶著它進行註冊）
- **鑑權資料（K、OP/OPc 等）**：用於 AKA 鑑權流程
- **切片與服務（S-NSSAI、DNN、允許的 PDU Session 設定）**：決定 UE 可以連到哪些網路/資料服務

完成訂閱者建立後，我們才會進入下一節的整合流程，將 free-ran-ue 以正確的設定接入 free5GC。

## 12.2 整合流程

下一節會再針對不同啟動方式做更細的操作說明；但在整合的「大方向」上，整體流程可以整理為：

1. **啟動核心網路**，並確認所有 NF 都正常運行
2. **透過 Web Console 新增訂閱者資料**（UE subscription）
3. **啟動 gNB 模擬器**，並確認其正常運作（包含）：

    - 正確連接 AMF（控制平面） / UPF（資料平面）
    - 開啟提供 UE 連接的控制平面端口與資料平面端口

4. **啟動 UE** 連接 gNB，並完成 UE 註冊流程與 PDU Session 建立流程

## 12.3 啟動方式

目前 free-ran-ue 提供的啟動方式為以下三種：

1. 獨立機器啟動

    - **優點**

        - **最直觀、最容易除錯**：直接在主機上跑，網路命名、介面與路由相對透明。
        - **效能最好**：少一層虛擬化/隔離，通常更接近裸機表現。
        - **適合快速驗證**：單機 PoC、功能確認、抓 log 都方便。

    - **缺點**

        - **隔離性較弱**：不同測試情境/多組實驗容易互相影響（路由、介面、Port、殘留狀態）。
        - **環境一致性較差**：不同人/不同機器的系統依賴可能造成「你能跑、我不能跑」。

    - **適合誰**

        - 想先把流程跑通、要頻繁改碼/看 log 的開發者。

    > [!Note]
    > 獨立機器啟動操作流程請參考：[官方教學](https://free-ran-ue.github.io/doc-user-guide/02-free-ran-ue/)

2. 命名空間啟動

    - **優點**

        - **隔離性佳但仍貼近主機網路**：可把不同 UE/gNB/情境放到不同 namespace，減少互相干擾。
        - **方便做多情境/多 UE 測試**：同一台機器上做較複雜的拓撲與網路隔離更自然。
        - **除錯仍相對友善**：比容器少一些封裝層，網路問題通常比較好追。

    - **缺點**

        - **操作門檻較高**：需要對 Linux namespace、veth、路由與權限有基本概念。
        - **清理成本**：測試中斷時需要確實清理 namespace/介面，避免殘留影響下次實驗。

    - **適合誰**

        - 想在單機上做更接近真實網路隔離的整合測試、或需要同時跑多組實驗的使用者。

    > [!Note]
    > 命名空間啟動操作流程請參考：[官方教學](https://free-ran-ue.github.io/doc-user-guide/03-quickstart-free-ran-ue/)

3. Docker 容器虛擬化啟動

    - **優點**

        - **環境可重現性最佳**：依賴與版本被容器鎖住，最利於團隊協作與 CI 自動化。
        - **部署/搬移方便**：同一套容器配置可在不同機器快速啟動。
        - **容易與整套測試平台整合**：例如與 free5GC、資料面工具、監控元件一起 compose 起來。

    - **缺點**

        - **網路除錯相對麻煩**：NAT、bridge、port mapping 會增加定位成本。
        - **效能與特權需求**：某些網路/介面操作可能需要 `--privileged` 或額外 capability，使用上要更小心，以 free5GC 來說，UPF 就會依賴核心的 gtp5g 模組來做封包的轉送。

    - **適合誰**

        - 想做可重複的整合測試、要上 CI、或要在多台環境快速部署的使用者。

    > [!Note]
    > Docker 容器虛擬化啟動操作流程請參考：[官方教學](https://free-ran-ue.github.io/doc-user-guide/10-docker/)

> [!Tip]
> 快速選擇建議（懶人決策）
>
> - **我在開發/除錯、想最快看到 log**：選 **獨立機器**
> - **我要在同一台機器上跑多情境、希望網路隔離更清楚**：選 **命名空間**
> - **我要可重現、要團隊共用、要跑 CI/自動化**：選 **Docker**

## 12.4 設定檔配置

啟動的核心概念與設定檔配置很簡單：**讓 free5GC（核心網）與 gNB/UE（模擬器）在同一個可互通的網路上，且 IP/Port 設定彼此一致**。

在第 8 章部署 free5GC 時，你會在 AMF/SMF/UPF 的設定檔中配置對外服務的位址；同樣地，free-ran-ue 的 gNB 設定檔也必須填入對應的 AMF/UPF 位址，才能成功建立控制面與資料面的連線。

1. 網路與位址規劃（建議做法）

    以下以 free-ran-ue 提供的 [gNB 設定](https://github.com/free-ran-ue/free-ran-ue/blob/main/config/gnb.yaml) 與 [UE 設定](https://github.com/free-ran-ue/free-ran-ue/blob/main/config/ue.yaml) 為例，將整體拓撲拆成兩段理解：

    - **核心網 ↔ gNB（N2/N3）**：使用 `10.0.1.0/24`

        - 核心網（AMF/UPF）：`10.0.1.1`
        - gNB（RAN）：`10.0.1.2`

    - **UE ↔ gNB（本地接入網段）**：使用 `10.0.2.0/24`

        - gNB 對 UE 開放的接入位址：`10.0.2.1`
        - UE 會透過此位址與 gNB 建立控制面/資料面連線

    > [!Note]
    > 重點：獨立機器模式通常不會幫你「自動做網路隔離」。因此你要先確保上述兩個網段在你的主機上可被正確路由/綁定（例如有對應的介面或路由規則），否則即使設定檔寫對，連線也會因為網路不可達而失敗。

2. gNB 設定檔重點

    下面列出最常需要對齊的欄位（其餘如 `plmnId`/`tai`/`snssai` 需與 UE/訂閱者資料一致）：

    - **核心網對接（N2/N3）**

        - `amfN2Ip`：AMF 的 N2 IP（例：`10.0.1.1`）
        - `ranN2Ip`：gNB 用來連 AMF 的 N2 IP（例：`10.0.1.2`）
        - `upfN3Ip`：UPF 的 N3 IP（例：`10.0.1.1`）
        - `ranN3Ip`：gNB 用來連 UPF 的 N3 IP（例：`10.0.1.2`）
        - `amfN2Port`：AMF N2 port（常見 `38412`）
        - `upfN3Port`：UPF N3 GTP-U port（常見 `2152`）

    - **提供 UE 接入（RAN 對 UE 開放）**

        - `ranControlPlaneIp` / `ranControlPlanePort`：UE 連線用的控制面位址/埠（例：`10.0.2.1:31413`）
        - `ranDataPlaneIp` / `ranDataPlanePort`：UE 連線用的資料面位址/埠（例：`10.0.2.1:31414`）

    - **Console API（選用）**

        - `api.ip` / `api.port`：提供 Web Console 拉取狀態用的 API 位址/埠（例：`10.0.1.2:40104`）

3. UE 設定檔重點（`ue` 區段）

    UE 設定檔的重點是「連到哪個 gNB」以及「UE 自己是誰」：

    - **連到 gNB 的接入位址**

        - `ranControlPlaneIp` / `ranControlPlanePort`：必須與 gNB 的 `ranControlPlaneIp/Port` 相同（例：`10.0.2.1:31413`）
        - `ranDataPlaneIp` / `ranDataPlanePort`：必須與 gNB 的 `ranDataPlaneIp/Port` 相同（例：`10.0.2.1:31414`）

    - **身分與鑑權（需與 free5GC 訂閱者一致）**

        - `plmnId (mcc/mnc)` + `msin`：共同組成 UE 的識別（IMSI/SUPI 的組成來源之一）
        - `authenticationSubscription`：鑑權所需資料（例如 `encPermanentKey`、`encOpcKey`、`sequenceNumber`）

    - **資料面服務**

        - `pduSession.dnn`：要連的 DNN（例：`internet`）
        - `pduSession.snssai`：要用的 slice（需與訂閱者設定一致）
        - `ueTunnelDevice`：UE 建立本地 TUN 介面的名稱前綴（例：`ueTun`，常見會生成 `ueTun0`）

4. 啟動順序與驗證方式

    建議按以下順序啟動並逐步驗證（每一步通過再往下）：

    1. **啟動 free5GC**，確認 AMF/UPF 等 NF 都正常，且 AMF（N2）與 UPF（N3）位址可被 gNB 連到
    2. **在 Web Console 建立訂閱者**，確保 IMSI/鑑權資料/S-NSSAI/DNN 與 UE 設定一致
    3. **啟動 gNB**

        - 你應該能在 log 看到與 AMF 的 NGAP 建立成功
        - 若有用戶面，應能看到與 UPF 的資料面對接/隧道相關訊息

    4. **啟動 UE**

        - 你應該能看到 UE 註冊成功、PDU Session 建立成功
        - 在主機上可觀察到 UE 的 TUN 介面（例如 `ueTun0`）出現

    > [!Caution]
    > 若卡住：第一優先檢查「IP 是否可達」與「設定檔的 IP/Port 是否對齊」，其次才看鑑權資料與 slice/DNN 是否一致。

## 12.5 本章小結

本章說明 free-ran-ue 與 free5GC 整合時，最關鍵的不是「指令怎麼打」，而是先把整體依賴關係理清楚：核心網要先有對應的訂閱者資料，gNB 才能完成控制面對接並為 UE 的資料面鋪路，UE 才能順利完成註冊與 PDU Session 建立，最終在本機建立如 `ueTun0` 的資料通道。

在實作上，你可以用兩個原則快速排查大多數問題：

- **先連通、再鑑權、最後資料面**：先確認 IP/Port 可達與對接成功，再檢查 IMSI/鑑權資料與 slice/DNN 是否一致，最後才看 PDU Session 與用戶面封包是否能跑通。
- **設定要一致**：gNB/UE 設定檔中的 PLMN、S-NSSAI、DNN 與 free5GC Web Console 的訂閱者設定，必須保持一致，否則流程會在註冊或 session 建立階段卡住。

此外，本章也比較了三種啟動方式（獨立機器、命名空間、Docker）。你可以依照「除錯便利性、隔離需求、可重現性與是否要上 CI」來選擇合適的部署方式，並將整合流程逐步工程化成可重複使用的測試環境。
