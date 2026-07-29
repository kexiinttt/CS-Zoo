# IPC（Inter-Process Communication）

IPC，全称 **Inter-Process Communication**，即**进程间通信**。

在操作系统中，不同进程拥有各自独立的虚拟地址空间。一个进程通常不能直接访问另一个进程的内存，因此如果多个进程之间需要交换数据、同步状态或协同完成任务，就需要 IPC 机制。

> IPC 是操作系统提供的一组机制，用来让不同进程之间安全、可控地通信。

---

# 为什么需要 IPC？

进程之间默认是隔离的，这种隔离带来了安全性和稳定性，但也带来了通信问题。

- 一个 Web Server 进程把请求转发给 Worker 进程处理
- 多个进程共享某些状态或缓存
- 一个主进程控制多个子进程
- 本地服务之间交换数据
- 不同语言实现的服务在同一台机器上协作
- 浏览器、数据库、后台服务等复杂系统内部模块通信

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

# IPC 和线程通信的区别

线程属于同一个进程，通常共享同一块地址空间；而进程之间地址空间隔离。

| 对比项 | 线程间通信 | 进程间通信 IPC |
|---|---|---|
| 地址空间 | 通常共享 | 默认隔离 |
| 数据共享 | 可以直接访问共享变量 | 不能直接访问对方内存 |
| 安全性 | 较低，容易数据竞争 | 较高，隔离性更强 |
| 开销 | 通常较小 | 通常较大 |
| 常见机制 | mutex、condition variable、atomic | pipe、socket、shared memory、message queue |

---

# 常见 IPC 方式

## Pipe（管道）

Pipe 是一种非常经典的 IPC 方式，通常用于**父子进程之间的单向通信**。

- 数据像字节流一样传输
- 通常是单向的
- 适合简单的数据流传递
- 常见于 Unix/Linux 命令行管道

```bash
cat file.txt | grep "error"
```

这里 `cat` 和 `grep` 是两个进程，中间通过 pipe 传输数据。

适合场景：

- 简单父子进程通信
- 命令行工具串联
- 流式文本数据处理

不足：

- 通常只适合同一台机器
- 数据结构需要自己定义编码方式
- 不适合复杂双向通信

## Named Pipe / FIFO（命名管道）

普通 pipe 通常用于有亲缘关系的进程，比如父子进程。Named Pipe 则有一个文件系统路径，不相关的进程也可以通过它通信。

特点：

- 有名字，存在于文件系统中
- 可以被无亲缘关系的进程打开
- 仍然主要是字节流通信

适合场景：

- 本机不同进程之间的简单通信
- 轻量级数据传输

## Message Queue（消息队列）

Message Queue 允许进程以**消息**为单位进行通信，而不是单纯的字节流。

特点：

- 消息有边界
- 可以支持异步通信
- 发送方和接收方可以解耦
- 可支持优先级、持久化等能力，取决于实现

适合场景：

- 生产者-消费者模型
- 异步任务处理
- 进程间事件通知

例子：

```text
Producer Process ---> Message Queue ---> Consumer Process
```

不足：

- 相比 shared memory，吞吐可能较低
- 需要序列化和反序列化
- 队列可能成为瓶颈

## Shared Memory（共享内存）

Shared Memory 是一种高性能 IPC 方式。多个进程可以把同一块物理内存映射到自己的虚拟地址空间中，从而直接读写同一份数据。

特点：

- 性能非常高
- 避免大量数据拷贝
- 适合大数据量通信
- 需要额外同步机制避免 race condition

示意图：

```text
Process A Virtual Memory ---> Shared Memory Region <--- Process B Virtual Memory
```

适合场景：

- 高频交易系统
- 多媒体处理
- 数据库 buffer/cache
- 大量数据在本机进程间传输

不足：

- 编程复杂度较高
- 需要 mutex、semaphore、atomic 等同步手段
- 一个进程写坏数据可能影响其他进程

## Semaphore（信号量）

Semaphore 本身通常不是用来传输数据的，而是用来做**同步和互斥**。

常见用途：

- 控制多个进程对共享资源的访问
- 配合 shared memory 使用
- 实现 producer-consumer 同步

例子：

```text
Shared Memory stores data
Semaphore controls when data is ready or consumed
```

适合场景：

- 多进程资源访问控制
- 共享内存读写同步

## Socket（套接字）

Socket 是非常常见且通用的 IPC 方式。它不仅可以用于同一台机器上的进程通信，也可以用于不同机器之间的网络通信。

常见类型：

- Unix Domain Socket：同一台机器上的进程通信
- TCP Socket：跨机器可靠通信
- UDP Socket：跨机器低延迟、不保证可靠通信

特点：

- 通用性强
- 支持双向通信
- 可以跨机器
- 很多 RPC 框架底层都基于 socket

适合场景：

- Client-server 架构
- 本机服务之间通信
- 分布式系统服务调用
- RPC / HTTP / gRPC 通信

Unix Domain Socket 示例：

```text
Process A <---- Unix Domain Socket ----> Process B
```

TCP Socket 示例：

```text
Machine A Process <---- TCP ----> Machine B Process
```

## Memory-mapped File（内存映射文件）

Memory-mapped file 可以把文件映射到进程地址空间中。多个进程映射同一个文件后，可以通过读写内存的方式共享数据。

特点：

- 可用于文件 I/O 优化
- 可用于进程间共享数据
- 数据可以落盘，具备一定持久性

适合场景：

- 大文件处理
- 数据库存储引擎
- 本机进程间共享大块数据

## Signal（信号）

Signal 是一种轻量级的进程通知机制。

常见用途：

- 通知进程退出
- 通知进程重新加载配置
- 通知异常事件

常见例子：

```bash
kill -SIGTERM <pid>
kill -SIGHUP <pid>
```

特点：

- 非常轻量
- 适合通知，不适合传输复杂数据
- 处理逻辑需要谨慎，signal handler 中能做的事情有限

---

# 常见 IPC 方式对比

| IPC 方式 | 是否适合大数据 | 是否支持双向通信 | 是否跨机器 | 性能 | 编程复杂度 | 常见用途 |
|---|---:|---:|---:|---:|---:|---|
| Pipe | 一般 | 通常单向 | 否 | 中 | 低 | 父子进程、命令行管道 |
| Named Pipe | 一般 | 可实现 | 否 | 中 | 低 | 本机简单通信 |
| Message Queue | 一般 | 可实现 | 通常否，取决于实现 | 中 | 中 | 异步任务、事件通知 |
| Shared Memory | 非常适合 | 是 | 否 | 很高 | 高 | 高性能本机通信 |
| Semaphore | 不传输数据 | 不适用 | 否 | 高 | 中 | 同步、互斥 |
| Socket | 适合 | 是 | 是 | 中到高 | 中 | RPC、网络通信、本机服务通信 |
| Memory-mapped File | 适合 | 是 | 否 | 高 | 中到高 | 大文件、数据库、共享数据 |
| Signal | 不适合 | 否 | 否 | 高 | 低到中 | 进程通知、控制信号 |

---

# IPC 的核心设计问题

## 数据怎么编码？

进程之间传递的数据需要有格式，例如：

- raw bytes
- JSON
- Protobuf
- FlatBuffers
- 自定义 binary protocol

如果追求性能，通常会选择二进制协议或共享内存结构。

## 是否需要同步？

多个进程同时访问共享资源时，需要考虑：

- race condition
- deadlock
- data consistency
- lock granularity

Shared memory 通常必须配合同步机制使用。

## 是否需要可靠性？

需要考虑：

- 消息是否可能丢失
- 接收方挂了怎么办
- 是否需要 ACK
- 是否需要重试
- 是否需要持久化

## 是否需要背压？

如果发送方速度远快于接收方，系统需要 backpressure：

- 限制队列长度
- 阻塞发送方
- 丢弃低优先级消息
- 降级处理
- 返回错误或重试信号

## 是否需要安全隔离？

进程之间通信时，也需要考虑权限问题：

- 谁可以连接？
- 谁可以读写 shared memory？
- socket 是否需要认证？
- 消息是否需要校验？

---

# 常见系统设计例子

## Master-Worker 模型

```text
   Master Process
   |      |      |
   v      v      v
Worker Worker Worker
```

Master 负责接收任务，然后通过 IPC 分发给多个 Worker。

可以使用：

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

这种设计常用于高性能 producer-consumer 场景。

## Local RPC Service

```text
Client Process <---- Unix Domain Socket ----> Local Service Process
```

例如本机 agent、daemon、sidecar 等组件经常通过 Unix Domain Socket 对外提供本地 API。
