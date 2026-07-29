# Thread vs asyncio

- `threading` &rarr; 开多个线程同时处理事情
- `asyncio` &rarr; 一个线程里管理很多可暂停/恢复的任务，类似高效等待

对于不同情况的任务：

- 阻塞 I/O 库很多，代码已经是同步风格：优先考虑 `multithreading`
- 高并发网络 I/O，希望大量任务复用一个事件循环：优先考虑 `asyncio`
- CPU-bound 计算：通常两者都不是最优解，更应该考虑 `multiprocessing`

---

# Thread

## threading.Thread
```py
import threading

counter = 0
lock = threading.Lock()

def worker():
    global counter
    for _ in range(10000):
        with lock:
            counter += 1

threads = [threading.Thread(target=worker, deamon=True) for _ in range(4)]

for t in threads:
    t.start()

for t in threads:
    t.join()

print(counter)
```

> `join()` 意味着让主线程等待join了的线程完成才能继续执行。比如此处要求main线程存活直到4个子线程都完成。否则可能出现主线程start完了就结束了，子线程还没跑完的情况。

> `daemon=True` 表示主线程退出后，这个子线程也会跟着结束。

## ThreadPool

线程池有几个好处：
- 复用线程，避免频繁创建和销毁
- 可以限制最大并发数
- submit() / map() / as_completed() 这些接口比较清晰
- 更适合批量 I/O 任务的并发执行

```py
from concurrent.futures import ThreadPoolExecutor

def worker(x):
    return x * x

with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(worker, [1, 2, 3, 4, 5]))

print(results)
```

### `submit()` 

是提交一个任务，返回一个 Future。
> 你可以理解成去麦麦点单了，给你的一个订单号。你可以出去溜达买奶茶，也可以原地玩手机等待，然后随时回来看订单是否完成。

```py
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def work(x):
    time.sleep(3 - x)
    return x * x

with ThreadPoolExecutor(max_workers=3) as ex:
    futures = [ex.submit(work, i) for i in [1, 2, 3]]

    for future in as_completed(futures):
        print(future.result())
```

按照完成的顺序进行输出。

常见方法：
- `future.result()`：拿结果；如果任务里抛异常，这里也会抛出来
- `future.done()`：是否完成
- `future.exception()`：查看异常
- `future.cancel()`：尝试取消还没开始的任务

### `map()`

是批量提交一组任务，并且**按输入顺序返回结果迭代器**。

```py
from concurrent.futures import ThreadPoolExecutor

def work(x):
    return x + 10

with ThreadPoolExecutor(max_workers=3) as ex:
    for result in ex.map(work, [1, 2, 3]):
        print(result)
```

### `submit()` vs `map()`

- 想按完成顺序处理结果：更适合 `submit()` + `as_completed()`
- 想批量提交并按输入顺序取结果：更适合 `map()`
- 想对每个任务拿到单独的 `Future`：用 `submit()`

---

# asyncio

**一个线程**里，用事件循环调度很多任务，在任务等待 I/O 时切出去做别的任务。

```py
import asyncio

async def worker(x):
    print(f"start {x}")
    await asyncio.sleep(1)
    print(f"done {x}")
    return x * x

async def main():
    results = await asyncio.gather(
        worker(1),
        worker(2),
        worker(3),
    )
    print(results)

asyncio.run(main())
```

`asyncio.run(main())` 可以理解成：
- 创建一个事件循环
- 把 `main()` 这个协程跑起来
- 跑完后关闭事件循环

## `asyncio.gather(task1, task2, task3)`
不是说真的开了 3 个线程。而是 3 个协程在一个事件循环里交替执行：
- worker1 跑到 await &rarr; 暂停
- worker2 跑到 await &rarr; 暂停
- worker3 跑到 await &rarr; 暂停
- 谁 I/O 好了，谁继续

他会（1）并发跑多个协程；（2）等全部结束；（3）按传入顺序返回结果。

如果你手里已经是一个列表，也可以这样写：

```py
coros = [worker(1), worker(2), worker(3)]
results = await asyncio.gather(*coros)
```

## `create_task()`

它会把协程包装成 Task，交给事件循环调度。如果你想先把任务发出去，后面再等结果，create_task 很常用。

```py
async def download():
    await asyncio.sleep(3)
    return "下载完成"

async def main():
    task = asyncio.create_task(download())

    print("先处理别的逻辑")
    await asyncio.sleep(1)
    print("别的逻辑处理完了")

    result = await task
    print(result)
```

> ⚠️ 传递的是对象`func()`而不是`func`。

## `await`

`await` 的意思不是“开新线程”，而是：

> 这个协程现在先暂停，把执行权还给事件循环，等结果好了再回来继续。

比如：

```py
async def main():
    result = await download()
    print(result)
```

这里 `main()` 在等待 `download()` 时，不是整个程序都卡死了。  
事件循环可以趁机去调度别的协程。

## `asyncio.sleep()`

```py
await asyncio.sleep(1)
```

它不是 `time.sleep(1)`。

- `time.sleep(1)`：会阻塞当前线程
- `await asyncio.sleep(1)`：会让出事件循环，让别的协程先跑

所以在协程里，通常不要写阻塞式的 `time.sleep()`。

## `gather()` vs `create_task()`

可以这样粗略理解：

- `gather()`：我现在有一批任务，想一起跑，并统一等待结果
- `create_task()`：我先把任务丢出去后台跑，稍后再决定什么时候等它

例如：

```py
async def main():
    task1 = asyncio.create_task(worker(1))
    task2 = asyncio.create_task(worker(2))

    print("先做别的事")

    result1 = await task1
    result2 = await task2
    print(result1, result2)
```

## `asyncio.as_completed()`

如果你想“谁先完成就先处理谁”，可以用：

```py
import asyncio

async def work(x):
    await asyncio.sleep(3 - x)
    return x * x

async def main():
    tasks = [asyncio.create_task(work(i)) for i in [1, 2, 3]]

    for task in asyncio.as_completed(tasks):
        result = await task
        print(result)
```

这个思路和线程池里的 `as_completed()` 很像。

## 协程不是线程

协程本身不是并行执行，而是在合适的 await 点主动让出执行权。

所以如果某个协程里没有 await，它就可能长时间霸占事件循环。

## 协程里的锁

如果多个协程要访问共享状态，应该用 `asyncio.Lock()`，而不是 `threading.Lock()`。

```py
import asyncio

lock = asyncio.Lock()
counter = 0

async def worker():
    global counter
    async with lock:
        counter += 1
```

这里保护的是“多个协程之间”的并发访问，不是多个线程。

---

# 线程和 asyncio 互相配合

现实项目里，它们经常不是二选一，而是混着用。

## `asyncio.to_thread()`

如果你在协程里，临时想调用一个阻塞函数，可以把它扔到线程里跑：

```py
import asyncio
import time

def blocking_work():
    time.sleep(2)
    return "done"

async def main():
    result = await asyncio.to_thread(blocking_work)
    print(result)
```

这在“项目主架构是 `asyncio`，但中间不得不调用老的同步函数”时非常有用。

## `loop.run_in_executor()`

更底层一点的做法是把阻塞任务丢到 executor 里：

```py
import asyncio
from concurrent.futures import ThreadPoolExecutor
import time

def blocking_work(x):
    time.sleep(1)
    return x * x

async def main():
    loop = asyncio.get_running_loop()

    with ThreadPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, blocking_work, 3)
        print(result)
```

可以理解成：在 `asyncio` 世界里借一个线程池来处理同步阻塞任务。

---

# ❌ 在协程里写阻塞代码

```py
import time

async def bad():
    time.sleep(3)
```

这会把整个事件循环卡住。  

协程里应该尽量使用可 `await` 的 I/O API，或者用 `asyncio.to_thread()` / `run_in_executor()` 包一下。
