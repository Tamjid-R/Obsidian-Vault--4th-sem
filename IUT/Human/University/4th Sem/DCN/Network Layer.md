# Network Layer: Host-to-Host Communication

The Network Layer is responsible for transporting segments from a sending host to a receiving host. Unlike the [[Transport Layer]], which runs only in end systems, network layer protocols run in **every host and router** in the network.

---

## 1. Key Functions and Planes

### 1.1 Forwarding vs. Routing
*   **Forwarding (Data Plane):** The local, per-router function of moving packets from a router's input port to the appropriate output port.
    *   *Analogy:* The process of getting through a single interchange on a trip.
*   **Routing (Control Plane):** The network-wide logic that determines the end-to-end path taken by packets from source to destination.
    *   *Analogy:* The process of planning the entire trip from start to finish.

### 1.2 Data Plane vs. Control Plane
*   **Data Plane:** Determines how a datagram arriving on a router input port is forwarded to an output port.
*   **Control Plane:** Determines how datagrams are routed among routers along the end-to-end path.
    *   **Traditional Approach:** Routing algorithms implemented in each router.
    *   **Software-Defined Networking (SDN):** Routing logic implemented in a remote, centralized server (SDN Controller).

---

## 2. IP: The Internet Protocol

### 2.1 IPv4 Datagram Format
A typical IPv4 header is **20 bytes**.
*   **Ver:** IP protocol version (4).
*   **Head. Len:** Header length (bytes).
*   **TOS (Type of Service):** Used for DiffServ and ECN.
*   **Length:** Total datagram length (header + data).
*   **16-bit Identifier/Flags/Fragment Offset:** Used for fragmentation and reassembly.
    *   **Offset Calculation:** $\text{Offset Field Value} = \frac{\text{Bytes from start of original data}}{8}$.
    *   **Flags:** MF (More Fragments) bit. $MF=1$ means more fragments are coming; $MF=0$ indicates the last fragment.
*   **TTL (Time to Live):** Remaining max hops (decremented at each router; prevents infinite loops).
*   **Upper Layer:** Protocol used in data (e.g., 6 for TCP, 17 for UDP).
*   **Header Checksum:** Detects header errors.
*   **Source/Dest IP Address:** 32-bit identifiers.

**Detailed Example: IP Fragmentation**
Imagine a 4000-byte datagram (20 bytes header + 3980 bytes data) arrives at a router with an **MTU of 1500 bytes**.

1.  **Max Data per Fragment:** $1500 - 20 = 1480$ bytes. 
    *   *Check:* 1480 is divisible by 8 ($1480 / 8 = 185$). This is required for the offset field.
2.  **Fragment 1:**
    *   **Length:** 1500 bytes (20 header + 1480 data).
    *   **ID:** 777 (same as original).
    *   **Offset:** 0.
    *   **MF:** 1 (More fragments follow).
3.  **Fragment 2:**
    *   **Length:** 1500 bytes (20 header + 1480 data).
    *   **ID:** 777.
    *   **Offset:** 185 (Calculated as $1480 / 8$).
    *   **MF:** 1.
4.  **Fragment 3:**
    *   **Length:** 1040 bytes (20 header + 1020 data).
    *   *Calculation:* $3980 - (1480 \times 2) = 1020$ bytes of remaining data.
    *   **ID:** 777.
    *   **Offset:** 370 (Calculated as $(1480 \times 2) / 8$).
    *   **MF:** 0 (Last fragment).

**Numerical Example: Overhead**
*   20 bytes TCP + 20 bytes IP = 40 bytes minimum total overhead (plus application layer overhead).

### 2.2 IPv4 Addressing
*   **IP Address:** A 32-bit identifier for a host or router **interface**.
*   **Interface:** The connection between a host/router and a physical link. Routers typically have multiple interfaces.
*   **Dotted-Decimal Notation:** e.g., `223.1.1.1`
    *   Binary: `11011111 00000001 00000001 00000001`

### 2.3 Subnets
A **subnet** consists of device interfaces that can physically reach each other without passing through an intervening router.
*   **Subnet Mask:** Indicates the network portion of the address (e.g., `/24` means the first 24 bits are the subnet part).
*   **CIDR (Classless InterDomain Routing):** Address format `a.b.c.d/x`, where `x` is the number of bits in the subnet portion.
*   **Subnet Math:**
    *   **Number of Addresses:** $2^{32-x}$ (Total addresses including network and broadcast).
    *   **Example:** `/23` subnet has $2^{32-23} = 2^9 = 512$ addresses.

**Numerical Example: ISP Address Allocation**
An ISP is allocated a block `200.23.16.0/20`. It can divide this into 8 smaller blocks of `/23` for different organizations:
*   **Organization 0:** `200.23.16.0/23`
*   **Organization 1:** `200.23.18.0/23`
*   **Organization 2:** `200.23.20.0/23`
*   ...
*   **Organization 7:** `200.23.30.0/23`
*   *Calculation:* Borrowing 3 bits from the host portion ($23 - 20 = 3$) allows for $2^3 = 8$ subnets.

---

## 3. Dynamic Host Configuration Protocol (DHCP)
**Goal:** Allow a host to dynamically obtain an IP address from a network server when it joins the network.

### 3.1 DHCP 4-Step Process
1.  **DHCP Discover:** Client broadcasts "Is there a DHCP server?" (`dest: 255.255.255.255`).
2.  **DHCP Offer:** Server responds with an available IP address.
3.  **DHCP Request:** Client broadcasts "I'd like to use this IP."
4.  **DHCP ACK:** Server confirms "You've got that IP."

*Note: DHCP can also return the address of the first-hop router, DNS server name/IP, and the network mask.*

---

## 4. NAT: Network Address Translation
**Goal:** Allow all devices in a local network to share just **one** public IPv4 address.

### 4.1 Implementation
*   **Outgoing Datagrams:** The NAT router replaces the (Private IP, Port) with (Public NAT IP, New Port).
*   **NAT Translation Table:** The router remembers these translation pairs.
*   **Incoming Datagrams:** The router uses the table to replace the (Public NAT IP, New Port) in the destination field with the original (Private IP, Port).

**Numerical Example: NAT Translation Table**
| WAN Side Addr (Public) | LAN Side Addr (Private) |
| :--- | :--- |
| 138.76.29.7, 5001 | 10.0.0.1, 3345 |
| 138.76.29.7, 5002 | 10.0.0.2, 3345 |

### 4.2 Advantages
*   Saves IPv4 addresses (only one needed from ISP).
*   Change internal IP addresses without notifying the outside world.
*   Security: Internal devices are not directly addressable from the outside.

---

## 5. IPv6
**Motivation:** IPv4 address space exhaustion (32-bit limit).

### 5.1 IPv6 Datagram Format
*   **Fixed 40-byte Header:** Speeds up processing at routers.
*   **128-bit Addresses:** Huge address space.
*   **Flow Label:** Identifies datagrams in the same "flow."
*   **No Checksum:** Removed to speed up processing (rely on transport/link layer checksums).
*   **No Fragmentation:** Routers no longer fragment; it's handled by end systems.

### 5.2 Transition: Tunneling
Since not all routers can be upgraded at once, **tunneling** is used: an IPv6 datagram is carried as the payload of an IPv4 datagram to travel through IPv4-only routers.

---

## 6. Real-World Examples
*   **Home Wi-Fi:** Your home router uses **DHCP** to give your phone an internal address (like `192.168.1.5`) and uses **NAT** so all your devices can access the internet using one public IP provided by your ISP.
*   **Route Aggregation:** A "Fly-By-Night-ISP" advertises a single prefix `200.23.16.0/20` to the Internet. This single advertisement covers multiple organizations (Org 0 to Org 7) that have their own `/23` blocks.
*   **Longest Prefix Match:** If Organization 1 (`200.23.18.0/23`) moves to a different ISP, the new ISP will advertise the specific `/23` route. Routers will prefer this more specific route over the aggregate `/20` route due to the **Longest Prefix Match** rule.
*   **Modern Cellular Networks (4G/5G):** Extensively use **IPv6** and **Tunneling** to manage the massive number of connected mobile devices.
