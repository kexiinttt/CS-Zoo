# member initializer list

构造函数里通常优先使用 member initializer list，因为成员对象和基类部分在进入构造函数体之前就已经要完成初始化。

如果不在初始化列表里做，而是在构造函数体里写赋值，那很多时候并不是“初始化”，而是“先默认构造，再赋值”，这通常更低效，而且有些成员根本没法这么做。

## 哪些内容必须使用 member initializer list

* const
* 引用
* 没有默认构造函数的成员对象
```cpp
class B {
public:
    B(int x) {}
};

class A {
public:
    A(int x) : b(x) {}
private:
    B b;
};
```
* 基类
```cpp
class Base {
public:
    Base(int x) {}
};

class Derived : public Base {
public:
    Derived(int x) : Base(x) {}
};
```

## 初始化顺序

并不是按照写的顺序，而是先**基类**，然后按**成员的声明顺序**。

```cpp
class A {
public:
    A() : y(2), x(y) {} // x先声明所以先初始化，此时y还未初始化
private:
    int x;
    int y;
};
```

---

# `class` 不只是“面向对象语法”

C++中的`class`不只是提供API和成员变量，他还包括了：
- 对象布局 (layout) &rarr; 对象占用内存大小
- 访问控制 &rarr; 可否移动，可否复制
- 资源管理 &rarr; RAII
- 生命周期管理
- 多态抽象 &rarr; 是否有virtual table，是否应该用运行时多态还是静态多态
- ABI &rarr; 是否能跨动态库稳定使用

---

# 面向对象三大特性

## 封装 (Encapsulation)

封装就是把数据和操作数据的方法放在一起，并且隐藏内部实现细节，只对外暴露必要接口。请参考[Encapsulation](./encapsulation.md)。

## 继承 (Inheritance)

继承就是让子类获得父类的接口或实现，从而建立层次关系。继承不只是代码复用，更重要的是表达 **is-a** 关系，如果只是为了复用代码，很多时候其实应该优先考虑 composition **has-a** 关系。请参考[Inheritance](./inheritance.md)。

## 多态 (Polymorphism)

同一个接口在不同对象上表现出不同的行为。请参考[Polymorphism](./polymorphism.md)。

---

# `class` vs `struct`

C++ 语言层面，`class` 和 `struct` 的主要区别只有默认值：
- `class`的成员和继承都是默认private
- `struct`的成员和继承都是默认public

一般而言只会在设计上进行选择：

- `struct` &rarr; plain data / aggregate / data structure
- `class` &rarr; 有不变量、有封装、有行为的类型

---

# special member functions

一个类的“对象语义”很大程度上由下面五个特殊成员函数决定：

* 析构函数: destructor
* 拷贝构造: copy constructor
* 拷贝赋值: copy assignment operator
* 移动构造: move constructor
* 移动赋值: move assignment operator

```cpp
class T {
private:
    int* data;

public:
    T() : data(new int(0)) {}

    ~T() {
        delete data;
    }

    T(const T& other) : data(new int(*other.data)) {}

    T& operator=(const T& other) {
        if (this != &other) {
            delete data;
            data = new int(*other.data);
        }
        return *this;
    }

    T(T&& other) noexcept : data(other.data) {
        other.data = nullptr;
    }

    T& operator=(T&& other) noexcept {
        if (this != &other) {
            delete data;
            data = other.data;
            other.data = nullptr;
        }
        return *this;
    }
};
```

> [!NOTE]
> 使用`noexcept`是为了让编译期可以优先选择使用移动而非拷贝。但是用户需要保证代码确实不会产生异常（比如执行非常简单的操作），否则会容易terminate程序。

## Rule of Zero

如果可能，尽量让类不要自己手写资源管理，而是把资源交给成员对象去管理。

```cpp
class User {
public:
    User(std::string name, std::vector<int> scores)
        : name_(std::move(name)), scores_(std::move(scores)) {}

private:
    std::string name_;
    std::vector<int> scores_;
};
```

## Rule of Five

**如果类直接管理原始资源，例如裸指针、文件句柄、socket 等，那通常就必须认真定义这五个特殊成员函数。**

对于默认拷贝构造/默认拷贝赋值：
* 值类型 &rarr; 赋值
* 对象类型 &rarr; 对象自己的拷贝构造/拷贝赋值
* 裸指针 &rarr; ⚠️ 浅拷贝，只复制指针地址

```cpp
struct Item {
    int x;
    string s;
    int* p;
}

int* p = new int{10};
Item i1 = {10, "hello", p};
Item i2 = i1;
```

此时对于i2，使用默认拷贝构造，相当于：
```cpp
// i2.p = i1.p
Item::Item(const Item& i1) : x(i1.x), s(i1.s), p(i1.p) {}
```
此时i2的p和i1的p指向同一地址，会有double free的危险。

### 自定义
```cpp
struct Item {
    int x;
    string s;
    int* p;

    Item(int x, const string& s, int val): x(x), s(s), p(new int{val}) {}
    ~Item() {
        delete p;
    }

    Item(const Item& other): x(other.x), s(other.x), p(nullptr) {
        if (other.p) {
            p = new int{*(other.p)};
        }
    }

    Item& operator=(const Item& other) {
        if (this == &other) {
            return *this;
        }

        int* new_p = nullptr;
        if (other.p) {
            new_p = new char{*(other.p)};
        }

        x = other.x;
        s = other.s;
        delete p;
        p = new_p;

        return *this;
    }

    Item(Item&& other) noexcept: x(other.x), s(move(other.s)), p(other.p) {
        other.p = nullptr;
    }

    Item& operator=(Item&& other) noexcept {
        if (this == &other) {
            return *this;
        }

        delete p;
        x = other.x;
        s = move(other.s);
        p = other.p;
        
        other.p = nullptr;

        return *this;
    }
}
```

### Copy-and-Swap
```cpp
struct Item {
    int x;
    string s;
    int* p;

    Item(int x, const string& s, int val): x(x), s(s), p(new int{val}) {}
    ~Item() {
        delete p;
    }

    Item(const Item& other): x(other.x), s(other.x), p(nullptr) {
        if (other.p) {
            p = new int{*(other.p)};
        }
    }

    Item(Item&& other) noexcept: x(other.x), s(move(other.s)), p(other.p) {
        other.p = nullptr;
    }

    // ⚠️ 此处先按值传递，可以同时支持copy/move
    // 当传入左值，则使用已经定义的copy constructor深拷贝临时量然后swap
    // 当传入右值，则使用已经定义的move constructor创建临时量然后swap
    Item& operator=(Item other) {
        swap(other);
        return *this;
    }

    void swap(Item& other) {
        using std::swap;
        swap(x, other.x);
        swap(s, other.s);
        swap(p, other.p);
    }
}
```

### 使用Smart Pointer
```cpp
struct Item {
    int x;
    string s;
    unique_ptr<int> p;

    Item(int x, const string& s, int val) : x(x), s(s), p(make_unique<int>(val)) {}
    ~Item() = default;

    Item(const Item& other): x(other.x), s(other.s), p(other.p ? make_unique<int>(*other.p) : nullptr) {}

    Item& operator=(const Item& other) {
        if (this == &other) {
            return *this;
        }

        x = other.x;
        s = other.s;
        p = other.p ? make_unique<int>(*other.p) : nullptr;

        return *this;
    }

    Item(Item&&) noexcept = default;
    Item& operator=(Item&&) noexcept = default;
}
```

---

# `this` 指针

每个**非静态**成员函数，背后都可以理解为接收了一个隐式参数 `this`。类似于Python中的类方法第一个参数是`self`。

## `const` 成员函数中的 `this`
```cpp
class T {
public:
    int x = 0;
    mutable int cnt = 0;

    void f() const {
        // this 的类型是 const T* const，不能改变指向，也不能改变对象内容
        int temp = x; // ✅
        // x = 10; ❌
        cnt++; // ✅
        // g(); ❌
    }

    void g() {
        // this 的类型是 T* const，不能改变指向，但是可以改变对象内容
        int temp = x; // ✅
        x = 10; // ✅
        cnt++; // ✅
        f(); // ✅
    }
};
```
对于`T::f()`而言：
- 可以调用其他const的成员函数
- 不可以调用其他非const的成员函数
- 可以访问成员变量
- 不可以改变成员变量（除非是`mutable`）

## ref-qualifier：根据对象左右值区分接口

```cpp
class Widget {
public:
    std::string& name() & { return name_; }
    const std::string& name() const & { return name_; }
    std::string&& name() && { return std::move(name_); }

private:
    std::string name_;
};
```

- 左值对象返回左值引用
- const 左值对象返回 const 引用
- 右值对象可以把内部资源 move 出去

---

# PImpl (Pointer to Implementation)

把类的实现细节藏到另一个内部实现类里，头文件里只放一个指针，一般用于大项目里的稳定类接口设计。

## 为什么用 PImpl

假设我有一个很大的类
```cpp
class Widget {
public:
    ...
    void doWork();

private:
    std::vector<int> data_;
    std::string name_;
    HeavyType heavy_;
};
```
上述代码的问题包括：
* 头文件里暴露了很多实现细节
* 一旦 HeavyType、vector、私有成员变化，所有包含这个头文件的 .cpp 可能都要重新编译
* 如果你在做库/SDK，还会有ABI问题

所以 PImpl 的思路是：
* 头文件里不要放这些私有实现细节
* 只放一个指向“实现对象”的指针

这样Widget只暴露稳定的接口，而具体的实现和数据都藏在Impl中。

```cpp
// widget.h
#include <memory>

class Widget {
public:
    ...
    void doWork();

private:
    class Impl;                  // 前向声明
    std::unique_ptr<Impl> impl_; // 只放一个指针
};
```
> [!NOTE]
> 由于我们在实现Impl之前就在Widget中使用了它的指针，所以我们需要显示声明它。

```cpp
// widget.cpp
#include "widget.h"
#include <vector>
#include <string>
#include <iostream>

class Widget::Impl {
public:
    std::vector<int> data_;
    std::string name_;

    void doWorkImpl() {
        std::cout << "working...\n";
    }
};

void Widget::doWork() {
    impl_->doWorkImpl();
}
```

## PImpl 优点

#### 降低编译依赖
如果 Widget 的私有成员很多，而且依赖重头文件，比如第三方库头文件或大型内部类定义：
- 不使用 PImpl 时，只要这些私有实现改了，所有 include widget.h 的地方都可能重新编译
- 用了 PImpl 后，widget.h 只需要知道 Impl 是个类，具体实现藏在 widget.cpp，私有实现变动时，通常只需要重编译 .cpp

#### 隐藏实现
用户只能看到有一个Impl，并不知道具体实现和成员。

#### ABI稳定
如果该lib是以二进制形式发布（比如.so），那么类的对象布局变化会改变ABI，导致旧的程序和新库不兼容。但是如果使用PImpl，由于是一个指针，所以Impl无论怎么改变都不会影响Widget的布局，ABI也就稳定了。

## PImpl 缺点

#### 多一层访问
调用的时候是通过`impl_->func()`来调用的。

#### 额外空间
因为Impl常常是动态分配（比如`impl_(std::make_unique<Impl>())`），所以需要额外的内存分配，同时还需要管理资源的生命周期，所以更加复杂。


## 析构函数

由于前向声明了 Impl，所以我们不能在 .h 中使用默认析构函数，因为`std::unique_ptr<Impl>`在销毁时需要知道 Impl 的完整定义。因此我们必须定义在 .cpp 文件中。

```cpp
// widget.cpp
Widget::~Widget() = default;
```

---

# EBO（Empty Base Optimization）

空类对象本身通常也至少要占 1 byte，但如果把空类作为基类，编译器常常能把它“优化掉”。

```cpp
struct Empty {};

class X : private Empty {
    int value;
};
```

这样 `X` 有机会只占 `int` 所需空间（加上对齐），而不额外为 `Empty` 单独留空间。很多 policy / allocator / comparator 类型本身就是空的。这时 private 继承有时比组合更节省空间。

## `[[no_unique_address]]`

```cpp
struct Empty {};

class X {
private:
    [[no_unique_address]] Empty e_;
    int value_;
};
```

如果没有这个属性，e_ 虽然是空类成员，也通常还是会占一点位置。加了 `[[no_unique_address]]` 后，编译器可以把 e_ “塞进”别的成员的空隙里，甚至让它不额外占空间。此时这个对象可能就只有一个int大小。

`[[no_unique_address]]` 还可以用于其他情况，本质就是要求某个成员不需要独立的地址空间，可以利用尾部填充，空隙对齐等方法“塞进去”从而减少空间使用。

---

# `std::variant`

请参考[std::variant](./std-variant-and-visit.md)。

---

# `()` vs `{}`

* `()` &rarr; parentheses / direct initialization
* `{}` &rarr; list / uniform initialization

## narrowing conversion

```cpp
int x = 3.14;   // ✅ 发生截断
int y(3.14);    // ✅ 发生截断
int z{3.14};    // ❌ 禁止 narrowing
```

- `()`：很多隐式转换会照样接受
- `{}`：对可能丢失信息的转换更严格

## `initializer_list` 构造函数

在 list-initialization 里，编译器会优先考虑接受 std::initializer_list 的构造函数。所以一旦类提供了这种构造函数，很多 {} 调用都优先调用该构造函数。

```cpp
std::vector<int> a(2, 1); // constructor => [1, 1]
std::vector<int> b{2, 1}; // initializer_list<int> constructor => [2, 1]
```

```cpp
template<typename T>
class A {
public:
    A(std::initializer_list<T>) {
        std::cout << "initializer_list\n";
    }
};
```


## `T()` vs `T{}` vs `T obj();` vs `T obj{}`

* `T()` &rarr; value-initialization `int x = int(); // x == 0`
* `T{}` &rarr; value-initialization `int y{}; // y == 0`
* `T obj();` &rarr; 此处为函数声明而非定义对象
* `T obj{};` &rarr; list-initialization

## `explicit`

```cpp
class A {
public:
    A(int x) {}
    A(initializer_list<int> init) {}
};

class B {
public:
    explicit B(int x) {}
    explicit B(int x, int y) {}
};

class C {
public:
    explicit C(int x) {}
    explicit C(int x, int y) {}
    C(initializer_list<int> init) {}
};

int main() {
    // 没有explicit的时候可以隐式转换
    A a1{1};       // A(initializer_list)
    A a2 = {1};    // A(initializer_list)
    A a3(1);       // A(int)
    A a4 = 1;      // A(int)

    // 没有initializer_list的时候都使用普通构造函数
    B b1{1};            // ✅ B(int)
    // B b2 = {1};      // ❌ B(int)
    B b3(1);            // ✅ B(int)
    // B b4 = 1;        // ❌ B(int)
    B b5{1, 1};         // ✅ B(int, int)
    // B b6 = {1, 1};   // ❌ B(int, int) 因为没有initializer_list，所以仍然使用普通构造
    B b7(1, 2);         // ✅ B(int, int)

    // 有initializer_list的时候{...}优先使用list构造函数
    C c1{1};            // ✅ C(initializer_list)
    C c2 = {1};         // ✅ C(initializer_list)
    C c3{1, 2};         // ✅ C(initializer_list)
    C c4 = {1, 2};      // ✅ C(initializer_list)
}
```

## `auto` 和 `{}` 组合

```cpp
auto x{1};      // int
auto y = {1};   // std::initializer_list<int>

auto a{1, 2};    // 错误
auto b = {1, 2}; // std::initializer_list<int>
```

## 默认初始化

当class没有默认构造函数的时候
* `A a;` &rarr; 成员未被初始化，可以是任意值
* `A a{};` &rarr; 会把成员设置为默认值
* `A a();` &rarr; ❌，这是函数声明
* `A a = A();` &rarr; 会把成员设置为默认值

当class有默认构造函数的时候
* `A a;` &rarr; 调用默认构造
* `A a{};` &rarr; 调用默认构造
* `A a();` &rarr; ❌，这是函数声明
* `A a = A();` &rarr; 调用默认构造
