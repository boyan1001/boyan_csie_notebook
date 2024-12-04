# 計算機網路: Ch3 Transport Layer  
## Transport-Layer Serices  
- Provide **logical communication** between application processes running on **different hosts**  
- Actions:  
    - **Sender:**  
        - breaks application messages into **segments**  
    - **Receiver**:  
        - reassembles segmens into messages  

### Transport protocals:  

| TCP                          | UDP                          |
| ---------------------------- | ---------------------------- |
| Reliable                     | Unreliable                   |
| **congestion control**       | No                           |
| **flow control**             | No                           |
| Connection setup             | don't need to connect first  |
| Without delay guarantees     | Without delay guarantees     |
| Without bandwidth guarantees | Without bandwidth guarantees |

## Multiplexing and Demultiplexing  
![image](https://hackmd.io/_uploads/By0CSws7Je.png)  

- How demultiplexing work:  
    - host use **IP address** and **port numbers** to direct segment to appropriate socket  

- **TCP/UDP segment format:**  
![image](https://hackmd.io/_uploads/By2XuvoXkl.png)  

- Connectionless demultiplexing (fot UDP)  
    - **UDP socket** must specify:  
        - Destination IP address  
        - Destination port number  
    - Same destination port number **different source IP address(Port number)** -> go to **same soclet** at receiveing host  

![image](https://hackmd.io/_uploads/By905PsQ1g.png)  

- Connection-oriented demultiplexing (for TCP)  
    - **TCP socket** must specify  
        - **Source IP address**  
        - **Source port number**  
        - Destination IP address  
        - Destination port number  
    - Server may support many simultaneous TCP sockets:  
        - Each socket associated with a **different connecting client**  
        - **The number of sockets** = The number of clients + 1 (**welcome socket**)  

![image](https://hackmd.io/_uploads/BklZjDomyg.png)  

- Summary  
    - **UDP**: demultiplexing using **destination port number**  
    - **TCP**: demultiplexing using **4-tuple**: source and destination IP addresses and port numbers  

## UDP  
### Intro  
- **Connectionless**: 
    - No handshaking  
    - Each UDP segment handled **independently** of others  
- **No congestion control**  
- **No flow control**  
- **Application**:  
    - DNS, SNMP, HTTP/3, streaming multimedia apps  
- Small header size:  
![image](https://hackmd.io/_uploads/rk25avsmyx.png)  

### Checksum  
- **Goal**: detect errors in transmitted segment  
![image](https://hackmd.io/_uploads/ry5aTvjQyx.png)  
- **How to work**:  
    - **Sender**:  
        - Compute the sum of segment content  
        - Change it to 1's complement  
        - Send the checksum to receiver  
    - **Receiver**:  
        - Compute checksum of received segment  
        - If $checksum_{\ receiver} \not= checksum_{\ sender}$: error detected  
        - If $checksum_{\ receiver} = checksum_{\ sender}$: **no error detected** (does not mean no error)  
![image](https://hackmd.io/_uploads/rkIDyOiXkl.png)  

- **Flaw**: 
    - Only can detected the error by **one bit incorrect**  
    - It cannot detected **bit flip**  
![image](https://hackmd.io/_uploads/r1fkldsXJl.png)  

## Principles of Reliable Data Transfer (rdt)  
### Principle  
- The channel from sender to receiving is **unreliable**  
![image](https://hackmd.io/_uploads/HyoF-doXyx.png)  

### Interfaces  
- **rdt_send()**:  
    - Called from above  
    - Transfer data to **rdt**  
- **udt_send()**:  
    - Called by **rdt**
    - Transfer data to **unreliable channel**  
- **rdt_rcv()**:  
    - Called when packet arrives on receiver side of channel  
    - Transfer data to **rdt**  
- **deliver_data()**:  
    - Called by **rdt**  
    - Deliver data to **upper layer**  
![image](https://hackmd.io/_uploads/BkSJmuiXJe.png)  

### rdt1.0  
- Circumstance:  
    - **no bit error**  
    - **no loss of packets**  
- FSM:  
    - **Sender side**:  
    ![image](https://hackmd.io/_uploads/B1X0NusQkl.png)  
    - **Receiver side**:  
    ![image](https://hackmd.io/_uploads/ryDUwOo71x.png)  

### rdt2.0  
- Circumstance:  
    - bit error  
    - **no loss of packets**  

- ACKs and NAKs:  
    - **Acknowledgements (ACKs)**: Receiver explicitly tells sender that **packet received OK**  
    - **Negative acknowledgements (NAKs)**: receiver explicitly tells sender that **packet had errors**  
    - sender **retransmits** packet on receipt of **NAK**  

- **Stop and wait**:  
    - Sender sends one packet, then **waits** for receiver response  

- FSM  
    - **Sender side**:  
    ![image](https://hackmd.io/_uploads/B1MitOimkx.png)  
    - **Receiver side**:  
    ![image](https://hackmd.io/_uploads/S1fY5uo7kl.png)  

- **Flaw**:  
    - What happens if **ACKs/NAKs** corrupted?  
    - Sender does not know what happened at receiver  
    - cannot just retransmit: **possible dulicate**  
    - **Solution**: Add **sequence number**  

### rdt2.1  
- Circumstance:  
    - bit error  
    - **no loss of packets**  

- Add the **sequence number** of each packet  

- FSM:  
    - **Sender side**:  
    ![image](https://hackmd.io/_uploads/rJQbCuomJe.png)  
    - **Receiver side**:  
    ![image](https://hackmd.io/_uploads/Bk4HtTs71g.png)  

### rdt2.0  
- Circumstance:  
    - bit error  
    - **no loss of packets**  

- Using **ACKs** only  
    - Duplicate ACK = NAK  

- FSM:  
    - **Sender side**:  
    ![image](https://hackmd.io/_uploads/Bkgj9pomyx.png)  
    - **Receiver side**:  
    ![image](https://hackmd.io/_uploads/rJHOoaimkg.png)  
    
### rdt3.0  
- Circumstance:  
    - bit error  
    - loss of packets  

- **Approach:**  
    - Sender waits **reasonable** amout of time for ACK  
    - Add **timer** in sender, if timeout and without receiving any ACK, sender retransmit the packet.  

- FSM  
    - **Sender side**:  
    ![image](https://hackmd.io/_uploads/HJDG0To71g.png)  
    - **Receiver side**:  
    ![image](https://hackmd.io/_uploads/rJHOoaimkg.png)  
    
- Performance of **rdt3.0**  
    - $U_{\ sender}$: **utilization** - fraction of time sender busy sending  
    - example:  
        - 1 Gbs link, RTT = 30ms, 8000 bit packet  
        - $D_{\ trans}= L / R = 8000\ bits / 10^9\ bits \ per\ sec = 8\  microsecs$  
        - Without pipelining  
        ![image](https://hackmd.io/_uploads/rkcIgRjQkx.png)  
        - Pipelining  
        ![image](https://hackmd.io/_uploads/HyJ_xRoXkl.png)  
 
### Pipelining Control for rdt3.0  
#### Go-Back-N  
![image](https://hackmd.io/_uploads/Hyx3z0i7yl.png)  
- **cumulative ACK**:  
    -  **ACK(n)**: ACKs all pacets $seq\ \#\le n$  
    -  on receiving ACK(n): move window forward to begin at n+1  

- timer for oldest in-flight packet  

- **timeout(n)**: retransmit packet **n** and all higher seq # packets in window  

- **ACK only**: receiver onlu send ACK for correctly-received packet with **highest in-order** seq #  
    - generate **duplicate ACKs**  
    - need only rememver **rcv_base**  
    - If receipt of **out-of-order packet**  
        - re-ACK pkt with **highest in-order** seq #  
        - Discard  
    ![image](https://hackmd.io/_uploads/Sk60N0s7kl.png)  
    ![image](https://hackmd.io/_uploads/rJIsHCjXJe.png)  
    
#### Selective repeat  
- Receiver **individually ACKs** all correctly received packets  
    - **Buffers packets**, as needed, for in-order delivery to uper layer  

![image](https://hackmd.io/_uploads/Bkg_jD27yl.png)  

- **Action**:  
    - Sender:  
        - 每個 packet 都有設置 timer  
        - 當 timeout 沒有收到 ACK -> 重新送出該 packet  
    - Receiver:  
        - 當 out-of-order 時，將成功抵達的 packet 放入 buffer 中  
        - 回傳該封包 seq # 的 ACK  
![image](https://hackmd.io/_uploads/rJM1fdnXyl.png)  

- **Dilemma**:  
    - 當 window size = n 時，如果我們丟失 n 個 packet 的話，receiver window 與 sender window 的 seq # 所對應的 packet 將會不一樣，產生問題  
![image](https://hackmd.io/_uploads/SytK4_nQkg.png)  

## TCP  
### Overview  
- **Point-to-point**:  
    - one sender, one receiver  
- **Reliable, in-order byte steam**:  
    - no **message boundaries**  
- **Full duplex data**:  
    - bi-directional data flow in **same connection**  
    - **MSS**: 
        - maximum segment size  
        - **without header**  
- **Cumulative ACKs**  
- **Pipelining**:  
    - TCP congestion and flow control set window size  
- **Connection-oriented**:  
    - Handshaking initializes sender, receiver state before data exchange  
- **Flow controlled**:  
    - Sender will not overwhelm receiver  

### TCP Segment Structure  
![image](https://hackmd.io/_uploads/SJOq_9h71g.png)  
- **Sequence number ans ACKs**:  
![image](https://hackmd.io/_uploads/HkzEt93Q1g.png)  

### TCP Round Trip Time  
- If timeout too **short**: unnecessary retransmissions  
- If timeout too **long**: slow reaction to segment loss  
- **Estimate RTT**:  
    - **SampleRTT**: measured time from segment transmission until ACK receipt  
    - Use **EWMA (Exponential Weighted Moving Average)**  
    - $EstimateRTT = (1-\alpha )*EstimateRTT + \alpha * SampleRTT,\  \alpha = 0.125$  
    ![image](https://hackmd.io/_uploads/BJvAc5hm1g.png)  
    
- **TimeoutInterval**:  
    - **EstimatedRTT** plus **safety margin**  
    - **DevRTT**: EWMA of SampleRTT deviation from EstimatedRTT  
    - $DevRTT = (1-\beta )*DevRTT + \beta *|SampleRTT-EstimatedRTT|,\ \beta = 0.25$  
    - $TimeoutInterval = EstimatedRTT + 4*DevRTT$  

### Fast Retransmit  
When receipt of **3 duplicate ACKs**, TCP **retransmit** missing segment without waiting for timeout  

![image](https://hackmd.io/_uploads/rJzWaq37Jx.png)  

### Flow control  
- **Goal**: Avoid **TCP socket receiver buffers** overflow  
![image](https://hackmd.io/_uploads/SJjpCq3mJl.png)  

- **Buffering**:  
    - **Receive Buffers**: Typical default is 4096 bytes  
    - **Receive Window**:  
        - received data but **unACKed**  
        - size of receive window = **rwnd**  

    ![image](https://hackmd.io/_uploads/HJFKfinmJe.png)  
    ![image](https://hackmd.io/_uploads/By8o3zaQyl.png)  
- **Sender** limit amout of unACKed data to received **rwnd**  
    - **Reciever** will send the size of rwnd back to **Sender**  

### Connection Management  
- Before exchanging data, **Sender/Receiver** need to handshake  

- **TCP 3-way handshake**:  
    - Use to establish TCP connection  
    ```sequence
    Client->Server: send SYN
    Server-->Client: response SYN + ACK
    Client->Server: send ACK
    ```

- **TCP 4-way  Wavehand**:  
    - Use to **close TCP connection**  
    ```sequence
    Client->Server: send FIN
    Server-->Client: responce ACK
    Server-->Client: send FIN
    Client->Server: responce ACK
    ```
    - **Client send FIN**: Client want to close connection  
    - **Server send FIN**: Server want to close connection  

## Principle of TCP Congestion Control  
- **Congestion**: **too many sources** sending too much data too fast for **network** to handle  

### Scenario 1  
- One router, **infinite** buffers  
- **no retransmission needed**  
![image](https://hackmd.io/_uploads/H1V6dn3Q1l.png)  

### Scenario 2  
- One router, **finite** buffers  
- sender **retransmits** lost, timed_out packet  
![image](https://hackmd.io/_uploads/S1HGt2h7yg.png)  

- Case 1: **perfect knowledge**  
    ![image](https://hackmd.io/_uploads/H1-qhf6m1l.png)  
    - Sender sends only when router buffer available  
    ![image](https://hackmd.io/_uploads/HygS523mJe.png)  
    
- Case 2: **Some perfect knowledge**  
    ![image](https://hackmd.io/_uploads/ByNT3faQJg.png)  
    - packets can be lost due to **full buffers**  
    - sender knows when packet has been dropped
    ![image](https://hackmd.io/_uploads/HkL_hMTmkx.png)  

- Case 3: **Un-needed Duplicates**  
    ![image](https://hackmd.io/_uploads/SkIA6fTXye.png)  
    - Sender does not know whether packet really lost or congestion.  
    - **Un-needed duplicate**  
    ![image](https://hackmd.io/_uploads/S16DRfpQJe.png)  
    
### Scenario 3  
![image](https://hackmd.io/_uploads/BkPjRzp71x.png)  
- Four senders  
- Multi-hop paths  
- Timeout/retransmit  
- If packet dropped, **any upstream transmission capacity** and buffer used for that packet was wasted  
![image](https://hackmd.io/_uploads/ryOyk7aQyx.png)  

### Insight  
| Router | Buffer | Duplicate | Graph |
| -------- | -------- | -------- | ------- |
| 1 | infinite | no |  ![image](https://hackmd.io/_uploads/HygS523mJe.png =75%x)  |
| 1 | finite | no |  ![image](https://hackmd.io/_uploads/SJcp-Xpmkx.png)|
| 1 | finite | yes | ![image](https://hackmd.io/_uploads/SyG1G7TQye.png) |
| 4 | finite | yes | ![image](https://hackmd.io/_uploads/rJieMXT7kx.png) |

## Methods of TCP Congestion Control  


| 分類 | 方法 |
| -------- | -------- |
| Loss-based | TCP AIMD(Reno), TCP Tache, TCP CUBIC|
| Delay-based | BBR, TCP Vegas|

### AIMD (Additive Increase Multiplicative Decrease, Reno)  
- **Approach**: 
    - increase sending rate until packet loss (congestion) occurs  
    - **Additive Increase**:  
        - Increase sending rate(cwnd) by 1 MSS every RTT  
        - Until loss detected  
    - **Multiplicative Decrease**:  
        - Sending rate in half at each loss event  
![image](https://hackmd.io/_uploads/HJvtIm6m1x.png)  

- **TCP sending behavior**:  
![image](https://hackmd.io/_uploads/rJwwiDTQkg.png)  
    - Roughly: send **cwnd** bytes, wait RTT for ACKs, then send more bytes  
    - $TCP\ rate \approx cwnd/RTT$  

- **Flaw**:  
    - It has bad efficiency in **high loss rate network**  

### Tahoe
#### Slow Start  
- Exponential growth  
- **IW** = Initial window  
    - 在 3 handshaking 時已設定  
- $cwnd = cwnd + min(N, MSS)$  
    - N = 收到 ACK 時，之前**尚未被確認的資料**之大小  
    - 防止接收端採用**分段確認(ACK Division)**  
![image](https://hackmd.io/_uploads/H1ecTLQaQJl.png)  

#### Congestion Avoidance  
- Linear growth  
- $cwnd += 1$  

#### Slow Start Threshold (ssthresh)  
- 一個 slow start 轉為 congestion avoidance 的門檻  
    - $cwnd < ssthresh$: Use **slow start**  
    - $cwnd > ssthresh$: Use **congestion avoidance**  
![image](https://hackmd.io/_uploads/HyxtYF6XJl.png)  
- When packet loss happened:  
    - Timeout:  
        - $ssthresh = cwnd/2$  
        - $cwnd = IW$  
        - Slow start  
    - 3 duplicate ACKs:  
        - Fast retransmit 

#### Flaw  
When packet loss, cwnd becomes IW. It is hard to recover to original cwnd  
    
### CUBIC  
![image](https://hackmd.io/_uploads/rygNYm6XJe.png)  
- **Insight**:  
    - $W_{\ max}$: sending rate at which congestion loss was detected  
    - Use **cube function**: after cutting rate, initially ramp to $W_{\ max}$ **faster**, but approach $W_{\ max}$ more **slowly**  

### Delay-based TCP Congestion Control  
- **Concept**: 
    - Keeping sender-to-receiver pipe **just full enough, but no fuller**  
    - Keep bottleneck link busy transmitting, but avoid high delays/buffering  

![image](https://hackmd.io/_uploads/Hy23iD6Xyl.png)  

- **Approach**:  
    - $RTT_{\ min} - minimum\ observed\ RTT$  
    - $Throughput_{\ uncongested} = cwnd/RTT_{\ min}$  
    - When $Throughput_{\ measured} \approx Throughput_{\ uncongested}$:
        - Path not congested  
        - **increase** $cwnd$ linearly  
    - When $Throughput_{\ measured} \ll Throughput_{\ uncongested}$: 
        - Path congested
        - **decrease** $cwnd$ linearly  



### Explicit Congestion Notification (ECN)  
- **Network-assisted** congestion control  
    - **IP**:  
        - Two bits in IP header maked **by network router** to indicate congestion  
        - Congestion indication carried to destination  
    - **TCP**:  
        - Destination **sets ECE bit on ACK segment** to notify sender of congestion  
![image](https://hackmd.io/_uploads/rJP7IOTmke.png)  