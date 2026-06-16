# DCN Chapter 1: Introduction to Computer Networking

This note provides a comprehensive summary of Chapter 1 from *Computer Networking: A Top-Down Approach (8th Edition)*.

## 1. The Internet: Two Perspectives
- **Nuts and Bolts (Architecture View):** A global network of **hosts/end systems** (billions of devices) connected by **communication links** (fiber, copper, radio, satellite) and **packet switches** (routers, link-layer switches).
- **Service-Oriented (Service view):** An infrastructure providing services to distributed applications via a **Application Programming Interface (API)**.

**Protocol:** a set of rules that defines the format, order, and handling of messages exchanged between communicating devices.

## 2. Network Edge: [[Access Networks]]
Q. How to connect end systems to edge router?
ANS: To connect an end system (like your laptop) to an edge router (first router), you use an **Access Network**. 

### Types:
- **DSL (Digital Subscriber Line):** Uses existing telephone lines to a Central Office (CO) DSLAM. **Dedicated** line.
    - data over DSL phone line goes to Internet
    - voice over DSL phone line goes to telephone net
    - Rates: 24-52 Mbps down, 3.5-16 Mbps up.
    -Q. What is *frequency division multiplexing (FDM)*?
ANS: Frequency Division Multiplexing (FDM) is an analog multiplexing technique where the total bandwidth of a shared medium is divided into a series of non-overlapping frequency sub-bands. 
Each sub-band is used to carry a separate signal (channel) simultaneously. To prevent interference between these channels, "guard bands" are used to separate them.


- **Cable:** Uses the coaxial cables originally laid for cable television. **Shared** access network.
    - Rates: 40 Mbps - 1.2 Gbps down, 30-100 Mbps up.
- **FTTH (Fiber to the Home):** Runs fiber-optic cables directly from the central office into private residences, High speed, point-to-point.
- **Wireless Access Networks:**
	- **Wireless LAN (WiFi):** 802.11b/g/n (11, 54, 450 Mbps).
	- **Wide-Area Cellular Access Networks:** 4G/5G (10s of Mbps).
- **Enterprise Networks:** Mix of Ethernet (100Mbps, 1Gbps, 10Gbps) and WiFi.
- **[[Data Centre Networks]]:** Connects hundreds to thousands of servers together and to the Internet. Used by major content providers (Google, Amazon) for high-speed data processing. 

### Summary of hosts sending packets of data

Hosts (end systems) communicate by breaking down large pieces of information (like a photo or a video) into smaller chunks called **packets**.

#### Technical Details
- **Packetization:** The host takes a large message and breaks it into smaller pieces of length $L$ bits.
- **Headers:** The host adds control information (header) to each piece, creating a **packet**. This header contains the destination address and other necessary metadata.
- **Transmission:** The packet is sent into the access network at a transmission rate $R$ (link capacity or bandwidth).
- **Time to Transmit:** The time required to "push" a packet of $L$ bits into a link of rate $R$ bps is:
  $$d_{trans} = \frac{L (bits)}{R (bits/sec)} \text{ seconds}$$

#### Examples
- **Streaming a Movie:** Netflix doesn't send you the whole 2GB movie file at once. It breaks it into millions of tiny packets. Your computer receives these and puts them back together so you can watch without waiting for the full download.
- **Sending an Email:** Your text and attachments are sliced into packets, routed through the internet, and reassembled in your friend's inbox.

### [[Physical Layer#Physical Media|Physical Media]]
Physical media are the actual physical materials used to carry signals. They are categorized into:

- **Guided Media:** Signals follow a solid medium.
    - **Twisted-Pair Copper:** Two insulated wires twisted together (Ethernet, DSL).
    - **Coaxial Cable:** Two concentric conductors. Bidirectional. Supports multiple channels via FDM (Broadband) or a single channel (Baseband). High noise immunity.
    - **Fiber Optics:** Glass fibers carrying light pulses. Extremely high capacity (Gbps), low error rate due to repeaters spaced apart, and immune to electromagnetic noise.
- **Unguided Media:** Signals propagate through the atmosphere.
    - **Terrestrial Microwave:** Line-of-sight links between fixed antennas.
    - **Wireless LAN (Wi-Fi):** Short-range, shared access based on 802.11 standards.
    - **Wide-Area Cellular:** Broad coverage (4G/5G) via cell tower networks.
    - **Satellite:** Includes high-latency Geostationary (GEO) and low-latency Low Earth Orbit (LEO) systems.

## 3. Network Core: Packet vs. Circuit Switching
Q. Human analogies of reserved resources (circuit-switching) vs. on-demand allocation (packet-switching)?
ANS: Circuit-switching is like making a restaurant reservation—the table is reserved for you whether you use it or not. Packet-switching is like a restaurant with no reservations—you only take a table if one is free, and you might have to wait (queue).

### Core networking functions:
1. **Routing:** The global action that determines the end-to-end path taken by packets from source to destination. This is handled by **routing algorithms** that calculate the best routes across multiple routers.
2. **Forwarding (Switching):** The local action within a router that moves a packet from an input link to the appropriate output link. Each router uses a **forwarding table** (created by the routing process) to determine the correct output for a packet's destination.

### Packet Switching
- **Store-and-Forward:** Entire packet of length $L$ bits must arrive at router before it can be transmitted on next link at rate $R$ bps.
- **Queueing and Loss:** If arrival rate exceeds transmission rate, packets queue. If buffer fills, packets are dropped (**lost**).
- **Statistical Multiplexing:** Resources are used as needed (On-demand). It relies on the statistical probability that not every user will transmit data at the exact same millisecond.
- **Efficiency Example:** 1 Gbps link, users active 10% of time, each needs 100 Mbps.
    - **Circuit Switching:** Supports only 10 users.
    - **Packet Switching:** With 35 users, the probability of >10 being active at once is extremely low ($< 0.0004$).
Q. How did we get the value 0.0004?
ANS: It is calculated using the Binomial Distribution formula $P(X > 10) = 1 - P(X \le 10)$, where $n=35$ users and the probability of being active $p=0.1$.

### Circuit Switching
- **Dedicated Resources:** Reserved end-to-end for a "call". No sharing. When two devices want to communicate, the network establishes a **dedicated, end-to-end circuit (path)** between them before any data can be sent. This connection remains open and exclusive to those two devices until the session is terminated.
- **[[Multiplexing]]:**
    - **FDM (Frequency Division Multiplexing):** 
        - The frequency spectrum of a link (link = physical media) is divided into narrow frequency bands. 
        - Each "call" or connection is assigned a dedicated band for the duration of the session.
        - **Guard Bands:** Small unused frequency strips are required between bands to prevent interference (crosstalk).
    ![[Pasted image 20260517122810.png|286]]
    - **TDM (Time Division Multiplexing):** 
        - Time is divided into frames of fixed duration, and each frame is divided into a fixed number of time slots.
        - Each "call" is assigned a specific periodic time slot in every frame.
        - **Synchronization:** Requires precise timing to ensure users transmit exactly within their assigned slots.
    ![[Pasted image 20260517122839.png|368]]

#### Packet Switching vs. Circuit Switching: Which is Better?
The choice between packet and circuit switching depends on the traffic pattern, but **packet switching is the clear winner for modern Internet data**.

| Feature                 | Circuit Switching                                                                | Packet Switching                                                                         |
| :---------------------- | :------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------- |
| **Resource Allocation** | **Dedicated:** Resources are reserved end-to-end for the duration of the "call." | **On-demand:** Resources are shared using statistical multiplexing (dynamic allocation). |
| **Efficiency**          | **Lower:** Resources are wasted if the user is idle.                             | **Higher:** Multiple users can share the same link capacity.                             |
| **Performance**         | **Guaranteed:** Fixed bandwidth and zero congestion delay.                       | **Best-effort:** Potential for variable delay (lagging) and packet loss.                 |
| **Setup**               | **Required:** Call setup phase is necessary before data can be sent.             | **None:** Data can be sent immediately.                                                  |

**Conclusion: Packet Switching is better for the Internet.**
- **Bursty Traffic:** Most data applications are "bursty." Packet switching allows many more users ($N_{packet} \gg N_{circuit}$) to share the same infrastructure.
- **Scalability:** Simpler and more cost-effective to scale.
- **Resilience:** Easier to route around link failures.

Q. How to provide circuit-like behavior with packet-switching?
ANS: This is complex and involves advanced techniques like congestion control, Quality of Service (QoS) guarantees, and resource reservation protocols to mimic the dedicated nature of circuit switching.

## 4. Internet Structure
Q. Given millions of access ISPs ( internet service provider ), how to connect them together?
ANS: Connecting every access ISP to each other directly doesn't scale ($O(N^2)$ connections). Instead, they connect to a smaller number of Tier-1 and regional ISPs in a hierarchical "network of networks" structure.
- **Hierarchy:** Hosts $\rightarrow$ Access ISPs (local)$\rightarrow$ Regional ISPs (national)$\rightarrow$ Tier-1 ISPs (global).
- **Tier-1 ISPs:** National/International coverage (e.g., Level 3, AT&T, Sprint).
- **IXP (Internet Exchange Point):** Where multiple ISPs interconnect.
- **Peering:** Economic agreement for ISPs to connect directly.
- **Content Provider Networks:** (e.g., Google, Microsoft) private networks connecting data centers, often bypassing Tier-1 ISPs to stay "close" to users.

## 5. Performance Metrics
### The Four Sources of Nodal Delay
The total time it takes for a single router to process and forward a packet is called the **nodal delay**.

Total nodal delay $d_{nodal} = d_{proc} + d_{queue} + d_{trans} + d_{prop}$
1. **Nodal Processing delay ($d_{proc}$):** The time the router takes to examine the packet's header and determine where it needs to go next. Check bit errors, determine output link (typically < microseconds).
2. **Queueing Delay ($d_{queue}$):** Time waiting in router's buffer before it can be transmitted onto the link. Duration depends on how congested the network traffic is. 
    - **Traffic Intensity:** $La/R$ (where $L$=bits/packet, $a$=avg arrival rate, $R$=bandwidth).
    - If $La/R \rightarrow 0$: Delay is small.
    - If $La/R \rightarrow 1$: Delay becomes large.
    - If $La/R > 1$: Average delay becomes infinite.
3. **Transmission Delay ($d_{trans}$):** $L/R$ (Time to "push" packet onto link).
4. **Propagation Delay ($d_{prop}$):** $d/s$ (Time for a single bit to travel through the physical medium of distance $d$ at speed $s \approx 2 \times 10^8$ m/sec).
$$d_{\text{prop}} = \frac{d}{s}$$
### Packet queueing delay
Queueing delay depends on the rate at which traffic arrives at the queue, the transmission rate of the link, and the nature of the arriving traffic (periodic or bursty).

#### Technical Details
- **Traffic Intensity:** Calculated as $\frac{La}{R}$, where:
    - $L$ = Length of the packet (bits).
    - $a$ = Average packet arrival rate (packets/sec).
    - $R$ = Transmission rate / Bandwidth (bits/sec).
- **Golden Rule of Traffic Intensity:**
    - If $\frac{L \cdot a}{R} \approx 0$: Average queueing delay is close to zero.
    - If $\frac{L.a}{R} \rightarrow 1$: Average queueing delay grows exponentially as the queue becomes longer.
    - If $\frac{L.a}{R} > 1$: The queue will grow without bound, and the average delay will become infinite (leading to packet loss).
- **Traffic Patterns:**
    - **Periodic Arrival:** If one packet arrives every $L/R$ seconds, there is no queueing delay because each packet is transmitted exactly when the next one arrives.
    - **Bursty Arrival:** If $N$ packets arrive at the same time every $(N \cdot L)/R$ seconds, the first packet has zero delay, but the $N$-th packet suffers a significant delay while waiting for the previous $N-1$ packets to be transmitted.

#### Queueing delay (model)
The mathematical model of average queueing delay is often visualized as a curve that stays near zero for low traffic intensity and shoots toward infinity as intensity approaches 1.
![[Pasted image 20260616214817.png|230]]
- **Formula for Average Delay:** For many queueing models (like M/M/1), the average queueing delay can be approximated as:
  $$d_{queue} = \frac{L/R}{1 - La/R} \text{ (when } La/R < 1)$$
- **Non-Linearity:** Unlike transmission delay (which is linear with respect to packet size), queueing delay is non-linear and highly sensitive to small changes in traffic arrival rates when the link is near capacity.

#### Examples
- **Periodic Example:** A sensor sending exactly one 1000-bit packet every 1ms into a 1Mbps link ($La/R = 1$). Despite high intensity, the delay is zero because of perfect timing.
- **Bursty Example:** 10 students all hitting "Refresh" on a web page at the exact same millisecond. The router's buffer will fill instantly, causing high queueing delay for the last few students, even if the link is idle for the rest of the minute.

### Throughput and Loss
- **Throughput:** Rate (bits/time) between sender/receiver.
    - **Instantaneous:** Rate at a given point in time.
    - **Average:** Rate over a longer period.
- **Packet Loss:** Occurs when a packet arrives at a full buffer.
- **Bottleneck Link:** The link in an end-to-end path that limits the end-to-end throughput.

#### Throughput: Network scenario
Throughput is determined by the "weakest link" in the network path.

**1. End-to-End Scenario (Server to Client):**
- Consider a server sending a large file to a client.
- $R_s$: Transmission rate of the server's access link.
- $R_c$: Transmission rate of the client's access link.
- $R$: Transmission rate of the core network links.
- **Formula:** The end-to-end throughput is $min\{R_s, R_c, R\}$.
- In practice, the core network bandwidth $R$ is much larger than $R_s$ or $R_c$, so the throughput is usually $min\{R_s, R_c\}$.

**2. Shared Bottleneck Scenario:**
- Consider 10 servers and 10 clients all sharing a single core link with capacity $R$.
- Each server/client pair has access links with rates $R_s$ and $R_c$.
- **Formula:** The throughput for each pair is $min\{R_s, R_c, R/10\}$.
- Even if individual access links are fast, the shared capacity of the core link can become the **bottleneck**.

#### Examples
- **Home Download:** You have a 1Gbps fiber connection ($R_c$), but the website you are downloading from only has a 10Mbps upload speed ($R_s$). Your throughput will be limited to 10Mbps ($min\{10, 1000\}$).
- **Public Wi-Fi:** You are at a cafe with 100 people sharing a single 50Mbps internet connection. Even if your phone's Wi-Fi link to the router is 300Mbps, your actual throughput will be much lower because the 50Mbps link is shared.


## 6. Protocol Layers and Service Models
- **Layering:** Organizing network protocols into a hierarchy.
- **Internet Protocol Stack:**
    1. **[[Application Layer]]**: (HTTP, SMTP, DNS) - Providing network services directly to software applications.
    2. **[[Transport Layer]]**: (TCP, UDP) - Process-to-process data transfer.
    3. **[[Network Layer]]**: (IP, routing) - Routing datagrams from source to destination.
    4. **[[Link Layer]]**: (Ethernet, WiFi, PPP) - Data transfer between neighboring network elements.
    5. **[[Physical Layer]]**: (Bits "on the wire") - Moving individual bits within a frame.
- **ISO/OSI (Open Systems Interconnection) Reference Model:** Adds **Presentation (ensures that data sent from the application layer of one system can be read by the application layer of another)** 
- and **Session(handles authentication and reconnection if a line drops)** layers.
- **Encapsulation:** The process of wrapping data from a higher protocol layer into the payload of a lower protocol layer, adding its own header (and sometimes a trailer).
    - **Data Units (PDU):** As data moves down the stack, it takes different names: **Message** (Application) $\rightarrow$ **Segment** (Transport) $\rightarrow$ **Datagram** (Network) $\rightarrow$ **Frame** (Link).
    - **Header Information:** Headers typically contain source/destination addresses, error-checking codes (checksums), and protocol-specific control bits.
    - **Decapsulation:** The reverse process at the destination. Each layer strips off its corresponding header, examines the control information, and passes the remaining payload up to the next layer.

### Problems of Layering
While layering simplifies complex network architectures, it introduces several technical and performance drawbacks:

- **Performance Overhead:** Each layer adds its own header (and sometimes trailer) to the data. This encapsulation increases the packet size, consuming more bandwidth and increasing transmission delay ($L/R$).
- **Duplication of Functionality:** Some functions are repeated at multiple layers. For example, error detection and recovery may be implemented at both the Link Layer (e.g., Ethernet CRC) and the Transport Layer (e.g., TCP checksums and retransmissions).
- **Information Silos (Lack of Transparency):** Layers are designed to be independent, which means a higher layer cannot see or utilize useful information from a lower layer.
    - *Example:* A Transport Layer protocol might not know the exact physical bandwidth available, leading to sub-optimal congestion control decisions.
- **Complexity in Troubleshooting:** When a failure occurs, it can be difficult to determine which layer is responsible, especially when layers interact in unexpected ways.

#### Examples
- **Double Error Recovery:** If a wireless Link Layer tries to retransmit a corrupted packet several times, the Transport Layer (TCP) might time out and start its own retransmission, leading to redundant traffic and wasted resources.
- **Overhead:** In extremely small data transfers (like a single keystroke in Telnet), the overhead of TCP, IP, and Ethernet headers can be many times larger than the actual data being sent.

## 7. Network Security
The Internet was originally designed based on a "group of mutually trusting users," which makes security a retrospective challenge.

### 1. Common Threats
- **Malware (Malicious Software):**
    - **Viruses:** Require some form of user interaction to spread (e.g., opening an email attachment).
    - **Worms:** Self-replicating programs that spread across a network by exploiting software vulnerabilities without user intervention.
    - **Botnets:** A collection of compromised "zombie" computers controlled by a central "botmaster," often used for massive spam or DoS attacks.
- **Denial of Service (DoS & DDoS):**
    - **Mechanism:** Attackers overwhelm a target server or network with a flood of packets, making it unavailable to legitimate users.
    - **DDoS:** Distributed DoS, where thousands of botnet hosts attack a single target simultaneously.
- **Packet Sniffing:**
    - **Mechanism:** A passive attack where an attacker uses software (like Wireshark) on a broadcast medium (like unencrypted Wi-Fi) to record a copy of every packet passing by. This can capture plaintext passwords and sensitive data.
- **IP Spoofing:**
    - **Mechanism:** An active attack where a sender injects packets into the network with a false source IP address, masquerading as a trusted host.

### 2. Lines of Defense
- **Authentication:** Verifying the identity of the sender/receiver (e.g., via passwords or digital certificates).
- **Confidentiality (Encryption):** Using cryptographic algorithms to ensure that only the intended receiver can read the message content.
- **Integrity Checks:** Using digital signatures or hashes to ensure the message was not modified during transit.
- **Firewalls:** Specialized middleboxes that filter traffic based on IP addresses, port numbers, or protocol types to block unauthorized access.

#### Examples
- **Ransomware Attack:** A type of malware that encrypts a user's files and demands payment for the decryption key.
- **HTTPS:** Using SSL/TLS to provide encryption and authentication for web traffic, preventing packet sniffing of login credentials.
- **Company Firewall:** A system that blocks all incoming traffic to a database server except for requests coming from a specific internal application server.

## 8. History of Networking
- **1961-1972:** Early packet-switching principles (Kleinrock, Baran, ARPAnet demo).
- **1972-1980:** Internetworking, **NCP** first host-to-host protocol. Cerf and Kahn's principles: Minimalism, Autonomy, Best-effort, Stateless routing.
- **1980-1990:** New protocols (TCP/IP), proliferation of networks (NSFNET, BITNET), DNS created.
- **1990-2005:** Commercialization, World Wide Web (HTML, HTTP, browser), high-speed access, P2P.
- **2005-Present:** Mobile Internet, social networking, cloud computing, SDN, NFV.

---
### Glossary of Key Terms
| Term | Definition |
| :--- | :--- |
| **Host / End System** | Any device connected to a network (laptop, smartphone, server, IoT device). |
| **Packet** | A small chunk of data created by breaking a larger message into manageable pieces for transmission. |
| **Bandwidth** | The maximum transmission rate of a communication link, measured in bits per second (bps). |
| **Router** | A packet switch that operates at the network layer to forward packets toward their destination. |
| **Link-Layer Switch** | A packet switch that typically operates at the link layer within a single network. |
| **Access Network** | The physical link connecting an end system to the edge router of its ISP. |
| **Core Network** | The mesh of interconnected routers that forms the backbone of the Internet. |
| **Store-and-Forward** | A mechanism where a switch/router must receive the entire packet before it can begin transmitting it to the next link. |
| **Statistical Multiplexing** | On-demand resource sharing where the transmission capacity of a link is shared among users as they have packets to send. |
| **FDM** | **Frequency Division Multiplexing**; sharing a link by dividing the frequency spectrum into dedicated bands for each user. |
| **TDM** | **Time Division Multiplexing**; sharing a link by dividing time into slots, with each user assigned a periodic slot. |
| **DSLAM** | **DSL Access Multiplexer**; a device at the ISP's central office that pools data from multiple DSL subscribers. |
| **HFC** | **Hybrid Fiber Coax**; a broadband network using both fiber-optic and coaxial cables, common in cable TV internet. |
| **Bottleneck Link** | The link in an end-to-end path with the lowest transmission rate, which limits the overall throughput. |
| **Encapsulation** | The process of a lower-layer protocol wrapping data from a higher-layer protocol with its own header information. |
