# GIL (Global Interpreter Lock)

The GIL is a global lock in the CPython interpreter. It guarantees that: **In the same process, only one thread can actually execute Python bytecode at any time**.

## Why is there GIL?

CPython's memory management relies on **reference counting** and needs to maintain consistency under multi-threading. With GIL, many object operations inside the interpreter do not require very fine-grained locks everywhere.

> [!IMPORTANT]
> The presence of GIL does not mean that Python programs do not need locks.
>
> GIL can only guarantee that one thread is executing Python bytecode at the same time, and it cannot guarantee that your business logic is thread-safe. For example, a "read -> change -> write" compound operation like `counter += 1` may still cause a race condition, so `Lock` should be added.

## Impact of GIL

### ❌ CPU-bound

If the task is mainly doing calculations, such as:
- Lots of loops
- Numerical calculations
- Pure Python logic in image processing
- Large scale processing of strings/lists

Then even if multiple threads are opened, only one thread is executing Python code at the same time, and the speed usually does not increase linearly with the number of threads.

### ✅ I/O-bound

If the task is mainly waiting:
- Network requests
- Disk reading and writing
- Database query
- socket communication

Threads usually release the GIL while waiting for I/O, so that other threads can continue executing. So Python multithreading is still very common and valuable in **I/O intensive scenarios**.

---

# Lock/Mutex



In Python, the most commonly used mutex is `threading.Lock()`.

```py
import threading

lock = threading.Lock()
counter = 0

# not good
def worker():
    global counter
    for _ in range(10000):
        lock.acquire()
        counter += 1
        lock.release()

# good
def good_worker():
    global counter
    for _ in range(10000):
        with lock: # 使用context manager能更好管理资源
            counter += 1
```

## Lock vs RLock

* Lock is an ordinary mutex lock. The same thread will get stuck if it acquires repeatedly.
* RLock is a reentrant lock. The same thread can obtain it repeatedly, and the number of times will be recorded internally.

If there are recursive calls in the code, or a method that already holds a lock calls another method that also requires the same lock, RLock will be safer.


```py
import threading

# Lock 示例 - 死锁
lock = threading.Lock()
lock.acquire()
# lock.acquire()  # 如果解开此行注释，会死锁

# RLock 示例 - 正常
rlock = threading.RLock()
rlock.acquire()
rlock.acquire() # 正常
rlock.release()
rlock.release() # 正常

```

---

# Semaphore/BoundedSemaphore

`threading.Semaphore(n)` is used in Python to mean "up to n threads are allowed to enter at the same time".

```py
import threading
import time

sem = threading.Semaphore(3)

def worker(i):
    with sem:
        print(f"task {i} start")
        time.sleep(2)
        print(f"task {i} end")
```

Common scenarios:
- Limit the number of concurrent requests
- Control database connection pool concurrency
- Control the number of workers working on the crawler at the same time

`threading.BoundedSemaphore(n)` is similar to `Semaphore(n)`, but it additionally checks to see if it is `release()` too many times. If you want to prevent the "one more release" bug, `BoundedSemaphore` is more stable.

---

# Condition

`Condition` is suitable for scenarios where "the thread waits for a certain condition to be established before continuing execution". It will have a lock associated with it internally.

- `cond.wait()`
- `cond.notify()`
- `cond.notify_all()`
- `cond.wait_for(predicate)`

```py
import threading

items = []
cond = threading.Condition()

def producer():
    with cond:
        items.append("data")
        cond.notify()

def consumer():
    with cond:
        cond.wait_for(lambda: len(items) > 0)
        item = items.pop()
        print(item)
```

- The associated lock must first be held when calling `wait()`
- `wait()` will release the lock first, and then retake the lock after waking up.
- `wait_for(...)` is more recommended, it is clearer than handwriting `while not condition: wait()`

> [!NOTE]
> If no parameters are passed in, Condition itself will automatically create an RLock. But when multiple condition variables share the same lock, a lock can be passed in explicitly.

---

# Event

`Event` is suitable for "inter-thread notification" or "stop signal".

```py
import threading
import time

stop_event = threading.Event()

def worker():
    while not stop_event.is_set():
        print("working...")
        time.sleep(1)

thread = threading.Thread(target=worker)
thread.start()

time.sleep(3)
stop_event.set()
thread.join()
```

Common interfaces:
- `event.set()`
- `event.clear()`
- `event.wait()`
- `event.is_set()`

If you just want to signal "it's time to start" or "it's time to stop", `Event` is often simpler than `Condition`.

---

# Barrier

`threading.Barrier(parties)` is suitable for the scenario where "multiple threads have reached a certain point and then continue together".

```py
import threading
import time

barrier = threading.Barrier(3)

def worker(i):
    print(f"{i} ready")
    time.sleep(i)
    barrier.wait()
    print(f"{i} go")
```

When all three threads execute `barrier.wait()`, they will go back together.

---

# queue.Queue

Python thread-safe FIFO queue, suitable for the standard producer-consumer model.

```py
import queue
import threading
import time

q = queue.Queue()

def producer():
    for i in range(5):
        print(f"produce {i}")
        q.put(i)
        time.sleep(0.1)

def consumer():
    while True:
        item = q.get()
        if item is None:
            break
        print(f"consume {item}")
        q.task_done()

t1 = threading.Thread(target=producer)
t2 = threading.Thread(target=consumer)

t1.start()
t2.start()

t1.join()
q.put(None)   # sentinel
q.join()
t2.join()
```

- `put(item)`
- `get()`
- `task_done()` → indicates that the task has been processed
- `join()` → will be blocked until all tasks in the queue are processed.

## How to achieve thread safety

The core still relies on **mutex lock** and **condition variable**.

The bottom layer uses a container to save data (such as deque), then uses mutex to protect the shared state, and several conditions to wait/wake up (queue empty/full).
* put
    * Get the lock first
    * If the queue is full, wait on not_full
    * If there is space, put the element in it
    * Notification not_empty: now available for pickup
    * release lock
* get
    * Get the lock first
    * If the queue is empty, wait on not_empty
    * There is an element, take out one
    * Notification not_full: You can continue playing now
    * release lock

## Simple implementation

```py
import threading
from collections import deque
from typing import Generic, TypeVar, Optional

T = TypeVar("T") # 类似c++的template typename


class MPMCQueue(Generic[T]):
    def __init__(self, maxsize: int = 0) -> None:
        self._queue: deque[T] = deque()
        self._maxsize = maxsize  # 0 means unbounded

        self._lock = threading.Lock()
        self._not_empty = threading.Condition(self._lock)
        self._not_full = threading.Condition(self._lock)

    def put(self, item: T) -> None:
        with self._not_full:
            self._not_full.wait_for(
                lambda: self._maxsize <= 0 or len(self._queue) < self._maxsize
            )

            self._queue.append(item)
            self._not_empty.notify()

    def get(self) -> T:
        with self._not_empty:
            self._not_empty.wait_for(
                lambda: len(self._queue) > 0
            )

            item = self._queue.popleft()
            self._not_full.notify()
            return item

    def qsize(self) -> int:
        with self._lock:
            return len(self._queue)

    def empty(self) -> bool:
        with self._lock:
            return not self._queue
```

---

# Atomic

There are **no** explicit atomic operations in the Python standard library. If you want to achieve similar functions, you can only lock it manually.
