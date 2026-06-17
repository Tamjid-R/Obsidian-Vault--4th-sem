# HTTP: The Web's Language

HTTP stands for **HyperText Transfer Protocol**. It's the protocol used by your browser to request web pages from a server. It operates over the [[Transport Layer]].

---

## 1. The "Amnesiac Waiter" (Statelessness)
HTTP is **Stateless**.
*   **Feynman Term:** Imagine a waiter who forgets who you are the second he leaves your table. If you ask for water, and then ask for a glass, he doesn't remember that he already took your order for water.
*   **The Problem:** This makes things like "Shopping Carts" or "Logins" impossible because the server forgets you between clicks.
*   **The Solution: Cookies.** The server gives you a [[Loyalty Card]] (a unique ID). Every time you send a request, you show the card, and the server looks up your "file" in its database.
Q. What happens if the network connection or client crashes during a stateful transaction (at time $t'$)?
ANS: In a stateful protocol, a crash can lead to "inconsistent views" between the client and server. For example, the server might think you bought the item, but your client doesn't know it was successful. Stateless protocols avoid this by treating every request as a fresh, independent interaction.

---

### 2. Persistent vs. Non-Persistent
*   **Non-Persistent:** At most one object sent over a TCP connection; connection then closed. Requires **2 RTTs** per object + transmission time.
    *   **RTT (Round-Trip Time):** Time for a small packet to travel from client to server and back.
    *   **Handshake:** 1 RTT for TCP connection setup.
    *   **Request/Response:** 1 RTT for HTTP request and first few bytes of response.
*   **Persistent (HTTP 1.1):** Server leaves connection open after sending response; subsequent messages sent over same connection.
    *   **Pipelining:** (In 1.1) Client can send multiple requests without waiting for responses (rarely used in practice due to HOL blocking).

---

## 3. Web Caching and Proxy Servers
*   **Goal:** Satisfy client requests without involving the origin server.
*   **Why?**
    1.  Reduces response time for client.
    2.  Reduces traffic on an institution's access link.
    3.  Reduces load on origin servers.
*   **Conditional GET:**
    *   **The Problem:** Cache might have a stale version of the object.
    *   **The Mechanism:** 
        1.  Client (or proxy) includes `If-modified-since: <date>` header.
        2.  Server responds with `304 Not Modified` if the object hasn't changed.
        3.  No entity body is sent, saving bandwidth.

---

## 4. Modern HTTP: Evolution to 2 and 3

### 4.1 HTTP/2 (RFC 7540)
*   **Goal:** Reduce perceived latency by enabling full request/response multiplexing.
*   **Key Features:**
    *   **Binary Framing:** Instead of text, messages are broken into binary-encoded frames.
    *   **Request Prioritization:** Client can assign weights/dependencies to streams.
    *   **Server Push:** Server can proactively send objects it knows the client will need (e.g., CSS/JS for an HTML page).
    *   **Multiplexing:** Multiple streams over a single TCP connection.
*   **HOL Blocking:** HTTP/2 solves HOL blocking at the **Application Layer**, but if a TCP packet is lost, all streams are blocked (Transport Layer HOL blocking).

### 4.2 HTTP/3 (over QUIC)
*   **Goal:** Address Transport Layer HOL blocking and improve security/mobility.
*   **QUIC (Quick UDP Internet Connections):** 
    *   Operates over **UDP**.
    *   Combines connection setup and security handshake (TLS 1.3) into 1 RTT.
    *   **Independent Streams:** A lost packet only affects the specific stream it belonged to, eliminating HOL blocking across streams.

---

## 5. Technical Specs & Message Formats
HTTP operates on a simple client–server request/response model, typically using [[TCP]] as its transport protocol.

### 5.1 Cookies: 4 Components
1.  Cookie header line in the HTTP response message.
2.  Cookie header line in the next HTTP request message.
3.  Cookie file kept on the user's end system and managed by the user’s browser.
4.  A back-end database at the Web site.

### 5.2 Message Syntax
All messages use **CRLF** (`\r\n`) to denote line endings.


#### Request Message Format
```http
<Method> <Request-URI> HTTP/<Major>.<Minor><CRLF>
<Header-Name-1>: <value-1><CRLF>
<Header-Name-2>: <value-2><CRLF>
...
<Header-Name-N>: <value-N><CRLF>
<CRLF>
<Optional Request Body>
```
*Example (Simple GET request):*
```http
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: MyBrowser/1.0
Accept: text/html, application/xhtml+xml
Connection: keep-alive
```

#### Response Message Format
```http
HTTP/<Major>.<Minor> <Status-Code> <Reason-Phrase><CRLF>
<Header-Name-1>: <value-1><CRLF>
<Header-Name-2>: <value-2><CRLF>
...
<Header-Name-N>: <value-N><CRLF>
<CRLF>
<Optional Response Body>
```
*Example (200 OK response):*
```http
HTTP/1.1 200 OK
Date: Tue, 20 May 2025 12:34:56 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
Content-Length: 1256
Connection: keep-alive

<html>
  <head><title>Example</title></head>
  <body><h1>Welcome to Example.com</h1>...</body>
</html>
```

---

## 5. Semantics & Headers

### 5.1 Request Methods
*   **GET:** Retrieve the resource identified by the URI.
*   **POST:** Submit data to the server (e.g., form data).
*   **HEAD:** Same as GET but only fetch headers (no body).
*   **PUT/DELETE/OPTIONS:** Used for resource modification, removal, or querying server capabilities.

### 5.2 Key Header Fields
*   **Host:** Required in HTTP/1.1. Enables **Virtual Hosting** (multiple domains on one IP).
*   **User-Agent:** Identifies client software (browser version, OS).
*   **Accept:** Indicates media types (MIME) the client is willing to accept (Content Negotiation).
*   **Connection:** Controls persistence (`keep-alive` vs `close`).
*   **Content-Length:** Number of bytes in the message body.
*   **Transfer-Encoding: chunked:** Indicates the response is sent in chunks, each beginning with its length in hex.

### 5.3 Status Codes
*   **1xx (Informational):** e.g., `100 Continue`.
*   **2xx (Success):** e.g., `200 OK`, `201 Created`.
*   **3xx (Redirection):** e.g., `301 Moved Permanently`, `302 Found`. Requires `Location` header.
*   **4xx (Client Error):** e.g., `400 Bad Request`, `401 Unauthorized`, `404 Not Found`.
*   **5xx (Server Error):** e.g., `500 Internal Server Error`, `503 Service Unavailable`.

---

## 6. Process Rules & Behavior

### 6.1 Client Behavior
1.  **Establish TCP Connection:** Open socket to server IP on port 80 (HTTP) or 443 (HTTPS).
2.  **Send Request:** Compose request line, add mandatory headers (Host), terminate with blank line, and send body (if any).
3.  **Wait for Response:** Read status line, then headers, then determine body length (via `Content-Length` or `Transfer-Encoding`).
4.  **Connection Management:** Close socket if `Connection: close` is specified; otherwise, reuse for persistent connections.

### 6.2 Server Behavior
1.  **Accept Connection:** Listen on port 80/443.
2.  **Read Request:** Parse request line and headers. Validate `Host` header (must exist in 1.1).
3.  **Process Request:** Look up resource.
    *   If found: Generate `200 OK` and determine `Content-Type`.
    *   If not found: Respond with `404 Not Found`.
    *   On error: Respond with `500 Internal Server Error`.
4.  **Send Response:** Write status line, mandatory headers (Date, Server, Content-Type/Length), blank line, and body.

---

## 7. Real-World Examples
*   **Web Browsing:** When you enter `google.com`, your browser sends a `GET` request. The server responds with `200 OK` and the HTML/CSS/JS files needed to render the page.
*   **Form Submission:** When you log into a site, your browser likely sends a `POST` request containing your username and password in the message body.
*   **API Interactions:** RESTful APIs often use `PUT` to update a record or `DELETE` to remove one.
