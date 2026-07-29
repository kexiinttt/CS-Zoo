# mutex

## `std::mutex`
```cpp
std::mutex lock;

lock.lock(); // 拿锁，拿不到就阻塞
bool ok = lock.try_lock();  //尝试拿锁，拿不到立即返回
lock.unlock();  // 解锁
```

## `std::lock_guard<std::mutex>`
- 更安全的RAII写法
- 不允许用户自己unlock
- 适用于整个作用域都要锁住
```cpp
std::mutex lock;
{
    std::lock_guard<std::mutex> guard(lock); // 拿锁
    ...
} // 解锁
```

## `std::unique_lock<std::mutex>`
它也会自动解锁，但额外支持：
- 延迟加锁
- 手动解锁再加锁
- 配合 condition_variable
- 所有权移动
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

condition_variable 用于：一个线程等待某个条件成立，另一个线程在条件变化时通知它。

一般而言都会和`std::unique_lock<std::mutex>`配合使用。

## `wait()`

释放锁并阻塞，被唤醒后重新加锁再返回。

```cpp
std::condition_variable cv;
cv.wait(lock, [] { return ready; }); // 只有后面的function是true才能拿锁
```

一定要检查条件因为会有spurious wakeup（假唤醒）
> 被唤醒时条件可能又被别的线程改掉了

## `notify()`

- notify_one()：唤醒一个等待线程
- notify_all()：唤醒所有等待线程

一般用法就是当前线程cv拿锁之后做一些事情，然后notify其他线程。

# atomic

## 初始化
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
