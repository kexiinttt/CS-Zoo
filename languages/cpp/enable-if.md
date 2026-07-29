# `enable_if`

`enable_if` 用来根据条件决定某个模板是否参与编译。
```cpp
template<bool expr, class T = void>
struct enable_if {};

template<class T>
struct enable_if<true, T> {
    using type = T;
};
```
其实现类似如上，当`expr` valid的时候就使用`T`作为类型，否则触发[SFINAE](./sfinae.md)将把该模版剔除出substitution。

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
> enable_if 本身就包含 "不正确就剔除，正确就返回一个类型"，因此**定义的时候不需要写返回类型**
> ```cpp
> template<typename T>
> std::enable_if<std::is_integer<T>::value, int>::type
> function(T t) {...}
> ```
> 当T是int的时候，函数自动生成签名`int function(int t)`。

# `enable_if_t`

是 `enable_if<>::type` 的缩写。
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

# 和 `decltype` 配合

```cpp
// 类型T必须是int，否则该模版被剔除出substitution
template<typename T>
auto func(T t) -> std::enable_if_t<std::is_integral<T>::value>
{
}
```

# 作为模版参数

在前面例子里我们展示了可以放在返回值里触发SFINAE。但是有一个问题，那就是比如 constructor/destructor/operator 等没有返回值。为了同样保证SFINAE，我们需要把`enable_if`放在模版参数里。

```cpp
template<typename T, typename = std::enable_if_t<std::is_integer_v<T>>>
```
我们甚至不需要给第二个type赋予名字，因为他的作用就是检查T是否满足条件，满足则可以参与重载选择，否则就忽略。
