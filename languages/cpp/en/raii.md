# RAII

Resource Acquisition Is Initialization Resource acquisition is initialization.

Bind the resource life cycle to the object life cycle. Resources are obtained when the object is constructed and automatically released when the object is destroyed.

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

# Destructor
The **destructor** of the object is the natural location for resource release, because C++ has deterministic destruction (deterministic destruction timing), which ensures that the destructor will definitely run under certain circumstances, thus ensuring that the steps for internal management of resource destruction will definitely be executed.

> For example, resources in Python may be recycled by GC later, but C++ guarantees that they will be recycled directly during destruction.

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

# Exceptionally safe

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

# Notes

## The destructor should not throw an exception

Because if the destructor throws an exception during the exception propagation process, it will cause std::terminate().

## Resource ownership

If the resource can only be owned by one object, you need to pay attention to disable copy & assign constructor to avoid copying.

## virtual destructor

For classes that need to be inherited, their destructors are generally defined as virtual to ensure that derived class resources can be released correctly.

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

## Manual control release

Sometimes it is not necessarily released until the function is executed, and you may want to release it early/delayed.

```cpp
void process() {
    {
        LargeBuffer buf(100000000);
        do_heavy_work(buf);
    } // 提前释放

    do_other_work();
}
```

## Does not necessarily mean destroying the object

For example, in a resource pool, the resources are not destroyed but returned after use. Then the object cannot be deleted in the destructor, but should be returned.

```cpp
struct PoolDeleter {
    ObjectPool* pool{};
    void operator()(Message* p) const {
        pool->release(p);
    }
};
```

## Pay attention to cross-threading

The most comfortable situation for RAII is: the object is created in which thread, used in which thread, and finally destroyed when the scope of that thread ends. But after cross-threading, the problem will be much more complicated.

* The resource is placed in the queue, the current thread has ended, but the resource is processed in another thread
* The destructor cannot be called in any thread
    * Destruction is too heavy, affecting the latency of some threads
	* Destruction access thread-local state, error
	* Destruction needs to be executed on the specified reactor thread
	* Close socket / free big memory in destructor causes tail delay jitter

## Resource release itself is very heavy, and it may not be appropriate to complete it in the destructor.

Some threads require very high latency requirements. If the destructor is executed in such a thread to manage time-consuming and labor-intensive resources, latency and thread lag will occur.
