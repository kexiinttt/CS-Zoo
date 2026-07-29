# RAII

Resource Acquisition Is Initialization 资源获取即初始化。

把资源的生命周期，绑定到对象的生命周期。对象构造时拿到资源，对象析构时自动释放资源。

```cpp
void func_without_RAII() {
    int* p = new int(42);
    if (some_condition()) {
        return;
    }
    delete p; // 资源管理的语句可能没法执行
}

void func_with_RAII() {
    std::unique_ptr<int> p = std::make_unique<int>(42);
    if (some_condition()) {
        return;
    }
    // 对象 p 在退出函数时析构，那么所管理的资源也就自动释放了
}
```
---

# 析构函数
其中对象的**析构函数**是资源释放的天然位置，因为C++有deterministic destruction (确定性的析构时机)，保证析构函数一定会在某些情况运行，从而保证内部管理资源销毁的步骤一定会执行。

> 比如Python中资源可能会在后面被GC统一回收，但是C++保证析构的时候就直接回收。

```cpp
class File {
private:
    FILE* _fp;
public:
    File(const char * name) {
        _fp = fopen(name, "r");
    }

    ~File() {
        if (_fp) { fclose(_fp); }
    }
}
```

---

# 异常安全

```cpp
void func_without_RAII() {
    m.lock();
    do_something(); // 如果抛出异常可能直接return，那么可能死锁
    m.unlock();
}

void func_with_RAII() {
    std::lock_guard<std::mutex> lock(m);
    do_something(); // 如果抛出异常，退出函数的时候lock_guard对象调用析构函数会释放锁
}
```

---

# 注意事项

## 析构函数不要抛出异常

因为如果在异常传播过程中，析构函数再抛异常，会导致 std::terminate()。

## 资源所有权

如果资源只能被一个对象拥有，那么需要注意disable copy & assign constructor，避免复制。

## virtual 析构函数

对于需要被继承的类，其析构函数一般定义成virtual，这样保证派生类资源能被正确的释放。

```cpp
class Base {
    virtual ~Base() {...}
}

class Derive : public Base {
    ~Derive() {...}
}

{
    std::unique_ptr<Base*> ptr = std::make_unique<Base>();
}
// ptr 调用 Derive::~Derive() 正确释放了资源
// 如果不是virtual的那么调用 Base::~Base() 可能出错
```

## 手动控制释放

有时不一定是函数执行完成才释放，可能希望提前/延迟释放。

```cpp
void process() {
    {
        LargeBuffer buf(100000000);
        do_heavy_work(buf);
    } // 提前释放

    do_other_work();
}
```

## 不一定是销毁对象

比如资源池，在用完之后不是销毁资源，而是归还资源，那么析构函数里就不能delete对象，而应该是归还。

```cpp
struct PoolDeleter {
    ObjectPool* pool{};
    void operator()(Message* p) const {
        pool->release(p);
    }
};
```

## 注意跨线程

RAII 最舒服的情况是：对象在哪个线程创建，在哪个线程使用，最后在那个线程作用域结束时析构。但跨线程以后，问题会复杂很多。

* 资源被放到queue中，当前线程已经结束，但是资源处理在另一个线程
* 析构函数不是在哪个线程调用都可以的
    * 析构太重，影响某些线程 latency
	* 析构访问 thread-local state，出错
	* 析构需要在指定 reactor thread 上执行
	* 析构里 close socket / free big memory 导致尾延迟抖动

## 资源释放本身很重，析构里做完不一定合适

有的线程是需要很高的延迟要求的，如果在这种线程里执行析构函数管理耗时耗力的资源，那么就会产生latency和线程卡顿。
