# DNS: The Internet's Phonebook

DNS stands for **Domain Name System**. It translates human-friendly names (like `google.com`) into computer-friendly IP addresses (like `142.250.190.46`).
Q. How to map between IP address and name, and vice versa?
ANS: This is performed through the DNS hierarchy where a client queries various name servers. It maps hostnames to IP addresses using **A records**, and can perform the reverse mapping using **PTR records**.

---

## 1. Why not one big Phonebook? (Scalability)
Q. Why not centralize DNS?
ANS: A centralized DNS does not scale for the entire planet. It would create a **Single Point of Failure** (the whole internet goes down if it breaks), **Traffic Volume** (one machine cannot handle trillions of queries), **Physical Distance** (signal travel time to a distant server), and **Maintenance** (it’s impossible to manage every record on one machine).
If there was only one DNS server in the world:
...
1.  **Single Point of Failure:** If it breaks, the whole Internet "vanishes" (you can't find anything).
2.  **Traffic Jam:** Billions of people asking one machine at the same time.
3.  **Distance:** If the server is in New York, someone in Tokyo has to wait for the signal to travel across the ocean just to look up a name.

---

## 2. The Hierarchy (The Chain of Command)
DNS is a **Distributed Database**. If your local server doesn't know a name, it asks up the chain:
1.  **Root Servers:** The "Grandmasters." They don't know the IP, but they know where the `.com` or `.org` books are.
2.  **TLD Servers (Top-Level Domain):** They know where the specific "Authority" for `google.com` is.
3.  **Authoritative Servers:** These belong to the organization (like Google) and have the final, correct answer.

---

## 3. Iterative vs. Recursive Queries
*   **Iterative (The "Ask-Around"):** You ask the Root, it says "I don't know, ask the .com guy." You then ask the .com guy yourself.
*   **Recursive (The "Do-It-For-Me"):** You ask your local server, and *it* does all the calling until it finds the answer for you.

---

## 4. Technical Specs
*   **Transport Protocol:** UDP (for speed)
*   **Port Number:** 53
*   **Records (RRs):**
    *   **Type A:** Name = Hostname, Value = IP Address.
    *   **Type NS:** Name = Domain, Value = Hostname of authoritative server.
    *   **Type CNAME:** Name = Alias, Value = Real (Canonical) name.
    *   **Type MX:** Name = Domain, Value = Mail server name.
