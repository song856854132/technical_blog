# Wireless Mobile Network
賴威光 教授  
###### tags: `class notes`

## Transmission Fundamentals
### sine wave:
>$s(t) = Asin{(2\pi ft+\phi)}$

### Nyquist Bandwidth
>given bandwith B, the highest transmission rate is 2B.
 
### SNR(signal to noise ratio)
>$(SNR)_{dB} = 10\cdot log_{10}{signal\over noise}$ 
-->high SNR mean high quality signal  

### Shannon Capacity Formula
> $C = B\cdot log_{2}{(1+SNR)}$

## IEEE 802.11
### 802.11 MAC Architecture

$\fbox{...contention-free...}$ $\fbox{...contention..}$
$\fbox{PCF...PCF...PCF}$
$\fbox{DCF...DCF...DCF...DCF...DCF...}$


### A lot of graph

## IEEE 802.11e
### 802.11e MAC Architecture
$\fbox{...contention-free...}$ $\fbox{...contention..}$
$\fbox{..PCF..}$ $\fbox{...HCCA...}$ $\fbox{.....EDCA.....}$ 
$\fbox{DCF...DCF...DCF...DCF...DCF...}$
* inter frame space priority
    *  $\fbox{SIFS(control frame)}$>$\fbox{PIFS(PCF frame)}$>$\fbox{DIFS(DCF frame)}$>$\fbox{AIFS(EDCF frame)}$

:::warning
### EDCF
EDCA提供在802.11 Contention-Based頻道存取priority的QoS服務。此優先順序是以區分訊務 流量類別而實現。4個(語音、視訊、盡力傳送、背景)Access Categories類別被指派至不同的資料佇列(Queue),每 個佇列根據下列基礎指定優先順序及使用時間存取媒介：

* 仲裁內框間隔(Arbitration Inter Frame Spacing, AIFS)
* 競爭窗框(Contention Window, CWmin/CWmax)
* 傳輸機會(Transmission Opportunity, TXOP)

![](https://i.imgur.com/SHsQnjS.png)
![](https://i.imgur.com/Icl10Yi.png)

:::

:::success
### HCCA
HCF Controlled Channel Access (HCCA) HCCA是HCF的一構成要素(component) 同時提供服務 品質(QoS)參數化(parameterized)的支持。

它除傳承傳統(legacy) PCF的規則外，同時引進一些新 規則。

HCCA不同於PCF的運作，而對無線媒介(Wireless Medium )提供polled access，這可用在無競爭時期(CFP) 與競爭時期(CP)。 


:::

## Bluetooth
![](https://i.imgur.com/2ZTvixS.png)

### connection 
![](https://i.imgur.com/zxykCh9.png)

### inquiry
在 Bluetooth 裝置還未進入連線狀態之前，並不隸屬於任何 Piconet 網路的成員，也未與任何 Master 之間的時序達到同步，並且跳頻順序也不相同，因此，Master 裝置必須利用特殊的方法，才能執行 Inquiry 與 Paging 呼叫。Master 執行 Inquiry 程序的特殊方法，是利用某些特定頻道(ISM 頻段79個頻道中的32頻道)來廣播 ID 封包。

Inquiry 呼叫程序一開始是由 Master 裝置進入 Inquiry 狀態，準備詢問是否有新的裝置欲加入 Piconet 網路；同時某些裝置啟動電源，也準備加入網路運作而進入 Inquiry Scan 狀態，聆聽廣播ID封包。

接下來，Master 裝置在特殊頻道上發送 ID 封包，該封包含著 Inquiry Access Code；相對地，Slave 裝置也由特殊頻道上收到 ID 封包，知道這是 Master 邀請加入網路的查詢訊號，便進入 Inquiry Response 狀態，並於隨後發送『跳頻同步封包』（FHS 封包）給 Master 裝置，FHS 封包內包含著 Slave 裝置本身的 BD_ADDR（Bluetooth Device Address）、原始時序（Native Clock）與裝置類別（Class of Device）訊息。由於同一時間可能有許多裝置被叫醒（Wake-up），所以在此採用隨機時間，以減少回應 FHS 封包時碰撞的機會，如果某一裝置的 FHS 封包和其它裝置碰撞，則必須回到 Inquiry Scan 狀態，等待下一個 ID 封包。

Master 裝置也許會在收到一個以上的 FHS 封包後，便進入 Page 狀態，執行下一個 Page 程序。同樣地，Slave 裝置在成功發送 FHS 封包後，也會進入 Page Scan 狀態。這裡特別要注意的是，Master 裝置是週期性的進入 Inquiry 狀態，詢問是否有新的裝置欲加入網路，因此所接收到個別裝置的 FHS 封包也會累積起來。
### paging 
在呼叫Paging時，同樣是使用到 ISM 頻段上的 32 個特殊頻道。Master 裝置進入 Page 狀態後，可能已收到若干個裝置回應的 FHS 封包，此時必須分別針對每一個裝置做 Paging 的處理。Master 裝置針對每一 Slave 的 BD_ADDR 來計算『翻頁跳頻序列』（Page Frequency-Hopping Sequence），並以 ID 封包回應給 Slave 裝置，ID 封包內包含著 Slave DAC（Device Access Code）的 LAP 位址。

Slave 裝置在 Page Scan 狀態下收到 Master 的 ID 封包後，便進入 Slave Response 狀態，同時也以翻頁跳頻序列回應相同的 DAC ID 封包給 Master 裝置。接下來，Master 裝置收到 Slave 的 DAC ID 封包後進入 Master Response 狀態，利用下一個 Master-to-Slave 時槽發送一個 FHS 封包給 Slave，此 FHS 封包包含有 Master 裝置的 BD_ADDR、以及即時的 Bluetooth 時序的值。緊接著，Slave 裝置回應一個 DAC ID 封包給 Master，表示有收到 FHS 封包。而 Master 也回應一個 ID 封包給 Slave，表示歡迎加入 Piconet 行列。

從此 Slave 擁有 Master 裝置的 BD_ADDR 與 CLKN，可以計算出 Master 的跳頻順序以及時序的同步，藉此進入連線狀態（Connection State）。同時 Master 也分配一個 AM_ADDR 位址給該 Slave，爾後便利用此位址來通訊，此裝置也正式成為 Piconet 的成員。然而 Master 裝置會連續向欲加入網路的成員做 Page Procedure，一直到全部完成或逾時，才回到連結狀態。

File transfer

Dial up network

LAN access

Synchronization

## Handover
![](https://i.imgur.com/fk7VG6P.png)
### DAD
### Optmistic DAD
### Fast DAD



## 3GPP revolution
>GSM

>UMTS

>    LTE
    >>System Architecture Evolution
    >>UE, eNodeB, MME, S-GW, P-GW, EPC
>>
>>


## coding technology
### orthogonal code
>Two codes are said to be orthogonal if when they are multiplied together the result is added over a period of time they sum to zero.
>For example a codes 1 -1 -1 1 and 1 -1 1 -1 when multiplied together give 1 1 -1 -1 which gives the sum zero.
### TDMA
>TDMA技術是將每個發言者發言的時間錯開。也就是同一時間只能有一個人發言，如此發言者的聲音便不會受到干擾，並可清楚的傳達。

### FDMA
>FDMA的方式是在頻率上直接切割，將全數頻寬切成每個等寬頻帶的通道，每個通道可供一個用戶使用。 

### CDMA
>舉例，在一個大房間內，有許多人正在交談，TDM的方式就是所有的人集中在房子中間，但是大家輪流交談，也就是先和某人說完後，再和另一人交談；FDM則是將人群分成好幾團，每一團同時之間都有自己的對話，但團與團之間是獨立開來的。CDMA則是讓大家集中在一起聊天，但每一對所說的語言都不同，說法語的這一對只認定法語，並將其他聲音視為雜訊。因此CDMA主要重點在於能夠萃取出自己想要的訊號，並將任何其他東西視為隨機雜訊。

### OFDMA
>簡單來說就是TDMA+FDMA
>基於OFDM數位調變技術而生的multi-user演進版本。OFDMA更將通道再細分為數個subcarrier，讓不同用戶的數據可以被同時傳輸。而且這些被切割的資料封包可以被分批傳送，無需等待所有封包到齊一並傳送。在OFDMA技術下，當用戶下載資料時，路由器會使用不同的subcarrier去傳輸資料給不同的用戶，藉此降低以往同時傳輸資料時所產生的延遲。這樣彈性且分散封包的方式能有效提升網路連線速度與效率。
>guard band 
>在無線環境中，最常遇到的問題是多重路徑延遲擴散(Multi-path Delay Spread)而產生符元間干擾(Inter Symbol Interference, ISI)，造成接收端接收到的訊號品質變差及錯誤率提高，於是為了降低此干擾，在每一個OFDM符元前加上一小段Guard Interval。
![](https://i.imgur.com/05YR3qK.png)

### SC-FDMA
SC-FDMA是Single-carrier，与OFDMA相比之下具有的较低的PAPR（peak-to-average power ratio），比多载波的PAPR低1-3dB左右（PAPR是由多载波在频域叠加引起）。更低的PAPR可以使移动终端（mobile terminal）在发送功效方面得到更大的好处，并进而延长电池使用时间。
### HSS
>

### DSSS
>
### EPC
演進式數據封包核心, 是一種基於 IP 網路協定的核心網路基礎架構，提供數據封包服務來支援已授權 (2G/3G/4G) 和未授權 (Wi-Fi*) 射頻技術之融合。其中三個核心元素是行動管理裝置 (Mobility Management Entity, **MME**)、服務閘道 (Serving Gateway, **SGW**) 以及數據封包網路(Packet Data Network Gateway, **PGW**)。

### E-UTRAN
屬於3GPP LTE 的空中介面，目前是 3GPP 的第八版本。與 HSPA 不同的是，LTE 的 E-UTRA 係一全新的系統，絕不相容於W-CDMA。它提供了更高的傳輸速率，低延遲和最佳化數據包的能力，用OFDMA無線接入給下行連接，SC-FDMA給上行連接。 
#### downlink physical layer procedure:
* cell search and synchronization
* scheduling
* link adaption
* hybrid ARQ

#### uplink physical layer procedure:
* random access
* uplink scheduling 
* uplink link adaption
* uplink timing control 
* hybrid ARQ

#### eNodeB
而UE所連接的基地台設備稱為E-UTRAN NodeB (eNodeB or eNB)，eNB之間可以透過X2介面互相連接。eNB透過S1介面連往後端核心網路(Core  Network或稱CN)
EPC(Evolved Packet Core)則包括P-GW、S-GW及MME(Mobility Management Entity)等元件。其中eNB連接MME設備的介面稱為S1-MME介面，而eNB連接S-GW設備的介面稱為S1-U介面，eNB和eNB之間連接的介面稱為X2介面，UE和eNB之間的界面稱為Uu介面。
![](https://i.imgur.com/eXF5CJz.png)

## 5G
[wiki](https://zh.wikipedia.org/wiki/5G%E6%96%B0%E6%97%A0%E7%BA%BF)
[web intro to 5G: part1](https://www.5g-jump.org.tw/zh-tw/report/content/2)
[web intro to 5G: part2](https://medium.com/@5gnr/5g-nr-en-dc-bearer-concept-291e21b79b38)
[web intro to 5G: part3](https://www.anritsu.com/zh-tw/test-measurement/technologies/5g-everything-connected/5g-world-freq)

為全面部署 5G 行動通訊，全球各國所採用的頻率逐漸明朗化，大致可分為兩種類型。

第一種是**由 3GPP 定義的頻段**，介於 410 MHz 至 7125 MHz，稱為 sub-6 GHz 或 sub-7 GHz 頻段。此頻段為 LTE/LTE-Advanced (LTE-A) 及 WLAN 技術所採用，另有新增的擴展頻段，關於RF特性等技術性問題相對較少；根據所選擇的頻率，其優勢是所用的射頻資源已經過3G (W-CDMA)與4G (LTE/LTE-A)的驗證，但**缺點是多數頻率已經被占用，無法按序保障寬頻段**。

第二種頻段約介於 30 GHz 至 100 GHz；3GPP 將介於 24250 MHz至 52600 MHz的頻率定名為毫米波頻段 (mmWave)，由於這個頻段鮮少使用，**得以確保其寬頻；其具備方便支援高速大容量的資料傳輸優勢**，**缺點則是空中傳輸(OTA) 的訊號大幅衰減**，且由於缺乏行動業者實際使用這個頻帶，因此仍有許多技術性問題尚待釐清。

除了這兩個頻段，另外也分為兩種作業模式：
* 1. **非獨立 (NSA) 模式**，使用結合 New Radio (NR) 技術的 5G 與 LTE/LTE-A 
* 2. **獨立 (SA) 模式**，使用獨有的 5G NR 技術，透過基地台和行動終端機 (UE) 之間的控制功能來收發資料。

### new radio

#### NSA
NSA(Non-Standalone architecture)架構下，將需仰賴**雙連結技術(Dual Connectivity)** 整合LTE與NR兩套不同的無線傳輸技術，其中**LTE eNB**將作為主節點MN(Master Node)，**NR gNB**作為次節點SN (Secondary Node)，因此可稱為**EN-DC**  (LTE-NR Dual Connectivity)。

為有效整合LTE eNB與NR gNB無線傳輸，過去於LTE DC中發展了MCG(Master Cell Group)資料分流乘載(Split bearer)，在**MCG split bearer中以主節點為資料分流點**，並透過X2介面分流至次節點。**然而在EN-DC下以主節點的LTE eNB作為分流點將會大幅增加LTE eNB端的處理能力需求，因此延伸發展了SCG**(Secondary Cell Group)分流乘載，以作為次節點的NR gNB作為資料分流點。相關乘載方式如下圖所示。

![](https://i.imgur.com/pgqmeeR.png)

而NSA 只是一個過渡期，在初期使用NSA 協助5G 盡早開台。等5G架設逐漸完善之後，5G NR就可以慢慢轉換到以 5G 系統為主的「獨立組網（SA）」模式。

#### SA

> **{network function}**:  a functional building block **within a network infrastructure**, which has well-defined external interfaces and a well-defined functional behavior. **In practical terms, a Network Function is today often a network node or physical appliance**.
> **{network entity}**: a hardware unit 

::: info
user plane function
* AMF (~~Access and Mobility Management Function~~)
* SMF (~~Session Management Function~~)
* PCF (~~Policy and Charge Function~~)
* UDM (~~User Data Management~~)
* AUSF (~~Authentication Server Function~~)
:::

::: danger
control plane funtion
* UPF （~~User Plane Function~~）
::: 
![](https://i.imgur.com/xVqGl23.png)
:::info
**3GPP Release 15 features** aim at support *Serviec Based Architecture* & *Network Slicing*: 
* **CUPS** (~~Control and User Plane Separation~~)
在4G EPC中，S/PGW被分解为 -> S/PGW-C+S/PGW-U，以提供高效的服务扩展。
这种分解持续到了5G，UPF仅在用户流量的数据包处理中起作用，而所有其他单点处理则由诸如其他控制平面功能来完成。例如：5G核心中的CUPS架構通過 **SMF**和 **UPF**分发到边缘数据中心，带来了成本节约和可扩展性的优势。
* **Service Based Architecture**
現今的4G核心網路中，我們都固定使用所有的entity去服務user，在5G的核心網路中，會將本來4G之中的MME具有的功能打散，成為許多小小的network function，能讓user連上的時候，選擇需要的function去載入，可以減少latency，也能減少執行的資源
* **Modularize**
* **Network Slicing**
網路切片(Network Slicing)技術可將一個5G實體網路劃分成滿足不同需求的多個虛擬切片，每個網路切片對應不同的需求，切片之間相互不影響。如此一來，網路切片可依客戶需求客製化設計，彈性建立私有網路架構模組(Modularize)。此外，因應各產業技術升級之大量聯網需求，軟體定義網路（SDN）、網路功能虛擬化（NFV）和雲端技術，使網路與其底層物理基礎架構分離開來，因此可以通過編程設計將連結作為服務提供，提高效率。
:::

### 5GC
[EPC->4G](https://medium.com/@RiverChan/%E4%B8%80%E5%88%86%E9%90%98%E5%BF%AB%E9%80%9F%E4%BA%86%E8%A7%A3%E6%89%8B%E6%A9%9F%E5%A6%82%E4%BD%95%E5%82%B3%E9%80%81%E6%AA%94%E6%A1%88-c18cb584519f)
![](https://i.imgur.com/rQrjDFy.png)

[EPC-->5GC](https://www.ithome.com.tw/pr/137446)
5GC 技術將採獨立（SA）模式的 5G NR 部署和虛擬網路架構結合在一起，讓網路業者能針對不同情境，充分利用網路切片（network slicing）功能來分配有限的資源。 
![](https://i.imgur.com/K4i2pWz.png)

> AMF ->access and mobility, from MME functionality from EPC world
::: spoiler
- 終止RAN CP接口（N2）。
- 終止NAS（N1），NAS加密和完整性保護。
- 註冊管理。
- 連接管理。
- 可達性管理。
- 流動性管理。
- 合法攔截（適用於AMF事件和LI系統的接口）。
- 為UE和SMF之間的SM消息提供傳輸。
- 用於路由SM消息的透明代理。
- 接入身份驗證。
- 接入授權。
- 在UE和SMSF之間提供SMS消息的傳輸。
- 安全錨功能（SEAF）。
- 監管服務的定位服務管理。
- 為UE和LMF之間以及RAN和LMF之間的位置服務消息提供傳輸。
- 用於與EPS互通的EPS 承載 ID分配。
- UE移動事件通知。
:::
> UPF ->user packet routing & forwarding
::: spoiler
- 用於RAT內/ RAT間移動性的錨點（適用時）。
- 外部PDU與數據網絡互連的會話點。
- 分組路由和轉發（例如，支持上行鏈路分類器以將業務流路由到數據網絡的實例，支持分支點以支持多宿主PDU會話）。
- 數據包檢查（例如，基於服務數據流模板的應用程式檢測以及從SMF接收的可選PFD）。
- 用戶平面部分策略規則實施，例如門控，重定向，流量轉向）。
- 合法攔截（UP收集）。
- 流量使用報告。
- 用戶平面的QoS處理，例如UL / DL速率實施，DL中的反射QoS標記。
- 上行鏈路流量驗證（SDF到QoS流量映射）。
- 上行鏈路和下行鏈路中的傳輸級分組標記。
- 下行數據包緩衝和下行數據通知觸發。
- 將一個或多個「結束標記」發送和轉發到源NG- RAN節點。
- ARP代理和/或乙太網PDU的IPv6 Neighbor Solicitation Proxying。 UPF通過提供與請求中發送的IP位址相對應的MAC地址來響應ARP和/或IPv6鄰居請求請求。

:::

> SMF ->session management, MME and PGW functionality from EPC world
::: spoiler
- 會話管理，例如會話建立，修改和釋放，包括UPF和AN節點之間的隧道維護。
- UE IP位址分配和管理（包括可選的授權）。
- DHCPv4（伺服器和客戶端）和DHCPv6（伺服器和客戶端）功能。
- ARP代理和/或乙太網PDU的IPv6 Neighbor Solicitation Proxying。 SMF通過提供與請求中發送的IP位址相對應的MAC地址來響應ARP和/或IPv6鄰居請求請求。
- 選擇和控制UP功能，包括控制UPF代理ARP或IPv6鄰居發現，或將所有ARP / IPv6鄰居請求流量轉發到SMF，用於乙太網PDU會話。
- 配置UPF的流量控制，將流量路由到正確的目的地。
- 終止接口到策略控制功能。
- 合法攔截（用於SM事件和LI系統的接口）。
- 收費數據收集和支持計費接口。
- 控制和協調UPF的收費數據收集。
- 終止SM消息的SM部分。
- 下行數據通知。
- AN特定SM信息的發起者，通過AMF通過N2發送到AN。
- 確定會話的SSC模式。
- 漫遊功能：
- 處理本地實施以應用QoS SLA（VPLMN）。
- 計費數據收集和計費接口（VPLMN）。
- 合法攔截（在SM事件的VPLMN和LI系統的接口）。
- 支持與外部DN的交互，以便通過外部DN傳輸PDU會話授權/認證的信令。

:::
> PCF ->policy, rule
::: spoiler

:::
> AUSF ->authentication server
::: spoiler

:::
> NRF ->new to 3GPP
::: spoiler
當UE發送請求到核網的時候，因為network function都被拆開散落在各個地方，不像原本的MME都集中在同一台主機上，直接呼叫就好。所以要知道他們這些network function的所在地(可能是IP)，就得靠NRF去**尋找**。
運作流程大概是：UE透過N1介面連上AMF，再連上NRF讓他辨識UE的要求去啟動所需的NF
:::
> NEF ->new to 3GPP
:::spoiler
網路曝光功能，這是個新鮮名字，但不算什麼新鮮的功能，該功能邏輯網元向外部展現了網路功能的一些能力，主要包含三種能力，監控能力、供給能力、策略/計費能力。其中監控能力主要指對5G系統中UE的特殊事件的監控，並將監控資訊通過NEF向外輸出，例如可以輸出UE位置資訊，接續性，漫遊狀態，連線保持性等；供給能力指外部實體可以通過NEF提供資訊以供5G系統UE使用，這些資訊可以包括移動性管理以及會話管理資訊，例如週期通訊時間，通訊持續時間，排程通訊時間；策略/計費能力指外部實體通過NEF傳遞需求用來處理QoS以及計費策略，這些資訊可以用來進行UE會話的某些特殊QoS/優先順序處理，同時也可以設定合適的計費實體或者計費率。如此看來，NEF網元是網路內部與外部實體進行資訊雙向互動的介面網元，同時也是內部資訊分發彙總的邏輯網元。總體來說，這是一個**內外部網元資訊安全傳遞的節點**，可以對不同應用功能網元資訊進行鑑權，授權和限流，總體來說這有點像核心網管，以及提供計費策略的boss經分功能。
:::
> UDM ->Authentication and Key Agreement
:::spoiler
統一的資料管理，由兩部分構成，一部分叫應用前端（FE），另一部分叫使用者資料倉庫（UDR）。從統一資料管理的框架來看，UDM有兩個前端，一個叫UDM-FE，負責處理信用評級，位置管理，訂閱管理等。UDM-FE可以訪問儲存在UDR中的訂閱使用者資訊，同時可以支援如下功能:
* 1.鑑權信用處理；
* 2.使用者標識處理；
* 3.訪問授權；
* 4.註冊/移動性管理；
* 5.訂閱管理；
* 6.短訊息管理；
:::
![](https://i.imgur.com/c4ykfJS.png)

some [feature of 5GC](https://www.sdnlab.com/24286.html)
#### 5G access network
> PHY layer
> > PRACH preamble 
> > faster speed, need to lager gap length
> > ![](https://i.imgur.com/4V4uAA3.png)

> MAC layer
> RLC layer
> PDCP layer -- user plane
> PDCP layer -- control plane
> RRC layer -- control plane
> NAS layer

![](https://i.imgur.com/1wn8ipo.png)

5G NR的CSI-RS參考訊號相比LTE系統中的CSI-RS參考訊號內涵更加豐富，主要功能分為兩大類:
* 其中一大類是為了輔助接收下行**PDSCH**共享通道，主要涵蓋如下三種作用，~~時頻域的追蹤定位~~、~~RRC連線態下物理通道的覆蓋電平RSRP計算~~以及基於~~波束級別的移動性測量~~；
* 另外一大類的作用就和LTE系統中所定義的CSI-RS類似，~~對下行通道質量進行測量~~並進行通道狀態上報以供基站進行鏈路自適應調整。
#### Radio physical layer
Within the Decode Settings parameter group, a decode mode parameter appears for each decodeable physical channel. The physical channels shown depend on the DL-BWP/UL-BWP selection. For **downlink (DL-BWP)**, you can decode the PBCH, PDCCH and PDSCH. For **uplink (UL-BWP)**, you can decode the PUCCH and PUSCH.
:a: DL-BWP
> PDSCH(DL shared channel)
> PBCH (broadcast channel)
> PDCCH(DL control channel)

:b: UL-BWP
> PUSCH(UL shared channel)
>>used to transmit one or two transport blocks
>
> PUCCH(UL control channel)
>>used to control uplink control info(UCI) like: HARQ-ACK, 
> 
> PRACH(random access channel)
>> 
:::success
Beamforming 
利用訊號處理的技術結合多根天線產生一個具有指向性的波束，將波束集中在希望傳播的方向，可以提升該方向訊號的能量並減少對其他方向用戶的干擾。
![](https://www.2cm.com.tw//2cm/img/cms/0/0/1/2020-08/202008061119051131597545.jpg)

假設半波偶極天線的根數N=6，指向𝜙=60°，天線的間距分別是(a)d=λ/4、(b)d=λ/2、(c)d=λ及(d)d=3λ/2。圖4為不同間距的結果，只往前看，也就是說只看𝜙=0°到𝜙=180°的場型，即圖形的上半部。
圖4(a)為間隔d=λ/4的結果，可以看到指向𝜙=60°的主波束太寬，也就是空間解析度不是很好。
圖4(c)為間隔d=λ的結果，可以看到指向𝜙=60°的主波束很窄，空間解析度很好，但是會產生旁瓣輻射(Grating Lobe)的效應。
圖4(b)，其間隔d=λ/2，沒有旁瓣輻射(Grating Lobe)的效應，空間解析度還算恰當，這也是大家選擇間隔d=λ/2的原因。
:::

![](https://i.imgur.com/tOLV814.png)

![](https://i.imgur.com/QrWcVSh.png)
from [3GPP documento](https://kknews.cc/news/vkg825y.html)
![](https://i.imgur.com/dRYf3Z4.png)
