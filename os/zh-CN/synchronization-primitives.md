# 同步原语

在并发程序里，多个线程或进程会同时访问共享资源。如果没有同步机制，就会出现：

- race condition（竞态条件）
- data corruption（数据损坏）
- lost update（更新丢失）
- visibility problem（可见性问题）
- deadlock（死锁）、starvation（饥饿）、livelock（活锁）

同步原语的目标主要有两个：

- 互斥（Mutual Exclusion）：同一时刻只允许一个执行单元进入临界区  
- 协调（Coordination / Ordering）：让线程按照某种条件或顺序运行

---

# 临界区（Critical Section）

就是访问共享资源的一段代码。

```cpp
// 1. 读取
// 2. ++
// 3. 写回
counter++;
```

如果两个线程同时做这件事，就可能都读到同一个旧值，最后只加了一次。

所以：

- **共享资源**：被多个线程同时访问的数据
- **临界区**：访问共享资源的代码片段
- **同步原语**：保护临界区、协调线程顺序的工具

---

# 同步原语的分类

## 互斥类

用于“同一时刻只允许一个线程进入”。

- mutex
- spinlock
- binary semaphore
- recursive mutex

## 条件等待类

用于“等某个条件成立再继续”。

- condition variable
- semaphore
- event / latch / barrier

## 读写分离类

用于“多个读者可并发，写者独占”。

- read-write lock（rwlock / shared_mutex）

## 原子类

用于“单个共享变量的无锁更新”。

- atomic
- CAS（compare-and-swap）
- fetch_add / exchange

---

# 互斥锁

## Mutex

互斥锁的语义非常直接：线程进入临界区前先 `lock()`，离开临界区后再 `unlock()`。

如果锁已经被别人持有，当前线程就阻塞等待（睡眠/挂起，让操作系统做context switch）

### 优点

- 语义清晰，容易保证正确性
- 对长临界区友好

### 缺点

- 有上下文切换开销
- 激烈竞争下性能可能下降
- 锁粒度太大时并发度差

## Spinlock

自旋锁也是互斥锁的一种，但拿不到锁时不是挂起/睡眠，而是原地等待。

### 优点
- 对短临界区友好
- 无上下文切换等开销

### 缺点
* CPU 浪费 &rarr; 如果锁持有时间长，等待线程会持续消耗 CPU
* 优先级反转 / 抢占问题 &rarr; 若持锁线程被调度走，而等待线程一直自旋，会非常低效
* 单核 &rarr; 单核时等待线程自旋会占住 CPU，持锁线程反而没机会运行并释放锁

> [!IMPORTANT]
> **Priority Inversion（优先级反转）**
> 
> 低优先级线程持有锁，高优先级线程等待它，而中优先级线程却不断抢占 CPU，导致高优先级线程迟迟得不到锁。
>
> 常见缓解手段
> - priority inheritance（优先级继承）
> - 减少持锁时间
> - 避免高优先级路径依赖低优先级线程持锁

## Mutex vs Spinlock

Spinlock的设计理念 &rarr; 因为线程睡眠/挂起涉及到上下文切换/内核态切换，如果锁很快就会释放，那么“忙等CPU cycles”可能比“睡眠再唤醒”更便宜。

---

# Semaphore

信号量本质上是一个**带计数器的同步原语**。它表示“可用资源的数量”或者“允许继续执行的配额”。


- `P`：计数减一；如果结果小于 0 或资源不足，则阻塞
- `V`：计数加一；必要时唤醒等待者

## Semaphore vs Mutex

* mutex
    - 主要用于**互斥**
    - 一般只有“锁住 / 解锁”两种状态
    - 通常“谁加锁谁解锁”
* semaphore
    - 主要用于**计数 / 资源配额 / 线程协调**
    - 计数可以大于 1
    - 不强调必须由同一线程释放

## Binary Semaphore

计数只在 0/1 附近使用时。这和 mutex 看起来很像，但语义还是不同：
- mutex 更强调“同一个线程加锁和解锁”
- semaphore 允许“一个线程V，另一个线程P”

## Counting Semaphore

限制最多同时有 N 个线程进入某区域。经典例子：生产者-消费者。

---

# Condition Variable

假设有一个消息队列，我们希望在里面有内容时进行消费。mutex 只能解决“只有一个线程能在某一时刻放入/取出消息”，不能优雅地解决“等到队列非空再取消息”。

```cpp
// 循环等待可以解决，但是浪费CPU
while (queue.empty()) {}
```

```cpp
#include <condition_variable>
#include <mutex>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> q;

void producer() {
    {
        std::lock_guard<std::mutex> lock(mtx);
        q.push(42);
    }
    cv.notify_one();
}

void consumer() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return !q.empty(); });

    int x = q.front();
    q.pop();
}
```

> [!IMPORTANT]
> 一定要配合 mutex，因为要保护“队列是否为空”这个条件，而这个条件是共享的，必须用mutex锁住后再check。

> [!NOTE]
> **虚假唤醒（Spurious Wakeup）**&rarr; 线程可能在没有真正满足条件时被唤醒。
> 
> 比如多个消费者同时收到队列有数据，但是某一个消费者已经消费了，此时队列实际上又是空的。因此为了真正的保证条件满足，需要再次check：
> ```cpp
> while (!condition) { // ⚠️ 不是 if
>     cv.wait(lock);
> }
> ```
> ```cpp
> cv.wait(lock, [] { return condition; });
> ```

---

# Read-Write Lock

有些共享资源读多写少，如果用普通 mutex，那么多个读线程也不能并发，会浪费并行性。
- 读锁（shared lock）&rarr; 只要没有写者持有，多个线程都可以拿
- 写锁（exclusive lock）&rarr; 写者拿到后，其他读者和写者都必须等待

## 饥饿问题

读写锁设计时容易面临饥饿问题：
- 不断有读者进来，写者可能长期拿不到锁
- 一旦有多个写者等待，新的读者也不能进

---

# Pessimistic vs Optimistic Lock

| 对比项 | 悲观锁 | 乐观锁 |
| :---: | :---: | :---: |
| 假设 | 冲突经常发生 | 冲突很少发生 |
| 策略 | 先加锁再操作 | 先操作，提交时校验 |
| 常见实现 | mutex、行锁 | version、timestamp、CAS |
| 优点 | 一致性强，逻辑直接 | 并发高，少阻塞 |
| 缺点 | 阻塞、死锁、吞吐低 | 冲突多时重试成本高 |
| 适合场景 | 写多冲突多 | 读多写少 |

> [!NOTE]
> 乐观锁的实现很多时候基于用户逻辑，比如：
> * 每次更改数据就把version number增加
> * 每次更改数据就把最新的timestamp更新
> 
> 然后提交的时候通过比较标志位是否一致来判断是否冲突

---

# Futex（Fast Userspace Mutex）

因为真正高性能的锁，不希望每次 `lock/unlock` 都进入内核：

- 无竞争 fast path：原子操作即可完成
- 竞争时 slow path：调用 futex 等待 / 唤醒

---

# Atomic

原子操作保证一个操作不会被并发地“撕裂”。

atomic 不能替代 lock，如果你需要维护的是**多个变量之间的一致性**，仅靠单个 atomic 往往不够。

## CAS（Compare-And-Swap）

只有当内存中的值等于预期值时，才把它更新成新值。

```cpp
if (*addr == expected) {
    *addr = desired;
    return true;
}
return false;
```

更详细的请参考[Atomic](./atomic.md)。

---

# Barrier / Latch

## Barrier

多个线程都到达某个点之后，才一起继续，强调的是**多个线程**都要继续。

适用于并行计算的阶段同步。

## Latch

一个或多个线程在等待某件事完成，满足条件后**只有等待方**会继续。

适合“父线程等待 N 个子线程全部完成”。

---

# Deadlock, Starvation, Livelock

* Deadlock &rarr; 多个线程彼此等待对方持有的资源，永远无法继续
* Starvation &rarr; 某个线程长期拿不到资源，但系统整体还在运行
* Livelock &rarr; 线程都在积极响应彼此，但没有真正推进工作，例如双方都很“礼貌”地不断让步

## 死锁四个必要条件

- 互斥 &rarr; 资源一次只能被一个线程占有
- 占有且等待 &rarr; 线程已经拿到一部分资源，同时还在等待其他资源
- 不可抢占 &rarr; 已经拿到的资源不能被系统强行夺走
- 循环等待 &rarr; 存在一个等待环，都在等彼此放出资源

## 预防、避免、检测和解除
- 预防（prevention）：事先破坏四个必要条件之一
    - 破坏互斥
        - 尽量把独占资源改成共享资源
    - 破坏占有等待
        - 一次性申请所有资源，拿不到就一个都不拿
        - 申请新资源前，先释放已持有资源
    - 破坏不可抢占
        - 当申请不到新资源时，主动释放已经占有的资源，稍后重试
    - 破坏循环等待
        - 给资源定义一个固定加锁的顺序，比如先A再B
- 避免（avoidance）：动态判断这次分配会不会进入危险状态，安全才分配（银行家算法）
- 检测（detection）：先允许发生，再检查有没有死锁
- 解除/恢复（recovery）：发生后回滚、撤销、杀进程、释放资源

---

## 16. 锁粒度（Lock Granularity）

| 类型 | 含义 | 优点 | 缺点 |
| --- | --- | --- | --- |
| 粗粒度锁 | 整个大对象只用一把锁 | 简单<br/>正确性容易保证 | 并发度低<br/>容易形成热点 |
| 细粒度锁 | 不同子结构用不同锁 | 并发度高<br/>减少不必要互斥 | 实现复杂<br/>死锁风险更高 |

实际工程中常常要平衡：

- 正确性
- 实现复杂度
- 吞吐量
- 延迟

---

# Fair vs Non-fair lock

| 类型 | 含义 | 优点 | 缺点 |
| --- | --- | --- | --- |
| 公平锁 | 等待时间更久的线程更早获得锁 | 更可预测<br/>不容易饥饿 | 吞吐量可能更低<br/>上下文切换可能更多 |
| 非公平锁 | 新来的线程也可能直接抢到锁 | 吞吐可能更高<br/>cache locality 有时更好<br/> | 可能导致某些线程等待很久 |

---

# 对比表

| 原语 | 主要用途 | 等待方式 | 是否有所有权概念 | 典型场景 |
|---|---|---|---|---|
| mutex | 互斥 | 阻塞 | 有 | 保护临界区 |
| spinlock | 互斥 | 忙等 | 通常有 | 极短临界区 |
| semaphore | 计数 / 协调 | 阻塞 | 弱或无 | 配额、限流、生产消费 |
| condition variable | 等条件成立 | 阻塞 | 不强调 | 队列非空、状态变化 |
| rwlock | 读多写少 | 阻塞 | 有共享/独占模式 | 配置、缓存、索引 |
| atomic | 单变量原子更新 | 不阻塞 | 无 | 计数器、状态位 |
