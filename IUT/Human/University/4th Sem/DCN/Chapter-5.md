# Chapter 5 — The Network Layer: Control Plane 


## 5.1 Introduction — Data Plane vs. Control Plane

The **forwarding table** (destination-based forwarding) / **flow table** (generalized forwarding) is the interface between data plane and control plane — the control plane's whole job is computing, installing, and maintaining these tables. Two architectural approaches:

- **Per-router control** ("monolithic"): a routing algorithm runs *inside every router*; the routing component and the forwarding component live in the same box, and routers talk directly to each other to compute their own forwarding tables. This is how OSPF and BGP work (§5.3, §5.4).
- **Logically centralized control**: a remote **controller** computes forwarding/flow tables for *every* router and pushes them down. Each router runs only a thin **Control Agent (CA)** that talks to the controller and installs what it's told — CAs don't talk to each other or compute anything themselves. "Logically centralized" means accessed *as if* one service point, even though it may be replicated across many physical servers for fault tolerance/scale. This is the SDN model (§5.5). Real deployments: Google B4 (WAN between its own datacenters), Microsoft SWAN, Comcast ActiveCore, Deutsche Telekom Access 4.0, AT&T, China Telecom/Unicom.

---

## 5.2 Routing Algorithms

**Graph model:** network = graph *G = (N, E)*; nodes = routers, edges = links, each edge (x,y) has cost *c(x,y)* (∞ if no edge). Undirected here (c(x,y)=c(y,x)), though algorithms generalize to directed. *y is a neighbor of x* if edge (x,y) ∈ E. **Path cost** = sum of edge costs along it. Goal: find the **least-cost path** between any source/destination. If all edge costs are equal, least-cost = fewest hops.

**Three classification axes:**
| Axis | Options |
|---|---|
| Information | **Centralized** (a.k.a. **link-state, LS** — full global topology/cost knowledge, e.g. computed at one controller or replicated identically at every router) vs **Decentralized** (a.k.a. **distance-vector, DV** — each node knows only its own neighbors' link costs, iterates by exchanging info with neighbors) |
| Time | **Static** (changes slowly, human-edited) vs **Dynamic** (reacts to topology/traffic changes, periodically or event-triggered) |
| Load | **Load-sensitive** (link cost reflects current congestion — early ARPANET, later abandoned) vs **Load-insensitive** (today's RIP/OSPF/BGP — cost doesn't reflect current traffic) |

### 5.2.1 Link-State (LS) Routing — Dijkstra's Algorithm

Every node **floods link-state packets** (its own attached links + costs) to *all* other nodes, so every node ends up with an *identical, complete* map of the network and independently computes the *same* shortest-path tree using **Dijkstra's algorithm**.

**Notation:**
- **D(v)**: current known cost of least-cost path from source u to v.
- **p(v)**: predecessor node of v along that path.
- **N′**: set of nodes whose least-cost path from u is *definitively* known.

**Pseudocode:**
```
Initialization:
  N' = {u}
  for all nodes v:
    if v adjacent to u: D(v) = c(u,v)
    else:                D(v) = ∞

Loop:
  find w not in N' such that D(w) is a minimum
  add w to N'
  update D(v) for each neighbor v of w, not in N':
     D(v) = min( D(v), D(w) + c(w,v) )
until N' = N
```
Each pass adds exactly one node to N′ — the one with globally smallest tentative cost — and that choice is *final* (never revisited), which is why it's a greedy algorithm. After termination, backtracking through each node's `p(v)` reconstructs the full shortest path; **the forwarding table entry for each destination is just the *first hop* on that path** (Figure 5.4).

**Complexity:** naive scan-for-minimum implementation = search through ~n nodes each of n iterations → **O(n²)**. A heap-based priority queue finds the min in log time → **O(n log n)**.

**Worked example (Figure 5.3 graph)** — nodes u,v,w,x,y,z; edges u-v=2, u-x=1, u-w=5, v-w=3, v-x=2, x-w=3, x-y=1, w-y=1, w-z=5, y-z=2. Dijkstra from **u**:

| Step | N′ | D(v),p(v) | D(w),p(w) | D(x),p(x) | D(y),p(y) | D(z),p(z) |
|---|---|---|---|---|---|---|
| 0 | u | 2,u | 5,u | 1,u | ∞ | ∞ |
| 1 | ux | 2,u | 4,x | — | 2,x | ∞ |
| 2 | uxy | 2,u | 3,y | — | — | 4,y |
| 3 | uxyv | — | 3,y | — | — | 4,y |
| 4 | uxyvw | — | — | — | — | 4,y |
| 5 | uxyvwz | — | — | — | — | — |

Final: v=2(direct), x=1(direct), w=3(via y), y=2(via x), z=4(via y). Forwarding table at u: dest v→v, x→x, w→x (next hop), y→x, z→x.

**Oscillation pathology (Figure 5.5):** if link costs are made proportional to *current traffic load* (a load-sensitive metric), routers can synchronize into oscillation: everyone shifts to the "clockwise" route once it looks cheap, which then makes it expensive, so everyone shifts back "counterclockwise," forever. Fixes: don't let cost depend on carried traffic (defeats the purpose), or desynchronize routers (randomize when each broadcasts its link-state update) — routers can otherwise **self-synchronize** even with independent random phases (Floyd & Jacobson).

### 5.2.2 Distance-Vector (DV) Routing — Bellman-Ford

DV is **iterative, asynchronous, distributed, and self-terminating** (no explicit stop signal — it just stops sending once nothing changes).

**Bellman-Ford equation:** `dx(y) = min_v { c(x,v) + dv(y) }`, minimized over all neighbors v of x. Intuition: to get from x to y, you must first hop to *some* neighbor v, then take the best path from there — so the best overall path is the best choice of first-hop.

Each node x maintains: (1) link costs c(x,v) to each neighbor v; (2) its own **distance vector** Dx = [Dx(y): y ∈ N]; (3) the most recently received distance vector Dv from each neighbor v.

**Pseudocode:**
```
Initialization:
  for all y in N: Dx(y) = c(x,y)     // ∞ if y not a neighbor
  for each neighbor w: send Dx to w

loop
  wait (for a link-cost change, or a distance vector from neighbor w)
  for each y in N:
     Dx(y) = min_v { c(x,v) + Dv(y) }
  if Dx(y) changed for any y:
     send Dx to all neighbors
forever
```
The next-hop for destination y is whichever neighbor v achieves the minimum — that's what actually populates the forwarding table.

**Worked example** matches book Figure 5.6's 3-node net (x,y,z; c(x,y)=2, c(y,z)=1, c(z,x)=7): starting from all-∞ except direct neighbors, after exchanging vectors once, x discovers `Dx(z) = min(2+Dy(z), 7+Dz(z)) = min(2+1, 7+0) = 3` — cheaper via y than the direct 7-cost link — and updates. Algorithm quiesces once no node's vector changes further.

**Link-cost changes:**
- **Cost decrease ("good news")** propagates fast — typically 1–2 iterations, since a node just adopts the better path and tells its neighbors.
- **Cost increase ("bad news")** can propagate *very* slowly and cause a **routing loop**: e.g. c(y,x) jumps 4→60 in a y–z–x triangle where z's only alternate route is *via y*. y, unaware anything is wrong with z's claimed route, computes a "new" path to x via z; z, unaware y's path now runs through it in a circle, does the same back. Packets for x bounce between y and z forever until the loop self-destructs by inflating cost past the real alternative — this is the **count-to-infinity problem**.

**Poisoned reverse:** if z routes to x *via* y, z tells y `Dz(x) = ∞` (a lie — z actually knows a finite cost) so y will never think of routing to x via z. This kills **2-node** ping-pong loops outright, but **does not fix loops involving 3+ nodes** — poisoned reverse only breaks the reflexive edge, not an indirect cycle further around the graph.

**LS vs. DV comparison:**

| | Link-State | Distance-Vector |
|---|---|---|
| Message complexity | O(\|N\|·\|E\|) — each node must learn every link's cost (flooded to all) | Only neighbor-to-neighbor exchanges; propagation only if the local best path actually changes |
| Convergence speed | O(n²) (or O(n log n) with a heap); can oscillate under load-sensitive cost | Can converge slowly; routing loops possible mid-convergence; count-to-infinity |
| Robustness | A compromised node can only lie about *its own* attached links, and each node computes its own table independently — more contained damage | A compromised node can advertise arbitrarily wrong paths to *any* destination, and this bad data diffuses transitively through the whole network (real 1997 incident: a malfunctioning small-ISP router poisoned national backbone tables, disconnecting parts of the Internet for hours) |

Neither algorithm strictly dominates — the real Internet uses **both** (LS: OSPF/IS-IS inside an AS; DV-style: RIP, and BGP itself is DV-like).

---

## 5.3 Intra-AS Routing: OSPF

**Why not one giant flat routing domain?** Two reasons: **scale** (hundreds of millions of routers — full LS/DV over all of them would never converge and would need impossible memory/bandwidth) and **administrative autonomy** (each ISP wants to run its own routing policy internally and hide its internal topology from outsiders).

Solution: partition the Internet into **autonomous systems (ASs)** — groups of routers under one administrative control, each identified by a globally unique **ASN** (assigned by ICANN regional registries, like IP addresses). Some tier-1 ISPs are a single giant AS; others split into many. The protocol routing *within* one AS is the **intra-AS routing protocol** (a.k.a. **IGP**, interior gateway protocol).

**OSPF (Open Shortest Path First)** — the dominant IGP (with cousin IS-IS). "Open" = publicly specified (RFC 2328), unlike Cisco's formerly-proprietary EIGRP. It is a **link-state** protocol: every router floods link-state info to *every other router in the AS* (not just neighbors), each builds an identical full topology map, and each independently runs **Dijkstra** rooted at itself. Link costs are **set by the network administrator** (OSPF doesn't mandate a policy — could be all-1 for min-hop routing, or inverse-proportional-to-capacity to steer away from low-bandwidth links).

Flooding triggers: whenever a link's state changes (cost or up/down), **and** periodically regardless (at least every 30 minutes) for robustness. OSPF messages ride **directly over IP** (protocol number 89) — so OSPF itself must implement its own reliable delivery and neighbor liveness checks (**HELLO** messages).

**Notable OSPF features:**
- **Security**: exchanges can be authenticated — **simple** (shared plaintext password, weak) or **MD5** (hash of packet + shared secret key, plus sequence numbers to block replay attacks).
- **Multiple same-cost paths**: OSPF can load-balance traffic across several equal-cost paths instead of forcing a single winner.
- **Multicast support**: MOSPF extends OSPF's link database for multicast routing.
- **Hierarchy via Areas**: a large AS can be split into **areas**, each running its own internal OSPF instance; **area border routers** connect areas; exactly one area is the **backbone area**, which routes traffic *between* other areas and contains all area border routers. Inter-area routing = intra-area to the border router → across the backbone → intra-area to destination.

**Setting OSPF weights (Principles in Practice sidebar):** naively, weights are given → Dijkstra computes routes to minimize them. In practice this is often **inverted**: an operator has a target traffic engineering goal (e.g., minimize max link utilization given known ingress/egress traffic patterns) and must *reverse-engineer* the link weights that will make OSPF/Dijkstra *produce* that desired routing.

---

## 5.4 Routing Among the ISPs: BGP

Intra-AS protocols only route *within* one AS. Crossing AS boundaries needs an **inter-AS routing protocol** — and the Internet uses exactly one: **BGP (Border Gateway Protocol)**, arguably the single most important Internet protocol besides IP itself — it's the "glue" holding the thousands of independently-administered ASs together. BGP is **decentralized, asynchronous, and DV-like** in spirit.

### 5.4.1 The Role of BGP

BGP routes to **CIDRized prefixes** (e.g. `138.16.68/22`), not individual addresses — forwarding table entries are `(prefix, interface)`. BGP gives every router two things:
1. **Prefix reachability** — lets every subnet "announce itself" to the rest of the Internet (without BGP, a subnet would be an unreachable isolated island).
2. **Best-route selection** — when multiple paths to the same prefix exist, a local BGP policy-driven procedure picks the winner.

### 5.4.2 Advertising BGP Route Information

Every AS has **gateway routers** (touch other ASs directly) and **internal routers** (only touch hosts/routers inside their own AS). Routers — not ASs — actually exchange messages, over **semi-permanent TCP connections on port 179**. A **BGP connection** spanning two different ASs = **eBGP** (external); one between two routers *inside* the same AS = **iBGP** (internal) — internal routers commonly form a full iBGP mesh so reachability info learned at one gateway propagates to every router in the AS.

Advertising a prefix x that lives in AS3, out to AS2 then AS1, looks like: AS3's gateway sends eBGP `"AS3 x"` to AS2's gateway → AS2 propagates via iBGP to its *other* gateway → that gateway sends eBGP `"AS2 AS3 x"` to AS1 → AS1 propagates via iBGP internally. Every router ends up knowing both *that x exists* and *the AS-level path* to reach it.

### 5.4.3 Determining the Best Routes

A **route** = a prefix + its **attributes**. The two key attributes:
- **AS-PATH**: the ordered list of ASs the advertisement has traversed (each AS prepends its own ASN as it re-advertises). Also doubles as a **loop-detector**: if a router sees its *own* AS already in the AS-PATH, it rejects the advertisement outright.
- **NEXT-HOP**: the IP address of the router interface that *begins* the AS-PATH — the critical link between inter-AS and intra-AS routing, since it's an address the receiving AS's own intra-AS protocol can actually compute a path to.

**Hot-potato routing**: the simplest policy — among all candidate routes, pick whichever has the *cheapest intra-AS path to its NEXT-HOP router* (i.e. get the packet off your own network ASAP, ignore what happens to it afterward outside your AS). "Selfish" by design — minimizes cost inside *your* AS, ignoring total end-to-end cost.

**Full BGP route-selection algorithm** (applied in order, each rule breaking ties left by the previous one, until one route remains):
1. Highest **local preference** value (a purely policy-set attribute — administrator's choice, possibly learned from another router in the AS).
2. Shortest **AS-PATH** length (this alone, if it were the only rule, would make BGP a plain DV algorithm using AS-hop-count as the metric).
3. **Hot-potato routing** — closest NEXT-HOP router.
4. Arbitrary tie-break via BGP router identifiers.

Real BGP tables at tier-1 routers often hold **500,000+ routes** (see routeviews.org for live examples).

### 5.4.4 IP-Anycast

BGP's route-selection naturally implements a "route to the nearest replica" service: assign the **same IP address** to multiple geographically-dispersed servers (or DNS servers), have each independently advertise it via ordinary BGP. Different BGP routers around the world will each see multiple "paths" to that one address (really: paths to different physical machines) and — via ordinary best-route selection (e.g., shortest AS-hop-count) — steer traffic to whichever instance is topologically closest. **CDNs generally avoid this in practice** because a mid-connection BGP route change can silently redirect packets of the same TCP connection to a *different* physical server. But it's **exactly** how DNS root servers work: 13 well-known IP addresses, but 100+ actual physical servers behind some of them, with IP-anycast + BGP routing each query to its nearest instance.

### 5.4.5 Routing Policy

Real inter-AS routing is dominated by **business relationships**, not pure cost-minimization, implemented entirely through *selective route advertisement*:
- A **stub/access network** (only source or destination of its own traffic, never a through-path) enforces this simply by **not advertising** any routes except to itself — e.g. even if X (a multi-homed access ISP with two providers B and C) knows a path to Y via C, it won't tell B, so B will never route B↔Y traffic through X.
- A **provider** (say B) will advertise a route it learned from upstream provider A (say path "AW") on to its own customer X ("BAW", so X knows it can reach W via B) — but generally will **not** advertise that same path on to peer C, because carrying transit traffic between two *other* backbone providers (A and C) is a cost B shouldn't have to absorb for free.
- **Rule of thumb** used by real commercial ISPs: any traffic crossing an ISP's backbone must have a **source or destination (or both) that is a customer** of that ISP — otherwise the traffic is a "free ride." Actual peering agreements are individually negotiated and typically confidential.

**Why are intra-AS and inter-AS protocols different at all?** (Principles in Practice sidebar)
| | Inter-AS (BGP) | Intra-AS (OSPF) |
|---|---|---|
| Policy | Dominant concern — who's allowed to route through whom | Minor — everything's under one administrator |
| Scale | Must handle the whole Internet's scale | Smaller; if an ISP outgrows it, just split into two ASs |
| Performance | Secondary to policy — a longer path satisfying policy beats a shorter one that doesn't; BGP doesn't even have a real "cost," only AS-hop-count | Primary focus, since policy isn't fighting it |

### 5.4.6 Putting the Pieces Together: Obtaining Internet Presence

How a new company gets a working public web/mail presence, tying together IP addressing, DNS, and BGP end to end:
1. Contract with a **local ISP** for physical connectivity *and* an **IP address block** (e.g. a /24 = 256 addresses); assign individual addresses to your web server, mail server, DNS server, gateway router, etc.
2. Contract with an **Internet registrar** for a **domain name** (e.g. `xanadu.com`), and give the registrar your DNS server's IP so it can be entered into the `.com` TLD servers — this is what lets anyone resolve your domain via the global DNS system.
3. Populate your own **DNS server** with host-name → IP mappings for `www.xanadu.com`, your mail server, etc., so resolvers can find them once they've found your DNS server.
4. The last, easily-overlooked step: **every router on the path to you needs a forwarding-table entry for your prefix** — and that's delivered purely by **BGP**: your local ISP advertises your new prefix (via eBGP) to the ISPs it connects to, who propagate it onward, until eventually the whole Internet's routers know how to reach you (or know an aggregate that covers you).

---

## Review Questions

**R1.** A network-layer packet is called a **datagram**. The fundamental difference between a router and a link-layer switch: a router is a network-layer device making forwarding decisions based on network-layer (IP) addresses/destination prefixes and running network-layer routing protocols; a link-layer switch is a link-layer device forwarding based on link-layer (MAC) addresses and is not usually visible to/aware of network-layer routing at all.

**R2.** Per-router control = a routing algorithm runs *in every router itself*, and each router's control-plane component talks directly to its peers to compute *its own* forwarding table — the routing (control) logic and the forwarding (data) logic live in the same physical box ("monolithic"). Logically centralized control = a distinct, typically remote, controller computes forwarding/flow tables for *all* routers and pushes them down to a thin Control Agent in each router; here the data plane (the router, forwarding packets) and the control plane (the controller, computing tables) are in **separate devices** — the router's local CA doesn't compute routes, it just installs what the controller sends it.

**R3.** Centralized (LS): needs complete global topology/cost info as input; the actual computation may still run at one place or be replicated identically at every node — the defining trait is *complete information*, not *where* it runs. Decentralized (DV): no node ever has global information — each starts knowing only its own directly-attached link costs, and reaches a solution only through iterative message exchange with neighbors. Example: OSPF (link-state, i.e. centralized-information style, computed independently at each router) vs. RIP or BGP (decentralized, distance-vector style).

**R4.** See the LS vs. DV comparison table under §5.2.2 above (message complexity, convergence speed, robustness).

**R5.** The **count-to-infinity problem**: when a link's cost *increases* (or a link fails), nodes that don't yet know about the increase may still be advertising an outdated, now-invalid "shortcut" through the very node that's trying to use *them*, forming a routing loop; the bad news then propagates only very slowly, as the loop's advertised cost inflates one small increment at a time until it finally exceeds the real alternative cost.

**R6.** No, it is **not** necessary — different ASs are completely free to run different intra-AS routing protocols internally, since each AS operates its interior under its own administrative autonomy; this is exactly why the Internet needs BGP as a *separate* inter-AS protocol that doesn't care what IGP any given AS uses internally.

**R7.** See the "Why are there different inter-AS and intra-AS routing protocols?" comparison table under §5.4.5 above: **policy** (dominant between ASs, minor within one), **scale** (inter-AS must handle the entire Internet), and **performance** (inter-AS routing subordinates route quality to policy; intra-AS can focus on performance since policy isn't in the way).

**R8.** **False.** An OSPF router **floods** its link-state information to **every other router in its AS** (or in its area, for a hierarchical AS) — not just to its directly attached neighbors. This is the defining trait of a link-state protocol: full topology info reaches everyone, not just 1-hop peers.

**R9.** An **area** is a hierarchical subdivision of an OSPF autonomous system — each area runs its own self-contained OSPF link-state instance internally, with **area border routers** handling routing between areas via a designated **backbone area**. It was introduced to solve the **scaling problem**: flooding full link-state info to every router in a very large AS becomes prohibitively expensive, so partitioning into areas keeps each area's flooding domain (and each router's topology database) small, while the backbone stitches areas together.

**R10.** A **subnet** is a physical/logical network segment where every attached interface shares a common address prefix (all devices reachable without going through a router). A **prefix** is the CIDR notation `a.b.c.d/x` that identifies such an address block (the leading x bits are the network portion). A **BGP route** is a *prefix* bundled together with its **BGP attributes** (AS-PATH, NEXT-HOP, local preference, etc.) — i.e., "route" = prefix + the metadata BGP uses to choose among competing paths to that prefix.

**R11.** **NEXT-HOP** = the IP address of the router interface that begins the advertised AS-PATH; it's what lets the *receiving* AS's own intra-AS routing protocol actually compute a real, concrete path to leave its own network toward that route (bridging inter-AS reachability info to intra-AS forwarding). **AS-PATH** = the list of ASs the route advertisement has passed through; used both to pick the "best" route (shorter AS-PATH generally preferred, rule 2 of route selection) and to **detect and prevent loops** (a router rejects any advertisement that already contains its own AS in the path).

**R12.** A network administrator implements policy in BGP mainly by controlling **which routes get advertised to which neighbors**, and by setting the **local preference** attribute. For an upper-tier (backbone) ISP specifically: it can refuse to advertise routes learned from one peer on to another peer (so it doesn't become an unpaid transit carrier between two other backbones — the "no free ride" rule of thumb), can set local preference to favor certain customer/peer routes over others, and can use attributes like MED to influence *which* of several peering points a neighboring AS should use to hand off traffic (see P16 below).

**R13.** **False.** A BGP router adds its *own* AS number to the AS-PATH only when re-advertising a route to a router in a **different** AS (an eBGP session) — this is what builds up the AS-level path. When re-advertising internally via **iBGP** to another router in the *same* AS, it does **not** add anything to the AS-PATH (since no AS boundary was crossed) — the AS-PATH only grows at AS boundaries, not at every hop.

---

## Problems

### P1 — Loop-free paths from y to u (Figure 5.3)
Edges: u-v, u-x, u-w, v-w, v-x, x-w, x-y, w-y, w-z, y-z. Enumerating all simple (non-repeating) paths from y to u gives **15**:

y-x-u · y-x-v-u · y-x-v-w-u · y-x-w-u · y-x-w-v-u · y-w-u · y-w-v-u · y-w-v-x-u · y-w-x-u · y-w-x-v-u · y-z-w-u · y-z-w-v-u · y-z-w-v-x-u · y-z-w-x-u · y-z-w-x-v-u

### P2 — Loop-free paths for x→z, z→u, z→w

**x → z (12 paths):** x-u-v-w-z · x-u-v-w-y-z · x-u-w-z · x-u-w-y-z · x-v-u-w-z · x-v-u-w-y-z · x-v-w-z · x-v-w-y-z · x-w-z · x-w-y-z · x-y-z · x-y-w-z

**z → u (17 paths — this is the "17 possible paths between u and z" the textbook prose itself mentions):** z-w-u · z-w-v-u · z-w-v-x-u · z-w-x-u · z-w-x-v-u · z-w-y-x-u · z-w-y-x-v-u · z-y-x-u · z-y-x-v-u · z-y-x-v-w-u · z-y-x-w-u · z-y-x-w-v-u · z-y-w-u · z-y-w-v-u · z-y-w-v-x-u · z-y-w-x-u · z-y-w-x-v-u

**z → w (7 paths):** z-w · z-y-w · z-y-x-w · z-y-x-u-w · z-y-x-u-v-w · z-y-x-v-w · z-y-x-v-u-w

*(All four counts above were verified by exhaustive DFS, not just hand-tracing.)*

### P3 — Dijkstra from x (new 7-node graph)
Graph: x-y=6, x-v=3, x-w=6, x-z=8, y-z=12, y-v=8, y-t=7, v-t=4, v-u=3, v-w=4, t-u=2, u-w=3.

| Step | N′ | D(y) | D(z) | D(v) | D(w) | D(t) | D(u) |
|---|---|---|---|---|---|---|---|
| 0 | x | 6,x | 8,x | 3,x | 6,x | ∞ | ∞ |
| 1 | xv | 6,x | 8,x | — | 6,x | 7,v | 6,v |
| 2 | xvy | — | 8,x | — | 6,x | 7,v | 6,v |
| 3 | xvyw | — | 8,x | — | — | 7,v | 6,v |
| 4 | xvywu | — | 8,x | — | — | 7,v | — |
| 5 | xvywut | — | 8,x | — | — | — | — |
| 6 | xvywutz | — | — | — | — | — | — |

**Result:** y=6(direct), z=8(direct), v=3(direct), w=6(direct), u=6(via v), t=7(via v).

### P4 — Dijkstra from t, u, v, w, y, z (same graph as P3)
*(Each verified by exhaustive Dijkstra computation.)*

**(a) From t:** y=7(direct), v=4(direct), u=2(direct), w=5(via u), x=7(via v), z=15(via x — i.e. path t-v-x-z).

**(b) From u:** t=2(direct), v=3(direct), w=3(direct), x=6(via v), y=9(via t), z=14(via x — path u-v-x-z).

**(c) From v:** x=3(direct), u=3(direct), t=4(direct), w=4(direct), y=8(direct), z=11(via x).

**(d) From w:** u=3(direct), v=4(direct), t=5(via u), x=6(direct), z=14(via x); **y=12 — a 3-way tie** (w-v-y = 4+8=12, w-x-y = 6+6=12, w-u-t-y = 3+2+7=12, all equal).

**(e) From y:** x=6(direct), t=7(direct), v=8(direct), z=12(direct), u=9(via t); **w=12 — a 3-way tie** (y-x-w=6+6=12, y-v-w=8+4=12, y-t-u-w=7+2+3=12, all equal).

**(f) From z:** x=8(direct), y=12(direct), v=11(via x), w=14(via x), u=14(via x then v), t=15(via x then v).

### P5 — Distance table at node z
Graph (square + pendant): u-v=1, v-x=3, x-y=3, y-u=2, v-z=6, x-z=2 (z is a "fragment" node with unlabeled edges beyond the shown network).

z's **distance table** (rows = destination, columns = via-neighbor, entry = c(z,neighbor)+D<sub>neighbor</sub>(destination)):

*Iteration 0 (direct only):* Dz(v)=6, Dz(x)=2, Dz(u)=∞, Dz(y)=∞.

*Iteration 1 (after z receives v and x's initial vectors):* via v: u=6+1=7, x=6+3=9, y=6+∞=∞. via x: u=2+∞=∞, v=2+3=5, y=2+3=5. → Dz(u)=7(via v), Dz(v)=5(via x, beating the direct link!), Dz(x)=2(direct), Dz(y)=5(via x).

*Converged (after v and x themselves finish converging — u and x stop changing after 1 more round):* **Dz(u)=6 (via x, path z-x-v-u), Dz(v)=5 (via x, path z-x-v), Dz(x)=2 (direct), Dz(y)=5 (via x, path z-x-y).** Full converged table:

| Destination | via v | via x | chosen |
|---|---|---|---|
| u | 6+1=7 | 2+4=6 | **6 (via x)** |
| v | 6+0=6 | 2+3=5 | **5 (via x)** |
| x | 6+3=9 | 2+0=2 | **2 (via x)** |
| y | 6+3=9 | 2+3=5 | **5 (via x)** |

*(Verified against ground-truth shortest paths — e.g. z-x-v-u = 2+3+1 = 6.)*

### P6 — Max iterations for synchronous DV to converge
**At most N−1 iterations**, where N = number of nodes. Justification: in an N-node graph, any least-cost (loop-free) path has **at most N−1 hops** (a simple path visits at most N nodes). In the synchronous algorithm, information about any given path can propagate outward at a rate of **at most one additional hop per iteration**, since each round a node only exchanges with its immediate neighbors. So after N−1 rounds, every node's estimate has had time to reflect the best path of up to N−1 hops — which is enough to guarantee it has found the true shortest path to every destination.

### P7 — Network fragment (x, w, y; c(x,w)=2, c(x,y)=5, c(w,y)=2; w's cost to u=5, y's cost to u=6)
**(a)** x's distance vector: **Dx(w) = 2** (direct), **Dx(y) = 4** (via w: 2+2=4, beats the direct link's 5), **Dx(u) = 7** (via w: 2+5=7, vs. via y: 5+6=11).

**(b)** Currently x's best path to u is via w (cost 7). Increase **c(x,w) to 7 or more** (e.g. c(x,w)=10): then via-w costs 10+5=15 while via-y still costs 5+6=11, so x's new minimum-cost path to u switches to **via y at cost 11** — a genuinely new next-hop, which x will announce to its neighbors.

**(c)** Any change to **c(x,y)** leaves Dx(u) untouched, since that link isn't on x's current best route to u and the via-w route (cost 7, fixed) will keep winning regardless: e.g. changing c(x,y) from 5 to 4 gives via-y = 4+6=10, still worse than via-w's 7, so x's minimum cost to u stays exactly 7 (unchanged) and it sends nothing new.

### P8 — 3-node distance tables with c(x,y)=3, c(y,z)=6, c(z,x)=4
Unlike the book's original Figure 5.6 example (2,1,7 — which does *not* satisfy the triangle inequality, forcing an update), **these costs already satisfy the triangle inequality** in every direction (x-y direct 3 < via z 3+... i.e. 4+6=10; x-z direct 4 < via y 3+6=9; y-z direct 6 < via x 3+4=7). Consequence: **nothing changes after the first exchange** — each node's own distance vector is already optimal from iteration 0, so the algorithm converges immediately (verified by direct shortest-path computation). Final (and initial) distance tables at every node:

| from \ to | x | y | z |
|---|---|---|---|
| x | 0 | 3 | 4 |
| y | 3 | 0 | 6 |
| z | 4 | 6 | 0 |

Each node's table = its own row plus the (unchanged) rows received from its two neighbors.

### P9 — Count-to-infinity: does it happen on a cost *decrease*, or a brand-new link?
**No, in neither case.** Count-to-infinity is specifically a consequence of "**bad news**" (a cost *increase* or link failure) propagating slowly while a stale, now-invalid route through a former shortcut keeps getting believed. A cost **decrease** — or equivalently, adding a brand-new link where none existed (cost ∞ → finite) — is always "**good news**": a node simply discovers a genuinely better path and correctly announces it; there's no stale/incorrect information anywhere in the loop for other nodes to keep believing, so no loop can form and the update propagates quickly and correctly (as in Figure 5.7(a)).

### P10 — Prove D(x) is non-increasing and stabilizes in finite steps
**Upper bound (non-increasing):** by induction, at initialization Dx(y) = c(x,y) (or ∞) — the cost of *some* real 1-hop candidate path, hence ≥ the true shortest cost dx(y). At each subsequent iteration, Dx(y) is recomputed as `min_v{c(x,v)+Dv(y)}` using the neighbors' *most recent* (by induction, non-increasing, hence ≤ their previous) values — minimizing over inputs that have each only decreased or stayed the same can only produce a result that has decreased or stayed the same. So Dx(y) never increases, iteration over iteration.

**Lower bound:** every value Dx(y) ever computed corresponds to the cost of some *actual* path through the graph (built by recursively following whichever neighbor achieved each minimum), so Dx(y) ≥ dx(y) (the true shortest-path cost) always.

**Finite stabilization:** Dx(y) is therefore a non-increasing sequence, bounded below by dx(y), in a *finite* graph — so it can only take on finitely many distinct values (costs of the finitely many simple paths from x to y) before it must stop decreasing. Hence it stabilizes after a finite number of iterations (and, by the Bellman-Ford equation's uniqueness, it stabilizes exactly at dx(y)).

### P11 — Poisoned reverse with an added router w (c(x,y)=4, c(x,z)=50, c(y,w)=1, c(z,w)=1, c(y,z)=3)
Verified by full simulation.

**(a)** At stability, true shortest distances to x are: y=4 (direct), w=5 (via y: 1+4), z=6 (via w: z-w-y-x = 1+1+4). Since z routes to x *via* w, poisoned reverse forces z to lie to w (∞); since w routes *via* y, w must lie to y (∞); y routes directly, so it never lies to either:

| Advertiser → Recipient | Value told |
|---|---|
| y → z | 4 (true) |
| y → w | 4 (true) |
| z → y | 6 (true, z doesn't route via y) |
| z → w | **∞ (poisoned — z routes via w)** |
| w → y | **∞ (poisoned — w routes via y)** |
| w → z | 5 (true, w doesn't route via z) |

**(b)** **Yes, a count-to-infinity problem occurs even with poisoned reverse.** The reason: y, z, and w form a **3-node cycle** (y–z–w–y), and poisoned reverse only ever suppresses the direct *reflexive* edge between a node and its own next-hop — it cannot see or block a loop that runs all the way around 3 (or more) nodes, exactly as the textbook notes. When c(x,y) jumps to 60, y switches to routing via z (cost 9, using z's stale advertised 6); z is meanwhile still routing via w (stale 5); w is still routing via y (stale, now-outdated 4/9) — the three chase each other's tails, with each lap around the loop inflating everyone's estimate by the loop's total cost (c(y,z)+c(z,w)+c(w,y) = 3+1+1 = 5). Exact round-by-round simulation confirms **32 message rounds** of escalation (y: 9→14→19→24→...→49→53; z: 6→11→16→...→46→50; w: 5→10→15→...→45→50→51) before it terminates — z's spiraling loop-estimate finally exceeds its own direct escape route (c(x,z)=50), so z abandons the loop and switches to its direct link; that correct information then cascades outward and the network restabilizes at **round 33** with the true optimum: **z=50 (direct)**, **w=51 (via z)**, **y=52 (via w)**.

**(c)** To eliminate the count-to-infinity problem entirely when c(y,x) changes 4→60, break the y–z–w cycle by making the **direct z–x link the unconditional best choice for z from the start**, so z never needs to route through the loop at all. Concretely, **increase c(y,z)** enough that z's *only* attractive route to x is always its direct 50-cost link — e.g. set c(y,z) ≥ 50 (so any path z→y→…→x costs ≥50+4=54, never beating the direct 50). With that change, z never enters the loop in the first place (before *or* after the y–x cost change), so w and y's escalating exchange has nothing to bounce off, and the whole scenario simply becomes a fast, direct convergence instead of a slow spiral.

### P12 — Loop detection in BGP
Via the **AS-PATH** attribute: every AS that re-advertises a route prepends its own ASN to the path list. When any router receives an advertisement, it checks whether **its own AS number already appears anywhere in the AS-PATH** — if so, it rejects/discards the advertisement outright, since accepting it would mean routing back into an AS the packet has (per the path) already passed through, i.e. a loop.

### P13 — Does BGP always pick the loop-free shortest-AS-path route?
**No.** BGP's route-selection algorithm applies **local preference first** (rule 1), and only falls through to **shortest AS-PATH** (rule 2) as a tie-breaker among routes sharing the *same* (highest) local preference. Local preference is a pure policy knob set by the network administrator — an AS can, and routinely does, set a higher local preference on a *longer* AS-PATH route (e.g. because it's a paying customer relationship rather than a free peering link), causing that longer-but-preferred route to win outright over a shorter but less-preferred one.

### P14 — Which protocol does each router learn x from?
*(x is attached at router 4a inside AS4; AS3 and AS2 run OSPF internally, AS1 and AS4 run RIP internally; no AS2–AS4 link yet.)*

**(a) Router 3c** (AS3's gateway peered directly with AS4's gateway 4c): learns x via **eBGP** (direct external session with 4c, who itself learned it internally via RIP from 4a).

**(b) Router 3a** (internal to AS3, not touching the AS4 border): learns x via **iBGP** (propagated internally from gateway 3c).

**(c) Router 1c** (AS1's gateway peered directly with AS3's gateway 3a): learns x via **eBGP** (direct external session with 3a).

**(d) Router 1d** (internal to AS1, touches neither AS2 nor AS3 directly): learns x via **iBGP** (propagated internally from gateway 1c).

### P15 — Which interface (I1 or I2) does router 1d use?
*(1d connects to 1a via I1 and to 1b via I2; 1a leads toward AS3's gateway 1c; 1b leads toward AS2's gateway.)*

**(a) I = I1.** With no AS2–AS4 link yet, the *only* AS path to x runs through AS3, so the only usable direction is toward AS3 — via 1a, i.e. interface I1.

**(b) I = I2.** Once the AS2–AS4 link exists, both routes have equal AS-PATH length (AS3-AS4 vs. AS2-AS4, both 2 AS-hops) — with rule 2 tied, BGP falls to **hot-potato routing** (rule 3), and 1b (the AS2-facing gateway) is a direct, 1-hop neighbor of 1d, while 1c (the AS3-facing gateway) is 2 intra-AS hops away — so 1d forwards out the closer gateway, I2.

**(c) I = I1 again.** Inserting AS5 on the AS2 path makes that AS-PATH length 3 (AS2-AS5-AS4) versus the AS3 path's still-2 (AS3-AS4) — now rule 2 (shortest AS-PATH) breaks the tie *before* hot-potato ever gets consulted, so the shorter AS3 route wins outright regardless of physical proximity, sending traffic back out I1.

### P16 — Getting ISP B to hand off at its East Coast peering point
C can set the **MED (Multi-Exit Discriminator)** attribute on the routes it advertises to B at each of the two peering points — advertising a **lower MED value at the East Coast** peering point than at the West Coast one. MED is a hint from the *advertising* AS telling a neighboring AS which of several entry points it prefers traffic be handed off at, and (when B honors MED from routes learned from the same neighboring AS C, which is common but requires mutual agreement since it isn't mandatory) B's tie-breaking will favor the lower-MED, i.e. East Coast, route — achieving exactly the outcome C wants instead of B's default hot-potato preference (which would otherwise favor whichever peering point is closest to B itself).

### P17 — W's and X's views of the topology (Figure 5.13: W-A, A-B, A-C, B-C, B-X, C-X, C-Y)
*Y's given view (a simple tree: W-A-C-X, C-Y) reflects that Y, a single-homed stub, only ever receives ONE best route per destination from its one provider C — it never sees any path C didn't actually pick.*

**W's view:** also a simple **tree**, since W too is single-homed (only provider A). W learns whichever AS-PATHs A actually advertises to it: directly-connected B and A directly-connected C (both 1 AS-hop from A, both get advertised), then A's chosen best path onward to X (either via B or via C — an AS-PATH tie A must break one way, e.g. arbitrarily toward C), and to Y (only reachable via C, since only C touches Y). Either way, **W never learns about the edge A didn't use** — e.g. if A picked C→X, W has no idea a B–X link even exists.

**X's view:** genuinely richer, because X is **multi-homed** — it has two providers (B and C) who each, per the "full BGP info to customers" rule, tell X everything they know, including about each other. From B, X learns B-A, B-C, and B's paths onward to W and Y; from C, X learns C-A, C-B, and C's paths onward to W and Y. Combining both feeds, X effectively reconstructs the **entire actual A-B-C triangle** (unlike W or Y, who each only see whichever single tree their one provider chose to advertise) — this is exactly the point of the exercise: a multi-homed stub's view of the network is structurally more complete than a single-homed stub's.

### P18 — An application whose data doesn't follow BGP's route
BGP-computed routes only describe **network-layer (IP)** paths — they say nothing about what happens at the **application layer**. A classic example: a **P2P file-sharing / overlay-relay application** (e.g. a BitTorrent-like client, or an old-style P2P VoIP supernode relay). A peer physically located inside X's network can fetch data from a peer inside B's network over one ordinary IP-layer connection (B→X), then — at the *application* layer — relay/forward that same data onward over a second, separate IP-layer connection to a peer inside Y's network (X→Y). No single network-layer route ever goes B→X→Y (BGP would never produce that, exactly as stated), but the **application** stitches two independent, BGP-compliant IP paths together via an overlay hop, so data still effectively flows that way end to end.

### P19 — Stub V added; A wants W-traffic from B only, V-traffic from either
**Advertising policy:** A should advertise its route to **W only to B** (withhold it from C entirely), and advertise its route to **V to both B and C**.

**What AS routes does C receive?** For **V**, C receives a **direct** route straight from A, AS-PATH = "A V" (since A advertised V to both providers). For **W**, C receives **no direct route from A** — but since B, as a matter of normal customer-route export policy, re-advertises the customer route it learned from A on to its peer C, **C still ends up with an *indirect* route to W via B**, AS-PATH = "B A W". This indirect route is longer/worse than a hypothetical direct one, but it still guarantees that *any* traffic C forwards toward W necessarily transits B first — so it physically enters A's network over the B–A link either way, achieving A's actual goal (W-bound traffic always arrives via B) without C being cut off from reaching W altogether.

### P20 — Can Z selectively transit Y's traffic but not X's? (X–Y peer, Y–Z peer, X and Z not directly connected)
**No, BGP does not allow this.** BGP's route advertisement and filtering machinery operates purely on **destination prefixes** — an AS decides which prefixes to accept/advertise to a given neighbor, but it has **no visibility into, or control based on, the ultimate *source* AS of the packets that actually flow along a resulting route**. Since X and Z aren't directly connected, any of X's traffic reaching Z necessarily arrives *via Y* — indistinguishable, at the IP/BGP level, from traffic that genuinely originated in Y's own network. Whatever export/import policy Z applies to its Y-facing session affects **all** traffic Y forwards across that link, regardless of whether the ultimate source was Y or was merely transiting through Y from X. Z can refuse to transit *specific prefixes* for Y, but it cannot condition that refusal on "was this particular packet's true origin X or Y" — that distinction simply isn't part of what BGP (a destination-based, source-oblivious protocol) is capable of expressing.
