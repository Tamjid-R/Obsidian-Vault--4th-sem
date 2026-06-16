# SMTP & IMAP: The Post Office

Email uses a combination of protocols to **Push** and **Pull** messages across the world.

---

## 1. SMTP: The "Push" Protocol
SMTP stands for **Simple Mail Transfer Protocol**.
*   **Feynman Term:** It’s like a delivery truck. You use SMTP to **push** your letter from your computer to your local post office (Mail Server), and the post office uses SMTP to **push** it to the recipient's post office.
*   **Direct Transfer:** The sending server talks directly to the receiving server. No "middle-man" servers involved.
*   **The Language:** It uses 7-bit ASCII. If you send a photo, it has to be encoded into text characters first!

---

## 2. IMAP: The "Pull" Protocol
IMAP stands for **Internet Mail Access Protocol**.
*   **Feynman Term:** Once your letter arrives at the recipient's post office, it sits in their **mailbox** (on the server). The recipient uses IMAP to **pull** the mail onto their phone or laptop to read it.
*   **Syncing:** IMAP keeps your folders (Inbox, Sent, Trash) the same across all your devices. If you delete an email on your phone, it disappears on your laptop too.

---

## 3. HTTP for Email (Webmail)
Modern services like Gmail use **HTTP** as the "user-to-server" interface. You use a browser to talk to Gmail's server via HTTP, but behind the scenes, Gmail's server still uses **SMTP** to talk to Outlook's server.

---

## 4. Technical Specs (SMTP)
*   **Transport Protocol:** TCP (Reliability is key!)
*   **Port Number:** 25
*   **Three Phases:**
    1.  **Handshaking:** "Hello, I am server A." "Hello, I am server B. Send your mail."
    2.  **Transfer:** The actual message is sent.
    3.  **Closure:** Saying goodbye and hanging up.
