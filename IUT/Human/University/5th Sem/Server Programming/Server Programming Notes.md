# SWE 4537 — Server Programming Notes

**Course:** SWE 4537: Server Programming  
**Department:** Department of Computer Science and Engineering (CSE), Islamic University of Technology (IUT)  
**Instructors:** Ishmam Tashdeed | Asaduzzaman Herok  
**Source Materials:** `Lecture-1.pdf` (The Client & Server, Digital Addresses, HTTP Protocol) and `Lecture-2.pdf` (Web Services, Web APIs, REST Architecture, MVC Pattern)

---

## Table of Contents
- [[#1. The Client-Server Architecture]]
  - [[#1.1 Web Client]]
  - [[#1.2 Web Server Hardware vs. Software]]
  - [[#1.3 Static vs. Dynamic Web Servers]]
  - [[#1.4 Web Applications (Frontend vs. Backend)]]
  - [[#1.5 Client-Server Interaction Flow]]
- [[#2. Digital Addressing: Networking Fundamentals]]
  - [[#2.1 IP Addresses]]
  - [[#2.2 Port Numbers & Categorization]]
  - [[#2.3 Network Sockets]]
  - [[#2.4 Anatomy of a URL (Uniform Resource Locator)]]
- [[#3. The HTTP Protocol (Hypertext Transfer Protocol)]]
  - [[#3.1 Core Characteristics of HTTP]]
  - [[#3.2 Structure of an HTTP Message]]
  - [[#3.3 HTTP Request Anatomy]]
  - [[#3.4 HTTP Response Anatomy]]
  - [[#3.5 HTTP Status Code Classes]]
- [[#4. Web Services & Paradigms of Machine Communication]]
  - [[#4.1 Web Application vs. Web Service]]
  - [[#4.2 Communication Paradigms: RPC, SOAP, and REST]]
  - [[#4.3 SOAP Deep Dive & WSDL Contract]]
  - [[#4.4 The Restaurant Analogy of APIs]]
- [[#5. REST (Representational State Transfer) Architecture]]
  - [[#5.1 The 6 Guiding Architectural Constraints]]
  - [[#5.2 REST vs. RPC: Nouns vs. Verbs]]
  - [[#5.3 RESTful API Design (Collections vs. Single Resources)]]
  - [[#5.4 HTTP Methods Matrix (Safety & Idempotence)]]
  - [[#5.5 Representations & Content Negotiation]]
  - [[#5.6 The Statelessness Constraint & Scalability]]
- [[#6. Architectural Organization: MVC Pattern for Web APIs]]
  - [[#6.1 The "Big Messy File" Anti-Pattern & Separation of Concerns]]
  - [[#6.2 The MVC Layers Explained]]
  - [[#6.3 Step-by-Step Execution Flow in an MVC API]]
  - [[#6.4 Practical Comparison: Monolithic Script vs. Clean MVC Structure]]

---

## 1. The Client-Server Architecture

The foundation of modern web systems rests upon the **Client-Server model**—a distributed application architecture that partitions tasks or workloads between service requesters (clients) and service providers (servers).

```mermaid
flowchart LR
    subgraph ClientDevice ["Client Device (Browser / Mobile / cURL)"]
        UI["User Interface / Runtime"]
    end

    subgraph InternetBoundary ["Network Boundary (IP / TCP / HTTP)"]
        Req["HTTP Request (URL, Headers, Body)"]
        Res["HTTP Response (Status, Headers, Payload)"]
    end

    subgraph ServerHost ["Server Host (Hardware & Daemon)"]
        Daemon["Web Server Software (Nginx / Apache / IIS)"]
        App["Web Application / Backend Logic"]
        DB[(Database)]
    end

    UI -->|1. Issues Request| Req
    Req -->|2. Network Route| Daemon
    Daemon -->|3. Route / Execute| App
    App <-->|4. SQL / Data| DB
    App -->|5. Build Response| Daemon
    Daemon -->|6. Transmit| Res
    Res -->|7. Parse & Render| UI
```

### 1.1 Web Client
A **Web Client** is any software agent executing on a user-controlled device (laptop, smartphone, IoT device, or automated script) that connects to a web server to retrieve and consume resources.

*   **Core Responsibilities:**
    1.  **Request Construction:** Accepts user input or programmatic instructions (e.g., typing a URL or executing an automated script) and synthesizes a compliant HTTP request.
    2.  **Transmission:** Dispatches the request across the network stack targeting a specific IP address and socket.
    3.  **Parsing & Processing:** Receives the server's byte stream and decodes headers, status codes, and the payload (HTML, CSS, JavaScript, JSON, XML, or binary data).
    4.  **Presentation / Execution:** In browsers, it parses the DOM (Document Object Model), calculates the CSSOM (CSS Object Model), executes client-side JavaScript, and renders the visual layout. In API clients (like mobile apps or microservices), it deserializes data structures directly into runtime memory.
*   **Examples:**
    *   *Web Browsers:* Google Chrome, Mozilla Firefox, Apple Safari, Microsoft Edge.
    *   *Developer & Testing Tools:* `curl`, Postman, Insomnia, HTTPie.
    *   *Native Software:* Mobile applications (iOS/Android) leveraging networking libraries (`URLSession`, `Retrofit`, `Axios`).

---

### 1.2 Web Server: Hardware vs. Software
The term "Web Server" refers to two interconnected layers:

| Layer | Technical Specification | Primary Function |
|---|---|---|
| **Hardware Server** | Physical bare-metal server, rack-mounted blade, or cloud Virtual Machine (e.g., AWS EC2, DigitalOcean Droplet) with CPU, RAM, storage, and network interface cards (NICs). | Provides raw compute and host environment with a static or routed IP address; maintains network connectivity and power 24/7. |
| **Software Server** | Daemon software process running in user/kernel space listening on designated transport ports (e.g., Port 80 for HTTP, Port 443 for HTTPS). | Implements the HTTP protocol, manages connection pooling, worker threads/event loops, terminates SSL/TLS, and routes requests to appropriate endpoints or files. |

*   **Prominent Web Server Software:**
    *   **Nginx:** Asynchronous, event-driven, high-concurrency server often used as a reverse proxy, static asset server, and load balancer.
    *   **Apache HTTP Server:** Process- and thread-driven modular server using `.htaccess` configurations.
    *   **Microsoft IIS (Internet Information Services):** Deeply integrated Windows Server daemon tailored for .NET runtime applications.
    *   **Caddy:** Modern Go-based web server featuring automatic TLS certificate provisioning via Let's Encrypt.

---

### 1.3 Static vs. Dynamic Web Servers

```mermaid
flowchart TD
    Req1["Request: GET /images/logo.png"] --> SServer["Static Web Server (e.g., Nginx)"]
    SServer -->|Direct File Read| Disk[("File System Disk")]
    Disk -->|Raw Byte Stream| SServer
    SServer --> Res1["Response: 200 OK (image/png)"]

    Req2["Request: GET /api/v1/profile"] --> DServer["Dynamic Web Server"]
    DServer --> AppRuntime["Application Backend Runtime (FastAPI / Node / Spring)"]
    AppRuntime --> BusinessLogic["Execute Business Logic & Auth Check"]
    BusinessLogic --> DB[("Database")]
    DB --> BusinessLogic
    BusinessLogic --> JSONGen["Serialize Data to JSON"]
    JSONGen --> Res2["Response: 200 OK (application/json)"]
```

1.  **Static Web Server:**
    *   **Behavior:** Direct 1-to-1 mapping between the requested URL path and files located on the server's storage disk.
    *   **Operation:** Receives `GET /about.html`, reads the `/var/www/html/about.html` file off disk, appends MIME headers (`Content-Type: text/html`), and streams bytes directly back.
    *   **Characteristics:** Extremely fast, low CPU utilization, heavily cacheable via CDNs, but incapable of delivering personalized or runtime-generated data.
2.  **Dynamic Web Server:**
    *   **Behavior:** Does not merely serve static files. It invokes an application runtime (Python, Node.js, Java, Go, PHP) to generate content on the fly.
    *   **Operation:** Receives `GET /orders/984`, executes backend code to authenticate the user, queries an SQL database for order #984, computes tax/discounts, packages the result into a response (HTML template or JSON payload), and returns it.
    *   **Characteristics:** Resource intensive, requires runtime database queries and logic execution, but enables user personalization, state modification, and transactional integrity.

---

### 1.4 Web Applications (Frontend vs. Backend)
A **Web Application** is an interactive software application whose components are split across client and server environments.

*   **Client-Side (Frontend):**
    *   Runs strictly within the client device/browser engine.
    *   Responsible for visual UI rendering, styling, capturing user interactions (clicks, keyboard input), local routing, and client-side form validation.
    *   *Technologies:* HTML5, CSS3, JavaScript/TypeScript, React, Vue, Angular, Svelte.
*   **Server-Side (Backend):**
    *   Runs in a secure, controlled server environment.
    *   Contains the core **business logic**, authorization gates, database interaction routines, payment processing, and cryptographic security.
    *   *Technologies:* Python (FastAPI, Django), Node.js (Express, NestJS), Java (Spring Boot), Go, PostgreSQL, Redis.
*   **Real-World Example:** In **Gmail**, the client-side renders the list of emails, displays rich animations, and immediately shows typed drafts. The server-side authenticates your credentials, validates spam filters, stores emails across distributed databases, and ensures security policies are enforced.

---

### 1.5 Client-Server Interaction Flow

```mermaid
sequenceDiagram
    autonumber
    actor User as User / Client
    participant DNS as DNS Server
    participant Server as Web Server (80 / 443)
    participant App as Backend App
    participant DB as Database

    User->>DNS: Resolve "api.example.com"
    DNS-->>User: Returns IP: 93.184.216.34
    User->>Server: TCP 3-Way Handshake (SYN -> SYN-ACK -> ACK)
    User->>Server: HTTP Request: GET /api/todos/42
    Server->>App: Forward request to matching route handler
    App->>DB: Query: SELECT * FROM todos WHERE id = 42
    DB-->>App: Raw table record: {id: 42, title: "Study HTTP"}
    App-->>Server: Formatted JSON payload + 200 OK
    Server-->>User: HTTP/1.1 200 OK (Content-Type: application/json)
    Note over User: Client parses JSON and updates UI
```

---

## 2. Digital Addressing: Networking Fundamentals

For two computing machines to establish communication, they must uniquely identify each other on the network and address the specific software application running on that machine.

### 2.1 IP Addresses
An **Internet Protocol (IP) Address** is a unique numerical label assigned to every device participating in a computer network that uses the Internet Protocol.

*   **Purpose:** Host identification and network layer addressing.
*   **IPv4 (Internet Protocol version 4):**
    *   32-bit numerical address expressed as four octets separated by dots (dotted-decimal notation).
    *   Range: `0.0.0.0` to `255.255.255.255` (~4.29 billion addresses).
    *   *Example:* `172.217.14.228` (Google server).
*   **IPv6 (Internet Protocol version 6):**
    *   128-bit address expressed as eight groups of four hexadecimal digits separated by colons.
    *   *Example:* `2001:0db8:85a3:0000:0000:8a2e:0370:7334`.

---

### 2.2 Port Numbers & Categorization
While an IP address directs data packets to a specific physical or virtual machine, a **Port Number** identifies the specific process or service running on that machine. A port is an unsigned 16-bit integer ranging from **`0` to `65535`**.

```
Total Port Range: 0 — 65535 (16-bit unsigned integer)
├── 0 to 1023: Well-Known (System) Ports
│   ├── Port 22:  SSH (Secure Shell)
│   ├── Port 53:  DNS (Domain Name System)
│   ├── Port 80:  HTTP (Standard Web Traffic)
│   └── Port 443: HTTPS (Encrypted Web Traffic)
├── 1024 to 49151: Registered Ports
│   ├── Port 3000: React / Node.js default development server
│   ├── Port 5432: PostgreSQL Database
│   ├── Port 6379: Redis in-memory cache
│   └── Port 8000 / 8080: FastAPI / Uvicorn / Spring Boot default ports
└── 49152 to 65535: Dynamic / Private / Ephemeral Ports
    └── Temporary ports automatically assigned by the client OS for outbound connections.
```

---

### 2.3 Network Sockets
A **Socket** represents the concrete software endpoint of a bidirectional network communication channel. It is the logical combination of an **IP Address** and a **Port Number**:

$$\text{Socket} = \langle \text{IP Address} : \text{Port Number} \rangle$$

*   **Example Endpoint:** `192.168.1.10:51255`
    *   When your browser opens a connection to `https://www.google.com`, the client operating system opens an **ephemeral client socket** (e.g., `192.168.1.45:54321`) bound to the remote **server socket** (e.g., `172.217.14.228:443`).

---

### 2.4 Anatomy of a URL (Uniform Resource Locator)
A **URL** is a human-readable structural string specifying the location of an internet resource and the mechanism used to access it.

```
https://www.example.com:8080/products/catalog?category=laptops&sort=price#specs
└─┬─┘   └──────┬──────┘ └──┬─┘└───────┬────────┘ └───────────┬──────────┘ └──┬──┘
Scheme      Hostname      Port       Path               Query String       Fragment
```

#### Detailed Breakdown of URL Components:
1.  **Scheme / Protocol (`https://`):** Declares the communication protocol. `http://` indicates standard cleartext communication; `https://` indicates HTTP encapsulated in TLS/SSL encryption.
2.  **Hostname / Domain (`www.example.com`):** The human-friendly alphanumeric identifier that maps directly to an IP address through Domain Name System (DNS) resolution.
3.  **Port (`:8080`):** Explicit target port on the host machine. If omitted, browsers infer defaults based on the scheme (`:80` for HTTP, `:443` for HTTPS).
4.  **Path (`/products/catalog`):** Hierarchical route specifying the exact resource or controller endpoint on the web server.
5.  **Query String (`?category=laptops&sort=price`):** Delimited by `?` followed by `key=value` pairs separated by `&`. Used to send optional filter parameters, sorting flags, and pagination tokens without altering the resource path.
6.  **Fragment Identifier (`#specs`):** Delimited by `#`. Points to a specific anchor within the client's rendered page. **Crucial rule:** Fragments are processed entirely on the client side; browsers do **not** transmit the fragment to the server in the HTTP request line.
7.  **URL Encoding (Percent-Encoding):** URLs may only contain a restricted set of ASCII characters. Any reserved or unsafe characters (like spaces or non-ASCII symbols) must be escaped using a `%` followed by two hexadecimal digits.
    *   *Example:* Space becomes `%20` or `+`. A URL like `/search?q=server programming` becomes `/search?q=server%20programming`.

---

## 3. The HTTP Protocol (Hypertext Transfer Protocol)

HTTP is an application-layer protocol designed to transfer hypermedia documents and raw data over the Internet. It is the universal language of web communication.

### 3.1 Core Characteristics of HTTP
1.  **Stateless:** The server retains **no conversational memory** between successive requests from the same client. Request #2 is evaluated completely independently of Request #1. (State is maintained artificially via cookies, server sessions, or authorization tokens).
2.  **Request-Response Paradigm:** Communication is always initiated by the client issuing a single request; the server processes the request and responds with a single corresponding response before closing or releasing the connection.
3.  **Extensible via Headers:** HTTP can easily be extended to support custom features, security configurations, and compression through arbitrary key-value headers.
4.  **Media Independent:** The payload format is not restricted; HTTP can transport plain text, HTML, JSON, XML, images, audio, video, and binary blobs as long as both parties agree on the MIME type (`Content-Type`).
5.  **Caching Built-in:** Includes explicit mechanisms (`Cache-Control`, `ETag`, `Last-Modified`) to allow intermediate proxies and clients to reuse previously fetched responses.

---

### 3.2 Structure of an HTTP Message
Both HTTP Requests and HTTP Responses share a strictly defined four-part text structure:

```
┌────────────────────────────────────────────────────────┐
│ 1. Start Line (Request Line OR Status Line)            │
├────────────────────────────────────────────────────────┤
│ 2. Headers (Zero or more Field-Name: Value pairs)      │
├────────────────────────────────────────────────────────┤
│ 3. Empty Line (A single CRLF delimiter "\r\n")         │
├────────────────────────────────────────────────────────┤
│ 4. Message Body (Optional payload: JSON, Form, etc.)   │
└────────────────────────────────────────────────────────┘
```

> [!IMPORTANT]
> The **Empty Line** (consisting of a Carriage Return followed by Line Feed: `\r\n`) is mandatory. It serves as the explicit boundary indicating to the parser that header processing is complete and the message payload begins.

---

### 3.3 HTTP Request Anatomy

#### Format:
```http
<METHOD> <Request-URI> <HTTP-Version> CRLF
<Header-Key>: <Header-Value> CRLF
...
CRLF
[Optional Request Body]
```

#### Real-World Example:
```http
POST /api/v1/users HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)
Content-Type: application/json
Content-Length: 48
Accept: application/json

{"username": "johndoe", "email": "john@iut.edu"}
```

*   **Request Line Components:**
    *   **Method (`POST`):** The action to be executed on the resource.
    *   **Request-URI (`/api/v1/users`):** The target endpoint path.
    *   **HTTP Version (`HTTP/1.1`):** Protocol standard in use.
*   **Request Headers:** Metadata providing contextual information about the request and client capabilities (e.g., target `Host`, payload size `Content-Length`, data format `Content-Type`).

---

### 3.4 HTTP Response Anatomy

#### Format:
```http
<HTTP-Version> <Status-Code> <Reason-Phrase> CRLF
<Header-Key>: <Header-Value> CRLF
...
CRLF
[Optional Response Body]
```

#### Real-World Example:
```http
HTTP/1.1 201 Created
Date: Sat, 07 Nov 2026 23:59:00 GMT
Server: uvicorn / fastapi
Content-Type: application/json
Content-Length: 63

{"id": 101, "username": "johndoe", "status": "account_created"}
```

*   **Status Line Components:**
    *   **HTTP Version (`HTTP/1.1`):** Protocol specification.
    *   **Status Code (`201`):** 3-digit numerical result of the request.
    *   **Reason Phrase (`Created`):** Human-readable text summarizing the numerical status code.

---

### 3.5 HTTP Status Code Classes
The first digit of the 3-digit status code defines the general classification of the server's response:

| Code Range | Category | Architectural Meaning | Common Examples |
|---|---|---|---|
| **`1xx`** | **Informational** | Request received, protocol handshake or process continuing. | `100 Continue`<br>`101 Switching Protocols` (used in WebSockets upgrade) |
| **`2xx`** | **Success** | The action was successfully received, understood, and accepted. | `200 OK` (Standard retrieval/update)<br>`201 Created` (Resource created via POST)<br>`204 No Content` (Success, no body returned; common in DELETE) |
| **`3xx`** | **Redirection** | Further client action required to complete request. | `301 Moved Permanently` (SEO redirects)<br>`304 Not Modified` (Client cache is still fresh) |
| **`4xx`** | **Client Error** | The request contains invalid syntax, missing auth, or requests non-existent data. | `400 Bad Request` (Malformed JSON/syntax)<br>`401 Unauthorized` (Authentication required/missing)<br>`403 Forbidden` (Authenticated but lacking permissions)<br>`404 Not Found` (Resource does not exist)<br>`422 Unprocessable Entity` (Schema validation failure) |
| **`5xx`** | **Server Error** | The server encountered an unexpected error while processing a valid request. | `500 Internal Server Error` (Unhandled exception)<br>`502 Bad Gateway` (Upstream service failure)<br>`503 Service Unavailable` (Server overloaded/maintenance)<br>`504 Gateway Timeout` (Database or upstream timed out) |

---

## 4. Web Services & Paradigms of Machine Communication

### 4.1 Web Application vs. Web Service

```
┌────────────────────────────────────────────────────────┐
│ Web Application (Engineered for Humans)               │
│ Output: HTML + CSS + JS (Graphical User Interface)     │
│ Analogy: A fully assembled, painted, drivable sports car│
└────────────────────────────────────────────────────────┘
                           vs.
┌────────────────────────────────────────────────────────┐
│ Web Service (Engineered for Machines / Programs)       │
│ Output: Structured Raw Data (JSON, XML)                │
│ Analogy: The modular engine exported to another factory│
└────────────────────────────────────────────────────────┘
```

*   **Web Application:** Focuses on human interaction. Returns rich markup files that web browsers parse into interactive buttons, visual layouts, and forms.
*   **Web Service:** A software system designed to support interoperable machine-to-machine interaction over a network. It exposes raw data structures without UI styling, allowing mobile apps, frontend SPAs, and third-party microservices to consume the data.

---

### 4.2 Communication Paradigms: RPC, SOAP, and REST

To exchange data between machines over a network, three primary paradigms have evolved:

```mermaid
flowchart TD
    subgraph Paradigms ["Paradigms of Web Communication"]
        direction TB
        RPC["1. RPC (Remote Procedure Call)\n- Focus: Actions & Procedures (Verbs)\n- Example: /calculateTax, /getUsers\n- Pros: Intuitive procedural syntax\n- Cons: Brittle, tight coupling"]
        SOAP["2. SOAP (Simple Object Access Protocol)\n- Focus: Rigid Formal Contract\n- Mechanism: Strict XML Envelopes + WSDL\n- Pros: Enterprise security & transactions\n- Cons: Heavy overhead, verbose, rigid"]
        REST["3. REST (Representational State Transfer)\n- Focus: Resources (Nouns) & Standard Verbs\n- Mechanism: Native HTTP + JSON representations\n- Pros: Highly scalable, decoupled, standardized\n- Cons: Less rigid schema enforcement"]
    end
```

#### Detailed Comparison:
1.  **RPC (Remote Procedure Call):**
    *   *Concept:* Treat remote network calls as if invoking local functions in programming memory (e.g., `remoteService.getUser(id)`).
    *   *Endpoint Design:* URLs represent functions/actions: `POST /createUser`, `POST /deleteUser`, `GET /getUsersByDept`.
    *   *Architectural Flaw:* **Tight Coupling and Brittleness.** If the server developer renames a function or alters parameter order, all client integrations immediately break. Adding query variations causes endpoint explosion (`/getUsersByAge`, `/getUsersByNameAndDept`).
2.  **SOAP (Simple Object Access Protocol):**
    *   *Concept:* A highly structured, formal communication protocol strictly enforcing rules defined in an XML-based schema.
    *   *Mechanism:* Every request and response is wrapped in a verbose XML **Envelope** containing an optional Header and a mandatory Body.
    *   *Enterprise Strengths:* Supports formal standards for end-to-end security (**WS-Security**) and distributed ACID transactions (**WS-AtomicTransaction**).
    *   *Architectural Flaw:* Enormous bandwidth and processing overhead due to large XML payloads; complex to implement and debug.
3.  **REST (Representational State Transfer):**
    *   *Concept:* An architectural style that leverages existing HTTP capabilities. Instead of treating the server as an arbitrary execution container for procedures, it models the server as a repository of **Resources**.
    *   *Mechanism:* Standard HTTP verbs (`GET`, `POST`, `PUT`, `DELETE`) operate on resource paths (`/todos`). Data representations are lightweight (predominantly JSON).

---

### 4.3 SOAP Deep Dive & WSDL Contract
A SOAP service relies entirely on a **WSDL (Web Services Description Language)** file—an XML document that functions as an immutable, machine-readable contract between client and server.

```mermaid
flowchart LR
    WSDL["WSDL Contract (XML Schema)"]
    WSDL --> T["1. <types>: Data types defined"]
    WSDL --> M["2. <message>: Request & Response payloads"]
    WSDL --> P["3. <portType>: Abstract operations (functions)"]
    WSDL --> B["4. <binding>: Protocol binding (e.g., SOAP over HTTP)"]
    WSDL --> S["5. <service>: Physical location / URL endpoint"]
```

#### The 5 Foundational Questions Answered by WSDL:
1.  `<types>`: What data types exist in this system? (e.g., `UserID` is an integer, `UserName` is a string).
2.  `<message>`: What do the request and response message envelopes look like? (e.g., `getUserRequest` takes a `UserID`).
3.  `<portType>`: What abstract operations/functions can be called? (e.g., `getUser` takes `getUserRequest` and returns `getUserResponse`).
4.  `<binding>`: Which concrete transport protocol and serialization format binds these operations? (e.g., SOAP 1.2 over HTTP).
5.  `<service>`: What is the physical network address (URL) where the service is deployed? (e.g., `http://api.example.com/UserService`).

#### Real-World Analogy: WSDL vs. SOAP
*   **WSDL:** The printed, official user manual or formal contract detailing the product specifications.
*   **SOAP:** The specific, highly standardized physical container packed according to that manual.

---

### 4.4 The Restaurant Analogy of APIs
An **API (Application Programming Interface)** is the formal contract defining how external applications interact with a software system.

```mermaid
sequenceDiagram
    participant Client as Client (Diner)
    participant API as Web API (The Waiter)
    participant Backend as Backend Logic (Chefs / Kitchen)
    participant DB as Database (Pantry / Ingredients)

    Note over Client: Sits at table (Not allowed inside kitchen)
    Client->>API: Reviews Menu (API Documentation)
    Client->>API: Places Order: "GET /steak"
    API->>Backend: Validates & Passes ticket: prepareSteak()
    Backend->>DB: Fetch ingredients (raw data)
    DB-->>Backend: Return raw ingredients
    Backend-->>API: Finished dish assembled
    API-->>Client: Serves meal (HTTP 200 OK + Payload)
```

#### Paradigm Translation Through the Restaurant Analogy:
*   **RPC Approach:** The diner yells across the restaurant directly into the kitchen: `"cookSteakMediumRareWithFries()"`. If the kitchen renames the internal function to `makeSteak()`, the order fails.
*   **SOAP Approach:** The waiter hands the diner a 20-page legal contract in Latin (WSDL). The diner must fill out a notarized form in triplicate using strict XML syntax to request water.
*   **REST Approach:** The waiter presents a clean, logical menu organized strictly by item categories (Resources/Nouns). The customer uses standardized verbs: `"I will GET the /steak/filet"`.

---

## 5. REST (Representational State Transfer) Architecture

Formulated in 2000 by **Roy Fielding** in his doctoral dissertation, REST is not a protocol or a library—it is an **architectural style** governed by six fundamental constraints.

### 5.1 The 6 Guiding Architectural Constraints

```
┌────────────────────────────────────────────────────────────────────────┐
│                   FIELDING'S 6 REST CONSTRAINTS                        │
├──────────────────────────┬─────────────────────────────────────────────┤
│ 1. Client-Server         │ Clear Separation of Concerns (UI vs. Data)  │
│ 2. Statelessness         │ Every request holds 100% of required context│
│ 3. Cacheability          │ Responses must explicitly declare freshness │
│ 4. Uniform Interface     │ Consistent resource URI and verb semantics  │
│ 5. Layered System        │ Intermediaries (proxies, CDNs) are invisible│
│ 6. Code on Demand (Opt.) │ Server can push executable client code (JS) │
└──────────────────────────┴─────────────────────────────────────────────┘
```

1.  **Client-Server:**
    *   Separates user interface concerns from data storage and business logic concerns.
    *   Allows frontend client code (React, iOS) to evolve independently from backend server architecture (FastAPI, PostgreSQL).
2.  **Statelessness:**
    *   The server never stores client session state between requests.
    *   Every request must contain complete authentication credentials, resource IDs, and instructions necessary to fulfill it.
3.  **Cacheability:**
    *   To prevent unnecessary network round-trips, responses must explicitly label themselves as cacheable or non-cacheable (`Cache-Control` header).
4.  **Uniform Interface (The Core Pillar):**
    *   A single, standardized mechanism for interacting with server resources regardless of platform.
    *   Divided into:
        *   *Identification of resources:* Unique URIs (`/users/12`).
        *   *Manipulation of resources through representations:* Sending JSON to modify server state.
        *   *Self-descriptive messages:* Headers (`Content-Type: application/json`) explicitly define how to process the message.
        *   *HATEOAS (Hypermedia As The Engine Of Application State):* Responses include hyperlinks guiding the client on available next actions.
5.  **Layered System:**
    *   The client cannot tell whether it is communicating directly with the origin server or via intermediary proxies, reverse proxies, caches, or load balancers.
6.  **Code on Demand (Optional):**
    *   Servers can temporarily extend client functionality by transmitting executable code (e.g., executing client-side JavaScript or WebAssembly).

---

### 5.2 REST vs. RPC: Nouns vs. Verbs

The most significant design shift between RPC and REST is the distinction between **Procedures (Actions)** and **Resources (Entities)**.

| Criteria | RPC Architectural Approach | REST Architectural Approach |
|---|---|---|
| **Core Philosophy** | Action-oriented (Verbs) | Resource-oriented (Nouns) |
| **URL Design** | URLs specify procedures to execute | URLs identify unique entity collections/items |
| **Fetch All Items** | `GET /getAllTodos` | `GET /todos` |
| **Fetch Single Item**| `POST /getTodoById` with `{id: 5}` | `GET /todos/5` |
| **Create Item** | `POST /createNewTodo` | `POST /todos` |
| **Update Item** | `POST /updateTodoTitle` | `PATCH /todos/5` (or `PUT /todos/5`) |
| **Delete Item** | `POST /removeTodo?id=5` | `DELETE /todos/5` |
| **Filtering/Search** | Requires new endpoints: `/getTodosByDate` | Uses query parameters: `/todos?date=2026-11-08` |
| **Maintainability** | Brittle; endpoint explosion occurs over time | Highly clean, predictable, and scalable |

---

### 5.3 RESTful API Design (Collections vs. Single Resources)

In a RESTful architecture, URIs represent **collections** or **individual resource instances**. We combine these clean noun-based paths with native **HTTP Verbs** to perform CRUD operations:

```
Collection URI: /todos
├── GET    /todos         → Read: Retrieve a paginated list of todos
└── POST   /todos         → Create: Add a new todo item to the collection

Instance URI:   /todos/{id}
├── GET    /todos/123     → Read: Retrieve details of todo #123
├── PUT    /todos/123     → Update: Completely replace todo #123 with new payload
├── PATCH  /todos/123     → Partial Update: Modify specific fields of todo #123
└── DELETE /todos/123     → Delete: Permanently remove todo #123
```

*   **Handling Query Filters:** In REST, you do not invent endpoints like `/getActiveTodos`. Instead, filter the collection using query strings:
    *   `GET /todos?status=completed&priority=high`

---

### 5.4 HTTP Methods Matrix (Safety & Idempotence)

When designing RESTful APIs, HTTP verbs are strictly categorized by two mathematical/engineering properties: **Safety** and **Idempotence**.

$$\begin{aligned}
\textbf{Safe Operation:} &\quad f(S) = S \quad \text{(State of the server is never mutated)} \\
\textbf{Idempotent Operation:} &\quad f(f(S)) = f(S) \quad \text{(Executing } N \text{ times produces identical state as } 1 \text{ time)}
\end{aligned}$$

| HTTP Method | CRUD Operation | Safe? | Idempotent? | Request Body Allowed? | Purpose / Practical Usage |
|---|---|:---:|:---:|:---:|---|
| **`GET`** | Read | **Yes** | **Yes** | No | Retrieves resource state without mutating server records. |
| **`HEAD`** | Read Metadata | **Yes** | **Yes** | No | Identical to `GET`, but server returns **headers only** (empty body). Used to check file size or cache headers before downloading large files. |
| **`OPTIONS`**| Discover Options | **Yes** | **Yes** | Optional | Probes the server to identify permitted HTTP methods on a URI (essential for browser **CORS preflight**). |
| **`POST`** | Create | **No** | **No** | **Yes** | Submits data to create a new subordinate resource. Executing 5 times creates 5 distinct records. |
| **`PUT`** | Full Replace | **No** | **Yes** | **Yes** | Replaces the *entire* target resource with the uploaded payload. Repeated identical calls produce the exact same final server state. |
| **`PATCH`** | Partial Update | **No** | **No\*** | **Yes** | Modifies only specified attributes of an existing resource (e.g., updating only a `completed: true` flag). |
| **`DELETE`** | Delete | **No** | **Yes** | Optional | Removes the resource. The first call deletes it; subsequent calls leave it absent. |

*\*Note: In pure HTTP specification, PATCH is non-idempotent by default because operations like "append to list" mutate state incrementally, though practical REST implementations often write idempotent field updates.*

---

### 5.5 Representations & Content Negotiation
In REST, a **Resource** is an abstract conceptual entity stored in the backend (e.g., a row in a relational database). A **Representation** is the serialized format used to transfer that resource's state across the network.

*   A single resource (`/todos/123`) can have multiple representations:
    *   `application/json` (consumed by a mobile app or frontend SPA)
    *   `text/html` (rendered by a browser)
    *   `application/xml` (consumed by an enterprise legacy service)

#### Content Negotiation Headers:
Clients and servers agree upon the representation format using standard HTTP headers:
1.  **Client Request:** `Accept: application/json` ("Server, please format your response as JSON.")
2.  **Server Response:** `Content-Type: application/json` ("I am returning the representation encoded in JSON.")

---

### 5.6 The Statelessness Constraint & Scalability

Statelessness is the cornerstone enabling cloud-scale distributed systems.

```mermaid
flowchart TD
    subgraph StatefulCluster ["Stateful Architecture (Anti-Pattern for Cloud Scale)"]
        ClientA["Client A"]
        LB1["Load Balancer"]
        S1["Server 1 (Memory: Session A)"]
        S2["Server 2 (Empty)"]
        ClientA -->|Login: Session Created| LB1
        LB1 --> S1
        ClientA -.->|Next Request routed to Server 2| S2
        S2 -.->|Error: Who are you?| ClientA
    end

    subgraph StatelessCluster ["Stateless Architecture (REST Compliant)"]
        ClientB["Client B (Holds JWT Token)"]
        LB2["Load Balancer"]
        Node1["Server Node 1"]
        Node2["Server Node 2"]
        Node3["Server Node N (1000th node)"]
        ClientB -->|Request with Bearer Token| LB2
        LB2 --> Node1
        LB2 --> Node2
        LB2 --> Node3
        Note over Node1,Node3: Any node can verify token cryptographically & process request!
    end
```

*   **Stateful Design (Flawed for scale):**
    *   The server maintains a session state in local RAM.
    *   If subsequent requests hit a different server node behind a load balancer, the user is suddenly logged out unless complex "sticky sessions" or synchronized session stores are engineered.
*   **Stateless REST Design (High Scalability):**
    1.  Upon `POST /login`, the server generates a cryptographically signed token (e.g., a **JWT** / JSON Web Token) and sends it to the client.
    2.  For every subsequent request, the client attaches the token in the header:
        ```http
        Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
        ```
    3.  **Horizontal Scaling Advantage:** In a cluster of 1,000 servers, **any available server node** can validate the signature on the token and process the request immediately. No shared server session memory is required.

---

## 6. Architectural Organization: MVC Pattern for Web APIs

### 6.1 The "Big Messy File" Anti-Pattern & Separation of Concerns

When first building an API, developers often place all routes, SQL statements, business logic, and serialization inside a single file (e.g., `main.py`).

```python
# The "Big Messy File" Anti-Pattern (Violates Separation of Concerns)
@app.get("/todos")
def get_all_todos():
    # 1. Direct database connection logic
    conn = psycopg2.connect("dbname=app user=postgres")
    cur = conn.cursor()
    # 2. Hardcoded raw SQL query
    cur.execute("SELECT id, title, completed FROM todos;")
    rows = cur.fetchall()
    # 3. Manual JSON formatting & business rules
    data = [{"id": r[0], "title": r[1], "completed": bool(r[2])} for r in rows]
    # 4. HTTP response packing
    return {"status": "success", "todos": data}
```

#### Why This Fails in Production:
*   As the application grows to support `/users`, `/posts`, `/billing`, and `/notifications`, `main.py` explodes into thousands of lines of unmaintainable code.
*   Database code is tightly coupled with HTTP routing logic.
*   It is nearly impossible to write isolated unit tests without spinning up a live HTTP server and live database.
*   Violates **Separation of Concerns (SoC)**: different software responsibilities should be isolated into independent modules.

---

### 6.2 The MVC Layers Explained
To solve the "Big Messy File" problem, server engineering adopts architectural patterns. The classic pattern adapted for modern Web APIs is **MVC (Model-View-Controller)**:

```mermaid
flowchart TD
    subgraph ClientLayer ["Client Layer"]
        ClientApp["Client Application (Web / Mobile)"]
    end

    subgraph MVCLayers ["Server-Side MVC API Architecture"]
        Controller["Controller Layer (The Brain / Coordinator)\n- Parses HTTP Requests\n- Validates parameters\n- Orchestrates Model calls\n- Dispatches Views"]
        Model["Model Layer (The Data & Business Core)\n- Interacts with Database\n- Executes SQL / ORM Queries\n- Enforces domain rules\n- Oblivious to HTTP"]
        View["View Layer (The Representation Formatter)\n- Formats domain objects to JSON / DTOs\n- Sets HTTP Status Codes & Headers"]
    end

    subgraph StorageLayer ["Persistence Layer"]
        Database[(Relational Database)]
    end

    ClientApp -->|1. HTTP Request: GET /todos| Controller
    Controller -->|2. Invoke method: get_all_todos()| Model
    Model -->|3. SQL Query: SELECT * FROM todos| Database
    Database -->|4. Return Raw Table Records| Model
    Model -->|5. Return Domain Objects| Controller
    Controller -->|6. Pass data for serialization| View
    View -->|7. Return Formatted JSON Payload| Controller
    Controller -->|8. HTTP 200 OK Response| ClientApp
```

#### Component Responsibilities:
1.  **Model (M) — The Data & Business Layer:**
    *   **Sole Responsibility:** Interacting with the database, maintaining schema definitions, executing SQL/ORM queries, and enforcing core business logic (e.g., verifying user balance, enforcing password length).
    *   **HTTP Oblivious:** The Model has **no knowledge of HTTP, routes, or URLs**. It accepts parameters (e.g., `user_id: int`) and returns data objects.
2.  **Controller (C) — The Brain / Coordinator ("The Waiter"):**
    *   **Sole Responsibility:** Acting as the traffic cop.
    *   Catches the incoming HTTP request at a specific route (`/todos`).
    *   Unpacks path variables, query parameters, and headers.
    *   Calls the relevant methods on the Model (`todo_model.find_by_id(123)`).
    *   Receives domain data from the Model and delegates to the View to format the output.
3.  **View (V) — The Presentation / Representation Layer:**
    *   **Traditional Web Applications:** The View consists of HTML templates (e.g., Jinja2, Blade, Thymeleaf) containing presentation tags.
    *   **REST Web APIs:** The "consumer" is not a human eye, but another software program. Therefore, the View is the **JSON Serializer / Data Transfer Object (DTO)** layer (e.g., Pydantic models or Marshmallow schemas) that formats objects into clean JSON strings accompanied by appropriate HTTP status codes.

---

### 6.3 Step-by-Step Execution Flow in an MVC API

Here is the sequential execution breakdown of a request (`GET /todos/42`):

```mermaid
sequenceDiagram
    autonumber
    actor Client as Web Client
    participant C as Controller (TodoController)
    participant M as Model (TodoModel)
    participant DB as Database (PostgreSQL)
    participant V as View (JSON Serializer)

    Client->>C: HTTP GET /todos/42
    Note over C: Unpack request path param: id = 42
    C->>M: get_by_id(todo_id=42)
    M->>DB: SELECT * FROM todos WHERE id = 42;
    DB-->>M: Record (42, 'Study MVC', True)
    M-->>C: Returns TodoEntity object
    C->>V: serialize_todo(todo_entity)
    V-->>C: Returns JSON: {"id": 42, "title": "Study MVC"}
    C-->>Client: HTTP/1.1 200 OK + JSON Payload
```

1.  **Request Arrival:** Client issues `GET /todos/42` to the server.
2.  **Controller Routing:** The router passes the request to the `TodoController.get_todo_detail` method.
3.  **Model Invocation:** The Controller parses the integer `42` and invokes `TodoModel.get_by_id(42)`.
4.  **Database Execution:** The Model runs `SELECT * FROM todos WHERE id = 42` against the database.
5.  **Domain Return:** The database returns raw rows; the Model constructs a `Todo` domain entity and passes it back to the Controller.
6.  **View Formatting:** The Controller delivers the domain entity to the View layer to serialize it into JSON.
7.  **Response Dispatch:** The Controller builds an HTTP response with status `200 OK` and sends the JSON byte payload back to the client.

---

### 6.4 Practical Comparison: Monolithic Script vs. Clean MVC Structure

#### Architectural Benefits of MVC:
*   **Database Portability:** If you migrate from SQLite to PostgreSQL, you only modify the **Model**. The Controller and API endpoints remain completely untouched.
*   **Routing Flexibility:** If you modify your endpoint from `/todos/123` to `/api/v2/tasks/123`, you only modify the **Controller/Router**. The database queries in the Model remain untouched.
*   **Testability:** You can easily write automated unit tests for the **Model** without needing an active HTTP connection, and you can test the **Controller** by mocking the Model.

---

## Related Notes & Obsidian Links
- [[Lecture-1.pdf]] — Slide deck covering Client-Server foundations, Addressing, and HTTP.
- [[Lecture-2.pdf]] — Slide deck covering Web Services, SOAP/WSDL, REST constraints, and MVC.
