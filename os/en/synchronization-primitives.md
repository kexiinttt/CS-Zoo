# Synchronization Primitives

Concurrent programs can have multiple threads or processes access shared resources at the same time. Without synchronization, they can suffer from:

- Race conditions
- Data corruption
- Lost updates
- Visibility problems
- Deadlock, starvation, and livelock

Synchronization has two main goals:

- **Mutual exclusion**: only one execution unit may enter a critical section at a time.
- **Coordination / ordering**: make threads run according to a condition or order.

---

# Critical Section

A critical section is code that accesses a shared resource.

```cpp
// 1. Read
// 2. ++
// 3. Write back
counter++;
```

If two threads do this simultaneously, both may read the same old value and the counter may increase only once.

Thus:

- **Shared resource**: data accessed by multiple threads.
- **Critical section**: code that accesses the shared resource.
- **Synchronization primitive**: a tool that protects the critical section and coordinates thread order.

---

# Categories

## Mutual Exclusion

Allow only one thread to enter at a time:

- Mutex
- Spinlock
- Binary semaphore
- Recursive mutex

## Conditional Waiting

Wait until a condition becomes true:

- Condition variable
- Semaphore
- Event, latch, or barrier

## Read/Write Separation

Allow multiple readers concurrently while writers have exclusive access:

- Read-write lock (`rwlock` / `shared_mutex`)

## Atomic Operations

Perform lock-free updates to one shared variable:

- Atomic
- CAS (compare-and-swap)
- `fetch_add` / `exchange`

---

# Mutex

A mutex has direct semantics: call `lock()` before entering a critical section and `unlock()` after leaving it.

If another thread owns the lock, the current thread blocks or sleeps and lets the OS perform a context switch.

### Advantages

- Clear semantics and easy correctness reasoning.
- Good for long critical sections.

### Disadvantages

- Context-switch overhead.
- Performance can decline under heavy contention.
- A coarse lock limits concurrency.

## Spinlock

A spinlock is also a mutual-exclusion primitive. Instead of sleeping when it cannot acquire the lock, the thread waits in place.

### Advantages
- Good for short critical sections.
- No context-switch overhead.

### Disadvantages
* CPU waste &rarr; a thread spins continuously if the lock is held for a long time.
* Priority inversion and preemption &rarr; if the lock holder is descheduled while another thread spins, the wait is very inefficient.
* Single-core problem &rarr; the spinning thread can occupy the CPU and prevent the lock holder from running and releasing the lock.

> [!IMPORTANT]
> **Priority inversion**
>
> A low-priority thread holds a lock, a high-priority thread waits for it, and a medium-priority thread repeatedly preempts the low-priority thread. The high-priority thread therefore cannot acquire the lock.
>
> Common mitigations:
> - Priority inheritance
> - Shorter lock-holding times
> - Avoiding high-priority paths that depend on a low-priority lock holder

## Mutex vs. Spinlock

A spinlock is based on this idea: sleeping and waking involves context and kernel-mode switches. If the lock will be released very soon, busy-waiting for CPU cycles may be cheaper than sleeping and waking.

---

# Semaphore

A semaphore is a synchronization primitive with a **counter**. It represents the number of available resources or the quota allowed to continue.

- `P`: decrement the counter; block if the result is below zero or resources are unavailable.
- `V`: increment the counter and wake a waiter when necessary.

## Semaphore vs. Mutex

* Mutex:
  - Mainly provides **mutual exclusion**.
  - Usually has only locked and unlocked states.
  - Usually the owner unlocks it.
* Semaphore:
  - Mainly provides **counting, resource quotas, and coordination**.
  - Its count can exceed one.
  - It does not require the same thread to release it.

## Binary Semaphore

When the count is used only near 0 and 1, a semaphore resembles a mutex, but the semantics differ:
- A mutex emphasizes that the same thread locks and unlocks it.
- A semaphore allows one thread to call `V` and another to call `P`.

## Counting Semaphore

A counting semaphore can limit a region to at most N concurrent threads. The classic example is producer-consumer coordination.

---

# Condition Variable

Suppose a message queue should be consumed only when it contains data. A mutex ensures that only one thread inserts or removes a message at a time, but it does not elegantly express "wait until the queue is non-empty."

```cpp
// Busy-waiting works but wastes CPU
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
> Always use a mutex with the condition variable. The condition "is the queue empty?" is shared state and must be protected while checked.

> [!NOTE]
> **Spurious wakeup** &rarr; a thread may wake even though the condition is not truly satisfied.
>
> For example, multiple consumers may be notified that data exists, but one consumer may consume it first, leaving the queue empty. Check the condition again:
> ```cpp
> while (!condition) { // ⚠️ not if
>     cv.wait(lock);
> }
> ```
> ```cpp
> cv.wait(lock, [] { return condition; });
> ```

---

# Read-Write Lock

Some shared resources are read-heavy and write-light. A normal mutex prevents readers from running concurrently and wastes parallelism.
- A shared/read lock can be held by multiple threads as long as no writer holds the lock.
- An exclusive/write lock blocks all other readers and writers.

## Starvation

Read-write locks can face starvation:
- A continuous stream of readers may prevent a writer from acquiring the lock for a long time.
- Once multiple writers are waiting, new readers may also be prevented from entering.

---

# Pessimistic vs. Optimistic Locking

| Aspect | Pessimistic | Optimistic |
| :---: | :---: | :---: |
| Assumption | Conflicts are frequent | Conflicts are rare |
| Strategy | Lock before operating | Operate first and validate at commit |
| Common implementations | Mutex, row lock | Version, timestamp, CAS |
| Advantage | Strong consistency and direct logic | High concurrency and little blocking |
| Disadvantage | Blocking, deadlocks, low throughput | High retry cost when conflicts are frequent |
| Suitable scenario | Write-heavy and conflict-heavy | Read-heavy and write-light |

> [!NOTE]
> Optimistic locks are often implemented at the application level:
> * Increment a version number for each update.
> * Update the latest timestamp for each update.
>
> Compare the marker at commit time to determine whether a conflict occurred.

---

# Futex (Fast Userspace Mutex)

A high-performance lock should avoid entering the kernel for every `lock` and `unlock`:

- Uncontended fast path: an atomic operation completes the operation.
- Contended slow path: call `futex` to wait or wake.

---

# Atomic

An atomic operation guarantees that an operation is not torn apart by concurrent execution.

Atomic operations cannot replace a lock when consistency must be maintained across **multiple variables**; one atomic variable is often insufficient.

## CAS (Compare-And-Swap)

Update the value only when the memory value equals the expected value:

```cpp
if (*addr == expected) {
    *addr = desired;
    return true;
}
return false;
```

See [Atomic](./atomic.md) for more detail.

---

# Barrier / Latch

## Barrier

Multiple threads reach a point and continue together. The important property is that **all participating threads** continue after the barrier. This is useful for synchronization between phases of parallel computation.

## Latch

One or more threads wait for an event to complete. Once the condition is met, **only the waiting side** continues. A common use is for a parent thread to wait for N child threads to finish.

---

# Deadlock, Starvation, and Livelock

* Deadlock &rarr; threads wait for resources held by one another and can never continue.
* Starvation &rarr; one thread cannot obtain resources for a long time while the system as a whole continues.
* Livelock &rarr; threads actively respond to one another but make no progress, such as both repeatedly yielding to be polite.

## Four Necessary Conditions for Deadlock

- Mutual exclusion &rarr; a resource can be held by only one thread at a time.
- Hold and wait &rarr; a thread holds some resources while waiting for others.
- No preemption &rarr; resources already held cannot be forcibly taken.
- Circular wait &rarr; a cycle exists in which each thread waits for another to release a resource.

## Prevention, Avoidance, Detection, and Recovery

- **Prevention**: break one necessary condition in advance:
  - Break mutual exclusion by making exclusive resources shared where possible.
  - Break hold and wait by requesting all resources at once, or releasing held resources before requesting new ones.
  - Break no preemption by releasing held resources and retrying when a new resource is unavailable.
  - Break circular wait by imposing a fixed lock order, such as A before B.
- **Avoidance**: dynamically determine whether an allocation creates an unsafe state and allocate only when safe, as in the Banker's algorithm.
- **Detection**: allow deadlocks and check afterward whether one exists.
- **Recovery**: roll back, revoke resources, kill processes, or release resources after a deadlock.

---

# Lock Granularity

| Type | Meaning | Advantage | Disadvantage |
|---|---|---|---|
| Coarse-grained lock | One lock protects a large object | Simple; correctness is easy to maintain | Low concurrency; easily becomes a hotspot |
| Fine-grained locks | Different substructures use different locks | Higher concurrency; less unnecessary exclusion | More complex; higher deadlock risk |

Engineering usually balances correctness, implementation complexity, throughput, and latency.

---

# Fair vs. Non-Fair Lock

| Type | Meaning | Advantage | Disadvantage |
|---|---|---|---|
| Fair lock | The thread that has waited longest gets the lock earlier | Predictable; less starvation | Potentially lower throughput and more context switches |
| Non-fair lock | A new thread may acquire the lock immediately | Potentially higher throughput; sometimes better cache locality | Some threads may wait for a long time |

---

# Comparison

| Primitive | Main use | Waiting method | Ownership concept | Typical scenario |
|---|---|---|---|---|
| Mutex | Mutual exclusion | Blocking | Yes | Protect a critical section |
| Spinlock | Mutual exclusion | Busy-waiting | Usually | Extremely short critical section |
| Semaphore | Counting and coordination | Blocking | Weak or none | Quotas, rate limiting, producer-consumer |
| Condition variable | Wait for a condition | Blocking | Not emphasized | Non-empty queue, state change |
| Read-write lock | Read-heavy/write-light access | Blocking | Shared/exclusive modes | Configuration, cache, index |
| Atomic | Atomic update of one variable | Non-blocking | None | Counter, state flag |
