# Chapter 6 — The Link Layer and LANs (§6.1–6.4)

Kurose & Ross, *Computer Networking: A Top-Down Approach*, 8th ed. Covers §6.1 (Introduction to the Link Layer), §6.2 (Error Detection/Correction), §6.3 (Multiple Access Links and Protocols), §6.4 (Switched LANs), plus every Chapter 6 review question and problem that tests those sections (R1–R16, P1–P26). Excluded as out of scope: §6.5 Link Virtualization/MPLS, §6.6 Data Centers, §6.7 "Day in the Life," and problems P27 (VoIP packetization — a general delay/overhead problem, not specific to these sections), P29–P30 (MPLS), P31 (whole-book synthesis).

---

## 6.1 Introduction to the Link Layer

**Terminology:** a **node** = any device running a link-layer (layer-2) protocol — hosts, routers, switches, WiFi access points. A **link** = the communication channel connecting adjacent nodes. A datagram must be moved over *every* individual link on its end-to-end path; over each link, the transmitting node encapsulates the datagram in a **link-layer frame**.

**Transportation analogy:** datagram = tourist; each link = one transportation segment (limousine, plane, train); a link-layer protocol = the transportation mode; a routing protocol = the travel agent planning the overall trip. Each segment is "direct" between two adjacent points, run by a different operator, using a different mode — yet each just moves the passenger from one point to the next adjacent point.

### 6.1.1 Services the Link Layer Can Offer
 
- **Framing** — encapsulate each network-layer datagram in a link-layer frame (header fields + data field) before transmission; frame format is protocol-specific.
- **Link access** — a **MAC (medium access control)** protocol governs *when* a frame may be sent onto the link. Trivial/nonexistent for point-to-point links (send whenever idle); the interesting case is multiple nodes sharing one broadcast link (§6.3).
- **Reliable delivery** — via ACKs/retransmissions (like TCP, §3.4), used on high-error links (e.g. wireless) to fix errors *locally* rather than forcing an end-to-end retransmission. Considered unnecessary overhead on low-bit-error links (fiber, coax, twisted-pair) — most wired link protocols skip it.
- **Error detection and correction** — bits get flipped by attenuation/noise; the transmitter includes error-detection bits, the receiver checks them. Usually more sophisticated than transport/network-layer checksums, and implemented in hardware. *Correction* additionally pinpoints *where* the error is (not just *that* one occurred).

### 6.1.2 Where the Link Layer Lives
Mostly in hardware, on a chip called the **network adapter** / **NIC (network interface controller)** — it implements framing, link access, error detection, etc. A thin software layer (running on the host CPU) handles higher-level functions: assembling addressing info, activating the controller, responding to interrupts on receipt, handling errors, passing datagrams up to the network layer. **The link layer is where software meets hardware.**

---

## 6.2 Error-Detection and -Correction Techniques

**Setup (Figure 6.3):** sender augments data *D* with error-detection/correction bits *EDC*, sends both; receiver gets (possibly corrupted) *D′* and *EDC′*. Crucially, the receiver can only ask "**was an error detected?**" — not "did an error occur?" — some corrupted frames slip through undetected. More powerful schemes catch more errors but cost more overhead bits/computation.

### 6.2.1 Parity Checks
**Single-bit parity:** append 1 bit so the total count of 1s among the *d+1* bits is even (even parity) or odd (odd parity). Catches any **odd** number of bit errors; an **even** number of errors is invisible (undetected). Real-world bit errors are often **bursty**, and under burst conditions single-bit parity's undetected-error probability can approach **50%** — too weak alone.

**Two-dimensional parity (Figure 6.5):** arrange the *d* bits into *i* rows × *j* columns; compute a parity bit for every row **and** every column → *i+j+1* total parity bits. Now a **single-bit error** flips exactly one row-parity and one column-parity check — the receiver can pinpoint *and correct* it from the (row, column) coordinates. Two-dimensional parity can also **detect** (but not correct) any 2-bit-error combination. This "detect-and-correct" capability is called **forward error correction (FEC)** — valuable because it fixes errors immediately at the receiver without waiting a full RTT for a NAK+retransmission (important for real-time apps / long-propagation-delay links like deep space).

### 6.2.2 Checksumming Methods
Treat the *d* data bits as a sequence of *k*-bit integers and sum them (the **Internet checksum**: sum 16-bit words, take the 1's complement). Receiver re-sums including the checksum field and checks the result is all 0s. Cheap (16 bits of overhead in TCP/UDP) but **weaker protection** than CRC — used at the transport layer specifically *because* it's implemented in host software (needs to be fast/simple), whereas CRC (more complex, more robust) is used at the link layer where it's implemented in dedicated hardware that can afford the extra computation.

### 6.2.3 Cyclic Redundancy Check (CRC)
Sender and receiver agree on an *(r+1)*-bit **generator** *G* (leftmost bit = 1). For *d*-bit data *D*, the sender picks *r* extra bits *R* so that the *(d+r)*-bit pattern `D·2ʳ XOR R` is **exactly divisible by G** under **modulo-2 arithmetic** (no carries/borrows — addition = subtraction = bitwise XOR). Then:

**R = remainder of (D · 2ʳ) / G** (mod-2 division)

Receiver divides the received *(d+r)* bits by *G*; nonzero remainder ⇒ error detected. **CRC-32** (used in many IEEE link protocols) uses a fixed 33-bit generator. Guarantees: detects **any burst error < r+1 bits**, detects a longer burst with probability **1 − 0.5ʳ**, and detects **any odd number of bit errors**.

---

## 6.3 Multiple Access Links and Protocols

Two link types: **point-to-point** (one sender, one receiver — simple/no MAC needed, e.g. PPP) vs. **broadcast** (many nodes share one channel — Ethernet, WiFi; whatever one node sends, everyone else's adapter receives). The **multiple-access problem**: coordinate who gets to transmit and when, so simultaneous transmissions ("**collisions**," which garble all involved frames and waste the channel) are managed.

**Four desirable properties of a multiple access protocol** (rate *R* channel): (1) sole active node gets throughput *R*; (2) *M* active nodes each average *R/M*; (3) fully **decentralized** — no single point of failure; (4) **simple**/cheap to implement.

Three broad protocol families:

### 6.3.1 Channel Partitioning Protocols
- **TDM** — divide time into frames of *N* slots, one slot per node; collision-free and perfectly fair, but a node is capped at *R/N* even if it's the *only* active node, and must always wait its turn even when idle.
- **FDM** — same idea, split by frequency instead of time; same pros/cons as TDM.
- **CDMA** — assign each node a distinct **code**; nodes can transmit *simultaneously*, and (with well-chosen codes) receivers decode correctly despite the interference. Used in military (anti-jam) and cellular telephony.

### 6.3.2 Random Access Protocols
No pre-allocated turns — a node transmits at the **full rate R**; on collision, it retransmits after an independently-chosen **random delay**.

**Slotted ALOHA:** time divided into *L/R*-second slots (one frame's transmit time); nodes transmit only at slot boundaries, and on collision retransmit each subsequent slot with probability *p* (coin flip). With *N* saturated nodes each transmitting w.p. *p* per slot, efficiency = **Np(1−p)^(N−1)**; maximizing over *p* gives *p\* = 1/N*, and as *N→∞* the max efficiency → **1/e ≈ 0.37** — i.e. at best only 37% of slots do useful work (also 37% idle, 26% collide).

**Pure (unslotted) ALOHA:** no slot synchronization — transmit immediately on arrival; on collision, retransmit w.p. *p* after one frame-time, else wait another frame-time. Because a frame can now collide with *anything* starting in a full frame-time window before **or** after it begins, the vulnerable period doubles → efficiency = **Np(1−p)^(2(N−1))**, max efficiency → **1/(2e) ≈ 0.18** — exactly half of slotted ALOHA (the cost of full decentralization/no slot sync).

**CSMA (Carrier Sense Multiple Access):** "listen before speaking" — sense the channel; only transmit if idle. Collisions can *still* happen because of nonzero **propagation delay**: a node can sense "idle" and start transmitting before an already-in-flight signal from a far-away node has physically reached it yet (Figure 6.12 space-time diagram) — the larger the propagation delay relative to frame time, the worse this gets.

**CSMA/CD (with Collision Detection):** additionally, "stop talking if someone else starts too" — a transmitting node keeps listening, and **aborts immediately** on detecting another signal, instead of wasting the channel transmitting a doomed frame to completion.
*Adapter algorithm:* (1) get datagram from network layer, frame it; (2) if channel idle, start sending; else wait for idle, then start; (3) monitor for other signal energy while sending; (4) if the whole frame goes out with no interference detected, done; if interference detected mid-send, **abort**; (5) after aborting, wait a random time, then go back to (2).

**Binary exponential backoff:** after a frame has suffered *n* collisions, choose *K* uniformly from `{0, 1, …, 2ⁿ−1}`, and wait `K × 512 bit times` (Ethernet); *n* is capped at 10 (so *K* ranges over at most `{0,…,1023}`). Growing the range exponentially with collision count keeps the wait short when few nodes are colliding and long when many are. Each **new** frame restarts the algorithm fresh, ignoring past collision history — so a freshly-arriving frame can "sneak in" a successful send while others sit in backoff.

**CSMA/CD efficiency** (large number of saturated nodes): `Efficiency = 1 / (1 + 5·d_prop/d_trans)`, where *d_prop* = max propagation delay between any two adapters, *d_trans* = time to send a max-size frame. → efficiency → 1 as *d_prop* → 0 or as *d_trans* grows large (overhead amortized away).

### 6.3.3 Taking-Turns Protocols
Trying to get *both* good properties random access lacks (idle-node-gets-full-rate *and* fair M-way sharing):
- **Polling** — a **master node** cyclically invites each node to send up to some max number of frames. Eliminates collisions/empty slots (high efficiency), but introduces **polling delay**, and the master is a **single point of failure**. (Bluetooth uses polling.)
- **Token passing** — a small **token** frame circulates node-to-node in a fixed order; holding the token = permission to send (up to a max), then pass it on; if nothing to send, forward it immediately. No master node (decentralized), but one node's failure can crash the whole ring, and a lost token needs a recovery procedure. (FDDI, IEEE 802.5 token ring.)

### 6.3.4 DOCSIS (Cable Access Link Layer)
A cable network is a case study using **all three** MAC families at once. DOCSIS uses **FDM** to split downstream (CMTS→modem) and upstream (modem→CMTS) traffic into separate frequency channels (downstream ≈ up to 1.6 Gbps/channel; upstream ≈ up to 1 Gbps/channel). **Downstream has no multiple-access problem** (only the CMTS transmits). **Upstream** is shared by many modems, so it's divided **TDM-style** into **mini-slots**; the CMTS explicitly grants specific mini-slots to specific modems via a downstream **MAP message** (collision-free once granted). But modems must first *request* a mini-slot, and those **mini-slot-request frames are sent random-access style** (can collide); a modem infers a collision if it gets no grant in the next MAP, and then applies **binary exponential backoff** before retrying.

---

## 6.4 Switched Local Area Networks

### 6.4.1 Link-Layer Addressing and ARP

**MAC addresses:** it's really the *adapter*, not the host/router, that has the address. 6 bytes long (2⁴⁸ possible), written in hex pairs, **flat** (non-hierarchical) and permanent-by-design (unlike a hierarchical, movement-dependent IP address) — analogy: MAC address ≈ social security number, IP address ≈ postal address. **Global uniqueness** is administered by the IEEE: a manufacturer buys a block of 2²⁴ addresses (fixing the first 24 bits) and assigns the remaining 24 bits itself. The **broadcast address** is `FF-FF-FF-FF-FF-FF`. An adapter only passes a frame's contents up the stack if the destination MAC matches its own address **or** is the broadcast address — otherwise it silently discards the frame (so only the intended destination gets interrupted).

*Why two layers of addressing at all?* (Principles in Practice sidebar) So layers stay independent: LANs must support network-layer protocols other than IP; storing/reconfiguring a network address inside adapter hardware on every move/power-up would be clunky; and without *some* filtering address, every host would be interrupted by every frame on the LAN.

**ARP (Address Resolution Protocol, RFC 826)** resolves an IP address → MAC address, **only for nodes on the same subnet** (unlike DNS, which is global) — an ARP query for an off-subnet address just errors out. Each host/router keeps an **ARP table** (IP↔MAC↔TTL, entries typically expire ~20 min). If the destination isn't in the table: the sender broadcasts an **ARP query packet** (dest MAC = broadcast) asking "who has IP X?"; every adapter on the subnet passes it up to its ARP module; only the matching node replies, with a **unicast** ARP response containing its MAC. The querying host then caches the mapping and sends the actual frame.

Two things worth remembering: (1) query is **broadcast**, response is **unicast** (only the target needs to reply — no need to interrupt everyone else a second time); (2) ARP is **plug-and-play** — tables build/expire automatically, no admin configuration. ARP doesn't fit neatly in one layer — its packet is carried inside a link-layer frame but contains both link- and network-layer addresses, so it's best thought of as straddling the link/network boundary.

**Sending off-subnet (Figure 6.19):** if source and destination are on *different* subnets, the source can't just use the destination's MAC — it must send the frame to its **first-hop router's** MAC address instead (obtained via ARP for the router's own on-subnet interface IP), even though the *IP* datagram's destination address is still the ultimate off-subnet host. The router receives the frame (its own MAC matched), strips it, consults its **forwarding table** to pick the correct outgoing interface, and re-encapsulates the *same* IP datagram into a *brand-new* frame — this time addressed (via a fresh ARP lookup on the destination subnet) to the actual destination's MAC. **Every hop gets a completely new link-layer frame**, even though the IP datagram inside is unchanged end to end.

### 6.4.2 Ethernet

Dominant wired LAN tech since the mid-1970s (Metcalfe & Boggs) — survived challenges from token ring, FDDI, ATM thanks to early-mover familiarity, simplicity/cost, and continually matching or beating rival data rates.

**Evolution of topology:** coax **bus** (1980s–mid-1990s, a true broadcast medium, needed CSMA/CD) → **hub-based star** (hub = dumb physical-layer repeater, re-broadcasts every bit to every other port — *still* a broadcast/collision domain) → **switch-based star** (today's norm — a switch buffers and forwards **frames**, not bits, and is "collision-less"; unlike a router it only processes up through **layer 2**).

**Ethernet frame fields (Figure 6.20):**

| Field | Size | Purpose |
|---|---|---|
| Preamble | 8 bytes | First 7 bytes = `10101010` (clock sync for the receiver, since sender's actual rate always drifts slightly from nominal); last byte = `10101011` (last two 1-bits signal "real content starts now") |
| Dest. address | 6 bytes | Receiving adapter's MAC (or broadcast) |
| Source address | 6 bytes | Sending adapter's MAC |
| Type | 2 bytes | Demuxes to the right network-layer protocol (IP, ARP=`0806`, IPX, AppleTalk…) — same role as IP's protocol field / TCP-UDP's port fields |
| Data | 46–1500 bytes | The IP datagram (MTU = 1500 bytes ⇒ triggers IP fragmentation if exceeded; minimum 46 bytes ⇒ short datagrams get **padded/"stuffed"**, and IP's own length field tells the network layer where the real data ends) |
| CRC | 4 bytes | Error detection (§6.2.3) |

Ethernet gives **connectionless, unreliable** service to the network layer: no handshake before sending, and a frame that fails its CRC check is just silently **dropped** — no ACK, no NAK. Whether the *application* ever sees a resulting gap depends on the transport protocol: UDP → gap is visible to the app; TCP → TCP's own end-to-end ACK/retransmission machinery re-sends the lost data (Ethernet itself has no idea it's "retransmitting" — from its perspective every frame is new).

**Technology naming convention** (e.g. `100BASE-TX`, `10GBASE-T`): first number = speed (10/100/1000/10G/40G Mbps-ish), "BASE" = baseband (medium carries only Ethernet traffic), final letters = physical medium ("T" ≈ twisted-pair copper). Across all these variants — 10 Mbps to 40+ Gbps, coax to copper to fiber — **the Ethernet frame format itself has stayed constant for 30+ years**; arguably *the* one true timeless centerpiece of the standard. In a fully switched, full-duplex modern Ethernet, there are **no collisions at all anymore** — so, strictly, no MAC protocol is even needed — yet it's still called Ethernet "by definition."

### 6.4.3 Link-Layer Switches

A switch is transparent to hosts (they address frames to each other, not to the switch) and does **filtering** (should this frame go anywhere, or get dropped?) and **forwarding** (which interface(s) should it go out?) via a **switch table**: entries of `(MAC address, interface, timestamp)`.

**Lookup logic** for a frame with dest address D arriving on interface *x*:
- **No entry for D** → **flood** (forward out every interface except *x*).
- **Entry says D is on interface x itself** → **filter** (drop — the destination is already on the same segment the frame came from, so it's already been delivered/no need to forward).
- **Entry says D is on interface y ≠ x** → **forward** only to *y*.

**Self-learning** (fully automatic, zero admin configuration): table starts empty; on **every** incoming frame, the switch records `(source MAC, arrival interface, current time)` — this is how it learns *where* senders live. Stale entries (no frame seen from that source in the **aging time**) are eventually purged (handles a PC being swapped out, etc.). Switches are **plug-and-play** and **full-duplex** (send+receive simultaneously on any port).

**Properties/benefits:** eliminates collisions entirely (aggregate throughput = sum of interface rates, like a router); isolates links so **different links can run different speeds/media** (mixing legacy and new gear); eases management (detects/isolates a "jabbering" malfunctioning adapter, a single cable cut only drops one host, gathers usage statistics) — plus a security bonus over hubs (a sniffer normally only sees frames actually addressed to it or genuinely broadcast, though a **switch poisoning** attack — flooding bogus source addresses to overflow the table — can force fallback-to-flooding and defeat this).

**Switches vs. routers (Table 6.1):**

| | Hubs | Routers | Switches |
|---|---|---|---|
| Traffic isolation | No | Yes | Yes |
| Plug and play | Yes | No | Yes |
| Optimal routing | No | Yes | No (restricted to a spanning tree) |

Switches: cheap/fast (only process up through layer 2), but the active topology must be restricted to a **spanning tree** to prevent broadcast frames cycling forever, large flat switched networks mean big ARP tables/traffic, and they're vulnerable to **broadcast storms** (one haywire host flooding broadcasts brings down the whole network). Routers: hierarchical addressing avoids cycling without a spanning-tree restriction (so richer topologies — e.g. multiple active transatlantic links — are possible), and they firewall off layer-2 broadcast storms — but aren't plug-and-play (IP addresses must be configured) and have higher per-packet processing cost (layer 3). Rule of thumb: small networks (few hundred hosts) → switches suffice; large networks (thousands of hosts) → mix in routers for robust isolation.

### 6.4.4 Virtual LANs (VLANs)

**Three drawbacks of a purely hierarchical switch-per-department LAN (Figure 6.15):**
1. **No traffic isolation** — broadcast traffic (ARP, DHCP, or any not-yet-learned destination) still traverses the *entire* institutional network, which is both a performance and a security/privacy concern (e.g. an exec team's traffic reaching a floor full of packet-sniffing employees).
2. **Inefficient switch use** — many small departments would each need their own physical switch even if a single big switch could technically hold everyone (but then loses isolation).
3. **Painful to manage moves** — an employee switching departments needs physical recabling.

**Port-based VLAN:** the network manager partitions a *single* switch's ports into groups; each group is a separate **broadcast domain** (VLAN) — broadcast traffic from a port only reaches other ports *in the same group*. E.g. one 16-port switch: ports 2–8 = EE VLAN, ports 9–15 = CS VLAN. Solves all three drawbacks at once: isolation is automatic, one physical switch serves everyone, and moving a user between departments is just a **software reconfiguration** of which VLAN their port belongs to — no recabling. Switch hardware simply never delivers a frame between ports in different VLAN groups.

**Inter-VLAN traffic:** since VLANs are now fully isolated, EE↔CS traffic needs a **router** — either an external router with a port belonging to both VLANs, or (commonly) a combined switch+router device — after which the flow mirrors Figure 6.19 exactly: EE→router (crossing the EE VLAN) → router re-forwards → CS (crossing the CS VLAN).

**VLAN trunking:** to connect *two* physical VLAN switches without wasting one port-pair *per VLAN* (which doesn't scale — *N* VLANs would need *N* port-pairs), designate one **trunk port** on each switch that belongs to *all* VLANs; frames for any VLAN cross the single trunk link. Frames on a trunk need to carry **which VLAN they belong to** — that's the **802.1Q** tag: a 4-byte insertion into the standard Ethernet frame consisting of a 2-byte **Tag Protocol Identifier** (fixed `81-00`) and a 2-byte **Tag Control Information** field (a 12-bit VLAN ID + a 3-bit priority field, analogous in spirit to IP's TOS field). The sending-side trunk switch adds the tag; the receiving-side trunk switch parses it and strips it back off.

---

## Review Questions

**R1.** The **link-layer frame** — just as the passenger (datagram) rides inside a given transportation segment's vehicle, the datagram is encapsulated inside a frame for its trip across one link.

**R2.** **No, not fully redundant**, for the same reasons discussed for IP/TCP double-checksumming in Chapter 4: (1) TCP connections don't necessarily run only over reliable links — TCP might run over a mix of link technologies, some reliable, some not, so TCP can't assume every hop guarantees delivery; (2) even where every *link* provides hop-by-hop reliability, that only guarantees each frame crosses *one specific* link correctly — it says nothing about **process-to-process, end-to-end** correctness across the whole path (e.g. a bug/corruption inside a router between two perfectly reliable links, or misdelivery/reordering at a layer TCP itself must still guard against). TCP's reliability is an end-to-end guarantee that per-link reliability alone cannot provide.

**R3.** Possible link-layer services: **framing**, **link access** (MAC), **reliable delivery**, **error detection and correction** (see §6.1.1 above). Corresponding services elsewhere: IP itself offers only a **header checksum** (weak error detection, no correction, no reliability, no framing/access — those aren't IP's job) — so link-layer error detection is analogous to IP's header checksum but usually stronger. TCP offers **reliable delivery** (ACKs/retransmission/sequencing) directly analogous to a link layer's optional reliable-delivery service, plus its own checksum for error detection — but TCP has no "framing" or "link access" analog, since those concepts (encapsulating for one physical hop, and coordinating access to a shared physical medium) are meaningless above the link layer.

**R4.** **Yes, there will always be a collision** in this scenario — both nodes begin transmitting at the exact same instant, so their signals necessarily overlap and interfere at both origin points regardless of *d_prop* vs. *L/R*. (What the *d_prop < L/R* condition actually governs is whether each node can **detect** that collision before it finishes sending its own frame — since a node keeps transmitting until time *L/R*, and the other node's colliding signal takes *d_prop* to arrive; if *d_prop < L/R* the interfering signal is guaranteed to arrive while the node is still transmitting, so collision detection is guaranteed to succeed in time.)

**R5.** **Slotted ALOHA** has properties (1) and (4): a lone active node *does* get the full rate *R* (transmits every slot, no collisions to slow it down), and the protocol is simple. It lacks (2) (M active nodes get, at best, ~37% aggregate efficiency, not a fair *R/M* each) and (3) — it's decentralized in *decision-making*, but does require slot-boundary synchronization, a mild centralizing dependency. **Token passing** has properties (2) and (3): with *M* active nodes cycling the token, each gets a roughly fair share of the channel over time (property 2), and there's no master/single point of failure (property 3, decentralized) — but a lone active node still incurs a token-circulation delay each cycle before it's "its turn" again, so it doesn't get the full clean rate *R* the way ALOHA's lone node does, and it isn't as simple to implement as ALOHA (property 4 weaker).

**R6.** After the 5th collision (*n*=5), *K* is drawn uniformly from `{0,1,…,2⁵−1} = {0,…,31}` — 32 equally likely values, so **P(K=4) = 1/32**. Delay = *K*×512 bit times = 4×512 = 2048 bit times; at 10 Mbps, that's 2048/(10×10⁶) s = **204.8 μs (2.048×10⁻⁴ s)**.

**R7.** **Polling** ≈ a party host explicitly going around asking each guest in turn, "do you have anything to say?" — only the currently-addressed guest may speak, then the host moves to the next guest. **Token passing** ≈ passing a single physical "talking stick" around the room in a fixed order — whoever is holding it may speak (or immediately pass it on if they have nothing to say); there's no host, just the stick's circulation.

**R8.** Because with token ring, the token — and hence permission to transmit — has to physically circulate around the *entire* ring before returning to a given node; on a LAN with a very large physical perimeter, the propagation time for the token to complete one full lap becomes large, inflating the **token-rotation delay** experienced by every node and driving down effective throughput, especially when traffic is light (most of the rotation time is wasted carrying an unused token around a very long wire).

**R9.** MAC address space: 6 bytes = **2⁴⁸** addresses. IPv4: 32 bits = **2³²** addresses. IPv6: 128 bits = **2¹²⁸** addresses.

**R10.** **Yes**, C's *adapter* will still physically receive and process every one of those frames (it's a shared broadcast medium — the electrical signal reaches every attached adapter), but since each frame's destination MAC is B's (not C's) and not the broadcast address, C's adapter will **discard** each frame at the hardware level and will **not** pass the enclosed IP datagrams up to C's network layer — C's host software is never even interrupted. If A instead used the **MAC broadcast address**, C's adapter would **not** discard the frames — it would pass every one of them up to C's network layer (which would then presumably inspect and likely discard them at the IP layer, since they're not addressed to C's IP address either, but the link layer no longer filters them out).

**R11.** An ARP **query** must be broadcast because the querier doesn't yet know *which* adapter on the subnet owns the target IP — it has no unicast address to send it to, so it must reach everyone and let the one matching node self-identify. The ARP **response**, by contrast, can (and should) be sent as a normal unicast frame, because by the time the responder replies, it already knows exactly which single node asked (the query packet carries the querier's own MAC/IP) — there's no need to interrupt every other node a second time with an unnecessary broadcast.

**R12.** **Yes, it's possible.** Each of the router's two ARP modules only learns about, and answers for, the subnet its *own* interface is attached to — there's nothing structurally preventing the *same numeric MAC address value* from appearing as an entry in both tables (e.g. in an unusual/misconfigured setup, or simply if two distinct devices on the two different subnets happened to be assigned colliding MAC values, which shouldn't normally happen given IEEE's uniqueness guarantee, but nothing in ARP's *mechanism* itself prevents the same address value being *recorded* in two independent per-interface tables — the two ARP modules operate completely independently and have no shared-state check against each other).

**R13.** They **don't differ** in frame structure — this is precisely the point emphasized in §6.4.2 (see Figure 6.21): 10BASE-T, 100BASE-T, and Gigabit Ethernet all use the **exact same Ethernet MAC frame format** (preamble, destination/source address, type, data, CRC) — only the underlying **physical-layer** signaling (speed, encoding, medium) differs between these standards, not the link-layer frame itself. This constancy of frame format across 30+ years and multiple orders of magnitude in speed is exactly what §6.4.2 calls Ethernet's one true timeless centerpiece.

**R14.** **Exactly one subnetwork.** In Figure 6.15, all four devices are plain layer-2 **switches** (not routers) — since only a switch-to-switch (or switch-to-host) connection doesn't create an IP-subnet boundary (only a router interface does), everything reachable purely through those four switches (EE, CS, Computer Engineering, the mail server, and the Web server) sits in **one single flat broadcast domain / one IP subnet**. The one router shown (to the external Internet) is what creates the *only* subnet boundary present in the figure — separating this one internal subnet from the outside world.

**R15.** **4096 (2¹²)** — the VLAN identifier field inside the 802.1Q Tag Control Information field is **12 bits** wide, so it can encode exactly 2¹² distinct VLAN IDs.

**R16.** **2(N−1) ports**, independent of *K*. Connecting *N* switches together (in a minimal, e.g. tree/chain, topology) requires *N*−1 trunk *links*; each link needs exactly **one trunk port on each of its two end switches** (2 ports per link), for a total of 2(N−1) ports — and crucially, since a single 802.1Q **trunk port carries all K VLANs simultaneously** (tagged per-frame), this total does **not** scale with *K* at all — which is precisely trunking's whole point (contrast with the naive "one physical cable-pair per VLAN" approach, which would need 2K(N−1) ports).

---

## Problems

### P1 — Two-dimensional even parity for `1110 0110 1001 0101`
Arrange the 16 data bits as a 4×4 grid (minimal square arrangement ⇒ minimal total parity overhead of *i+j+1* = 4+4+1 = 9 bits):
```
1 1 1 0   → row parity 1
0 1 1 0   → row parity 0
1 0 0 1   → row parity 0
0 1 0 1   → row parity 0
--------
1 0 0 0   ← column parity, with corner (overall) parity bit = 1
```
**Row parities: 1,0,0,0. Column parities: 1,0,0,0. Corner/overall parity bit: 1.** (Verified computationally.)

### P2 — Two-dimensional parity: correcting 1 error, detecting-but-not-correcting 2 errors
Using the P1 grid as a base: flip a single bit, e.g. row 1, column 2 (`1→0`, changing row 1 from `1110`→`1010`). The receiver recomputes: row 1's parity now fails, column 2's parity now fails — the intersection of the *failing* row and *failing* column uniquely identifies and lets the receiver **flip back** exactly that one bit. **Detects and corrects.**

For an undetectable-but-only-*apparently*-correctable double error: flip **two bits in the same row** (e.g. both bit (1,1) and bit (1,2)). That row's own parity is unaffected (two flips cancel out, even count of errors in that row), but **both** affected columns' parities will show an error — the receiver sees 2 column failures with 0 row failures, a pattern that doesn't fit the single-error "exactly one row + one column" signature, so it can tell *something* is wrong (**detects**) but cannot identify which 2 of the 4 candidate cells (the two flipped ones, or the two "wrong" alternates in those columns) are the actual errors — it **cannot correct** it.

### P3 — Internet checksum of ASCII "Internet"
ASCII bytes of "Internet" = `[73,110,116,101,114,110,101,116]` (8 bytes = four 16-bit words). Summing as 16-bit words with end-around carry and taking the 1's complement gives:

**Checksum = 0x6A49** (binary `0110 1010 0100 1001`).

### P4 — Internet checksum for three more 10-byte payloads
*(The problem's "10 bytes" framing carries over from P3; computed directly on the given byte sequences.)*

**(a)** Binary representation of 1 through 10 (bytes `1..10`): **checksum = 0xE6E1**.
**(b)** ASCII "B" through "K" uppercase (bytes `66..75`): **checksum = 0xA09B**.
**(c)** ASCII "b" through "k" lowercase (bytes `98..107`): **checksum = 0xFFFA**.

*(All computed and cross-checked programmatically.)*

### P5 — CRC: G = 10011, D = 1010101010
*r* = 4 (G is 5 bits). Performing mod-2 division of `D·2⁴` by `G`:

**R = 0100.** Transmitted bits: `10101010100100`. *(Verified: dividing the transmitted bit string by G leaves remainder 0000.)*

### P6 — CRC for three more D values (same G = 10011)
**(a)** D = 1000100101 → **R = 1000** (transmitted `10001001011000`).
**(b)** D = 0101101010 → **R = 1111** (transmitted `01011010101111`).
**(c)** D = 0110100011 → **R = 1110** (transmitted `01101000111110`).

*(All four R values in P5/P6 independently verified by mod-2 dividing the full transmitted string back through G and confirming a zero remainder.)*

### P7 — Properties of G = 1001
**(a) Why does G=1001 detect any single-bit error?** G has more than one nonzero term and, critically, its lowest-order term (rightmost bit) is 1 while it isn't a simple pattern like `1000...0` — more precisely, a single-bit error in the transmitted codeword is equivalent to adding an "error polynomial" `E = 2ⁱ` (a lone 1 in position *i*) to the valid, G-divisible codeword. Since G's own low-order bit is 1 (G is *not* divisible by *x*, i.e. `100...0` alone), G cannot evenly divide any single term `2ⁱ` for any *i* — so the corrupted word (valid codeword XOR `2ⁱ`) can never itself be divisible by G, meaning the receiver's remainder check will always come out nonzero for exactly one flipped bit. **Any single-bit error is therefore always caught.**

**(b) Can it detect any odd number of bit errors?** **Yes.** An error pattern with an odd number of 1-bits corresponds to an error polynomial *E(x)* that (over GF(2)) has an odd number of terms, which means *E(1) = 1* (evaluating the polynomial at x=1 mod 2). Since G = 1001 = `x³+1` has **(x+1)** as a factor (check: `x³+1 = (x+1)(x²+x+1)` over GF(2)), and any polynomial divisible by G must therefore also be divisible by (x+1) — i.e. must evaluate to 0 at x=1 — any *E(x)* with *E(1)=1* (odd weight) can **never** be a multiple of G. So the corrupted codeword can never accidentally land back on a multiple of G, and **every odd-weight error pattern is guaranteed to be detected**, for this exact reason that G contains `(x+1)` as a factor.

### P8 — Completing the slotted ALOHA efficiency derivation
**(a)** Maximize `f(p) = Np(1−p)^(N−1)` over p: `f′(p) = N(1−p)^(N−2)[1 − pN]`, zero at **p\* = 1/N**.
**(b)** Efficiency at p\*: `N·(1/N)·(1−1/N)^(N−1) = (1−1/N)^(N−1)`. As `N→∞`, `(1−1/N)^N → 1/e`, so this → **1/e ≈ 0.368**.

### P9 — Max efficiency of pure ALOHA = 1/(2e)
`g(p) = Np(1−p)^(2(N−1))`. `g′(p)=0` at **p\* = 1/(2N−1)**. Plugging in and taking `N→∞` (using the same `(1−1/N)^N→1/e` limit, now applied to the doubled exponent): efficiency → **N·(1/(2N))·e⁻¹ = 1/(2e) ≈ 0.184**, exactly half the slotted-ALOHA result of P8. *(Directly follows from P8's method — the extra factor of 2 in the exponent of the "nobody else transmits" term is exactly what halves it.)*

### P10 — Two-node slotted ALOHA, unequal retransmission probabilities
**(a)** A's throughput = `pA(1−pB)` (A sends and B doesn't); B's throughput = `pB(1−pA)`. **Total efficiency = pA(1−pB) + pB(1−pA)**.

**(b)** With `pA = 2pB`: A's throughput = `2pB(1−pB)`, B's = `pB(1−2pB)`; ratio = `2(1−pB)/(1−2pB)`, which equals exactly 2 **only in the limit pB→0** — for any actual positive pB it's *more* than 2 (since 1−pB > 1−2pB, i.e. denominator shrinks faster than numerator). So **no**, doubling the retransmission probability does *not* generally double the throughput exactly. To make A's throughput *exactly* twice B's in general, solve `pA(1−pB) = 2pB(1−pA)` for pA: **pA = 2pB/(1+pB)**.

**(c)** N nodes, A uses 2p, everyone else uses p: **A's throughput = 2p(1−p)^(N−1)** (A sends, all N−1 others don't); **any other single node's throughput = p(1−2p)(1−p)^(N−2)** (that node sends, A doesn't, the remaining N−2 ordinary nodes don't).

### P11 — Four-node slotted ALOHA (A, B, C, D, each transmits w.p. p per slot)
Per-slot success probability for a *specific* node (e.g. A) = `p(1−p)³`; per-slot success probability for *some* node = `4p(1−p)³`.

**(a)** P(A succeeds first in slot 4) = `[1 − p(1−p)³]³ · p(1−p)³`.
**(b)** P(some node succeeds in slot 5) = **4p(1−p)³** (memoryless — same for any single slot).
**(c)** P(first-ever success occurs in slot 4) = `[1 − 4p(1−p)³]³ · 4p(1−p)³`.
**(d)** Efficiency of the 4-node system = **4p(1−p)³** (same expression as (b) — the long-run success rate per slot *is* the efficiency).

### P12 — Graphing slotted/pure ALOHA efficiency vs. p, for N=10, 30, 50
Both curves (`Np(1−p)^(N−1)` slotted, `Np(1−p)^(2(N−1))` pure) are **unimodal**: rising ~linearly from 0 at p=0, peaking, then decaying back toward 0 as p→1 (once p is large, collisions dominate almost every slot). The peak location and height:

| N | Slotted peak at p\* | Slotted peak efficiency `(1−1/N)^(N−1)` | Pure peak at p\* | Pure peak efficiency |
|---|---|---|---|---|
| 10 | 0.100 | 0.387 | 0.053 | 0.190 |
| 30 | 0.033 | 0.372 | 0.017 | 0.181 |
| 50 | 0.020 | 0.370 | 0.010 | 0.180 |

As N grows, the pure-ALOHA curve is always **narrower and lower** (peak ≈ half the height, at roughly half the p), and both curves' peaks converge toward the asymptotic 1/e ≈0.368 and 1/(2e)≈0.184 limits from P8/P9 as N→∞.

### P13 — Max throughput of a polling protocol
Per polling round of *N* nodes, total time = `N(d_poll + Q/R)` (each node incurs a poll delay then sends up to Q bits at rate R); total useful data sent = `N·Q` bits. **Maximum throughput = NQ / [N(d_poll + Q/R)] = Q / (d_poll + Q/R) = RQ / (Q + R·d_poll)** bps.

### P14 — Figure 6.33, three LANs via two routers
**(a)/(b)** IP/MAC assignment (any valid scheme works — one clean choice):
- Subnet 1 (192.168.1.0/24): A=192.168.1.1, B=192.168.1.2, Router1-left=192.168.1.3
- Subnet 2 (192.168.2.0/24, the *middle* subnet shared by both routers): Router1-right=192.168.2.1, Router2-left=192.168.2.2, C=192.168.2.3, D=192.168.2.4
- Subnet 3 (192.168.3.0/24): Router2-right=192.168.3.1, E=192.168.3.2, F=192.168.3.3

Each interface also gets a distinct MAC address (arbitrary hex values).

**(c)** Sending E → B (ARP tables all up to date, so no ARP traffic needed — pure forwarding):
1. E's network layer sees B (192.168.1.2) is off-subnet → sends to its default gateway, Router2's Subnet-3 interface (192.168.3.1); looks up its MAC in its (already-populated) ARP table.
2. E builds a frame (src MAC=E, dst MAC=Router2-right) carrying the IP datagram (src IP=E, dst IP=B) and sends it into Subnet 3; the Subnet-3 switch delivers it to Router2.
3. Router2 strips the frame, consults its forwarding table for 192.168.1.0/24 → next hop out its Subnet-2 interface toward Router1; builds a **new** frame (src MAC=Router2-left, dst MAC=Router1-right, from its own up-to-date ARP table) around the *same* IP datagram, sends into Subnet 2.
4. Subnet-2 switch delivers it to Router1.
5. Router1 strips it, sees B is on its own directly-attached Subnet 1, builds a **new** frame (src MAC=Router1-left, dst MAC=B, from its ARP table) and sends into Subnet 1.
6. Subnet-1 switch delivers it to B; B's adapter matches its own MAC, passes the datagram up to its network layer.

**(d)** Same, but E's ARP cache is empty: before step 2, E must first **broadcast an ARP query** ("who has 192.168.3.1?") into Subnet 3; Router2 recognizes the query as its own IP and sends back a **unicast ARP reply** with its MAC; E caches it, then proceeds exactly as steps 2–6 above.

### P15 — Figure 6.33 with the Subnet1↔2 router replaced by switch S1
**(a) E → F** (both hosts already on Subnet 3): pure same-subnet delivery — E ARPs directly for F's MAC (if needed) and sends straight to F via the Subnet-3 switch. **No router involvement at all** — R1 is never consulted, since E's network layer already sees F on its own subnet. Frame: src IP=E, dst IP=F, src MAC=E, dst MAC=F.

**(b) E → B:** even though S1 now physically bridges Subnets 1 and 2 into one flat wire, **E's network layer still only looks at IP addressing** — B's IP (192.168.1.x) is not on E's own subnet (192.168.3.x), so E still sends via its default gateway R1, exactly as before (E has no way of knowing, or reason to care, that the path beyond R1 happens to now be a flat switched segment). The frame delivered to R1: src IP=E, dst IP=B; src MAC=E, dst MAC=R1's Subnet-3-facing interface.

**(c) A → B**, neither ARP cache populated, S1's table only has entries for B and R1: A **broadcasts** an ARP request (dest MAC = broadcast). Because S1 has no way to unicast a *broadcast* frame, it **floods** it out every other port regardless of what's in its table — reaching both the rest of Subnet 1/2 (now one flat domain) *and* R1's attached interface. **R1 does receive** the broadcast frame (its adapter is physically on that same segment) but, being a router, it does **not** re-forward/extend it onward into Subnet 3 — routers don't propagate link-layer broadcasts across subnet boundaries (that containment is exactly what a router provides that a switch doesn't). B, seeing the query is for its own IP, replies with a **unicast** ARP response directly to A — it does **not** need to send its *own* ARP query first, since the original request already carried A's IP↔MAC mapping. When S1 receives B's unicast reply, it can forward it precisely to A's port **without flooding**, because S1's self-learning already recorded A's `(MAC, port)` the moment A's original broadcast request passed through it.

### P16 — Same as P15, but the Subnet2↔3 router is *also* replaced by a switch
**(a) E → F:** unaffected — still a trivial same-subnet, no-router delivery exactly as in P15(a).

**(b) E → B:** this now exposes the scenario's underlying inconsistency — the network is **entirely router-free**, yet E's IP configuration still tells it B is on a *different* subnet, so E will still try to ARP for / send via a default-gateway IP that **no longer has any router behind it**. That ARP request will simply go unanswered (or, if some device still holds that IP, unpredictably) — **communication breaks**, illustrating that merely bridging everything together at layer 2 does *not* by itself fix logically-separate IP subnet configurations; without an actual router, cross-"subnet" traffic has nowhere to go even though the hosts are now physically on the very same wire.

**(c) A → B:** now the *entire* three-subnet topology is one flat broadcast domain (S1 and S2 both flood broadcasts). A's ARP broadcast reaches literally every host (including C, D, E, F). B replies via unicast as before; both S1 and S2 will have self-learned A's location from relaying the original broadcast, so B's reply is forwarded back to A via targeted unicast hops through both switches, not flooded.

### P17 — CSMA/CD backoff wait time for K=100
Wait = `K × 512` bit times.
- **100 Mbps:** 100×512 = 51,200 bits ÷ (100×10⁶ bps) = **512 μs**.
- **1 Gbps:** 51,200 bits ÷ (10⁹ bps) = **51.2 μs**.

### P18 — Can A finish transmitting before detecting B's collision? (10 Mbps, d_prop = 325 bit times)
A begins at t=0; in the worst case (using the problem's own hint), a minimum-size frame + preamble finishes transmitting at **t = 512+64 = 576 bit times**. Worst case for *missed* detection: B senses idle and begins its own transmission at the *last possible moment* before A's signal reaches it, i.e. at **t ≈ 325** (one full propagation delay after A started); B's signal must then travel the same 325 bit-time distance back to reach A, arriving at **t = 325+325 = 650 bit times**. Since **650 > 576**, A has *already finished* sending its whole frame before B's colliding signal ever reaches it back. **Yes — A can finish transmitting without detecting the collision**, and will incorrectly believe its frame was sent successfully, even though a real collision occurred at B. (This is exactly why real Ethernet standards cap network diameter/propagation delay relative to the minimum frame size — to make this scenario impossible.)

### P19 — Do A and B's retransmissions collide again? (d_prop = 245 bit times, K_A=0, K_B=1)
Both begin at t=0, both detect the collision at **t = 245** (one prop delay later, when each other's signal arrives). 
- A (K_A=0): waits 0 extra bit times → returns to sense-and-transmit at **t = 245**, and (channel appearing idle to A at that instant) begins retransmitting immediately at **t = 245**.
- B (K_B=1): waits 512 bit times → schedules its retry for **t = 245 + 512 = 757**.
- A's retransmitted signal reaches B at **t = 245 + 245 = 490** (one more prop delay). Since A's frame (at least 512 bits, started at t=245) will still be arriving at B all the way through roughly **t = 245+512 = 757**, B senses the channel **busy** right at its own scheduled retry time (757) and **refrains from transmitting** — so in this instance, **the retransmissions do not collide again**.

### P20 — Efficiency of the CSMA/CD-like slotted protocol
Let `P_succ = Np(1−p)^(N−1)` (probability a given contention slot is successful).

**(a)** Since contention slots are i.i.d. Bernoulli(P_succ) until the first success, the expected number of *unproductive* slots is `x = (1−P_succ)/P_succ`. Efficiency = `k/(k+x) = k·P_succ / [1 + (k−1)·P_succ]`.

**(b)** Efficiency is monotonically increasing in `P_succ` for any k≥1, so maximizing it over p is equivalent to maximizing `P_succ` itself — same as ordinary slotted ALOHA: **p\* = 1/N** (from P8).

**(c)** Plugging p\* in and letting N→∞: `P_succ → 1/e`, so efficiency → `k(1/e) / [1+(k−1)/e] = k/(k−1+e)`.

**(d)** As `k→∞` (frames become very long relative to a contention slot), `k/(k−1+e) → 1` — **efficiency approaches 1**, exactly mirroring why the real CSMA/CD formula (`1/(1+5d_prop/d_trans)`) also approaches 1 as frames get long relative to propagation delay: fixed per-round contention overhead gets amortized away.

### P21 — MAC/IP addresses and frame contents, A → F (Figure 6.33, both routers present)
Using the P14 addressing scheme (A=192.168.1.1, F=192.168.3.3, Router1 left/right = 192.168.1.3/192.168.2.1, Router2 left/right = 192.168.2.2/192.168.3.1), sending A→F:

| Hop | Src IP | Dst IP | Src MAC | Dst MAC |
|---|---|---|---|---|
| (i) A → left router (R1) | A | F | A | Router1-left |
| (ii) R1 → right router (R2) | A | F | Router1-right | Router2-left |
| (iii) R2 → F | A | F | Router2-right | F |

**The IP addresses never change** (A→F throughout, as required for true end-to-end delivery) — only the **MAC addresses change at every hop**, each pair being the two adapters actually attached to that specific physical link.

### P22 — Same as P21, but the leftmost router is replaced by a switch
Now A, B, C, D, and the right router are all star-connected into one switch (one flat subnet spanning what were Subnets 1 & 2). Sending A→F:

| Hop | Src IP | Dst IP | Src MAC | Dst MAC |
|---|---|---|---|---|
| (i) A → switch | A | F | A | Router2-left (A's frame is addressed directly to the router, since F is off A's own subnet — the switch is transparent) |
| (ii) switch → right router | A | F | A | Router2-left *(unchanged — a switch never rewrites a frame, it only forwards it)* |
| (iii) right router → F | A | F | Router2-right | F |

Key contrast with P21: because a switch is **transparent** (it forwards frames as-is, never re-addressing them), the frame's MAC addresses stay **identical** across hops (i) and (ii) — only crossing an actual **router** (hop iii) produces a new pair of MAC addresses. IP addresses again never change throughout.

### P23 — Max aggregate throughput, Figure 6.15, all-switch, all 1 Gbps links
With genuine **switches** (collision-free, full-duplex, each capable of simultaneously forwarding on all its ports as long as no two flows contend for the *same* output port), up to `⌊11/2⌋ = 5` simultaneous, non-conflicting source→destination pairs can be active among the 9 hosts + 2 servers (11 endpoints) at once, each running at the full 1 Gbps. **Maximum aggregate throughput = 5 Gbps** (5 disjoint pairs each achieving 1 Gbps; the 11th, unpaired endpoint is simply idle in that instant — with 11 being odd, one node is always left over).

### P24 — Same, but the 3 departmental switches become hubs (routers/top switches unchanged)
Within each hub-based department, only **one** transmission can succeed at a time (a hub is a shared broadcast/collision domain) — so each of the 3 hub-based departments contributes **at most 1 Gbps** total (not per host), regardless of how many hosts it has. Across the 3 departments plus the still-switched top level, up to 3 flows (one per hub-department, assuming the top-level switches/servers can absorb them without further contention) can proceed concurrently. **Maximum aggregate throughput ≈ 3 Gbps** — a single hub can source only one 1 Gbps stream at a time no matter how many hosts sit behind it.

### P25 — Same, but *all* switches (including the top-level ones) become hubs
Now the **entire network** is one giant shared collision domain — only **one** transmission can be happening anywhere in the whole network at any instant. **Maximum aggregate throughput = 1 Gbps.**

### P26 — Learning switch trace (A–F star-connected; empty initial table)
| Event | Table before | Learned this event | Table after | Forwarded to |
|---|---|---|---|---|
| (i) B→E | {} | B→port_B | {B→port_B} | **Flooded** to all ports except B's (E not yet known) |
| (ii) E→B | {B→port_B} | E→port_E | {B→port_B, E→port_E} | **Unicast** to port_B only (B is now known) |
| (iii) A→B | {B, E} | A→port_A | {A→port_A, B→port_B, E→port_E} | **Unicast** to port_B only (B is known) |
| (iv) B→A | {A, B, E} | (B's entry refreshed, nothing new) | {A→port_A, B→port_B, E→port_E} | **Unicast** to port_A only (A is known) |

C, D, F never appear in the table (they've neither sent nor been the destination of anything the switch has seen), and every flood in event (i) reaches C, D, F harmlessly (they simply discard it, since it's not addressed to them).

### P28 — VLAN inter-communication, Figure 6.25 (external router on port 1)
Assign IP subnets per VLAN: EE VLAN = `10.0.1.0/24` (EE hosts get `10.0.1.x`; router's EE-side logical interface = `10.0.1.1`), CS VLAN = `10.0.2.0/24` (CS hosts get `10.0.2.x`; router's CS-side logical interface = `10.0.2.1`) — both logical router interfaces live behind the *same* physical port-1 cable (via 802.1Q trunking, "router-on-a-stick," or a combined switch+router unit).

Sending EE host (`10.0.1.5`) → CS host (`10.0.2.5`) mirrors Figure 6.19 exactly, just inside one physical box:
1. EE host's network layer sees the CS host is off-subnet → targets its default gateway `10.0.1.1`; ARPs for its MAC if not cached (query confined to the EE VLAN's broadcast domain).
2. EE host sends the frame (src IP=EE host, dst IP=CS host; dst MAC=router's EE-side MAC) — the VLAN switch delivers it to port 1 (the router), stripping/handling 802.1Q tagging as needed.
3. The router strips the frame, sees the destination belongs to the CS subnet it's also attached to, and — after ARPing for the CS host's MAC if needed (this time within the CS VLAN) — builds a **new** frame (src MAC=router's CS-side MAC, dst MAC=CS host) around the *same, unchanged* IP datagram, and sends it back out port 1, now tagged for the CS VLAN.
4. The switch delivers it to the CS host, whose adapter matches its own MAC and passes the datagram up.

Exactly as in the two-physical-router case, the **IP addresses never change** end to end, and a **brand-new frame** (with new MAC addresses) is generated at the router for the second leg — the only difference from Figure 6.19 is that both "legs" happen to physically share the same wire (port 1), distinguished purely by their 802.1Q VLAN tags rather than being separate physical interfaces.
