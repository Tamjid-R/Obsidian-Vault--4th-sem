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

## 2. Persistent vs. Non-Persistent
*   **Non-Persistent:** Like a one-question-per-phone-call rule. You call, ask for the HTML, and hang up. Then you call again for the first image, and hang up. It's slow because each call ([[TCP]] connection) takes time to set up (**2 RTTs** per object).
*   **Persistent (HTTP 1.1/2.0):** Like staying on the line. You open one connection and ask for everything you need. It’s much faster.

---

## 3. The "Library Bookshelf" (Caching)
*   **Feynman Term:** Instead of driving to the National Library (Origin Server) every time you want a book, you check the **Little Free Library** on your street corner (Web Cache/Proxy Server).
*   **Why?** It's faster for you and reduces traffic on the main highway.
*   **Conditional GET:** Your browser asks the server: "I have a copy of this book from Tuesday. Has it been updated?" If not, the server says "304 Not Modified," and you just use your old copy. No data wasted!

---

## 4. Technical Specs & Message Formats
HTTP operates on a simple client–server request/response model, typically using [[TCP]] as its transport protocol.

### 4.1 Message Syntax
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
