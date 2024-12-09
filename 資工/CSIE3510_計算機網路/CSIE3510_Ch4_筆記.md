# 計算機網路: Ch4 Network Layer: Data Plane  

建議在 [HackMD](https://hackmd.io/@Toast1001/B1ipsZGEkl) 上面查看效果會更好哦~  

## Overview  
- **Two key functions**:  
    - **Forwarding**: move packets from a router's input link to appropriate router output link  
    - **Routing**: determine route  

- **Data Plane**:  
    - Local  
    - Use on **forwarding**   
    ![image](https://hackmd.io/_uploads/rkSdFMfEJe.png)  
- **Control plan**:  
    - **Network-wide** logic  
    - Use on **routing**  
    - 2 control-plane approaches  
        - **Traditional routing algorithms**  
            - Implemented in routers  
            - Local view  
            ![image](https://hackmd.io/_uploads/SyJpYGGN1e.png)  
        - **Software-defined networking (SDN)**  
            - Implemented in remote servers  
            - Global view  
            - Remoted controller computes, install **forwarding tables in routers**  
            ![image](https://hackmd.io/_uploads/S1ZfqMGV1x.png)  

- **Network service model**:  
    - For **individual datagram**  
        - Guaranteed delivery  
        - Guaranteed delivery with less than 40 msec delay  
    - For **flow of datagrams**  
        - In-ordeer datagram delivery  
        - Guaranteed minimum bandwidth to flow  
        - Guaranteed stable **packet gap**  
    - **Best-effort service**  
        - No guarantees on:  
            - Successful datagram delivery to destination  
            - Timing or order of delivery  
            - **Bandwidth** available to end-end flow  
        - **Simplicity of mechanism**
            - Allowed Internet to be widely deployed adopted  
            - Sufficient **provisioning of bandwidth** allows performance of real-time applications  
            - Replicated, application-layer distributed services  
            - Congestion control  

## What's inside a router  
### Router architecture overview  
- **Routing processor**:  
    - Control plane  
    - **Software**  
    - Use to **Routing**  
    - Operates in msec timeframe  
- **Switching fabric**:  
    - Data plane  
    - **Hardware**  
    - Use to **forwarding**  
    - Operates in nanosecond timeframe  
![image](https://hackmd.io/_uploads/SypUxNzEyl.png)  

### Input port functions  
- **Physical layer**:  
    - Bit-level reception  
    - Line termination  
- **Link layer**:  
    - Ethernet  
- **Decentralized switching**:  
    - Lookup forwarding queueing  
    - Usingheader field values, lookup output port uswing  
    - **Goal**: complete input port processing at **line speed**  
    - **Input port queueing**: if datagrams arrive faster than forwarding rate into switch fabric  
![image](https://hackmd.io/_uploads/HyJdPaMEyg.png)  

### Switching fabrics  
- Transfer packet from input link to appropriate output link  
- **Switching rate**: rate at which packets can be transfer from inputs to outputs  
![image](https://hackmd.io/_uploads/ryeXTKMNyg.png)  

#### 3 major types of switching fabrics  
![image](https://hackmd.io/_uploads/r1v1AYMNkx.png)  
- **Memory**:  
    - Traditional computers wit switching under direct control of CPU  
    - Packet copied to system's memory  
    - Speed limited by memory bandwidth  
    - **2 bus crossing** per datagram  
    ![image](https://hackmd.io/_uploads/BJoeCFM4Jl.png)  
    ![image](https://hackmd.io/_uploads/S10vE9zNJg.png)  

- **Bus**:  
    - Datagram from input port memory to output port memory via a **shared bus**  
    - **Bus connection**: switching speed limited by **bus bandwidth**  
    ![image](https://hackmd.io/_uploads/HkxIAtM4ke.png)  
    - Use **NFE processor** to replace passing CPU memory  
    ![image](https://hackmd.io/_uploads/BJ6dVcMVJg.png)  
    
- **Interconnection newwork**:  
    - **Multistage switch**: $n\times n$ switch from **multiple stages** of smaller switches  
    ![image](https://hackmd.io/_uploads/B1SUk5GEkx.png)  
    ![image](https://hackmd.io/_uploads/ryE1r9M41l.png)  
    - **Cisco CRS router**:  
        - Basic unit: 8 switching planes  
        - Each plane: 3-stage interconnection newwork  
        - Up to 100's Tbps swithcing capacity  
        ![image](https://hackmd.io/_uploads/BkQZNcfV1x.png)  
    - **Bayon Network**:  
        - **Basic element**:  
            - **Larger** input tag go down  
            ![image](https://hackmd.io/_uploads/H17oS5f4Jx.png)  
        - **Large Bayon Network**:  
            - It can change input port to appropriate output port  
            - Much greater likelihood of **collisions**  
            ![image](https://hackmd.io/_uploads/SyaPU5zEJe.png)  

    - **Batcher Network**:  
        - Use for **sorting**  
        - Sorting can reduce collisions  
        ![image](https://hackmd.io/_uploads/rk2Gw5fVyg.png)  
    
    - **Batcher-Banyan network**:  
        ![image](https://hackmd.io/_uploads/SkLPw9fNyl.png)  
        ![image](https://hackmd.io/_uploads/Byxl9czEJl.png)  

### Queueing  
- **Input port queueing**:  
    - When switch fabric **slower** than input ports combined -> **queueing** may occur at input queues  
    - **Head-of-the-Line (HOL) blocking**:  
        - 前面塞車導致後面無法做 forwarding  
        - 可以透過調整順序解決  
    ![image](https://hackmd.io/_uploads/SJRTR9fEJe.png)  
- **Output port queueing**:  
    - **Buffering** when arrival rate via switch exceeds output line speed  
    -  Datagrams can be lost due to **congestion**, lack of buffer  
    - **Scheduling discipline**  
        - chooses among queued datagrams for transmission  
        - **Priority scheduling**  
    ![image](https://hackmd.io/_uploads/H1LsPTG41e.png)  
    
### Buffering 
#### How to buffering  
- **Recent recommendation**:  
    - $\frac{RTT\cdot C}{\sqrt{N}}$  
    - $C$: link capacity  
    - $N$: the number of flows  
- Too much buffering can increase **delay**  
    - Particularly in home routers  
    - Long RTTs: poor performance for realtime apps  
    - **Delay-based congestion control**: keep bottleneck link just full enough but no fuller  

#### Buffer management  
- **Drop**:  
    - Which packet to add drop when buffers are full  
    - **Tail drop**: drop arriving packet  
    - **Priority drop**: drop on priority basis  
- **Marking**:  
    - Which packets to mark to signal congestion  
    - ECN, RED  
![image](https://hackmd.io/_uploads/H1mcbAGV1e.png)  

### Packet scheduling  


| Methods | Priority | Cycle | Weighted | Queue |
| -------- | -------- | -------- | -----| ------ |
| **FCFS**  | No | No | No | ![image](https://hackmd.io/_uploads/r1wZT0fV1l.png) |
| **Priority**  | Yes | No | No | ![image](https://hackmd.io/_uploads/SkRZpRz4kx.png) |
| **RR**  | Yes | Yes | No | ![image](https://hackmd.io/_uploads/HymzTRGE1e.png) |
| **WFQ**  | Yes | Yes | Yes | ![image](https://hackmd.io/_uploads/By9fTAfV1e.png) |

#### First come, first served(FCFS)  
Packets transmitted in order of arrival to output port  

#### Priority  
- Any header field can be used for classification  
![image](https://hackmd.io/_uploads/Byu-S0zNJe.png)  
- High priority send first  
- **FCFS** within **priority class**  
![image](https://hackmd.io/_uploads/rJRbrAzVye.png)  

#### Round robin(RR)  
- Arriving traffic classified queue by class  
- Server **cyclically**, repeatedly scans class queues  
- Sending one complete packet from each class in turn  
![image](https://hackmd.io/_uploads/BJu5rAME1l.png)  

#### Weighted fait queueing(WFQ)  
- Generalized **Round Robin**
- Each class $i$, has weight, $w_{i}$  
- Weighted amount of service in each cycle = $\frac{w_{i}}{\Sigma _{j} w_{j}}$  
- **Minimum bandwidth guarantee** (per-traffic-class)  
![image](https://hackmd.io/_uploads/Hy7UFCzEyl.png)  

## IP: the Internet Protocol  
### Internet  
- **Path-selection algorithms**:  
    - Routing protocols  
    - SDN controller  
- **IP protocol**:  
    - Datagram format  
    - Addressing  
    - Packet handling conventions  
- **ICMP protocol**:  
    - Error reporting  
    - Router "signaling"  
![image](https://hackmd.io/_uploads/BJccy1QV1l.png)  

### IP Datagram Format  
- **Overhead**:  
    - 20 bytes of TCP  
    - 20 bytes of IP  
![image](https://hackmd.io/_uploads/HkiHQJ7Vkg.png)  

### IP addressing  
#### Introduction  
![image](https://hackmd.io/_uploads/ry2kS1mNye.png)  
- **IP address**: **32-bit** identifier associated with each host or router **interface**  
![image](https://hackmd.io/_uploads/BkElBJmEJg.png)  
- **Interface**: connection between host/router and physical link  
    - **Router**: multiple interfaces  
    - **Host**: 1 or 2 interfaces  

#### Subnets  
- **Definition**:  
    - Device interfaces that can physically reach each other **without paswsing through an intervening router**  
    - Each **isolated** network is called **subnet**  
- **IP addresses structure**:  
    - **Subnet part**:  
        - High order bits  
        - Devices in same subnet have common subnet part  
    - **Host part**:  
        - Remaining low order bits  
![image](https://hackmd.io/_uploads/S1doSJQVkg.png)  

#### Classless InterDomain Routing(CIDR)  
- Subnet portion of address of arbitrary length  
- **Address format**:  
    - $a.b.c.d/x$  
    - $x$ is **the number of bits in subnet portion** of address  
    ![image](https://hackmd.io/_uploads/HybOU1QN1x.png)  
    
#### How to get IP address  
- How does **a host get IP address** within its network(host part of address)?  
    - Hard-coded by **sysadmin** in config file  
    - **Dynamic Host Configuration Protocol(DHCP)**:  
        - Dynamically get address from as server  
        - **plug-and-play**  
        - Dynamic IP address is **not stable**   
- How does a **network** get IP address for itself(network part of address)?  
    - Gets allocated portion of its provider ISP's address space  
    ![image](https://hackmd.io/_uploads/H1aVnlQVkx.png)  

#### Dynamic Host Configuration Protocol(DHCP)  
- **UDP** based  
- **Goal**: host **dynamically** obtain IP address from **network server** when it join network  
    - Can **renew** its lease on addresses  
    - Support for **mobile users** who join/leave network  
    - Allows **reuse** of address  
- **Secnario**:  
![image](https://hackmd.io/_uploads/BkVuUlXVyg.png)  

- **Overview**:  
    ```sequence
    Arriving client->DHCP server: DHCP discover
    DHCP server-->Arriving client: DHCP offer
    Arriving client->DHCP server: DHCP request
    DHCP server-->Arriving client: DHCP ACK
    ```
    - **DHCP discover**:  
        - Client **broadcast** packet to whole subnet  
        - Ask **DHCP server** of the subnet offer IP address  
    - **DHCP offer**:  
        - **DHCP server** send a **unused IP address** to client  
    - **DHCP request**:  
        - Client request to DHCP server that it got the IP address  
    - **DHCP ACK**:  
        - DHCP server talk to client that it can connect to network by the IP address  
    
#### Hierarchical addressing  
- **Goal**: allow **efficient advertisement** of routing information  
![image](https://hackmd.io/_uploads/Sk9_QWQ41e.png)  
- **More specific routes**  
    - **Organization 1** moves from Fly-By-Night-ISP to ISPs-R-Us  
    -  ISPs-R-Us now advertises a **more specific route** to Organization 1  
    ![image](https://hackmd.io/_uploads/BkGMI-7VJg.png)  

#### Internet Corporation for Assigned Names and Numbers(ICANN)  
- **Goal**: let ISP get block of addresses  
- Allocated IP addresses, through 5 **regional registries(RRs)**  
- Manage **DNS root zone**, including **TLD** management  
#### How to get IP address ?  


| Asker | Method |
| -------- | -------- |
| **Host**    | Use **DHCP** |
| **Network**    | Allocated by **ISP** |
| **ISP** | Allocated by **ICANN** |

### Network address translation(NAT)  
- All devices have:  
    - **Public IP address**: use for ouside world  
    - **Private IP address**: use in local network  
![image](https://hackmd.io/_uploads/ryuKu-7NJg.png)  
- **Advantages**:  
    - Just **one** IP address needed from provider ISP for all devices  
    - Can change ISP without changing addreses of devices in local network  
    - **Security**:  
        - Devices inside local network **not directly addressable** by outside world  

- **Implementation**:  
    - **Outgoing datagram**:  
        - Source IP address(LAN) -> NAT IP address(WAN)  
        - Source port number(LAN) -> New new port number(WAN)  
    - **NAT translation table**:  
        - **Remember** every (source IP address, port number) to (NAT IP address, new port number)  
    - **Incoming datagrams**:  
        - NAT IP address(WAN) -> Destination IP address(LAN)  
        - New port number(WAN) -> Destination port number(LAN)  
    ![image](https://hackmd.io/_uploads/S1Jx0bXNkx.png)  

### IPv6  
#### Motivation  
- **Initial motivation**: 32-bit IPv4 address space would be completely allocated  
- **Additional motivation**:  
    - Speed processing and forwarding  
    - Enable different network-layer treatment of flows  

#### IPv6 datagram format  
![image](https://hackmd.io/_uploads/rJJjQM741g.png)  
- Compare with IPv4:  

| Properties | IPv4 | IPv6 |
| -------- | -------- | -------- |
| **Addresses size** | **32-bit** | **128-bit** |
| **Checksum** | Yes | No |
| **Fragmentation** | Yes | No |
| **Reassembly** | Yes | No |
| **Options** | Yes | No |

#### Transition from IPv4 to IPv6  
- Not all router can be upgraded simultaneously  
- **Tunneling**:  
    - **IPv6** datagram carried as **payload** in **IPv4** datagram among IPv4 routers  
    - Packet within a packet  
    ![image](https://hackmd.io/_uploads/rkmfPzmNkg.png)  
    
![image](https://hackmd.io/_uploads/HJeysfXNJl.png)  

## Generalized Forwarding, SDN  
### Match + action  
- **Detination-based forwarding**: forward based on **destination IP address**  
- **Generalized forwarding**:  
    - Many header fields can determine action  
    - Many action possible:
        - Drop  
        - Copy  
        - Modify  
        - Log packet  
    ![image](https://hackmd.io/_uploads/HyMYyQQ4kl.png)  

### Flow table abstraction  
- **Flow**: defined y header field values  
- **Generalized forwarding**:  
    - **Match**: pattern values in packet header fields  
    - **Actions**: for matched packet  
        - Drop  
        - Forward  
        - Modify  
        - Matched  
        - Send matched packet to controller  
    - **Priority**: disambiguate overlapping patterns  
    - **Counters**: number bytes and number packets  
    ![image](https://hackmd.io/_uploads/ryZnm774kl.png)  
 
 ### OpenFlow  
 - **Goal**: 允許用軟體的方式遠端操縱硬體  

 #### Flow table entries  
 ![image](https://hackmd.io/_uploads/By8-X7X4kl.png)  
 
 #### Examples  
 ![image](https://hackmd.io/_uploads/HJkxSQQ4Je.png)  
 ![image](https://hackmd.io/_uploads/HyU4LXm4ye.png)  
 
 #### Abstraction  
 - **Match + action**: abstraction unifies different kinds of devices  

| Devices | Match | Action |
| -------- | -------- | -------- |
| **Router** | Longest destination **IP prefix** | Forward out a link |
| **Switch** | Destination **MAC address** | Forward or flood |
| **Firewall** | IP address, TCP/UDP port number | Permit or deny |
| **NAT** | IP address and port number | Rewrite address and port number |

## Middleboxes  
- **Definition**: Any intermediary box performing functions **apart form normal, stadard function of an IP router** on the data path between a source host and destination host  
![image](https://hackmd.io/_uploads/rJaIxv7Vkx.png)  
- **Initially**: proprietary hardware solutions  
- Move towards **whitebox hardware** implementing open API  
    - Move away from proprietary hardware solution  
    - **Programmable local actions** via match+action  
    - Move towards innovation/differentiation in software  
- **Network functions virtualization(NFV)**:  
    - Programmable services services over white box **networking, computation, storage**  
- **The IP hourglass**:  
![image](https://hackmd.io/_uploads/S1sqEdQEkx.png)  
