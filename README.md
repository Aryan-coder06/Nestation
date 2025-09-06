# Nestation

### BEGINNING OF A WORKSTATION BALANCED WITH MULTI COMMODITY FEATURES


# 🧑‍💻 Operating Systems — Unit 1 (Introduction & System Calls)
*A compact, exam-ready, beautifully structured guide*

---

## How to use this file
- Read each section’s **What / Why / How** in order.
- Copy the **Exam Nuggets** verbatim when short on time.
- Keep tables to **keywords only** (no long sentences).

---

## 1) System Calls — the user→kernel gateway

> **What**: System calls are controlled entry points from **user mode** to **kernel mode** to request privileged services (files, processes, memory, devices, IPC).  
> **Why**: Apps can’t touch hardware directly. Syscalls enforce **protection**, **portability**, and **accounting**.  
> **How**: The process executes a special **trap** instruction → CPU flips to **kernel mode** → kernel validates args, performs work, returns a **status/errno** and possibly data.

### 1.1 Categories (with top examples)
- **Process control**: `fork()`, `execve()`, `exit()`, `waitpid()`, `kill()`  
- **File management**: `open()`, `read()`, `write()`, `lseek()`, `close()`  
- **Device management**: `ioctl()`, async I/O, DMA setup (via kernel drivers)  
- **Memory management**: `mmap()`, `munmap()`, `brk()/sbrk()`  
- **Information maintenance**: `getpid()`, `gettimeofday()`, `uname()`  
- **IPC**: `pipe()`, `socket()`, `shmget()/shmat()`, `msgget()/msgrcv()`, `semget()/semop()`

### 1.2 Syscall flow (ASCII sketch)
```
User space                      Kernel space
----------                      ------------
app()                           syscall_table[no] -> handler()
  |  prepare args → trap  ---->  validate & copy args
  |                              check perms/caps
  |                              do work (FS, MM, IPC, I/O)
  |  copy results  <---- ret ---- set errno / return value
```

### 1.3 Mini examples
**Create a process tree (UNIX)**
```c
pid_t pid = fork();             // parent gets child's pid; child gets 0
if (pid == 0) {                 // child
  execl("/bin/ls","ls","-l",NULL); // replace address space
  _exit(127);                   // exec failed
} else {
  int status; waitpid(pid,&status,0); // parent waits
}
```
**Read file → write to stdout**
```c
int fd = open("in.txt", O_RDONLY);
char buf[4096]; ssize_t n;
while ((n = read(fd, buf, sizeof buf)) > 0) write(1, buf, n);
close(fd);
```

### 1.4 Gotchas
- Passing **invalid pointers** or **user buffers** not pinned → `EFAULT`.
- **Partial I/O** is normal (short reads/writes); always loop.
- **Blocking**: many syscalls block; prefer non-blocking or async when needed.
- **ABI vs API**: **glibc** wrappers != raw syscall numbers; don’t hard-code numbers.

### 1.5 Exam Nuggets
- Always say: “**trap** to kernel, **mode switch**, **validate**, **perform**, **copy back**, **return errno**.”
- Mention categories with 1–2 concrete calls each.
- Draw the user→kernel flow if asked for “system call mechanism.”

---

## 2) Hardware Requirements — protection, context switching, privileged mode

> **What**: CPU + MMU features the OS relies on to isolate processes and safely multiplex hardware.  
> **Why**: Without hardware help, one buggy app could corrupt everything.  
> **How**: **Dual mode** (user/kernel), **privileged instructions**, **memory protection** (base/limit or paging), **timer interrupts**, **context switch** support.

### 2.1 Dual mode & privileged instructions
- **Kernel mode**: can execute **privileged** ops (I/O port access, page-table updates, disable/enable interrupts, set timer, load MMU registers, halt).  
- **User mode**: normal instructions only; privileged ones **trap**.  
- **Timer**: preemption heartbeat; user code **must not** set it.

**Keywords table (no long sentences):**

| Item | Kernel-only examples |
|---|---|
| Privileged ops | I/O, set timer, change page tables, int mask |
| Triggers of kernel entry | Syscall (trap), interrupt, fault |
| Return to user | `iret`/`sysret`-style instruction |

### 2.2 Memory protection
Two classic mechanisms:

**(a) Base/Limit (relocation+bound)**  
- Logical address `LA` → physical `PA = BASE + LA`, allowed iff `LA < LIMIT`.  
- Fast, simple; per-process registers loaded on switch.

**(b) Paging + TLB**  
- Virtual pages → physical frames via **page tables**.  
- **TLB** caches translations; miss → page table walk.  
- Easy sharing/protection per page; internal fragmentation possible.

### 2.3 Context switch (step-by-step)
1. **Interrupt or scheduler decision** arrives.  
2. Save running thread’s **PC + registers** into **TCB/PCB**.  
3. Maybe save/restore **FPU** or **SIMD** state (lazy save).  
4. Load next runnable’s registers; **switch address space** (CR3).  
5. Return from interrupt to new context; warm up TLB/caches.

**Cost**: pure overhead (no user progress). Keep switches **short & infrequent**.

### 2.4 Exam Nuggets
- State: user vs kernel; list **3 examples** of privileged ops.  
- Draw base/limit picture and write `PA = BASE + LA`.  
- Enumerate **5** switch steps (save PC/regs → load next → ret).

---

## 3) Types of Operating Systems (keywords + where they shine)

> **What**: Deployment styles for different workloads.  
> **Why**: Picking the right OS model maximizes throughput, responsiveness, or determinism.  
> **How**: Organize work as batches, time-shared quanta, parallel CPUs, or hard-deadline tasks; scale across machines when needed.

### 3.1 Quick glossary (keywords only)

| Type | Core idea | Example | When to use |
|---|---|---|---|
| **Batch** | Offline job queues | IBM JCL | Long, non-interactive runs |
| **Multiprogramming** | Keep CPU busy | Early UNIX | Mix CPU/I/O jobs |
| **Time-sharing** | Small quanta per user | Unix, Linux | Interactive terminals |
| **Multiprocessing** | >1 CPU/core | Linux SMP | Parallel compute |
| **Real-Time (RTOS)** | Deadlines first | VxWorks, FreeRTOS | Control/embedded |
| **Distributed** | Many nodes cooperate | Kubernetes + Linux | Scalability/fault-tolerance |
| **Network OS** | Share resources over LAN | Windows Server | File/print, auth |
| **Mobile/Embedded** | Constrained devices | Android, RTOS | Battery/footprint |

### 3.2 Notes & contrasts (bullets)
- **Batch vs Time-sharing**: throughput vs responsiveness.  
- **Multiprogramming** increases CPU utilization by overlapping I/O-bound and CPU-bound jobs.  
- **Real-time**: *hard* RT ⇒ missing a deadline is catastrophic; *soft* RT ⇒ occasional misses tolerable.  
- **Distributed**: embrace **latency**, **partial failure**, **consistency** trade-offs.  
- **Mobile/Embedded**: emphasize **power**, **footprint**, **predictability**.

### 3.3 Exam Nuggets
- “List & explain any **four** OS types with one real example each.”  
- If “RTOS challenges”: say **WCET**, **bounded latency**, **priority inversion** (fix: inheritance/ceiling), **admission control**.

---

## 4) Glueing it all together (mental model)
1. **Boot** loads the **kernel**; the **init** process starts services.  
2. **User programs** make **system calls** to request services.  
3. **Hardware** enforces isolation (dual mode, MMU).  
4. The **scheduler** context-switches among ready threads.  
5. **I/O & interrupts** drive progress; kernel returns to user when safe.

---

## 5) One-minute revision (copy-paste friendly)
- “Syscalls = trap to kernel → validate → privileged work → return `errno`.”  
- “Protection = user/kernel + privileged ops + MMU.”  
- “Context switch = save/restore + (maybe) CR3 change + TLB warmup.”  
- “OS types = batch, time-sharing, real-time, distributed, mobile.”

---

### Further reading (optional)
- Any standard OS text (Silberschatz/Galvin, Tanenbaum).  
- `man` pages for: `fork`, `execve`, `waitpid`, `open`, `read`, `write`, `mmap`, `pipe`, `socket`.
