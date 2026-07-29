# IPC (Inter-Process Communication)

IPC means **Inter-Process Communication**.

Different processes have independent virtual address spaces. One process usually cannot directly access another process's memory. When processes need to exchange data, synchronize state, or cooperate on a task, they need an IPC mechanism.

> IPC is a group of operating-system mechanisms for safe and controlled communication between processes.

---

# Why Is IPC Needed?

Process isolation improves security and stability but also creates a communication problem.

- A Web Server process forwards requests to a Worker process.
- Multiple processes share state or a cache.
- A main process controls several child processes.
- Local services exchange data.
- Services written in different languages cooperate on one machine.
- Browsers, databases, and backend services communicate internally.

```text
Client Request
      |
      v
Main Process  ---- IPC ----> Worker Process
      |                         |
      | <---- IPC Response -----|
      v
Return Response
```

---

# IPC vs. Thread Communication

Threads belong to one process and normally share an address space; processes are isolated from one another.

| Aspect | Thread communication | IPC |
|---|---|---|
| Address space | Usually shared | Isolated by default |
| Data sharing | Directly access shared variables | Cannot directly access the other process's memory |
| Security | Lower; data races are easy | Higher; stronger isolation |
| Cost | Usually lower | Usually higher |
| Common mechanisms | Mutex, condition variable, atomic | Pipe, socket, shared memory, message queue |

---

# Common IPC Methods

## Pipe

A pipe is a classic IPC method, usually used for **one-way communication between related processes**.

- Transfers data as a byte stream.
- Is usually one-way.
- Suits simple data-stream transfer.
- Common in Unix/Linux command-line pipelines.

```bash
cat file.txt | grep "error"
```

Here `cat` and `grep` are two processes connected by a pipe.

Suitable scenarios:

- Simple parent-child communication
- Chaining command-line tools
- Streaming text processing

Limitations:

- Usually limited to one machine.
- The application must define its own data encoding.
- Not suitable for complex bidirectional communication.

## Named Pipe / FIFO

A normal pipe is usually used by related processes such as a parent and child. A Named Pipe has a file-system path, so unrelated processes can use it.

Characteristics:

- Has a name and exists in the file system.
- Can be opened by unrelated processes.
- Still mainly provides byte-stream communication.

Suitable for simple communication and lightweight transfer between local processes.

## Message Queue

A message queue communicates in **messages** rather than an undifferentiated byte stream.

Characteristics:

- Preserves message boundaries.
- Can support asynchronous communication.
- Decouples senders and receivers.
- May support priority and persistence depending on the implementation.

Suitable for producer-consumer systems, asynchronous task processing, and inter-process event notification.

```text
Producer Process ---> Message Queue ---> Consumer Process
```

Limitations:

- Potentially lower throughput than shared memory.
- Requires serialization and deserialization.
- The queue can become a bottleneck.

## Shared Memory

Shared memory is a high-performance IPC method. Multiple processes map the same physical memory into their virtual address spaces and directly read and write the same data.

Characteristics:

- Very high performance.
- Avoids large amounts of copying.
- Suitable for large data transfers.
- Requires extra synchronization to prevent race conditions.

```text
Process A Virtual Memory ---> Shared Memory Region <--- Process B Virtual Memory
```

Suitable for high-frequency trading, multimedia processing, database buffers and caches, and large local transfers.

Limitations:

- Higher programming complexity.
- Requires mutexes, semaphores, atomics, or other synchronization.
- One process corrupting the data can affect other processes.

## Semaphore

A semaphore is generally not used to transfer data. It provides **synchronization and mutual exclusion**.

Common uses:

- Controlling process access to a shared resource.
- Coordinating access to shared memory.
- Synchronizing producers and consumers.

```text
Shared Memory stores data
Semaphore controls when data is ready or consumed
```

## Socket

Sockets are common and general-purpose IPC. They work both between processes on one machine and between processes on different machines.

Common types:

- Unix Domain Socket: local process communication.
- TCP Socket: reliable communication across machines.
- UDP Socket: low-latency communication without guaranteed reliability.

Characteristics:

- General-purpose.
- Supports bidirectional communication.
- Can cross machine boundaries.
- Many RPC frameworks are built on sockets.

Suitable for client-server architectures, local services, distributed-service calls, and RPC/HTTP/gRPC communication.

```text
Process A <---- Unix Domain Socket ----> Process B
```

```text
Machine A Process <---- TCP ----> Machine B Process
```

## Memory-Mapped File

A memory-mapped file maps a file into a process's address space. Multiple processes mapping the same file can share data by reading and writing memory.

Characteristics:

- Can optimize file I/O.
- Can share data between processes.
- Data can be persisted to disk.

Suitable for large-file processing, database storage engines, and sharing large data blocks between local processes.

## Signal

A signal is a lightweight process-notification mechanism.

Common uses:

- Notify a process to exit.
- Ask a process to reload configuration.
- Notify a process of an exceptional event.

```bash
kill -SIGTERM <pid>
kill -SIGHUP <pid>
```

Signals are lightweight and suitable for notification, not complex data transfer. Signal handlers must be designed carefully because only limited operations are safe inside them.

---

# Comparison of IPC Methods

| IPC method | Suitable for large data | Bidirectional | Cross-machine | Performance | Complexity | Common use |
|---|---:|---:|---:|---:|---:|---|
| Pipe | Moderate | Usually one-way | No | Medium | Low | Parent-child process, command pipeline |
| Named Pipe | Moderate | Possible | No | Medium | Low | Simple local communication |
| Message Queue | Moderate | Possible | Usually no; implementation-dependent | Medium | Medium | Async tasks, event notification |
| Shared Memory | Very suitable | Yes | No | Very high | High | High-performance local communication |
| Semaphore | Does not transfer data | N/A | No | High | Medium | Synchronization, mutual exclusion |
| Socket | Suitable | Yes | Yes | Medium to high | Medium | RPC, network and local-service communication |
| Memory-Mapped File | Suitable | Yes | No | High | Medium to high | Large files, databases, shared data |
| Signal | Not suitable | No | No | High | Low to medium | Process notification and control |

---

# Core IPC Design Questions

## How Should Data Be Encoded?

Data exchanged between processes needs a format, such as:

- Raw bytes
- JSON
- Protobuf
- FlatBuffers
- A custom binary protocol

Binary protocols or shared-memory structures are common when performance is important.

## Is Synchronization Needed?

When multiple processes access a shared resource, consider:

- Race conditions
- Deadlocks
- Data consistency
- Lock granularity

Shared memory generally must be paired with synchronization.

## Is Reliability Needed?

Consider:

- Can messages be lost?
- What happens if the receiver crashes?
- Is an ACK needed?
- Are retries needed?
- Is persistence needed?

## Is Backpressure Needed?

If the sender is much faster than the receiver, the system needs backpressure:

- Limit queue length.
- Block the sender.
- Drop low-priority messages.
- Degrade processing.
- Return an error or retry signal.

## Is Security Isolation Needed?

IPC also requires permission design:

- Who may connect?
- Who may read or write shared memory?
- Does the socket need authentication?
- Do messages need validation?

---

# Common System-Design Examples

## Master-Worker Model

```text
   Master Process
   |      |      |
   v      v      v
Worker Worker Worker
```

The Master receives tasks and distributes them to Workers through IPC.

Possible mechanisms:

- Pipe
- Unix Domain Socket
- Message Queue

## Shared Memory + Semaphore

```text
Producer Process
      |
      v
Shared Memory Buffer
      |
      v
Consumer Process

Semaphore: controls available slots and ready data
```

This design is common in high-performance producer-consumer systems.

## Local RPC Service

```text
Client Process <---- Unix Domain Socket ----> Local Service Process
```

Local agents, daemons, and sidecars often expose a local API through a Unix Domain Socket.
