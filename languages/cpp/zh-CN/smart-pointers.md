# TL;DR
* `unique_ptr` 表示独占所有权，**不能拷贝但可以移动**，开销小，通常是默认首选
* `shared_ptr` 表示共享所有权，通过引用计数管理生命周期，适合多个 owner，但有额外成本且可能出现循环引用
* `weak_ptr` 是对 shared_ptr 管理对象的非拥有观察者，不增加引用计数，常用于打破循环引用。

# 为什么需要smart pointer

防止内存泄漏，smart pointer可以自动管理动态内存的生命周期。其核心思想是: **把资源释放绑定到对象生命周期中，利用RAII自动回收资源**。

```cpp
void function() {
    int* ptr = new int(100);
    if (...) {
        ...  // if throw error, it won't run delete
    }
    delete ptr;
}
```

---

# RAII

RAII (Resource Acquisition Is Initialization) 资源在对象构造时获得，在对象解构时释放。常见的RAII的资源有socket, mutex lock, database connection等。

详情请参考[RAII](./raii.md)。

---

# 类别

## `unique_ptr`
### 唯一拥有
当指针生命周期结束时会自动delete。
```cpp
std::unique_ptr<int> p = std::make_unique<int>(42);
```

> [!NOTE]
> 推荐使用`std::make_unique<int>(42);`而不是`std::unique_ptr<int>(new int(42))`。主要原因是因为`make_unique<T>`更安全。
> ```cpp
> foo(make_unique<T>(), make_unique<U>());
> foo(unique_ptr<T>(new T()), unique_ptr<U>(new U()));
> ```
> 第二个不安全，因为编译器可能先`new T()`，然后unique_ptr还没指向对象，编译器就去执行`new U()`，如果此时异常那么T就内存泄漏了。`make_unique<T>`更像一个atomic operation，保证了指针指向内容。

### 不能拷贝
```cpp
std::unique_ptr<int> p1 = std::make_unique<int>(42);
std::unique_ptr<int> p2 = p1; // ❌
```

### 可以移动

```cpp
std::unique_ptr<int> p1 = std::make_unique<int>(42);
std::unique_ptr<int> p2 = std::move(p1); // 此时p1是nullptr
```

> [!NOTE]
> ```cpp
> unique_ptr<int> p1 = make_unique<int>(10);
> unique_ptr<int> p2;
> unique_ptr<int> p3；
>
> // p2 = p1; ❌ 这个是copy，只能move
> p2 = move(p1); // ✅ move 
> p3 = make_unique<int>(*p2); // ✅ make_unique相当于打包了valid的操作
> ```

### 作为函数参数/返回值
#### 按值传递 &rarr; 接管所有权
```cpp
void func(std::unique_ptr<int> p) {
    std::cout << *p << std::endl;
}

auto p = std::make_unique<int>(42);
func(std::move(p)); // 之后p就是nullptr了，因为已经移交给func，func退出时析构了p，释放了资源
```

#### 按引用传递 &rarr; 借用所有权
```cpp
void func(std::unique_ptr<int>& p) {
    std::cout << *p << std::endl;
}

auto p = std::make_unique<int>(42);
func(p); // 之后p仍然有效
```

#### 按原始指针传递 &rarr; 无法使用智能指针的方法，只能使用raw pointer的操作
```cpp
void func(int* p) {
    std::cout << *p << std::endl;
}

auto p = std::make_unique<int>(42);
func(p.get());
```

#### 返回值 (按值返回)

一般都是按值返回，调用者接管所有权。
```cpp
std::unique_ptr<int> make_value() {
    return std::make_unique<int>(42);
}

auto p = make_value(); // p接管了临时对象的所有权
```

### 自定义deleter
```cpp
// ⚠️ 此处使用函数指针类型而非函数类型，即 &fclose 而非 fclose
std::unique_ptr<FILE, decltype(&fclose)> fp(fopen(...), &fclose);
```

> [!NOTE]
> 函数类型 != 函数指针类型
> ```cpp
> int add(int x, int y) { return x + y; }
> 
> int(int, int) p = add; // ❌，函数类型不能当作对象类型
> int (*p)(int, int) = add; // ✅，函数指针类型可以是对象类型
> auto p = add; // 推导为 int(*)(int, int)
> ```
> 函数本体不是一块"可拷贝存放到变量里"的普通对象

## `shared_ptr`

多个对象共享拥有同一资源。多个指向同一个内存，内部有计数器维护引用次数，多一个shared_ptr拷贝计数器就增加，反之减少，直到计数器为0则释放内存。

### 控制块

control block 是在堆上维护的一块"共享管理信息"。
* 强引用计数 (shared_ptr 个数)
* 弱引用计数 (weak_ptr 个数)
* 删除器 (deleter)
* 分配器 (allocator，可选)
* 有时还包含对象本体 (make_shared 常见)

### `make_shared`

更推荐使用`make_shared`，因为可以**一次性分配控制块和对象**，从而减少一次堆分配。
```cpp
auto p1 = std::make_shared<int>(100);
auto p2 = std::shared_ptr<int>(new int(100));
```

### 作为函数参数/返回值

#### 按值传递 &rarr; 函数共享所有权，引用计数+1

```cpp
void func(std::shared_ptr<int> p) {
    std::cout << p.use_count() << std::endl;
}

auto p = std::make_shared<int>(100);
func(p);  // 2
```

> [!NOTE]
> 一般用于延长对象的生命周期，即使函数外面的指针都失效了，函数只要不结束，强引用计数就还不为0，对象就仍然存活直到函数结束。**保证对象至少存活到函数结束**。

#### 按引用传递 &rarr; 不增加引用计数，只是借用指针访问内存

```cpp
void func(std::shared_ptr<int>& p) {
    std::cout << p.use_count() << std::endl;
}

auto p = std::make_shared<int>(100);
func(p);  // 1
```

#### 返回值

一般都是按值返回，代表把所有权移交。

### 自定义deleter
同`unique_ptr`。

### 多线程安全

shared_ptr 在多线程里要分两层看：

* ✅ 引用计数层面是线程安全的
    * 指向同一控制块的不同 shared_ptr 实例，可在不同线程同时拷贝/销毁。
    * 计数增减是原子操作。
* ❌ 对象内容不是自动线程安全的
    * shared_ptr 只保证“谁来析构对象”这件事安全。
    * 不保证 *ptr 的读写并发安全，仍需 mutex/原子等同步 &rarr; 同一个 shared_ptr 变量本体被多个线程同时读写，仍可能数据竞争
    * 最后一个引用在哪个线程释放，对象析构就在哪个线程发生


### 循环引用
指对象彼此用 shared_ptr 持有，形成环，导致引用计数永远不为 0，内存无法释放。
```cpp
struct B;
struct A { std::shared_ptr<B> b; };
struct B { std::shared_ptr<A> a; };
```
解决办法是使用`weak_ptr`，详情参考下面内容。

## `weak_ptr`

weak_ptr 是为了解决 shared_ptr 的循环引用问题。

### 不拥有对象，只是观察，不增加强引用计数

控制块有强引用计数和弱引用计数。shared_ptr控制强引用计数，为0时对象销毁；weak_ptr控制弱引用计数，当强弱计数都为0时才会释放控制块。
```cpp
auto sp = std::make_shared<int>(42); // 拥有对象
std::weak_ptr<int> wp = sp; // 观察对象
```

### 不能直接解引用

weak_ptr 不能直接 `*ptr` 或者 `ptr->`，因为它是观察对象的，不保证对象还存在(如果强引用计数为0，那么对象已经被销毁了，但是弱引用计数不为0，那么控制块还存在)。

```cpp
if (auto p = wp.lock()) { // lock() 返回一个 shared_ptr / nullptr
    std::cout << *p << "\n";
} else {
    std::cout << "object already destroyed\n";
}
```

---

# 注意点

1. ❌ `unique_ptr.get()` 拿到raw ptr然后delete，导致unique_ptr本身并不知道指向的内容已经清除了，未来再次delete会出错
2. ❌ 使用raw ptr构造多个 `shared_ptr`，会产生多个控制块，最后同一块内存被删除多次
3. ⚠️ `get()` 是拿raw ptr，本质上还是拥有资源的；`release()` 是放弃所有权并返回raw ptr，之后需要用户自己管理
4. ⚠️ 不能用`use_count()`的返回值来作为业务逻辑判断依据，因为它是一个瞬时值，并发下引用计数会随时变化
5. ⚠️ `shared_ptr` 仍然可能内存泄漏，比如循环引用
6. ⚠️ `weak_ptr` 并不会让对象生命周期延长

> [!NOTE]
> * `get()` &rarr; 获取raw pointer，对象仍然存在
> * `release()` &rarr; 获取raw pointer，指针变为nullptr
> * `reset(ptr)` &rarr; 销毁原来管理的资源，开始管理新的资源
