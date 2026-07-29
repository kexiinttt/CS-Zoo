What is #`std::any`

`std::any` is a type erasure container. Its core function is:

- Ability to save a value of "any type" at runtime
- But this type must satisfy **CopyConstructible**
- When reading, you need to explicitly know the target type and retrieve it through `std::any_cast<T>`

You can think of it as "a box that can hold many different types of objects, but the box itself does not expose internal object types at compile time."

## Usage

```cpp
#include <typeinfo>
#include <any>
#include <string>

int main() {
    // 初始化
    std::any a;
    std::any b = 123;
    std::any c = std::string("x");

    // 重新赋值：旧对象销毁，新对象放入
    a = 10;

    // 是否有保存内容
    if (!a.has_value()) {}

    // 获取type：⚠️ 此处获取的是typeid，并不是human-readable
    std::cout << a.type().name() << "\n";
    if (a.type() == typeid(int)) {}

    // 取值：类型不匹配会抛出 std::bad_any_cast
    int x = std::any_cast<int>(a);

    // 取指针：类型不匹配只会设为nullptr而不会异常
    auto p = std::any_cast<int>(&a);

    // 清空内容
    a.reset();
}
```
---

# `std::any` vs `void*` vs `std::variant`

## vs `void*`

`void*` can also "install the address of anything", but its problem is obvious:

- Don’t know the true type of the object
- Not responsible for object life cycle
- Prone to incorrect conversions
- Not safe, requires manual memory management

And `std::any`:

- Manage object life cycle yourself
- Record the true type of the object
- Provides controlled type checking and conversion

So `std::any` is inherently much safer than `void*`.

## vs `std::variant`

`std::variant<int, std::string, double>`:

- **Know at compile time which types** may be stored
- Type collection fixed
- Access is more efficient and type-safe

`std::any`:

- **It is not known during compilation what type** will be saved
- Greater flexibility
- But you have to make your own type judgment when getting the value
- Usually more expensive

---

# Limitations

## Performance

The price of `std::any` usually includes:

- Indirect calls caused by type erasure
- Heap allocation may occur
- `any_cast` requires runtime type checking

Therefore, it prioritizes flexibility rather than extreme performance.

## Must know the type

You have to get whatever type you put in it.

## Reference and copy

```cpp
std::any a = std::string("hello");
auto s_copy = std::any_cast<std::string>(a);
auto& s_ref = std::any_cast<std::string&>(a);
s_ref += " world";
```

---

# Type erasure

The key reason why it can "install any type" is:

- Only the unified interface is exposed to the outside world
- Hide the true type internally
- Different types of objects are managed uniformly through an abstract base class or function table

There are two common ideas:

* **Virtual function base class + template derived class**
* **Function pointer table (similar to vtable) + raw storage**

For industrial-grade codes, function pointer tables are generally used for implementation. The reasons are:
* Virtual function methods require vptr to occupy additional space
* The function pointer table is **static**, that is, the same type shares the same handler, but the virtual function method needs to create space for each object
* Function pointer methods include **Small Buffer Optimization (SBO)**

which is roughly similar to
```cpp
// 类型拥有一个方法指针，指向“如何操作类型”的静态表
// 同时有一个buffer/ptr_to_buffer用来做SBO
//  * 如果元素很小，那么放在buffer（栈）
//  * 如果元素很大，那么放在堆，然后用指针指向
struct Any {
    void* buffer_or_ptr_to_buffer;
    const Handler* handler;
};

// 像 interface 定义了不管什么类型都要支持的一些方法
struct Handler {
    void (*destroy)(Any&);
    void (*copy)(const Any&, Any&);
    void (*move)(Any&, Any&);
    const std::type_info& (*type)();
    void* (*get)(Any&);
};

// 对不同类型都编译一个**静态**的表出来
// 这样存int就查HandlerFor<int>，存str就查HandlerFor<str>
template<typename T>
struct HandlerFor {
    static void destroy(Any& a);
    static void copy(const Any& src, Any& dst);
    static void move(Any& src, Any& dst);
    static const std::type_info& type();
    static void* get(Any& a);
};
```

---

# Simple implementation

For simple implementation, you can use the method of **virtual function base class + template derived class**.

`Any` does not know the real type, but can correctly copy, destroy and query the type through polymorphism:
- An abstract base class `Base` provides the interface
- One `Holder<T> : Base` for each actual type
- `Any` internally holds `Base*` or `std::unique_ptr<Base>`

```cpp
#include <iostream>
#include <memory>
#include <typeinfo>
#include <utility>
#include <string>
#include <stdexcept>

class bad_any_cast : public std::bad_cast {
public:
    const char* what() const noexcept override {
        return "bad_any_cast";
    }
};

class Any {
private:
    struct Base {
        virtual ~Base() = default;
        virtual std::unique_ptr<Base> clone() const = 0;
        virtual const std::type_info& type() const noexcept = 0;
    };

    template <typename T>
    struct Holder : Base {
        T value;

        template <typename U>
        explicit Holder(U&& v) : value(std::forward<U>(v)) {}

        std::unique_ptr<Base> clone() const override {
            return std::make_unique<Holder<T>>(value);
        }

        const std::type_info& type() const noexcept override {
            return typeid(T);
        }
    };

    std::unique_ptr<Base> ptr_;

public:
    Any() = default;

    Any(const Any& other)
        : ptr_(other.ptr_ ? other.ptr_->clone() : nullptr) {}

    Any(Any&& other) noexcept = default;

    template <
        typename T,
        typename D = std::decay_t<T>,
        typename = std::enable_if_t<!std::is_same_v<D, Any>>
    >
    Any(T&& value)
        : ptr_(std::make_unique<Holder<D>>(std::forward<T>(value))) {}

    Any& operator=(const Any& other) {
        if (this != &other) {
            ptr_ = other.ptr_ ? other.ptr_->clone() : nullptr;
        }
        return *this;
    }

    Any& operator=(Any&& other) noexcept = default;

    template <
        typename T,
        typename D = std::decay_t<T>,
        typename = std::enable_if_t<!std::is_same_v<D, Any>>
    >
    Any& operator=(T&& value) {
        ptr_ = std::make_unique<Holder<D>>(std::forward<T>(value));
        return *this;
    }

    bool has_value() const noexcept {
        return static_cast<bool>(ptr_);
    }

    void reset() noexcept {
        ptr_.reset();
    }

    void swap(Any& other) noexcept {
        ptr_.swap(other.ptr_);
    }

    const std::type_info& type() const noexcept {
        return ptr_ ? ptr_->type() : typeid(void);
    }

    template <typename T>
    friend T* any_cast(Any* a);

    template <typename T>
    friend const T* any_cast(const Any* a);
};

template <typename T>
T* any_cast(Any* a) {
    if (!a || !a->ptr_) {
        return nullptr;
    }

    using U = std::remove_cv_t<std::remove_reference_t<T>>;
    if (a->ptr_->type() != typeid(U)) {
        return nullptr;
    }

    auto holder = static_cast<Any::Holder<U>*>(a->ptr_.get());
    return &holder->value;
}

template <typename T>
const T* any_cast(const Any* a) {
    if (!a || !a->ptr_) {
        return nullptr;
    }

    using U = std::remove_cv_t<std::remove_reference_t<T>>;
    if (a->ptr_->type() != typeid(U)) {
        return nullptr;
    }

    auto holder = static_cast<const Any::Holder<U>*>(a->ptr_.get());
    return &holder->value;
}

template <typename T>
T any_cast(Any& a) {
    using U = std::remove_reference_t<T>;
    auto p = any_cast<U>(&a);
    if (!p) {
        throw bad_any_cast();
    }
    return static_cast<T>(*p);
}

template <typename T>
T any_cast(const Any& a) {
    using U = std::remove_reference_t<T>;
    auto p = any_cast<U>(&a);
    if (!p) {
        throw bad_any_cast();
    }
    return static_cast<T>(*p);
}

template <typename T>
T any_cast(Any&& a) {
    using U = std::remove_reference_t<T>;
    auto p = any_cast<U>(&a);
    if (!p) {
        throw bad_any_cast();
    }
    return static_cast<T>(std::move(*p));
}
```

> [!IMPORTANT]
> Because `Any` needs to support copying, it must support `clone()`.
>
> The static type of `ptr_` is just `std::unique_ptr<Base>`, only that it points to some `Base`, not the real derived type (`Holder<T>`). Therefore, the base class must provide a virtual function: `virtual std::unique_ptr<Base> clone() const = 0;`, so that different `Holder<T>` can know how to copy itself.
