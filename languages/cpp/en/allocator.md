# What is Allocator

In C++, **allocator** is an abstraction layer used to manage memory allocation and release. The most classic usage scenario is **STL containers**. These containers are not directly written to use `new` / `delete` to manage element memory, but use allocators to complete underlying memory operations.

> **"A set of policy interfaces for how containers apply for, release, construct, and destroy objects."**

It has two core meanings:

1. **Separate "memory management strategy" from "data structure logic"**
2. **Allow users to customize memory allocation methods**, such as:
  - memory pool
  - Arena on the stack
  - shared memory
  - NUMA-aware allocation
  - Aligned allocation
  - Debug allocator (statistics, tracking, leak detection)

---

# Why do we need Allocator?

If there is no allocator, this may be done directly inside the container:

```cpp
T* p = new T[n];
delete[] p;
```

The problems with doing this are numerous:

- Strong coupling between containers and memory management strategies
- Unable to flexibly replace memory sources
- Inconvenient for performance optimization
- Unable to perform unified memory statistics/debugging
- Difficult to support special scenarios (shared memory, fixed address pool, lock-free pool, etc.)

The emergence of Allocator makes STL containers more scalable.

---

# `malloc` / `new` What is

Before getting into allocators, let’s separate out a few concepts that are easily confused:

- `malloc` / `free`: C-style memory allocation and release
- `new` / `delete`: C++ object creation and destruction expressions

## `malloc`

`malloc` comes from the C standard library:

```cpp
#include <cstdlib>

int* p = (int*)std::malloc(sizeof(int) * 10);
std::free(p);
```

What it does is simple:

- Apply for a piece of raw memory with a specified number of bytes
- Return `void*`
- **Constructor will not be called**
- Use `free` when releasing

So `malloc` is more like:

> Give me a byte area. It does not control whether there is an object in this memory or how the object is constructed.

## `new`

`new` is an expression in C++, for example:

```cpp
Person* p = new Person();
delete p;
```

It does one more critical thing than `malloc`:

1. First allocate enough raw memory
2. Then call the constructor on that memory

`delete` is the opposite:

1. Call the destructor first
2. Release the underlying memory

## The difference between `malloc/free` and `new/delete`

1. `malloc/free` only handles memory
2. `new/delete` handles memory + constructs objects

Common comparisons:

- `malloc` returns `void*`; `new` returns the correct type pointer
- `malloc` returns `nullptr` when it fails; `new` defaults to `std::bad_alloc` when it fails.
- `malloc` does not call construction/destruction; `new/delete` does
- `malloc/free` cannot directly express "create an object"; `new/delete` can

## `::operator new`

This name can easily be confused with `new`, but they are not the same thing.

```cpp
void* raw = ::operator new(sizeof(Person));
::operator delete(raw);
```

`::operator new` is only responsible for:

- Allocate a large enough piece of raw memory

It does not automatically call the constructor.  
So it is closer to `malloc`, except that it belongs to the C++ allocation mechanism and usually throws an exception when it fails instead of returning a null pointer.

If you really want to "get the original memory first and then manually construct the object", you usually use placement new:

```cpp
void* raw = ::operator new(sizeof(Person));
Person* p = new (raw) Person();  // placement new，在指定地址构造对象

p->~Person();
::operator delete(raw);
```

This pattern is very close to allocator:

- Get the original memory first
- Construct the object explicitly again
- Destroy object
- Release original memory

## `new` vs placement new

The two names are very similar, but the difference is very important:

- `new`: **Allocate memory + Construct object**
- placement new: **Construct objects on existing memory**

Normal `new`:

```cpp
Person* p = new Person();
```

It usually does two things:

1. Call `operator new` to allocate memory
2. Call the constructor of `Person`

And placement new:

```cpp
void* raw = ::operator new(sizeof(Person));
Person* p = new (raw) Person();
```

The `new (raw) Person()` here will not apply for memory again.  
It just tells the compiler:

> Please construct a `Person` object at the address pointed to by `raw`.

So placement new is more suitable for:

- allocator
- memory pool
- arena
- Handwriting container
- Scenarios where "allocating memory" and "constructing objects" need to be separated

### Different release methods

Common `new`:

```cpp
Person* p = new Person();
delete p;
```

placement new：

```cpp
void* raw = ::operator new(sizeof(Person));
Person* p = new (raw) Person();

p->~Person();
::operator delete(raw);
```

Because placement new is not responsible for allocating that memory, it cannot be simply understood as "allocate a `delete p` and that's it."  
Usually requires:

1. Manually call the destructor
2. Then use the corresponding method to release the original memory

## Why STL containers do not directly depend on `new[]`

This is the key to understanding allocators.

For example, when expanding `vector`, it is often necessary to:

- Apply for a larger piece of raw memory first
- Just move existing elements over and construct them one by one
- Unused locations do not construct objects temporarily

But `new T[n]` will directly:

- Allocate `n` `T` space
- By the way, construct all these `n` objects

What a container really needs is often: **Allocate raw memory** and **Control when objects are constructed/destroyed** These two things are separated.

This is also the basis of allocator design.

---

# What problem does Allocator solve?

## Raw memory management

- Apply for a piece of raw memory of **unconstructed object**
- Release this original memory

> ⚠️ This step is not "creating an object", it is just getting a large enough byte area.

```cpp
#include <memory>
#include <iostream>

int main() {
    std::allocator<int> alloc;

    int* p = alloc.allocate(3);   // 只分配内存，不初始化

    std::construct_at(p, 10);     // 在特定位置初始化
    std::construct_at(p + 1, 20);
    std::construct_at(p + 2, 30);

    for (int i = 0; i < 3; ++i) { // 在特定位置销毁对象
        std::destroy_at(p + i);
    }

    alloc.deallocate(p, 3);      // 回收内存空间
}
```

This process is divided into four steps:

1. `allocate(3)`: Get original memory
2. `construct_at(...)`: Construct objects one by one
3. `destroy_at(...)`: Destruction of objects one by one
4. `deallocate(...)`: Release original memory

## Object life cycle management

In the past, the allocator interface was also responsible for:

- `construct` → Construct object on allocated raw memory
- `destroy` → Explicitly destroy the object

This part is usually done in modern C++ with `std::allocator_traits` + placement new.

---

# `std::allocator`

```cpp
template<class T>
struct allocator {
    using value_type = T;

    T* allocate(std::size_t n);
    void deallocate(T* p, std::size_t n);
};
```

> A very important reason why containers do not use `new[]` / `delete[]` is: **Containers often need to "allocate a space, but only construct part of the elements"**. `new[N]` will directly allocate space of N object size and construct N objects.

> [!NOTE]
> `new` includes space allocation and object construction; `::operator new` only includes space allocation. Generally speaking, it can be considered that `::operator new` is the underlying implementation of `allocate()`.

---

# `std::allocator_traits`

A unified adapter provided by the standard library to shield the details of different allocators. You can understand it as: "The container should not directly assume what the allocator looks like, but call it uniformly through allocator_traits."

```cpp
std::allocator_traits<Alloc>
```

Its main functions are:

- Provide a unified interface for all allocators
- If the allocator has its own special construct/destroy, use it
- If not, fall back to standard default behavior (essentially placement new / explicit destruction)

## Common interfaces

```cpp
using traits = std::allocator_traits<Alloc>;

T* p = traits::allocate(alloc, n);
traits::deallocate(alloc, p, n);

traits::construct(alloc, p, args...);
traits::destroy(alloc, p);
```

There is no need to assume that an allocator must have a complete interface when implementing a container. As long as it meets the basic requirements, the traits can be completed as far as possible. See [basic requirements for a custom allocator](#basic-requirements-for-a-custom-allocator) for details.

## Template copy construction of allocator

```cpp
template<typename U>
MyAllocator(const MyAllocator<U>&) {}
```

This is to support **rebind** of allocator.

Because memory is not necessarily allocated only to `T` inside the container. For example some container node structures may not be `T` itself, but internal node types. Therefore the allocator needs to be able to convert from `Allocator<T>` to `Allocator<U>`.

> For example, if I have a linked list `LinkedListNode<int>`, then T is an int, but the Node node is initialized, and its value is of type int.

---

# `rebind`

Because the type received by the container is T, but memory for T is not necessarily allocated internally.

> For example, if I have a linked list `LinkedListNode<int>`, then T is an int, but the Node node is initialized, and its value is of type int.

In the old allocator model, it was often written:

```cpp
template<typename U>
struct rebind {
    using other = MyAllocator<U>;
};
```

It means "Rebind the current allocator to another type `U`."

For example:

- The current allocator is `MyAllocator<int>`
- Memory must be allocated inside the container for `Node`
- Just get `MyAllocator<Node>` through `rebind`

In modern C++ this is usually done via `allocator_traits<Alloc>::rebind_alloc<U>`:

```cpp
using NodeAlloc = std::allocator_traits<Alloc>::template rebind_alloc<Node>;
```

> [!NOTE]
> `::template` is used here because `rebind_alloc` is a member template function, and the previous part is not a known type, but the template type used.
>
> ```cpp
> using A = std::allocator_traits<int>;
> using B = std::allocator_traits<A>::rebind_alloc<Node>;
>
> template<typename Alloc>
> struct X {
>   using NodeAlloc = std::allocator_traits<Alloc>::template rebind_alloc<Node>;
> };
> ```

---

# Stateful allocator vs stateless allocator

## Stateless allocator（stateless allocator）

The allocator itself does not save any additional information. For example, if it simply calls global functions such as `::operator new`, then each allocator instance will be the same. It is equivalent to just defining the rules.

```cpp
template<typename T>
struct MyAlloc {
    using value_type = T;

    T* allocate(std::size_t n) {
        return static_cast<T*>(::operator new(n * sizeof(T)));
    }

    void deallocate(T* p, std::size_t) {
        ::operator delete(p);
    }
};

// a，b是一样的，没有成员变量，没有特殊的配置，都是直接操作内存
MyAlloc<int> a;
MyAlloc<int> b;
```

## Stateful allocator (stateful allocator)

The allocator object carries some data internally, which affects how it allocates memory, for example:

- Holds memory pool pointer
- holds arena pointer
- Hold shared memory handle
- Holds NUMA node information
- holds statistics context

It's equivalent to **not just rules, but also context**.

```cpp
struct MemoryPool {
    void* allocate(std::size_t bytes);
    void deallocate(void* p);
};

template<typename T>
struct PoolAlloc {
    using value_type = T;
    MemoryPool* pool;

    PoolAlloc(MemoryPool* p) : pool(p) {}

    T* allocate(std::size_t n) {
        return static_cast<T*>(pool->allocate(n * sizeof(T)));
    }

    void deallocate(T* p, std::size_t) {
        pool->deallocate(p);
    }
};

// a，b不一样，因为一个基于pool1，一个基于pool2
PoolAlloc<int> a(&pool1);
PoolAlloc<int> b(&pool2);
```

---

# allocator propagation (propagation rules)

When copy assignment/move assignment/swap occurs in a container, does the allocator propagate along with it?

Standards are controlled through a number of traits:

- `propagate_on_container_copy_assignment` → `a = b`
- `propagate_on_container_move_assignment` → `a = move(b)`
- `propagate_on_container_swap` → `swap(a, b)`

They are usually `std::true_type` or `std::false_type`.

- If propagation is true, then in the corresponding operation, a's allocator can be replaced by b's allocator, so a can use the same memory management context as b to manage that memory
- If propagation is false, then a retains its own allocator, and it may not be able to directly take over b's memory.

For example, if two vectors are executed:

```cpp
a = std::move(b);
```

- Can `a` take over the memory of `b`?
- If the allocator is stateless then it should be OK
- If the allocator has state and `pool1 != pool2`, then `a.allocator` theoretically does not know how to manage pool2

## `is_always_equal`

Any two allocator instances can be considered equivalent and can release the memory allocated by each other. This generally applies to stateless allocators.

```cpp
struct MyAlloc {
    using is_always_equal = std::true_type;
};
```

If allocators are always equivalent, the container can do more optimizations in scenarios such as move / swap, because it does not need to worry about "this memory can only be reclaimed by a specific allocator".

---

# Polymorphic Memory Resource (PMR)

C++17 feature; see [PMR documentation](https://en.cppreference.com/w/cpp/memory_resource) for details.

---

# Allocator common usage scenarios

**Performance Optimization**

- Reduce system calls
- Reduce malloc/free overhead
- Improve cache locality
- Reduce fragmentation
- Improve batch allocation efficiency

**Resource Isolation**

Different modules use different allocators:

- Convenient for statistics
- Easy to limit
- Easy to track leaks
- Facilitate fault location

**Special Memory Source**

- shared memory
- huge page
- NUMA node local memory
- GPU/pinned memory (usually requires more specialized interface)
- Fixed mapping area

**Debugging and Observability**

- Count the total number of allocations
- Statistical peak memory
- Mark the source of the call
- Do out-of-bounds detection
- Detect double release

> [!NOTE]
> Roughly calculate the dynamic memory occupied by the container.
>
> ```cpp
> #include <atomic>
> #include <iostream>
> #include <memory>
> #include <vector>
>
> struct AllocStats {
>     std::atomic<std::size_t> bytes{0};
> };
>
> template<typename T>
> struct CountingAllocator {
>     using value_type = T;
>
>     AllocStats* stats = nullptr;
>
>     CountingAllocator() = default;
>     explicit CountingAllocator(AllocStats* s) : stats(s) {}
>
>     template<typename U>
>     CountingAllocator(const CountingAllocator<U>& other) : stats(other.stats) {}
>
>     T* allocate(std::size_t n) {
>         std::size_t b = n * sizeof(T);
>         if (stats) stats->bytes += b;
>         return static_cast<T*>(::operator new(b));
>     }
>
>     void deallocate(T* p, std::size_t n) {
>         std::size_t b = n * sizeof(T);
>         if (stats) stats->bytes -= b;
>         ::operator delete(p);
>     }
> };
> ```

---

# Basic requirements for custom allocator

Minimum requirements typically include:

```cpp
template<typename T>
struct MyAlloc {
    using value_type = T;

    MyAlloc() = default;

    template<typename U>
    MyAlloc(const MyAlloc<U>&);

    T* allocate(std::size_t n);
    void deallocate(T* p, std::size_t n);
};
```

It is also often recommended to provide:

- Comparison operator `==` / `!=`
- `is_always_equal`
- propagation traits
- noexcept guarantee (write if you can)
- Behavior compatible with `allocator_traits`

---

# Allocator exception

Consider vector expansion:

1. allocate new memory
2. Move/copy one by one to construct new elements
3. If an exception is thrown halfway:
  - Destroy new elements that have been constructed
  - deallocate new memory
  - Keep old memory and old elements intact

This shows that allocator must work closely with object construction/destruction logic, and it often appears together with RAII.
