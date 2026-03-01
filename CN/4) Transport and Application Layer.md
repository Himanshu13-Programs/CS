# 📘 CN Chapter 4 — Transport & Application Layer 
---

## 📌 Table of Contents
1. [Transport Layer — Overview](#1-transport-layer--overview)
2. [TCP — Deep Dive](#2-tcp--deep-dive)
3. [TCP 3-Way Handshake](#3-tcp-3-way-handshake)
4. [TCP Flow Control — Sliding Window](#4-tcp-flow-control--sliding-window)
5. [TCP Congestion Control](#5-tcp-congestion-control)
6. [UDP — User Datagram Protocol](#6-udp--user-datagram-protocol)
7. [TCP vs UDP Comparison](#7-tcp-vs-udp-comparison)
8. [Application Layer Protocols](#8-application-layer-protocols)
9. [DNS — Domain Name System](#9-dns--domain-name-system)
10. [DHCP](#10-dhcp)
11. [HTTP & HTTPS](#11-http--https)
12. [FTP, SMTP, POP3, IMAP](#12-ftp-smtp-pop3-imap)
13. [Numericals — Full Solved Sets](#13-numericals--full-solved-sets)
14. [MCQ Traps & Exam Q&A](#14-mcq-traps--exam-qa)

---

## 1. Transport Layer — Overview

```
PURPOSE: End-to-end communication between APPLICATIONS (processes).
         Adds port numbers to identify which application gets the data.

KEY RESPONSIBILITIES:
  Multiplexing/Demultiplexing: using port numbers to route to correct process
  Segmentation:  break large messages into smaller segments
  Reassembly:    put segments back together at destination
  Error detection: checksum in both TCP and UDP
  Flow control:  (TCP only) don't overwhelm receiver
  Congestion control: (TCP only) don't overwhelm network
  Reliability:   (TCP only) ensure all data arrives correctly

SOCKET = IP address + Port number
  Identifies a specific process on a specific host.
  Example: 192.168.1.5:80 = web server on that host
  Connection identified by 4-tuple: (src IP, src port, dst IP, dst port)
```

---

## 2. TCP — Deep Dive

### 🔧 TCP Segment Header
```
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 ┌─────────────────────────────┬─────────────────────────────────┐
 │        Source Port          │       Destination Port          │
 ├─────────────────────────────────────────────────────────────┤
 │                     Sequence Number                          │
 ├─────────────────────────────────────────────────────────────┤
 │                  Acknowledgment Number                       │
 ├──────┬─────────┬─────────────────────────────────────────┤
 │ Data │Reserved │ U A P R S F │        Window Size          │
 │ Offset│        │ R C S S Y I │                             │
 │(4 bit)│        │ G K H T N N │                             │
 ├──────┴─────────┴─────────────┬─────────────────────────────┤
 │           Checksum            │       Urgent Pointer        │
 ├───────────────────────────────┴─────────────────────────────┤
 │                    Options (variable)                        │
 └─────────────────────────────────────────────────────────────┘

FIELDS:
Source/Dest Port: 16 bits each (0-65535)
Sequence Number:  32 bits — byte number of first byte in segment
ACK Number:       32 bits — next byte sender expects to receive
Data Offset:      4 bits — header length in 32-bit words (min=5=20 bytes)
Control Flags:    6 bits (one-hot):
  URG: urgent pointer field valid
  ACK: acknowledgment number valid
  PSH: push data to application immediately
  RST: reset connection (error)
  SYN: synchronize sequence numbers (connection setup)
  FIN: no more data (connection teardown)
Window Size:      16 bits — receive window size (flow control)
Checksum:         16 bits — error detection
Urgent Pointer:   16 bits — offset of urgent data
```

### 🔧 TCP Sequence and Acknowledgment Numbers
```
SEQUENCE NUMBER: Identifies byte position in data stream.
  ISN (Initial Sequence Number): random, chosen at connection start.
  If ISN=1000 and segment carries 500 bytes: seq=1000, covers bytes 1000-1499.

ACK NUMBER: Next byte expected.
  If received bytes 0-999: ACK = 1000 (expecting byte 1000 next).
  CUMULATIVE ACK: acknowledges all bytes up to ACK-1.

Example:
  Client sends: seq=100, 10 bytes of data
  Server receives: gets bytes 100-109
  Server sends: ACK=110 (expecting byte 110 next)
```

---

## 3. TCP 3-Way Handshake

### 🔧 Connection Establishment
```
CLIENT                              SERVER
  │                                   │
  │──── SYN (seq=x) ──────────────►   │  Step 1: Client sends SYN
  │                                   │         (x = random ISN)
  │                                   │
  │   ◄── SYN-ACK (seq=y, ack=x+1) ──│  Step 2: Server responds SYN-ACK
  │                                   │         (y = server's ISN)
  │                                   │         (ack = x+1, expects x+1 next)
  │                                   │
  │──── ACK (ack=y+1) ─────────────►  │  Step 3: Client sends ACK
  │                                   │         (ack = y+1)
  │                                   │
  │═══════════ Data Transfer ══════════│

WHY 3-WAY (not 2-way)?
  2-way: Server sends SYN-ACK, but doesn't know if client received it.
  3-way: Client's final ACK confirms server's SYN received.
  Both sides establish ISNs and confirm the other side's ISN.

STATES:
  Client: CLOSED → SYN_SENT → ESTABLISHED
  Server: CLOSED → LISTEN → SYN_RECEIVED → ESTABLISHED
```

### 🔧 Connection Termination (4-Way)
```
CLIENT                              SERVER
  │                                   │
  │──── FIN (seq=u) ──────────────►   │  Client done sending
  │                                   │
  │   ◄── ACK (ack=u+1) ─────────────│  Server ACKs
  │                                   │  (half-close: server can still send)
  │                                   │
  │   ◄── FIN (seq=v) ───────────────│  Server done sending
  │                                   │
  │──── ACK (ack=v+1) ─────────────►  │  Client ACKs
  │                                   │
  Client enters TIME_WAIT (2×MSL seconds before CLOSED)

TIME_WAIT purpose:
  1. Ensure server received final ACK (if lost, server retransmits FIN)
  2. Let old duplicate segments die (MSL = Maximum Segment Lifetime)
  MSL typically 60-120 seconds → TIME_WAIT = 2-4 minutes

WHY 4-WAY not 3-way for termination?
  TCP is full-duplex. Each side closes independently.
  Server may still have data to send after client sends FIN.
  FIN and ACK can't always be combined (unlike SYN+ACK in setup).
```

---

## 4. TCP Flow Control — Sliding Window

### 🔧 Receive Window (rwnd)
```
Receiver advertises how much buffer space it has.
Sender cannot have more unacknowledged data than rwnd.

If receiver buffer = 10000 bytes and 3000 bytes already in buffer:
  rwnd = 10000 - 3000 = 7000 bytes (advertised in ACK segment)

Sender maintains: LastByteSent - LastByteAcked ≤ rwnd

ZERO WINDOW (Window = 0):
  Receiver's buffer is FULL.
  Sender stops sending.
  Receiver sends Window Update when buffer has space.
  DEADLOCK PROBLEM: if Window Update gets lost → sender waits forever.
  SOLUTION: Persist Timer — sender periodically sends 1-byte probe to check window.
```

### 🔧 TCP Window in Practice
```
Window scaling (RFC 1323):
  16-bit window field → max 65,535 bytes.
  Too small for high-speed/high-latency links.
  Window Scale Option: shift multiplier (up to ×2^14 = 16 GB window).

Silly Window Syndrome:
  Receiver: opens tiny window (e.g., 1 byte) → sender sends tiny segment.
  NAGLE'S ALGORITHM: wait to accumulate data or wait for ACK before sending small segments.
  Clark's Solution: receiver doesn't advertise window until it has ≥ MSS or ≥ half buffer.
```

---

## 5. TCP Congestion Control

### 🧠 Key Variables
```
cwnd (congestion window): sender's limit based on NETWORK congestion
rwnd (receive window):    sender's limit based on RECEIVER capacity
Actual send window = min(cwnd, rwnd)

ssthresh (slow start threshold): dividing line between slow start and congestion avoidance
```

### 🔧 Phase 1: Slow Start
```
Initial cwnd = 1 MSS (Maximum Segment Size)

Each ACK received: cwnd += 1 MSS
→ Doubles each round trip time (RTT)! (Exponential growth)

Example:
  RTT 1: cwnd=1 → send 1 segment → receive 1 ACK → cwnd=2
  RTT 2: cwnd=2 → send 2 segments → 2 ACKs → cwnd=4
  RTT 3: cwnd=4 → send 4 → 4 ACKs → cwnd=8
  ...continues until cwnd ≥ ssthresh or loss occurs

Name "slow start" is MISLEADING — it's actually EXPONENTIAL!
Called "slow" because starts at 1 MSS (not at full window).
```

### 🔧 Phase 2: Congestion Avoidance
```
When cwnd ≥ ssthresh:
Each RTT: cwnd += 1 MSS (LINEAR growth)
More specifically: each ACK → cwnd += MSS²/cwnd

Example (ssthresh = 8 MSS):
  cwnd=8: send 8 → 8 ACKs → cwnd=9 (added 1 total per RTT)
  cwnd=9: → cwnd=10
  ...linear increase until congestion event
```

### 🔧 Congestion Events

#### Loss Detected by Triple Duplicate ACK (3 dup ACKs)
```
Network is still working (some packets getting through — ACKs arriving).
Just one packet dropped.
Mild congestion → gentle response.

Tahoe:    ssthresh = cwnd/2, cwnd = 1, restart slow start
Reno:     ssthresh = cwnd/2, cwnd = ssthresh, enter congestion avoidance
          (Fast Recovery: cut in half but don't restart from 1)
CUBIC/modern: more sophisticated
```

#### Loss Detected by Timeout
```
No ACKs arriving → severe congestion (many packets lost).
Hard reset:
  ssthresh = cwnd/2
  cwnd = 1 MSS
  Restart slow start
```

### 📝 TCP Congestion Control Trace

**Initial ssthresh=8 MSS. Trace cwnd over time with events.**

```
RTT | cwnd | Phase       | Event
────┼──────┼─────────────┼────────────────────────────────
 1  |  1   | Slow Start  | Start
 2  |  2   | Slow Start  | 
 3  |  4   | Slow Start  |
 4  |  8   | Con. Avoid  | cwnd = ssthresh, switch phase
 5  |  9   | Con. Avoid  |
 6  | 10   | Con. Avoid  |
 7  | 11   | Con. Avoid  |
 8  | 12   | Con. Avoid  | → 3 dup ACKs at cwnd=12
    |      |             | ssthresh = 12/2 = 6
    |      |             | cwnd = 6 (Reno: fast recovery)
 9  |  6   | Con. Avoid  | (was already ≥ ssthresh=6)
10  |  7   | Con. Avoid  |
11  |  8   | Con. Avoid  | → Timeout at cwnd=8!
    |      |             | ssthresh = 8/2 = 4
    |      |             | cwnd = 1
12  |  1   | Slow Start  |
13  |  2   | Slow Start  |
14  |  4   | Slow Start  |
15  |  4   | Con. Avoid  | cwnd = ssthresh, switch
16  |  5   | Con. Avoid  |
...
```

---

## 6. UDP — User Datagram Protocol

### 🔧 UDP Header (8 bytes only!)
```
┌─────────────────────┬─────────────────────┐
│    Source Port      │  Destination Port   │
│      (16 bits)      │     (16 bits)       │
├─────────────────────┼─────────────────────┤
│       Length        │      Checksum       │
│      (16 bits)      │     (16 bits)       │
└─────────────────────┴─────────────────────┘

Length: total UDP segment length (header + data), minimum 8 bytes.
Checksum: optional in IPv4, mandatory in IPv6.
NO: sequence numbers, ACK numbers, window size, flags.
```

### 🔧 UDP Characteristics
```
CONNECTIONLESS: No handshake. Just send.
UNRELIABLE:     No ACKs, no retransmission, no ordering.
FAST:           No connection overhead, no congestion control.
STATELESS:      No connection state maintained.
LIGHTWEIGHT:    8-byte header vs TCP's 20-byte header.

UDP USE CASES:
  DNS:           Small queries, retransmission handled by application.
  DHCP:          Broadcast-based, connectionless by nature.
  TFTP:          Simple file transfer.
  VoIP/Video:    Real-time, old packets useless (don't retransmit).
  SNMP:          Management queries, occasional loss acceptable.
  Gaming:        Low latency critical, application handles reliability.
  Multicast:     TCP can't multicast.
  QUIC:          Modern protocol over UDP (HTTP/3 uses QUIC over UDP).

WHY UDP FOR REAL-TIME?
  Late packet = useless (worse than no packet for voice/video).
  Better to skip a frame than stall and wait for retransmit.
  Application controls what to do with loss.
```

---

## 7. TCP vs UDP Comparison

| Feature | TCP | UDP |
|---|---|---|
| **Connection** | Connection-oriented | Connectionless |
| **Reliability** | Guaranteed delivery | Best effort |
| **Ordering** | In-order delivery | No ordering guarantee |
| **Flow Control** | Yes (receiver window) | No |
| **Congestion Control** | Yes (slow start, CA) | No |
| **Error Detection** | Checksum (mandatory) | Checksum (optional IPv4) |
| **Header Size** | 20-60 bytes | 8 bytes |
| **Speed** | Slower (overhead) | Faster |
| **Handshake** | 3-way SYN | None |
| **Socket Type** | Stream (SOCK_STREAM) | Datagram (SOCK_DGRAM) |
| **Use Cases** | HTTP, FTP, SMTP, SSH | DNS, VoIP, Video, Gaming |

---

## 8. Application Layer Protocols

### 🔧 Port Numbers Master Table

| Protocol | Port | Transport | Purpose |
|---|---|---|---|
| **FTP** | 20 (data), 21 (control) | TCP | File transfer |
| **SSH** | 22 | TCP | Secure remote login |
| **Telnet** | 23 | TCP | Remote login (insecure) |
| **SMTP** | 25 | TCP | Send email |
| **DNS** | 53 | UDP (TCP for large) | Name resolution |
| **DHCP** | 67 (server), 68 (client) | UDP | IP assignment |
| **TFTP** | 69 | UDP | Trivial file transfer |
| **HTTP** | 80 | TCP | Web browsing |
| **POP3** | 110 | TCP | Receive email (download) |
| **IMAP** | 143 | TCP | Receive email (sync) |
| **HTTPS** | 443 | TCP | Secure web |
| **SMB** | 445 | TCP | Windows file sharing |
| **SMTPS** | 465/587 | TCP | Secure email sending |
| **IMAPS** | 993 | TCP | Secure IMAP |
| **POP3S** | 995 | TCP | Secure POP3 |
| **RDP** | 3389 | TCP | Remote Desktop |
| **MySQL** | 3306 | TCP | Database |
| **NTP** | 123 | UDP | Time sync |
| **SNMP** | 161 | UDP | Network management |

---

## 9. DNS — Domain Name System

### 🔧 What DNS Does
```
Translates human-readable names → IP addresses.
www.google.com → 142.250.x.x

Also resolves:
  IP → hostname    (Reverse DNS, PTR records)
  mail server      (MX records)
  aliases          (CNAME records)
  IPv6 addresses   (AAAA records)
  text info        (TXT records, used for SPF, DKIM)
```

### 🔧 DNS Hierarchy
```
                         . (Root)
                        / | \
                      com org net ...
                      /
                   google
                   /    \
                 www   mail

DOMAIN NAME: Read right to left (root → TLD → domain → subdomain)
  www.google.com.  (trailing dot = root, usually omitted)
```

### 🔧 DNS Resolution Process
```
User types: www.example.com

Step 1: CHECK LOCAL CACHE (browser + OS cache)
        If found and not expired → done!

Step 2: CHECK LOCAL DNS SERVER (usually ISP's or company's)
        If cached there → return answer.

Step 3: LOCAL DNS QUERIES ROOT SERVER
        Root knows who handles .com

Step 4: LOCAL DNS QUERIES .COM TLD SERVER
        .com TLD server knows who handles example.com

Step 5: LOCAL DNS QUERIES AUTHORITATIVE SERVER for example.com
        Returns IP of www.example.com

Step 6: Local DNS caches result (TTL determines how long).
        Returns IP to client.

RECURSIVE QUERY: Client asks local DNS to do all the work.
ITERATIVE QUERY: Each DNS server tells local DNS where to go next.
                 (Local DNS does all the querying itself)

Local DNS resolver uses recursive (to client) + iterative (to root/TLD).
```

### 🔧 DNS Record Types
```
A:     hostname → IPv4 address
AAAA:  hostname → IPv6 address
CNAME: canonical name (alias → real hostname)
       www.example.com → example.com
MX:    mail exchange — which server handles email for domain
NS:    name server — which DNS server is authoritative for domain
PTR:   reverse lookup — IP → hostname
SOA:   Start of Authority — administrative info about zone
TXT:   text info — SPF, DKIM, verification records
```

### 🔧 DNS Caching & TTL
```
TTL (Time to Live): how long to cache the answer (in seconds).
Common TTLs: 300 (5 min), 3600 (1 hour), 86400 (1 day).
Low TTL: changes propagate fast, more DNS load.
High TTL: less DNS load, slow propagation of changes.

Negative TTL: cache "not found" responses (SOA record's negative TTL).
```

---

## 10. DHCP

### 🔧 DORA Process
```
DHCP is the process of automatically assigning IP addresses.
Process: DISCOVER → OFFER → REQUEST → ACK

  CLIENT                          DHCP SERVER
    │                                 │
    │──── DISCOVER (broadcast) ─────► │  "Anyone have an IP for me?"
    │       src=0.0.0.0:68           │  src=0.0.0.0 (no IP yet!)
    │       dst=255.255.255.255:67   │  dst=broadcast
    │                                 │
    │   ◄── OFFER (broadcast) ───────│  "Here's 192.168.1.100 for you"
    │                                 │
    │──── REQUEST (broadcast) ──────► │  "I accept 192.168.1.100"
    │                                 │  (still broadcast — others may have offered)
    │                                 │
    │   ◄── ACK (broadcast/unicast) ─│  "Confirmed. Use 192.168.1.100"
    │                                 │
  Configure IP! 🎉

LEASE: IP assigned for a limited time (lease time).
RENEW: At 50% of lease → try to renew with same server.
REBIND: At 87.5% of lease → try any DHCP server.
EXPIRE: At 100% → release IP, start DORA again.

DHCP provides:
  - IP address + subnet mask
  - Default gateway
  - DNS server addresses
  - Lease time
  - (optionally: NTP server, domain name, etc.)
```

### 🔧 DHCP Relay Agent
```
DHCP uses broadcasts → can't cross routers (routers don't forward broadcasts).
DHCP Relay Agent: router or host that:
  - Intercepts DHCP broadcasts
  - Forwards them as UNICAST to DHCP server
  - Forwards server's response back
Allows ONE DHCP server to serve multiple subnets.
```

---

## 11. HTTP & HTTPS

### 🔧 HTTP — HyperText Transfer Protocol
```
Application layer protocol for web.
Request-Response model.
Stateless (each request independent).
Port 80 (HTTP), 443 (HTTPS).

HTTP VERSIONS:
  HTTP/1.0: One TCP connection per request (inefficient).
  HTTP/1.1: Persistent connections (keep-alive), pipelining.
  HTTP/2:   Binary, multiplexed (multiple requests on one connection).
  HTTP/3:   Uses QUIC (UDP-based, built-in TLS, faster).
```

### 🔧 HTTP Request Structure
```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 ...
Accept: text/html
Connection: keep-alive
[blank line]
[optional body]

METHOD: GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS
  GET:    Retrieve resource (idempotent, cacheable)
  POST:   Submit data / create resource (not idempotent)
  PUT:    Update/replace resource (idempotent)
  DELETE: Remove resource
  HEAD:   Like GET but response has no body (get headers only)
```

### 🔧 HTTP Response Structure
```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234
Date: Sun, 01 Mar 2026 12:00:00 GMT
[blank line]
[response body: HTML content]

STATUS CODES:
  1xx: Informational (100 Continue)
  2xx: Success
    200 OK
    201 Created
    204 No Content
  3xx: Redirection
    301 Moved Permanently
    302 Found (temporary redirect)
    304 Not Modified (cached)
  4xx: Client Error
    400 Bad Request
    401 Unauthorized (not authenticated)
    403 Forbidden (authenticated but no permission)
    404 Not Found
    405 Method Not Allowed
  5xx: Server Error
    500 Internal Server Error
    503 Service Unavailable
```

### 🔧 HTTPS
```
HTTP + TLS (Transport Layer Security) = HTTPS.
TLS provides: encryption, authentication, integrity.
Certificate: server proves identity using SSL/TLS certificate (CA-signed).
Port 443.

TLS Handshake (simplified):
  1. Client Hello: supported cipher suites, random number.
  2. Server Hello: chosen cipher, certificate, random number.
  3. Client verifies certificate (trusted CA?).
  4. Key exchange (asymmetric crypto).
  5. Session keys derived (both sides).
  6. Data transmitted encrypted with session keys.
```

### 🔧 Cookies & Sessions
```
HTTP is STATELESS — server doesn't remember previous requests.
COOKIES: Small data stored by browser.
  Server sends: Set-Cookie: sessionID=abc123
  Browser sends cookie with every request to same domain.
  Used for: login sessions, shopping carts, preferences.

SESSION: Server-side storage linked to cookie ID.
```

---

## 12. FTP, SMTP, POP3, IMAP

### 🔧 FTP — File Transfer Protocol
```
Ports: 21 (control channel) + 20 (data channel)
Uses TWO separate TCP connections.

Control connection (port 21): persistent, send commands (LIST, GET, PUT, QUIT...)
Data connection (port 20 or ephemeral): opened for each file transfer.

TWO MODES:
  Active mode:  Server initiates data connection from port 20 to client.
                Problem: client behind NAT/firewall may block incoming.
  Passive mode: Client initiates data connection to server (client-friendly).
                Server opens random port, tells client.

SFTP (SSH File Transfer Protocol): secure, uses SSH (port 22).
FTPS: FTP over TLS (port 990 implicit, or explicit on 21).
```

### 🔧 Email Protocols
```
SENDING EMAIL: SMTP (Simple Mail Transfer Protocol) — port 25, 587
  MUA (Mail User Agent — email client) → SMTP → MTA → SMTP → MTA → ...

RECEIVING EMAIL:
  POP3 (Post Office Protocol v3) — port 110:
    Downloads email to local device.
    Typically DELETES from server after download.
    Good for offline access, bad for multi-device.

  IMAP (Internet Message Access Protocol) — port 143:
    Keeps email ON SERVER, syncs across devices.
    Downloads headers first (body on demand).
    Good for multi-device, web-based email.
    Bad for offline-only use.

EMAIL FLOW:
  Sender → [SMTP] → Sender's Mail Server → [SMTP] → Recipient's Mail Server
  Recipient → [POP3/IMAP] → Recipient's Mail Server
```

---

## 13. Numericals — Full Solved Sets

### 📝 Numerical 1 — TCP Sequence Numbers

**Client sends 3 segments: each 500 bytes. Initial seq = 1000.**
**What are the sequence numbers and expected ACKs?**

```
Segment 1: seq=1000, data=bytes 1000-1499 (500 bytes)
           Server ACK = 1500

Segment 2: seq=1500, data=bytes 1500-1999
           Server ACK = 2000

Segment 3: seq=2000, data=bytes 2000-2499
           Server ACK = 2500
```

---

### 📝 Numerical 2 — TCP Congestion Control Trace

**Initial ssthresh=32 MSS. Trace TCP Reno for 12 RTTs. Assume:**
- **Timeout at RTT 8**
- **3 dup ACKs at RTT 16**

```
RTT | cwnd | ssthresh | Phase    | Event
────┼──────┼──────────┼──────────┼──────────────────────
  1 |   1  |    32    | Slow Start|
  2 |   2  |    32    | Slow Start|
  3 |   4  |    32    | Slow Start|
  4 |   8  |    32    | Slow Start|
  5 |  16  |    32    | Slow Start|
  6 |  32  |    32    | Con. Avoid| (cwnd=ssthresh, switch)
  7 |  33  |    32    | Con. Avoid|
  8 |  34  |    32    | Con. Avoid| TIMEOUT at cwnd=34
    |      |          |          | ssthresh = 34/2 = 17
    |      |          |          | cwnd = 1
  9 |   1  |    17    | Slow Start|
 10 |   2  |    17    | Slow Start|
 11 |   4  |    17    | Slow Start|
 12 |   8  |    17    | Slow Start|
 13 |  16  |    17    | Slow Start|
 14 |  17  |    17    | Con. Avoid| (cwnd=ssthresh)
 15 |  18  |    17    | Con. Avoid|
 16 |  19  |    17    | Con. Avoid| 3 DUP ACKs at cwnd=19
    |      |          |          | ssthresh = 19/2 = 9 (round down)
    |      |          |          | cwnd = 9 (Reno: fast recovery → ssthresh)
 17 |   9  |     9    | Con. Avoid|
 18 |  10  |     9    | Con. Avoid|
...
```

---

### 📝 Numerical 3 — Effective Throughput

**TCP window size = 65,535 bytes. RTT = 200 ms. What is maximum throughput?**

```
Throughput = Window Size / RTT
           = 65,535 bytes / 0.2 s
           = 327,675 bytes/s
           ≈ 327 KB/s ≈ 2.6 Mbps

This is why window scaling is needed for high-speed links!
A 100 Mbps link with RTT=200ms needs:
  Window = 100×10⁶ × 0.2 = 20,000,000 bytes = 20 MB
  Far exceeds 65,535 byte limit → must use window scaling.
```

---

### 📝 Numerical 4 — DNS TTL

**TTL=300 seconds. Client resolves google.com at time T=0.**
**At what times will the client need to re-resolve?**

```
First resolution: T=0, cached until T=300.
Second resolution: T=300 (cache expired).
Third resolution: T=600.
...every 300 seconds (5 minutes).
```

---

### 📝 Numerical 5 — UDP Header Size

**Application sends 100 bytes of data via UDP over IP over Ethernet.**
**What is the total frame size?**

```
Data:          100 bytes
UDP header:    8 bytes
IP header:     20 bytes (minimum, no options)
Ethernet header: 14 bytes (dest MAC 6 + src MAC 6 + type 2)
Ethernet FCS:  4 bytes

Total: 100 + 8 + 20 + 14 + 4 = 146 bytes

Overhead: (8+20+14+4)/146 = 46/146 = 31.5% overhead
(For TCP: replace 8-byte UDP with 20-byte TCP = 152 bytes total)
```

---

## 14. MCQ Traps & Exam Q&A

### ⚠️ THE TRAPS

---

**TRAP 1: TCP SYN-ACK consumes a sequence number**
```
SYN occupies sequence number x.
Server must ACK with ack=x+1 (even though SYN carries no data).
FIN also consumes a sequence number.
Data bytes and SYN/FIN each consume one sequence number.
```

---

**TRAP 2: TCP "Slow Start" is actually exponential**
```
cwnd doubles each RTT in slow start (because each ACK adds 1 MSS,
and there are cwnd ACKs per RTT → cwnd increases by cwnd → doubles).
Congestion Avoidance is the LINEAR phase (additive increase).
```

---

**TRAP 3: 3 duplicate ACKs ≠ timeout in TCP Reno**
```
3 dup ACKs → mild congestion → fast retransmit + fast recovery.
  ssthresh = cwnd/2, cwnd = ssthresh (Reno)
Timeout → severe congestion → hard reset.
  ssthresh = cwnd/2, cwnd = 1
Timeout is a more drastic response than 3 dup ACKs.
```

---

**TRAP 4: HTTP is stateless — cookies add state**
```
HTTP itself: stateless (server doesn't remember you between requests).
Sessions/Cookies: application-level state management (not HTTP protocol).
```

---

**TRAP 5: DNS uses UDP for regular queries, TCP for zone transfers**
```
UDP port 53: regular queries (fast, small responses).
TCP port 53: zone transfers (large data) and responses > 512 bytes.
```

---

**TRAP 6: POP3 deletes from server, IMAP keeps on server**
```
POP3: downloads and (by default) deletes. Single device friendly.
IMAP: keeps on server, syncs. Multi-device friendly.
```

---

**TRAP 7: DHCP client port 68, server port 67**
```
DHCP Discover sent FROM port 68 TO port 67.
DHCP Offer sent FROM port 67 TO port 68.
Ports 67 and 68 are well-known DHCP ports.
```

---

**TRAP 8: HTTP 401 vs 403**
```
401 Unauthorized: Not authenticated (no credentials, or credentials failed).
403 Forbidden:    Authenticated but doesn't have permission.
Think: 401 = "Who are you?", 403 = "I know who you are, but no."
```

---

### ⚡ Quick Fire Q&A

| Question | Answer |
|---|---|
| TCP header minimum size? | 20 bytes |
| UDP header size? | 8 bytes (fixed) |
| TCP 3-way handshake messages? | SYN, SYN-ACK, ACK |
| TCP connection teardown messages? | FIN, ACK, FIN, ACK (4-way) |
| TIME_WAIT duration? | 2 × MSL (typically 2-4 minutes) |
| TCP initial cwnd? | 1 MSS |
| Slow start growth rate? | Doubles per RTT (exponential) |
| Congestion avoidance growth? | +1 MSS per RTT (linear) |
| 3 dup ACKs → cwnd (Reno)? | cwnd = ssthresh = old_cwnd/2 |
| Timeout → cwnd (Reno)? | cwnd = 1, ssthresh = old_cwnd/2 |
| Effective window = ? | min(cwnd, rwnd) |
| TCP throughput formula? | Window Size / RTT |
| DNS hierarchy order? | Root → TLD → Domain → Subdomain |
| DNS A record does what? | hostname → IPv4 address |
| DNS MX record does what? | domain → mail server |
| DHCP DORA stands for? | Discover, Offer, Request, Acknowledge |
| DHCP uses which ports? | Client: 68, Server: 67 |
| FTP uses how many connections? | 2 (control port 21 + data port 20) |
| SMTP is for? | Sending email |
| POP3 vs IMAP key diff? | POP3 downloads+deletes, IMAP keeps on server |
| HTTP GET vs POST? | GET retrieves (idempotent), POST submits data |
| HTTP 200 means? | OK (success) |
| HTTP 404 means? | Not Found |
| HTTP 301 means? | Moved Permanently (redirect) |
| HTTPS port? | 443 |
| HTTP/3 runs over? | QUIC (UDP-based) |
| Nagle's algorithm prevents? | Silly Window Syndrome (tiny TCP segments) |

---

### 📋 Quick Reference

```
TCP vs UDP:
  TCP: reliable, ordered, flow+congestion control, 20B header, connection-oriented
  UDP: unreliable, unordered, no control, 8B header, connectionless

3-WAY HANDSHAKE: SYN(x) → SYN-ACK(y, ack=x+1) → ACK(ack=y+1)
4-WAY TEARDOWN:  FIN(u) → ACK(u+1) → FIN(v) → ACK(v+1)

TCP CONGESTION CONTROL:
  Slow Start:    cwnd=1, double/RTT, until cwnd=ssthresh
  Con. Avoid:    +1 MSS/RTT (linear), until loss
  3 dup ACKs:    ssthresh=cwnd/2, cwnd=ssthresh (Reno fast recovery)
  Timeout:       ssthresh=cwnd/2, cwnd=1, restart slow start

DNS RECORDS: A(IPv4), AAAA(IPv6), CNAME(alias), MX(mail), NS(nameserver), PTR(reverse)
DNS: UDP 53 (queries), TCP 53 (zone transfers, large responses)

EMAIL:
  SMTP 25/587: send
  POP3 110: receive (download+delete)
  IMAP 143: receive (server-side sync)

HTTP STATUS:
  2xx=success, 3xx=redirect, 4xx=client error, 5xx=server error
  200=OK, 301=perm redirect, 401=unauth, 403=forbidden, 404=not found

DHCP DORA: all on UDP, client port 68, server port 67
HTTPS = HTTP + TLS, port 443
```

---
