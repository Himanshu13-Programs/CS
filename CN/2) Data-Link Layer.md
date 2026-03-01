# 📘 CN Chapter 2 — Data Link Layer: MAC, ARP, Ethernet & Error Detection
---

## 📌 Table of Contents
1. [Data Link Layer — Overview](#1-data-link-layer--overview)
2. [Framing](#2-framing)
3. [Error Detection](#3-error-detection)
4. [Error Correction — Hamming Code](#4-error-correction--hamming-code)
5. [Flow Control Protocols](#5-flow-control-protocols)
6. [MAC Sub-layer & Multiple Access](#6-mac-sub-layer--multiple-access)
7. [Ethernet (IEEE 802.3)](#7-ethernet-ieee-8023)
8. [ARP — Address Resolution Protocol](#8-arp--address-resolution-protocol)
9. [Switches & Bridges](#9-switches--bridges)
10. [Numericals — Full Solved Sets](#10-numericals--full-solved-sets)
11. [MCQ Traps & Exam Q&A](#11-mcq-traps--exam-qa)

---

## 1. Data Link Layer — Overview

```
PURPOSE: Reliable node-to-node delivery on a SINGLE NETWORK SEGMENT.

Two sub-layers:
  LLC (Logical Link Control) — upper sub-layer
    - Error detection/correction
    - Flow control
    - Multiplexing protocols

  MAC (Media Access Control) — lower sub-layer
    - MAC addressing (hardware addresses)
    - Media access control (who gets to transmit)
    - Frame construction

KEY FUNCTIONS:
  1. Framing         — package bits into frames with boundaries
  2. Addressing      — source and destination MAC addresses
  3. Error detection — CRC, parity, checksum
  4. Flow control    — don't overwhelm receiver
  5. Access control  — CSMA/CD, CSMA/CA, Token Ring
```

---

## 2. Framing

### 🔧 Why Framing?
```
Physical layer delivers raw bits — no boundaries between messages.
Data Link layer adds FRAME DELIMITERS so receiver knows where each frame starts/ends.
```

### 🔧 Framing Methods

#### Character Count
```
First field in frame = LENGTH of frame.
Problem: If count field corrupted → receiver loses sync forever.
Rarely used alone.
```

#### Flag Bytes with Byte Stuffing
```
Special FLAG byte (e.g., 0x7E) marks frame start and end.
If FLAG byte appears in DATA → stuff escape byte before it.
If ESCAPE byte appears in DATA → stuff another escape before it.

Example (PPP uses 0x7E as flag, 0x7D as escape):
  Data: ... 0x7E ... 0x7D ...
  Transmitted: ... 0x7D 0x5E ... 0x7D 0x5D ...
  (XOR data byte with 0x20 after stuffing)

Receiver: strip escape bytes, un-XOR → recover original data.
```

#### Bit Stuffing (HDLC)
```
Flag pattern: 01111110 (six 1s between 0s)
Rule: After every five consecutive 1s in data → INSERT a 0.
Receiver: sees five 1s followed by 0 → REMOVE the 0 (destuff).
If sees five 1s followed by 1 → it's a flag or abort.

Example:
  Data:     011111111110
  Stuffed:  0111110111110
  (extra 0 inserted after each run of five 1s)
```

---

## 3. Error Detection

### 🔧 Parity Check

#### Simple (Single-bit) Parity
```
Add ONE parity bit to make total 1-bits EVEN (even parity) or ODD (odd parity).

Even parity: total number of 1s (including parity bit) is even.
  Data: 1011001 → four 1s → already even → parity bit = 0
  Data: 1011000 → three 1s → odd → parity bit = 1

Detection: single bit errors only.
Cannot detect: 2-bit errors (cancel each other out).
Cannot correct any errors (only detects).
```

#### 2D Parity (Vertical Redundancy Check)
```
Arrange data in a grid. Add parity bit for each row AND each column.
Can detect and CORRECT single-bit errors.
Detects all 2-bit errors. Most 3-bit and 4-bit errors detected.

Example (even parity):
  Data rows:          Row parity:
  1 0 1 1              1
  0 1 1 0              0
  1 1 0 1              1
  ───────────
  Col parity:
  0 0 0 0

If bit at row 2, col 3 flips → row 2 parity fails, col 3 parity fails → identifies exact position!
```

### 🔧 Checksum
```
SENDER:
  1. Divide data into k-bit segments
  2. Add all segments using 1's complement arithmetic
  3. Take 1's complement of sum → CHECKSUM
  4. Append checksum to data

RECEIVER:
  1. Add all segments INCLUDING checksum
  2. Result should be all 1s (if no error)
  3. If not all 1s → error detected

1's COMPLEMENT ADDITION:
  Normal binary add, but if carry out of MSB → wrap around (add 1 to result).

Example: Add 10011001 and 11100010
  10011001
+ 11100010
-----------
1 01111011  (carry out!)
+         1 (wrap around)
-----------
  01111100  (sum)
  10000011  (checksum = 1's complement)

Stronger than parity — detects all single-bit and most multi-bit errors.
Used in: IP, TCP, UDP headers.
```

### 🔧 CRC — Cyclic Redundancy Check ⭐ (Most Tested)

#### Concept
```
CRC treats data as a POLYNOMIAL.
Both sender and receiver agree on a GENERATOR polynomial G(x).

Sender:
  1. Append r zeros to message M (where r = degree of G)
  2. Divide M×x^r by G using binary long division (XOR division)
  3. Remainder R = CRC (checksum)
  4. Send M followed by R (replace the r zeros with R)

Receiver:
  1. Divide received message (data + CRC) by G
  2. If remainder = 0 → no error
  3. If remainder ≠ 0 → error detected

KEY FACT: CRC detects ALL single-bit errors, ALL double-bit errors,
          ALL odd number of errors, ALL burst errors of length ≤ r.
```

#### CRC Division — Binary XOR Long Division
```
Rules:
  XOR replaces subtraction (both give same result for binary!)
  1 XOR 1 = 0
  1 XOR 0 = 1
  0 XOR 1 = 1
  0 XOR 0 = 0

  Align divisor with leftmost 1 of dividend.
  XOR them. Bring down next bit.
  Repeat until dividend is smaller than divisor.
  Remainder = CRC.
```

### 📝 CRC Numerical — Step by Step

**Message M = 1101011011, Generator G = 10011**

```
Step 1: Degree of G = 4 (G has 5 bits, highest power = x⁴)
        Append 4 zeros to M: 11010110110000

Step 2: Binary long division (XOR division):

        11010110110000  ÷  10011
        
     10011 | 11010110110000
            10011
            ─────
             10011
             10011
             ─────
              00001011
              ... let me do this properly:

Dividend: 1 1 0 1 0 1 1 0 1 1 0 0 0 0
Divisor:  1 0 0 1 1

Step-by-step XOR division:
  Take first 5 bits: 11010
  11010
  10011 ← XOR
  ─────
  01001  → leading 0, bring down next bit → 10011

  10011
  10011 ← XOR
  ─────
  00000  → bring down → 00001 → bring down → 00011 → bring down → 00110

  Wait, when result starts with 0, bring down until starts with 1:
  After XOR: 00000, bring down 1 → 000001, bring down 1 → 0000011
  Still less than divisor (10011), so bring down 0 → 00000110
  00000110 → still less → bring down 0 → 000001100
  
Let me redo cleanly:

  11010110110000
  
  Step 1: 11010 XOR 10011 = 01001 → bring down 1 → 10011
  Step 2: 10011 XOR 10011 = 00000 → bring down 1 → 00001 → 0
            → bring down 0 → 000010 → bring down 0 → 0000100 →
            Hmm, only 5-bit divisor, keep bringing down till 5-bit with leading 1
  Step 2: 10011 XOR 10011 = 00000, bring down 1→00001, bring down 0→00010, bring down 0→00100
  Step 3: 00100 < 10011, so we consider this as 0... bring down 0 → 001000
  
  Actually, standard CRC long division:

  Msg appended: 11010110110000

  ┌──────────────────────────────
  │11010110110000 ÷ 10011
  │
  │11010  ÷ 10011 → quotient bit 1
  │10011
  │─────
  │01001  → bring down 1 → 10011
  │
  │10011 ÷ 10011 → quotient bit 1
  │10011
  │─────
  │00000 → bring down 1 → 00001
  │        bring down 0 → 00010
  │        bring down 0 → 00100
  │        (all less than 10011 → quotient bits 0,0,0)
  │        bring down 0 → 01000
  │        (less than 10011 → quotient bit 0)
  │        This seems wrong - let me restate with proper 5-bit windows:

Proper CRC long division for 11010110110000 ÷ 10011:

Position:  1 1 0 1 0 1 1 0 1 1 0 0 0 0

11010  XOR 10011 = 01001   ←  first 5 bits
 1001 1 (shift, bring down next '1') → 10011
10011  XOR 10011 = 00000
 0000 1 (shift, bring down '1') → 00001
 0001 0 (shift, bring down '0') → 00010  
 0010 1 (shift, bring down '1') → 00101  <-- wait next digit is '0' not '1'

Let me list digits clearly:
M appended = 1 1 0 1 0 1 1 0 1 1 0 0 0 0
positions:   1 2 3 4 5 6 7 8 9 ...

Division:
Take [1-5] = 11010, XOR 10011 = 01001
Shift left (bring in bit 6=1): 10011
[result]: 10011, XOR 10011 = 00000
Shift (bring in bit 7=1): 00001 → too small
Shift (bring in bit 8=0): 00010 → too small  
Shift (bring in bit 9=1): 00101 → too small
Shift (bring in bit 10=1): 01011 → too small
Shift (bring in bit 11=0): 10110
10110 XOR 10011 = 00101
Shift (bring in bit 12=0): 01010 → too small
Shift (bring in bit 13=0): 10100
10100 XOR 10011 = 00111
Shift (bring in bit 14=0): 01110 → too small

REMAINDER = 1110 (last 4-bit remainder, since divisor degree = 4)

Step 3: CRC (FCS) = 1110
Step 4: Transmitted frame = 1101011011 1110
                            ←message→ ←CRC→

VERIFICATION:
  Receiver divides 11010110111110 by 10011.
  If remainder = 0000 → no error. ✓
```

---

## 4. Error Correction — Hamming Code

### 🧠 Concept
```
Hamming code: adds REDUNDANCY BITS (parity bits) at positions that are powers of 2.
Can DETECT and CORRECT single-bit errors.
Can DETECT (not correct) double-bit errors (with extra parity bit — SECDED).

Parity bit positions: 1, 2, 4, 8, 16, 32, ...  (powers of 2)
Data bit positions:   3, 5, 6, 7, 9, 10, 11, ...  (everything else)
```

### 🔧 How Many Parity Bits?
```
For m data bits, need r parity bits such that:
  2^r ≥ m + r + 1

Examples:
  m=1: 2^r ≥ r+2 → r=2 (4 ≥ 3 ✓)
  m=4: 2^r ≥ r+5 → r=3 (8 ≥ 7 ✓)
  m=8: 2^r ≥ r+9 → r=4 (16 ≥ 13 ✓)
  m=11: 2^r ≥ r+12 → r=4 (16 ≥ 15 ✓)
  m=26: 2^r ≥ r+27 → r=5 (32 ≥ 31 ✓)
```

### 🔧 Parity Bit Coverage
```
Each parity bit Pi covers positions where bit i of the position number is 1:

P1 (position 1 = 0001): covers positions 1,3,5,7,9,11,... (all with bit0=1)
P2 (position 2 = 0010): covers positions 2,3,6,7,10,11,.. (all with bit1=1)
P4 (position 4 = 0100): covers positions 4,5,6,7,12,13,.. (all with bit2=1)
P8 (position 8 = 1000): covers positions 8,9,10,11,12,...  (all with bit3=1)

RULE: Position n is covered by parity bit Pi if the i-th bit of n's binary representation is 1.
```

### 📝 Hamming Code Numerical — Encoding

**Data: 1011 (4 bits). Use even parity. Find Hamming code.**

```
Step 1: Number of parity bits r.
  m=4. 2^r ≥ 4+r+1 → r=3 (2³=8 ≥ 8 ✓ barely works)
  Total bits = m + r = 4 + 3 = 7

Step 2: Assign positions.
  Position: 1  2  3  4  5  6  7
  Type:     P1 P2 D1 P4 D2 D3 D4
  
  Data bits D1 D2 D3 D4 = 1 0 1 1
  Place data in non-power-of-2 positions:
  Position: 1  2  3  4  5  6  7
  Bits:     P1 P2  1 P4  0  1  1

Step 3: Calculate parity bits (even parity).

  P1 covers positions 1,3,5,7 → bits: P1, 1, 0, 1
    XOR of data bits: 1 XOR 0 XOR 1 = 0 → P1 = 0

  P2 covers positions 2,3,6,7 → bits: P2, 1, 1, 1
    XOR of data bits: 1 XOR 1 XOR 1 = 1 → P2 = 1

  P4 covers positions 4,5,6,7 → bits: P4, 0, 1, 1
    XOR of data bits: 0 XOR 1 XOR 1 = 0 → P4 = 0

Step 4: Final Hamming code:
  Position: 1  2  3  4  5  6  7
  Bits:     0  1  1  0  0  1  1
  
  Hamming code: 0110011
```

### 📝 Hamming Code Numerical — Error Detection & Correction

**Received code: 0110111 (from above, where correct is 0110011). Find and correct error.**

```
Step 1: Check parity bits.

  P1: covers pos 1,3,5,7 → bits 0,1,1,1 → XOR = 0 XOR 1 XOR 1 XOR 1 = 1 ≠ 0 → FAIL
  P2: covers pos 2,3,6,7 → bits 1,1,1,1 → XOR = 1 XOR 1 XOR 1 XOR 1 = 0 ✓ PASS
  P4: covers pos 4,5,6,7 → bits 0,1,1,1 → XOR = 0 XOR 1 XOR 1 XOR 1 = 1 ≠ 0 → FAIL

Step 2: Syndrome = failed parity bits (read as binary number, P4 is MSB)
  P4 failed = 1, P2 passed = 0, P1 failed = 1
  Syndrome = P4 P2 P1 = 1 0 1 = 5 (decimal)

Step 3: Error is at position 5!
  Position: 1  2  3  4  5  6  7
  Received: 0  1  1  0  1  1  1  ← bit 5 = 1
  Correct:  0  1  1  0  0  1  1  ← flip bit 5 → 0

Corrected code: 0110011 ✓ (matches original)
```

---

## 5. Flow Control Protocols

### 🔧 Stop-and-Wait (Simplest)
```
Sender sends ONE frame → waits for ACK → sends next frame.
Very inefficient for large bandwidth-delay products.

Efficiency = 1 / (1 + 2a)   where a = tp / tt
  tp = propagation delay, tt = transmission delay

Example: tt = 1ms, tp = 5ms
  a = 5/1 = 5
  Efficiency = 1/(1+10) = 1/11 ≈ 9%  ← terrible!
```

### 🔧 Sliding Window Protocol
```
Sender can have up to W UNACKNOWLEDGED frames in transit.
"Window" of W frames can be outstanding at once.

SENDER WINDOW: frames waiting for ACK
RECEIVER WINDOW: frames it can accept

Efficiency with sliding window:
  If W ≥ 1 + 2a:  Efficiency = 1 (100% — pipe always full!)
  If W < 1 + 2a:  Efficiency = W / (1 + 2a)

Optimal window size = 1 + 2a = 1 + 2(tp/tt)
```

### 🔧 Go-Back-N (GBN)
```
Sender window size: up to 2^n - 1 frames (n = sequence number bits)
Receiver window size: 1 (only accepts in-order frames)

On ERROR: receiver DISCARDS error frame AND all subsequent frames.
Sender must RETRANSMIT from error frame onwards (goes back N).

Wasteful if errors are frequent.
Receiver buffer: 1 frame only.
Sequence numbers needed: 2^n
Max window size: 2^n - 1
```

### 🔧 Selective Repeat (SR)
```
Sender window size: 2^(n-1) frames
Receiver window size: 2^(n-1) frames

On ERROR: receiver BUFFERS out-of-order frames.
Sender retransmits ONLY the errored frame.

More efficient (less retransmission) but needs more receiver buffer.
Max window size: 2^(n-1)  ← EXACTLY HALF of sequence space
```

### 📊 GBN vs SR Comparison
| Feature | Go-Back-N | Selective Repeat |
|---|---|---|
| **Sender window** | 2^n - 1 | 2^(n-1) |
| **Receiver window** | 1 | 2^(n-1) |
| **On error** | Retransmit from error | Retransmit only errored |
| **Buffer needed** | Low (sender only) | High (both sides) |
| **Efficiency** | Lower (more retransmit) | Higher |

### 📝 Window Size Numerical

**n = 3 sequence number bits. What are max window sizes for GBN and SR?**
```
Sequence numbers: 0 to 2^3 - 1 = 0 to 7 (8 numbers total)

GBN max sender window = 2^3 - 1 = 7
SR  max sender window = 2^(3-1) = 2^2 = 4

WHY SR can't use 7?
  If sender window = 7 and all frames lost:
  Sender wraps around, sends frame 0 again.
  Receiver can't tell if it's a NEW frame 0 or RETRANSMIT of old frame 0!
  With window = 4: wraparound impossible in one window.
```

---

## 6. MAC Sub-layer & Multiple Access

### 🔧 CSMA/CD — Ethernet (Wired)
```
Carrier Sense Multiple Access / Collision Detection

Rules:
  1. CARRIER SENSE: Listen before transmitting. If busy → wait.
  2. MULTIPLE ACCESS: All stations share medium.
  3. COLLISION DETECTION: While transmitting → monitor for collision.
     If collision → stop, send JAM signal, wait random backoff time, retry.

BINARY EXPONENTIAL BACKOFF:
  After k-th collision: wait random time in [0, 2^k - 1] × slot_time
  k=1: wait 0 or 1 slots
  k=2: wait 0-3 slots
  k=10: wait 0-1023 slots
  After 16 failures → give up, report error.

MINIMUM FRAME SIZE:
  Frame must be long enough so sender is still transmitting when collision detected.
  Minimum frame size = 2 × tp × B (bits)
  Ethernet minimum: 64 bytes (512 bits)

WHY: Propagation delay both ways = 2tp.
     If frame finishes before 2tp → sender can't detect collision!
```

### 🔧 CSMA/CA — Wi-Fi (Wireless)
```
Carrier Sense Multiple Access / Collision AVOIDANCE

Can't detect collisions (radio: transmit and receive on same frequency → can't hear collisions)
So → AVOID collisions instead of detecting them.

Steps:
  1. Listen. If channel idle for DIFS (Distributed Inter Frame Space) → proceed.
  2. Choose random backoff in contention window.
  3. Count down backoff. If channel busy → pause countdown.
  4. When countdown = 0 → transmit.
  5. Wait for ACK. If no ACK → assume collision, retry.

RTS/CTS (optional):
  Sender sends RTS (Request To Send) → receiver responds CTS (Clear To Send)
  Solves HIDDEN TERMINAL PROBLEM (two senders can't hear each other).
```

### 🔧 Pure ALOHA & Slotted ALOHA
```
PURE ALOHA: Transmit whenever you have data. If collision → wait random time, retry.
  Efficiency: 18.4% (max throughput = 1/(2e) ≈ 0.184)

SLOTTED ALOHA: Time divided into slots. Transmit only at START of slot.
  Efficiency: 36.8% (max throughput = 1/e ≈ 0.368)
  EXACTLY 2× Pure ALOHA efficiency.

Why Pure ALOHA has lower efficiency:
  Vulnerable period = 2 × slot time (frame can collide if other starts
  during full frame duration before or after).
  Slotted: vulnerable period = 1 × slot time.
```

---

## 7. Ethernet (IEEE 802.3)

### 🔧 Ethernet Frame Format
```
┌─────────┬──────────┬──────────┬──────┬──────┬────────────┬─────┐
│Preamble │Dest MAC  │Src MAC   │Type/ │ Data │   Padding  │ FCS │
│8 bytes  │6 bytes   │6 bytes   │Length│46-1500│(if needed)│4 B  │
│         │          │          │2 bytes│ bytes │           │     │
└─────────┴──────────┴──────────┴──────┴──────┴────────────┴─────┘

Preamble: 7 bytes of 10101010... + 1 byte 10101011 (SFD = Start Frame Delimiter)
  Purpose: clock synchronization, tells NIC frame is coming.

Dest/Src MAC: 6 bytes each (48-bit MAC address)
  Format: XX:XX:XX:XX:XX:XX (hex octets)
  First 3 bytes: OUI (Organizationally Unique Identifier) — manufacturer
  Last 3 bytes: device-specific

Type/Length:
  ≥ 1536 (0x0600): EtherType (type of payload: 0x0800=IPv4, 0x86DD=IPv6, 0x0806=ARP)
  ≤ 1500: Length of data (older 802.3 format)

Data: 46-1500 bytes (minimum 46 to ensure 64-byte minimum frame)
Padding: added if data < 46 bytes
FCS: 4-byte CRC for error detection
```

### 🔧 Ethernet Variants
```
10Base5 (Thick Ethernet): 10 Mbps, coaxial, 500m
10Base2 (Thin Ethernet):  10 Mbps, coaxial, 185m
10BaseT:                  10 Mbps, twisted pair
100BaseTX (Fast Ethernet): 100 Mbps, Cat5 UTP
1000BaseT (Gigabit):      1 Gbps, Cat5e/Cat6
10GBaseT:                 10 Gbps, Cat6a/Cat7

Format: [Speed][Signaling][Medium/Distance]
BASE = baseband (digital), BROAD = broadband (analog)
```

---

## 8. ARP — Address Resolution Protocol

### 🧠 Purpose
```
Know: IP address of destination.
Need: MAC address (to put in Ethernet frame header).
ARP maps IP → MAC.

RARP (Reverse ARP): MAC → IP (deprecated, replaced by DHCP)
```

### 🔧 ARP Process
```
Step 1: Check ARP CACHE (table of IP→MAC mappings).
        If found and not expired → use it directly.

Step 2: If not in cache → BROADCAST ARP Request:
        "Who has IP 192.168.1.5? Tell 192.168.1.1"
        Dest MAC = FF:FF:FF:FF:FF:FF (broadcast)
        All devices on segment receive this.

Step 3: Device with IP 192.168.1.5 sends unicast ARP Reply:
        "192.168.1.5 is at AA:BB:CC:DD:EE:FF"
        Sent directly to requester (unicast, not broadcast).

Step 4: Requester stores mapping in ARP cache (typically 20 min timeout).

Step 5: Now can send frame to MAC AA:BB:CC:DD:EE:FF.
```

### 🔧 ARP Cache Poisoning (Security)
```
Attacker sends fake ARP replies: "192.168.1.1 is at [attacker's MAC]"
Victims update their ARP cache with wrong mapping.
Traffic meant for router goes to attacker (Man-in-the-Middle).
Defense: Static ARP entries, ARP inspection on switches.
```

### 🔧 Proxy ARP
```
Router responds to ARP requests on behalf of hosts on another network.
Hosts don't need to know about routing — router intercepts and responds.
```

### 🔧 Gratuitous ARP
```
Device sends ARP request for its OWN IP address.
Purpose:
  1. Check if another device has same IP (conflict detection)
  2. Update ARP caches of other devices (e.g., after MAC address change)
  Sent on startup and IP configuration change.
```

---

## 9. Switches & Bridges

### 🔧 Bridge
```
Connects two network segments at Layer 2.
Learns which MACs are on which segment (builds forwarding table).
Forwards frames only to correct segment (not broadcast everywhere).
Reduces collision domain.
```

### 🔧 Switch
```
Multi-port bridge. Each port = separate collision domain.
Full-duplex per port → no CSMA/CD needed!

SWITCH LEARNING ALGORITHM:
  When frame arrives on port X with source MAC = A:
    → Record: MAC A is reachable via port X (in MAC table)

  When frame arrives with destination MAC = B:
    → If B in MAC table → forward ONLY to that port (unicast)
    → If B NOT in table → FLOOD to all ports except source (like broadcast)
    → If B = FF:FF:FF:FF:FF:FF → FLOOD to all ports (broadcast)

Switch = multiple collision domains, one broadcast domain.
Router = multiple collision domains, multiple broadcast domains.
```

### 🔧 STP — Spanning Tree Protocol (IEEE 802.1D)
```
Problem: Loops in switched network → broadcast storms (frames loop forever).
STP: Logically disables some ports to create a LOOP-FREE topology.

Steps:
  1. Elect ROOT BRIDGE (switch with lowest Bridge ID = priority + MAC)
  2. Each non-root switch: find ROOT PORT (best path to root)
  3. Each segment: find DESIGNATED PORT (best port toward root)
  4. All other ports: BLOCKED (disabled, no forwarding)

Port states: Blocking → Listening → Learning → Forwarding → Disabled
Convergence time: ~30-50 seconds (problematic — RSTP = rapid STP = faster)
```

---

## 10. Numericals — Full Solved Sets

### 📝 Numerical 1 — Sliding Window Efficiency

**Link: bandwidth = 1 Mbps, propagation delay = 20 ms, frame size = 1000 bits.**
**Find: (a) transmission time, (b) value of 'a', (c) minimum window size for 100% efficiency, (d) efficiency with window size W=5.**

```
(a) Transmission time tt = frame size / bandwidth
    tt = 1000 bits / 1×10⁶ bps = 1 ms

(b) a = tp / tt = 20 ms / 1 ms = 20

(c) Minimum window W for 100% efficiency:
    W ≥ 1 + 2a = 1 + 2(20) = 41 frames

(d) Efficiency with W=5:
    Since W=5 < 1+2a=41:
    Efficiency = W / (1 + 2a) = 5 / 41 ≈ 12.2%
```

---

### 📝 Numerical 2 — GBN vs SR Window Sizes

**Sequence number field = 4 bits.**
**Find max window sizes for (a) Stop-and-Wait, (b) GBN, (c) SR.**

```
Sequence numbers: 2^4 = 16 (numbers 0-15)

(a) Stop-and-Wait: window = 1 (by definition)

(b) GBN max window = 2^n - 1 = 2^4 - 1 = 15

(c) SR max window = 2^(n-1) = 2^3 = 8
```

---

### 📝 Numerical 3 — Efficiency Comparison

**Propagation delay = 270 ms (satellite link), frame size = 512 bytes, bandwidth = 64 Kbps.**
**Compare Stop-and-Wait vs Sliding Window (W=127) efficiency.**

```
tt = (512 × 8) / 64000 = 4096/64000 = 64 ms
tp = 270 ms
a = 270/64 = 4.22
1 + 2a = 1 + 8.44 = 9.44

Stop-and-Wait (W=1):
  Efficiency = 1/(1+2a) = 1/9.44 ≈ 10.6%

Sliding Window W=127:
  W=127 > 1+2a=9.44 → Efficiency = 1 (100%)
  127 frames more than enough to fill the pipe!
  
If W=7:
  W=7 < 9.44 → Efficiency = 7/9.44 ≈ 74.2%
```

---

### 📝 Numerical 4 — ALOHA Throughput

**Pure ALOHA network. What is maximum throughput if there are 100 stations, each generating 1 frame/second? Frame transmission time = 100 ms.**

```
Maximum throughput of Pure ALOHA = 1/(2e) ≈ 18.4% of channel capacity.

If channel can transmit 1/0.1 = 10 frames/second (capacity):
Maximum useful throughput = 0.184 × 10 = 1.84 frames/second

So with 100 stations generating 1 frame/s each = 100 frames/s offered load.
Channel can handle max 1.84 frames/s throughput → severely congested.
```

---

## 11. MCQ Traps & Exam Q&A

### ⚠️ THE TRAPS

---

**TRAP 1: GBN window size = 2^n - 1, NOT 2^n**
```
With 3-bit sequence numbers (0-7):
GBN max window = 7 (NOT 8)
WHY: If window = 8 = 2^n, and all ACKs lost, sender retransmits 0-7.
     But receiver already accepted 0-7 and moved forward.
     Receiver can't distinguish new frames from old retransmits!
     Window must be < 2^n to prevent ambiguity.
```

---

**TRAP 2: SR window size = 2^(n-1), exactly half**
```
With 3 bits: SR window = 4 (NOT 7, NOT 8)
WHY: Sender and receiver windows must not overlap in sequence space.
     Each = 2^(n-1) ensures: sender_window + receiver_window = 2^n
     No overlap → no confusion between old and new frames.
```

---

**TRAP 3: ARP Reply is UNICAST, not broadcast**
```
ARP Request = broadcast (to find the MAC)
ARP Reply   = unicast   (sent directly back to requester)
Only the target replies, and only to the requester.
```

---

**TRAP 4: CRC remainder on correct reception = 0**
```
Receiver divides (data + CRC) by generator.
If no error → remainder = 0 (exactly divisible).
If error → remainder ≠ 0.
```

---

**TRAP 5: Hamming syndrome = error position**
```
The binary number formed by which parity checks fail
= exact position of the error.
Example: P4 fails, P2 passes, P1 fails → syndrome = 101₂ = 5 → bit 5 is wrong.
```

---

**TRAP 6: Switch creates multiple collision domains**
```
Each switch PORT = its own collision domain.
All switch ports = ONE broadcast domain (unless VLANs).
Router separates broadcast domains.
Hub = ONE collision domain (all ports share).
```

---

**TRAP 7: CSMA/CD used in wired Ethernet, CSMA/CA in wireless**
```
CSMA/CD: detect collision AFTER it happens, retransmit.
CSMA/CA: avoid collision BEFORE it happens.
Can't use CSMA/CD in wireless because you can't hear your own collision
while transmitting on a wireless channel.
```

---

### ⚡ Quick Fire Q&A

| Question | Answer |
|---|---|
| CRC appends how many zeros? | r zeros (r = degree of generator) |
| CRC remainder on error-free receive? | 0 |
| CRC can detect all burst errors of length? | ≤ r (degree of generator) |
| Hamming: parity bits at which positions? | Powers of 2: 1, 2, 4, 8, 16... |
| Hamming: m=8 needs r=? parity bits? | 4 (2⁴=16 ≥ 8+4+1=13) |
| Pure ALOHA max efficiency? | 18.4% (1/2e) |
| Slotted ALOHA max efficiency? | 36.8% (1/e) |
| GBN window size (n-bit seq num)? | 2^n - 1 |
| SR window size (n-bit seq num)? | 2^(n-1) |
| Stop-and-Wait efficiency formula? | 1 / (1 + 2a) |
| Sliding window efficiency (W < 1+2a)? | W / (1 + 2a) |
| ARP request destination MAC? | FF:FF:FF:FF:FF:FF (broadcast) |
| ARP reply type? | Unicast (to requester) |
| Switch operates at which layer? | Layer 2 (Data Link) |
| Hub operates at which layer? | Layer 1 (Physical) |
| STP purpose? | Prevent loops in switched networks |
| CSMA/CD backoff algorithm? | Binary exponential backoff |
| Ethernet minimum frame size? | 64 bytes |
| Ethernet max data payload? | 1500 bytes |
| MAC address length? | 6 bytes (48 bits) |
| Manchester encoding used in? | 10 Mbps Ethernet |

---
