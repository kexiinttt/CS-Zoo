# Type Traits

`type traits` is a set of tools in the C++ standard library that check types, convert types, and make selections based on types at compile time:
* **Judge type properties** → For example, whether a type is an integer, whether it is a pointer, whether it is a reference, whether two types are the same.
* **Conversion type** → For example, remove `const`, remove references, make `decay`, and select types based on conditions.
* **Supports generic programming and compile-time branching** → Let templates automatically select different implementations based on different types.

---

# traits type

## Type judgment

This class usually returns a compile-time boolean value.

- `std::is_same_v<T, U>`
- `std::is_integral_v<T>`
- `std::is_pointer_v<T>`
- `std::is_reference_v<T>`
- `std::is_const_v<T>`
- `std::is_base_of_v<Base, Derived>`

> Before C++17, `std::is_integral<int>::value` was used. In the new version, the `_v` suffix represents `::value`.

## Type conversion

This class takes in one type and outputs another type.

- `std::remove_const_t<T>`
- `std::remove_reference_t<T>`
- `std::remove_cv_t<T>`
- `std::add_pointer_t<T>`
- `std::decay_t<T>`
- `std::conditional_t<B, T, U>`
- `std::common_type_t<T, U>`

> Before C++14, `std::remove_const<T>::type` was used, and the new version has the `_t` suffix instead of `::type`.


## Construction/assignment capabilities

- `std::is_default_constructible_v<T>`
- `std::is_copy_constructible_v<T>`
- `std::is_move_constructible_v<T>`
- `std::is_copy_assignable_v<T>`
- `std::is_move_assignable_v<T>`

## Template constraint/detection class
- `std::enable_if`
- `std::void_t`
- `std::invoke_result`

---

# `enable_if` / `concept` / `requires`

- [enable_if](./enable-if.md)
- [concept / requires](./concepts-and-requires.md)

---

# Special traits

## `std::bool_constant`

It is essentially:

```cpp
template<bool B>
using bool_constant = std::integral_constant<bool, B>;
```

It allows us to express "compile-time Boolean constants" more conveniently.

```cpp
using TrueType = std::bool_constant<true>;
using FalseType = std::bool_constant<false>;

template<typename T>
struct is_int_like : std::bool_constant<std::is_same_v<T, int>> {};
```

## `std::conjunction` / `std::disjunction` / `std::negation`

This is the "trait logical combinator" provided by C++17.

```cpp
// conjunction：逻辑与
using A = std::conjunction<std::is_integral<int>, std::is_signed<int>>;

// disjunction：逻辑或
using B = std::disjunction<std::is_integral<double>, std::is_floating_point<double>>;

// negation：逻辑非
using C = std::negation<std::is_pointer<int>>;
```

## `std::common_type`

Used to find the "common type" of multiple types.

```cpp
using T = std::common_type_t<int, double>;   // double
```

## `std::underlying_type`

Used to obtain the underlying type of the enumeration.

```cpp
enum class Color : unsigned char {
    Red, Green, Blue
};

using T = std::underlying_type_t<Color>;  // unsigned char
```

## `std::invoke_result`

Used to deduce "the return type after calling a callable object".

```cpp
#include <type_traits>

int foo(double) { return 42; }

using R = std::invoke_result_t<decltype(foo), double>;  // int
```

## `if constexpr` + type traits

In modern C++, a very core way to use type traits is with `if constexpr`.

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

`if constexpr` is the **compile-time branch**. Branches that do not satisfy the condition are discarded, so you can safely write code that is only valid for certain types.

---

# Custom traits

Type traits are not just for use, you can also write them yourself.

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

The meaning is: if `Cond == true`, the type is `T`; otherwise, the type is `F`.

Example:

```cpp
template<typename T>
using maybe_signed_t =
    std::conditional_t<(sizeof(T) < 4), int, long long>;
```

It is a compile-time ternary operator (`cond ? true : false`), but instead of selecting a value, it selects a type.

---

# `decay_t`

Convert type T to a "normal type common to pass-by-value". It will roughly do three things:
* Remove quotes
    * int& -> int
	* int&& -> int

* Remove the top cv
	* const int -> int
	* volatile int -> int

* Arrays/functions degenerate into pointers
	* int[10] -> int*
	* void(int) -> void(*)(int)

---

# `void_t`

```cpp
template<typename...>
using void_t = void;
```

No matter what parameters are passed in, they will eventually become void. It sounds useless, but the key point is: **The expression of the template parameter must be true/valid before it can be replaced with void**. SFINAE if the inner expression does not hold.

```cpp
// 检测成员是否含有foo函数
// ⚠️ has_foo是两个参数的模版，当写has_foo<T>的时候另一个参数被自动计算
template<typename, typename = void>
struct has_foo : std::false_type {};

template<typename T>
struct has_foo<T, std::void_t<decltype(std::declval<T>().foo())>> : std::true_type {};
```
