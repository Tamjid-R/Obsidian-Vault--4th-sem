# Chapter 4 — Sections 4.3.2–4.3.4 (Kurose & Ross, *Computer Networking: A Top-Down Approach*, 8th ed.)

Covers §4.3.2 (IPv4 Addressing), §4.3.3 (NAT), §4.3.4 (IPv6): the Section 4.3 review questions (R17–R31) and the Chapter 4 problems relevant to those topics.

---

## Review Questions

**R17.** Host B's network layer reads the **protocol field** in the IP header (a value of 6 = TCP, 17 = UDP). This tells it which transport-layer process (TCP or UDP) should receive the datagram's payload — it's exactly analogous to the port number that later demuxes to the right socket.

**R18.** The **TTL (Time to Live)** field. Each router decrements it by 1; once it hits 0 the datagram is discarded, guaranteeing it's forwarded through no more than N routers.

**R19.** Yes — they overlap. The TCP/UDP checksum isn't computed only over the segment; it's computed over the segment **plus a pseudo-header** that includes the IP datagram's source and destination address fields. Those same address bytes are also covered by the IP header's own checksum. So the source/destination IP address bytes are common to both checksum computations (the rest of the IP header — TTL, header length, etc. — is *not* covered by the transport-layer checksum).

**R20.** Only at the **final destination host**. Fragments are reassembled by the destination's network layer, never by intermediate routers.

**R21.** Yes. A router has **one IP address per interface** — since a router has multiple interfaces (one per attached link), it has multiple IP addresses, not just one.

**R22.** `223.1.3.27` →
```
11011111 00000001 00000011 00011011
```

**R23.** This one requires you to actually inspect a live host. On Windows: `ipconfig /all` in a terminal (look at IPv4 Address, Subnet Mask, Default Gateway, and DNS Servers). On macOS/Linux: `ifconfig`/`ip addr` plus `cat /etc/resolv.conf`, or System Settings → Network.

**R24.** With 3 routers between source and destination, the path is `Source–R1–R2–R3–Destination` → **4 links**, so the datagram is sent out over 4 interfaces (source's, then each router's outbound interface). Only the **3 intermediate routers** actually perform a forwarding-table lookup (the source just hands the datagram to its default gateway; the destination doesn't forward at all), so **3 forwarding tables** are indexed.

**R25.** Overhead per datagram = 20 bytes (IP header) + 20 bytes (TCP header) = 40 bytes. Payload = 40 bytes.
Total datagram = 80 bytes → **overhead = 40/80 = 50%**, **application data = 50%**.

**R26.** The ISP gives the wireless router exactly **one public IP address** (on its WAN interface). The 5 PCs get **private IP addresses** (e.g., `192.168.1.x`) from the router's built-in DHCP server. **Yes, the router uses NAT** — it's the only way for 5 devices to share a single public IP address, translating each PC's private `(IP, port)` to `(public IP, unique port)` for outbound traffic.

**R27.** Route aggregation ("route summarization") is advertising **one short prefix** (e.g. `200.23.16.0/20`) to represent many smaller, contiguous blocks (e.g. eight `/23`s) that all sit "inside" it. It's useful because routers outside the organization only need **one forwarding-table entry** instead of many, which keeps forwarding tables small and lookups fast as the Internet scales.

**R28.** A **plug-and-play / zeroconf** protocol lets a device join a network and get fully configured (IP address, subnet mask, default gateway, DNS server) **automatically**, with zero manual admin intervention. DHCP is the canonical example.

**R29.** A private address is one drawn from the blocks reserved by RFC 1918 (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) that only has meaning **inside** a specific local network — it isn't globally unique (hundreds of thousands of home networks all reuse `10.0.0.0/24`). Such an address should **never** appear as a source or destination in a datagram on the public Internet, since it's meaningless (and likely ambiguous/duplicated) outside its own private realm; routers generally drop such packets at the network edge.

**R30.** Common ground: both have a **Version** field, and both carry **source/destination addresses**, plus roughly equivalent counterparts — IPv4's TTL ↔ IPv6's Hop Limit, IPv4's Type-of-Service ↔ IPv6's Traffic Class, IPv4's Protocol ↔ IPv6's Next Header.
What IPv6 **drops**: header checksum, fragmentation fields (identification/flags/fragment offset), and options (moved to optional extension headers) — this is why IPv6 has a fixed, streamlined 40-byte header vs. IPv4's variable-length (≥20-byte) header.
What IPv6 **adds**: the 20-bit Flow Label (for flow-based QoS handling), and expands addresses from 32 to 128 bits.

**R31.** Agree. When two IPv6 nodes tunnel through a run of IPv4 routers, the entire IPv4 sub-network functions, from the two IPv6 endpoints' point of view, as a single logical hop: the sending tunnel endpoint encapsulates the whole IPv6 datagram as the *payload* of an IPv4 datagram, the intervening IPv4 routers forward it completely obliviously (just like any IPv4 traffic), and the receiving tunnel endpoint decapsulates it and continues IPv6 routing. That's exactly the role a link-layer technology plays for IP — carrying it across one "hop" while hiding its internal mechanics — so IPv6 is effectively treating the IPv4 cloud as its link layer.

---

## Problems relevant to 4.3.2–4.3.4

Chapter 4 has 25 problems total (P1–P25). Excluded: P1–P7 (router architecture/scheduling, §4.2), P8–P10/P12–P13 (longest-prefix-matching forwarding tables, §4.2.1/4.2.2 — the book explicitly ties these to Section 4.2.2), P17 (datagram/header-overhead sizing, §4.3.1), and P21–P25 (SDN/OpenFlow §4.4, ICMP). That leaves **P11, P14, P15, P16, P18, P19, P20** as the ones testing IPv4 addressing/subnetting (4.3.2), NAT (4.3.3), and IPv6 (4.3.4) — note there's no dedicated IPv6 problem in this edition's problem set; it's covered purely by R30/R31.

### P11 — Subnetting a /24 for three subnets
Pool: `223.1.17.0/24` (256 addresses). Needs: ≥60, ≥90, ≥12 interfaces.

Rule: smallest block size is the smallest power of two ≥ (needed hosts + 2 for network/broadcast).

| Subnet | Needed | Min block | Prefix | Assigned range |
|---|---|---|---|---|
| 2 (90 hosts) | ≥92 | 128 | /25 | `223.1.17.0/25` (.0–.127) |
| 1 (60 hosts) | ≥62 | 64 | /26 | `223.1.17.128/26` (.128–.191) |
| 3 (12 hosts) | ≥14 | 16 | /28 | `223.1.17.192/28` (.192–.207) |

Total used: 128+64+16 = 208 of 256, all constraints satisfied (any valid non-overlapping assignment of these sizes works — this is just one clean packing).

### P14 — Example host address + splitting a /26 into four
**Part 1:** `128.119.40.128/26` covers `.128–.191` (64 addresses, 6 host bits). Any address in that range except the network id (`.128`) and broadcast (`.191`) works, e.g. **`128.119.40.129`**.

**Part 2:** ISP owns `128.119.40.64/26` (64 addresses: `.64–.127`). Split into 4 equal subnets → 16 addresses each → borrow 2 more bits → **/28**:
- `128.119.40.64/28` (.64–.79)
- `128.119.40.80/28` (.80–.95)
- `128.119.40.96/28` (.96–.111)
- `128.119.40.112/28` (.112–.127)

### P15 — Subnetting the Figure 4.20 topology
Figure 4.20: R1 (top) connects to hosts `223.1.1.1`/`223.1.1.4` via its own interface `223.1.1.3`; R2 (bottom-left) connects to hosts `223.1.2.1`/`223.1.2.2` via `223.1.2.6`; R3 (bottom-right) connects to hosts `223.1.3.1`/`223.1.3.2` via `223.1.3.27`; and R1–R2, R1–R3, R2–R3 are point-to-point links with no hosts.

Going clockwise from 12:00: **A** = top subnet (needs 250), **B** = bottom-right (needs 120), **C** = bottom-left (needs 120); **D, E, F** = the three router-to-router links (2 addresses each).

Minimum block sizes (same rule as P11/P14):
- A (250 hosts) → needs ≥252 → **/24** (256 addrs)
- B (120 hosts) → needs ≥122 → **/25** (128 addrs)
- C (120 hosts) → needs ≥122 → **/25** (128 addrs)
- D, E, F (2 hosts each) → **/30** (4 addrs) each

**Worth flagging honestly:** the problem specifies the whole pool as `214.97.254.0/23` — only 512 addresses. A + B + C alone already need 256 + 128 + 128 = **512**, i.e., they exactly consume the entire pool with zero bits left for D, E, and F (even at 2 addresses apiece). This isn't an allocation-order mistake — it's a hard capacity fact given those interface counts and that pool size; no VLSM packing order gets around it. In practice you'd request a `/22` (1024 addresses) instead. Allocating as if the pool were `214.97.252.0/22`:
- A: `214.97.254.0/24`
- B: `214.97.255.0/25`
- C: `214.97.255.128/25`
- D: `214.97.253.0/30`, E: `214.97.253.4/30`, F: `214.97.253.8/30`

(Method and per-subnet prefix lengths are the graded skill here; if your instructor wants strict `/23`-only compliance, flag this capacity conflict as your answer to (a).)

### P16 — whois / geolocation
This needs a live `whois` query (e.g. `whois -h whois.arin.net <domain-or-IP>` or the ARIN web form) against three university IP blocks, plus checking those same IPs at MaxMind. Conceptual answer to the second part: **no**, `whois` reports the *registered organization's* address (often their legal/admin HQ), not the physical location of a given IP's traffic — for that you need a geo-IP database like MaxMind, and even those are only approximate (often resolve to the ISP's regional hub, not the exact building).

### P18 — NAT addressing (Figure 4.25 variant)
Figure 4.25 shows the LAN side has exactly 4 interfaces on `10.0.0.0/24`: router = `10.0.0.4`, hosts = `10.0.0.1`, `10.0.0.2`, `10.0.0.3`. Mapping onto `192.168.1/24` with WAN address `24.34.112.235`:

**a.** Router LAN interface: `192.168.1.4`; hosts: `192.168.1.1`, `192.168.1.2`, `192.168.1.3`.

**b.** Each host has 2 TCP connections, both to `128.119.40.86:80` → 6 entries. Each host must use two different local source ports for its own two connections (same destination), and NAT must assign 6 distinct WAN-side ports overall:

| WAN side | LAN side |
|---|---|
| 24.34.112.235, 6001 | 192.168.1.1, 5001 |
| 24.34.112.235, 6002 | 192.168.1.1, 5002 |
| 24.34.112.235, 6003 | 192.168.1.2, 5001 |
| 24.34.112.235, 6004 | 192.168.1.2, 5002 |
| 24.34.112.235, 6005 | 192.168.1.3, 5001 |
| 24.34.112.235, 6006 | 192.168.1.3, 5002 |

(Port numbers are arbitrary — only uniqueness on the WAN side matters.)

### P19 — Counting hosts behind a NAT via IP ID
**a.** Each host maintains its own **sequential** IP identification counter, incrementing by ~1 per datagram it sends. Since every host's traffic exits with the *same* NAT source IP, you can't distinguish hosts by address — but you can by watching the **ID field's** time series: packets belonging to one host form a (roughly) monotonically increasing sub-sequence, interleaved in time with other hosts' sub-sequences. Count the number of distinct interleaved increasing "tracks" of ID values in the captured stream → that's your estimate of the number of hosts behind the NAT.

**b.** No — if IDs were assigned **randomly** rather than sequentially, there'd be no numeric correlation between consecutive datagrams from the same host, so the ID field would look like uncorrelated noise regardless of source. You'd lose the signal that lets you group packets into per-host sequences, and the technique breaks down.

### P20 — NAT traversal for P2P (Arnold ↔ Bernard)
Since both are behind NAT, neither can accept a cold inbound connection — each NAT drops unsolicited inbound packets that don't match an existing NAT table entry created by an *outbound* request. The standard fix is **NAT hole punching**, using the server they already both talk to (the one Arnold used to discover Bernard) as a rendezvous point:

1. Both peers stay connected to the rendezvous server; through it, each learns the other's current **public (WAN-side) IP:port**, as observed by the server.
2. Both peers then send an outbound packet **directly to each other's advertised public IP:port** at roughly the same time.
3. Each side's own outbound packet opens ("punches") a pinhole in its own NAT for that specific remote IP:port — so when the other side's inbound packet arrives moments later, it matches that freshly-opened mapping and is let through.
4. This yields a direct connection with no manual port-forwarding configuration needed.

Caveat: this fails against **symmetric NATs**, which assign a *different* external port for every different destination — the public port learned via the rendezvous server won't be the port actually used toward the peer, so no hole gets punched. In that case you fall back to relaying all traffic through a third-party server (e.g. a TURN relay) that both peers can reach.
