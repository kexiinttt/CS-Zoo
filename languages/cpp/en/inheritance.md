# Public inheritance

Public inheritance should first express the **is-a** relationship, not just for code reuse. To achieve reuse, composition should be considered.

## LSP (Liskov Substitution Principle)

If `Derived` public inherits `Base`, that means: **Anywhere `Base` is used, it can theoretically be safely replaced with `Derived`**.

Parent classes usually imply some conventions:

- What effect should be achieved after calling this function?
- Which inputs are allowed
- Whether an exception will be thrown
- What invariants should the object state hold?

Subclasses cannot break these conventions at will:
- Subclasses cannot require stronger preconditions than the parent class
- Subclasses cannot give weaker postconditions
- Subclasses cannot destroy the invariants of the parent class

A classic counterexample is as follows: mathematically square is a rectangle, but behaviorally it does not satisfy the contract.

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

If you assign a `Derived` object **by value** to a `Base` object, the derived part will be cut off.

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

At this time, there is only `x` left in `b` but no `y`. Because the space of `b` can only fit `Base`, there is no place to put the extra parts.

## How to avoid

If a type is designed to be a polymorphic base class, it should generally avoid using it by value and instead use it in the following ways:

- `Base&`
- `const Base&`
- `Base*`
- `std::unique_ptr<Base>`
- `std::shared_ptr<Base>`

## Why by value is not possible but by pointer/reference is possible

When passing by value, a new `Base` object is really created, and its memory size, member layout, etc. are all fixed.

However, references and pointers do not create `Base` objects, but use a third party to point/reference a `Derived` object, whose memory size and member layout are the contents of `Derived`.

---

# Don't use virtual function in construction/destruction

In constructors and destructors, virtual calls are not "dynamically dispatched to the most derived class" as you usually think.

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

When creating a `Derived` object, `f()` is called during the base class construction phase, which usually results in `Base::f()`, not `Derived::f()`.
- When the base class is constructed, the derived part has not yet been initialized.
- When the base class is destructed, the derived part has been destroyed first

---

# Multiple Inheritance Multiple inheritance

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

In this way, only one shared copy of `A` is retained in `D`.

> [!NOTE]
> The virtual base class is constructed by the **lowest final derived class**. For example, if the constructors of A, B, C, and D initialize x to 1, 2, 3, and 4 respectively, then the constructor of D will eventually be called, that is, x=4. If D does not initialize x, then B and C will not be called (because the middle ones are hidden, BC has no right to initialize the shared virtual base class A), the compiler will try to call the default constructor of A.


Virtual inheritance usually also brings:

- More complex object layouts
- More complex construction rules
- Higher cost of understanding

---

# Name hiding

As long as a function with the same name appears in a derived class, all overloads with the same name in the base class may be hidden.

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

At this time, both `Base::f(int)` and `Base::f(double)` are hidden.


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

Composition represents a **has-a** relationship. Inheritance represents an **is-a** relationship.

Inheritance mainly reuses "interface + implementation + relationship type"; combination reuses "capabilities".

## *Prefer composition over inheritance*

#### The coupling of inheritance is too strong

Subclasses and parent classes are tightly bound. Once the parent class is changed, the subclasses may be affected. Especially when the parent class is a "big and heavy" base class, the subclass will be forced to accept a lot of things that it does not need.

#### Inheritance exposes implementation details

Subclasses often rely on the internal behavior of the parent class, which will cause a fragile base class problem between the subclass and the parent class: a seemingly harmless change in the base class may break many derived classes.

#### The combination can replace components at runtime and perform dependency injection (Dependency Injection)

For example, a class can have a Logger, and then the user decides whether to pass in a ConsoleLogger or a FileLogger at runtime.

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


