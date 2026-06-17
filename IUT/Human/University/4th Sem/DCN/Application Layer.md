
The Application Layer is the top floor of the networking house. It’s where the apps you actually use (browser, email, Spotify) live. These apps don't care how the bits travel; they just want to send and receive messages.

---

## 1. The Two Big Ways to Build Apps (Paradigms)

Think of this as how you organize a party.

### A. Client-Server
There is a **central server** that provides services/resources, and multiple **clients** request those services.
*   **Key Fact:** Clients don't talk to each other; they only talk to the boss (the server).
*   **Examples:** Browsing a website (HTTP) where browser is the client and the web server is the server, sending an email (SMTP).
Client ----\
Client ----- > Server
Client ----/

### B. Peer-to-Peer / P2P (The Community Potluck)
No central server. Each peer can act as both the server and the client (Servents). Each peer can share data to others as well as request data. 
*   **Self-Scalability:** More peers -> More upload capacity.
*   **Example: BitTorrent**
    *   **Tracker:** A central server that tracks peers participating in a torrent.
    *   **Chunks:** Files are divided into 256KB chunks.
    *   **Churn:** Peers join and leave the network frequently.
    *   **Trading Strategy (Tit-for-Tat):** A peer sends chunks to the four neighbors who are currently providing it with data at the highest rate (unchoked). Every 30 seconds, it "optimistically unchokes" a random neighbor to explore new high-rate connections.
    *   **Rarest First:** Peers always request the rarest chunks among their neighbors first.
Q. How much time to distribute a file (size F) from one server to N peers?
ANS: In a Client-Server model, the distribution time increases linearly as N grows ($D_{c-s} \ge \max\{NF/u_s, F/d_{min}\}$). In P2P, the distribution time is significantly lower because each peer brings their own upload capacity to the network ($D_{p2p} \ge \max\{F/u_s, F/d_{min}, NF/(u_s + \sum u_i)\}$), making it much more scalable for large groups.


| Client-Server                                                 | Peer-Peer                                                  |
| ------------------------------------------------------------- | ---------------------------------------------------------- |
| Cost of maintaining data center and bandwidth at server side. | Cost Effective                                             |
| Offers Secure and reliable communication.                     | Do not require significant infrastructure                  |
| Performance is predictable                                    | Face challenges in security, reliability and communication |
|                                                               | Highly Scalable                                            |
|                                                               | Better Resource Utilization                                |


---

## 2. How Apps Talk: Sockets (The "Door" Analogy)
A **Process** is just a program running on a host. To talk to another process, it uses a **Socket**.
Within same host, two processes communicate via **Inter-process Communication**
*   **Feynman Term:** Imagine your app is a room. The Socket is the **door**. 
*   To send a message, you "shove it out the door." 
*   The **Transport Layer** (the hallway) picks it up and delivers it to the door of the room on the other side of the world.
*   **The Address:** To find the right door, you need two things:
    1.  **IP Address:** The address of the house (e.g., `103.82.173.186`).
    2.  **Port Number:** The specific room number inside the house (e.g., HTTP is Room 80, Mail is Room 25).

---

## 3. What do Apps Need? (Transport Requirements)

*   **Data Integrity:** Some apps (File transfer, Web) need **100% reliability**. If a single bit is lost, the file is broken.
*   **Timing:** Some apps (Gaming, Zoom) need **low delay**. They can handle a little bit of lost data, but they can't handle "lag."
*   **Throughput:** Video streaming needs a **minimum speed** to work without buffering.
*   **Security:** Encryption (TLS) to stop people from reading your "mail" as it passes through the hallway.

---
## Internet transport protocols services:

### 1. TCP (Transmission Control Protocol) Services
*   **Connection-Oriented:** Requires a "handshake" (setup) between client and server processes before data flows.
*   **Reliable Transport:** Ensures that all data sent is received correctly and in the right order.
*   **Flow Control:** Ensures the sender doesn't overwhelm the receiver's buffer.
*   **Congestion Control:** Throttles the sender when the network itself is overloaded to prevent "traffic jams."
*   **Examples:** Web browsing (HTTP), Email (SMTP), File Transfer (FTP).

### 2. UDP (User Datagram Protocol) Services
*   **Connectionless:** No handshake; it just starts "shouting" data immediately.
*   **Unreliable Data Transfer:** Does not guarantee that data will arrive, nor that it will arrive in order. No "receipts" (ACKs).
*   **No Bells and Whistles:** No congestion control, no flow control. It's "best effort" and as fast as the hardware allows.
*   **Examples:** DNS lookups, VoIP (Voice over IP), Video Streaming, Online Gaming.

### 4. Securing TCP
*   **The Limitation:** TCP itself does not provide encryption or security; it only manages the delivery of bytes.
*   **The Solution (TLS):** We use **TLS (Transport Layer Security)**, formerly known as SSL.
    *   TLS sits between the Application Layer and the TCP layer.
    *   It takes the cleartext data from the app, encrypts it, and then hands it to TCP to deliver reliably.
    *   This is what upgrades HTTP to HTTPS.



## 4. Key Internet Protocols

### [[HTTP]] (HyperText Transfer Protocol)
*   **Overview:** HTTP is the Web's application-layer protocol, operating on a client/server model. Browsers (clients) request objects, and web servers send objects in response.
*   **Transport:** Uses TCP as its underlying transport protocol (default port 80).
*   **Stateless:** HTTP is a stateless protocol; the server maintains no information about past client requests.
*   **Cookies:** Used to maintain state. Components include: (1) cookie header line in HTTP response, (2) cookie header line in next HTTP request, (3) cookie file on user's host, and (4) back-end database at the website.
*   **Connections:**
    *   **Non-persistent HTTP:** At most one object sent over a TCP connection; connection then closed. Requires 2 RTTs per object + transmission time.
    *   **Persistent HTTP:** Server leaves connection open after sending response; subsequent messages sent over same connection.
*   **HTTP Message Format:**
    *   **Request:** Line (Method: GET, POST, HEAD, PUT, DELETE; URL; Version), Headers (Host, User-Agent, Connection, etc.), Entity Body.
    *   **Response:** Status Line (Protocol, Status Code, Status Phrase), Headers (Date, Server, Content-Type), Entity Body.
*   **Status Codes:** 200 OK, 301 Moved Permanently, 400 Bad Request, 404 Not Found, 505 HTTP Version Not Supported.
*   **HTTP/2:** Goal is to decrease delay in multi-object requests. Introduced binary frames and request prioritization to mitigate **Head-of-Line (HOL) blocking** at the application layer.
*   **HTTP/3:** Adds security, per-object error control, and congestion control by using **QUIC** over UDP, further reducing latency and HOL blocking.

### [[DNS]] (Domain Name System)
*   **Overview:** A distributed database implemented in a hierarchy of name servers. An application-layer protocol allows hosts to resolve names to IP addresses.
*   **Services:** Hostname-to-IP translation, Host aliasing (canonical vs. alias names), Mail server aliasing, Load distribution.
*   **Hierarchy:**
    *   **Root Servers:** Contact-of-last-resort; 13 logical root servers worldwide.
    *   **Top-Level Domain (TLD) Servers:** Responsible for .com, .org, .net, .edu, and country domains (.uk, .jp).
    *   **Authoritative Servers:** Provide authoritative hostname-to-IP mappings for an organization's servers.
    *   **Local DNS Servers:** Not strictly in hierarchy; each ISP has one. Acts as a proxy, forwarding queries into the hierarchy.
*   **Query Types:**
    *   **Iterated Query:** Contacted server replies with the name of the next server to contact ("I don't know this name, but ask this server").
    *   **Recursive Query:** Puts the burden of name resolution on the contacted name server.
*   **Caching:** DNS servers cache mappings to improve response time; entries disappear after TTL (Time To Live).
*   **Resource Records (RR):** Format: `(name, value, type, ttl)`
    *   **Type A:** name=hostname, value=IP address.
    *   **Type NS:** name=domain, value=hostname of authoritative name server.
    *   **Type CNAME:** name=alias, value=canonical name.
    *   **Type MX:** value=name of mail server associated with name.

### [[SMTP]] / [[IMAP]] (E-mail)
*   **SMTP (Simple Mail Transfer Protocol):** Uses TCP (port 25) for reliable transfer. Three phases: Handshaking, Transfer, Closure. It is a "push" protocol.
*   **IMAP (Internet Mail Access Protocol):** A "pull" protocol used to retrieve, delete, and organize messages stored on a mail server.

### Modern Video & [[CDN]]s

#### 1. Video Streaming
*   **CBR (Constant Bit Rate):** Video rate fixed.
*   **VBR (Variable Bit Rate):** Video rate changes with amount of spatial/temporal coding.
*   **DASH (Dynamic, Adaptive Streaming over HTTP):** 
    *   **Manifest File:** Provides URLs for different versions (bitrates) of video chunks.
    *   **Client Intelligence:** The client periodically measures bandwidth and requests the appropriate chunk version.

#### 2. CDN (Content Distribution Networks)
*   **Challenge:** Distributing massive amounts of content to millions of users globally.
*   **Placement Strategies:**
    *   **Enter Deep:** Push CDN servers deep into access networks (many small clusters).
    *   **Bring Home:** Place fewer large clusters at IXPs or POPs.
*   **Operation:** When a client requests a video, the CDN redirects the client to a nearby cluster using **DNS Redirection** (CNAME records).


## 6. Socket Programming (DIY)
In Python, you can build your own "door" to the transport layer.

### A. UDP Sockets (Unreliable Datagram)
*   **Characteristics:** No "handshake" or connection setup. Sender explicitly attaches the IP destination and port number to each packet. Receiver extracts the sender's IP/port from the received packet.
*   **Use Case:** Fast transmission where occasional data loss is acceptable (e.g., DNS, VoIP).

### B. TCP Sockets (Reliable Byte Stream)
*   **Characteristics:** Client must contact the server first. Server must be running and have a "welcoming" socket. TCP establishes a connection ("pipe") between the client and server.
*   **Reliability:** Provides reliable, in-order byte-stream transfer.

### C. Handling Timeouts (The try-except Pattern)
In networking, programs often need to wait for events (like a response). To prevent a program from "freezing" forever, we use **timeouts**.
*   **`settimeout(n)`:** Sets a timer for `n` seconds on a socket.
*   **`try-except timeout`:** A robust way to catch the error if no data arrives within the timeout period.

**Example: TCP Server with Timeout (Python)**
```python
from socket import *
serverPort = 12000
serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(('', serverPort))
serverSocket.listen(1)

while True:
    connectionSocket, addr = serverSocket.accept()
    connectionSocket.settimeout(10) # Wait only 10 seconds for data
    try:
        message = connectionSocket.recv(1024).decode()
        # Process message...
    except timeout:
        print("Request timed out!")
    finally:
        connectionSocket.close()
```
