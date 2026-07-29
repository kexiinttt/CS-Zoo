# What Is Context Switching?

Context switching is the set of state-saving and state-restoring operations the operating system performs when it pauses the current execution entity and resumes another one.

> [!NOTE]
> An execution entity can be:
> - a process
> - a thread
> - a kernel or user thread
> - an interrupt-handling context

---

# Why Is Context Switching Needed?

A CPU core can truly execute only one thread at a time. At the user level, however, the system cannot run one thread forever while leaving all other threads unresponsive. The operating system therefore provides:

- **Concurrency**: make multiple tasks appear to run at the same time.
- **Fairness**: prevent one task from monopolizing the CPU.
- **Responsiveness**: run interactive tasks promptly.
- **Isolation**: keep processes independent.

The OS therefore switches continuously between tasks.

---

# What Does the Context Contain?

The state normally saved and restored includes:
* [CPU registers](./registers.md)
* Stack information &rarr; each thread depends on its own stack, so switching threads must also switch to the corresponding stack.
* Address-space information &rarr; switching processes involves switching virtual address spaces.
* Scheduler metadata &rarr; current state (`running`, `ready`, or `blocked`), priority, time slice, and so on.

---

# Process Switching vs. Thread Switching

## Thread Switching

Thread switching usually manages:

- Registers
- Stack
- Scheduling state

If two threads belong to the same process, they share:

- Address space
- Page tables
- Open file descriptors
- Most process resources

Thread switching is therefore usually cheaper than process switching.

## Process Switching

In addition to the state managed for a thread, process switching may require:

- Switching page tables
- Changing address spaces
- Affecting the TLB
- Greater cache and locality loss

---

# When Does Context Switching Happen?

## Time Slice Exhaustion

A preemptive scheduler gives a thread a time slice. When it expires, the scheduler may assign the CPU to another thread.

## A Thread Blocks Voluntarily

For example, a thread may execute a blocking operation, enter the blocked state, and let another runnable thread use the CPU:

- `sleep`
- `read` waiting for I/O
- `futex`
- `pthread_mutex_lock` when the mutex is held
- `wait`
- `epoll_wait`

## A Higher-Priority Task Arrives

The current thread may be preempted when a higher-priority thread becomes runnable.

## Interrupt or Exception

- Timer interrupt
- I/O interrupt
- Page fault
- Scheduling after a system call enters the kernel

---

# Cost of a Context Switch

* Saving and restoring registers
* Kernel scheduling-path overhead &rarr; entering the kernel, making a scheduling decision, operating on queues, and updating state all cost time.
* Cache misses &rarr; the new thread uses different data, so the old cache contents may miss.
* Address-space changes during process switching
* Effects on the CPU pipeline and branch predictor
* Broken memory locality

---

# Voluntary vs. Involuntary Context Switch

In a **voluntary context switch**, the thread willingly gives up the CPU, commonly because it is:
- waiting for a lock
- waiting for I/O
- calling `sleep`
- making a blocking system call

In an **involuntary context switch**, the thread does not want to yield but is preempted by the scheduler, for example because:

- its time slice expires
- a higher-priority task preempts it

---

# CPU-Bound and I/O-Bound

## CPU-Bound

The task mainly consumes CPU computation, such as:

- Compression
- Encryption
- Image processing
- Numerical computation

These tasks are more likely to consume their full time slice and be preempted.

## I/O-Bound

The task mainly waits for a device or network, such as:

- Disk reads and writes
- Network requests
- Waiting for a database connection

These tasks block voluntarily often, so context switches also occur frequently.

---

# Reducing Unnecessary Context Switches

## Reduce Lock Contention

- Shorten critical sections.
- Reduce shared state.
- Use better concurrency structures.
- Shard the data.

## Control the Thread Count

- Do not blindly create one thread per request.
- Use a thread pool.
- Match the number of threads to the CPU and workload.

## Reduce Blocking I/O

- Non-blocking I/O
- `epoll`
- Asynchronous models
- Batching

## Improve Locality

- CPU affinity
- Avoid frequent migration between cores
- NUMA-aware allocation

## Choose the Right Synchronization Primitive

- Spinlock
- Mutex
- Read-write lock
- Lock-free or wait-free structure
