# What Is a Process?

A process is an instance of a running program. It is more than the code itself: it includes the resources and context required while the program runs, such as:

- Virtual address space
- Text segment
- Data segment
- Heap
- Stack
- Open file descriptors
- Signal-handling state
- Current working directory
- Register context
- Permissions, environment variables, and more

**A process is a basic unit of operating-system resource allocation.**

## Core Characteristics

### Independent Address Space

Different processes generally have isolated virtual address spaces:

- A process cannot directly access another process's memory.
- A crash usually does not directly corrupt another process's address space.
- Isolation improves security.

### Resource Owner

A process normally owns resources such as:
- Memory mappings
- File-descriptor table
- Sockets
- Signal handlers
- Child-process information

### Container for Scheduled Entities

The CPU generally schedules **threads**, but every **thread must belong to a process**.

---

# What Is a Thread?

A thread is an execution flow inside a process. A process can contain multiple threads. They share most process resources, but each has an independent execution context.

**A thread is a basic unit of operating-system scheduling.**

## Core Characteristics

### Shared
- Text segment
- Global variables
- Heap
- Open file descriptors
- Process address space
- Signal disposition, much of which is process-wide

### Independent
- Register context
- Program Counter (PC)
- Stack
- Thread-Local Storage (TLS)
- Scheduling state
- `errno`, which is usually thread-local

---

# Process vs. Thread

| Dimension | Process | Thread |
|---|---|---|
| Definition | Resource-allocation unit | CPU-scheduling unit |
| Address space | Independent | Shared within a process |
| Memory isolation | Strong | Weak |
| Creation cost | High | Low |
| Switch cost | Usually higher | Usually lower |
| Communication | IPC and kernel mechanisms | Direct shared memory |
| Crash impact | Usually does not affect other processes | One failed thread may bring down the entire process |

---

# Concurrency vs. Parallelism

* **Parallelism** &rarr; use multiple CPU cores to process multiple tasks at the same time.
* **Concurrency** &rarr; use switching on one core to make multiple tasks appear to run; only one task is actually executing at any instant on that core.

---

# Threads and Processes from the Address-Space Perspective

A process's virtual address space may look like this:
- Code, data, heap, and `mmap` regions are shared.
- Each thread has its own stack.

```text
+---------------------+ <-------
| Code / Text Segment |     |
+---------------------+     |
| Data Segment        |     |
+---------------------+     |
| BSS                 | shared by threads
+---------------------+     |
| Heap                |     |
+---------------------+     |
| mmap region         |     |
+---------------------+ <-------
| Thread Stack A      |
+---------------------+
| Thread Stack B      |
+---------------------+
| Thread Stack C      |
+---------------------+
```

This layout means:
- Sharing data between threads is convenient.
- Data races are also easy to create.
- Local variables usually live on a thread's stack.
- Global variables and heap objects are shared.

---

# Context Switch

When the CPU switches between execution entities, it saves and restores context such as:

- General-purpose registers
- Program counter
- Stack pointer
- Scheduling information
- TLB-related state

During a thread switch within one process:

- The address space usually does not change.
- Page tables may not need to change.
- TLB-flush cost can be lower.
- Shared-resource mappings do not need to be rebuilt.

During a process switch:

- The CPU changes to another address space.
- TLB and cache interference is more likely.
- The cost is usually higher.

See [Context Switch](./context-switch.md) for details.

---

# Advantages and Costs of Multithreading

## Advantages
* Higher concurrency &rarr; one process can execute multiple tasks.
* Lower communication cost &rarr; threads share memory, so passing data is easier than IPC.
* Hides I/O latency &rarr; while one thread blocks on I/O, other threads can continue.
* Better use of multiple cores &rarr; threads can run in parallel on multiple CPU cores.

## Costs
* Complex synchronization &rarr; shared memory requires handling races, deadlocks, and memory ordering.
* One failed thread can bring down the process &rarr; a wild pointer or out-of-bounds write can corrupt memory used by other threads.
* Debugging is more difficult.

---

# Advantages and Costs of Multiprocessing

## Advantages
* Strong isolation &rarr; one process crash does not necessarily affect others.
* Better security &rarr; natural address-space isolation and clear permission boundaries.
* Better fault isolation.

## Costs
* Heavier communication &rarr; requires IPC such as sockets, shared memory, or message queues.
* Higher creation and switching costs.
* More complex resource copying and management.

---

# Thread Synchronization

Threads need synchronization because they share an address space and can access the same memory concurrently. Interleaved execution can cause lost updates.

Synchronization primitives include:

- Mutex
- Spinlock
- Read-write lock
- Semaphore
- Condition variable
- Atomic operation

See [Synchronization Primitives](./synchronization-primitives.md).

---

# Advanced Thread Topics

## User Thread vs. Kernel Thread

| Type | Management | Advantages | Disadvantages |
|---|---|---|---|
| User-level thread | Managed by a user-mode library; the kernel may not know each thread exists | Fast switching; user-controlled | One blocking system call may affect the whole process; the kernel cannot directly schedule different user threads on different cores |
| Kernel-level thread | Directly managed and scheduled by the kernel | True parallelism; natural blocking awareness; better multicore support | More kernel-management overhead |

## Thread-Local Storage (TLS)

Sometimes every thread needs its own copy of data while the code should remain as convenient as a global variable:

```cpp
thread_local int x = 0;
```

Each thread sees its own copy of `x`.

## Thread Stack

Every thread has its own stack for local variables, return addresses, and call frames.

* Stack size is limited, and many threads consume substantial virtual memory.
* Deep recursion can overflow the stack.
* Objects on a thread's stack cannot be referenced freely from another thread.

## False Sharing

Two threads may access different variables that happen to occupy the same cache line, causing serious performance problems.

> [!NOTE]
> A **cache line** is the smallest unit transferred between CPU cache and memory. Reading one variable loads the entire surrounding cache-line region.

If two threads modify `x` and `y` on the same cache line, hardware cache-coherence protocols can repeatedly invalidate that line even though the variables are not logically shared.

Common solutions:

- Padding
- Cache-line alignment
- Separating hot write data

## Memory Model

A central multithreading question is when a write by one thread becomes visible to another. Code that appears sequential may not be observed in that order by other threads.

## Lock vs. Lock-Free

Synchronization does not always require locks. Lock-free algorithms commonly use CAS (compare-and-swap).

---

# Advanced Process Topics

## Copy-on-Write (COW)

Unix and Linux commonly create processes with `fork()`. The child initially appears to be a copy of the parent, but modern systems do not immediately copy all memory. With Copy-on-Write, parent and child share physical pages as long as they only read. When either writes, that page is copied.

## Zombie Process

After a child exits, the kernel retains a small amount of exit information, such as its exit status, until the parent reaps it. If the parent never calls `wait()`, the child becomes a zombie process and continues to occupy a process-table entry.

> [!NOTE]
> Process creation allocates a process-table entry containing key information such as PID, process state, exit code, and parent process.

## Orphan Process

If a parent exits while its child is still running, the child becomes an orphan. It is normally adopted by a new parent such as `init` or `systemd`.

## Inter-Process Communication (IPC)

Process address spaces are isolated, so processes need IPC:
- pipe
- named pipe (FIFO)
- socket
- shared memory
- message queue
- semaphore
- signal

See [IPC](./ipc.md).

---

# Control Block

The OS maintains kernel data structures to manage running entities. Since an OS manages many processes and threads, it needs a place to store each object's information and locate its state during a switch.

## PCB (Process Control Block)

A PCB describes and manages a process. The kernel normally creates one when a process is created and keeps it while the process is alive. It is reclaimed after the process exits.

A PCB generally stores:
* Process identity &rarr; PID, parent process, process name.
* Process state &rarr; New, Ready, Running, Blocked, Terminated.
* Scheduling information &rarr; priority, time slice, and scheduling policy.
* Memory information &rarr; page-table pointer, virtual address space, code segment, and other regions.

## TCB (Thread Control Block)

A TCB describes and manages a thread. If a process manages multiple threads, its PCB is associated with multiple TCBs.

A TCB generally stores:
* Thread identity &rarr; TID and thread name.
* Thread state &rarr; Ready, Running, Blocked, Sleep, Terminated.
* CPU context:
  * [Program Counter (PC)](./registers.md#program-counter)
  * [Stack Pointer (SP)](./registers.md#stack-pointer)
  * [General-purpose registers](./registers.md#general-purpose-registers)
  * [Status / flags register](./registers.md#status--flags-register)
  * [Floating-point / SIMD registers](./registers.md#floating-point--simd--vector-registers)
* Scheduling information &rarr; priority, time slice, and scheduling policy.
* Thread-local storage.
* Thread-stack information &rarr; threads normally have their own stacks, so the TCB records stack-related information.
