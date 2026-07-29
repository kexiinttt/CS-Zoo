# `concept` & `requires`
它们解决的是"传统template错误信息难读 + SFINAE写法复杂"的问题。
* concept 是对template参数能力的一种声明式约束
* requires 是给模板加“能力要求”，不满足就不允许实例化

---

# 用法

## 概论

使用concept之前，我们定义一个接受某种特殊约束的类型的模版函数时需要如下。
```cpp
template<typename T>
std::enable_if_t<std::is_integral_v<T>>
void func(T t) {}
```

我们可以定义一个concept，然后用该concept代替enable_if_t。然后使用下面的格式就可以实现之前非常复杂的表达。
```cpp
template<typename T>
concept Integral = std::is_integral_v<T>;

template<Integral T>
void func(T t) {}

template<typename T>
requires Integral<T>
void func(T t) {}
```

## 具体使用的位置

注意，可以使用多个条件配合起来。

##### 直接约束template参数
```cpp
template <std::integral T>
void func(T t) {}
```

##### 和requires配合
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

# 自定义 concept

##### 单一参数

```cpp
template<typename T>
concept TypeWithSize = requires(T t) {
    t.size();
}
```

##### 单一参数多个变量
```cpp
template<typename T>
concept Addable = requires(T a, T b)
{
    { a + b } -> std::convertible_to<T>;
};
```

##### 多个参数
```cpp
template<typename T, typename U>
concept Addable = requires(T t, U u) {
    t + u;
}
```

##### 多行requires expression
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

concept可以限制auto。

```cpp
std::integral auto x = ...; // x 必须是int

void function(std::integral auto var) {}
```

---

# concept 编译期检查

Concept 发生在 template substitution 阶段。他只会提示 "constraint not satisfied" 而非报错。

它取代了很多：
```
SFINAE
enable_if
void_t
trait hacks
```
让模板代码：更安全，更可读，更容易维护。

---

# `requires requires`

其实是一种简写，不太推荐，可读性较差。第一个`requires`是requires clause，第二个`requires`是requires expression。所以相当于是`requires requires(expression) {}`。

```cpp
template<typename T>
requires requires(T t) { t.size(); }
void function(T t) {} // 要求T必须支持size这个函数
```

更推荐下面写法
```cpp
template<typename T>
concept TypeWithSize = requires(T t) { t.size(); };

template<TypeWithSize T>
void function(T t) {}
```
