# SMTP & IMAP: The Post Office

Email uses a combination of protocols to **Push** and **Pull** messages across the world.

---

## 1. SMTP: The "Push" Protocol
SMTP stands for **Simple Mail Transfer Protocol**.
*   **Feynman Term:** It’s like a delivery truck. You use SMTP to **push** your letter from your computer to your local post office (Mail Server), and the post office uses SMTP to **push** it to the recipient's post office.
*   **Direct Transfer:** The sending server talks directly to the receiving server. No "middle-man" servers involved.
*   **The Language:** It uses 7-bit ASCII. If you send a photo, it has to be encoded into text characters first!

---

## 2. Mail Access Protocols (Pulling Mail)
Once mail is at the recipient's server, a "pull" protocol is needed to retrieve it.

### A. POP3 (Post Office Protocol - Version 3)
*   **Mode:** "Download and Delete" (default) or "Download and Keep".
*   **State:** Stateless across sessions. If you move an email to a folder on one device, it doesn't reflect on others.
*   **Port:** TCP 110.

### B. IMAP (Internet Mail Access Protocol)
*   **Mode:** Keeps all messages on the server.
*   **State:** Stateful. Synchronizes folders and read/unread status across all devices.
*   **Port:** TCP 143.

### C. HTTP (Webmail)
*   **Usage:** Gmail, Outlook.com use HTTP for the user-to-server interface. SMTP is still used between mail servers.

---

## 3. Technical Specs (SMTP)
*   **Transport Protocol:** TCP (Reliability is key!)
*   **Port Number:** 25
*   **Comparison with HTTP:**
    *   Both use persistent connections.
    *   HTTP is a "pull" protocol; SMTP is a "push" protocol.
    *   SMTP requires 7-bit ASCII encoding for binary data; HTTP does not.
    *   HTTP encapsulates each object in its own response message; SMTP sends all objects in one message.

---

## 4. SMTP Phases
1.  **Handshaking:** (HELO/EHLO)
2.  **Transfer of messages:** (MAIL FROM, RCPT TO, DATA)
3.  **Closure:** (QUIT)

