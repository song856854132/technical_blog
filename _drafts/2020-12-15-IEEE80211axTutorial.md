# A Tutorial on IEEE 802.11ax High Efficiency WLANs

## <font color="orange">802.11ax at a glance</font>
### A. Before 802.11ax: 
In the last 20 years, and specifically 802.11a/b/g/n/ac (we restrict to the ones focusing on the “traditional” ISM 2.4 and 5 GHz bands), have been proposed to improve the nominal data rate.
**802.11a/b/g**, “simply” introduce new modulation and coding schemes so as to bring the data rate from the original 2 Mbit/s of the “legacy” 802.11-1997
up to 54 Mbit/s in both the 2.4 GHz (802.11g) and the 5 GHz (802.11a) ISM unlicensed bands.
**The 802.11n** proposal represents a significant step forward to the above early Wi-Fi standards. Data rates significantly increased: 
1. the ability to exploit channels with a width of 40 MHz, which is twice larger than those used in previous 802.11 PHYs. 
2. the usage of higher 5/6 coding rates opposed to the previous 3/4 coding rates, and — arguably the most notable 802.11n breakthrough 
3. the transition towards MIMO technology, i.e., the usage of multiple antennas to transmit up to 4 spatial streams simultaneously between a pair of devices, hence significantly increasing data rates.

**802.11n** also provides several crucial improvements at the MAC layer:
1. Reduced InterFrame Space (RIFS) of 2 µs which can be used instead of the 10 or 16 µs 
2. Short InterFrame Space (SIFS) to separate transmissions of the same STA

From the very beginning IEEE 802.11 tried to add various **contention-free channel** access mechanisms to the standard: 
1. Both the “historical” **PCF** and the subsequent **HCCA** allow an AP to access the channel without contention. 
2. Channel access coordination is accomplished by introducing an Interframe Space called **PIFS** which shorter than the **DIFS** used by the remaining STAs, permits the AP to acquire the channel access without any contention, so as to transmit data or poll the STAs and grant them channel access. 

<font color = "red">In practice, contention-free access techniques have seen a very marginal deployment, especially because of their inefficiency in scenarios when several APs work in the same area.</font> Indeed, if several APs use PIFS, their transmissions will start simultaneously and collide. ~~This problem is partially addressed in the HCCA TXOP Negotiation mechanism introduced in 802.11aa.~~ The mechanism allows various APs to use different time intervals for transmission. <font color="red">Unfortunately, HCCA TXOP Negotiation can only avoid collisions between APs which can communicate with each other, it does not reduce the collision probabilities between an AP and the alien STAs, which still can use random access.</font>
The IEEE 802.11 also has historically put a significant effort to improve the **QoS** in Wi-Fi networks. Specifically, the **802.11e** amendment introduces **EDCA** and **HCCA** which distinguish voice, video, best effort and background traffic and serve them differently.
1. **EDCA** just assigns different priorities to these types of traffic. 
2. the sophisticated **HCCA** allows an AP to schedule transmissions taking into account specific QoS requirements, like the delay bound, the packet loss ratio, or the required bandwidth. 

<font color = "red">However, determining exact requirements is a non trivial task, and arguably another key reason behind the scarce deployment of the contention-free HCCA </font>

For many devices which use Wi-Fi power consumption is an important issue. In **802.11**, power management is based on alternating between two states: *awake* and *doze*. Many amendments introduce new powersaving features, but most of them are related to switching off the radio for a rather long time(for hundreds of milliseconds or even for seconds). Some of them require a PS STA to contend for the channel if it wants to retrieve data from the AP. <font color="red">Such methods are inefficient in dense environments because of collisions, huge overhead and large delays. Some other methods allow an AP and a PS STA to schedule a series of times when the STA retrieves data from the AP</font> 
### B. Main Features of 802.11ax
To improve the nominal bit rates, 802.11ax contains a new PHY protocol with higher modulation and coding schemes:
1.  In contrast to 802.11ac, **802.11ax** does not increase the number of the MIMO spatial streams and does not widen the channel. Thus the nominal data rates are increased up to 9.6 Gbps, which is just 37% higher than that of 802.11ac.
2.  The key feature of 802.11ax is the adoption of an **OFDMA** approach. According to the latest TGax investigations,<font color="red"> OFDMA provides a 6 times higher throughput than legacy DCF.</font>
3.  BSS coloring, Network Allocation Vector (NAV), virtualization, microsleep operation, opportunistic power save.
![](https://i.imgur.com/UF5ee4p.png)


## <font color="yellow">Pysical layer: MODULATION AND FRAME FORMAT</font>
### A. Modulation
1. To **increase the number of tones**, which is favorable for OFDMA, TGax has quadrupled the duration of the OFDM symbols used for the PHY payload up to 12.8 μs.Moreover, longer symbols permit to **reduce the overhead due to Guard Intervals** (GI),which allows the reduction of overhead down to 6%, opposed to the 12–25% GI overhead in the 802.11ac standard.
2. The 802.11ax amendment also introduces new modulation techniques in addition to legacy BPSK(16-QAM, 64-QAM, and 256-QAM). The first one is an optional 1024-QAM, which may be exploited in indoor scenarios with very good channel conditions - i.e., a high SINR. 
3. Additionally, the 802.11ax amendment describes an optional Dual Carrier Modulation (DCM). DCM enhances transmission robustness by allocating the same signal on a pair of tones, which are separated far apart in the frequency domain.
### B. PHY Frame Format
TGax defines 4 types of PHY frames(PPDU, PHY Protocol Data Unit):
1. for **Single User**(SU) transmission 
2. for **extend range SU** transmission
3. for **DL** MU transmission
4. for **UL** MU transmission

To simplify the 802.11ax frame detection in case of high interference, the HE part of the preamble starts with a rep- etition of the L-SIG field, which is followed by the mandatory HE-SIG-A field, an optional HE-SIG-B field and training fields (HE-STF and HE-LTF) needed for tuning MIMO.
### C. Open PHY Issue
Having extended the set of possible data rates, the amendment also adds new degrees of freedom — such as **DCM** and **shorter GIs** — which affect the transmission rate and the reliability. <font color="red">A wide palette of 802.11ax options increases the time needed to obtain statistics</font>. Moreover, in 802.11ax <font color="red">dense networks, every 20MHz sub-band may have its own level of interference.</font> Thus, the best rate may be different for various sub-bands. In 802.11ax networks, the AP not only selects an appropriate rate for its own transmission, but also for the UL MU transmission. For that, it collects reports on signal strength from associated devices prior to allocating UL channel time to them.
Another issue is that the <font color="red">802.11ax PHY preamble is longer than the legacy one</font>. Thus it should be used only for long transmissions which benefit from the new 802.11ax features. 
## <font color="green">MU transmission & channel access</font> 
### A. 802.11ax OFDMA Fundamentals
In 802.11ax, the channel resources are allocated over time and frequency, but in order to simplify resource management and device operation, and to retain compatibility with legacy devices, the OFDMA transmission is organized on a per-frame basis. This means that <font color="red">a frame can carry information from or to multiple STAs. In such a frame, various tones are assigned to different STAs</font> but the duration of all the RUs within such a frame is the same.
Thanks to **MU-MIMO**, up to eight users can be assigned to an RU. It is also possible to allocate up to four spatial streams per user, if the total number of spatial streams does not exceed eight.
* In the case of the **DL** OFDMA transmission, the HE-SIG-B field of the common preamble contains an RU allocation map which is followed by per-user content fields indicating the RUs assigned to an STA and the transmission parameters to be used by the STA(NSTS, MCS, coding, etc). ~~Note that an RU can represent either an SU or an MU-MIMO allocation.~~ 
* <font color="red">Organizing the **UL** MU transmission is a more challenging task.</font> For UL MU OFDMA transmissions, the AP shall receive signals from different STAs at almost the same power level. For that, 802.11ax defines a **power pre-correction mechanism**, according to which the AP indicates in the Trigger frame its current transmit power and the target signal strength that the AP is expected to receive from a STA in the following UL transmission. Thus, having known the AP’s transmit power and the signal strength of the received Trigger frame, the STA can estimate the path loss to the AP and it can calculate an appropriate transmit power for the following UL transmission.
![](https://i.imgur.com/SsafNwX.png)

### B. Performance Improvements
802.11ax also allows performing a **UL** MU transmission just after a DL MU transmission, which can be useful.
### C. Special Trigger Frames
OFDMA permits to cope with frequency selective interference by assigning the best subcarriers for STAs. Apart from that, it reduces the overhead caused by backoffs(interframe spaces) preambles and PHY headers(~~which carry common information for all the STAs in case of a DL transmission~~). <font color="red">In addition to the basic Trigger frame for data and management frames, 802.11ax has special Trigger frames which initiate parallel RTS/CTS handshakes, request block acknowledgments from a group of STAs, and collect beamforming reports or buffer status reports (BSR).</font>
### D. UL OFDMA Random Access
Besides the scheduled UL MU access described above, TGax has designed an optional mechanism which allows performing random UL OFDMA transmissions, which is important when the AP does not know which associated STAs have data to transmit, or when an unassociated STA wants to transmit an association request.
The designed random access is similar to the **multichannel slotted Aloha**. Specifically, a Trigger frame can allocate some RUs for random access.
### E. EDCA Improvements
<font color="red">In 802.11ax networks, **OFDMA works on** ~~top of the legacy CSMA/CA mechanism called ~~**EDCA** or DCF</font>. It means that to transmit a Trigger frame, the AP shall contend for the channel with other STAs. Since the number of STAs is usually much higher than one, the AP rarely wins the contention if the AP uses the same channel access parameters. As shown in Section II-B, OFDMA is much more efficient than EDCA. So, to achieve higher throughput, the STAs should rarely access the channel with EDCA but they should almost always use OFDMA. In other words, the AP shall almost always win the contention.
### F. Open MU & Channel Access Issues
Having introduced OFDMA, Wi-Fi developers made Wi-Fi similar to LTE.However, resource allocation in Wi-Fi is much more difficult than in LTE for the following reasons.
1. Traditional **LTE** networks operate in license bands. This means that an operator can control interference from neighboring cells and adjust inter-cell interference to achieve better performance. In contrast, <font color="red">**Wi-Fi** networks operate in license-exempt bands where nobody can guarantee the interference level in future. This complicates channel quality estimation and makes Wi-Fi developers design sophisticated algorithms to reduce interference</font>.
2. In LTE networks the channel is divided into resource blocks of equal size. For the **downlink channel**, the base station can select an arbitrary subset of resource blocks to transmit some data for a user. For the uplink, the resource blocks in the subset need to be contiguous. <font color="red">In Wi-Fi, the restrictions on possible RUs are more sophisticated</font>, which complicates the development of optimal schedulers, i.e., algorithms which allocate RUs for each STA in order to maximize some utility function.
3. <font color="red">For UL transmissions, Wi-Fi allows the increase of the power spectral density if the STA transmits in a narrow RU.</font> Specifically, the STA can transmit with the same power whatever RU it uses. The first issue is that according to the standard the <font color="red">highest MCSs cannot be used with 26-tone RUs</font>. Thus by splitting the channel into too narrow RUs we may obtain a lower throughput. The second one is the <font color="red">impossibility of splitting some channels into a given number of RUs</font>.
4. a portion of RUs shall be allocated for the RA. Obviously, the number of RUs allocated for the RA affects the latency and the network capacity and shall be selected based on some estimation of the traffic patterns.
5. a Wi-Fi network consists of devices produced by various manufacturers. In the legacy Wi-Fi, all the STAs in the network should use the same channel access parameters broadcast by the AP. Thus, all the devices have the same opportunity to transmit. In an 802.11ax network, the channel resources are allocated by the AP. So a misbehaving AP can allocate more channel time to those STAs which are produced by the same vendor. The methods of detecting such misbehaving APs should be a subject of further investigation.
6. <font color="red">how to select an appropriate duration of an MU frame.</font> This may affect the efficiency of the channel usage as well as the fairness and the QoS. Moreover, an AP shall find a trade-off between long frames favorable for heavy data traffic and short frames efficient for random access and for BSR.
## <font color="blue">overlappign BSS management and spatial reuse</font>
### A. BSS color
To determine which BSS is the originator of a frame without decoding the entire frame, 802.11ax uses the non-unique ID of the BSS, called the BSS color, which is transmitted in the frame preamble. An example shows as follow, black channel36 wants to deliever data to black36. In legacy 802.11ac, sender will stop transmission because interfere by green or puple since they are busy. With BSS color, black channel36 will transmitt directly to  black channel36.
![](https://i.imgur.com/Vp6agAl.png)

### B. two NAVS
The Wi-Fi channel access follows the listen-before-talk principle(**CSMA/CA**, a STA performs carrier sensing before transmitting a frame). The virtual carrier sensing in Wi-Fi, called NAV, it indicate buzyness of the channel. 
In the legacy Wi-Fi, STAs do not take into account by which frame the NAV value was set. However, this may lead to the following misbehavior: while the STA receives a CF-End frame coming from an **Overlapping BSS** (OBSS), the STA will reset the NAV and it will not consider the medium to be virtually busy anymore. Thus, to prevent resetting NAV by CF-End from an OBSS, **802.11ax STAs will support two NAVs: one for their own BSS and the other for all the OBSSs, and they will modify the NAVs separately**.
### C. Quiet Time Period
802.11ax amendment defines the **Quiet Time Period** (QTP) mechanism. It allows a STA to request the AP for a QTP, if the AP satisfies the request, it disseminates information about the reserved QTP and forbids the other STAs to access the channel during QTP. 
### D. Adjustment of Sensitivity Threshold and Transmit Power
**Dynamic Sensitivity Control** (DSC), dynamic adjustment of the carrier sensing threshold, which determines when the STA considers the medium to be busy.
![](https://i.imgur.com/EMV0tKe.png)

The results show above: the increase of these metrics observed with DSC improve the gain in throughput, and fairness is achieved at the cost of a higher number of hidden nodes and, a higher PER. It is natural to think that DSC may decrease fairness. However, DSC reduces the number of exposed nodes which allows the achievement of a gain in fairness. 
### E. Channel Bonding and Preamble Puncturing
To improve the efficiency of channel bonding in dense environment, 802.11ax introduces a new optional feature called preamble puncturing. For an MU OFDMA transmission in a channel greater than or equal to 80 MHz, one or more busy 20 MHz subchannels can be punctured. It means that frame preamle is not transmitted and RUs are not allocated in these subchannels, which bring more flexibility to use channel in dense network.
### F. Vertualization 
One of the widespread features in modern APs is the support for multiple **“virtual” APs** (VAPs). This means that a single physical device can create multiple independent BSSs, reaching up to 32 VAPs in some equipment.
This may be useful, for example: one wants to separate a guest Wi-Fi network from an internal corporate network without installing an additional AP. 
### G. Load Balancing
In dense networks, load balancing is an important problem. But in this paper hadn't discuss more detail in it, my guess is that 802.11ax load balancing feature occupy the legacy IEEE802 familiy rahter than develope a new one. 

## <font color="purple">power management</font> 
### A. Legacy Power Management
In 802.11 networks, power management is based on alternating between two states: **awake** and **doze**. **In the awake state**, an STA can transmit and receive frames, while **in the doze state**, its radio is switched off. To notify the PS STAs about the buffered packets, the AP includes a **Traffic Indication Map (TIM) in beacons**. **STAs then decide to doze or awake**, depend on whether there are buffered packets are destined to itself.
### B. Micro-sleep
The microsleep approach was introduced in 802.11ac. In 802.11ac, the PHY header contains the Partial AID which indicates the transmitter and the receiver(s) of a frame. 802.11ax extends this idea by allowing an STA to doze during UL transmissions or the TXOP of another STA in the same BSS. 
### C. TWT
In order to minimize the contention between STAs and to reduce power consumption, TGax adapted the TWT mechanism introduced in 802.11ah. TWT requesting STA can doze always except during the TWT SP intervals. In particular, having established TWT SPs with the AP, the STA is not required to wake up even for beacons, which can significantly reduce energy consumption.
### D. Opportunistic Power Save
The OPS mechanism allows an AP to split a beacon interval into several subintervals. This mechanism is based on the joint usage of TWT and the legacy TIM element.  In OPS, TIM is transmitted by the AP together with the broadcast TWT SP advertisement at the beginning of the TWT SP.~~(TIM is used in legacy power management mechanisms to indicate the set of STAs for which the AP has buffered data)~~


## <font color="grey">My conclusion</font> 
<font color="red">802.11ax is kind of similar to 5G, they both use OFDMA, and support MU-MIMO. And they all focus on same situation like: high density network, high efficiency on transmission or power saving.</font> As the competitor, 3GPP and IEEE has begun closer and closer, which we can observe in 5G and wifi6. It implies that: the network developement has proceed to the certain level and confronted with the same engineering delima. Convergent result may indicate a new resolution of combing two protocal into a new technology <font color="red">(my bold assumption). As vertual network and software define network become recent trend, upper layer section might be concluded into one uniform protocal one day. After that, a large scale, compatible, highly dense network will come out, using same mechanism such as OFDMA, MU-MIMO .etc. while different infrastructure or base station used in the different network domain.</font> 

###### tags: `IEEE`


