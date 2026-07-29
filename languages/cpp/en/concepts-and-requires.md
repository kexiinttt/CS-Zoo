# `concept` & `requires`
They solve the problem of "traditional template error messages are difficult to read + SFINAE writing is complicated".
* concept is a declarative constraint on the template parameter capabilities
* requires is to add "capability requirements" to the template. If they are not met, instantiation is not allowed.

---

# Usage

## Introduction

Before using concept, we need to do the following when defining a template function of a type that accepts some special constraints.
```cpp
template<typename T>
std::enable_if_t<std::is_integral_v<T>>
void func(T t) {}
```

We can define a concept and then use this concept to replace enable_if_t. Then use the following format to achieve the very complex expression before.
```cpp
template<typename T>
concept Integral = std::is_integral_v<T>;

template<Integral T>
void func(T t) {}

template<typename T>
requires Integral<T>
void func(T t) {}
```

## Specific location used

Note that multiple conditions can be combined.

##### Directly constrain template parameters
```cpp
template <std::integral T>
void func(T t) {}
```

##### Cooperate with requirements
```cpp
template<typename T>
requires std::integral<T> && std::signed_integral<T>
void func(T t) {}
```

##### Trailing requires
```cpp
template<typename T>
void func(T t)
    requires std::integral<T> && std::signed_integral<T>
{
}
```

---

# Custom concept

##### Single parameter

```cpp
template<typename T>
concept TypeWithSize = requires(T t) {
    t.size();
}
```

##### Single parameter and multiple variables
```cpp
template<typename T>
concept Addable = requires(T a, T b)
{
    { a + b } -> std::convertible_to<T>;
};
```

##### Multiple parameters
```cpp
template<typename T, typename U>
concept Addable = requires(T t, U u) {
    t + u;
}
```

##### Multi-line requires expression
```cpp
// T 必须要支持下列三种方法
template<typename T>
concept TypeWithConstrains = requires(T t) {
    t.size();
    t.begin();
    t.end();
}
```

---

# `concept auto`

concept can limit auto.

```cpp
std::integral auto x = ...; // x 必须是int

void function(std::integral auto var) {}
```

---

# concept compile time check

Concept occurs during the template substitution phase. It will only prompt "constraint not satisfied" instead of reporting an error.

It replaces a lot of:
```
SFINAE
enable_if
void_t
trait hacks
```
Make template code: safer, more readable, and easier to maintain.

---

# `requires requires`

In fact, it is an abbreviation, which is not recommended and is less readable. The first `requires` is the requires clause, and the second `requires` is the requires expression. So it is equivalent to `requires requires(expression) {}`.

```cpp
template<typename T>
requires requires(T t) { t.size(); }
void function(T t) {} // 要求T必须支持size这个函数
```

It is more recommended to write the following way
```cpp
template<typename T>
concept TypeWithSize = requires(T t) { t.size(); };

template<TypeWithSize T>
void function(T t) {}
```
