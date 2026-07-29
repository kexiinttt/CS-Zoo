# member initializer list

The member initializer list is usually used first in the constructor, because the member object and base class part have to be initialized before entering the constructor body.

If you do not do it in the initialization list, but write the assignment in the constructor body, then in many cases it is not "initialization", but "default construction first, then assignment", which is usually more efficient, and some members cannot do this at all.

## What content must use member initializer list

* const
* Quote
* Member object without default constructor
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
* Base class
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

## Initialization sequence

Not in the order of writing, but in the base class first, and then in the order of member declaration.

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

# `class` is not just "object-oriented syntax"

`class` in C++ not only provides API and member variables, it also includes:
- Object layout (layout) → memory size occupied by the object
- Access control → Whether it can be moved and whether it can be copied
- Resource Management → RAII
- Life cycle management
- Polymorphic abstraction → Whether there is a virtual table, whether runtime polymorphism or static polymorphism should be used
- ABI → Whether it can be used stably across dynamic libraries

---

# Three major characteristics of object-oriented

## Encapsulation

Encapsulation is to put data and methods of operating data together, hide internal implementation details, and expose only necessary interfaces to the outside world. Please refer to [Encapsulation](./encapsulation.md).

## Inheritance

Inheritance is to allow subclasses to obtain the interface or implementation of the parent class, thereby establishing a hierarchical relationship. Inheritance is not only code reuse, but more importantly, it expresses the **is-a** relationship. If it is just for code reuse, in many cases the composition **has-a** relationship should be given priority. Please refer to [Inheritance](./inheritance.md).

## Polymorphism

The same interface shows different behaviors on different objects. Please refer to [Polymorphism](./polymorphism.md).

---

# `class` vs `struct`

At the C++ language level, the main difference between `class` and `struct` is only the default value:
- `class` members and inheritance are private by default
- The members and inheritance of `struct` are public by default.

Generally speaking, only design choices are made:

- `struct` → plain data / aggregate / data structure
- `class` → A type with invariants, encapsulation, and behavior

---

# special member functions

The "object semantics" of a class are largely determined by the following five special member functions:

* Destructor: destructor
* copy constructor: copy constructor
* copy assignment: copy assignment operator
* Move constructor: move constructor
* Move assignment: move assignment operator

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
> The purpose of using `noexcept` is to allow the compiler to give priority to moving rather than copying. However, users need to ensure that the code does not generate exceptions (such as performing very simple operations), otherwise it will be easy to terminate the program.

## Rule of Zero

If possible, try not to let the class write resource management by itself, but hand over resources to member objects for management.

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

**If the class directly manages primitive resources, such as raw pointers, file handles, sockets, etc., then these five special member functions must usually be carefully defined.**

For default copy construction/default copy assignment:
* value type → assignment
* Object type → object's own copy construction/copy assignment
* Naked pointer → ⚠️ Shallow copy, only copies the pointer address

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

At this time, for i2, the default copy structure is used, which is equivalent to:
```cpp
// i2.p = i1.p
Item::Item(const Item& i1) : x(i1.x), s(i1.s), p(i1.p) {}
```
At this time, the p of i2 and the p of i1 point to the same address, and there is a danger of double free.

### Customize
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

### Using Smart Pointer
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

# `this` pointer

Each **non-static** member function can be understood as receiving an implicit parameter `this`. Similar to the class method in Python, the first parameter is `self`.

## `this` in `const` member function
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
For `T::f()`:
- Can call other const member functions
- Cannot call other non-const member functions
- Can access member variables
- Member variables cannot be changed (unless it is `mutable`)

## ref-qualifier: Distinguish interfaces based on the left and right values of the object

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

- lvalue object returns lvalue reference
- const lvalue objects return const references
- Rvalue objects can move internal resources out

---

# PImpl (Pointer to Implementation)

Hide the implementation details of the class in another internal implementation class, and only put a pointer in the header file. It is generally used for stable class interface design in large projects.

## Why use PImpl

Let's say I have a large class
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
Problems with the above code include:
* Many implementation details are exposed in the header file
* Once HeavyType, vector, and private members change, all .cpp containing this header file may need to be recompiled
* If you are making a library/SDK, there will also be ABI issues

So the idea of PImpl is:
* Do not put these private implementation details in the header file
* Just put a pointer to the "implementation object"

In this way, Widget only exposes stable interfaces, and the specific implementation and data are hidden in Impl.

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
> Since we used Impl's pointer in Widget before implementing it, we need to declare it explicitly.

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

## PImpl Advantages

#### Reduce compilation dependencies
If the Widget has many private members and relies on heavy header files, such as third-party library header files or large internal class definitions:
- When PImpl is not used, as long as these private implementations are changed, all include widget.h places may be recompiled
- After using PImpl, widget.h only needs to know that Impl is a class, and the specific implementation is hidden in widget.cpp. When the private implementation changes, it usually only needs to recompile .cpp

#### Hidden implementation
Users can only see that there is an Impl and do not know the specific implementation and members.

#### ABI stable
If the lib is released in binary form (such as .so), changes in the object layout of the class will change the ABI, causing the old program to be incompatible with the new library. But if you use PImpl, since it is a pointer, no matter how Impl changes, it will not affect the layout of the Widget, and the ABI will be stable.

## PImpl Disadvantages

#### One more level of access
When calling, it is called through `impl_->func()`.

#### Extra space
Because Impl is often dynamically allocated (such as `impl_(std::make_unique<Impl>())`), it requires additional memory allocation and also needs to manage the life cycle of the resource, so it is more complicated.


## Destructor

Since Impl is forward declared, we cannot use the default destructor in .h because `std::unique_ptr<Impl>` needs to know the complete definition of Impl when it is destroyed. So we have to define it in the .cpp file.

```cpp
// widget.cpp
Widget::~Widget() = default;
```

---

# EBO (Empty Base Optimization)

The empty class object itself usually takes up at least 1 byte, but if the empty class is used as a base class, the compiler can often "optimize" it.

```cpp
struct Empty {};

class X : private Empty {
    int value;
};
```

In this way, `X` has the opportunity to only occupy the space required by `int` (plus alignment), without leaving additional space for `Empty`. Many policy / allocator / comparator types are themselves empty. In this case private inheritance is sometimes more space efficient than composition.

## `[[no_unique_address]]`

```cpp
struct Empty {};

class X {
private:
    [[no_unique_address]] Empty e_;
    int value_;
};
```

Without this attribute, although e_ is an empty class member, it usually still occupies some space. After adding `[[no_unique_address]]`, the compiler can "stuff" e_ into the gaps of other members, or even make it take up no extra space. At this time, this object may only have an int size.

`[[no_unique_address]]` can also be used in other situations. The essence is that a certain member does not need an independent address space. It can be "stuffed in" using tail padding, gap alignment and other methods to reduce space usage.

---

# `std::variant`

Please refer to [std::variant](./std-variant-and-visit.md).

---

# `()` vs `{}`

* `()` → parentheses / direct initialization
* `{}` → list / uniform initialization

## narrowing conversion

```cpp
int x = 3.14;   // ✅ 发生截断
int y(3.14);    // ✅ 发生截断
int z{3.14};    // ❌ 禁止 narrowing
```

- `()`: Many implicit conversions will still be accepted
- `{}`: stricter on conversions that may lose information

## `initializer_list` constructor

In list-initialization, the compiler will give priority to constructors that accept std::initializer_list. So once a class provides such a constructor, many {} calls will prefer that constructor.

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

* `T()` → value-initialization `int x = int(); // x == 0`
* `T{}` → value-initialization `int y{}; // y == 0`
* `T obj();` → This is a function declaration rather than a definition object
* `T obj{};` → list-initialization

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

## `auto` and `{}` combination

```cpp
auto x{1};      // int
auto y = {1};   // std::initializer_list<int>

auto a{1, 2};    // 错误
auto b = {1, 2}; // std::initializer_list<int>
```

## Default initialization

When a class does not have a default constructor
* `A a;` → The member is not initialized and can be any value
* `A a{};` → will set the member to the default value
* `A a();` → ❌, this is the function declaration
* `A a = A();` → will set the member to the default value

When a class has a default constructor
* `A a;` → call the default constructor
* `A a{};` → call the default constructor
* `A a();` → ❌, this is the function declaration
* `A a = A();` → call the default constructor
