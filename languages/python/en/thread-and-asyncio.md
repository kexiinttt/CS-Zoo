# Thread vs asyncio

- `threading` → Open multiple threads to handle things at the same time
- `asyncio` → Manage many pause/resume tasks in one thread, similar to efficient waiting

For tasks in different situations:

- There are many blocking I/O libraries, and the code is already synchronous style: give priority to `multithreading`
- High concurrency network I/O, hope that a large number of tasks reuse an event loop: give priority to `asyncio`
- CPU-bound calculation: Usually neither is the optimal solution, and `multiprocessing` should be considered

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

> `join()` means to let the main thread wait for the joined thread to complete before continuing execution. For example, the main thread is required to survive until all four child threads are completed. Otherwise, it may happen that the main thread ends after it starts, but the sub-thread has not finished running.

> `daemon=True` means that after the main thread exits, this sub-thread will also end.

## ThreadPool

Thread pools have several benefits:
- Reuse threads to avoid frequent creation and destruction
- Can limit the maximum number of concurrencies
- submit() / map() / as_completed() These interfaces are relatively clear
- More suitable for concurrent execution of batch I/O tasks

```py
from concurrent.futures import ThreadPoolExecutor

def worker(x):
    return x * x

with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(worker, [1, 2, 3, 4, 5]))

print(results)
```

### `submit()`

Submit a task and return a Future.
> You can understand it as going to Maimai to place an order, and we will give you an order number. You can go out for a walk to buy milk tea, or you can wait on your phone while waiting, and then come back at any time to see if the order has been completed.

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

Output in the order of completion.

Common methods:
- `future.result()`: Get the result; if an exception is thrown in the task, it will also be thrown here
- `future.done()`: Complete or not
- `future.exception()`: View exceptions
- `future.cancel()`: Try to cancel a task that has not started yet

### `map()`

Submits a set of tasks in batches and returns an iterator of results in input order.

```py
from concurrent.futures import ThreadPoolExecutor

def work(x):
    return x + 10

with ThreadPoolExecutor(max_workers=3) as ex:
    for result in ex.map(work, [1, 2, 3]):
        print(result)
```

### `submit()` vs `map()`

- Want to process results in order of completion: more suitable `submit()` + `as_completed()`
- If you want to submit in batches and get the results in the order entered: more suitable for `map()`
- To get a separate `Future` for each task: use `submit()`

---

# asyncio

**A thread** uses an event loop to schedule many tasks, and when the tasks are waiting for I/O, they are cut off to do other tasks.

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

`asyncio.run(main())` can be understood as:
- Create an event loop
- Run the coroutine `main()`
- Close the event loop after running

## `asyncio.gather(task1, task2, task3)`
It's not that 3 threads are actually opened. Instead, 3 coroutines are executed alternately in an event loop:
- worker1 runs to await → pause
- worker2 runs to await → pause
- worker3 runs to await → pause
- Who I/O is done, who continues

It will (1) run multiple coroutines concurrently; (2) wait for them all to end; (3) return results in the order they are passed in.

If you already have a list in hand, you can also write it like this:

```py
coros = [worker(1), worker(2), worker(3)]
results = await asyncio.gather(*coros)
```

## `create_task()`

It will wrap the coroutine into a Task and hand it over to the event loop for scheduling. If you want to send the task out first and wait for the result later, create_task is very commonly used.

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

> ⚠️ The object `func()` is passed instead of `func`.

## `await`

`await` does not mean "open a new thread", but:

> This coroutine is paused now, and the execution rights are returned to the event loop, and then come back and continue when the result is good.

for example:

```py
async def main():
    result = await download()
    print(result)
```

Here, when `main()` is waiting for `download()`, the entire program is not stuck.  
The event loop can take the opportunity to schedule other coroutines.

## `asyncio.sleep()`

```py
await asyncio.sleep(1)
```

It's not `time.sleep(1)`.

- `time.sleep(1)`: Will block the current thread
- `await asyncio.sleep(1)`: Will leave the event loop and let other coroutines run first

Therefore, in coroutines, usually do not write blocking `time.sleep()`.

## `gather()` vs `create_task()`

It can be roughly understood like this:

- `gather()`: I now have a batch of tasks that I want to run together and wait for the results together.
- `create_task()`: I will throw the task out and run it in the background first, and then decide when to wait for it later.

For example:

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

If you want to "whoever finishes first will be processed first", you can use:

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

This idea is very similar to `as_completed()` in the thread pool.

## Coroutines are not threads

The coroutine itself is not executed in parallel, but actively gives up execution rights at the appropriate await point.

So if there is no await in a coroutine, it may occupy the event loop for a long time.

## Locks in coroutines

If multiple coroutines want to access shared state, `asyncio.Lock()` should be used instead of `threading.Lock()`.

```py
import asyncio

lock = asyncio.Lock()
counter = 0

async def worker():
    global counter
    async with lock:
        counter += 1
```

What is protected here is concurrent access "between multiple coroutines", not multiple threads.

---

# Threads and asyncio cooperate with each other

In real projects, they often do not choose one of the two, but use them in combination.

## `asyncio.to_thread()`

If you are in a coroutine and want to call a blocking function temporarily, you can throw it into the thread and run:

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

This is very useful when "the main structure of the project is `asyncio`, but the old synchronization function has to be called in the middle".

## `loop.run_in_executor()`

A lower-level approach is to throw blocking tasks into the executor:

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

It can be understood as: borrowing a thread pool to handle synchronous blocking tasks in the `asyncio` world.

---

# ❌ Write blocking code in the coroutine

```py
import time

async def bad():
    time.sleep(3)
```

This will jam the entire event loop.  

In the coroutine, you should try to use the I/O API that can be `await`, or wrap it with `asyncio.to_thread()` / `run_in_executor()`.
