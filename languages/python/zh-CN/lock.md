# GIL (Global Interpreter Lock)

GIL 是 CPython 解释器里的一个全局锁。它保证了：**同一个进程内，任意时刻只能有一个线程真正执行 Python 字节码**。

## 为什么会有 GIL

CPython 的内存管理依赖于**引用计数**，需要在多线程下保持一致性。有了 GIL，解释器内部很多对象操作就不需要到处加非常细粒度的锁。

> [!IMPORTANT]
> 有 GIL 不代表 Python 程序就不需要锁了。
>
> GIL 只能保证同一时刻一个线程在执行 Python 字节码，**不能保证你的业务逻辑是线程安全的**。比如 `counter += 1` 这种“读 -> 改 -> 写”复合操作，依然可能发生竞态条件，所以该加 `Lock` 还是要加。

## GIL 带来的影响

### ❌ CPU-bound

如果任务主要是在做计算，比如：
- 大量循环
- 数值计算
- 图像处理中的纯 Python 逻辑
- 字符串/列表的大规模处理

那么即使开多个线程，同一时刻也只有一个线程在执行 Python 代码，速度通常不会随着线程数线性提升。

### ✅ I/O-bound

如果任务主要在等待：
- 网络请求
- 磁盘读写
- 数据库查询
- socket 通信

线程在等待 I/O 时通常会释放 GIL，这时别的线程就能继续执行。所以 Python 多线程在 **I/O 密集型场景**下仍然很常见，也很有价值。

---

# Lock / Mutex



在 Python 里，最常用的 mutex 就是 `threading.Lock()`。

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

* Lock 是普通互斥锁，同一个线程重复 acquire 会卡死
* RLock 是可重入锁，同一个线程可以重复获取，内部会记录次数

如果代码里有递归调用，或者一个已经持锁的方法又调用另一个也要同一把锁的方法，RLock 会更安全。


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

# Semaphore / BoundedSemaphore

Python 里用 `threading.Semaphore(n)` 来表示“最多允许 n 个线程同时进入”。

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

常见场景：
- 限制并发请求数
- 控制数据库连接池并发
- 控制爬虫同时工作的 worker 数量

`threading.BoundedSemaphore(n)` 和 `Semaphore(n)` 很像，但它会额外检查是不是 `release()` 太多次。如果想防止“多释放一次”这种 bug，`BoundedSemaphore` 更稳。

---

# Condition

`Condition` 适合“线程先等待某个条件成立，再继续执行”的场景。它内部会关联一把锁。

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

- `wait()` 调用时必须先持有关联的锁
- `wait()` 会先释放锁，醒来后再重新拿锁
- 更推荐 `wait_for(...)`，比手写 `while not condition: wait()` 更清晰

> [!NOTE]
> 如果不传入参数，Condition本身会自动创建一把RLock。但是在多个条件变量共享同一个锁的时候，可以显式传入一个锁。

---

# Event

`Event` 适合做“线程间通知”或“停止信号”。

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

常见接口：
- `event.set()`
- `event.clear()`
- `event.wait()`
- `event.is_set()`

如果你只是想发一个“可以开始了”或者“该停了”的信号，`Event` 往往比 `Condition` 更简单。

---

# Barrier

`threading.Barrier(parties)` 适合“多个线程都到达某个点以后，再一起继续”的场景。

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

当 3 个线程都执行到 `barrier.wait()` 时，才会一起往后走。

---

# queue.Queue

Python 线程安全的 FIFO 队列，适合标准的生产者-消费者模型。

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
- `task_done()` &rarr; 表示这个任务处理完了
- `join()` &rarr; 会阻塞到队列里所有任务都处理完成

## 如何做到线程安全

核心依然是依赖**互斥锁**和**条件变量**。

其底层使用容器保存数据（比如deque），然后用mutex保护共享状态，若干个condition来做等待/唤醒（队列空/满）。
* put
    * 先拿锁
    * 如果队列满了，就在 not_full 上等待
    * 有空间了，把元素放进去
    * 通知 not_empty：现在可以取了
    * 释放锁
* get
    * 先拿锁
    * 如果队列空了，就在 not_empty 上等待
    * 有元素了，取出一个
    * 通知 not_full：现在可以继续放了
    * 释放锁

## 简单实现

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

Python 标准库里**没有**明确的原子操作。如果想实现相似的功能只能手动加锁。
