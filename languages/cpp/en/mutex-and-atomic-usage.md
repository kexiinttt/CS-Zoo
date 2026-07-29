# mutex

## `std::mutex`
```cpp
std::mutex lock;

lock.lock(); // 拿锁，拿不到就阻塞
bool ok = lock.try_lock();  //尝试拿锁，拿不到立即返回
lock.unlock();  // 解锁
```

## `std::lock_guard<std::mutex>`
- Safer way to write RAII
- Users are not allowed to unlock by themselves
- Applies to the entire scope and must be locked
```cpp
std::mutex lock;
{
    std::lock_guard<std::mutex> guard(lock); // 拿锁
    ...
} // 解锁
```

## `std::unique_lock<std::mutex>`
It also unlocks automatically, but additionally supports:
- Delayed locking
- Manually unlock and then lock
- Cooperate with condition_variable
- Ownership moves
```cpp
std::mutex m;
{
    std::unique_lock<std::mutex> lock(m); // 加锁
    ...
    lock.unlock();

    lock.lock(); // 加锁
    ...
} // 解锁
```

# condition_variable

condition_variable is used: one thread waits for a certain condition to be true, and another thread notifies it when the condition changes.

Generally speaking, it will be used in conjunction with `std::unique_lock<std::mutex>`.

## `wait()`

Release the lock and block, re-lock after being awakened and return.

```cpp
std::condition_variable cv;
cv.wait(lock, [] { return ready; }); // 只有后面的function是true才能拿锁
```

Be sure to check the conditions because there will be spurious wakeup (false wakeup)
> The conditions may have been changed by other threads when awakened

## `notify()`

- notify_one(): wake up a waiting thread
- notify_all(): wake up all waiting threads

The general usage is to do something after the current thread cv gets the lock, and then notify other threads.

# atomic

## Initialization
```cpp
std::atomic<int> cnt{0};
std::atomic<bool> done{false};
```

## `load` / `store`
```cpp
int x = cnt.load();
cnt.store(10);

bool b = done.load();
done.store(true);
```

## `exchange`
```cpp
int old = cnt.exchange(5); // 替换新值并返回旧值
```

## CAS (Compare and Swap)
```cpp
std::atomic<int> x{10};
int expected = 10;

// x = (x's old value == expected) ? 20 : x
bool ok = x.compare_exchange_strong(expected, 20);
```

# RMW (Read-Modify-Write)
```cpp
std::atomic<int> x{10};
int old_value = x.fetch_add(10); // old_value: 10, x: 20
```
