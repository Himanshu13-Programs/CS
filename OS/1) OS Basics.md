# 📘 OS Chapter 1 — OS Basics 
---

## 📌 Table of Contents
1. [What is an Operating System?](#1-what-is-an-operating-system)
2. [Types of Operating Systems](#2-types-of-operating-systems)
3. [OS Structure & Kernel Types](#3-os-structure--kernel-types)
4. [Dual Mode Operation](#4-dual-mode-operation)
5. [System Calls — Deep Dive](#5-system-calls--deep-dive)
6. [Interrupts & I/O](#6-interrupts--io)
7. [Booting Process](#7-booting-process)
8. [Numericals — Solved](#8-numericals--solved)
9. [MCQ Traps & Exam Q&A](#9-mcq-traps--exam-qa)

---

## 1. What is an Operating System?

### 🧠 Three Ways to Define an OS (examiners love all three)

**Definition 1 — Resource Manager:**
> The OS is a program that manages computer hardware and software resources and provides common services for programs.

**Definition 2 — Extended Machine / Virtual Machine:**
> The OS provides a clean abstract interface to ugly hardware. Programs see "files" and "processes", not sectors and transistors.

**Definition 3 — Government Analogy:**
> Like a government — does nothing useful by itself but enables everything else to happen in an orderly way.

---

### 🔧 What the OS Actually Does
```
┌──────────────────────────────────────────────────────────┐
│                  USER PROGRAMS                           │
│   (Chrome, VS Code, your C++ program, games...)          │
├──────────────────────────────────────────────────────────┤
│              SYSTEM PROGRAMS                             │
│   (Compiler, Shell, File Explorer, Task Manager...)      │
├──────────────────────────────────────────────────────────┤
│              OPERATING SYSTEM                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐  │
│  │ Process  │ │ Memory   │ │  File    │ │    I/O     │  │
│  │ Manager  │ │ Manager  │ │  System  │ │  Manager   │  │
│  └──────────┘ └──────────┘ └──────────┘ └────────────┘  │
├──────────────────────────────────────────────────────────┤
│                   HARDWARE                               │
│   CPU    RAM    Disk    NIC    GPU    USB    Keyboard     │
└──────────────────────────────────────────────────────────┘
```

### 🔧 OS Responsibilities — Detailed

| Responsibility | What it involves |
|---|---|
| **Process Management** | Create/delete processes, scheduling, IPC, synchronization |
| **Memory Management** | Track which memory is used/free, allocate/deallocate, virtual memory |
| **File System Management** | Create/delete files and dirs, map files to physical storage, backup |
| **I/O Management** | Manage device drivers, buffering, caching, spooling |
| **Security & Protection** | Authentication, access control, isolation between processes |
| **Networking** | Manage TCP/IP stack, sockets, network interfaces |
| **Secondary Storage** | Disk scheduling, free space management, storage allocation |

---

## 2. Types of Operating Systems

### 2.1 Batch OS
```
No direct interaction with user.
Jobs collected, grouped (batched), submitted together.
CPU idle while I/O happens → poor utilization.

Timeline: [Job1 CPU][Job1 I/O (CPU idle)][Job2 CPU][Job2 I/O (CPU idle)]...

Example: Early IBM systems (1950s-60s), payroll processing
Problem: One job at a time. If job waits for I/O, CPU sits idle.
```

### 2.2 Multiprogramming OS
```
Key insight: Keep MULTIPLE jobs in memory simultaneously.
When one job waits for I/O → run ANOTHER job on CPU.
CPU utilization improved dramatically.

Memory: [OS | Job1 | Job2 | Job3]
                ↓
If Job1 doing I/O → CPU runs Job2
If Job2 done → CPU runs Job3
→ CPU almost never idle

Single CPU. No preemption — job runs until it does I/O or finishes.
```

### 2.3 Multitasking / Time-Sharing OS
```
Extension of multiprogramming with PREEMPTION.
CPU switches between jobs so fast users feel they have dedicated CPU.
Response time goal: < 1 second.

Each process gets a TIME QUANTUM (e.g., 10ms).
After quantum → preempt, run next process.

Example: Unix, Linux, Windows
```

### 2.4 Multiprocessing OS
```
MULTIPLE CPUs (cores) share memory and work simultaneously.
TRUE parallelism (vs concurrency in time-sharing).

Symmetric Multiprocessing (SMP): all CPUs equal, share one OS
Asymmetric Multiprocessing: one master CPU controls others

Example: Modern quad-core/octa-core laptops
```

### 2.5 Real-Time OS (RTOS)
```
Correctness depends on BOTH the result AND the time it's produced.
Deadline is a hard constraint.

Hard RTOS: Missing deadline = FAILURE (airbag controller, pacemaker, ATC)
Soft RTOS: Missing deadline = degraded quality (video streaming, online gaming)

No virtual memory in hard RTOS (page faults unpredictable!)
Scheduling: EDF (Earliest Deadline First) or Rate Monotonic
```

### 2.6 Distributed OS
```
Multiple independent computers connected by network.
Appear to user as a single coherent system.
Resources shared across network.
Example: Google's internal systems, cluster computing
```

### 2.7 Embedded OS
```
Designed for embedded systems (specific hardware).
Minimal features, very small footprint.
Example: FreeRTOS (smartwatch), VxWorks (spacecraft), QNX (cars)
```

### 📊 Comparison Table

| Type | Preemption | Multi-user | Real-time | Example |
|---|---|---|---|---|
| Batch | No | No | No | IBM OS/360 |
| Multiprogramming | No | No | No | Early Unix |
| Time-Sharing | Yes | Yes | No | Unix, Linux |
| Multiprocessing | Yes | Yes | No | Windows, macOS |
| Real-Time | Special | Limited | YES | VxWorks, FreeRTOS |
| Distributed | Yes | Yes | No | Plan 9, Amoeba |

---

## 3. OS Structure & Kernel Types

### 🧠 The Kernel
The kernel is the **core of the OS** — the part that runs in privileged mode with full hardware access. Everything else (file managers, shells, GUIs) runs in user space.

```
┌─────────────────────────────────┐
│      User Applications          │  User Space
│      System Utilities           │  (restricted)
├─────────────────────────────────┤  ← System Call Interface
│           KERNEL                │  Kernel Space
│  (memory mgmt, scheduling,      │  (full hardware access)
│   device drivers, file system)  │
├─────────────────────────────────┤
│           Hardware              │
└─────────────────────────────────┘
```

---

### 3.1 Monolithic Kernel
```
ENTIRE OS runs in kernel space as one large program.
All services (file system, memory, drivers, scheduling) in kernel.

Pros:
  + Fast (no mode switching between services)
  + Services can directly call each other

Cons:
  - Large, complex, hard to maintain
  - One bug can crash entire system
  - Adding new feature = modify kernel

Examples: Linux, early Unix, MS-DOS

Linux is monolithic but uses LOADABLE KERNEL MODULES (LKM)
→ can add drivers at runtime without recompiling kernel
```

### 3.2 Microkernel
```
Kernel contains ONLY the essentials:
  - Basic IPC (inter-process communication)
  - Basic memory management
  - Basic scheduling

Everything else (file system, device drivers, network stack)
runs in USER SPACE as separate processes (servers).

Communication: via message passing (IPC)

Pros:
  + More stable (driver crash doesn't crash kernel)
  + Easier to extend and port
  + More secure (less code in privileged mode)

Cons:
  - SLOWER — mode switch for every service call
  - Message passing overhead

Examples: Minix, QNX, L4, GNU Hurd, macOS (hybrid)
```

### 3.3 Hybrid Kernel
```
Compromise: microkernel architecture but some services
moved back into kernel space for performance.

Examples: Windows NT (and all modern Windows), macOS (XNU)
```

### 3.4 Exokernel
```
Even more minimal than microkernel.
Kernel only does resource PROTECTION, not management.
Applications manage their own resources directly.
Research-oriented. Example: MIT Exokernel
```

### 3.5 Unikernel
```
Application + only the OS components it needs → single image.
No separation between kernel and app.
Used in cloud VMs for ultra-lightweight deployment.
```

### 📊 Kernel Type Comparison

| Type | Speed | Stability | Size | Example |
|---|---|---|---|---|
| Monolithic | ⚡ Fast | Medium | Large | Linux |
| Microkernel | 🐢 Slower | High | Small | QNX, Minix |
| Hybrid | Medium | Medium | Medium | Windows, macOS |
| Exokernel | ⚡ Fast | Application-managed | Tiny | MIT Exokernel |

---

## 4. Dual Mode Operation

### 🧠 Why Two Modes?
Without protection, any program could corrupt OS memory, talk directly to hardware, or crash the entire system. Dual mode creates a hard boundary.

```
┌─────────────────────────────────────────────────────┐
│                 USER MODE (Mode bit = 1)             │
│  - Restricted instruction set                        │
│  - Cannot access hardware directly                   │
│  - Cannot execute privileged instructions            │
│  - Runs user programs, system libraries              │
└────────────────────┬────────────────────────────────┘
                     │ System call / Interrupt / Exception
                     ▼ (hardware sets mode bit = 0)
┌─────────────────────────────────────────────────────┐
│               KERNEL MODE (Mode bit = 0)             │
│  - Full instruction set available                    │
│  - Direct hardware access                            │
│  - Can execute ALL instructions                      │
│  - Runs OS kernel code                               │
└────────────────────┬────────────────────────────────┘
                     │ return from system call
                     ▼ (hardware sets mode bit = 1)
                 User Mode
```

### 🔧 Privileged Instructions (only in kernel mode)
```
- I/O instructions (IN, OUT) — direct hardware access
- Halt instruction — stop CPU
- Timer management — set hardware timer
- Interrupt enable/disable
- Switch to/from user mode
- Access/modify page tables
- Access control registers (CR0, CR3 on x86)
```

### 🔧 How Mode Switch Happens

**User → Kernel (3 ways):**
```
1. SYSTEM CALL   → program intentionally asks OS for service
                   (write(), read(), fork()...)
                   Uses software interrupt (int 0x80 on x86, syscall on x86-64)

2. INTERRUPT     → hardware device signals CPU
                   (timer interrupt, keyboard press, disk I/O complete)
                   CPU stops current task, runs interrupt handler

3. EXCEPTION     → CPU detects error in running program
                   (division by zero, page fault, illegal instruction)
                   CPU traps to OS exception handler
```

**Kernel → User:**
```
After handling system call / interrupt / exception:
  - OS sets mode bit = 1
  - Restores user program registers
  - Returns to user program (next instruction or restart faulting one)
```

### 🔧 Timer — Preventing CPU Monopoly
```
Hardware timer generates interrupt every N milliseconds.
When timer fires → always transfers control to OS (regardless of user code).
This is HOW preemption works — OS can't rely on programs to voluntarily give up CPU.

Without timer: a program could run infinite loop and monopolize CPU forever.
```

### 📝 Numerical — Mode Switch Overhead
```
Given:
  User mode instruction: 1 ns
  Mode switch (user→kernel): 200 ns
  Kernel mode instruction: 1 ns
  Mode switch (kernel→user): 200 ns
  System call does 10 kernel instructions

Time for one system call = 200 + 10×1 + 200 = 410 ns

vs. same code in user mode = 10 × 1 = 10 ns
→ system call is 41× more expensive!

This is why minimizing system calls matters for performance.
```

---

## 5. System Calls — Deep Dive

### 🧠 What is a System Call?
The **programming interface** between user programs and the OS kernel. The only legitimate way to request kernel services.

```
User Program (C code)
     │
     │  write(1, "hello", 5);   ← library function (libc)
     ▼
  libc / glibc wrapper
     │
     │  mov rax, 1      ← syscall number for write (Linux x86-64)
     │  mov rdi, 1      ← file descriptor (stdout)
     │  mov rsi, buf    ← buffer address
     │  mov rdx, 5      ← byte count
     │  syscall         ← trap into kernel
     ▼
  Kernel Mode
     │
     │  sys_write() executes
     │  validates params, writes to fd
     │  puts result in rax
     ▼
  Return to user mode
     │
     │  return value in rax (bytes written or -1 for error)
     ▼
  User Program continues
```

### 🔧 Complete System Call Categories

#### Process Control
```
fork()     → create child process (copy of parent)
             return value: 0 in child, child_PID in parent, -1 on error
exec()     → replace process image with new program
             exec does NOT return on success
exit(n)    → terminate process with exit code n
wait(&status) → parent waits for any child
waitpid(pid, &status, 0) → wait for specific child
kill(pid, signal) → send signal to process
getpid()   → return current process's PID
getppid()  → return parent's PID
```

#### File Management
```
open(path, flags) → opens file, returns file descriptor (int)
                    flags: O_RDONLY, O_WRONLY, O_RDWR, O_CREAT, O_APPEND
read(fd, buf, n)  → read n bytes from fd into buf, returns bytes read
write(fd, buf, n) → write n bytes from buf to fd, returns bytes written
close(fd)         → close file descriptor
lseek(fd, offset, whence) → move file position
                    SEEK_SET from beginning, SEEK_CUR from current, SEEK_END from end
stat(path, &buf)  → get file metadata (size, permissions, timestamps)
unlink(path)      → delete file (removes directory entry)
```

#### Memory Management
```
brk(addr)        → set end of data segment (grow/shrink heap)
mmap(addr, len, prot, flags, fd, offset)
                 → map file or anonymous memory into address space
munmap(addr, len) → unmap memory
```

#### Communication (IPC)
```
pipe(fd[2])      → create pipe, fd[0]=read end, fd[1]=write end
socket(domain, type, protocol) → create socket
bind, listen, accept, connect → network/local socket ops
send, recv       → send/receive data on socket
```

### 🔧 File Descriptor Table
```
Every process has a File Descriptor Table:

fd 0 → stdin  (keyboard by default)
fd 1 → stdout (screen by default)
fd 2 → stderr (screen by default)
fd 3 → first file you open
fd 4 → second file...
...

File descriptors are inherited by child processes after fork()!
This is how shell redirection works:
  close(1)              ← close stdout
  open("file.txt", ...) ← opens as fd 1 (lowest available)
  exec("ls")            ← ls writes to fd 1 → goes to file.txt
```

### 📝 Numerical — fork() Output Tracing
```c
// Classic exam question: how many processes created?

int main() {
    fork();   // creates 1 child → now 2 processes
    fork();   // EACH of the 2 processes forks → now 4 processes
    fork();   // EACH of the 4 processes forks → now 8 processes
    printf("Hello\n");
    return 0;
}

Answer: 8 processes, "Hello" printed 8 times.

Rule: n fork() calls → 2^n processes (if no conditionals)

─────────────────────────────────────────────────
More complex example:
int main() {
    int x = fork();
    if (x == 0) {   // only child executes this
        fork();
    }
    printf("Hi\n");
}

Process tree:
  Parent (x = child_PID, skips if block)
  Child  (x = 0, enters if block)
    Grandchild (child's fork)

"Hi" printed 3 times (Parent, Child, Grandchild)
```

### 📝 Numerical — pipe() Communication
```c
// Trace output of this program:
int fd[2];
pipe(fd);

if (fork() == 0) {
    // CHILD: write to pipe
    close(fd[0]);                    // close unused read end
    write(fd[1], "Hello", 5);
    close(fd[1]);
    exit(0);
} else {
    // PARENT: read from pipe
    close(fd[1]);                    // close unused write end
    char buf[10];
    int n = read(fd[0], buf, 10);   // reads 5 bytes: "Hello"
    buf[n] = '\0';
    printf("%s\n", buf);             // prints: Hello
}

Output: Hello
```

---

## 6. Interrupts & I/O

### 🧠 What is an Interrupt?
A signal to the CPU that something needs immediate attention. Forces CPU to stop current task, handle the interrupt, then resume.

```
CPU executing User Program
         │
  [Hardware device sends interrupt signal]
         │
  CPU finishes current instruction (not mid-instruction)
         │
  CPU saves current state (PC, registers) on kernel stack
         │
  CPU looks up Interrupt Vector Table → finds handler address
         │
  CPU jumps to Interrupt Handler (kernel code)
         │
  Handler runs (e.g., "keyboard pressed, store character")
         │
  CPU restores saved state
         │
  CPU resumes user program from where it stopped
```

### 🔧 Types of Interrupts

| Type | Source | Example |
|---|---|---|
| **Hardware Interrupt** | External device | Timer, keyboard, disk I/O complete, NIC |
| **Software Interrupt** | Program instruction | `int 0x80` syscall, deliberate trap |
| **Exception** | CPU error during execution | Divide by zero, page fault, illegal instruction |

### 🔧 Interrupt Vector Table (IVT)
```
Array of function pointers — one per interrupt number.
Each entry = address of the handler for that interrupt.

IVT[0]  → divide by zero handler
IVT[1]  → debug handler
IVT[14] → page fault handler
IVT[32] → timer interrupt handler
IVT[33] → keyboard interrupt handler
...
IVT[128 / 0x80] → system call handler (Linux x86)
```

### 🔧 I/O Methods — How CPU and Devices Communicate

#### 1. Programmed I/O (Polling / Busy-Wait)
```
CPU continuously checks if device is ready.

while (device_not_ready) { /* spin */ }
read_from_device();

Pros: Simple
Cons: CPU wastes 100% time polling — can't do other work
Use: Only for very fast devices or real-time systems
```

#### 2. Interrupt-Driven I/O
```
CPU starts I/O operation, then does OTHER WORK.
When device finishes → sends interrupt → CPU handles it.

CPU: start_IO();  → continues running other processes
...
[Device done] → INTERRUPT → OS reads data, wakes waiting process

Pros: CPU free to do other work during I/O
Cons: Interrupt overhead, one interrupt per byte for slow devices
```

#### 3. DMA — Direct Memory Access
```
CPU tells DMA controller: "copy N bytes from device to memory address X"
DMA does the actual transfer INDEPENDENTLY (CPU not involved)
DMA sends ONE interrupt when entire transfer complete

Timeline:
CPU: [start DMA] [does other work ............] [gets interrupt, processes data]
DMA:             [transfers entire block ......]

Pros: CPU minimally involved — only 1 interrupt per block
Cons: DMA controller hardware needed, bus contention
Use: Disk reads, network packets, USB — any bulk transfer
```

### 🔧 I/O Software Layers
```
┌─────────────────────────┐
│   User-level I/O (stdio)│ ← printf, scanf, fread...
├─────────────────────────┤
│  Device-Independent OS  │ ← buffering, caching, spooling, naming
├─────────────────────────┤
│    Device Drivers       │ ← device-specific code (written by vendors)
├─────────────────────────┤
│  Interrupt Handlers     │ ← low-level interrupt response
├─────────────────────────┤
│       Hardware          │ ← actual device
└─────────────────────────┘
```

### 🔧 Buffering, Caching, Spooling

**Buffering:**
```
Temporary storage to handle speed mismatch between producer and consumer.
Example: Network data arrives faster than app processes it → buffer it.
Double buffering: fill buffer A while OS processes buffer B → better throughput.
```

**Caching:**
```
Keep frequently used data in faster storage.
Disk cache in RAM → if data requested is in cache (hit) → return from RAM
                  → if not (miss) → read from disk, store in cache
```

**Spooling (Simultaneous Peripheral Operations On-Line):**
```
For slow devices that can't handle interleaved data (like printer).
Each job's output written to disk (spool).
Spooler feeds to printer one job at a time.

Without spooling: only one process can use printer at a time
With spooling: all processes write to disk, spooler manages printer queue
```

---

## 7. Booting Process

### 🔧 Step-by-Step Boot Sequence
```
STEP 1: POWER ON
  → CPU starts executing from a fixed address (0xFFFF0 on x86)
  → This address contains a jump to BIOS/UEFI

STEP 2: BIOS / UEFI (Basic Input/Output System)
  → Stored in ROM/Flash on motherboard
  → POST (Power-On Self Test): check RAM, CPU, keyboard, disk
  → Find bootable device (disk, USB, network)
  → Load first 512 bytes from disk → this is the MBR

STEP 3: MBR — Master Boot Record
  → First sector of disk (512 bytes)
  → Contains: Boot loader (446 bytes) + Partition table (64 bytes) + Signature (2 bytes)
  → Boot loader's job: find and load the actual OS bootloader

STEP 4: Bootloader (GRUB, Windows Boot Manager)
  → Loads OS kernel from disk into RAM
  → Passes control to kernel
  → On GRUB: can select which OS to boot (dual boot)

STEP 5: Kernel Initialization
  → Kernel decompresses itself
  → Initializes CPU, memory management, interrupt handlers
  → Detects hardware, loads device drivers
  → Mounts root filesystem

STEP 6: Init Process (PID 1)
  → First user-space process (systemd on modern Linux, init on older)
  → Starts all system services (network, logging, display manager...)
  → Presents login prompt or GUI

STEP 7: User Login
  → Shell or GUI starts
  → User-space is fully operational
```

### 🔧 BIOS vs UEFI

| | BIOS | UEFI |
|---|---|---|
| **Age** | 1970s | 2000s (modern) |
| **Interface** | Text only | GUI possible |
| **Boot target** | MBR (512 bytes, 446 for code) | GPT partition table |
| **Max disk size** | 2TB | 9.4 ZB |
| **Secure Boot** | No | Yes (prevents bootkit malware) |
| **Boot speed** | Slower | Faster (parallel initialization) |
| **Architecture** | 16-bit real mode | 32/64-bit |

---

## 8. Numericals — Solved

### 📝 Numerical 1 — System Call Overhead

**Problem:**
A program makes 1000 system calls. Each system call involves:
- User → Kernel mode switch: 1 μs
- Kernel executes: 5 μs
- Kernel → User mode switch: 1 μs
- The same operation in user space would take: 2 μs

Calculate: (a) Total time with system calls, (b) Time if no system call overhead, (c) Overhead ratio.

**Solution:**
```
(a) Time per system call = 1 + 5 + 1 = 7 μs
    Total = 1000 × 7 = 7000 μs = 7 ms

(b) Time if user space (no overhead) = 1000 × 2 = 2000 μs = 2 ms
    (kernel still does 5μs work, no mode switch)
    Actually: kernel work still needed so = 1000 × (5+2) = 7000... 
    
    Better interpretation: If operation could be done entirely in user space:
    Total = 1000 × 2 = 2000 μs

(c) Overhead = (7000 - 2000) / 2000 × 100 = 250% overhead
    Mode switch overhead alone = 1000 × (1+1) = 2000 μs out of 7000 μs = 28.6%
```

---

### 📝 Numerical 2 — Process Tree with fork()

**Problem:** What is the output of this program? How many times is "NIT" printed?

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("Start\n");
    fork();                        // Line A
    printf("NIT\n");
    if (fork() == 0) {            // Line B
        printf("Trichy\n");
    }
    printf("End\n");
    return 0;
}
```

**Solution:**
```
Before Line A: 1 process (P1)
After Line A (fork):  2 processes (P1, P2)

Both P1 and P2 execute "printf NIT" → "NIT" printed 2 times

Line B: both P1 and P2 call fork()
  P1 forks → creates P3 (child, gets 0) → now P1, P2, P3
  P2 forks → creates P4 (child, gets 0) → now P1, P2, P3, P4

After Line B:
  P3 (child of P1): fork() returned 0 → prints "Trichy", then "End"
  P4 (child of P2): fork() returned 0 → prints "Trichy", then "End"
  P1 (parent of P3): fork() returned P3's PID (≠0) → skips if, prints "End"
  P2 (parent of P4): fork() returned P4's PID (≠0) → skips if, prints "End"

Full output (order may vary due to scheduling):
  Start       ← 1 time (before any fork)
  NIT         ← 2 times (P1 and P2)
  Trichy      ← 2 times (P3 and P4)
  End         ← 4 times (P1, P2, P3, P4)

"NIT" is printed: 2 times ✓
Total lines printed: 1 + 2 + 2 + 4 = 9
```

---

### 📝 Numerical 3 — Interrupt Handling Time

**Problem:**
A CPU runs at 1 GHz. An I/O device generates interrupts at 10,000 interrupts/second.
Each interrupt handler takes 500 clock cycles.
What percentage of CPU time is spent handling interrupts?

**Solution:**
```
CPU speed: 1 GHz = 10^9 cycles/second
Interrupts per second: 10,000
Cycles per interrupt handler: 500

Cycles spent on interrupts per second = 10,000 × 500 = 5,000,000 cycles/sec
Total cycles per second = 10^9 = 1,000,000,000 cycles/sec

Percentage = (5,000,000 / 1,000,000,000) × 100 = 0.5%

Answer: 0.5% of CPU time spent on interrupt handling.
```

---

### 📝 Numerical 4 — DMA vs Programmed I/O

**Problem:**
A disk transfers data at 10 MB/s. CPU can execute 10^9 instructions/second.
With Programmed I/O: CPU executes 4 instructions per byte transferred.
With DMA: CPU executes 1000 instructions for setup + 500 for interrupt handling.
Transfer size: 1 MB

Calculate CPU instructions wasted for each method.

**Solution:**
```
Transfer size = 1 MB = 1,048,576 bytes ≈ 10^6 bytes

Programmed I/O:
  Instructions = 4 × 10^6 = 4,000,000 instructions
  CPU busy time = 4,000,000 / 10^9 = 0.004 seconds = 4 ms
  During this 4ms, CPU can do NOTHING else

DMA:
  Instructions = 1000 (setup) + 500 (interrupt) = 1500 instructions
  CPU busy time = 1500 / 10^9 ≈ 0.0000015 seconds = 1.5 μs
  During remaining (4ms - 1.5μs) CPU is FREE

DMA saves: 4,000,000 - 1500 = 3,998,500 instructions worth of CPU time
Speedup ratio: 4,000,000 / 1,500 ≈ 2667× more efficient
```

---

### 📝 Numerical 5 — Effective Memory Access Time with Cache

**Problem:**
A system has:
- Cache access time: 10 ns
- RAM access time: 100 ns
- Cache hit rate: 90%

Calculate Effective Memory Access Time (EMAT).

**Solution:**
```
EMAT = hit_rate × cache_time + (1 - hit_rate) × (cache_time + RAM_time)

Note: On a miss, you still check cache first, then go to RAM.

EMAT = 0.9 × 10 + 0.1 × (10 + 100)
     = 9 + 0.1 × 110
     = 9 + 11
     = 20 ns

Without cache: every access = 100 ns
With cache: effective access = 20 ns
Speedup = 100 / 20 = 5×

Alternative formula (if miss = RAM only, no cache check counted):
EMAT = 0.9 × 10 + 0.1 × 100 = 9 + 10 = 19 ns
(Both formulas appear in different textbooks — know both)
```

---

### 📝 Numerical 6 — Degree of Multiprogramming

**Problem:**
A process spends 40% of its time doing I/O (waiting) and 60% using CPU.
Memory can hold 4 processes simultaneously (degree of multiprogramming = 4).
What is approximate CPU utilization?

**Solution:**
```
If one process has probability p of doing I/O = 0.4
Probability it's NOT using CPU = 0.4 (doing I/O)
Probability it IS using CPU = 0.6

With n processes, probability ALL are doing I/O simultaneously:
P(CPU idle) = p^n = (0.4)^4 = 0.0256

CPU utilization = 1 - P(all doing I/O) = 1 - (0.4)^4 = 1 - 0.0256 = 0.9744 ≈ 97.4%

Compare:
  n=1: CPU utilization = 1 - 0.4 = 60%
  n=2: 1 - 0.4² = 1 - 0.16 = 84%
  n=3: 1 - 0.4³ = 1 - 0.064 = 93.6%
  n=4: 1 - 0.4⁴ = 1 - 0.0256 = 97.4%  ← our answer

This shows why multiprogramming dramatically improves CPU utilization!
```

---

## 9. MCQ Traps & Exam Q&A

### ⚠️ THE TRAPS — Read Every One

---

**TRAP 1: "OS is just software"**
❌ Wrong framing — OS is BOTH software AND uses hardware features (mode bit, timer, MMU, TLB) that are specifically designed to support it. The OS is a hardware-software partnership.

---

**TRAP 2: Monolithic kernel = unstructured**
❌ Linux is monolithic but HIGHLY structured internally with subsystems.
Monolithic means all services run in kernel space — NOT that it's a mess.

---

**TRAP 3: Microkernel is always better**
❌ Microkernel is more stable but SLOWER due to IPC overhead.
For performance-critical systems, monolithic wins.
Linux chose monolithic for performance — it's not wrong.

---

**TRAP 4: fork() creates a completely separate copy immediately**
❌ Modern fork() uses Copy-On-Write (COW).
Pages are SHARED between parent and child, marked read-only.
Only when one process WRITES does it get its own copy.
Fork is fast — O(1) pages copied at fork time.

---

**TRAP 5: exec() creates a new process**
❌ exec() does NOT create a new process.
It REPLACES the current process's code, data, heap, stack with the new program.
Same PID. No new process. Combined fork()+exec() = new process running new program.

---

**TRAP 6: Interrupts and exceptions are the same**
❌ 
- Interrupt: EXTERNAL, asynchronous, from hardware device
- Exception: INTERNAL, synchronous, from CPU executing instruction
  - Fault: can be fixed, restart instruction (page fault)
  - Trap: intentional, continue next instruction (system call)
  - Abort: unrecoverable (hardware failure, double fault)

---

**TRAP 7: DMA removes CPU from I/O completely**
❌ CPU is involved in:
1. Setting up DMA (telling it what to transfer, where)
2. Handling the interrupt when DMA finishes
CPU is NOT involved in the actual data transfer itself.

---

**TRAP 8: Higher degree of multiprogramming always helps**
❌ Beyond a point it causes THRASHING — processes compete for memory, spend more time page-faulting than executing. Utilization DROPS.
Graph: utilization increases with multiprogramming, then peaks, then crashes (thrashing).

---

**TRAP 9: UEFI replaced BIOS completely**
❌ Modern systems use UEFI, but UEFI has a BIOS compatibility mode (CSM - Compatibility Support Module) to boot older OSes.

---

**TRAP 10: Mode bit = 0 means user mode**
❌ It's the OPPOSITE on most architectures:
Mode bit = 0 → Kernel mode (privileged)
Mode bit = 1 → User mode (restricted)
(Some exam questions flip this — always check context)

---

### ⚡ Quick Fire Q&A — MCQ Style

| Question | Answer |
|---|---|
| Which OS type guarantees response before deadline? | Real-Time OS |
| Which kernel type has device drivers in user space? | Microkernel |
| What does POST stand for? | Power-On Self Test |
| What is stored in MBR? | Bootloader (446B) + Partition table (64B) + Signature (2B) |
| Can kernel mode code execute user mode instructions? | Yes — kernel can do everything user can + more |
| What triggers a mode switch from user to kernel? | System call, interrupt, or exception |
| Which I/O method wastes most CPU time? | Programmed I/O (polling/busy-wait) |
| What is the purpose of the Interrupt Vector Table? | Maps interrupt numbers to handler function addresses |
| Which is faster: kernel→user or user→kernel switch? | They take the same time (same hardware mechanism) |
| What is spooling used for? | Managing slow devices (like printers) that can't interleave jobs |
| PID of init/systemd process? | Always PID 1 |
| Can two processes have same PID simultaneously? | No — PID is unique system-wide at any moment |
| What happens to child's file descriptors after fork()? | Inherited from parent (same open files) |
| After exec(), what is preserved? | PID, open file descriptors, environment variables (not code/stack/heap) |
| What is a zombie process? | Process finished but parent hasn't called wait() — PCB still in table |
| What is an orphan process? | Process whose parent died — adopted by PID 1 (init) |
| Name 2 advantages of UEFI over BIOS | Faster boot, supports >2TB disks, Secure Boot, GUI |
| What is double buffering? | Using two buffers alternately — fill one while OS processes other |
| Which I/O method uses one interrupt per block? | DMA |
| What is the mode bit stored in? | PSW (Program Status Word) / FLAGS register |

---

### 🔥 5 Questions Likely in CoreTex Round 1

**Q1.** A program calls `fork()` three times with no conditionals. How many processes exist?
**A:** 2³ = **8 processes**

**Q2.** Which of the following is NOT a privileged instruction?
(a) Halt  (b) I/O instructions  (c) ADD register  (d) Disable interrupts
**A:** **(c) ADD register** — arithmetic is not privileged

**Q3.** In interrupt-driven I/O, when does the CPU get interrupted?
**A:** When the **I/O operation completes** (device sends interrupt signal)

**Q4.** What is the key difference between a trap and a fault?
**A:** Trap → intentional, OS returns to **next** instruction. Fault → unintentional, OS returns to **same** instruction to retry.

**Q5.** A process is in the "waiting" state. What event will move it to "ready"?
**A:** Completion of the I/O operation it was waiting for (or the event it was blocked on)

---

### 📋 Formula Summary

```
CPU Utilization (multiprogramming) = 1 - p^n
  where p = I/O wait fraction, n = degree of multiprogramming

EMAT (with cache) = h × Tc + (1-h) × (Tc + Tm)
  where h = hit rate, Tc = cache time, Tm = main memory time

EMAT (with TLB) = h × (Tt + Tm) + (1-h) × (Tt + 2×Tm)
  where h = TLB hit rate, Tt = TLB access time, Tm = memory access time

Mode switch overhead per syscall = 2 × mode_switch_time
  (one user→kernel + one kernel→user)

DMA efficiency = (PIO_instructions - DMA_instructions) / PIO_instructions × 100%

Interrupt CPU% = (interrupts/sec × cycles/interrupt) / (total cycles/sec) × 100%
```

---