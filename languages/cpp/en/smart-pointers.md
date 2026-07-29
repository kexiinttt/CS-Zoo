# TL;DR
* `unique_ptr` means exclusive ownership, **cannot be copied but can be moved**, has low overhead, and is usually the default choice
* `shared_ptr` represents shared ownership, life cycle management through reference counting, suitable for multiple owners, but has additional costs and possible circular references
* `weak_ptr` is a non-owning observer of shared_ptr managed objects, does not increase the reference count, and is often used to break reference cycles.

# Why do we need smart pointer?

To prevent memory leaks, smart pointer can automatically manage the life cycle of dynamic memory. The core idea is: **Bind resource release to the object life cycle and use RAII to automatically recycle resources**.

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

RAII (Resource Acquisition Is Initialization) resources are acquired when the object is constructed and released when the object is deconstructed. Common RAII resources include socket, mutex lock, database connection, etc.

Please refer to [RAII](./raii.md) for details.

---

# Category

## `unique_ptr`
### The only one who owns
The pointer is automatically deleted when its life cycle ends.
```cpp
std::unique_ptr<int> p = std::make_unique<int>(42);
```

> [!NOTE]
> It is recommended to use `std::make_unique<int>(42);` instead of `std::unique_ptr<int>(new int(42))`. The main reason is because `make_unique<T>` is safer.
> ```cpp
> foo(make_unique<T>(), make_unique<U>());
> foo(unique_ptr<T>(new T()), unique_ptr<U>(new U()));
> ```
> The second one is not safe, because the compiler may `new T()` first, and then the unique_ptr has not pointed to the object, and the compiler will execute `new U()`. If there is an exception at this time, T will leak memory. `make_unique<T>` is more like an atomic operation, ensuring that the pointer points to the content.

### Cannot copy
```cpp
std::unique_ptr<int> p1 = std::make_unique<int>(42);
std::unique_ptr<int> p2 = p1; // ❌
```

### Can be moved

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

### As function parameter/return value
#### Pass by value → Take ownership
```cpp
void func(std::unique_ptr<int> p) {
    std::cout << *p << std::endl;
}

auto p = std::make_unique<int>(42);
func(std::move(p)); // 之后p就是nullptr了，因为已经移交给func，func退出时析构了p，释放了资源
```

#### Pass by reference → Borrow ownership
```cpp
void func(std::unique_ptr<int>& p) {
    std::cout << *p << std::endl;
}

auto p = std::make_unique<int>(42);
func(p); // 之后p仍然有效
```

#### Pass by raw pointer → You cannot use smart pointer methods, you can only use raw pointer operations.
```cpp
void func(int* p) {
    std::cout << *p << std::endl;
}

auto p = std::make_unique<int>(42);
func(p.get());
```

#### Return value (return by value)

Generally, it is returned by value, and the caller takes over ownership.
```cpp
std::unique_ptr<int> make_value() {
    return std::make_unique<int>(42);
}

auto p = make_value(); // p接管了临时对象的所有权
```

### Custom deleter
```cpp
// ⚠️ 此处使用函数指针类型而非函数类型，即 &fclose 而非 fclose
std::unique_ptr<FILE, decltype(&fclose)> fp(fopen(...), &fclose);
```

> [!NOTE]
> Function type != function pointer type
> ```cpp
> int add(int x, int y) { return x + y; }
> 
> int(int, int) p = add; // ❌，函数类型不能当作对象类型
> int (*p)(int, int) = add; // ✅，函数指针类型可以是对象类型
> auto p = add; // 推导为 int(*)(int, int)
> ```
> The function body is not an ordinary object that can be copied and stored in a variable.

## `shared_ptr`

Multiple objects share the same resource. Multiple points point to the same memory, and there is an internal counter to maintain the number of references. If there is one more shared_ptr copy, the counter will increase, and vice versa, until the counter reaches 0, the memory will be released.

### Control block

The control block is a piece of "shared management information" maintained on the heap.
* Strong reference counting (number of shared_ptr)
* Weak reference count (number of weak_ptr)
* deleter
* allocator (allocator, optional)
* Sometimes also contains the object body (common in make_shared)

### `make_shared`

It is more recommended to use `make_shared`, because it can allocate control blocks and objects at once, thus reducing one heap allocation.
```cpp
auto p1 = std::make_shared<int>(100);
auto p2 = std::shared_ptr<int>(new int(100));
```

### As function parameter/return value

#### Pass by value → shared ownership of function, reference count +1

```cpp
void func(std::shared_ptr<int> p) {
    std::cout << p.use_count() << std::endl;
}

auto p = std::make_shared<int>(100);
func(p);  // 2
```

> [!NOTE]
> Generally used to extend the life cycle of the object. Even if the pointers outside the function are invalid, as long as the function does not end, the strong reference count will not be 0, and the object will still survive until the function ends. **Guarantee that the object survives at least until the end of the function**.

#### Pass by reference → does not increase the reference count, just borrows the pointer to access the memory

```cpp
void func(std::shared_ptr<int>& p) {
    std::cout << p.use_count() << std::endl;
}

auto p = std::make_shared<int>(100);
func(p);  // 1
```

#### Return value

Generally, it is returned by value, which means transferring ownership.

### Custom deleter
Same as `unique_ptr`.

### Multi-thread safe

Shared_ptr needs to be viewed at two levels in multi-threading:

* ✅ The reference counting level is thread-safe
    * Different shared_ptr instances pointing to the same control block can be copied/destroyed in different threads at the same time.
    * Count increment and decrement are atomic operations.
* ❌ Object contents are not automatically thread-safe
    * shared_ptr only ensures the safety of "who will destroy the object".
    * The read and write concurrency safety of *ptr is not guaranteed, mutex/atom synchronization is still required → If the same shared_ptr variable body is read and written by multiple threads at the same time, data competition is still possible
    * In which thread the last reference is released, object destruction occurs in which thread


### Circular reference
It means that objects are held by shared_ptr to each other, forming a ring, causing the reference count to never be 0 and the memory cannot be released.
```cpp
struct B;
struct A { std::shared_ptr<B> b; };
struct B { std::shared_ptr<A> a; };
```
The solution is to use `weak_ptr`, please refer to the following content for details.

## `weak_ptr`

weak_ptr is to solve the circular reference problem of shared_ptr.

### Does not own the object, just observes, does not increase the strong reference count

Control blocks have strong reference counting and weak reference counting. shared_ptr controls the strong reference count, and the object is destroyed when it is 0; weak_ptr controls the weak reference count, and the control block is released only when the strong and weak counts are both 0.
```cpp
auto sp = std::make_shared<int>(42); // 拥有对象
std::weak_ptr<int> wp = sp; // 观察对象
```

### Cannot be dereferenced directly

weak_ptr cannot directly `*ptr` or `ptr->`, because it observes the object and does not guarantee that the object still exists (if the strong reference count is 0, then the object has been destroyed, but the weak reference count is not 0, then the control block still exists).

```cpp
if (auto p = wp.lock()) { // lock() 返回一个 shared_ptr / nullptr
    std::cout << *p << "\n";
} else {
    std::cout << "object already destroyed\n";
}
```

---

# Note

1. ❌ `unique_ptr.get()` gets the raw ptr and then deletes it. As a result, the unique_ptr itself does not know that the content pointed to has been cleared. Delete again in the future and it will cause an error.
2. ❌ Using raw ptr to construct multiple `shared_ptr` will generate multiple control blocks, and finally the same block of memory will be deleted multiple times.
3. ⚠️ `get()` takes the raw ptr and essentially owns the resources; `release()` gives up ownership and returns the raw ptr, and then the user needs to manage it himself
4. ⚠️ The return value of `use_count()` cannot be used as the basis for business logic judgment, because it is an instantaneous value and the reference count will change at any time under concurrent conditions.
5. ⚠️ `shared_ptr` may still have memory leaks, such as circular references
6. ⚠️ `weak_ptr` will not extend the object life cycle

> [!NOTE]
> * `get()` → Get raw pointer, the object still exists
> * `release()` → Get raw pointer, the pointer becomes nullptr
> * `reset(ptr)` → Destroy the original managed resources and start managing new resources
