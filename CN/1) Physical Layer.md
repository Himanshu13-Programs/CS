# 📘 CN Chapter 1 — OSI Model, TCP/IP Model & Physical Layer 
---

## 📌 Table of Contents
1. [OSI Model — All 7 Layers](#1-osi-model--all-7-layers)
2. [TCP/IP Model](#2-tcpip-model)
3. [OSI vs TCP/IP Comparison](#3-osi-vs-tcpip-comparison)
4. [Physical Layer — Signals & Transmission](#4-physical-layer--signals--transmission)
5. [Bandwidth, Throughput & Latency](#5-bandwidth-throughput--latency)
6. [Nyquist & Shannon Theorems](#6-nyquist--shannon-theorems)
7. [Transmission Media](#7-transmission-media)
8. [Multiplexing](#8-multiplexing)
9. [Numericals — Full Solved Sets](#9-numericals--full-solved-sets)
10. [MCQ Traps & Exam Q&A](#10-mcq-traps--exam-qa)

---

## 1. OSI Model — All 7 Layers

### 🧠 Mnemonic
```
Layer 7 → Layer 1 (top to bottom):
"All People Seem To Need Data Processing"
  A = Application  (7)
  P = Presentation (6)
  S = Session      (5)
  T = Transport    (4)
  N = Network      (3)
  D = Data Link    (2)
  P = Physical     (1)

Bottom to top: "Please Do Not Throw Sausage Pizza Away"
```

---

### 🔧 Layer 7 — Application Layer
```
PURPOSE: Interface between user application and network.
         Provides network services DIRECTLY to end-user applications.

FUNCTIONS:
  - File transfer, email, web browsing
  - User authentication and privacy
  - Identifies communication partners
  - Determines resource availability

PROTOCOLS:
  HTTP  (80)   → web browsing
  HTTPS (443)  → secure web
  FTP   (21)   → file transfer (control), (20) data
  SFTP  (22)   → secure file transfer
  SMTP  (25)   → sending email
  POP3  (110)  → receiving email (downloads, deletes from server)
  IMAP  (143)  → receiving email (stays on server, sync)
  DNS   (53)   → domain name resolution (UDP usually, TCP for large)
  DHCP  (67/68)→ dynamic IP assignment
  Telnet(23)   → remote login (insecure)
  SSH   (22)   → secure remote login
  SNMP  (161)  → network management
  NTP   (123)  → time synchronization

DATA UNIT: Message / Data
```

---

### 🔧 Layer 6 — Presentation Layer
```
PURPOSE: Data translation, encryption, compression.
         "Translator" of the network.

FUNCTIONS:
  - Translation: converts data formats (EBCDIC ↔ ASCII)
  - Encryption/Decryption: SSL/TLS lives here conceptually
  - Compression: reduces data size before transmission
  - Serialization: converts objects to byte streams (marshalling)

EXAMPLES: JPEG, MPEG, GIF, PNG, ASCII, EBCDIC, SSL, TLS

DATA UNIT: Data
```

---

### 🔧 Layer 5 — Session Layer
```
PURPOSE: Establishes, manages, and terminates sessions between apps.
         "Traffic cop" between applications.

FUNCTIONS:
  - Session establishment, maintenance, termination
  - Synchronization (checkpoints for long transfers)
  - Dialog control: half-duplex vs full-duplex
  - Session recovery (resume from checkpoint after failure)

PROTOCOLS: NetBIOS, RPC (Remote Procedure Call), PPTP, SIP

DATA UNIT: Data
```

---

### 🔧 Layer 4 — Transport Layer
```
PURPOSE: End-to-end communication, reliability, flow control.
         "Quality control" of data delivery.

FUNCTIONS:
  - Segmentation and reassembly
  - Connection management (TCP: connection-oriented)
  - Error detection and recovery
  - Flow control (don't overwhelm receiver)
  - Congestion control
  - Multiplexing via port numbers

PROTOCOLS:
  TCP (Transmission Control Protocol):
    - Connection-oriented (3-way handshake)
    - Reliable (acknowledgments, retransmission)
    - Flow control (sliding window)
    - Congestion control
    - Ordered delivery
    - Use: HTTP, FTP, SMTP, SSH

  UDP (User Datagram Protocol):
    - Connectionless (no handshake)
    - Unreliable (no ACK, no retransmit)
    - No flow control
    - Faster, lower overhead
    - Use: DNS, DHCP, VoIP, video streaming, gaming

PORT NUMBERS:
  0-1023:     Well-known ports (HTTP=80, HTTPS=443, SSH=22...)
  1024-49151: Registered ports (applications)
  49152-65535: Dynamic/ephemeral ports (client side)

DATA UNIT: Segment (TCP) / Datagram (UDP)
DEVICES: Gateways, Firewalls (layer 4)
```

---

### 🔧 Layer 3 — Network Layer
```
PURPOSE: Logical addressing and routing. End-to-end delivery across networks.

FUNCTIONS:
  - Logical addressing (IP addresses)
  - Routing: finding best path across multiple networks
  - Packet forwarding
  - Fragmentation and reassembly
  - Congestion control (at network level)

PROTOCOLS:
  IP (IPv4, IPv6) — main protocol
  ICMP — error reporting, ping, traceroute
  OSPF, RIP, BGP — routing protocols
  ARP — Address Resolution Protocol (technically between L2/L3)
  NAT — Network Address Translation
  IGMP — multicast group management

DATA UNIT: Packet
DEVICES: Routers, Layer 3 switches
```

---

### 🔧 Layer 2 — Data Link Layer
```
PURPOSE: Node-to-node delivery on same network segment.
         Reliable transfer between adjacent nodes.

SUB-LAYERS:
  LLC (Logical Link Control): error detection, flow control
  MAC (Media Access Control): addressing, media access

FUNCTIONS:
  - Physical addressing (MAC addresses)
  - Frame synchronization (knowing where frames start/end)
  - Error detection (CRC, checksums)
  - Flow control (between adjacent nodes)
  - Access control (who can transmit — CSMA/CD, CSMA/CA)

PROTOCOLS:
  Ethernet (IEEE 802.3)
  Wi-Fi (IEEE 802.11)
  PPP (Point-to-Point Protocol)
  HDLC
  ARP (often placed here)

DATA UNIT: Frame
DEVICES: Switches, Bridges, NICs (Network Interface Cards)
```

---

### 🔧 Layer 1 — Physical Layer
```
PURPOSE: Transmits raw bits over physical medium.

FUNCTIONS:
  - Bit representation (encoding 0s and 1s as signals)
  - Transmission rate (bits per second)
  - Physical characteristics of medium (voltage, frequency, cables)
  - Synchronization of bits
  - Line configuration (point-to-point, multipoint)
  - Physical topology (bus, star, ring, mesh)
  - Transmission mode (simplex, half-duplex, full-duplex)

PROTOCOLS/STANDARDS: RS-232, RJ45, DSL, ISDN, 100BASE-T, Bluetooth (PHY)

DATA UNIT: Bit
DEVICES: Hubs, Repeaters, Cables, Connectors, Modems
```

---

### 📊 OSI Layers — Master Reference Table

| Layer | Name | Data Unit | Key Protocols | Devices |
|---|---|---|---|---|
| 7 | Application | Message | HTTP, FTP, SMTP, DNS, DHCP | — |
| 6 | Presentation | Data | SSL/TLS, JPEG, ASCII | — |
| 5 | Session | Data | NetBIOS, RPC, SIP | — |
| 4 | Transport | Segment/Datagram | TCP, UDP | Gateway, Firewall |
| 3 | Network | Packet | IP, ICMP, OSPF, RIP, BGP | Router, L3 Switch |
| 2 | Data Link | Frame | Ethernet, ARP, PPP | Switch, Bridge |
| 1 | Physical | Bit | RS-232, DSL | Hub, Repeater |

---

## 2. TCP/IP Model

### 🔧 Four Layers
```
TCP/IP has 4 layers (combines some OSI layers):

Layer 4: APPLICATION
  Combines OSI layers 5 + 6 + 7
  Protocols: HTTP, FTP, SMTP, DNS, DHCP, SSH, Telnet

Layer 3: TRANSPORT
  Same as OSI Layer 4
  Protocols: TCP, UDP

Layer 2: INTERNET
  Same as OSI Layer 3
  Protocols: IP, ICMP, ARP, RARP, OSPF, RIP, BGP

Layer 1: NETWORK ACCESS (Link Layer)
  Combines OSI layers 1 + 2
  Protocols: Ethernet, Wi-Fi, PPP, ARP
```

---

## 3. OSI vs TCP/IP Comparison

| Feature | OSI | TCP/IP |
|---|---|---|
| **Layers** | 7 | 4 |
| **Development** | ISO standard (theoretical) | DARPA (practical) |
| **Usage** | Reference model | Actual implementation |
| **Protocol** | Protocol-independent | Built around TCP/IP |
| **Transport** | TCP, UDP, SPX | TCP, UDP |
| **Reliability** | At transport layer | At transport layer |
| **Session/Presentation** | Separate layers | Part of Application |

### 🔧 Layer Mapping
```
OSI Layer 7 (Application)  ─┐
OSI Layer 6 (Presentation) ─┤ → TCP/IP Layer 4 (Application)
OSI Layer 5 (Session)      ─┘

OSI Layer 4 (Transport)    ─── TCP/IP Layer 3 (Transport)

OSI Layer 3 (Network)      ─── TCP/IP Layer 2 (Internet)

OSI Layer 2 (Data Link)    ─┐
OSI Layer 1 (Physical)     ─┘ → TCP/IP Layer 1 (Network Access)
```

---

## 4. Physical Layer — Signals & Transmission

### 🔧 Analog vs Digital
```
ANALOG SIGNAL: Continuous, infinite values, varies smoothly.
  Examples: voice on old telephone, FM radio waves
  Properties: Amplitude, Frequency, Phase

DIGITAL SIGNAL: Discrete, finite values (0 or 1).
  Examples: computer data, modern telephone
  Properties: Bit rate, Bit intervals

ANALOG DATA → ANALOG SIGNAL: AM/FM radio
DIGITAL DATA → DIGITAL SIGNAL: NRZ encoding (computers)
ANALOG DATA → DIGITAL SIGNAL: ADC (Analog-to-Digital Converter) — PCM in telephony
DIGITAL DATA → ANALOG SIGNAL: Modem (MOdulator-DEModulator)
```

### 🔧 Digital Encoding Schemes
```
NRZ-L (Non-Return to Zero - Level):
  0 = low voltage, 1 = high voltage
  Problem: long runs of same bit → sync loss

NRZ-I (Non-Return to Zero - Invert):
  0 = no change, 1 = invert signal
  Better for 1s, still bad for long 0s

Manchester Encoding:
  0 = high→low transition at middle of bit
  1 = low→high transition at middle of bit
  Self-clocking! Used in Ethernet (10Mbps)
  Bandwidth = 2× data rate (overhead)

Differential Manchester:
  Transition at START = 0, No transition at start = 1
  Transition always in MIDDLE (for clocking)
  Used in Token Ring

4B/5B:
  Every 4 bits encoded as 5 bits (avoids long runs)
  Used in 100Base-TX (Fast Ethernet)
  Efficiency = 4/5 = 80%
```

### 🔧 Transmission Modes
```
SIMPLEX: One direction only. No return path.
  Example: keyboard to computer, TV broadcast

HALF-DUPLEX: Both directions, but NOT simultaneously.
  Example: Walkie-talkie, old Ethernet (CSMA/CD)

FULL-DUPLEX: Both directions simultaneously.
  Example: Telephone, modern Ethernet with switches
```

---

## 5. Bandwidth, Throughput & Latency

### 🔧 Key Definitions
```
BANDWIDTH:  Maximum data rate the channel CAN support (capacity).
            Measured in bps (bits per second).
            Hardware property — doesn't change with traffic.

THROUGHPUT: Actual data rate ACHIEVED in practice.
            Always ≤ Bandwidth.
            Affected by: congestion, errors, protocol overhead.

LATENCY:    Time for a bit to travel from source to destination.
(DELAY)     = Propagation delay + Transmission delay + Queuing delay + Processing delay

PROPAGATION DELAY:  Distance / Speed of signal
                    tp = d / v
                    (v ≈ 2×10⁸ m/s in copper, 3×10⁸ m/s in vacuum)

TRANSMISSION DELAY: Packet size / Bandwidth
                    tt = L / B
                    (time to push all bits onto the wire)

QUEUING DELAY:      Time waiting in router/switch queue (variable)

PROCESSING DELAY:   Time to process header, check errors (usually small)
```

### 🔧 Bandwidth-Delay Product
```
BDP = Bandwidth × Round-Trip Time
    = B × RTT

Represents: amount of data "in flight" (in the pipe) at any time.
            = pipe capacity in bits

Example:
  B = 1 Gbps, RTT = 10 ms
  BDP = 10⁹ × 10×10⁻³ = 10⁷ bits = 1.25 MB

If you want to keep the pipe full (max utilization):
  Window size ≥ BDP
```

---

## 6. Nyquist & Shannon Theorems

### 🔧 Nyquist Theorem — Noiseless Channel
```
For a noiseless channel:
Maximum Bit Rate = 2 × B × log₂(L)

Where:
  B = bandwidth of channel (Hz)
  L = number of discrete signal levels
  log₂(L) = bits per signal level

Example:
  B = 3000 Hz, L = 8 signal levels
  Max bit rate = 2 × 3000 × log₂(8) = 2 × 3000 × 3 = 18,000 bps = 18 Kbps
```

### 🔧 Shannon's Theorem — Noisy Channel
```
Maximum Channel Capacity = B × log₂(1 + S/N)

Where:
  B   = bandwidth (Hz)
  S/N = Signal-to-Noise Ratio (SNR, linear scale, NOT dB)
  C   = capacity in bps

Converting SNR dB to linear:
  SNR_linear = 10^(SNR_dB / 10)
  Example: SNR = 30 dB → linear = 10^3 = 1000

Example:
  B = 3000 Hz, SNR = 30 dB
  SNR_linear = 1000
  C = 3000 × log₂(1001) ≈ 3000 × 9.97 ≈ 29,900 bps ≈ 30 Kbps
```

### 📝 Combined Nyquist + Shannon Numerical

**Given:** B = 1 MHz, SNR = 63 (linear), signal levels L = 4
**Find:** (a) Nyquist max rate, (b) Shannon capacity, (c) practical max rate

```
(a) Nyquist: 2 × 1×10⁶ × log₂(4) = 2 × 10⁶ × 2 = 4 Mbps

(b) Shannon: 10⁶ × log₂(1+63) = 10⁶ × log₂(64) = 10⁶ × 6 = 6 Mbps

(c) Practical max = min(Nyquist, Shannon) = min(4, 6) = 4 Mbps
    Shannon is the UPPER BOUND — Nyquist says we can't exceed 4 Mbps
    with only 4 signal levels even though channel could support 6 Mbps.
    To reach 6 Mbps → need L = 8 levels (Nyquist = 2×10⁶×3 = 6 Mbps).
```

---

## 7. Transmission Media

### 🔧 Guided Media (Wired)

#### Twisted Pair Cable
```
UTP (Unshielded Twisted Pair):
  Categories: Cat3(10Mbps), Cat5e(1Gbps), Cat6(10Gbps)
  Most common LAN cable, cheap, easy to install
  Susceptible to EMI (electromagnetic interference)

STP (Shielded Twisted Pair):
  Metal shield reduces interference
  More expensive, harder to install
  Used in environments with high EMI
```

#### Coaxial Cable
```
Central copper wire + insulator + metal shield + outer jacket
Better noise immunity than twisted pair
Used in: cable TV, old Ethernet (10Base2, 10Base5)
Two types: Baseband (digital, single channel) and Broadband (analog, multiple channels)
```

#### Fiber Optic
```
Light pulses through glass/plastic core
FASTEST, IMMUNE to EMI, no signal degradation over long distance
Most expensive, hardest to install

Single-mode fiber:
  Very thin core (~9 μm), laser light
  Long distance (100+ km), high bandwidth
  More expensive
  Used in: WAN, submarine cables

Multi-mode fiber:
  Thicker core (~50-62.5 μm), LED light
  Shorter distance (~2 km), multiple light paths (modes)
  Less expensive
  Used in: campus networks, data centers
```

### 🔧 Unguided Media (Wireless)

```
RADIO WAVES:
  Low frequency, long range, penetrates walls
  Omnidirectional
  Used in: AM/FM radio, Wi-Fi (2.4 GHz, 5 GHz), cellular

MICROWAVES:
  High frequency, line-of-sight
  Used in: satellite communication, long-distance telephone, radar

INFRARED:
  Very short range, can't penetrate walls
  Used in: TV remotes, IrDA devices
  Line-of-sight required
```

---

## 8. Multiplexing

### 🔧 FDM — Frequency Division Multiplexing
```
Different signals transmitted on different FREQUENCIES simultaneously.
Each signal gets a dedicated frequency band (channel).
Used for: analog signals, cable TV, FM radio, DSL (ADSL separates voice and data)

          freq1  freq2  freq3
          ──────────────────── → single medium
          signal1 signal2 signal3

No time sharing — all signals simultaneous at different frequencies.
Guard bands separate channels to prevent interference.
```

### 🔧 TDM — Time Division Multiplexing
```
Different signals transmitted in different TIME SLOTS.
Each signal gets the full bandwidth but only for a fraction of time.

SYNCHRONOUS TDM: Fixed time slots per source (even if source has nothing to send)
  Wastes bandwidth when sources are idle.
  Used in: T1/E1 telephone lines

STATISTICAL TDM (STDM): Time slots allocated ON DEMAND.
  More efficient — idle sources don't waste slots.
  Used in: packet switching networks

Frame structure (example, 3 sources):
  | S1 slot | S2 slot | S3 slot | S1 slot | S2 slot | S3 slot | ...
  ←──── one TDM frame ────────→
```

### 🔧 WDM — Wavelength Division Multiplexing
```
Like FDM but for optical fiber.
Different data streams on different WAVELENGTHS (colors) of light.
DWDM (Dense WDM): 80+ wavelengths on single fiber!
Used in: long-haul fiber optic networks, internet backbone.
```

### 🔧 CDM — Code Division Multiplexing (CDMA)
```
Each signal encoded with a unique CODE.
All signals transmitted simultaneously on SAME frequency.
Receiver uses code to extract its signal (orthogonal codes).
Used in: cellular networks (3G CDMA), GPS.
```

---

## 9. Numericals — Full Solved Sets

### 📝 Numerical 1 — Propagation + Transmission Delay

**A 1000-bit frame sent over a 1 Mbps link. Distance = 1000 km. Signal speed = 2×10⁸ m/s. Find total delay.**

```
Transmission delay = Frame size / Bandwidth
  tt = 1000 bits / 1×10⁶ bps = 1×10⁻³ s = 1 ms

Propagation delay = Distance / Speed
  tp = 1000×10³ m / 2×10⁸ m/s = 10⁶ / 2×10⁸ = 5×10⁻³ s = 5 ms

Total delay = tt + tp = 1 + 5 = 6 ms
```

---

### 📝 Numerical 2 — Nyquist Theorem

**A telephone channel has bandwidth 4000 Hz. If we use 16 signal levels, what is the maximum bit rate?**

```
Nyquist: Max bit rate = 2 × B × log₂(L)
  = 2 × 4000 × log₂(16)
  = 2 × 4000 × 4
  = 32,000 bps = 32 Kbps
```

---

### 📝 Numerical 3 — Shannon's Theorem

**A channel has bandwidth 5 MHz and SNR of 1000. Find Shannon capacity.**

```
C = B × log₂(1 + SNR)
  = 5×10⁶ × log₂(1001)
  ≈ 5×10⁶ × 9.97
  ≈ 49.85 Mbps ≈ 50 Mbps
```

---

### 📝 Numerical 4 — TDM Frame Rate

**4 sources, each with 1 Kbps data rate. TDM with time slot = 1 bit per source. Find: (a) frame duration, (b) frame rate, (c) total output rate.**

```
Frame = 4 slots × 1 bit = 4 bits per frame

Each source must get 1000 bits/sec = 1000 frames/sec (since 1 bit per slot per frame)

(a) Frame duration = 1 / frame_rate = 1/1000 s = 1 ms
(b) Frame rate = 1000 frames/sec
(c) Total output = 4 bits/frame × 1000 frames/sec = 4000 bps = 4 Kbps
    (= sum of all source rates = 4 × 1 Kbps ✓)
```

---

### 📝 Numerical 5 — Bandwidth-Delay Product

**Link: bandwidth = 100 Mbps, one-way propagation delay = 20 ms. How many bits fill the pipe? What window size (in bytes) is needed for 100% utilization?**

```
BDP = B × RTT = 100×10⁶ × 2×20×10⁻³ = 100×10⁶ × 0.04 = 4×10⁶ bits

Bytes = 4×10⁶ / 8 = 500,000 bytes = 500 KB

Minimum window size for 100% utilization = BDP = 500 KB
(Sender needs to keep 500KB of unacknowledged data in flight at all times)
```

---

## 10. MCQ Traps & Exam Q&A

### ⚠️ THE TRAPS

---

**TRAP 1: Where does ARP operate?**
```
ARP is between Layer 2 and Layer 3.
ARP uses Layer 2 (Ethernet frames) but resolves Layer 3 (IP) addresses.
Most exams: Layer 2 or "between L2 and L3"
Technically: ARP is a Network Layer protocol that uses Data Link Layer.
```

---

**TRAP 2: TCP/IP has 4 or 5 layers?**
```
ORIGINAL TCP/IP = 4 layers.
Some textbooks (Forouzan) show 5 layers: Physical + Data Link + Internet + Transport + Application.
The 5-layer model is actually a hybrid/modified model.
For exams: TCP/IP = 4 layers (Application, Transport, Internet, Network Access).
If they say "5-layer TCP/IP" they mean the hybrid model.
```

---

**TRAP 3: OSI vs TCP/IP — which is used in practice?**
```
OSI = reference model (theoretical, never fully implemented)
TCP/IP = actual internet protocol suite (what the internet uses)
OSI is used for UNDERSTANDING. TCP/IP is used for COMMUNICATION.
```

---

**TRAP 4: Hub vs Switch vs Router layers**
```
Hub:     Layer 1 (Physical) — broadcasts to all ports, no intelligence
Switch:  Layer 2 (Data Link) — uses MAC addresses, sends to correct port
Router:  Layer 3 (Network)   — uses IP addresses, routes between networks
Gateway: Layer 4-7            — translates between different protocols
```

---

**TRAP 5: Nyquist needs noiseless, Shannon includes noise**
```
Nyquist = theoretical max for NOISELESS channel
Shannon = theoretical max for NOISY channel (absolute limit)
Practical rate = min(Nyquist, Shannon)
Shannon gives UPPER BOUND — can never exceed it regardless of signal levels.
```

---

**TRAP 6: Transmission delay vs Propagation delay**
```
Transmission delay = packet size / bandwidth → depends on PACKET SIZE and BANDWIDTH
Propagation delay = distance / speed → depends on DISTANCE and MEDIUM
They are independent! A short packet on a slow link may have:
  - Small propagation delay (close nodes)
  - Large transmission delay (slow link)
```

---

**TRAP 7: Full-duplex vs Half-duplex devices**
```
Hub:    Half-duplex (CSMA/CD needed — collision domain)
Switch: Full-duplex per port (no collision domain per port)
Router: Full-duplex
Modern Ethernet with switch: full-duplex (no collisions!)
Old Ethernet with hub/coax: half-duplex (CSMA/CD)
```

---

### ⚡ Quick Fire Q&A

| Question | Answer |
|---|---|
| OSI layer for routing? | Layer 3 (Network) |
| OSI layer for MAC addresses? | Layer 2 (Data Link) |
| OSI layer for end-to-end reliability? | Layer 4 (Transport) |
| Data unit at Layer 3? | Packet |
| Data unit at Layer 2? | Frame |
| Data unit at Layer 4 (TCP)? | Segment |
| Device at Layer 1? | Hub, Repeater |
| Device at Layer 2? | Switch, Bridge |
| Device at Layer 3? | Router |
| Protocol for error reporting? | ICMP |
| Port number for HTTP? | 80 |
| Port number for HTTPS? | 443 |
| Port number for FTP? | 21 (control), 20 (data) |
| Port number for SSH? | 22 |
| Port number for DNS? | 53 |
| Port number for SMTP? | 25 |
| TCP vs UDP key difference? | TCP=reliable+connected, UDP=fast+connectionless |
| Nyquist formula? | 2 × B × log₂(L) |
| Shannon formula? | B × log₂(1 + SNR) |
| Propagation delay formula? | Distance / Signal speed |
| Transmission delay formula? | Packet size / Bandwidth |
| Manchester encoding advantage? | Self-clocking (sync information built in) |
| FDM used for? | Analog multiplexing (different frequencies) |
| TDM used for? | Digital multiplexing (different time slots) |
| WDM used for? | Fiber optic (different wavelengths) |
| Single-mode vs multi-mode fiber? | Single: longer distance, laser. Multi: shorter, LED. |

---

### 📋 Quick Reference

```
OSI 7 LAYERS (top→bottom):
7-Application:  Message,  HTTP/FTP/SMTP/DNS/DHCP
6-Presentation: Data,     SSL/JPEG/ASCII
5-Session:      Data,     NetBIOS/RPC
4-Transport:    Segment,  TCP/UDP
3-Network:      Packet,   IP/ICMP/OSPF/BGP
2-Data Link:    Frame,    Ethernet/ARP/PPP — Switch/Bridge
1-Physical:     Bit,      RS232/DSL — Hub/Repeater

TCP/IP 4 LAYERS:
Application (= OSI 5+6+7)
Transport   (= OSI 4)
Internet    (= OSI 3)
Network Access (= OSI 1+2)

KEY FORMULAS:
Nyquist:  Max_rate = 2 × B × log₂(L)
Shannon:  C = B × log₂(1 + SNR)
t_trans:  L / B  (bits / bps = seconds)
t_prop:   d / v  (meters / m/s = seconds)
BDP:      B × RTT  (bits in transit)
```

---
