# 什么是 Process

进程是一个正在运行的程序的实例。它不仅仅是代码本身，还包括程序运行时所需要的各种资源和上下文，例如：

- 虚拟地址空间
- 代码段（text segment）
- 数据段（data segment）
- 堆（heap）
- 栈（stack）
- 打开的文件描述符
- 信号处理状态
- 当前工作目录
- 寄存器上下文
- 权限、环境变量等

**进程是操作系统资源分配的一个基本单位**。

## 核心特征

### 独立的地址空间

不同进程通常拥有彼此隔离的虚拟地址空间。

- 一个进程不能直接访问另一个进程的内存
- 一个进程崩溃通常不会直接破坏另一个进程的地址空间
- 隔离性好，安全性更高

### 资源拥有者

进程通常拥有自己的一组系统资源，例如：
- 内存映射
- 文件描述符表
- socket
- signal handler
- 子进程信息

### 调度实体的容器

真正被 CPU 调度执行的往往是 **thread**，但 **thread 必须属于某个 process**。

---

# 什么是 Thread

线程是进程中的一个执行流(execution flow)。同一个进程中可以有多个线程，它们共享大部分进程资源，但每个线程有自己独立的执行上下文。

**线程是操作系统进行调度的基本单位**。

## 核心特征

### 共享的内容
- 代码段
- 全局变量
- 堆
- 打开的文件描述符
- 进程地址空间
- signal disposition（很多信号处理配置是进程级别的）

### 独有的内容
- 寄存器上下文
- Program Counter（PC）
- Stack
- Thread Local Storage（TLS）
- 调度状态
- errno（通常是线程局部的）

---

# Process vs Thread

| 维度 | Process | Thread |
|---|---|---|
| 定义 | 资源分配单位 | CPU 调度单位 |
| 地址空间 | 独立 | 同一进程内共享 |
| 内存隔离 | 强 | 弱 |
| 创建开销 | 大 | 小 |
| 切换开销 | 通常更大 | 通常更小 |
| 通信方式 | IPC，需要内核机制 | 直接共享内存 |
| 崩溃影响 | 通常不影响其他进程 | 一个线程出错可能搞挂整个进程 |

---

# Concurrency vs Parallelism

* 并行 (Parallelism) &rarr; 是利用多个CPU核，同时处理多个任务
* 并发 (Concurrency) &rarr; 是单核中利用切换，实现“看起来”处理多个任务，实际上同一时刻只有一个任务被执行

---

# 地址空间角度看 Thread 和 Process

一个进程的虚拟地址空间可能长这样：
- 共享 code/data/heap/mmap
- 每个线程有自己独立的 stack

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
这样的布局导致了：
- 多线程共享数据很方便
- 但也容易 data race
- 局部变量通常在线程栈上
- 全局变量和堆对象会共享

---

# Context Switch

CPU 在不同执行实体之间切换时，需要保存和恢复上下文，例如：

- 通用寄存器
- 程序计数器
- 栈指针
- 调度信息
- TLB 相关影响

Thread 在同一进程内线程切换时：

- 地址空间通常不变
- 页表不一定需要切换
- TLB flush 成本可能更低
- 共享资源不需要重新建立映射关系

Process 切换时：

- 需要切换到另一个地址空间
- 往往更容易引发 TLB/cache 干扰
- 代价通常更高

详细信息请参考[Context Switch](./context-switch.md)。

---

# 多线程的优势与代价

## 优势
* 更高的并发能力 &rarr; 一个进程内可以同时执行多个任务
* 更低的通信成本 &rarr; 线程之间共享内存，传数据比进程间 IPC 更方便
* 隐藏IO延迟 &rarr; 一个线程阻塞在 IO 上时，其他线程仍可继续执行
* 更容易利用多核 &rarr; 多个线程可以被调度到多个 CPU core 上并行执行

## 代价
* 同步复杂 &rarr; 共享内存意味着你必须处理：race condition，deadlock，memory ordering等
* 一个线程出错可能拖垮整个进程 &rarr; 比如野指针写坏共享数据，越界写破坏其他线程使用的内存
* 调试更困难

---

# 多进程的优势与代价

## 优势
* 隔离性强 &rarr; 一个进程崩溃不一定影响其他进程
* 安全性更高 &rarr; 天然地址空间隔离，权限边界清晰
* 更适合 fault isolation

## 代价
* 通信更重 &rarr; 需要 IPC，例如：socket，shared memory，message queue等
* 创建和切换开销更高
* 资源复制与管理更复杂

---

# 线程同步 (synchronization)

线程同步需要加锁，因为多个线程共享地址空间，所以多个线程可能同时访问同一块内存。如果两个线程交错执行，就可能丢失更新。

因此需要同步原语，例如：

- mutex
- spinlock
- rwlock
- semaphore
- condition variable
- atomic

详情参考[同步原语](./synchronization-primitives.md)。

---

# Thread 的高级内容

## User Thread vs Kernel Thread

| 类型 | 管理方式 | 优点 | 缺点 |
| --- | --- | --- | --- |
| User-level Thread | 由用户态线程库管理，内核未必知道每个线程的存在 | 切换快；用户态可控 | 一个线程阻塞 `syscall`，可能整个进程都受影响；内核不能直接把不同用户线程调度到不同核上 |
| Kernel-level Thread | 由内核直接管理和调度 | 真正并行；阻塞感知更自然；多核支持更好 | |

## Thread-Local Storage（TLS）

有些数据希望每个线程都有一份，但写法上像全局变量一样方便，例如：

```cpp
thread_local int x = 0;
```

每个线程访问到的 `x` 都是自己的副本。

## 线程栈（Thread Stack）

每个线程都有自己的栈，用于存放局部变量，返回地址，函数调用帧等。

* 栈大小有限，线程过多会消耗大量虚拟内存
* 深递归容易栈溢出
* 线程栈上的对象不能跨线程随便引用

## False Sharing

多个线程虽然没有访问同一个变量，但如果它们访问的数据恰好落在同一个 cache line 上，也会造成严重性能问题。

> [!NOTE]
> **Cache line**：是 CPU Cache 和内存之间传输数据的最小单位。当 CPU 要读取某个变量时会把这个变量所在的一整段数据一起从内存载入到 Cache。

例如两个线程去修改 `x` 和 `y`，但是他俩恰好在同一个 cache line。虽然语义上没共享变量，但硬件缓存一致性协议会频繁让 cache line 失效。

解决方法通常包括：

- padding
- cache line 对齐
- 分离热点写数据

## Memory Model

多线程中一个非常高级也非常核心的问题是：一个线程写入的数据，另一个线程什么时候能看到？“看起来顺序执行”的代码，在多线程下未必按顺序被其他线程观察到。


## Lock vs Lock-Free

多线程同步不一定只能靠锁。无锁（Lock-free）通常使用 CAS（compare-and-swap）。

---

# Process 的高级内容

## Copy-on-Write（COW）

Unix/Linux 中创建进程常见方式是 `fork()`。`fork()` 后，子进程初始上看起来像是父进程的副本。但现代系统通常不会真的立刻复制所有内存，而是采用：Copy-on-Write：父子进程先共享同一份物理页，只读没问题，一旦某一方尝试写，才真正复制页。

## 僵尸进程（Zombie Process）

当子进程退出后，内核还会保留一小部分退出信息（如 exit status），等待父进程来回收。如果父进程一直不 `wait()`，子进程就会变成 zombie process，一直占用进程表项（内存）。

> [!NOTE]
> 进程创建时会分配一个进程表项，记录进程的关键信息，比如PID，进程状态，退出码，父进程等。

## 孤儿进程（Orphan Process）

如果父进程先退出，子进程还在运行，那么子进程会变成孤儿进程。通常会被新的父进程（如 init/systemd）接管。

## 进程间通信（IPC）

由于进程地址空间隔离，所以需要 IPC：
- pipe
- named pipe (FIFO)
- socket
- shared memory
- message queue
- semaphore
- signal

详情参考[IPC](./ipc.md)。

---

# Control Block

操作系统为了管理“正在运行的实体”而维护的一份内核数据结构。

因为一个OS可以管理很多Process/Thread，所以需要有一个地方存储每一个对象的信息，这样在切换执行的时候能找到对应的状态。

## PCB (Process Control Block)

PCB 是操作系统用来描述和管理一个进程的数据结构。

一个进程被创建出来后，内核通常就会给它建立一个 PCB。只要进程还活着，PCB 通常就存在。进程结束后，PCB 才会被回收。

一般而言PCB会保存下列内容：
* 进程标识 &rarr; PID，父进程，进程名
* 进程状态 &rarr; New / Ready / Running / Blocked / Terminated
* 调度信息 &rarr; 优先级，时间片，调度策略
* 内存信息 &rarr; 页表指针，虚拟地址空间，代码段等各种区域信息

## TCB (Thread Control Block)

TCB 是操作系统用来描述和管理一个线程的数据结构。

如果一个进程管理多个线程，那么一个PCB会关联多个TCB。

一般而言TCB会保存下列内容：
* 线程标识 &rarr; TID，线程名
* 线程状态 &rarr; Ready / Running / Blocked / Sleep / Terminated
* CPU上下文
    * [程序计数器 PC](./registers.md#program-counter)
    * [栈指针 SP](./registers.md#stack-pointer)
    * [通用寄存器](./registers.md#通用寄存器general-purpose-registers)
    * [状态寄存器/标志寄存器](./registers.md#状态寄存器--标志寄存器status--flags-register)
    * [浮点 / SIMD 寄存器](./registers.md#浮点--simd--向量寄存器)
* 调度信息 &rarr; 优先级，时间片，调度策略
* Thread Local Storage
* 线程栈信息 &rarr; 线程通常有自己的栈，所以 TCB 也要记录和线程栈有关的信息
