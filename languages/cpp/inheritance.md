# 公有继承

public 继承首先应该表达 **is-a** 关系，而不是单纯为了代码复用。对实现复用，应考虑 composition。

## LSP (Liskov Substitution Principle)

如果 `Derived` public 继承 `Base`，那意味着：**任何使用 `Base` 的地方，理论上都可以安全替换成 `Derived`**。

父类通常隐含了一些约定：

- 这个函数调用后应该满足什么效果
- 哪些输入是允许的
- 是否会抛异常
- 对象状态应保持哪些不变式

子类不能随便破坏这些约定：
- 子类不能比父类要求更强的前置条件
- 子类不能给出更弱的后置条件
- 子类不能破坏父类的不变式

一个经典的反例如下：数学上 square 是 rectangle，但行为上并不满足契约。

```cpp
class Rectangle {
public:
    virtual void setWidth(int w) { width_ = w; }
    virtual void setHeight(int h) { height_ = h; }

protected:
    int width_ = 0;
    int height_ = 0;
};

class Square : public Rectangle {
public:
    void setWidth(int w) override {
        width_ = height_ = w;
    }

    void setHeight(int h) override {
        width_ = height_ = h;
    }
};
```

---

# Object Slicing

如果把一个 `Derived` 对象 **按值** 赋给一个 `Base` 对象，那么派生部分会被切掉。

```cpp
class Base {
public:
    int x = 1;
};

class Derived : public Base {
public:
    int y = 2;
};

Derived d;
Base b = d;  // object slicing
```

这时候 `b` 里只剩下 `x` 而没有了 `y`。因为 `b` 的空间只能放得下 `Base`，额外的部分没地方放。

## 如何避免

如果一个类型设计成了多态基类，那一般应该避免按值而是通过以下方式使用：

- `Base&`
- `const Base&`
- `Base*`
- `std::unique_ptr<Base>`
- `std::shared_ptr<Base>`

## 为什么按值不行但是按指针/引用可以

按值传递的时候是真的新建了一个 `Base` 对象，其内存大小，成员布局等都是固定的。

但是引用和指针并不创建 `Base` 对象，而是用一个第三方去指向/引用一个 `Derived` 对象，其在内存大小，成员布局都是 `Derived` 的内容。

---

# 不要在构造/析构里用virtual function

在构造函数和析构函数里，虚调用不会像你平时想的那样“动态分派到最派生类”。

```cpp
class Base {
public:
    Base() { f(); }
    virtual void f() { std::cout << "Base\n"; }
    virtual ~Base() { f(); }
};

class Derived : public Base {
public:
    void f() override { std::cout << "Derived\n"; }
};
```

创建 `Derived` 对象时，基类构造阶段调用 `f()`，通常会走 `Base::f()`，不是 `Derived::f()`。
- 基类构造时，派生部分还没完成初始化
- 基类析构时，派生部分已经先被销毁了

---

# Multiple Inheritance 多重继承

```cpp
class A {};
class B {};
class C : public A, public B {}
```

## Diamond Problem

```cpp
class A {
public:
    int x = 0;
};

class B : public A {};
class C : public A {};
class D : public B, public C {};

D d; // 此时 `D` 里会有两份 `A` 子对象
```

```
    A
   / \
  B   C
   \ /
    D
```

## Virtual Inheritance

```cpp
class A {
public:
    int x = 0;
};

class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {};
```

这样 `D` 中只保留一份共享的 `A`。

> [!NOTE]
> 虚基类由**最底层的最终派生类**负责构造。比如A，B，C，D的构造函数分别初始化x为1，2，3，4，那么最终会调用D的构造函数，即x=4。如果D没有初始化x，那么也不会调用B，C的（因为中间的都被隐藏，BC没有权利初始化共享的虚基类A），编译器会尝试调用A的默认构造。


虚继承通常也会带来：

- 更复杂的对象布局
- 更复杂的构造规则
- 更高的理解成本

---

# 名字隐藏（name hiding）

派生类里只要出现同名函数，就可能把基类同名重载全部隐藏掉。

```cpp
class Base {
public:
    void f(int) {}
};

class Derived : public Base {
public:
    void f(std::string) {}
};

Derived d;
d.f("abc"); // ✅
d.f(1);     // ❌
```

这时 `Base::f(int)` 和 `Base::f(double)` 都被隐藏了。


```cpp
class Derived : public Base {
public:
    using Base::f; // 引入Base的函数到该作用域
    void f(std::string) {}
};

Derived d;
d.f("abc"); // ✅ Derived::f(std::string)
d.f(1);     // ✅ Base::f(int)
```

---

# Composition vs Inheritance

Composition 表示的是一种 **has-a** 的关系。而 Inheritance 表示的是一种 **is-a** 的关系。

继承主要复用了“接口+实现+关系类型”；组合复用的是“能力”。

## *Prefer composition over inheritance*

#### 继承的耦合太强了

子类和父类绑定得很紧。父类一改，子类可能就受影响。尤其当父类是一个“很大很重”的基类时，子类会被迫接受很多自己不需要的东西。

#### 继承会暴露实现细节

子类往往会依赖父类内部行为，这样会让子类和父类之间形成**脆弱基类问题（fragile base class problem）**：基类一个看似无害的改动，可能破坏很多派生类。

#### 组合可以运行时替换部件并做依赖注入 (Dependency Injection)

比如一个类可以拥有一个Logger，然后运行时由用户决定传入的是ConsoleLogger还是FileLogger。

```cpp
class ILogger {
public:
    virtual ~ILogger() = default;
    virtual void log() = 0;
};

class ConsoleLogger : public ILogger {
public:
    void log() override {...};
}

class FileLogger : public ILogger {
public:
    void log() override {...};
}

class Service {
private:
    ILogger* _logger;
public:
    Service(ILogger* logger): _logger(logger) {...}

    void setLogger(ILogger* logger) {
        _logger = logger;
    }

    void run() {
        ...
        _logger->log();
    }
}

ConsoleLogger consoleLogger;
FileLogger fileLogger;

Service svc(&consoleLogger);
svc.run();   // 用 ConsoleLogger

svc.setLogger(&fileLogger);
svc.run();   // 运行时切换成 FileLogger
```

