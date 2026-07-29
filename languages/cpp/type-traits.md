# Type Traits

`type traits` 是 C++ 标准库里一组**在编译期检查类型、转换类型、根据类型做选择**的工具：
* **判断类型性质** &rarr; 比如一个类型是不是整数、是不是指针、是不是引用、两个类型是不是相同。
* **转换类型** &rarr; 比如去掉 `const`、去掉引用、做 `decay`、按条件选类型。
* **支持泛型编程和编译期分支** &rarr; 让模板根据不同类型自动选择不同实现。

---

# traits 类型

## 类型判断

这一类通常返回一个编译期布尔值。

- `std::is_same_v<T, U>`
- `std::is_integral_v<T>`
- `std::is_pointer_v<T>`
- `std::is_reference_v<T>`
- `std::is_const_v<T>`
- `std::is_base_of_v<Base, Derived>`

> C++17前使用`std::is_integral<int>::value`，新版本中`_v`后缀代表了`::value`。

## 类型转换

这一类输入一个类型，输出另一个类型。

- `std::remove_const_t<T>`
- `std::remove_reference_t<T>`
- `std::remove_cv_t<T>`
- `std::add_pointer_t<T>`
- `std::decay_t<T>`
- `std::conditional_t<B, T, U>`
- `std::common_type_t<T, U>`

> C++14前使用`std::remove_const<T>::type`，新版本`_t`后缀代替`::type`。


## 构造/赋值能力

- `std::is_default_constructible_v<T>`
- `std::is_copy_constructible_v<T>`
- `std::is_move_constructible_v<T>`
- `std::is_copy_assignable_v<T>`
- `std::is_move_assignable_v<T>`

## 模板约束/探测类
- `std::enable_if`
- `std::void_t`
- `std::invoke_result`

---

# `enable_if` / `concept` / `requires`

- [enable_if](./enable-if.md)
- [concept / requires](./concepts-and-requires.md)

---

# 特殊 traits

## `std::bool_constant`

它本质上是：

```cpp
template<bool B>
using bool_constant = std::integral_constant<bool, B>;
```

它让我们更方便地表达“编译期布尔常量”。

```cpp
using TrueType = std::bool_constant<true>;
using FalseType = std::bool_constant<false>;

template<typename T>
struct is_int_like : std::bool_constant<std::is_same_v<T, int>> {};
```

## `std::conjunction` / `std::disjunction` / `std::negation`

这是 C++17 提供的“trait 逻辑组合器”。

```cpp
// conjunction：逻辑与
using A = std::conjunction<std::is_integral<int>, std::is_signed<int>>;

// disjunction：逻辑或
using B = std::disjunction<std::is_integral<double>, std::is_floating_point<double>>;

// negation：逻辑非
using C = std::negation<std::is_pointer<int>>;
```

## `std::common_type`

用来求多个类型的“公共类型”。

```cpp
using T = std::common_type_t<int, double>;   // double
```

## `std::underlying_type`

用于获取枚举底层类型。

```cpp
enum class Color : unsigned char {
    Red, Green, Blue
};

using T = std::underlying_type_t<Color>;  // unsigned char
```

## `std::invoke_result`

用来推导“调用一个可调用对象后的返回类型”。

```cpp
#include <type_traits>

int foo(double) { return 42; }

using R = std::invoke_result_t<decltype(foo), double>;  // int
```

## `if constexpr` + type traits

type traits 在现代 C++ 里，一个非常核心的使用方式就是配合 `if constexpr`。

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
void print_info(const T& value) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "integral: " << value << '\n';
    } else if constexpr (std::is_floating_point_v<T>) {
        std::cout << "floating point: " << value << '\n';
    } else {
        std::cout << "other type\n";
    }
}
```

`if constexpr` 是**编译期分支**。不满足条件的分支会被丢弃，因此你可以安全地写出只对某些类型合法的代码。

---

# 自定义 trait

type traits 不只是拿来用，也可以自己写。

```cpp
// 判断一个类型是不是 `int`
#include <type_traits>

template<typename T>
struct is_my_int : std::false_type {};

template<>
struct is_my_int<int> : std::true_type {};

template<typename T>
inline constexpr bool is_my_int_v = is_my_int<T>::value;
```

---

# `std::conditional_t`

```cpp
std::conditional_t<Cond, T, F>
```

含义是：如果 `Cond == true`，类型是 `T`；否则类型是 `F`。

示例：

```cpp
template<typename T>
using maybe_signed_t =
    std::conditional_t<(sizeof(T) < 4), int, long long>;
```

它是**编译期**的三元运算符(`cond ? true : false`)，但是不是选值，而是**选类型**。

---

# `decay_t`

把类型 T 转成一种“按值传递时常见的普通类型”。它大致会做三件事：
* 去掉引用
    * int& -> int
	* int&& -> int

* 去掉顶层 cv
	* const int -> int
	* volatile int -> int

* 数组 / 函数退化为指针
	* int[10] -> int*
	* void(int) -> void(*)(int)

---

# `void_t`

```cpp
template<typename...>
using void_t = void;
```

无论传入什么参数最后都变成void。听上去很没有用，但其实关键在于：**模版参数的表达式必须为true/valid才能替换成void**。如果内部表达式不成立则SFINAE。

```cpp
// 检测成员是否含有foo函数
// ⚠️ has_foo是两个参数的模版，当写has_foo<T>的时候另一个参数被自动计算
template<typename, typename = void>
struct has_foo : std::false_type {};

template<typename T>
struct has_foo<T, std::void_t<decltype(std::declval<T>().foo())>> : std::true_type {};
```
