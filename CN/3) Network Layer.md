# 📘 CN Chapter 3 — Network Layer: IP, Subnetting, CIDR, VLSM & Routing 
---

## 📌 Table of Contents
1. [IPv4 Addressing](#1-ipv4-addressing)
2. [Classful Addressing](#2-classful-addressing)
3. [Subnetting — Step-by-Step](#3-subnetting--step-by-step)
4. [CIDR — Classless Inter-Domain Routing](#4-cidr--classless-inter-domain-routing)
5. [VLSM — Variable Length Subnet Mask](#5-vlsm--variable-length-subnet-mask)
6. [NAT — Network Address Translation](#6-nat--network-address-translation)
7. [IPv6](#7-ipv6)
8. [Routing — Concepts & Algorithms](#8-routing--concepts--algorithms)
9. [Routing Protocols — RIP, OSPF, BGP](#9-routing-protocols--rip-ospf-bgp)
10. [ICMP & Ping](#10-icmp--ping)
11. [Numericals — Full Solved Sets](#11-numericals--full-solved-sets)
12. [MCQ Traps & Exam Q&A](#12-mcq-traps--exam-qa)

---

## 1. IPv4 Addressing

### 🔧 IPv4 Structure
```
32-bit address → written in dotted decimal: w.x.y.z
Each octet = 8 bits = 0 to 255

Example: 192.168.1.100
Binary:  11000000.10101000.00000001.01100100

Total possible addresses: 2^32 = 4,294,967,296 ≈ 4.3 billion
```

### 🔧 Special Addresses
```
Network address:    All HOST bits = 0     (identifies network)
Broadcast address:  All HOST bits = 1     (send to all hosts on network)
First usable host:  Network address + 1
Last usable host:   Broadcast address - 1

Example: 192.168.1.0/24
  Network:   192.168.1.0
  Broadcast: 192.168.1.255
  First host:192.168.1.1
  Last host: 192.168.1.254
  Usable hosts: 254 (= 2^8 - 2)

Special ranges:
  0.0.0.0/8         This network (source only)
  127.0.0.0/8       Loopback (127.0.0.1 = localhost)
  10.0.0.0/8        Private Class A
  172.16.0.0/12     Private Class B
  192.168.0.0/16    Private Class C
  255.255.255.255   Limited broadcast
  169.254.0.0/16    APIPA (Auto-config when DHCP fails)
  224.0.0.0/4       Multicast
```

---

## 2. Classful Addressing

### 🔧 Five Classes
```
Class A: 0xxxxxxx.hosthost.hosthost.hosthost
  First bit = 0
  Range: 1.0.0.0 – 126.255.255.255  (0.x.x.x reserved, 127.x.x.x loopback)
  Default mask: /8 (255.0.0.0)
  Networks: 2^7 - 2 = 126
  Hosts per network: 2^24 - 2 = 16,777,214

Class B: 10xxxxxx.xxxxxxxx.hosthost.hosthost
  First two bits = 10
  Range: 128.0.0.0 – 191.255.255.255
  Default mask: /16 (255.255.0.0)
  Networks: 2^14 = 16,384
  Hosts per network: 2^16 - 2 = 65,534

Class C: 110xxxxx.xxxxxxxx.xxxxxxxx.hosthost
  First three bits = 110
  Range: 192.0.0.0 – 223.255.255.255
  Default mask: /24 (255.255.255.0)
  Networks: 2^21 = 2,097,152
  Hosts per network: 2^8 - 2 = 254

Class D: 1110xxxx... (Multicast)
  Range: 224.0.0.0 – 239.255.255.255
  No subnet mask

Class E: 1111xxxx... (Experimental)
  Range: 240.0.0.0 – 255.255.255.255
  Reserved
```

---

## 3. Subnetting — Step-by-Step

### 🧠 Core Formula
```
Given a network and need N subnets (or H hosts per subnet):

Subnet bits borrowed: s where 2^s ≥ N
Host bits remaining:  h where 2^h - 2 ≥ H (hosts needed)

New prefix length = original prefix + s

Subnet mask: all 1s in network+subnet bits, all 0s in host bits.

For each subnet:
  Subnet address:    host bits all 0
  Broadcast:         host bits all 1
  Usable hosts:      subnet+1 to broadcast-1
  Usable count:      2^h - 2
```

### 📝 Subnetting Master Numerical

**Network: 192.168.10.0/24. Divide into 4 equal subnets.**

```
Step 1: Need 4 subnets.
  s bits needed: 2^s ≥ 4 → s = 2

Step 2: New prefix = 24 + 2 = /26
  New subnet mask = 255.255.255.192
  (26 ones: 11111111.11111111.11111111.11000000)

Step 3: Host bits remaining = 32 - 26 = 6
  Hosts per subnet = 2^6 - 2 = 62

Step 4: Subnet increment = 2^(host bits) = 2^6 = 64

Step 5: List all subnets (increment by 64 in 4th octet):

  Subnet 0: 192.168.10.0/26
    Network:   192.168.10.0
    Broadcast: 192.168.10.63
    Hosts:     192.168.10.1 – 192.168.10.62   (62 hosts)

  Subnet 1: 192.168.10.64/26
    Network:   192.168.10.64
    Broadcast: 192.168.10.127
    Hosts:     192.168.10.65 – 192.168.10.126  (62 hosts)

  Subnet 2: 192.168.10.128/26
    Network:   192.168.10.128
    Broadcast: 192.168.10.191
    Hosts:     192.168.10.129 – 192.168.10.190 (62 hosts)

  Subnet 3: 192.168.10.192/26
    Network:   192.168.10.192
    Broadcast: 192.168.10.255
    Hosts:     192.168.10.193 – 192.168.10.254 (62 hosts)
```

### 🔧 Finding Subnet for a Given IP

**Which subnet does 192.168.10.100/26 belong to?**
```
Subnet mask = /26 = 255.255.255.192

Method: AND the IP with the subnet mask.
  192.168.10.100 = 11000000.10101000.00001010.01100100
  255.255.255.192 = 11111111.11111111.11111111.11000000
  AND:              11000000.10101000.00001010.01000000
                  = 192.168.10.64

→ Belongs to subnet 192.168.10.64/26 ✓
```

### 🔧 Subnet Mask Quick Reference Table

| Prefix | Mask | Subnets from /24 | Hosts/subnet |
|---|---|---|---|
| /24 | 255.255.255.0 | 1 | 254 |
| /25 | 255.255.255.128 | 2 | 126 |
| /26 | 255.255.255.192 | 4 | 62 |
| /27 | 255.255.255.224 | 8 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 32 | 6 |
| /30 | 255.255.255.252 | 64 | 2 |
| /32 | 255.255.255.255 | — | 1 (host route) |

**Memorize the 4th octet values: 128, 192, 224, 240, 248, 252**

---

## 4. CIDR — Classless Inter-Domain Routing

### 🧠 Concept
```
CIDR (RFC 1519): Ignore the class boundaries of IPv4.
Any address can be any prefix length (not just /8, /16, /24).
Written as: IP/prefix (e.g., 10.0.0.0/22)

CIDR AGGREGATION (Route Summarization):
  Combine multiple contiguous subnets into ONE route.
  Reduces routing table size.

Example:
  192.168.0.0/24
  192.168.1.0/24
  192.168.2.0/24
  192.168.3.0/24
  → All can be summarized as 192.168.0.0/22

VERIFICATION: All 4 networks share first 22 bits.
  192.168.0.x = 11000000.10101000.00000000.xxxxxxxx
  192.168.3.x = 11000000.10101000.00000011.xxxxxxxx
  First 22 bits: 11000000.10101000.000000 ← same!
  So /22 covers all 4 networks. ✓
```

### 📝 CIDR Aggregation Numerical

**Can 172.16.12.0/24, 172.16.13.0/24, 172.16.14.0/24, 172.16.15.0/24 be summarized?**

```
Convert 3rd octets to binary:
  12 = 00001100
  13 = 00001101
  14 = 00001110
  15 = 00001111

Shared prefix: 000011xx → first 22 bits are common.

Summary: 172.16.12.0/22
  Network: 172.16.12.0
  Broadcast: 172.16.15.255
  Covers all 4 /24 networks ✓
```

---

## 5. VLSM — Variable Length Subnet Mask

### 🧠 Concept
```
VLSM: Different subnets can have DIFFERENT sizes.
      Allocate subnet sizes based on actual NEED.
      Avoids wasting address space.

Strategy: Allocate LARGEST subnets first, then smaller ones.
          Always uses next available address block.
```

### 📝 VLSM Numerical — Full Allocation

**Network: 192.168.1.0/24. Requirements:**
- Subnet A: 100 hosts
- Subnet B: 50 hosts
- Subnet C: 25 hosts
- Subnet D: 10 hosts
- Router links: 2 hosts each (×3 = 3 links)

**Allocate using VLSM (largest first).**

```
Sort by size (largest first):
  A=100, B=50, C=25, D=10, Links=2(×3)

SUBNET A (100 hosts):
  Need: 2^h - 2 ≥ 100 → h=7 (2^7-2=126 ≥ 100)
  Prefix: /25, block size: 128
  Network:   192.168.1.0/25
  Broadcast: 192.168.1.127
  Hosts:     .1 to .126
  Next available: 192.168.1.128

SUBNET B (50 hosts):
  Need: 2^h - 2 ≥ 50 → h=6 (2^6-2=62 ≥ 50)
  Prefix: /26, block size: 64
  Network:   192.168.1.128/26
  Broadcast: 192.168.1.191
  Hosts:     .129 to .190
  Next available: 192.168.1.192

SUBNET C (25 hosts):
  Need: 2^h - 2 ≥ 25 → h=5 (2^5-2=30 ≥ 25)
  Prefix: /27, block size: 32
  Network:   192.168.1.192/27
  Broadcast: 192.168.1.223
  Hosts:     .193 to .222
  Next available: 192.168.1.224

SUBNET D (10 hosts):
  Need: 2^h - 2 ≥ 10 → h=4 (2^4-2=14 ≥ 10)
  Prefix: /28, block size: 16
  Network:   192.168.1.224/28
  Broadcast: 192.168.1.239
  Hosts:     .225 to .238
  Next available: 192.168.1.240

LINK 1 (2 hosts):
  Need: h=2 (2^2-2=2 exactly)
  Prefix: /30, block size: 4
  Network:   192.168.1.240/30
  Broadcast: 192.168.1.243
  Next available: 192.168.1.244

LINK 2: 192.168.1.244/30 (broadcast .247)
LINK 3: 192.168.1.248/30 (broadcast .251)

Remaining: 192.168.1.252 – .255 (4 addresses unused)
Total used: 128+64+32+16+4+4+4 = 252 out of 256 ✓
```

---

## 6. NAT — Network Address Translation

### 🔧 Types of NAT
```
STATIC NAT:   One private IP ↔ One public IP (1:1 mapping, permanent)
DYNAMIC NAT:  Pool of public IPs. Assign on demand to private IPs.
PAT (Port Address Translation) / NAT Overload:
  Many private IPs → ONE public IP, distinguished by PORT numbers.
  Example: 192.168.1.10:1234 → 203.0.113.5:10001
           192.168.1.11:5678 → 203.0.113.5:10002
  Most common form ("NAT" usually means PAT in practice).

BENEFITS:
  - Conserves public IPv4 addresses
  - Hides internal network structure (security)
  - Easy to change ISP (only public IP changes)

DRAWBACKS:
  - Breaks end-to-end connectivity
  - Some applications need special handling (VoIP, FTP)
  - Adds latency (translation table lookup)
  - Makes inbound connections harder
```

---

## 7. IPv6

### 🔧 IPv6 Basics
```
128-bit addresses → written in hex with colons: 2001:0db8:85a3:0000:0000:8a2e:0370:7334

COMPRESSION RULES:
  1. Leading zeros in each group can be omitted:
     0db8 → db8,  0000 → 0
  2. One consecutive group of all-zero groups → :: (double colon)
     Only ONCE per address!

Example: 2001:0db8:0000:0000:0001:0000:0000:0001
  Step 1: Remove leading zeros: 2001:db8:0:0:1:0:0:1
  Step 2: Compress longest zero run: 2001:db8::1:0:0:1
          OR: 2001:db8:0:0:1::1
          (can't compress both groups — only one :: allowed)

ADDRESS TYPES:
  Unicast:   one-to-one (like IPv4 unicast)
  Multicast: one-to-many (FF00::/8)
  Anycast:   one-to-nearest (same address on multiple devices)
  (No broadcast in IPv6! Multicast replaces it)

LOOPBACK: ::1 (equivalent to 127.0.0.1)
ALL ZEROS: :: (unspecified)
Link-local: FE80::/10 (auto-configured, not routable)
Global unicast: 2000::/3 (internet-routable)
```

### 🔧 IPv4 vs IPv6
| Feature | IPv4 | IPv6 |
|---|---|---|
| **Address length** | 32 bits | 128 bits |
| **Notation** | Dotted decimal | Colon-separated hex |
| **Address space** | ~4.3 billion | 3.4 × 10^38 |
| **Header size** | 20-60 bytes | 40 bytes (fixed!) |
| **Fragmentation** | Routers can fragment | Only source can fragment |
| **Checksum** | In header | Removed (L4 handles it) |
| **Broadcast** | Yes | No (multicast instead) |
| **ARP** | Yes | Replaced by NDP (Neighbor Discovery) |
| **DHCP** | Optional | SLAAC (auto-config) |
| **IPSec** | Optional | Mandatory |

---

## 8. Routing — Concepts & Algorithms

### 🔧 Routing vs Forwarding
```
ROUTING:    Process of finding the best path. (Control plane)
            Routing algorithm runs, updates routing table.

FORWARDING: Moving a packet from input to output interface. (Data plane)
            Fast lookup in forwarding table (FIB).

ROUTING TABLE: Contains network prefixes + next-hop info.
  Destination | Next Hop | Interface | Metric
  10.0.0.0/8  | 192.168.1.1 | eth0 | 10
```

### 🔧 Distance Vector Routing
```
Each router knows:
  - Distance to each destination (metric)
  - Direction (next-hop router)

Periodic updates: send routing table to IMMEDIATE NEIGHBORS only.
Uses BELLMAN-FORD algorithm.

COUNT-TO-INFINITY problem:
  When a link fails, routers keep incrementing metric trying to route
  through each other → count up to infinity.

Solutions:
  Split Horizon: Don't advertise route back to the neighbor you learned it from.
  Poison Reverse: Advertise route back with metric = infinity (16 in RIP).
  Hold-down timers: Ignore updates for a period after learning route is down.
```

### 🔧 Link State Routing
```
Each router:
  1. Discovers neighbors and their link costs (via HELLO packets)
  2. Broadcasts Link State Advertisement (LSA) to ALL routers (flooding)
  3. Each router builds complete network topology (graph)
  4. Runs Dijkstra's algorithm on full graph → shortest path tree
  5. Installs best paths in routing table

Faster convergence than Distance Vector.
More memory and CPU (full topology).
```

### 🔧 Dijkstra's Shortest Path — Key for Exams

**Network graph:**
```
    A ──5── B ──3── E
    │       │
    2       1
    │       │
    C ──4── D

Find shortest paths from A.
```

```
Initialize: dist = {A:0, B:∞, C:∞, D:∞, E:∞}
            visited = {}

Step 1: Visit A (dist=0).
  Update neighbors: B=5, C=2.
  dist = {A:0, B:5, C:2, D:∞, E:∞}
  visited = {A}

Step 2: Visit C (smallest unvisited dist=2).
  Update: D = 2+4 = 6.
  dist = {A:0, B:5, C:2, D:6, E:∞}
  visited = {A, C}

Step 3: Visit B (dist=5).
  Update: E = 5+3 = 8. D = min(6, 5+1=6) = 6 (no change).
  dist = {A:0, B:5, C:2, D:6, E:8}
  visited = {A, C, B}

Step 4: Visit D (dist=6). No better paths found.
  visited = {A, C, B, D}

Step 5: Visit E (dist=8). Done.

Shortest paths from A:
  A→C: 2 (direct)
  A→B: 5 (direct)
  A→D: 6 (A→C→D)
  A→E: 8 (A→B→E)
```

---

## 9. Routing Protocols — RIP, OSPF, BGP

### 🔧 RIP — Routing Information Protocol

```
TYPE: Distance Vector (Bellman-Ford)
METRIC: Hop count (each router = 1 hop)
MAX HOPS: 15 (16 = infinity = unreachable)
UPDATE: Every 30 seconds (periodic)
CONVERGENCE: Slow (count-to-infinity problem)

VERSIONS:
  RIPv1: Classful (doesn't send subnet masks), no authentication
  RIPv2: Classless (sends subnet masks), supports VLSM, MD5 auth
  RIPng: IPv6 version

TIMERS:
  Update timer:    30 sec (send routing table)
  Invalid timer:   180 sec (route becomes invalid if no update)
  Hold-down timer: 180 sec (ignore updates to prevent loops)
  Flush timer:     240 sec (remove route from table)

BEST FOR: Small networks (≤ 15 hops)
```

### 🔧 OSPF — Open Shortest Path First

```
TYPE: Link State (Dijkstra)
METRIC: Cost (based on bandwidth: cost = 10^8 / interface_bandwidth)
MAX HOPS: No limit
UPDATE: Triggered (only when topology changes, not periodic)
CONVERGENCE: Fast
STANDARD: Open standard (RFC 2328)

HIERARCHY:
  Backbone area: Area 0 (all other areas must connect to it)
  Regular areas: Area 1, 2, 3...
  ABR (Area Border Router): connects areas to backbone
  ASBR (AS Boundary Router): connects to external AS

PACKET TYPES:
  Hello:       discover/maintain neighbors
  DBD:         Database Description (topology summary)
  LSR:         Link State Request
  LSU:         Link State Update (contains LSAs)
  LSAck:       Link State Acknowledgment

ROUTER TYPES:
  DR (Designated Router):     elected on broadcast networks
  BDR (Backup DR):            takes over if DR fails
  Election: based on priority (default=1), then highest Router ID

OSPF vs RIP:
  OSPF: faster convergence, no hop limit, supports VLSM, hierarchical
  RIP:  simpler, less CPU/memory, small networks only
```

### 🔧 BGP — Border Gateway Protocol

```
TYPE: Path Vector (hybrid distance-vector with full path info)
USE:  Routing BETWEEN Autonomous Systems (Internet's routing protocol)
METRIC: Multiple attributes (AS-PATH, WEIGHT, LOCAL_PREF, MED...)
PORT:  TCP port 179
CONVERGENCE: Slow (designed for stability, not speed)
STANDARD: RFC 4271

KEY CONCEPTS:
  AS (Autonomous System): network under single administrative control
  IBGP: BGP within same AS
  EBGP: BGP between different ASes

MOST IMPORTANT ATTRIBUTE: AS-PATH
  List of ASes packet must traverse.
  Shorter AS-PATH preferred.
  Also used for loop detection (don't accept if own AS in path).

BGP vs OSPF:
  BGP: inter-AS (between ISPs), policy-based, TCP-based
  OSPF: intra-AS (within company), metric-based, IP-based
```

### 📊 Routing Protocol Comparison

| Feature | RIP | OSPF | BGP |
|---|---|---|---|
| **Type** | Distance Vector | Link State | Path Vector |
| **Algorithm** | Bellman-Ford | Dijkstra | (custom) |
| **Metric** | Hop count | Cost (BW) | Multiple attributes |
| **Max hops** | 15 | Unlimited | Unlimited |
| **Convergence** | Slow | Fast | Slow |
| **Updates** | Periodic (30s) | Triggered | Triggered |
| **Scope** | Intra-AS | Intra-AS | **Inter-AS** |
| **Transport** | UDP port 520 | IP (protocol 89) | TCP port 179 |
| **Use case** | Small LAN | Enterprise | Internet |

---

## 10. ICMP & Ping

### 🔧 ICMP — Internet Control Message Protocol
```
IP provides best-effort delivery — no error reporting built in.
ICMP: error messages and operational information for IP.
Lives at Network Layer (encapsulated in IP packet).

ICMP MESSAGE TYPES:
  Type 0: Echo Reply (ping response)
  Type 3: Destination Unreachable (and codes: 0=network, 1=host, 3=port...)
  Type 4: Source Quench (deprecated — slow down request)
  Type 5: Redirect (better route available)
  Type 8: Echo Request (ping)
  Type 11: Time Exceeded (TTL expired — used by traceroute)
  Type 12: Parameter Problem
```

### 🔧 ping
```
Sends ICMP Echo Request → waits for ICMP Echo Reply.
Tests: reachability, round-trip time, packet loss.

ping 8.8.8.8:
  PING 8.8.8.8: 56 data bytes
  64 bytes from 8.8.8.8: icmp_seq=1 ttl=57 time=12.5 ms
  64 bytes from 8.8.8.8: icmp_seq=2 ttl=57 time=11.8 ms
  ...
  --- 8.8.8.8 ping statistics ---
  4 packets transmitted, 4 received, 0% packet loss
  round-trip min/avg/max = 11.8/12.1/12.5 ms
```

### 🔧 traceroute
```
Exploits TTL field to trace route.
  TTL = 1: first router sends ICMP Time Exceeded → reveals hop 1.
  TTL = 2: second router sends Time Exceeded → reveals hop 2.
  Continue until destination reached (ICMP Echo Reply or port unreachable).

Uses UDP in Unix, ICMP Echo in Windows (tracert).
```

### 🔧 TTL — Time to Live
```
Each router DECREMENTS TTL by 1.
If TTL reaches 0: router drops packet + sends ICMP Type 11 Time Exceeded.
Prevents packets from circling forever.
Typical starting TTL: Linux=64, Windows=128, Cisco=255.
```

---

## 11. Numericals — Full Solved Sets

### 📝 Numerical 1 — Find Subnet Details

**IP: 172.16.45.200/20. Find: network address, broadcast, first host, last host, number of hosts.**

```
/20 = 20 ones: 11111111.11111111.11110000.00000000 = 255.255.240.0

172.16.45.200 in binary:
  172.16.45.200 = 10101100.00010000.00101101.11001000

Subnet mask:       11111111.11111111.11110000.00000000

Network address (AND):
  10101100.00010000.00100000.00000000 = 172.16.32.0

Broadcast (set all host bits to 1):
  10101100.00010000.00101111.11111111 = 172.16.47.255

First host: 172.16.32.1
Last host:  172.16.47.254
Hosts: 2^12 - 2 = 4094
```

---

### 📝 Numerical 2 — CIDR Block Size

**How many addresses are in 10.0.0.0/12?**

```
/12 = 12 network bits → 32-12 = 20 host bits
Addresses = 2^20 = 1,048,576

Range:
  10.0.0.0 = 00001010.00000000.00000000.00000000
  Mask /12: 11111111.11110000.00000000.00000000
  Broadcast: 00001010.00001111.11111111.11111111 = 10.15.255.255

Covers: 10.0.0.0 to 10.15.255.255
```

---

### 📝 Numerical 3 — Subnetting for Specific Hosts

**ISP gives you 203.0.113.0/24. You need 5 subnets with at least 30 hosts each. What's the best prefix?**

```
Need: 30 hosts → 2^h - 2 ≥ 30 → h = 5 (2^5-2=30 exactly ✓)
Prefix = 32 - 5 = /27
Block size = 2^5 = 32

Need 5 subnets. Subnets from /24 with /27 = 2^(27-24) = 2^3 = 8 subnets.
8 ≥ 5 ✓

Subnets:
  203.0.113.0/27   (hosts .1-.30,   broadcast .31)
  203.0.113.32/27  (hosts .33-.62,  broadcast .63)
  203.0.113.64/27  (hosts .65-.94,  broadcast .95)
  203.0.113.96/27  (hosts .97-.126, broadcast .127)
  203.0.113.128/27 (hosts .129-.158, broadcast .159)
  ... (3 more available for future)
```

---

### 📝 Numerical 4 — RIP Distance Vector Convergence

**Router R1 routing table:**
```
Dest    | Next | Hops
N1      | —    | 0  (directly connected)
N2      | R2   | 3
N3      | R3   | 7

R2 sends update to R1:
N2: 1 hop (R2 directly connected)
N3: 2 hops (R2→R3→N3? wait no)
Actually R2 sends:
N1: 5 hops
N2: 0 hops (directly connected to R2)
N3: 3 hops
```

**R1 processes R2's update (add 1 hop for link R1-R2):**
```
For each route in R2's table, add 1 (cost of R1→R2 link):
  N1 via R2: 5+1=6. Current N1=0 (better). NO UPDATE.
  N2 via R2: 0+1=1. Current N2=3 (worse). UPDATE! N2=1 via R2.
  N3 via R2: 3+1=4. Current N3=7 (worse). UPDATE! N3=4 via R2.

Updated R1 table:
  N1: 0 (direct)
  N2: 1 (via R2)  ← improved from 3
  N3: 4 (via R2)  ← improved from 7
```

---

## 12. MCQ Traps & Exam Q&A

### ⚠️ THE TRAPS

---

**TRAP 1: Network and broadcast addresses are NOT usable**
```
Network address (all host bits 0): identifies the network, not assignable to host.
Broadcast address (all host bits 1): sends to all hosts, not assignable.
Usable hosts = 2^h - 2 (subtract both)
EXCEPTION: /31 and /32 have special rules (RFC 3021 allows /31 for point-to-point)
```

---

**TRAP 2: /30 gives only 2 usable hosts (for router links)**
```
/30: 2^2 - 2 = 2 hosts. Perfect for router-to-router links (need exactly 2 IPs).
/31: technically 0 usable hosts in classful, but RFC 3021 allows it for P2P.
/32: single host route (loopback, specific host).
```

---

**TRAP 3: CIDR aggregation — addresses must be CONTIGUOUS and start at correct boundary**
```
Can aggregate 192.168.0.0/24 + 192.168.1.0/24 → 192.168.0.0/23 ✓
Can aggregate 192.168.2.0/24 + 192.168.3.0/24 → 192.168.2.0/23 ✓
CANNOT aggregate 192.168.1.0/24 + 192.168.2.0/24 → 192.168.1.0/23 ✗
(192.168.1.0/23 would cover 192.168.0.0 and 192.168.1.0, not .1 and .2)
```

---

**TRAP 4: RIP max hop count is 15, NOT 16**
```
Hop count 16 = INFINITY = unreachable.
Max reachable = 15 hops.
Networks > 15 hops away = unreachable in RIP.
```

---

**TRAP 5: OSPF cost formula**
```
Cost = 10^8 / bandwidth (in bps)
  100 Mbps link: cost = 10^8 / 10^8 = 1
  10 Mbps link:  cost = 10^8 / 10^7 = 10
  1 Gbps link:   cost = 10^8 / 10^9 = 0.1 → rounds to 1
  (Gigabit gets same cost as FastEthernet — workaround: use ref BW 10^9)
```

---

**TRAP 6: BGP uses TCP, others use UDP or IP directly**
```
BGP: TCP port 179
RIP: UDP port 520
OSPF: IP directly (protocol number 89, not TCP/UDP)
```

---

### ⚡ Quick Fire Q&A

| Question | Answer |
|---|---|
| Class A default mask? | /8 (255.0.0.0) |
| Class B default mask? | /16 (255.255.0.0) |
| Class C default mask? | /24 (255.255.255.0) |
| Private Class A range? | 10.0.0.0/8 |
| Private Class B range? | 172.16.0.0/12 |
| Private Class C range? | 192.168.0.0/16 |
| Loopback address? | 127.0.0.1 |
| APIPA range? | 169.254.0.0/16 |
| /26 gives how many hosts? | 62 |
| /28 gives how many hosts? | 14 |
| /30 gives how many hosts? | 2 |
| Hosts in /n network? | 2^(32-n) - 2 |
| VLSM: allocate in what order? | Largest subnet first |
| RIP metric? | Hop count |
| OSPF metric? | Cost (bandwidth-based) |
| BGP used where? | Between Autonomous Systems (Internet) |
| RIP max hops? | 15 (16 = infinity) |
| OSPF backbone area number? | Area 0 |
| BGP port? | TCP 179 |
| ICMP Type 8? | Echo Request (ping) |
| ICMP Type 0? | Echo Reply (ping response) |
| ICMP Type 11? | Time Exceeded (TTL expired — traceroute) |
| ICMP Type 3? | Destination Unreachable |
| TTL at 0 — what happens? | Router drops + sends ICMP Type 11 |
| IPv6 address length? | 128 bits |
| IPv6 loopback? | ::1 |
| IPv6 has broadcast? | No — uses multicast instead |
| NAT Overload = ? | PAT (Port Address Translation) |

---
