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
*   **Recursive Query:** Puts the burden of name resolution on the contacted name server (e.g., your host asking a local DNS server).
*   **Iterative Query:** The contacted server replies with the name of the next DNS server in the chain ("I don't know this name, but ask this server").

---

## 4. DNS Caching and Performance
*   **DNS Caching:** Once a DNS server receives a mapping, it caches the mapping in its local memory. 
*   **TTL (Time to Live):** The length of time a mapping is cached before being discarded.
*   **Effect:** Reduces the need for queries to propagate up the hierarchy; often, root servers are bypassed because TLD servers are cached.

---

## 5. Local DNS Servers
*   **Role:** Each ISP (or organization) has a **Local DNS Server** (also called a default name server).
*   **Function:** When a host makes a DNS query, the query is sent to its local DNS server, which acts as a proxy and forwards the query into the DNS server hierarchy.

---

## 6. Technical Specs
*   **Transport Protocol:** UDP (for speed, Port 53). TCP is used for large responses or zone transfers.
*   **Resource Records (RR):** A four-tuple: `(Name, Value, Type, TTL)`.
    *   **Type A:** Name = Hostname, Value = IP Address.
    *   **Type NS:** Name = Domain, Value = Hostname of authoritative server for this domain.
    *   **Type CNAME:** Name = Alias, Value = Real (Canonical) name.
    *   **Type MX:** Name = Domain, Value = Hostname of mail server associated with name.

---

## 7. DNS Security
*   **DDoS Attacks:** Bombarding root or TLD servers with traffic. Mostly unsuccessful due to caching and distribution.
*   **Redirect Attacks:** 
    *   **Man-in-the-middle:** Intercepting queries and returning false IPs.
    *   **DNS Poisoning:** Sending bogus RRs to a DNS server to corrupt its cache.
*   **DNSSEC:** Uses digital signatures to provide authentication and integrity to DNS data.
