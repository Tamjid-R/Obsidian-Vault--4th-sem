# Transport Layer: Process-to-Process Communication

The Transport Layer provides **logical communication** between application processes running on different hosts. It relies on and enhances the services provided by the [[Network Layer]] (host-to-host communication).

---

## 1. Transport Services and Protocols

### 1.1 Key Functions
*   **Sender Side:** Breaks application messages into **segments** and passes them to the [[Network Layer]].
*   **Receiver Side:** Reassembles segments into messages and passes them to the [[Application Layer]] via **sockets**.
two transport protocols available to
Internet applications
		*• TCP: reliable, in-order delivery
			congestion control - protects
			network
			 flow control - protects endpoints
			connection setup
		UDP

### 1.2 Transport vs. Network Layer (The Household Analogy)
*   **Hosts** = Houses
*   **Processes** = Kids in the houses
*   **App Messages** = Letters in envelopes
*   **Transport Protocol** = Ann and Bill (siblings) who distribute letters to their brothers and sisters (demux).
*   **Network Protocol** = Postal Service (moves mail between houses).

---

## 2. Multiplexing and Demultiplexing
How the transport layer directs data to the correct socket.

### 2.1 Demultiplexing (at Receiver)
The host uses IP addresses and **port numbers** to direct segments to the appropriate socket.
*   **Connectionless (UDP) Demux:** Sockets are identified by a 2-tuple: (**Destination IP**, **Destination Port**). Segments with the same destination port but different source IPs/ports are directed to the same socket.
*   **Connection-Oriented (TCP) Demux:** Sockets are identified by a **4-tuple**:
    1.  Source IP address
    2.  Source port number
    3.  Destination IP address
    4.  Destination port number
    *   The receiver uses **all four values** to direct the segment to the correct socket. A web server (e.g., Apache) may have many simultaneous TCP sockets, each identified by its own 4-tuple.

---

## 3. UDP: User Datagram Protocol [RFC 768]
A "no-frills," "bare-bones" Internet transport protocol.

### 3.1 Characteristics
*   **Connectionless:** No handshaking between sender and receiver.
*   **Best-Effort Service:** Segments may be lost or delivered out-of-order.
*   **Why use UDP?**
    *   No connection establishment delay (important for DNS).
    *   Simple: no connection state at sender/receiver.
    *   Small header size (8 bytes).
    *   No congestion control: UDP can "blast" data as fast as desired.
*   **Typical Uses:** Streaming multimedia (loss-tolerant, rate-sensitive), DNS, SNMP, HTTP/3.

### 3.2 UDP Segment Structure (8-byte Header)
| Field | Size | Description |
| :--- | :--- | :--- |
| **Source Port #** | 16 bits | Sending process port |
| **Dest Port #** | 16 bits | Receiving process port |
| **Length** | 16 bits | Length in bytes of UDP segment (including header) |
| **Checksum** | 16 bits | Used to detect errors (flipped bits) |

### 3.3 Internet Checksum
**Goal:** Detect errors in the transmitted segment.
*   **Sender:**
    1.  Treat contents (header + data + pseudo-header) as a sequence of 16-bit integers.
    2.  Add them using **one's complement sum**.
    3.  Carry out from MSB is added to the result (**Wraparound**).
    4.  The final result is bit-inverted to get the checksum.
*   **Receiver:** Compute checksum of the received segment and check if it results in all 1s.

---

## 4. TCP: Transmission Control Protocol [RFC 793, etc.]
A connection-oriented, reliable, in-order byte stream protocol.

### 4.1 Key Characteristics
*   **Point-to-Point:** One sender, one receiver.
*   **Reliable, In-Order Byte Stream:** No "message boundaries."
*   **Full Duplex Data:** Bi-directional data flow in the same connection.
*   **Connection-Oriented:** Handshaking initializes state before data exchange.
*   **Flow Controlled:** Sender will not overwhelm receiver.
*   **Congestion Controlled:** Sender limits rate to protect the network.

### 4.2 TCP Segment Structure
*   **Sequence Number:** Byte stream number of the first byte in the segment's data.
*   **Acknowledgement Number:** Sequence number of the **next byte expected** from the other side (Cumulative ACK).
*   **Flags:**
    *   **SYN:** Connection establishment.
    *   **FIN:** Connection teardown.
    *   **RST:** Reset connection.
    *   **ACK:** Indicates ACK field is valid.
    *   **C, E:** Congestion notification.
*   **Receive Window (rwnd):** Number of bytes the receiver is willing to accept (Flow Control).

### 4.3 Connection Management
#### 3-Way Handshake (Establishment)
1.  **SYN:** Client sends `SYN=1`, `Seq=x`.
2.  **SYN-ACK:** Server responds with `SYN=1`, `ACK=1`, `Seq=y`, `AckNum=x+1`.
3.  **ACK:** Client sends `ACK=1`, `AckNum=y+1`. May contain payload.

#### Closing Connection
Both sides send a segment with `FIN=1` and respond with an `ACK`.

### 4.4 Flow Control
The receiver "advertises" free buffer space in the `rwnd` field of the TCP header. The sender limits the amount of unACKed ("in-flight") data to the received `rwnd` value to ensure the receiver's buffer does not overflow.

### 4.5 Retransmission and Timeout
*   **Timeout Interval:** `EstimatedRTT + 4 * DevRTT`
*   **EstimatedRTT:** `(1 - α) * EstimatedRTT + α * SampleRTT` (typically `α = 0.125`).
*   **Fast Retransmit:** If the sender receives **3 duplicate ACKs** for the same data, it resends the unACKed segment with the smallest sequence number immediately, without waiting for a timeout.

---

## 5. Principles of Congestion Control
**Congestion:** Informally, "too many sources sending too much data too fast for the network to handle."

### 5.1 Manifestations
*   **Long Delays:** Queueing in router buffers.
*   **Packet Loss:** Buffer overflow at routers.

### 5.2 Costs of Congestion
*   Throughput can never exceed capacity.
*   Loss/retransmission decreases effective throughput.
*   **Unneeded duplicates** (premature timeouts) further decrease throughput.
*   Upstream transmission capacity/buffering is wasted for packets lost downstream.

### 5.3 Approaches
*   **End-to-End (TCP):** Congestion inferred from observed loss/delay (no explicit feedback from network).
*   **Network-Assisted:** Routers provide direct feedback (e.g., ECN).

---

## 6. Real-World Examples
*   **Web Browsing (HTTP):** Uses TCP to ensure that the HTML file is received perfectly and in order. If a packet is lost, TCP retransmits it so the page doesn't break.
*   **Video Conferencing (Zoom/Teams):** Often uses UDP (or UDP-based protocols) because a slight glitch in video is better than a long delay caused by retransmitting old data.
*   **DNS:** Uses UDP for quick, single-packet queries and responses, avoiding the overhead of setting up a TCP connection.
