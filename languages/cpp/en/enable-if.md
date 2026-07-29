# `enable_if`

`enable_if` is used to determine whether a template should participate in compilation based on conditions.
```cpp
template<bool expr, class T = void>
struct enable_if {};

template<class T>
struct enable_if<true, T> {
    using type = T;
};
```
The implementation is similar to the above. When `expr` is valid, use `T` as the type. Otherwise, triggering [SFINAE](./sfinae.md) will remove the template from the substitution.

```cpp
#include <type_traits>
#include <iostream>

template<typename T>
std::enable_if<std::is_integral<T>::value>::type
print(T value)
{
    std::cout << "int";
}

template<typename T>
std::enable_if<std::is_floating_point<T>::value>::type
print(T value)
{
    std::cout << "float";
}
```

> [!NOTE]
> enable_if itself contains "If it is incorrect, delete it, if it is correct, return a type", so **you don't need to write the return type when defining**
> ```cpp
> template<typename T>
> std::enable_if<std::is_integer<T>::value, int>::type
> function(T t) {...}
> ```
> When T is an int, the function automatically generates the signature `int function(int t)`.

# `enable_if_t`

It is the abbreviation of `enable_if<>::type`.
```cpp
#include <type_traits>
#include <iostream>

template<typename T>
std::enable_if_t<std::is_integral<T>::value>
print(T value)
{
    std::cout << "int";
}
```

# Works with `decltype`

```cpp
// 类型T必须是int，否则该模版被剔除出substitution
template<typename T>
auto func(T t) -> std::enable_if_t<std::is_integral<T>::value>
{
}
```

# as template parameter

In the previous example we showed that SFINAE can be triggered by placing it in the return value. But there is a problem, that is, constructor/destructor/operator, etc. have no return value. In order to also ensure SFINAE, we need to put `enable_if` in the template parameters.

```cpp
template<typename T, typename = std::enable_if_t<std::is_integer_v<T>>>
```
We don't even need to give a name to the second type, because its function is to check whether T meets the conditions. If it meets the conditions, it can participate in overload selection, otherwise it will be ignored.
