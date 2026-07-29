# 左值与右值是什么

可以先用一句话记：

- **左值**：有身份，能定位，通常能取地址的表达式
- **右值**：临时值，即将消失，可移动，通常不该被长期引用的临时表达式
  - pvalue：纯右值，临时值，比`1+1`, `x + y` 
  - xvalue：将亡值(expring value)，有对象身份但是可以被移动

## 左值 lvalue

左值通常有这些特点：

- 有名字，或者至少有明确、稳定的内存位置
- 可以出现在赋值号左边
- 通常可以取地址
- 生命周期通常不会在当前表达式结束时立刻消失

例子：

```cpp
// `x` 都是左值
int x = 10;
int y = x;
x = 20;
int* p = &x;

// `s[0]` 也是左值
std::string s = "hello";
s[0] = 'H';
```

## 右值 rvalue

右值通常有这些特点：

- 往往是临时结果
- 一般不能放在赋值号左边
- 生命周期通常很短
- 适合被“移动”而不是“拷贝”

例子：

```cpp
// `10` 是右值
int x = 10;
// `x + 1` 只是一个计算结果，没有独立名字，也是右值
int y = x + 1;
// `make_name()` 返回出来的那个临时 `std::string`，通常也看作右值
std::string make_name() {
    return "Alice";
}
```

---

# 左值引用和右值引用

## 左值引用 `T&`

左值引用通常只能绑定到左值。

```cpp
int x = 10;
int& ref = x; // ✅

int& ref = 10; // ❌
```

## 右值引用 `T&&`

右值引用通常绑定右值：

```cpp
int&& r = 10;
```

它的意义主要不是“给右值起名字”，而是为了支持：

- **move semantics (移动语义)**
- **perfect forwarding (完美转发)**

例如：

```cpp
std::string s1 = "hello";
std::string s2 = std::move(s1);
```

这里 `std::move(s1)` 把 `s1` 转成右值表达式，从而允许 `s2` 直接“偷走” `s1` 持有的资源，而不是深拷贝。

## const 左值引用 `const T&`

`const T&` 很特殊，它既可以绑定左值，也可以绑定右值。

```cpp
int x = 10;
const int& a = x;   // ✅
const int& b = 20;  // ✅
```

为什么可以绑定右值？

因为编译器会为右值生成一个临时对象，并延长它的生命周期到这个引用的作用域结束。

这也是为什么 `const T&` 常用于：

- 避免拷贝
- 接收各种类型的实参

例如：

```cpp
void print(const std::string& s) {
    std::cout << s << '\n';
}

std::string s = "abc";
print(s); // 左值
print("abc"); // 右值
```

> [!NOTE]
> **有名字的右值引用变量，本身是左值**。
>
> ```cpp
> int&& r = 10;
> ```
>
> 虽然 `r` 的类型是 `int&&`，但是**表达式 `r` 本身是左值**，因为它有名字，有稳定身份。
>
> ```cpp
> void f(int&)  { std::cout << "lvalue\n"; }
> void f(int&&) { std::cout << "rvalue\n"; }
>
> int main() {
>     int&& r = 10;
>     f(r);            // 调用 lvalue 版本
>     f(std::move(r)); // 调用 rvalue 版本
> }
> ```

## const 右值引用 `const T&&` (基本不用)

右值引用的主要价值在于"可以修改这个即将销毁的对象，把资源搬走"。但是const限制了变化。

---

# `std::move`

`std::move` 是一个**强制类型转换**，把一个表达式转换成右值表达式（更准确说是 xvalue）。

> xvalue &rarr; eXpiring value (将亡值)，它有"身份/地址"(像左值)，但资源"可以被搬走"(像右值)。

大致可以理解为：

```cpp
template <typename T>
constexpr std::remove_reference_t<T>&& move(T&& t) noexcept {
    return static_cast<std::remove_reference_t<T>&&>(t);
}
```

```cpp
// 资源转移并不是move触发的，而是那个"="
// 相当于调用了std::string的移动构造或者是移动赋值
std::string ss = std::move(s);
```

> - `std::remove_reference_t<T>` 用于去掉引用，比如`int& -> int`, `int&& -> int`
> - `std::remove_reference_t<T>&&` 表示把T的引用去掉再加上右值引用，即`int& -> int&&`, `int -> int&&`。

## 移动语义

如果没有移动，那么只能依靠拷贝，但是拷贝可能很贵，例如：

- `std::string` 要分配堆内存
- `std::vector` 要复制大量元素
- 文件句柄、锁、socket 甚至不能被正常拷贝

右值的引入让编译器和库可以知道：这个对象是一个即将销毁的临时对象，你可以安全地搬走它的资源。

示例：

```cpp
class Buffer {
public:
    Buffer(size_t n) : size_(n), data_(new int[n]) {}

    ~Buffer() {
        delete[] data_;
    }

    // copy
    Buffer(const Buffer& other) : size_(other.size_), data_(new int[other.size_]) {
        std::copy(other.data_, other.data_ + size_, data_);
    }

    // move，不用复制内容，直接接管指针即可
    Buffer(Buffer&& other) noexcept : size_(other.size_), data_(other.data_) {
        other.size_ = 0;
        other.data_ = nullptr;
    }

private:
    size_t size_{};
    int* data_{};
};
```

## `std::move` 之后原对象还能用吗?

能用，但只能处于**有效但未指定状态**。

例如：

```cpp
std::string s = "hello";
std::string t = std::move(s);
```

此后：

- `s` 仍然是合法对象
- 你可以给它重新赋值
- 你可以销毁它
- 但不能假设它还保留原内容 `"hello"`

---

# `std::forward`

> [!IMPORTANT]
> ⚠️ `forward`一定是和`template`/`auto&&`配合类型推导，对于确定的类型`int&&`是无法传入左值引用的。

在模板里，把参数"按它原本的值类别"转发出去，传进来左值就传出左值，传进来右值就传出右值。

```cpp
void process(int& x) {
    std::cout << "lvalue\n";
}

void process(int&& x) {
    std::cout << "rvalue\n";
}

template<typename T>
void wrapper(T arg) {
    process(arg);
}

int x = 10;
wrapper(x); // lvalue
wrapper(10); // lvalue
```

两个函数都是调用的lvalue版本。因为在 wrapper 里面，arg 是一个有名字的变量。任何有名字的变量，表达式上都是左值。

如果使用`std::move`，则不论传递的左值还是右值，都会调用rvalue版本。

```cpp
template<typename T>
void wrapper(T arg) {
    process(std::move(arg));
}
```

## 完美转发 (Perfect Forwarding)

在一个**模板函数**中，接收任意类型和任意值类别的参数，再把它原封不动地传给另一个函数。

> 原封不动一般包括：
>
> - 类型不变
> - const 不变
> - 左值右值不变

```cpp
template<typename T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg));
}
```

## 模版中的 `T&&`

当 `T&&` 出现在**模板类型推导**里时，它可能不是普通右值引用，而是**转发引用 (forwarding reference)**。

```cpp
template <typename T>
void func(T&& x) {
}
```

此处发生引用折叠，即实现了传入左值则传出左值，传入右值则传出右值。

## 引用折叠 (Reference Collapsing)

**只要有左值引用参与，结果就是左值引用**。

- `T&&` &rarr; `T&&`
- `T& &` &rarr; `T&`
- `T& &&` &rarr; `T&`
- `T&& &` &rarr; `T&`
- `T&& &&` &rarr; `T&&`

---

# 常见问题

#### ❌ 右值就是不能取地址

某些右值在语言规则下也可能有地址或能被编译器物化处理。更稳妥的说法是：

- 左值强调身份
- 右值强调临时性和可移动性

#### ❌ `std::move` 会自动移动

`std::move` 只是转换。真正是否移动，要看后续是否调用了移动构造/移动赋值。

#### ❌ `T&&` 一定是右值引用

在模板推导场景下，它可能是转发引用。

#### ❌ 有名字的 `T&&` 变量是右值

有名字就是左值表达式。

#### ❌ 被 `std::move` 之后对象就不能用了

它仍然有效，只是值未指定。

#### ✅ 移动构造通常要标 `noexcept`？

因为标准容器（尤其是 `std::vector`）在扩容搬迁元素时，更倾向于使用 `noexcept` 的移动构造。如果移动可能抛异常，容器为了保证强异常安全，可能改用拷贝。

#### 🔍 函数返回值是左值还是右值？

- 返回 `T`：得到的是一个临时值，通常视作右值
- 返回 `T&`：得到左值
- 返回 `T&&`：得到将亡值/xvalue

---

# auto&&

与模版函数中的`T&&`类似，也会做引用折叠，既能绑定左值也能绑定右值。

```cpp
int x = 10;
auto&& a = x;   // a 是 int&
auto&& b = 20;  // b 是 int&&
```

比如循环的时候使用`auto&&`而非`auto`可以保证引用和cv属性，同时避免元素拷贝。

```cpp
for (auto&& x : vec_x) {...}
```

但是根据引用折叠，**`auto&&`代表的一定是引用类型，一定不会指向基本类型**。

---

# decltype(auto)

和普通 auto 最大区别是：

- auto 会丢掉一些引用和 cv 信息
- decltype(auto) 可以保留表达式的精确类型

```cpp
int x = 1;
int& rx = x;

decltype(auto) a = x; // int
decltype(auto) b = rx; // int&
```

```cpp
// 如果 f(...) 返回引用，那么 decltype(auto) 也返回引用
// 如果返回值类型，那么它也返回值
template<typename F, typename... Args>
decltype(auto) call(F&& f, Args&&... args) {
    return std::forward<F>(f)(std::forward<Args>(args)...);
}
```

---

# Copy Elision 和 RVO/NRVO

这是优化"返回对象时别乱拷贝/移动"的机制。

编译器可以直接在最终位置构造对象，跳过中间临时对象的拷贝/移动。

```
// 代码中
1. 先构造临时对象
2. 再拷贝/移动到目标对象

// 编译器
直接在目标对象所在内存上构造
```

- RVO (Return Value Optimization) &rarr; 当函数返回一个临时对象时，直接构造到接收者位置。
- NRVO (Named Return Value Optimization) &rarr; 当返回一个有名字的局部变量时做优化。

```cpp
std::string rvo_func() {
    return "Hello World";
}

std::string nrvo_func() {
    std::string s = "Hello World";
    return s;
}

// 相当于不会先通过函数创建一个临时变量，再移动/拷贝到s1和s2
// 而是相当于直接对s1和s2调用了一次构造函数
//
// old: std::string(const char *) + std::string(std::string&&)
// new: std::string(const char *)
std::string s1 = rvo_func();
std::string s2 = nrvo_func();
```

> [!NOTE]
> 通过下面例子可以看出，即使删掉了移动和拷贝构造函数，程序仍可编译。
>
> ```cpp
> struct A {
>     A() = default;
>     A(const A&) = delete;
>     A(A&&) = delete;
> };
>
> A make() {
>     return A{};
> }
> ```

---

# 生命周期延长规则

生命周期延长机制主要针对“本来很快就销毁的**临时对象/右值**临时量”。左值通常本来就有独立生命周期（由其定义位置决定），一般不需要“延长”。

- `const T&`

```cpp
const std::string& s = std::string("abc");
```

⚠️ 函数返回引用的时候不能返回临时变量，即临时变量的生命周期不能通过如下方式延长

```cpp
const std::string& func() {
    return std::string("abc");
}

const std::string& ref = func(); // ref 引用的是已经被销毁的内容！
```

- `T&&`

```cpp
std::string&& ref = std:;string("abc");
```

