# DCN Chapter 2 Summary: The Application Layer

This comprehensive summary is compiled from *Computer Networking: A Top-Down Approach (8th Edition)* Chapter 2. It integrates all technical notes into a single structured document.

---

## 2.1 Principles of Network Applications

### 2.1.1 Network Application Architecture
- **Client-Server Architecture:** 
    - Always-on host (server) serves requests from many other hosts (clients). 
    - Clients do not communicate directly with each other. 
    - Server has a fixed, well-known IP address.
- **P2P Architecture:** 
    - Minimal or no reliance on dedicated servers. 
    - Exploits direct communication between pairs of intermittently connected hosts (peers). 
    - **Self-scalability:** Peers contribute resources (bandwidth, storage) as they consume them.

### 2.1.2 Processes Communicating
- **Process:** A program running within a host.
- **Socket:** The software interface through which a process sends and receives messages from the network. It is the "API" between the application and the transport layer.
- **Addressing:** To receive messages, a process must have an **identifier**:
    1.  **IP Address:** Identifies the host.
    2.  **Port Number:** Identifies the specific process on the host (e.g., HTTP: 80, SMTP: 25).

### 2.1.3 Transport Services Available to Applications
- **Reliable Data Transfer:** Ensuring all data is delivered correctly and in order.
- **Throughput:** Some apps (multimedia) need a minimum guaranteed rate; others (elastic apps) use whatever is available.
- **Timing:** Guaranteed low latency for real-time applications (gaming, VoIP).
- **Security:** Encryption and data integrity (e.g., TLS).

### 2.1.4 Internet Transport Protocols
- **TCP (Transmission Control Protocol):**
    - Connection-oriented (handshake required).
    - Reliable transport.
    - Flow control and Congestion control.
    - Does *not* provide timing or minimum throughput guarantees.
- **UDP (User Datagram Protocol):**
    - Connectionless.
    - Unreliable data transfer (no guarantee of delivery or order).
    - No congestion control; can send at any rate.
- **Securing TCP (TLS):** Sits between the application and transport layers to provide encryption, data integrity, and end-point authentication.

---

## 2.2 The Web and HTTP

### 2.2.1 Overview
- **HTTP (HyperText Transfer Protocol):** The Web’s application-layer protocol.
- **Stateless:** The server maintains no information about past client requests.

### 2.2.2 Persistent and Non-Persistent Connections
- **Non-Persistent HTTP:** 
    - Each TCP connection is closed after the server sends one object.
    - **Response Time:** 2 RTTs + transmission time per object.
- **Persistent HTTP (Default in HTTP/1.1):** 
    - Server leaves the connection open after sending a response.
    - Subsequent requests/responses can be sent over the same connection.

### 2.2.3 HTTP Message Format
- **Request Message:** Includes Method (GET, POST, HEAD, PUT, DELETE), URL, HTTP Version, Headers, and Entity Body.
- **Response Message:** Includes Status Line (Version, Status Code, Phrase), Headers, and Entity Body.
- **Status Codes:** 
    - `200 OK`, `301 Moved Permanently`, `400 Bad Request`, `404 Not Found`, `505 HTTP Version Not Supported`.

### 2.2.4 User-Server Interaction: Cookies
1.  Cookie header line in the HTTP response.
2.  Cookie header line in the next HTTP request.
3.  Cookie file on the user's host.
4.  Back-end database at the website.

### 2.2.5 Web Caching (Proxy Servers)
- **Conditional GET:** 
    - Goal: Don't send the object if the cache has an up-to-date version.
    - Header: `If-modified-since: <date>`.
    - Response: `304 Not Modified` if not changed.

### 2.2.6 HTTP/2 and HTTP/3
- **HTTP/2:** 
    - **Multiplexing:** Multiple streams over one TCP connection to avoid application-layer **Head-of-Line (HOL) blocking**.
    - **Binary Framing:** Breaks messages into smaller binary frames.
    - **Server Push:** Server proactively sends assets.
- **HTTP/3 (over QUIC):** 
    - Uses **UDP**.
    - Eliminates transport-layer HOL blocking (lost packets only affect their specific stream).

---

## 2.3 Electronic Mail (SMTP, POP3, IMAP)

### 2.3.1 SMTP (Simple Mail Transfer Protocol)
- **Type:** Push protocol.
- **Transport:** TCP (Port 25).
- **Phases:** Handshaking (HELO), Transfer (MAIL FROM, RCPT TO, DATA), Closure (QUIT).
- **Format:** 7-bit ASCII.

### 2.3.2 Mail Access Protocols (Pull)
- **POP3:** Stateless "download and delete" (or keep). Simple but lacks synchronization across devices.
- **IMAP:** Stateful; maintains folder structure and status (read/unread) on the server.
- **HTTP:** Common webmail interface (Gmail, etc.).

---

## 2.4 DNS (Domain Name System)

### 2.4.1 Services Provided
- Hostname-to-IP translation.
- Host aliasing (canonical vs. alias names).
- Mail server aliasing.
- Load distribution (mapping one name to a set of IP addresses).

### 2.4.2 Hierarchy of Name Servers
1.  **Root DNS Servers:** The top of the hierarchy.
2.  **Top-Level Domain (TLD) Servers:** Responsible for `.com`, `.org`, `.net`, etc.
3.  **Authoritative DNS Servers:** Provide the actual IP mapping for an organization.
- **Local DNS Server:** Acts as a proxy, forwarding queries into the hierarchy.

### 2.4.3 DNS Caching and Resolution
- **Recursive Query:** Burden of resolution is on the contacted server.
- **Iterative Query:** Server replies with the address of the next server to contact.
- **Caching:** Servers cache mappings (with a **TTL**) to speed up resolution and reduce root server load.

### 2.4.4 Resource Records (RR)
- Format: `(Name, Value, Type, TTL)`
- **Type A:** Name=hostname, Value=IP.
- **Type NS:** Name=domain, Value=Authoritative hostname.
- **Type CNAME:** Name=alias, Value=Canonical name.
- **Type MX:** Name=domain, Value=Mail server name.

### 2.4.5 DNS Security
- **DDoS:** Flooding servers.
- **Man-in-the-Middle:** Intercepting queries.
- **DNS Poisoning:** Injecting fake RRs into caches.
- **DNSSEC:** Adds authentication and integrity via digital signatures.

---

## 2.5 Peer-to-Peer Applications (BitTorrent)

### 2.5.1 P2P File Distribution
- **Distribution Time ($D$):** 
    - **Client-Server:** $D_{c-s} \ge \max\{NF/u_s, F/d_{min}\}$ (Linear increase with $N$).
    - **P2P:** $D_{p2p} \ge \max\{F/u_s, F/d_{min}, NF/(u_s + \sum u_i)\}$ (Scales much better).

### 2.5.2 BitTorrent
- **Tracker:** Tracks peers participating in a torrent.
- **Chunks:** Files divided into 256KB chunks.
- **Rarest First:** Peers request rarest chunks first.
- **Tit-for-Tat:** Unchokes the top 4 neighbors providing the best data rates; optimistically unchokes one random neighbor every 30s.

---

## 2.6 Video Streaming and CDNs

### 2.6.1 DASH (Dynamic, Adaptive Streaming over HTTP)
- Video is divided into chunks at multiple bitrates.
- **Manifest File:** Provides URLs for chunks.
- Client measures bandwidth and requests the appropriate bitrate.

### 2.6.2 Content Distribution Networks (CDNs)
- **Strategies:** 
    - **Enter Deep:** Many small clusters in access networks.
    - **Bring Home:** Fewer large clusters at IXPs/POPs.
- **Operation:** Uses **DNS Redirection** (CNAME) to point clients to the closest/best CDN node.

---

## 2.7 Socket Programming

### 2.7.1 UDP Sockets (Connectionless)
- No handshake.
- Packets sent with explicit Destination IP and Port.
- Receiver extracts Source IP/Port from packet.

### 2.7.2 TCP Sockets (Connection-Oriented)
- Handshake required to establish connection.
- "Welcome Socket" (Server) accepts connections and creates a new "Connection Socket" for each client.

---

## Questions and Answers (Q&A)

Q. How much time to distribute a file (size F) from one server to N peers?
ANS: In a Client-Server model, the distribution time increases linearly as N grows ($D_{c-s} \ge \max\{NF/u_s, F/d_{min}\}$). In P2P, the distribution time is significantly lower because each peer brings their own upload capacity to the network ($D_{p2p} \ge \max\{F/u_s, F/d_{min}, NF/(u_s + \sum u_i)\}$), making it much more scalable for large groups.

Q. What happens if the network connection or client crashes during a stateful transaction (at time $t'$)?
ANS: In a stateful protocol, a crash can lead to "inconsistent views" between the client and server. For example, the server might think you bought the item, but your client doesn't know it was successful. Stateless protocols avoid this by treating every request as a fresh, independent interaction.

Q. Why not centralize DNS?
ANS: A centralized DNS does not scale for the entire planet. It would create a Single Point of Failure, overwhelming Traffic Volume, high latency due to Physical Distance, and impossible Maintenance for billions of records.

Q. What is the difference between recursive and iterative DNS queries?
ANS: In a recursive query, the host asks the DNS server to find the answer for it, putting the entire burden on the server. In an iterative query, the contacted server replies with the address of another DNS server that might have the answer, and the host (or local DNS server) continues the search itself.

Q. What is Head-of-Line (HOL) blocking in HTTP?
ANS: HOL blocking occurs when a single large or slow object at the front of a queue prevents subsequent objects from being transmitted. HTTP/2 addresses this at the application layer via multiplexing, while HTTP/3 (QUIC) addresses it at the transport layer by using independent streams over UDP.

---
*See also:* [[Application Layer]], [[HTTP]], [[DNS]], [[SMTP]], [[IMAP]], [[Chapter 1 Summary]]
