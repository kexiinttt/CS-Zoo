# Lambda 本质

C++ lambda 本质上是一个由编译器生成的匿名类对象，也就是一个 **closure object**。

```cpp
auto f = [x](int y) {
    return x + y;
};
```

可以粗略理解成编译器生成了类似下面的类：

```cpp
class __Lambda {
private:
    int x_;  // 捕获的变量成为 closure object 的成员

public:
    __Lambda(int x) : x_(x) {}

    int operator()(int y) const {
        return x_ + y;
    }
};

auto f = __Lambda{x};
```

所以 lambda 不是“函数”，而是一个 **可调用对象**。它能被调用，是因为它重载了 `operator()`。

---

# Lambda 的基本语法

```cpp
[capture](parameters) specifiers -> return_type {
    body
};
```

```cpp
[]      // 捕获列表
()      // 参数列表
mutable // 可选修饰符
noexcept // 可选异常说明
-> T    // 可选返回类型
{}      // 函数体
```

---

# 捕获方式

### 不捕获

不捕获任何外部变量。这种 lambda 可以隐式转换成普通函数指针：

```cpp
auto f = [](int x) {
    return x * 2;
};
```

```cpp
int (*fp)(int) = [](int x) { return x * 2; };
```

但只要有捕获，就不能转换成函数指针。

### 按值捕获

按值捕获是**在 lambda 创建时**，把变量拷贝进 closure object。
> 创建之后再修改也不会影响 lambda 内部存储的值。

```cpp
int x = 10;
auto f = [x] {
    return x;
};

x = 20;
std::cout << f(); // 10
```

```cpp
class __Lambda {
    int x_;
public:
    __Lambda(int x) : x_(x) {}
    int operator()() const { return x_; }
};
```

### 按引用捕获

按引用捕获时，lambda 内部引用的是外部变量本身。它可以看到外部变量后续变化。

```cpp
int x = 10;
auto f = [&x] {
    return x;
};

x = 20;
std::cout << f(); // 20
```

```cpp
class __Lambda {
    int& x_;
public:
    __Lambda(int& x) : x_(x) {}
    int operator()() const { return x_; }
};
```


> [!IMPORTANT]
> 注意生命周期问题：
> ```cpp
> std::function<int()> bad() {
>     int x = 10;
>     return [&x] {
>         return x; // dangling reference
>     };
> }
> ```
> 这里 `x` 在函数返回后已经销毁，lambda 持有悬垂引用，调用是未定义行为。

### 默认捕获

```cpp
[=]  // 默认按值捕获使用到的外部变量
[&]  // 默认按引用捕获使用到的外部变量
```

---

# Lambda 默认 `operator() const`

默认情况下，lambda 生成的调用运算符是 `const` 的。


```cpp
int x = 10;

auto f = [x]() -> int {
    // x++; // 编译错误
    return x;
};
```

```cpp
class __Lambda {
private:
    int x_;

public:
    int operator()() const {
        // x_++; // 错，因为 operator() 是 const 成员函数
        return x_;
    }
};
```

注意这里的 `const` 限制的是 closure object 自己的成员，也就是按值捕获进来的那份副本。所以，按值捕获的变量默认不能在 lambda 内修改。

---

# `mutable`：可以修改按值捕获的副本

如果想修改按值捕获进来的副本，需要使用 `mutable`：

```cpp
int x = 10;

auto f = [x]() mutable {
    x++;
    return x;
};

std::cout << f(); // 11
std::cout << f(); // 12
std::cout << x;   // 10，外部 x 不变
```

等价理解：

```cpp
class __Lambda {
private:
    int x_;

public:
    int operator()() {
        x_++;
        return x_;
    }
};
```

`mutable` 的效果是把 `operator() const` 变成非 `const operator()`。

它不是让外部变量变 mutable，而是让 lambda 可以修改自己内部保存的那份副本。

---

# 按引用捕获时可以修改外部变量

```cpp
int x = 10;

auto f = [&x]() {
    x++;
};
```

虽然 `operator()` 默认仍然是 `const`，但这里修改的不是 closure object 的成员本身，而是成员引用所指向的外部对象。

```cpp
class __Lambda {
private:
    int& x_;

public:
    void operator()() const {
        x_++; // OK，引用本身不可重新绑定，但引用指向的对象可以修改
    }
};
```

---

# 捕获 `const` 变量

```cpp
const int x = 10;

auto f = [x]() mutable {
    // x++; // 仍然编译错误
};
```

即使加了 `mutable`，如果捕获进来的成员本身是 `const int`，仍然不能修改。

因为 `mutable` 只是去掉 `operator()` 的 `const`，不会去掉变量自身的 `const`。

---

# 初始化捕获 (generalized capture)

初始化捕获：

```cpp
auto f = [x = 42] {
    return x;
};
```

移动捕获：

```cpp
std::unique_ptr<int> p = std::make_unique<int>(10);

auto f = [p = std::move(p)] { // unique_ptr 无法按值传递，因为无法拷贝
    return *p;
};
```

表达式捕获：

```cpp
auto f = [value = expensive_compute()] {
    return value;
};
```

---

# Lambda 与 `this` 捕获

在成员函数里：

```cpp
class Foo {
private:
    int x = 10;

public:
    auto getLambda() {
        return [this] { // 捕获的是当前对象指针
            return x;
        };
    }
};
```

⚠️ 如果对象已经销毁，lambda 里再访问 `this` 就是悬垂指针。

可以用 `[*this]` 捕获当前对象的副本，但是会涉及到拷贝，需要注意性能。

```cpp
auto getLambda() {
    return [*this] { // 捕获的是当前对象的副本
        return x;
    };
}
```

---

# `auto`

lambda 参数可以写 `auto`，这相当于创建一个带模板的`operator()`函数。

```cpp
auto add = [](auto a, auto b) {
    return a + b;
};

add(1, 2);       // int
add(1.5, 2.5);   // double
```

```cpp
class __Lambda {
public:
    template <typename T, typename U>
    auto operator()(T a, U b) const {
        return a + b;
    }
};
```
