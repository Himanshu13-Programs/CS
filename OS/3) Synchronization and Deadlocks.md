# 📘 OS Chapter 3 — Synchronization & Deadlocks 
---

## 📌 Table of Contents
1. [The Concurrency Problem](#1-the-concurrency-problem)
2. [Critical Section Problem](#2-critical-section-problem)
3. [Software Solutions — Peterson's Algorithm](#3-software-solutions--petersons-algorithm)
4. [Hardware Solutions — Atomic Instructions](#4-hardware-solutions--atomic-instructions)
5. [Mutex Locks](#5-mutex-locks)
6. [Semaphores — Deep Dive](#6-semaphores--deep-dive)
7. [Classic Synchronization Problems](#7-classic-synchronization-problems)
8. [Monitors](#8-monitors)
9. [Deadlocks — Complete Coverage](#9-deadlocks--complete-coverage)
10. [Banker's Algorithm — Full Numericals](#10-bankers-algorithm--full-numericals)
11. [Resource Allocation Graph — Numericals](#11-resource-allocation-graph--numericals)
12. [MCQ Traps & Exam Q&A](#12-mcq-traps--exam-qa)

---

## 1. The Concurrency Problem

### 🧠 Why Does This Problem Exist?
Modern OS runs multiple processes/threads simultaneously sharing memory.
When two or more touch the same data at the same time → **undefined behavior**.

### 🔧 Race Condition — The Root Cause

```
Shared variable: int counter = 5;

Thread 1: counter++        Thread 2: counter--

counter++ in machine code:    counter-- in machine code:
  LOAD  R1, counter             LOAD  R2, counter
  ADD   R1, 1                   SUB   R2, 1
  STORE counter, R1             STORE counter, R2

Possible interleaving:
  T1: LOAD R1 = 5
  T2: LOAD R2 = 5
  T1: ADD  R1 = 6
  T2: SUB  R2 = 4
  T1: STORE counter = 6
  T2: STORE counter = 4   ← FINAL VALUE = 4 (WRONG! Should be 5)

Another interleaving → counter = 6 (also wrong)
Only correct if they don't interleave → counter = 5

RACE CONDITION: outcome depends on the ORDER of execution.
Non-deterministic. Extremely hard to debug (happens rarely, not reproducible).
```

### 🔧 What Is a Race Condition Formally?
```
A race condition occurs when:
1. Two or more processes access shared data concurrently
2. At least one access is a WRITE
3. The result depends on the relative order of execution

Read-Read → No race condition (reads don't modify data)
Read-Write or Write-Write → Race condition possible
```

---

## 2. Critical Section Problem

### 🧠 Definition
```
CRITICAL SECTION: The segment of code that accesses shared resources
                  (shared variables, files, hardware devices).

Only ONE process should execute its critical section at any time.
```

### 🔧 Structure of a Process
```c
do {
    // ENTRY SECTION
    // (request permission to enter CS)

        // CRITICAL SECTION
        // (access shared resource)

    // EXIT SECTION
    // (signal that CS is released)

    // REMAINDER SECTION
    // (rest of the program, unrelated to shared resource)

} while (true);
```

### 🔧 Three Requirements for a Valid Solution

#### 1. Mutual Exclusion
```
If process Pᵢ is executing in its CS → NO OTHER process can be in its CS.
This is the PRIMARY requirement. Without this, race conditions occur.
```

#### 2. Progress
```
If NO process is in its CS AND some processes want to enter →
the decision of who enters next cannot be postponed indefinitely.

In other words: if CS is free and someone wants in → someone MUST get in.
The system cannot deadlock at the entry section.
```

#### 3. Bounded Waiting
```
There must be a BOUND on the number of times other processes
can enter their CS after a process has requested entry.

In other words: No starvation. Every requesting process
eventually gets to enter.
```

---

## 3. Software Solutions — Peterson's Algorithm

### 🧠 For Two Processes Only
Classic elegant solution. Uses two shared variables.

### 🔧 Shared Variables
```c
int turn;           // whose turn it is (0 or 1)
bool flag[2];       // flag[i] = true means Pᵢ WANTS to enter CS
```

### 🔧 Algorithm for Process Pᵢ (i=0 or 1, j=1-i)
```c
// ENTRY SECTION
flag[i] = true;         // "I want to enter"
turn = j;               // "But you go first" (polite!)
while (flag[j] && turn == j);  // wait if j wants AND it's j's turn

    // CRITICAL SECTION
    // ... do stuff with shared resource ...

// EXIT SECTION
flag[i] = false;        // "I'm done, others can enter"

// REMAINDER SECTION
```

### 🔧 Why It Works — Proving All Three Properties
```
MUTUAL EXCLUSION:
  For both P0 and P1 to be in CS simultaneously:
  P0 needs: flag[1]=false OR turn=0
  P1 needs: flag[0]=false OR turn=1
  turn can't be BOTH 0 and 1 simultaneously.
  If both flags are true → turn decides → only one enters. ✓

PROGRESS:
  If P1 is not interested (flag[1]=false) → P0's while loop exits immediately.
  P0 doesn't wait unnecessarily. ✓

BOUNDED WAITING:
  After P0 sets turn=j=1 (gives turn to P1):
  P1 gets to go, sets turn=0 when done → P0 enters next.
  P0 waits at most 1 turn of P1. ✓
```

### ⚠️ Limitation
```
Works for ONLY 2 processes.
Assumes LOAD and STORE are atomic (true on modern hardware).
May not work on modern CPUs with instruction reordering (need memory barriers).
```

---

## 4. Hardware Solutions — Atomic Instructions

### 🧠 Why Hardware?
Software solutions are complex and limited to 2 processes.
Modern hardware provides **atomic instructions** — execute completely without interruption.

### 🔧 Test-And-Set (TAS)
```c
// Hardware implementation (atomic, cannot be interrupted):
bool TestAndSet(bool *target) {
    bool rv = *target;    // read old value
    *target = true;       // set to true
    return rv;            // return old value
}
// All three steps happen atomically — no interleaving possible

// Using TAS for mutual exclusion:
bool lock = false;   // shared lock variable

// Process Pᵢ:
while (TestAndSet(&lock));  // spin until we get lock (old value was false)
    // CRITICAL SECTION
lock = false;               // release lock
```

```
How it works:
  Lock = false → TAS returns false (old value) AND sets lock=true.
  while(false) exits → process enters CS. ✓
  
  Another process calls TAS when lock=true → returns true (old value).
  while(true) spins → cannot enter CS. ✓
  
  After CS, lock=false → waiting process gets in on next TAS call. ✓
```

### 🔧 Compare-And-Swap (CAS)
```c
// Hardware implementation (atomic):
int CompareAndSwap(int *value, int expected, int new_value) {
    int temp = *value;
    if (*value == expected)
        *value = new_value;
    return temp;
}

// Using CAS for mutex:
int lock = 0;

while (CompareAndSwap(&lock, 0, 1) != 0);  // spin until lock=0, then set to 1
    // CRITICAL SECTION
lock = 0;
```

### ⚠️ Problem with Hardware Solutions — Busy Waiting (Spinlock)
```
while (TestAndSet(&lock));  ← process keeps checking in a loop

SPINLOCK: wastes CPU cycles while waiting.
Good for: multicore, very short CS (context switch overhead > wait time)
Bad for:  single core (waiting process blocks the one that has the lock!)
          long CS (wastes lots of CPU)

Solution → Mutex with blocking (process sleeps instead of spinning)
```

---

## 5. Mutex Locks

### 🧠 Higher-Level Abstraction
OS provides mutex as a high-level tool built on hardware atomics.
Waiting process **sleeps** (goes to blocked state) instead of spinning.

### 🔧 Operations
```c
acquire() {
    while (!available);     // busy wait (internally)
    available = false;
}

release() {
    available = true;
}

// Usage:
mutex lock;

acquire(lock);
    // CRITICAL SECTION
release(lock);
```

### 🔧 Mutex Properties

```
OWNERSHIP:    Only the thread that locked the mutex can unlock it.
              (Key difference from semaphore)

BINARY:       Either locked (0) or unlocked (1). No counting.

BLOCKING:     Thread that can't acquire → goes to BLOCKED state.
              OS scheduler picks another thread. No CPU wasted.

RECURSIVE MUTEX: Same thread can lock multiple times without deadlock.
                 Must unlock same number of times.
                 Useful for recursive functions that use shared resources.
```

### 🔧 Mutex vs Spinlock Decision
```
Use SPINLOCK when:
  ✓ CS is very short (< context switch time)
  ✓ Running on multicore (another core can release the lock)
  ✓ OS kernel code (can't sleep in interrupt handler)

Use MUTEX (blocking) when:
  ✓ CS is long
  ✓ Single core system
  ✓ Want to save CPU for other work
```

---

## 6. Semaphores — Deep Dive

### 🧠 What is a Semaphore?
An integer variable S with only TWO atomic operations allowed:
`wait()` (P, down, acquire) and `signal()` (V, up, release).

### 🔧 Operations — Formal Definition
```c
wait(S) {           // aka P(S), down(S), acquire(S)
    while (S <= 0); // busy wait (basic version)
    S--;
}

signal(S) {         // aka V(S), up(S), release(S)
    S++;
}

// S can be any non-negative integer (unlike mutex which is only 0/1)
// S initialized to number of available instances of resource
```

### 🔧 Blocking Semaphore (Better Version)
```c
// Instead of busy waiting, block the process:
wait(S) {
    S--;
    if (S < 0) {
        add process to S's waiting queue;
        block();    // process goes to sleep (blocked state)
    }
}

signal(S) {
    S++;
    if (S <= 0) {   // there are processes waiting
        remove process P from S's waiting queue;
        wakeup(P);  // move P to ready queue
    }
}

// When S < 0: |S| = number of processes waiting on this semaphore
// When S >= 0: S = number of available resources
```

### 🔧 Types of Semaphores

```
BINARY SEMAPHORE (S initialized to 1):
  S=1 → resource available, S=0 → resource taken
  Behaves like mutex BUT has NO ownership constraint
  Any process can call signal() — dangerous but flexible

COUNTING SEMAPHORE (S initialized to N):
  S=N → N resources available
  S=0 → all N resources taken, next wait() will block
  S<0 → |S| processes are waiting
  Used to control access to a pool of N identical resources
```

### 🔧 Semaphore for Ordering/Signaling
```c
// Ensure that B executes AFTER A completes:
Semaphore sync = 0;    // initialized to 0!

// Process 1:           // Process 2:
execute A;              wait(sync);    // blocks (sync=0)
signal(sync);           execute B;     // only runs after A

// signal() increments sync to 1 → wakes up Process 2
// Guarantees B happens after A regardless of scheduling
```

### 🔧 Mutex vs Semaphore — Master Comparison

| Property | Mutex | Binary Semaphore | Counting Semaphore |
|---|---|---|---|
| **Value range** | 0 or 1 | 0 or 1 | 0 to N |
| **Ownership** | YES — only locker unlocks | NO — anyone can signal | NO |
| **Primary use** | Mutual exclusion | Mutual exclusion / Signaling | Resource counting |
| **Initialized to** | 1 (unlocked) | 0 (for signaling) or 1 (for mutex) | N (# resources) |
| **Starvation** | Possible | Possible | Possible |
| **Deadlock risk** | If owner dies | Lower | Lower |

---

## 7. Classic Synchronization Problems

### 7.1 Producer-Consumer (Bounded Buffer)

#### 🧠 Problem
```
Producer creates items, puts them in buffer.
Consumer takes items from buffer, processes them.
Buffer has fixed capacity N.

Constraints:
  - Producer must wait if buffer FULL
  - Consumer must wait if buffer EMPTY
  - Producer and Consumer must not access buffer simultaneously
```

#### 🔧 Solution
```c
// Shared variables:
int buffer[N];           // circular buffer
int in = 0, out = 0;    // next empty slot, next full slot

// Semaphores:
Semaphore mutex = 1;    // mutual exclusion for buffer access
Semaphore empty = N;    // counts EMPTY slots (initialized to N — all empty)
Semaphore full  = 0;    // counts FULL slots  (initialized to 0 — none full)

// PRODUCER:                    // CONSUMER:
do {                            do {
    produce item;                   wait(full);   // wait if no items
    wait(empty);  // wait if full   wait(mutex);  // get exclusive access
    wait(mutex);  // get access     item = buffer[out];
    buffer[in] = item;              out = (out+1) % N;
    in = (in+1) % N;                signal(mutex);
    signal(mutex);                  signal(empty); // one more empty slot
    signal(full); // one more item  consume item;
} while(true);                  } while(true);
```

#### ⚠️ Critical Order — DON'T SWAP wait() calls
```
WRONG (causes deadlock):
    wait(mutex);   ← acquire buffer lock FIRST
    wait(empty);   ← then wait for space

If buffer is FULL:
  Producer acquires mutex, then blocks on empty.
  Consumer tries mutex → BLOCKED (mutex taken).
  Producer waits for Consumer to signal empty.
  Consumer waits for Producer to release mutex.
  DEADLOCK! ✗

CORRECT: Always wait on resource semaphore (empty/full) BEFORE mutex.
```

---

### 7.2 Readers-Writers Problem

#### 🧠 Problem
```
Database shared between readers and writers.
Rules:
  - Multiple readers CAN read simultaneously (no conflict)
  - Writer needs EXCLUSIVE access (no readers or writers while writing)
  - If writer is writing → no readers or writers allowed
```

#### 🔧 First Readers-Writers Solution (Readers Priority)
```c
// Shared:
Semaphore mutex = 1;      // protects read_count
Semaphore db    = 1;      // controls database access
int read_count  = 0;      // number of active readers

// READER:                        // WRITER:
wait(mutex);                      wait(db);        // exclusive access
  read_count++;                       // write to database
  if (read_count == 1)            signal(db);
      wait(db);  // first reader blocks writers
signal(mutex);

// read the database

wait(mutex);
  read_count--;
  if (read_count == 0)
      signal(db);  // last reader releases db for writers
signal(mutex);
```

```
Analysis:
  First reader: acquires db → blocks writers
  Subsequent readers: increment count, read freely
  Last reader: releases db → writer can proceed
  
  PROBLEM: If readers keep arriving → writer STARVES (never gets db)
  This is "Readers Priority" — readers always preferred.

Second Readers-Writers: Writers Priority (writers don't starve)
  → More complex, writers get priority once they're waiting
  → Readers may starve in this version
```

---

### 7.3 Dining Philosophers Problem

#### 🧠 Problem Setup
```
5 philosophers sit at round table.
5 forks (chopsticks), one between each pair.
Philosopher does: THINK → pick up left fork → pick up right fork → EAT → put down both

Problem: If ALL philosophers pick up their LEFT fork simultaneously:
  Each holds 1 fork, needs 1 more, but it's held by neighbor.
  CIRCULAR WAIT → DEADLOCK!
```

#### 🔧 Naive (Wrong) Solution
```c
Semaphore fork[5] = {1,1,1,1,1};  // each fork is a semaphore

// Philosopher i:
wait(fork[i]);            // pick up left fork
wait(fork[(i+1)%5]);      // pick up right fork
// EAT
signal(fork[(i+1)%5]);    // put down right fork
signal(fork[i]);          // put down left fork

PROBLEM: Philosopher 0 picks fork[0], Philosopher 1 picks fork[1], ...
         All 5 pick their left fork. All 5 wait for right fork. DEADLOCK!
```

#### 🔧 Correct Solutions

**Solution 1: Allow at most 4 philosophers to sit**
```c
Semaphore room = 4;   // only 4 philosophers can pick up forks

// Philosopher i:
wait(room);
wait(fork[i]);
wait(fork[(i+1)%5]);
// EAT
signal(fork[(i+1)%5]);
signal(fork[i]);
signal(room);

// At least one philosopher can always complete eating → no deadlock ✓
```

**Solution 2: Asymmetric Solution**
```c
// Odd philosophers pick LEFT then RIGHT
// Even philosophers pick RIGHT then LEFT

// Philosopher i:
if (i % 2 == 0) {
    wait(fork[(i+1)%5]);   // right first
    wait(fork[i]);          // then left
} else {
    wait(fork[i]);          // left first
    wait(fork[(i+1)%5]);   // then right
}
// EAT
signal(fork[i]);
signal(fork[(i+1)%5]);

// Breaks circular wait → no deadlock ✓
```

**Solution 3: All-or-Nothing (Atomic pickup)**
```c
// Pick up BOTH forks atomically or pick up NEITHER

// Use a mutex to make fork pickup atomic:
wait(mutex);
if (fork[i] && fork[(i+1)%5]) {
    fork[i] = fork[(i+1)%5] = false;
    signal(mutex);
    // EAT
    wait(mutex);
    fork[i] = fork[(i+1)%5] = true;
}
signal(mutex);
// If couldn't get both → release mutex, wait, retry
```

---

## 8. Monitors

### 🧠 What is a Monitor?
High-level synchronization construct. Like a class where only one method runs at a time.
The compiler guarantees mutual exclusion — programmer doesn't manage semaphores.

### 🔧 Structure
```
monitor MonitorName {
    // shared variables (private — accessible only through monitor procedures)
    
    procedure P1() { ... }
    procedure P2() { ... }
    
    initialization code { ... }
}

// GUARANTEE: Only ONE process can be active inside the monitor at any time.
// Other processes trying to enter → automatically blocked.
```

### 🔧 Condition Variables
```c
// Monitors use condition variables for waiting inside the monitor:
condition x, y;

x.wait()   → process suspends itself, releases monitor lock
x.signal() → wakes up ONE waiting process on x
             if no one waiting → signal is LOST (unlike semaphore where it's remembered)

// KEY DIFFERENCE FROM SEMAPHORE:
// signal() on semaphore with no waiters → increments S (remembered for future)
// signal() on condition variable with no waiters → LOST (no effect)
```

### 🔧 Monitor Solution to Dining Philosophers
```c
monitor DiningPhilosophers {
    enum {THINKING, HUNGRY, EATING} state[5];
    condition self[5];

    void pickup(int i) {
        state[i] = HUNGRY;
        test(i);
        if (state[i] != EATING)
            self[i].wait();  // wait until can eat
    }

    void putdown(int i) {
        state[i] = THINKING;
        test((i+4)%5);  // check left neighbor
        test((i+1)%5);  // check right neighbor
    }

    void test(int i) {
        if (state[(i+4)%5] != EATING &&
            state[i] == HUNGRY &&
            state[(i+1)%5] != EATING) {
            state[i] = EATING;
            self[i].signal();
        }
    }
}
// No deadlock. No starvation (with fair condition variable scheduling).
```

---

## 9. Deadlocks — Complete Coverage

### 🧠 What is a Deadlock?
```
A set of processes is DEADLOCKED if every process in the set is waiting
for an event (resource release) that can only be caused by another
process in the same set.

Result: ALL processes in the set are permanently blocked. No progress.

Example:
  P1 holds R1, needs R2.
  P2 holds R2, needs R1.
  → Both wait forever. Neither can proceed.
```

### 🔧 The Four Coffman Conditions
**ALL FOUR must hold simultaneously for deadlock to occur.**
**Break ANY ONE → deadlock impossible.**

#### Condition 1: Mutual Exclusion
```
At least one resource must be held in non-sharable mode.
Only ONE process can use the resource at a time.
If another process requests it → must wait.

Cannot break for: printers, tape drives (inherently non-sharable)
Can be broken for: read-only files (make them sharable)
```

#### Condition 2: Hold and Wait
```
A process must be HOLDING at least one resource
AND waiting to acquire additional resources held by other processes.

Break by:
  (a) Request ALL resources before starting (low utilization, starvation)
  (b) Request resource only when holding NONE (release all, then re-request)
```

#### Condition 3: No Preemption
```
Resources CANNOT be forcibly taken from a process.
Process must voluntarily release them.

Break by: If process can't get needed resource → release all it holds,
          restart from scratch when all available.
Works for: CPU registers, memory (can be virtualized)
Doesn't work for: printers, file writes (can't undo partial output)
```

#### Condition 4: Circular Wait
```
A circular chain of processes exists:
P1 waits for P2, P2 waits for P3, ..., Pn waits for P1.

Break by: Assign a GLOBAL ORDER to all resource types.
          Processes must always request resources in increasing order.
          
Example: R1(id=1), R2(id=2), R3(id=3)
         All processes must acquire in order: R1 before R2 before R3.
         No circular wait possible. ✓
```

---

### 🔧 Deadlock Handling Strategies

#### Strategy 1: Prevention
```
Prevent at least one Coffman condition from holding.
Guarantees deadlock never occurs.
Usually expensive — wastes resources or reduces system performance.

Break Hold & Wait:   → request all resources upfront (low utilization)
Break No Preemption: → preempt resources if needed (not always possible)
Break Circular Wait: → impose total ordering on resource types (practical!)
```

#### Strategy 2: Avoidance — Banker's Algorithm
```
OS dynamically checks before granting resource:
"If I grant this request, does a SAFE STATE still exist?"

SAFE STATE: There exists a sequence where every process can finish.
UNSAFE STATE: No guarantee deadlock won't happen (not necessarily deadlock).

Process requests resource → OS runs Banker's Algorithm:
  If safe state remains → GRANT
  If would enter unsafe state → DENY (make process wait)

Requires knowing MAXIMUM resource needs upfront.
```

#### Strategy 3: Detection & Recovery
```
Allow deadlocks to occur. Run detection algorithm periodically.
When deadlock detected → recover.

Recovery options:
  1. Kill ALL deadlocked processes (simple, expensive)
  2. Kill ONE process at a time, re-run detection (costly in steps)
  3. Preempt resources from victims (rollback to checkpoint)

Victim selection criteria:
  - Minimize cost (pick process that has done least work)
  - Priority of process
  - Resources held (pick one that releases most)
```

#### Strategy 4: Ignorance (Ostrich Algorithm)
```
Pretend deadlock doesn't exist.
Reboot when system hangs.

Used by: Windows, Linux, macOS (!)
Rationale: Deadlocks rare in practice, prevention cost > occasional reboot.
Named "Ostrich" — ostrich sticks head in sand, ignores problems.
```

---

## 10. Banker's Algorithm — Full Numericals

### 🧠 Setup — Data Structures
```
n processes: P0, P1, ..., Pn-1
m resource types: R1, R2, ..., Rm

ALLOCATION[n×m]:  resources currently allocated to each process
MAX[n×m]:         maximum resources each process may ever request
AVAILABLE[m]:     currently available instances of each resource type

NEED[n×m]:        remaining resources each process may still request
                  NEED[i][j] = MAX[i][j] - ALLOCATION[i][j]
```

### 🔧 Safety Algorithm — Is Current State Safe?
```
Step 1: Initialize:
        WORK = AVAILABLE    (copy of available resources)
        FINISH[i] = false for all i

Step 2: Find process i where:
        FINISH[i] = false   (not yet finished)
        AND NEED[i] ≤ WORK  (its remaining needs can be satisfied)
        If found → go to Step 3
        If not found → go to Step 4

Step 3: Simulate process i completing:
        WORK = WORK + ALLOCATION[i]   (it releases its resources)
        FINISH[i] = true
        Go to Step 2

Step 4: If FINISH[i] = true for ALL i → SAFE STATE, output safe sequence
        Else → UNSAFE STATE (deadlock possible)

Time complexity: O(n² × m)
```

### 🔧 Resource-Request Algorithm — Can We Grant This Request?
```
Process Pi makes request REQUEST[i] for resources:

Step 1: If REQUEST[i] ≤ NEED[i] → continue
        Else → ERROR (exceeded maximum claim)

Step 2: If REQUEST[i] ≤ AVAILABLE → continue
        Else → WAIT (resources not available)

Step 3: Pretend to allocate (temporarily update):
        AVAILABLE  = AVAILABLE  - REQUEST[i]
        ALLOCATION[i] = ALLOCATION[i] + REQUEST[i]
        NEED[i]    = NEED[i]    - REQUEST[i]

Step 4: Run Safety Algorithm on this new state:
        If SAFE → grant the request (make temporary changes permanent)
        If UNSAFE → rollback changes, Pi must wait
```

---

### 📝 Numerical 1 — Basic Safety Check

**Given (5 processes, 3 resource types A, B, C):**

| Process | Allocation (A B C) | Max (A B C) | Need (A B C) |
|---|---|---|---|
| P0 | 0 1 0 | 7 5 3 | 7 4 3 |
| P1 | 2 0 0 | 3 2 2 | 1 2 2 |
| P2 | 3 0 2 | 9 0 2 | 6 0 0 |
| P3 | 2 1 1 | 2 2 2 | 0 1 1 |
| P4 | 0 0 2 | 4 3 3 | 4 3 1 |

**Available = [3, 3, 2]**

**Q: Is the system in a safe state? If yes, find the safe sequence.**

**Solution:**
```
WORK = [3, 3, 2], FINISH = [F, F, F, F, F]

Pass 1:
  P0: NEED=[7,4,3], WORK=[3,3,2]. 7>3 → Cannot satisfy P0. Skip.
  P1: NEED=[1,2,2], WORK=[3,3,2]. 1≤3, 2≤3, 2≤2 → CAN satisfy!
      WORK = [3,3,2] + [2,0,0] = [5,3,2]. FINISH[1]=true.
  P2: NEED=[6,0,0], WORK=[5,3,2]. 6>5 → Cannot. Skip.
  P3: NEED=[0,1,1], WORK=[5,3,2]. 0≤5, 1≤3, 1≤2 → CAN satisfy!
      WORK = [5,3,2] + [2,1,1] = [7,4,3]. FINISH[3]=true.
  P4: NEED=[4,3,1], WORK=[7,4,3]. 4≤7, 3≤4, 1≤3 → CAN satisfy!
      WORK = [7,4,3] + [0,0,2] = [7,4,5]. FINISH[4]=true.

Pass 2:
  P0: NEED=[7,4,3], WORK=[7,4,5]. 7≤7, 4≤4, 3≤5 → CAN satisfy!
      WORK = [7,4,5] + [0,1,0] = [7,5,5]. FINISH[0]=true.
  P2: NEED=[6,0,0], WORK=[7,5,5]. 6≤7, 0≤5, 0≤5 → CAN satisfy!
      WORK = [7,5,5] + [3,0,2] = [10,5,7]. FINISH[2]=true.

All FINISH = true → SAFE STATE ✓
Safe sequence: P1 → P3 → P4 → P0 → P2
```

---

### 📝 Numerical 2 — Resource Request

**Same system as above. P1 requests [1, 0, 2].**
**Q: Can the request be granted?**

**Solution:**
```
Step 1: Check REQUEST ≤ NEED:
        [1,0,2] ≤ [1,2,2]? → Yes ✓

Step 2: Check REQUEST ≤ AVAILABLE:
        [1,0,2] ≤ [3,3,2]? → Yes ✓

Step 3: Pretend to allocate:
        AVAILABLE   = [3,3,2] - [1,0,2] = [2,3,0]
        ALLOCATION[1] = [2,0,0] + [1,0,2] = [3,0,2]
        NEED[1]     = [1,2,2] - [1,0,2] = [0,2,0]

Step 4: Run Safety Algorithm with new state:
        Updated table:
        P0: Alloc=[0,1,0], Need=[7,4,3]
        P1: Alloc=[3,0,2], Need=[0,2,0]  ← updated
        P2: Alloc=[3,0,2], Need=[6,0,0]
        P3: Alloc=[2,1,1], Need=[0,1,1]
        P4: Alloc=[0,0,2], Need=[4,3,1]
        Available = [2,3,0]

        WORK=[2,3,0], FINISH=[F,F,F,F,F]
        P1: NEED=[0,2,0] ≤ [2,3,0]? → Yes! WORK=[2,3,0]+[3,0,2]=[5,3,2]. FINISH[1]=T
        P3: NEED=[0,1,1] ≤ [5,3,2]? → Yes! WORK=[5,3,2]+[2,1,1]=[7,4,3]. FINISH[3]=T
        P4: NEED=[4,3,1] ≤ [7,4,3]? → Yes! WORK=[7,4,3]+[0,0,2]=[7,4,5]. FINISH[4]=T
        P0: NEED=[7,4,3] ≤ [7,4,5]? → Yes! WORK=[7,4,5]+[0,1,0]=[7,5,5]. FINISH[0]=T
        P2: NEED=[6,0,0] ≤ [7,5,5]? → Yes! WORK=[7,5,5]+[3,0,2]=[10,5,7]. FINISH[2]=T

        All FINISH=true → SAFE STATE → GRANT REQUEST ✓
        Safe sequence: P1 → P3 → P4 → P0 → P2
```

---

### 📝 Numerical 3 — Unsafe Request

**Same system. P4 requests [3, 3, 0].**
**Q: Can the request be granted?**

**Solution:**
```
Step 1: REQUEST=[3,3,0] ≤ NEED[4]=[4,3,1]? → Yes ✓
Step 2: REQUEST=[3,3,0] ≤ AVAILABLE=[3,3,2]? → Yes ✓

Step 3: Pretend to allocate:
        AVAILABLE     = [3,3,2] - [3,3,0] = [0,0,2]
        ALLOCATION[4] = [0,0,2] + [3,3,0] = [3,3,2]
        NEED[4]       = [4,3,1] - [3,3,0] = [1,0,1]

Step 4: Safety check with AVAILABLE=[0,0,2]:
        WORK=[0,0,2]
        P0: NEED=[7,4,3] ≤ [0,0,2]? → 7>0. NO.
        P1: NEED=[1,2,2] ≤ [0,0,2]? → 1>0. NO.
        P2: NEED=[6,0,0] ≤ [0,0,2]? → 6>0. NO.
        P3: NEED=[0,1,1] ≤ [0,0,2]? → 1>0. NO.
        P4: NEED=[1,0,1] ≤ [0,0,2]? → 1>0. NO.

        No process can proceed! UNSAFE STATE.
        → DENY REQUEST. P4 must wait. ✓
```

---

### 📝 Numerical 4 — Find Available from Allocation

**Often asked: Calculate AVAILABLE when not given directly.**

| Process | Allocation (A B C) | Max (A B C) |
|---|---|---|
| P0 | 1 0 2 | 4 2 3 |
| P1 | 2 1 0 | 5 3 2 |
| P2 | 1 1 1 | 2 2 2 |
| P3 | 0 0 2 | 3 1 4 |

**Total instances: A=8, B=4, C=6**

**Q: Find AVAILABLE and NEED. Is system safe?**

```
AVAILABLE = Total - Σ(ALLOCATION)
A: 8 - (1+2+1+0) = 8-4 = 4
B: 4 - (0+1+1+0) = 4-2 = 2
C: 6 - (2+0+1+2) = 6-5 = 1
AVAILABLE = [4, 2, 1]

NEED = MAX - ALLOCATION:
P0: [4,2,3]-[1,0,2] = [3,2,1]
P1: [5,3,2]-[2,1,0] = [3,2,2]
P2: [2,2,2]-[1,1,1] = [1,1,1]
P3: [3,1,4]-[0,0,2] = [3,1,2]

Safety Check: WORK=[4,2,1]
P0: [3,2,1]≤[4,2,1]? → Yes! WORK=[4,2,1]+[1,0,2]=[5,2,3]. FINISH[0]=T
P2: [1,1,1]≤[5,2,3]? → Yes! WORK=[5,2,3]+[1,1,1]=[6,3,4]. FINISH[2]=T
P1: [3,2,2]≤[6,3,4]? → Yes! WORK=[6,3,4]+[2,1,0]=[8,4,4]. FINISH[1]=T
P3: [3,1,2]≤[8,4,4]? → Yes! WORK=[8,4,4]+[0,0,2]=[8,4,6]. FINISH[3]=T

SAFE! Sequence: P0 → P2 → P1 → P3 ✓
```

---

## 11. Resource Allocation Graph — Numericals

### 🔧 RAG Rules
```
NODES:
  Circles (○) = Processes: P1, P2, P3...
  Rectangles (□) = Resource types: R1, R2...
  Dots inside rectangles = instances of that resource

EDGES:
  Pi → Rj (Request edge):   Process Pi is WAITING for resource Rj
  Rj → Pi (Assignment edge): One instance of Rj is ASSIGNED to Pi

DEADLOCK DETECTION:
  If each resource has EXACTLY ONE instance:
    Deadlock ⟺ CYCLE exists in the graph
    
  If resources have MULTIPLE instances:
    Cycle is NECESSARY but NOT SUFFICIENT for deadlock
    Must run detection algorithm (similar to Banker's)
```

### 📝 Numerical 1 — Deadlock Detection (Single Instance)

```
Draw and analyze:

P1 → R1 (P1 requests R1)
R1 → P2 (R1 is held by P2)
P2 → R2 (P2 requests R2)
R2 → P3 (R2 is held by P3)
P3 → R1 (P3 requests R1) ← CYCLE!

Cycle: P1→R1→P2→R2→P3→R1→P2...
        or: P2→R2→P3→R1→P2 (simpler cycle)

Since R1 and R2 each have 1 instance → DEADLOCK
P1, P2, P3 are all deadlocked.
```

### 📝 Numerical 2 — Cycle Without Deadlock (Multiple Instances)

```
Resources: R1 has 2 instances, R2 has 2 instances
Processes: P1, P2, P3, P4

Edges:
  R1 → P1 (P1 holds one R1)
  R1 → P3 (P3 holds one R1)
  R2 → P2 (P2 holds one R2)
  R2 → P4 (P4 holds one R2)
  P1 → R2 (P1 requests R2)
  P3 → R2 (P3 requests R2)

Cycle? P1→R2→P2... P2 doesn't request anything. No cycle.
       P1→R2→P4... P4 doesn't request anything. No cycle.

Actually no cycle here. But even IF there were a cycle with multiple instances,
we'd need Banker's-style detection to confirm deadlock.

KEY POINT: With single-instance resources, cycle = deadlock.
           With multi-instance resources, cycle ≠ necessarily deadlock.
```

### 📝 Numerical 3 — Find Deadlocked Processes

```
Processes: P1, P2, P3, P4, P5
Resources: R1(2 instances), R2(1 instance), R3(3 instances)

Current state:
Allocation:   P1=[1,0,0], P2=[0,0,1], P3=[0,1,0], P4=[1,0,1], P5=[0,0,1]
Request:      P1=[0,1,0], P2=[1,0,1], P3=[0,0,0], P4=[0,1,0], P5=[0,0,1]
Available:    [0,0,0]

Detection Algorithm (like Safety but with FINISH[i]=true if Allocation[i]=0):
P3 has Allocation=[0,1,0] ≠ 0, so FINISH[3]=false initially.
(All have non-zero allocation, so all start with FINISH=false)

WORK=[0,0,0]
P1: REQUEST=[0,1,0] ≤ [0,0,0]? → 1>0. NO.
P2: REQUEST=[1,0,1] ≤ [0,0,0]? → 1>0. NO.
P3: REQUEST=[0,0,0] ≤ [0,0,0]? → YES! (P3 needs nothing!)
    WORK = [0,0,0] + [0,1,0] = [0,1,0]. FINISH[3]=true.
P4: REQUEST=[0,1,0] ≤ [0,1,0]? → YES!
    WORK = [0,1,0] + [1,0,1] = [1,1,1]. FINISH[4]=true.
P1: REQUEST=[0,1,0] ≤ [1,1,1]? → YES!
    WORK = [1,1,1] + [1,0,0] = [2,1,1]. FINISH[1]=true.
P2: REQUEST=[1,0,1] ≤ [2,1,1]? → YES!
    WORK = [2,1,1] + [0,0,1] = [2,1,2]. FINISH[2]=true.
P5: REQUEST=[0,0,1] ≤ [2,1,2]? → YES!
    WORK = [2,1,2] + [0,0,1] = [2,1,3]. FINISH[5]=true.

All FINISH=true → NO DEADLOCK! System is fine. ✓
```

---

## 12. MCQ Traps & Exam Q&A

### ⚠️ THE TRAPS

---

**TRAP 1: "Deadlock requires all four Coffman conditions"**
✅ TRUE — but trap is in the CONVERSE.
"If all four conditions hold → deadlock" is FALSE.
All four conditions are NECESSARY but NOT SUFFICIENT.
Example: conditions hold but no circular wait yet → no deadlock.
Correct: "Deadlock can only occur if all four conditions hold simultaneously."

---

**TRAP 2: "A cycle in RAG always means deadlock"**
❌ FALSE. Only true when ALL resources have SINGLE instances.
With multiple instances: cycle is necessary but not sufficient.
Must run detection algorithm to confirm.

---

**TRAP 3: "Semaphore signal() wakes up ALL waiting processes"**
❌ FALSE. signal() wakes up exactly ONE waiting process.
Which one? Depends on implementation (usually FIFO or random).
To wake all → signal() must be called once per waiting process.

---

**TRAP 4: "Binary semaphore and mutex are the same thing"**
❌ NOT exactly. Key difference: OWNERSHIP.
Mutex: only the locker can unlock. (Priority inversion protection)
Binary semaphore: any process can signal. (More flexible, more dangerous)
Both provide mutual exclusion but semantics differ.

---

**TRAP 5: "Deadlock avoidance guarantees no deadlock ever"**
✅ TRUE — but avoidance says NO to SAFE requests sometimes.
System may refuse resource even when no deadlock would occur.
Avoidance is CONSERVATIVE — it says no to any request that could
POSSIBLY lead to deadlock, even if it wouldn't in practice.
→ Lower resource utilization than detection.

---

**TRAP 6: Order of wait() in Producer-Consumer**
```
ALWAYS: wait(resource_semaphore) BEFORE wait(mutex)
NEVER:  wait(mutex) BEFORE wait(resource_semaphore)

Wrong order causes DEADLOCK:
  Producer does wait(mutex) when buffer full → holds mutex, blocks on empty
  Consumer does wait(mutex) → blocks because producer holds mutex
  Producer waits for Consumer. Consumer waits for Producer. DEADLOCK!
```

---

**TRAP 7: "More resources always prevents deadlock"**
❌ WRONG. Adding more resources can still have deadlock if circular wait exists.
Deadlock is about the PATTERN of requests, not just total resources.

---

**TRAP 8: "Preemption breaks deadlock"**
⚠️ PARTIAL. Preemption breaks the "No Preemption" Coffman condition.
But preemption must be possible for the resource (works for CPU, memory).
Cannot preempt printer mid-print, or file write mid-operation.

---

**TRAP 9: "Safe state = no deadlock"**
✅ TRUE. Unsafe state ≠ deadlock. This direction is FALSE.
Unsafe state means deadlock is POSSIBLE, not guaranteed.
Safe state guarantees deadlock cannot occur.
Unsafe state means if processes request max, deadlock MAY occur.

---

**TRAP 10: Peterson's algorithm works for n>2 processes**
❌ FALSE. Peterson's is for EXACTLY 2 processes.
For n processes, need bakery algorithm or hardware solutions.

---

### ⚡ Quick Fire Q&A

| Question | Answer |
|---|---|
| What are the 4 Coffman conditions? | Mutual Exclusion, Hold & Wait, No Preemption, Circular Wait |
| Which condition is easiest to break? | Circular Wait (impose resource ordering) |
| What is a safe sequence? | Order in which all processes can finish using available resources |
| Can a safe state have a cycle in RAG? | YES — if resources have multiple instances |
| What does NEED[i][j] = MAX[i][j] - ALLOCATION[i][j] give? | Remaining resource needs of process i for resource j |
| Banker's algorithm requires knowing what upfront? | Maximum resource needs of each process |
| What is the time complexity of Banker's safety algorithm? | O(n² × m) where n=processes, m=resource types |
| When is FINISH[i] initialized to true in detection? | When ALLOCATION[i] is all zeros (process holds nothing) |
| What is starvation in context of semaphores? | A process waits indefinitely because others always go first |
| How does monitor differ from semaphore? | Monitor provides automatic mutual exclusion; compiler-enforced |
| What happens to signal() on condition variable with no waiters? | Signal is LOST (no effect) |
| What happens to signal() on semaphore with no waiters? | Semaphore value increments (remembered) |
| What is the Ostrich Algorithm? | Ignore deadlocks, reboot when system hangs |
| Name two deadlock recovery methods | Kill processes OR preempt resources from victims |
| What is priority inversion? | High-priority process waits for low-priority one that holds needed resource |
| What solves priority inversion? | Priority inheritance (low-priority process inherits high-priority temporarily) |
| What is livelock? | Processes keep changing state in response to each other but no progress |
| How is livelock different from deadlock? | Deadlock: processes stopped. Livelock: processes active but making no progress |
| What is the critical section problem? | Ensure only one process executes in CS at a time |
| Name 3 solutions to critical section problem | Peterson's, TestAndSet, Semaphore/Mutex |

---

### 🔥 5 Most Likely CoreTex Questions

**Q1.** Given Allocation, Max, Available matrices → Is system in safe state? Find sequence.
→ Run safety algorithm step by step (most common pen-and-paper question)

**Q2.** What happens if we swap wait(mutex) and wait(empty) in Producer-Consumer?
→ Deadlock (must explain why with scenario)

**Q3.** A process requests resources. Should Banker's grant or deny? (Given updated matrices)
→ Run resource-request algorithm + safety check

**Q4.** Draw RAG from given scenario. Is there deadlock?
→ Check for cycle + check if single or multi-instance resources

**Q5.** Which Coffman condition does each solution break?
→ "All processes request all resources at start" → breaks HOLD AND WAIT
→ "Resources have total ordering, request only in increasing order" → breaks CIRCULAR WAIT

---

### 📋 Formula & Quick Reference

```
NEED = MAX - ALLOCATION

AVAILABLE = Total_Resources - Σ(ALLOCATION_all_processes)

Semaphore value after k waits, m signals (starting at S):
  S_final = S + m - k
  If S_final < 0: |S_final| = number of processes waiting

Safety Algorithm time: O(n² × m)
Deadlock detection time: O(n² × m)

Peterson's: works for n=2 only
Bakery Algorithm: works for n processes (uses "ticket number" like bakery)

Counting semaphore initialized to N:
  After k waits: S = N - k
  If N - k < 0: |N - k| waiting processes
```

---
