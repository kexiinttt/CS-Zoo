# `std::any` 是什么

`std::any` 是一个类型擦除（type erasure）容器，它的核心作用是：

- 能在**运行时**保存“某个任意类型”的一个值
- 但这个类型必须满足**可拷贝构造（CopyConstructible）**
- 读取时需要你显式知道目标类型，并通过 `std::any_cast<T>` 取出

你可以把它理解成“一个可以装很多不同类型对象的盒子，但盒子自己不在编译期暴露内部对象类型”。

## 使用方式

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

`void*` 也能“装任意东西的地址”，但它的问题很明显：

- 不知道对象真实类型
- 不负责对象生命周期
- 容易发生错误转换
- 不安全，需要手动管理内存

而 `std::any`：

- 自己管理对象生命周期
- 记录对象真实类型
- 提供受控的类型检查和转换

所以 `std::any` 本质上比 `void*` 安全得多。

## vs `std::variant`

`std::variant<int, std::string, double>`：

- **编译期就知道**可能存哪些类型
- 类型集合固定
- 访问更高效、更类型安全

`std::any`：

- **编译期不知道**会存什么类型
- 灵活性更强
- 但取值时要自己做类型判断
- 通常开销更大

---

# 局限性

## 性能

`std::any` 的代价通常包括：

- 类型擦除带来的间接调用
- 可能发生堆分配
- `any_cast` 需要做运行时类型检查

所以它更偏灵活性优先，不是极致性能优先。

## 必须知道类型

你放进去的是什么类型就得取什么类型。

## 引用和拷贝

```cpp
std::any a = std::string("hello");
auto s_copy = std::any_cast<std::string>(a);
auto& s_ref = std::any_cast<std::string&>(a);
s_ref += " world";
```

---

# 类型擦除

它之所以能“装任意类型”，关键在于：

- 对外只暴露统一接口
- 对内把真实类型隐藏起来
- 不同类型对象通过一个抽象基类或函数表来统一管理

常见思路有两种：

* **虚函数基类 + 模板派生类**
* **函数指针表（类似 vtable）+ 原始存储**

对于工业级代码，一般使用函数指针表来实现，原因：
* 虚函数的方法需要有 vptr 占用额外空间
* 函数指针表是 **static** 的，即相同类型共享同样的 handler，但是虚函数的方法需要为每一个对象创建空间
* 函数指针方法有 **Small Buffer Optimization (SBO)**

其大致上类似于
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

# 简单实现

为了简单实现，可以使用 **虚函数基类 + 模版派生类** 的方法。

`Any` 并不知道真实类型，但能通过多态正确复制、销毁和查询类型：
- 一个抽象基类 `Base` 提供接口
- 每种实际类型对应一个 `Holder<T> : Base`
- `Any` 内部持有 `Base*` 或 `std::unique_ptr<Base>`

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
> 因为 `Any` 要支持拷贝，所以必须要支持 `clone()`。
> 
> `ptr_` 的静态类型只是 `std::unique_ptr<Base>`，只知道它指向某个 `Base`，不知道真实派生类型（`Holder<T>`）。所以必须让基类提供一个虚函数：`virtual std::unique_ptr<Base> clone() const = 0;`，这样不同 `Holder<T>` 才知道怎么复制自己。
