# decltype 介绍

`decltype` 是 C++11 引入的一个关键字，用来 **在编译期获取表达式的类型**。他不会计算表达式，只是 **推导表达式的类型**。

* 如果 expr 是一个 id-expression，那么 decltype 不会推导表达式的值类别
  * 如果 expr 中只包含一个变量名，此时 decltype 会返回这个变量的类型
  * 如果 expr 只包含一个访问类数据成员的表达式，此时 decltype 会返回这个数据成员的类型
* 否则，decltype 会同时推导 expr 的类型（假设为 T）和值类别
  * 如果 expr 是一个 lvalue，那么 decltype 将返回 T&
  * 如果 expr 是一个 xvalue，那么 decltype 将返回 T&&
  * 如果 expr 是一个 prvalue，那么 decltype 将返回 T

> [!NOTE]
> ```cpp
> decltype(name)      // 拿声明类型
> decltype(lvalue)    // T&
> decltype(xvalue)    // T&&
> decltype(prvalue)   // T
> ```

---

# 基本用法

## 获取变量 / 表达式类型
```cpp
int a = 10;
decltype(a) b = 20; // 🟰 int b = 20;
decltype(a + b) c; // 🟰 int c;
```

## 获取函数返回值类型
一般用于 template 中。因为传入参数类型未知，所以返回类型也未知，这个时候使用 **Trailing Return Type**。
```cpp
template<typename T, typename U>
auto function(T t, U u) -> decltype(t + u) {
    return t + u;
}

// ❌ 不能写在前面，因为变量还未定义
// decltype(t + u) fucntion(T t, U u) {...};
```

## 保留引用类型
`decltype` 会保留表达式的引用属性。
```cpp
int a = 10;
int& ref = a;
decltype(ref) b = a; // b 的类型是int&
```

---

# auto vs decltype
| | auto | decltype |
| :---: | :---: | :---: |
| 是否保留引用 | 否 | 是 |
| 是否保留 const | 否    | 是 |
| 是否需要初始化 | 需要 | 不需要 |

所以 `auto` 看值， `decltype` 看表达式。

```cpp
int a = 10;
int& ref = a;
const int b = 10;

auto xa = a;          // int
decltype(ref) ya = a; // int&

auto xb = b;        // int
decltype(b) yb = b; // const int
```

---

# `decltype((var))` != `decltype(var)`

```cpp
int a = 10;
decltype(a)               // int
decltype((a))             // int&
decltype(a + 10)          // int
decltype((a + 10))        // int
decltype(std::move(a))    // int&&
decltype((std::move(a)))  // int&&
```

decltype 对于id-expression和表达式是有不同的判断标准的，请参考[介绍](#decltype-介绍)中的总结。

---

# `-> decltype(expr, void())`

C++中逗号表达式作用就是：保证每一个都valid，然后用最后一个的值。

```cpp
int a = (1, 2);     // a = 2
int b = (1 / 0, 2); // 报错，因为 1/0 是invalid操作
```

所以看到 `-> decltype(expr, void())` 的意思是：保证 expr 不报错，且最终推断的类型是void。
```cpp
// 如果类型T有size()这个方法，那么返回类型是void
template<typename T>
auto print(T t) -> decltype(t.size(), void()) {
    std::cout << "has size()\n";
}
```

---

# `std::declval`

与`decltype`配合使用。

比如`decltype`中需要使用`decltype(T().size())`，但是T可能没有构造函数，那么`T()`就是错误的。而且我们只需要知道`T`是否含有`size()`即可，我们并不需要真正创建并运行它。

`decltype(std::declval<T>().size())` 可以假装我们有一个T，并不会真的创建任何东西。

> [!NOTE]
> 注意 `std::declval` 并没有具体实现，所以不能放在运行代码中，只能放在 `decltype`, `sizeof` 等表达式中。