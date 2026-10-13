# DMIT 香港評測：三條線路延遲與路由實測、全套餐價格對比，搞懂 CN2 GIA、Eyeball、Tier 1 到底差在哪

搜「DMIT 香港評測」的人，通常不是想知道 DMIT 是誰，而是想知道一件更具體的事：官網上那三條線路、十幾個方案，價格從年付 36.9 美元一路到月付 500 美元，到底差在哪裡，我這一單該下在哪一格。

所以這篇不鋪陳品牌故事，直接從線路、數字、價格三個角度拆。

## 先給結論：什麼人該買，什麼人別碰

DMIT 香港節點部署在 Equinix HK2，硬體是 KVM 架構配 AMD EPYC，全系列標配 1 個 IPv4 加 1 個 IPv6 /64，付款支援支付寶、微信、PayPal 和信用卡。

判斷邏輯其實只有一條：**你的訪客在哪裡。**

- 訪客在中國大陸、對延遲敏感 → 只有 Premium（HKG.Pro）值得考慮，其餘兩條線路在這件事上幫不上忙。
- 訪客是移動寬頻為主、預算中等 → Eyeball（HKG.EB）是折衷，但官方把它標為 Beta，這句話不是裝飾。
- 訪客在歐美、東南亞、日本，或你自己有中轉方案 → Tier 1（HKG.T1）性價比最高，流量和頻寬給得最兇。

反過來說，如果你的網站主要服務大陸用戶，卻為了省錢買了 Tier 1，結果很可能是電信去程繞到歐洲再回香港，延遲比買美西機器還難看。這不是猜測，是實測路由裡出現過的結果。

## 三條線路的本質差別：差別在路由，不在硬體

三個系列用的都是同一套虛擬化架構和同一代 EPYC 平台，價格落差全部來自「封包走哪條路」。

| 線路系列 | 定位 | 中國大陸優化 | 頻寬特性 | 適合誰 |
| --- | --- | --- | --- | --- |
| Premium（HKG.Pro） | 旗艦 | 高，三網回程經香港 CTG 接入 CN2 GIA | 1Gbps，流量給得保守 | 面向大陸用戶的正式業務 |
| Eyeball（HKG.EB） | 中階 | 中等，回程移動 CMI、聯通 CUG+CMI 動態負載 | 1Gbps，流量約為同價 Pro 的 160% | 兼顧大陸住宅寬頻與預算 |
| Tier 1（HKG.T1） | 入門 | 無 | 4–10Gbps，流量極大 | 國際業務、大流量、自建落地 |

Premium 的具體路由是：電信去程走 CN2 GIA 直連香港 CTG，聯通去程經 AS4837 與 AS10099 進香港，移動去程走 CMI；回程三網統一收束到香港 CTG 再接回國內 CN2 GIA。這種「去程各自骨幹直連、回程強制走 CN2」的設計，是它延遲數字好看的主要原因。

Tier 1 則相反，它接的是國際骨幹（含 RETN），主打香港到歐洲的低延遲，對大陸沒有專項優化。

## 延遲與路由實測：三條線各自跑出什麼數字

### Premium：43–49ms 區間，晚高峰最穩

多組第三方測試的結果落在同一個範圍。有一組全國三網 Ping 平均約 43ms（電信 42ms、聯通 42ms、移動 46ms）；另一組針對 AN5 平台的測試，大陸多地平均延遲約 49ms。

硬碟 I/O 部分，AN5 平台（AMD EPYC 9656P）測到約 1050MB/s，UnixBench 單核約 1285 分。這個 I/O 表現在香港節點裡算前段班。

### Eyeball：30–70ms，但官方自己標了 Beta

回程走移動 CMI 的表現不錯，到國內主要城市大致落在 30–70ms，晚高峰波動不算劇烈。要留意的是 DMIT 官方頁面明確把 HKG.EB 標為 Eyeball Network Beta，並註明網路路由與效能仍會持續調整，不建議用在對穩定性要求高的生產環境。

這句話要當真看。同一條線路在不同時間測出不同路由，是 Beta 階段的正常現象，不要把某一次測試結果當成長期固定的線路。

### Tier 1：大陸不保證，國際方向很漂亮

以 HKG.T1.WEE 的實測為例，硬體是 AMD EPYC 7C13 單核，Geekbench 5 單核 944、多核 927，64k 區塊 fio 讀寫約 520MB/s。國際延遲表現：新加坡約 30ms、日本東京約 80ms、台灣約 80ms、芬蘭約 130ms、柏林約 145ms、洛杉磯約 180ms。

至於大陸方向，電信去程在測試中出現了先繞荷蘭再回香港的走法，聯通則是繞日本 NTT。移動用戶體驗相對好一些。買之前務必先 ping 測試 IP，DMIT 公開的測試位址包括 Premium 的 103.117.100.199 與 191.222.216.216、Eyeball 的 154.12.188.3、Tier 1 的 154.12.176.28。

## 全套餐價格對比

先說一句必須說的話：DMIT 的庫存和配置調整很頻繁，同一系列會出現 AS3 與 AN5 兩代平台並存、部分方案售完即止的情況，實際可下單的項目與價格請以下單頁顯示為準。下表整理的是目前公開資料中出現的方案與定價。

### Tier 1（HKG.T1，國際線路）

| 方案 | CPU | 記憶體 | SSD | 頻寬／月流量 | 超量後 | 價格 | 購買 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| WEE | 1 核 | 1GB | 20GB | 4Gbps / 1TB | 50Mbps 不限量 | $36.9／年 | [ 查看 T1 全系列即時庫存](https://www.dmit.io/aff.php?aff=12046) |
| TINY | 1 核 | 1GB | 20GB | 4Gbps / 2TB | 100Mbps 不限量 | $6.9／月 | [ 查看 T1 全系列即時庫存](https://www.dmit.io/aff.php?aff=12046) |
| STARTER | 1 核 | 2GB | 40GB | 10Gbps / 4TB | 100Mbps 不限量 | $12.9／月 | [ 查看 T1 全系列即時庫存](https://www.dmit.io/aff.php?aff=12046) |
| MINI | 2 核 | 2GB | 60GB | 10Gbps / 8TB | 100Mbps 不限量 | $21.9／月 | [ 查看 T1 全系列即時庫存](https://www.dmit.io/aff.php?aff=12046) |
| MICRO | 4 核 | 4GB | 80GB | 10Gbps / 16TB | 200Mbps 不限量 | $32.9／月 | [ 查看 T1 全系列即時庫存](https://www.dmit.io/aff.php?aff=12046) |
| MEDIUM | 4 核 | 8GB | 160GB | 10Gbps / 32TB | 200Mbps 不限量 | $49.9／月 | [ 查看 T1 全系列即時庫存](https://www.dmit.io/aff.php?aff=12046) |
| LARGE | 8 核 | 16GB | 320GB | 10Gbps / 64TB | 500Mbps 不限量 | $99.9／月 | [ 查看 T1 全系列即時庫存](https://www.dmit.io/aff.php?aff=12046) |
| GIANT | 8 核 | 24GB | 640GB | 10Gbps / 128TB | 1Gbps 不限量 | $199.9／月 | [ 查看 T1 全系列即時庫存](https://www.dmit.io/aff.php?aff=12046) |

T1 最值得記的一點是流量用完不斷線，而是降速繼續跑。WEE 降到 50Mbps、其餘多數降到 100–200Mbps。對建站或長跑服務來說，這比月底流量歸零然後服務直接被停要友善得多。

### Eyeball（HKG.EB，官方標示 Beta）

| 方案 | 平台 | CPU | 記憶體 | SSD | 月流量／埠 | 價格 | 購買 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| HKG.AS3.EB.WEE | AS3 | 1 核 | 1GB | 20GB | 500GB / 500Mbps | $24.9／月 | [ 比較 Eyeball 各檔配置](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.EB.TINY | AS3 | 1 核 | 1GB | 20GB | 800GB / 1Gbps | $39.9／月 | [ 比較 Eyeball 各檔配置](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.EB.STARTER | AS3 | 1 核 | 2GB | 40GB | 1.5TB / 1Gbps | $79.9／月 | [ 比較 Eyeball 各檔配置](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.EB.MINI | AS3 | 2 核 | 4GB | 60GB | 2.2TB / 1Gbps | $126.9／月 | [ 比較 Eyeball 各檔配置](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.EB.MICRO | AS3 | 4 核 | 4GB | 80GB | 3TB / 1Gbps | $179.9／月 | [ 比較 Eyeball 各檔配置](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.EB.MEDIUM | AS3 | 4 核 | 8GB | 160GB | 4TB / 1Gbps | $239.9／月 | [ 比較 Eyeball 各檔配置](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.EB.MINI | AN5 | 4 核 | 4GB | 80GB | 2.2TB / 1Gbps | $149.9／月 | [ 比較 Eyeball 各檔配置](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.EB.MICRO | AN5 | 4 核 | 4GB | 160GB | 3TB / 1Gbps | $199.9／月 | [ 比較 Eyeball 各檔配置](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.EB.MEDIUM | AN5 | 6 核 | 8GB | 160GB | 4TB / 1Gbps | $279.9／月 | [ 比較 Eyeball 各檔配置](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.EB.LARGE | AN5 | 8 核 | 16GB | 320GB | 4.5TB / 1Gbps | $359.9／月 | [ 比較 Eyeball 各檔配置](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.EB.GIANT | AN5 | 12 核 | 24GB | 640GB | 9TB / 1Gbps | $759.9／月 | [ 比較 Eyeball 各檔配置](https://www.dmit.io/aff.php?aff=12046) |

要提醒一件事：Eyeball 這條產品線動過大刀。HKG EBv2 系列已於 2026 年 8 月 10 日正式停產並停止服務，原有用戶獲得退款，並可透過工單申請其他產品的循環折扣（Pro 20%、EB 25%、T1 30%，不含 WEE）。這說明 Eyeball 系列的定位仍在調整中，買之前想清楚自己能不能接受這種變動。

### Premium（HKG.Pro，CN2 GIA）

| 方案 | 平台 | CPU | 記憶體 | SSD | 頻寬／月流量 | 價格 | 購買 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| HKG.Pro.TINY | AS3 | 1 核 | 1GB | 20GB | 1Gbps / 500GB | $39.9／月 | [ 入手 CN2 GIA 旗艦方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.STARTER | AS3 | 1 核 | 2GB | 40GB | 1Gbps / 1TB | $79.9／月 | [ 入手 CN2 GIA 旗艦方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.MINI | AS3 | 2 核 | 2GB | 60GB | 1Gbps / 1.5TB | $126.9／月 | [ 入手 CN2 GIA 旗艦方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.MICRO | AS3 | 2 核 | 4GB | 80GB | 1Gbps / 2TB | $179.9／月 | [ 入手 CN2 GIA 旗艦方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.MEDIUM | AS3 | 4 核 | 4GB | 160GB | 1Gbps / 2.5TB | $239.9／月 | [ 入手 CN2 GIA 旗艦方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.Pro.MINI | AN5 | 2 核 | 2GB | 80GB | 1Gbps / 1.5TB | $149.9／月 | [ 入手 CN2 GIA 旗艦方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.Pro.MICRO | AN5 | 2 核 | 4GB | 160GB | 1Gbps / 2TB | $159.9／月 | [ 入手 CN2 GIA 旗艦方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.Pro.MEDIUM | AN5 | 4 核 | 4GB | 160GB | 1Gbps / 2.5TB | $180／月 | [ 入手 CN2 GIA 旗艦方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.Pro.LARGE | AN5 | 4 核 | 8GB | 320GB | 1Gbps / 3TB | $240／月 | [ 入手 CN2 GIA 旗艦方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.Pro.GIANT | AN5 | 8 核 | 16GB | 640GB | 1Gbps / 6TB | $500／月 | [ 入手 CN2 GIA 旗艦方案](https://www.dmit.io/aff.php?aff=12046) |

Premium 的溢價全在線路上。$39.9 一個月只給 500GB 流量，同樣的錢在 Eyeball 能拿到更多資源。如果你的專案對 50ms 以內的延遲沒有硬需求，這筆錢花得會有點冤。

## AS3 與 AN5：同一條線路，兩代硬體

看價格表會發現同一條線路底下有兩種命名，這是平台代際差異。

- **AS3**：AMD EPYC 7003 系列，較成熟的平台，價格相對低。
- **AN5**：AMD EPYC 9005 系列，配 DDR5 ECC 記憶體與 PCIe 5.0 NVMe，單核效能與 I/O 較強，同級的硬碟容量通常也更大。

實際上 DMIT 已把 HKG Pro 的常規產品逐步升級到 AN5，Tier 1 仍以 AS3 為主。硬體代際帶來的差異主要體現在單核效能與磁碟吞吐，網路線路本身不變——所以如果只是建站或跑 API，AS3 通常夠用；要跑資料庫、爬蟲、影音處理這類吃單核和 I/O 的工作，AN5 的差價才划得來。

順帶一提，DMIT 在 2025 年 9 月上線了跨機房內網互通，香港與美國、日本與美國、香港與日本之間可以走內網傳資料。有多機房部署需求的話，這個功能比規格表上的數字實用。

## 優惠碼與計費方式

DMIT 的折扣幾乎都是**活動制**，不是常駐。歷史上比較有代表性的一組是香港 Tier 1 升級期間的循環折扣碼：年付 `HKG-T1-ANNUALLY-45OFF-RECUR`（55 折）、半年付 `HKG-T1-SEMIANNUALLY-35OFF-RECUR`（65 折）、季付 `HKG-T1-QUARTERLY-20OFF-RECUR`（8 折），限定 HKG T1 STARTERv2 以上方案。

不過 DMIT 官方的活動頁面目前已標示 HKG Tier 1 的產品升級促銷已經結束，並註明所有折扣碼只在活動期間內有效。這家商家的優惠碼生命週期就是這樣——看到不等於現在還能用。

實際操作建議只有一個：選好方案與付款週期後，在結帳頁面填入折扣碼，按 Validate Code 驗證，系統會直接告訴你有效沒效。驗不過就是過期了，別浪費時間找替代寫法。

付款方式支援支付寶、微信支付、PayPal 與信用卡，這對大陸用戶門檻不高。

## 買之前該知道的幾件事

**IP 相關。** DMIT 不保證月付方案分配的 IP 全球可用；下單時可以加購約 1 美元的保障，確保拿到可用的 IP。若沒加購又需要換 IP，一般收 5 美元一次。Premium 系列另有政策，付費週期內被牆的 IP 可免費更換，週期約 15 天一次——這對面向大陸的站長是實質性的保障。

**流量超量處理。** Tier 1 明確是降速不停機，這是它最被低估的優點。其餘系列的處理方式與方案有關，下單前看清楚該方案頁面的說明。

**退款窗口很短。** DMIT 的退款條件以服務條款為準，且通常要求購買後極短時間內提出、流量用量極少。條款會調整，下單前直接去讀一遍，不要憑印象。

**維護不一定提前發郵件。** 有第三方測評提到，DMIT 在升級宿主機時通常不會提前通知，用戶可能突然發現伺服器短暫不可用。對 SLA 有硬性要求的業務要自己做好監控與備援。

**原生 IP 解鎖。** 香港節點的 IP 屬性在測試中能解鎖部分香港區流媒體服務，但商家不做保證，屬於「可以試，別當賣點」的範疇。

## 按場景給建議

| 你的情況 | 建議 |
| --- | --- |
| 面向大陸用戶的正式網站，對延遲與丟包敏感 | HKG.Pro，從 STARTER 起跳，$39.9 的 TINY 流量偏少，容易撐不住 |
| 個人專案、開發環境，大陸訪問要「還行」 | HKG.EB，$39.9 的 TINY 是合理起點，但要接受 Beta 標籤 |
| 跨境電商、面向歐美與東南亞的 API 服務 | HKG.T1，STARTER $12.9／月或 WEE $36.9／年 |
| 大流量落地、CDN 邊緣節點 | HKG.T1 中高階方案，10Gbps 加降速不停機 |
| 只想先試水溫 | 先買 HKG.T1.WEE 年付 $36.9，成本最低，把香港節點的網路品質摸一遍再升級 |

## 結論

DMIT 香港的邏輯其實不複雜：**T1 拼流量與頻寬，Eyeball 講平衡，Premium 追延遲。**

真正容易踩的坑有兩個。一是把 Tier 1 當成「便宜的香港節點」買給大陸用戶用，結果延遲比美西還差；二是把 Eyeball 當成 Premium 的平替，忽略官方自己掛著的 Beta 標籤。

如果你已經確定自己需要的是大陸方向的最佳延遲，Premium 的溢價就是你要付的代價；如果只是需要一個香港 IP、大頻寬、大流量，Tier 1 在這個價位帶很難找到對手。

不確定就別一次砸錢，先月付測一輪，路由對了再考慮年付——DMIT 的庫存常缺常補，但折扣碼年年重來，沒必要為了一個碼買錯線路。
