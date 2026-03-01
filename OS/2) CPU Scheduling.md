# 📘 OS Chapter 2 — CPU Scheduling 
---

## 📌 Table of Contents
1. [Why CPU Scheduling?](#1-why-cpu-scheduling)
2. [Key Terms & Formulas](#2-key-terms--formulas)
3. [Scheduling Criteria](#3-scheduling-criteria)
4. [FCFS — First Come First Served](#4-fcfs--first-come-first-served)
5. [SJF — Shortest Job First (Non-Preemptive)](#5-sjf--shortest-job-first-non-preemptive)
6. [SRTF — Shortest Remaining Time First (Preemptive SJF)](#6-srtf--shortest-remaining-time-first-preemptive)
7. [Round Robin](#7-round-robin)
8. [Priority Scheduling](#8-priority-scheduling)
9. [MLFQ — Multilevel Feedback Queue](#9-mlfq--multilevel-feedback-queue)
10. [Numericals — Full Solved Sets](#10-numericals--full-solved-sets)
11. [Algorithm Comparison & Decision Guide](#11-algorithm-comparison--decision-guide)
12. [MCQ Traps & Exam Q&A](#12-mcq-traps--exam-qa)

---

## 1. Why CPU Scheduling?

### 🧠 The Core Problem
In a multiprogramming OS, multiple processes are in memory simultaneously.
When a process does I/O, it gives up the CPU. **Who gets the CPU next?**
That's the scheduler's job.

### 🔧 Two Types of Scheduler Behavior

```
NON-PREEMPTIVE (Cooperative):
  Once CPU is given to a process → it keeps CPU until:
    (a) it finishes, OR
    (b) it voluntarily gives up CPU (waiting for I/O)
  OS cannot forcibly take CPU back.

PREEMPTIVE:
  OS can forcibly take CPU from a running process at any time.
  Happens when: timer interrupt fires, higher priority process arrives,
                current process's time quantum expires.
  Enables fairness and responsiveness — required for interactive OS.
```

### 🔧 When Does Scheduling Happen?
```
1. Process switches from RUNNING → WAITING     (e.g., I/O request)
   → Must pick new process. Both preemptive and non-preemptive.

2. Process switches from RUNNING → READY       (e.g., timer interrupt)
   → Only in PREEMPTIVE scheduling.

3. Process switches from WAITING → READY       (e.g., I/O complete)
   → May trigger preemption if new process has higher priority.

4. Process TERMINATES
   → Must pick new process.

Cases 1 & 4: No choice — must schedule.
Cases 2 & 3: Choice — preemptive vs non-preemptive differs here.
```

### 🔧 Dispatcher vs Scheduler
```
SCHEDULER  → decides WHICH process to run next (policy)
DISPATCHER → actually does the switch (mechanism):
             1. Context switch (save/load PCB)
             2. Switch to user mode
             3. Jump to correct location in new process

DISPATCH LATENCY = time dispatcher takes to stop one process and start another.
Should be minimized — it's pure overhead.
```

---

## 2. Key Terms & Formulas

### 🔧 Definitions — Memorize These Cold

```
ARRIVAL TIME (AT)    → when process enters the ready queue
BURST TIME (BT)      → total CPU time the process needs to complete
COMPLETION TIME (CT) → when the process finishes execution
TURNAROUND TIME (TAT)→ total time from arrival to completion
                       TAT = CT - AT
WAITING TIME (WT)    → time spent WAITING in ready queue (not executing)
                       WT = TAT - BT
RESPONSE TIME (RT)   → time from arrival to FIRST time process gets CPU
                       RT = first_CPU_time - AT
                       (different from WT in Round Robin!)

THROUGHPUT           → number of processes completed per unit time
CPU UTILIZATION      → % of time CPU is doing useful work (not idle)
```

### 🔧 Formula Sheet
```
TAT = CT - AT
WT  = TAT - BT  =  CT - AT - BT
RT  = first_CPU_allocation - AT

Average WT  = (WT₁ + WT₂ + ... + WTₙ) / n
Average TAT = (TAT₁ + TAT₂ + ... + TATₙ) / n

CPU Utilization = (Total_time - Idle_time) / Total_time × 100%

Throughput = n / total_time_to_complete_all_n_processes
```

### 🔧 How to Draw a Gantt Chart
```
Step 1: List all processes with AT and BT
Step 2: Apply scheduling algorithm to determine execution order
Step 3: Draw timeline bar:
         |─────P1─────|──P2──|────P3────|
         0            5      8         14
Step 4: For each process, mark CT (when its bar ends)
Step 5: Calculate TAT = CT - AT for each
Step 6: Calculate WT = TAT - BT for each
Step 7: Calculate averages
```

---

## 3. Scheduling Criteria

### 🔧 What Makes a Good Scheduler?

| Criterion | Goal | Conflicts With |
|---|---|---|
| **CPU Utilization** | Maximize (keep CPU busy) | — |
| **Throughput** | Maximize (jobs/unit time) | Response time |
| **Turnaround Time** | Minimize (total job time) | — |
| **Waiting Time** | Minimize (time in queue) | Fairness |
| **Response Time** | Minimize (time to first response) | Throughput |
| **Fairness** | Each process gets fair CPU share | Optimal avg wait |

```
TRADEOFF: Minimizing average WT (SJF) → starvation for long processes
          Ensuring fairness (RR) → worse average WT
          There is NO perfect scheduling algorithm for all scenarios.
```

---

## 4. FCFS — First Come First Served

### 🧠 Concept
Simplest algorithm. Ready queue is a FIFO queue.
Process that arrives first gets CPU first.
**Non-preemptive.**

### 🔧 Algorithm
```
1. Sort processes by Arrival Time
2. Run first arrived process to COMPLETION
3. Run next arrived process to COMPLETION
4. Repeat
(If tie in AT → run in process order P1, P2...)
```

### 📝 Solved Example 1 — All arrive at t=0

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 24 |
| P2 | 0 | 3 |
| P3 | 0 | 3 |

```
Gantt Chart:
|────────────────P1────────────────|──P2──|──P3──|
0                                 24     27     30

CT:  P1=24, P2=27, P3=30
TAT: P1=24-0=24, P2=27-0=27, P3=30-0=30
WT:  P1=24-24=0, P2=27-3=24, P3=30-3=27

Avg WT  = (0 + 24 + 27) / 3 = 17
Avg TAT = (24 + 27 + 30) / 3 = 27

⚠️ CONVOY EFFECT: P2 and P3 (short jobs) wait 24ms behind P1 (long job)
   If order was P2, P3, P1:
   Avg WT = (0 + 3 + 6) / 3 = 3 — much better!
```

### 📝 Solved Example 2 — Different Arrival Times

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 5 |
| P2 | 1 | 3 |
| P3 | 2 | 8 |
| P4 | 3 | 6 |

```
FCFS order by arrival: P1, P2, P3, P4

t=0: P1 arrives, starts immediately
t=5: P1 done. P2 (arrived t=1) waiting → P2 starts
t=8: P2 done. P3 (arrived t=2) waiting → P3 starts
t=16: P3 done. P4 (arrived t=3) waiting → P4 starts
t=22: P4 done.

Gantt Chart:
|──P1──|─P2─|────────P3────────|──────P4──────|
0      5    8                 16             22

CT:  P1=5,  P2=8,  P3=16, P4=22
TAT: P1=5-0=5, P2=8-1=7, P3=16-2=14, P4=22-3=19
WT:  P1=5-5=0, P2=7-3=4, P3=14-8=6,  P4=19-6=13

Avg WT  = (0 + 4 + 6 + 13) / 4 = 23/4 = 5.75
Avg TAT = (5 + 7 + 14 + 19) / 4 = 45/4 = 11.25
```

### ⚠️ Key Properties
```
✅ Simple to implement (just a queue)
✅ No starvation (everyone eventually gets CPU)
❌ Convoy Effect — short processes wait behind long ones
❌ Poor average waiting time
❌ Not suitable for interactive systems
```

---

## 5. SJF — Shortest Job First (Non-Preemptive)

### 🧠 Concept
When CPU is free, pick the process with **shortest burst time** from the ready queue.
**Non-preemptive** — once started, runs to completion.
**Provably optimal** for minimum average waiting time among non-preemptive algorithms.

### 🔧 Algorithm
```
1. At each scheduling point (CPU becomes free):
2. Look at ALL processes currently in ready queue
3. Pick the one with smallest BT
4. Run it to completion
5. Repeat
(Tie in BT → use FCFS among tied processes)
```

### 📝 Solved Example 1 — All arrive at t=0

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 6 |
| P2 | 0 | 8 |
| P3 | 0 | 7 |
| P4 | 0 | 3 |

```
All available at t=0. Sort by BT: P4(3), P1(6), P3(7), P2(8)

Gantt Chart:
|─P4─|──P1──|───P3───|────P2────|
0    3      9       16         24

CT:  P4=3,  P1=9,  P3=16, P2=24
TAT: P4=3,  P1=9,  P3=16, P2=24  (all arrived at 0)
WT:  P4=0,  P1=3,  P3=9,  P2=16

Avg WT  = (0 + 3 + 9 + 16) / 4 = 7
Avg TAT = (3 + 9 + 16 + 24) / 4 = 13

Compare with FCFS (P1,P2,P3,P4 order):
Avg WT = (0+6+14+21)/4 = 10.25  ← SJF is better!
```

### 📝 Solved Example 2 — Different Arrival Times (CRITICAL — most exam questions)

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 7 |
| P2 | 2 | 4 |
| P3 | 4 | 1 |
| P4 | 5 | 4 |

```
Step through time carefully:

t=0: Ready queue = {P1}. Only P1 available → start P1 (runs to completion, non-preemptive)
t=7: P1 done. Ready queue = {P2(BT=4), P3(BT=1), P4(BT=4)}
     All arrived by t=7. Shortest BT = P3(1) → run P3
t=8: P3 done. Ready queue = {P2(BT=4), P4(BT=4)}
     Tie! Use FCFS among tied → P2 arrived at t=2, P4 at t=5 → run P2
t=12: P2 done. Ready queue = {P4} → run P4
t=16: P4 done.

Gantt Chart:
|───────P1───────|─P3─|────P2────|────P4────|
0                7    8         12         16

CT:  P1=7,  P2=12, P3=8,  P4=16
TAT: P1=7-0=7, P2=12-2=10, P3=8-4=4, P4=16-5=11
WT:  P1=7-7=0, P2=10-4=6,  P3=4-1=3, P4=11-4=7

Avg WT  = (0 + 6 + 3 + 7) / 4 = 16/4 = 4
Avg TAT = (7 + 10 + 4 + 11) / 4 = 32/4 = 8
```

### ⚠️ Key Properties
```
✅ Optimal average WT (non-preemptive)
✅ Better than FCFS
❌ STARVATION — long processes may never run if short ones keep arriving
❌ Not practical — we don't know burst time in advance (estimated using exponential averaging)
❌ Not preemptive — once running, can't be stopped
```

### 🔧 Burst Time Estimation (bonus concept)
```
Can't know exact burst time → estimate using past behavior:
τ(n+1) = α × t(n) + (1-α) × τ(n)

Where:
  τ(n+1) = predicted burst for next CPU burst
  t(n)   = actual last CPU burst
  τ(n)   = previous prediction
  α      = 0 to 1 (how much weight to give recent vs history)

α=0 → ignore recent, use only history
α=1 → use only most recent burst
α=0.5 → exponential moving average (common choice)
```

---

## 6. SRTF — Shortest Remaining Time First (Preemptive)

### 🧠 Concept
Preemptive version of SJF.
When a NEW process arrives, compare its BT with **remaining time** of current process.
If new process is shorter → **preempt** current process, run new one.
**Optimal overall** — minimizes average waiting time globally.

### 🔧 Algorithm
```
At every event (new arrival OR process completion):
1. Check remaining burst times of all processes in ready queue + running
2. Run the one with MINIMUM remaining time
3. If new arrival has shorter remaining time than current → preempt
```

### 📝 Solved Example — Classic SRTF

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 8 |
| P2 | 1 | 4 |
| P3 | 2 | 9 |
| P4 | 3 | 5 |

```
Simulate event by event:

t=0: Only P1 available → run P1. Remaining: P1=8
t=1: P2 arrives (BT=4). Compare: P1 remaining=7, P2=4.
     P2 < P1 → PREEMPT P1, run P2. Remaining: P1=7, P2=4
t=2: P3 arrives (BT=9). Compare: P1=7, P2=3, P3=9.
     P2 still shortest → continue P2. Remaining: P1=7, P2=3, P3=9
t=3: P4 arrives (BT=5). Compare: P1=7, P2=2, P3=9, P4=5.
     P2 still shortest → continue P2. Remaining: P1=7, P2=2, P3=9, P4=5
t=5: P2 finishes. Compare: P1=7, P3=9, P4=5.
     P4 shortest → run P4. Remaining: P1=7, P3=9, P4=5
t=10: P4 finishes. Compare: P1=7, P3=9.
      P1 shortest → run P1. Remaining: P1=7, P3=9
t=17: P1 finishes. Only P3 left → run P3.
t=26: P3 finishes.

Gantt Chart:
|─P1─|────P2────|─────P4─────|───────P1───────|──────────P3──────────|
0    1          5            10               17                     26

CT:  P1=17, P2=5,  P3=26, P4=10
TAT: P1=17-0=17, P2=5-1=4, P3=26-2=24, P4=10-3=7
WT:  P1=17-8=9,  P2=4-4=0, P3=24-9=15, P4=7-5=2

Avg WT  = (9 + 0 + 15 + 2) / 4 = 26/4 = 6.5
Avg TAT = (17 + 4 + 24 + 7) / 4 = 52/4 = 13
```

### ⚠️ Key Properties
```
✅ Optimal average WT (best possible)
✅ New short jobs get CPU quickly
❌ STARVATION — long processes can starve indefinitely
❌ High context switch overhead (many preemptions)
❌ Impractical (still need to know burst times)
❌ High overhead for tracking remaining times
```

---

## 7. Round Robin

### 🧠 Concept
Each process gets CPU for a fixed **time quantum (q)**.
After quantum expires → preempt, go to END of ready queue.
**Preemptive. Fair. Designed for time-sharing.**

### 🔧 Algorithm
```
1. Maintain a circular ready queue (FIFO)
2. Dispatcher picks first process, sets timer for q ms
3. If process finishes before q → voluntarily releases CPU
4. If timer fires → preempt, put at END of ready queue
5. Pick next process from front of queue
6. Repeat
```

### 🔧 Time Quantum Choice — Critical
```
q TOO SMALL:
  → Too many context switches → huge overhead
  → More time context switching than doing real work
  → Extreme: q=1 → n context switches per unit time

q TOO LARGE:
  → Degenerates to FCFS (each process runs to completion)
  → Poor response time for interactive users

OPTIMAL RULE OF THUMB:
  80% of CPU bursts should be shorter than q
  Typical values: 10ms - 100ms
  Context switch should be << q (usually 0.1ms - 1ms)
```

### 📝 Solved Example 1 — q=4

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 10 |
| P2 | 0 | 6 |
| P3 | 0 | 4 |

```
All arrive at t=0. Initial queue: [P1, P2, P3]

t=0:  Run P1 for q=4. P1 remaining=6. Queue after: [P2, P3, P1]
t=4:  Run P2 for q=4. P2 remaining=2. Queue after: [P3, P1, P2]
t=8:  Run P3 for q=4. P3 finishes (BT=4=q). Queue after: [P1, P2]
t=12: Run P1 for q=4. P1 remaining=2. Queue after: [P2, P1]
t=16: Run P2 for q=2 (only 2 remaining). P2 finishes. Queue after: [P1]
t=18: Run P1 for q=2 (only 2 remaining). P1 finishes.
t=20: Done.

Gantt Chart:
|──P1──|──P2──|──P3──|──P1──|─P2─|─P1─|
0      4      8     12     16   18   20

CT:  P1=20, P2=18, P3=12
TAT: P1=20, P2=18, P3=12  (all AT=0)
WT:  P1=20-10=10, P2=18-6=12, P3=12-4=8

Avg WT  = (10 + 12 + 8) / 3 = 10
Avg TAT = (20 + 18 + 12) / 3 = 16.67

Response Time (time to FIRST CPU):
  P1=0, P2=4, P3=8
  Avg RT = (0+4+8)/3 = 4  ← much better than SJF for response time!
```

### 📝 Solved Example 2 — q=2, Different Arrival Times (HARD)

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 5 |
| P2 | 1 | 3 |
| P3 | 2 | 1 |
| P4 | 3 | 2 |
| P5 | 4 | 3 |

```
⚠️ IMPORTANT RULE: When a process is preempted AND new arrivals join simultaneously,
new arrivals are added to queue BEFORE the preempted process (in most implementations)
OR AFTER — check which version your exam uses. Most common: preempted goes to END,
new arrivals join just before it (arrived during quantum).

Let's trace (new arrivals added to end, preempted goes to end):

t=0:  Queue=[P1]. Run P1 (q=2). P1 remaining=3.
      During [0,2]: P2 arrives at t=1 → added to queue.
t=2:  P1 preempted. Queue=[P2, P1]. P2 arrives at t=1 already in queue.
      Run P2 (q=2). P2 remaining=1.
      During [2,4]: P3 arrives t=2→added, P4 arrives t=3→added.
t=4:  P2 preempted. Queue=[P1, P3, P4, P2].
      P5 arrives at t=4 → added: Queue=[P1, P3, P4, P2, P5]
      Run P1 (q=2). P1 remaining=1.
t=6:  P1 preempted. Queue=[P3, P4, P2, P5, P1]
      Run P3 (q=1, only 1 remaining). P3 finishes at t=7.
t=7:  Queue=[P4, P2, P5, P1]. Run P4 (q=2). P4 finishes at t=9.
t=9:  Queue=[P2, P5, P1]. Run P2 (q=1 remaining). P2 finishes at t=10.
t=10: Queue=[P5, P1]. Run P5 (q=2). P5 remaining=1.
t=12: P5 preempted. Queue=[P1, P5]. Run P1 (q=1 remaining). P1 finishes at t=13.
t=13: Queue=[P5]. Run P5 (q=1 remaining). P5 finishes at t=14.

Gantt Chart:
|─P1─|─P2─|─P1─|P3|─P4─|P2|─P5─|P1|P5|
0    2    4    6  7    9 10   12 13 14

CT:  P1=13, P2=10, P3=7, P4=9,  P5=14
TAT: P1=13, P2=9,  P3=5, P4=6,  P5=10
WT:  P1=8,  P2=6,  P3=4, P4=4,  P5=7

Avg WT  = (8+6+4+4+7)/5 = 29/5 = 5.8
Avg TAT = (13+9+5+6+10)/5 = 43/5 = 8.6
```

### ⚠️ Key Properties
```
✅ No starvation — every process gets CPU within n×q time
✅ Good response time — best for interactive systems
✅ Fair — equal CPU share for equal priority processes
❌ Higher average TAT than SJF (due to context switches)
❌ If q too large → degrades to FCFS
❌ If q too small → too much context switch overhead
```

---

## 8. Priority Scheduling

### 🧠 Concept
Each process has a **priority number**. CPU given to highest priority process.
**Convention: Lower number = Higher priority** (priority 1 > priority 5).
Can be **preemptive** or **non-preemptive**.

### 🔧 Priority Types
```
STATIC PRIORITY:  Fixed at process creation. Doesn't change.
DYNAMIC PRIORITY: Changes over time.
                  Aging: priority increases the longer a process waits.
                  Prevents starvation.

INTERNAL PRIORITY: Set by OS based on measurable quantities
                   (memory requirements, # open files, I/O to CPU ratio)
EXTERNAL PRIORITY: Set by users/administrators
                   (paid more → higher priority, niceness value in Linux)
```

### 📝 Solved Example — Non-Preemptive Priority

| Process | AT | BT | Priority |
|---|---|---|---|
| P1 | 0 | 10 | 3 |
| P2 | 0 | 1  | 1 |
| P3 | 0 | 2  | 4 |
| P4 | 0 | 1  | 5 |
| P5 | 0 | 5  | 2 |

```
All arrive at t=0. Sort by priority (lower number = higher priority):
P2(1), P5(2), P1(3), P3(4), P4(5)

Gantt Chart:
|P2|──P5──|────────P1────────|─P3─|P4|
0  1      6                 16   18  19

CT:  P2=1,  P5=6,  P1=16, P3=18, P4=19
TAT: P2=1,  P5=6,  P1=16, P3=18, P4=19  (all AT=0)
WT:  P2=0,  P5=1,  P1=6,  P3=16, P4=18

Avg WT  = (0+1+6+16+18)/5 = 41/5 = 8.2
Avg TAT = (1+6+16+18+19)/5 = 60/5 = 12

⚠️ STARVATION: P3 and P4 wait a long time. If P2-priority processes
keep arriving, P4 (priority 5) may NEVER run.
```

### 📝 Solved Example — Preemptive Priority with Aging

| Process | AT | BT | Priority |
|---|---|---|---|
| P1 | 0 | 5 | 2 |
| P2 | 1 | 3 | 1 |
| P3 | 4 | 4 | 3 |

```
Preemptive — new arrival preempts if higher priority.

t=0: P1 arrives (priority 2) → run P1. Remaining: P1=5
t=1: P2 arrives (priority 1). P1 has priority 2, P2 has priority 1.
     Priority 1 > Priority 2 (lower number = higher) → PREEMPT P1.
     Run P2. Remaining: P1=4, P2=3
t=4: P2 finishes. P3 arrives at t=4. Queue = {P1(pri=2), P3(pri=3)}.
     P1 has higher priority → run P1. Remaining: P1=4, P3=4
t=8: P1 finishes. Run P3.
t=12: P3 finishes.

Gantt Chart:
|─P1─|────P2────|────P1────|────P3────|
0    1          4          8         12

CT:  P1=8,  P2=4,  P3=12
TAT: P1=8,  P2=3,  P3=8
WT:  P1=3,  P2=0,  P3=4

Avg WT  = (3+0+4)/3 = 7/3 = 2.33
Avg TAT = (8+3+8)/3 = 19/3 = 6.33
```

### 🔧 Aging — Solving Starvation
```
Every T time units that a process waits in the ready queue:
Increase its priority by 1 (decrease priority number by 1).

Example:
  Process P with priority 127 (very low).
  Every 15 minutes in ready queue → priority decreases by 1.
  After 127×15 minutes ≈ 32 hours → priority = 0 (highest).
  Guaranteed to eventually run — NO starvation.
```

---

## 9. MLFQ — Multilevel Feedback Queue

### 🧠 Concept
Most realistic, most complex, used in real OS (Linux CFS, Windows).
Multiple queues with different priorities and different time quantums.
Processes move BETWEEN queues based on behavior.

### 🔧 Structure
```
Queue 0 (highest priority, q=8ms)   ← NEW processes start here
     ↓ (demoted if uses full quantum)
Queue 1 (medium priority, q=16ms)
     ↓ (demoted if uses full quantum)
Queue 2 (lowest priority, FCFS)     ← CPU-bound long jobs end up here
```

### 🔧 Rules
```
1. Higher priority queue → always runs first
2. Process in Q0 preempts process in Q1 or Q2
3. If process uses its FULL quantum → demote to next lower queue
4. If process gives up CPU before quantum (I/O) → stays in SAME queue
   (I/O-bound = interactive = should stay high priority)
5. After some time T → move ALL processes to Q0 (aging, prevent starvation)
```

### 🔧 Why It Works
```
Interactive programs (editors, shells):
  → Short CPU bursts, frequent I/O
  → Never use full quantum → stay in Q0
  → Get fast response time ✓

CPU-bound programs (video encoding, simulations):
  → Long CPU bursts, rare I/O
  → Use full quantum repeatedly → sink to Q2
  → Get low priority but no starvation (eventually get CPU in FCFS) ✓

NEW programs:
  → Start in Q0 (treated as interactive initially)
  → Behavior determines where they end up ✓
```

---

## 10. Numericals — Full Solved Sets

### 📝 Problem Set A — Mixed Algorithm Comparison

**Given processes:**

| Process | AT | BT | Priority |
|---|---|---|---|
| P1 | 0 | 8 | 3 |
| P2 | 1 | 4 | 1 |
| P3 | 2 | 9 | 4 |
| P4 | 3 | 5 | 2 |

**Solve for FCFS, SJF (non-preemptive), SRTF, RR(q=2), Priority (non-preemptive).**

---

**A1. FCFS:**
```
Order by AT: P1→P2→P3→P4

|────────P1────────|────P2────|─────────P3─────────|─────P4─────|
0                  8         12                   21           26

CT:  P1=8,  P2=12, P3=21, P4=26
TAT: P1=8,  P2=11, P3=19, P4=23
WT:  P1=0,  P2=7,  P3=10, P4=18

Avg WT  = (0+7+10+18)/4 = 35/4 = 8.75
Avg TAT = (8+11+19+23)/4 = 61/4 = 15.25
```

**A2. SJF (Non-Preemptive):**
```
t=0: Only P1 available → run P1 (BT=8, non-preemptive)
t=8: P2(BT=4), P3(BT=9), P4(BT=5) all in queue.
     Shortest = P2(4) → run P2
t=12: P3(9), P4(5) in queue. Shortest = P4(5) → run P4
t=17: Only P3 → run P3
t=26: Done.

|────────P1────────|────P2────|─────P4─────|─────────P3─────────|
0                  8         12           17                   26

CT:  P1=8,  P2=12, P3=26, P4=17
TAT: P1=8,  P2=11, P3=24, P4=14
WT:  P1=0,  P2=7,  P3=15, P4=9

Avg WT  = (0+7+15+9)/4 = 31/4 = 7.75
Avg TAT = (8+11+24+14)/4 = 57/4 = 14.25
```

**A3. SRTF:**
```
t=0: P1(8) → run P1. Remaining: P1=8
t=1: P2(4) arrives. P1 remaining=7. 4<7 → PREEMPT. Run P2. Rem: P1=7,P2=4
t=2: P3(9) arrives. P2 remaining=3. 3<7,9 → continue P2. Rem:P1=7,P2=3,P3=9
t=3: P4(5) arrives. P2 remaining=2. 2<7,9,5 → continue P2. Rem:P1=7,P2=2,P3=9,P4=5
t=5: P2 finishes. Min remaining: P4(5)<P1(7)<P3(9) → run P4.
t=10: P4 finishes. Min: P1(7)<P3(9) → run P1.
t=17: P1 finishes. Run P3.
t=26: P3 finishes.

|P1|────P2────|─────P4─────|───────P1───────|─────────P3─────────|
0  1          5            10               17                   26

CT:  P1=17, P2=5,  P3=26, P4=10
TAT: P1=17, P2=4,  P3=24, P4=7
WT:  P1=9,  P2=0,  P3=15, P4=2

Avg WT  = (9+0+15+2)/4 = 26/4 = 6.5  ← BEST among all!
Avg TAT = (17+4+24+7)/4 = 52/4 = 13
```

**A4. Round Robin (q=2):**
```
t=0:  Queue=[P1]. Run P1(q=2). Rem=6.
      P2 arrives t=1 → joins queue during P1's quantum.
t=2:  Queue=[P2,P1]. Run P2(q=2). Rem=2.
      P3 arrives t=2, P4 arrives t=3 → join queue.
t=4:  Queue=[P1,P3,P4,P2]. Run P1(q=2). Rem=4.
t=6:  Queue=[P3,P4,P2,P1]. Run P3(q=2). Rem=7.
t=8:  Queue=[P4,P2,P1,P3]. Run P4(q=2). Rem=3.
t=10: Queue=[P2,P1,P3,P4]. Run P2(q=2 but only 2 rem). P2 DONE at t=12.
t=12: Queue=[P1,P3,P4]. Run P1(q=2). Rem=2.
t=14: Queue=[P3,P4,P1]. Run P3(q=2). Rem=5.
t=16: Queue=[P4,P1,P3]. Run P4(q=2 but only 3 rem). Rem=1.
t=18: Queue=[P1,P3,P4]. Run P1(q=2 but 2 rem). P1 DONE at t=20.
t=20: Queue=[P3,P4]. Run P3(q=2). Rem=3.
t=22: Queue=[P4,P3]. Run P4(q=1 rem). P4 DONE at t=23.
t=23: Queue=[P3]. Run P3(q=2). Rem=1.
t=25: Queue=[P3]. Run P3(q=1 rem). P3 DONE at t=26.

CT:  P1=20, P2=12, P3=26, P4=23
TAT: P1=20, P2=11, P3=24, P4=20
WT:  P1=12, P2=7,  P3=15, P4=15

Avg WT  = (12+7+15+15)/4 = 49/4 = 12.25
Avg TAT = (20+11+24+20)/4 = 75/4 = 18.75
```

**A5. Priority (Non-Preemptive, lower=higher):**
```
t=0: Only P1(pri=3) → run P1.
t=8: Queue = {P2(pri=1), P3(pri=4), P4(pri=2)}. Highest = P2(1) → run P2.
t=12: Queue = {P3(pri=4), P4(pri=2)}. Highest = P4(2) → run P4.
t=17: Only P3 → run P3.
t=26: Done.

|────────P1────────|────P2────|─────P4─────|─────────P3─────────|
0                  8         12           17                   26

Same as SJF in this case! (coincidence of priorities matching burst order)
CT:  P1=8,  P2=12, P3=26, P4=17
TAT: P1=8,  P2=11, P3=24, P4=14
WT:  P1=0,  P2=7,  P3=15, P4=9

Avg WT  = 7.75
```

### 📊 Final Comparison Table (Problem Set A)

| Algorithm | Avg WT | Avg TAT | Notes |
|---|---|---|---|
| FCFS | 8.75 | 15.25 | Worst WT |
| SJF | 7.75 | 14.25 | Better |
| SRTF | **6.50** | **13.00** | **Best WT** |
| RR(q=2) | 12.25 | 18.75 | Worst (but best response time) |
| Priority | 7.75 | 14.25 | Same as SJF here |

---

### 📝 Problem Set B — CPU Utilization & Throughput

**Problem:**
3 processes with BT = {4, 8, 12}. All arrive at t=0.
Calculate throughput and CPU utilization for FCFS.
(Assume 1ms overhead per context switch, 1 context switch at start)

**Solution:**
```
Total CPU time = 4 + 8 + 12 = 24ms
Context switch overhead = 3 switches × 1ms = 3ms (one before each process)
Total time = 24 + 3 = 27ms

Throughput = 3 processes / 27ms = 0.111 processes/ms = 111 processes/second

CPU Utilization = 24 / 27 × 100% = 88.9%
Idle time = 0 (no gaps, FCFS with all arrived at t=0)
```

---

### 📝 Problem Set C — Response Time Focus (Round Robin)

**Problem:** Why is Round Robin better for response time even though TAT is worse?

```
Given: P1(BT=100), P2(BT=1), P3(BT=1). All arrive at t=0. q=1.

FCFS:
  P1 runs: P1 done at t=100, P2 done at t=101, P3 done at t=102
  Response time: P1=0, P2=100, P3=101
  Avg Response Time = (0+100+101)/3 = 67ms ← P2 and P3 wait forever

Round Robin (q=1):
  t=0: P1 runs 1ms, preempted
  t=1: P2 runs 1ms, DONE (CT=2)
  t=2: P3 runs 1ms, DONE (CT=3)
  t=3: P1 runs 1ms, ... continues until t=102

  Response time: P1=0, P2=1, P3=2
  Avg Response Time = (0+1+2)/3 = 1ms ← excellent!

Conclusion: RR ensures no process waits more than (n-1)×q before first response.
This is why ALL interactive OS use Round Robin (or derivatives).
```

---

### 📝 Problem Set D — Gantt Chart from Scratch (Exam Style)

**Problem:** Given the following, draw Gantt chart and compute all metrics for SJF preemptive (SRTF):

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 6 |
| P2 | 0 | 8 |
| P3 | 3 | 4 |
| P4 | 5 | 2 |
| P5 | 6 | 1 |

```
t=0: Available={P1(6),P2(8)}. Min=P1→run P1. Rem:P1=6,P2=8
t=3: P3(4) arrives. Rem:P1=3,P2=8,P3=4. Min=P1(3)→continue P1.
t=5: P4(2) arrives. Rem:P1=1,P2=8,P3=4,P4=2. Min=P1(1)→continue P1.
t=6: P1 done. P5(1) arrives. Rem:P2=8,P3=4,P4=2,P5=1. Min=P5(1)→run P5.
t=7: P5 done. Rem:P2=8,P3=4,P4=2. Min=P4(2)→run P4.
t=9: P4 done. Rem:P2=8,P3=4. Min=P3(4)→run P3.
t=13: P3 done. Run P2.
t=21: P2 done.

Gantt:
|──────P1──────|P5|─P4─|────P3────|────────P2────────|
0              6  7    9         13                  21

CT:  P1=6, P2=21, P3=13, P4=9,  P5=7
TAT: P1=6, P2=21, P3=10, P4=4,  P5=1
WT:  P1=0, P2=13, P3=6,  P4=2,  P5=0

Avg WT  = (0+13+6+2+0)/5 = 21/5 = 4.2
Avg TAT = (6+21+10+4+1)/5 = 42/5 = 8.4
```

---

## 11. Algorithm Comparison & Decision Guide

### 📊 Master Comparison Table

| Algorithm | Preemptive | Starvation | Avg WT | Response Time | Overhead | Use Case |
|---|---|---|---|---|---|---|
| FCFS | ❌ No | ❌ No | Poor | Poor | Low | Batch systems |
| SJF | ❌ No | ✅ Yes | Best (NP) | Medium | Low | Batch, known bursts |
| SRTF | ✅ Yes | ✅ Yes | Best overall | Good | High | Theoretical optimal |
| Round Robin | ✅ Yes | ❌ No | Medium | Best | Medium | Time-sharing, interactive |
| Priority | Both | ✅ Yes | Depends | Depends | Low-Med | Real-time, mixed |
| MLFQ | ✅ Yes | ❌ No* | Good | Very Good | High | General-purpose OS |

*With aging implemented

### 🔧 Decision Flowchart
```
Need real-time guarantees?    → Real-Time Scheduling (EDF, RMS)
Need fairness above all?      → Round Robin
Need min average wait time?   → SJF/SRTF (if burst times known)
Need good response time?      → Round Robin or MLFQ
General purpose OS?           → MLFQ
Simple batch processing?      → FCFS
Processes have priorities?    → Priority Scheduling + Aging
```

---

## 12. MCQ Traps & Exam Q&A

### ⚠️ THE TRAPS

---

**TRAP 1: "SJF always gives minimum average waiting time"**
❌ Partially wrong. SJF is optimal among **non-preemptive** algorithms.
SRTF (preemptive SJF) gives even lower average WT.
Full statement: "SJF is optimal among non-preemptive scheduling algorithms."

---

**TRAP 2: "Round Robin has the best average TAT"**
❌ WRONG. RR has WORST average TAT (due to frequent context switches and multiplexing).
RR has BEST average response time (first CPU allocation).
These are DIFFERENT metrics — exams love to swap them.

---

**TRAP 3: "Increasing time quantum always improves RR performance"**
❌ Depends on the metric.
Larger q → TAT approaches FCFS (worse), context switch overhead decreases.
There's a sweet spot. Not monotonically better.

---

**TRAP 4: "FCFS can cause starvation"**
❌ WRONG. FCFS cannot cause starvation. Every process eventually gets CPU
in order of arrival. No process can be bypassed.
Starvation can happen in: SJF, SRTF, Priority Scheduling.

---

**TRAP 5: Confusing Waiting Time and Response Time in RR**
```
In FCFS/SJF/Priority: WT = TAT - BT (time in ready queue total)
In Round Robin: WT ≠ Response Time

Response Time = time to FIRST CPU allocation - AT (one-time)
Waiting Time  = total time spent in ready queue (sum of all waits between bursts)

Example: P1(BT=10), q=2, P1 first gets CPU at t=0.
  Response Time = 0 (got CPU immediately)
  But WT = total ready queue time = 10 (WT = TAT - BT)
```

---

**TRAP 6: "SRTF and SJF give same result when all arrive at t=0"**
✅ TRUE! When all arrive simultaneously, no preemption ever occurs in SRTF
(no new arrivals to trigger preemption). Both degenerate to same schedule.

---

**TRAP 7: "Lower priority number = lower priority"**
❌ Convention matters. Most textbooks: **lower number = higher priority**.
But some books/exams use higher number = higher priority.
READ THE QUESTION — it usually says "priority 1 is highest" or similar.

---

**TRAP 8: Forgetting idle CPU time in Gantt charts**
```
If no process is available when CPU becomes free → CPU is IDLE.
Must show this in Gantt chart!

Example: P1(AT=3, BT=2). What happens from t=0 to t=3?
|──IDLE──|──P1──|
0        3      5

CPU Utilization = 2/5 = 40% (NOT 100%!)
```

---

**TRAP 9: "Preemption happens at every clock tick in SRTF"**
❌ WRONG. Preemption in SRTF only happens when a NEW process arrives.
Check remaining times only at arrival events, not every clock tick.

---

**TRAP 10: Turnaround Time for process that hasn't started yet**
```
If a process arrives at t=5 and finishes at t=10:
TAT = CT - AT = 10 - 5 = 5 (NOT 10)

Students often forget to subtract Arrival Time.
WT = TAT - BT (never WT = CT - BT directly, unless AT=0)
```

---

### ⚡ Quick Fire Q&A

| Question | Answer |
|---|---|
| Which algorithm is optimal for average WT? | SRTF (preemptive), SJF (non-preemptive) |
| Which algorithm has no starvation? | FCFS, Round Robin |
| Which algorithms can cause starvation? | SJF, SRTF, Priority Scheduling |
| What is the convoy effect? | Short processes wait behind a long process in FCFS |
| What is dispatch latency? | Time dispatcher takes to stop one process and start another |
| What is aging? | Gradually increasing priority of waiting processes to prevent starvation |
| RR with q=∞ becomes which algorithm? | FCFS |
| RR with q=1 problem? | Too many context switches, too much overhead |
| Which scheduling is used in real OS? | MLFQ (Linux CFS, Windows scheduler) |
| Formula for TAT? | CT - AT |
| Formula for WT? | TAT - BT |
| Formula for Response Time? | First_CPU_time - AT |
| If all processes arrive at t=0, SJF vs SRTF? | Identical results |
| What does MLFQ stand for? | Multilevel Feedback Queue |
| MLFQ demotes on? | Using full time quantum (CPU-bound behavior) |
| MLFQ promotes on? | I/O before quantum expires (interactive behavior) |
| What prevents starvation in MLFQ? | Periodic promotion of all processes to highest queue |
| Throughput formula? | # processes / total time |
| Best algorithm for interactive systems? | Round Robin or MLFQ |
| Can Priority Scheduling be non-preemptive? | YES — runs current process to completion even if higher priority arrives |

---

### 🔥 Likely CoreTex Numerical Question Types

```
Type 1: Given AT, BT for 4-5 processes → draw Gantt for 2 algorithms → compare Avg WT
Type 2: Given processes → identify which algorithm produces given Gantt chart
Type 3: Calculate CPU utilization given idle periods
Type 4: Given RR with specific q → trace execution, find when each process completes
Type 5: Find optimal time quantum for Round Robin given burst time distribution
Type 6: Prove or disprove: "SRTF always outperforms SJF" (True — but how much?)
```

---

### 📋 Formula Cheatsheet

```
TAT  = CT - AT
WT   = TAT - BT  =  CT - AT - BT
RT   = First_CPU_allocation - AT

Avg WT  = Σ(WTᵢ) / n
Avg TAT = Σ(TATᵢ) / n

CPU Utilization = (Total - Idle) / Total × 100%
Throughput      = n / Total_completion_time

Round Robin — max wait before first CPU = (n-1) × q
  where n = number of processes in queue

Aging — priority after waiting time t = initial_priority + (t / aging_interval)
```

---