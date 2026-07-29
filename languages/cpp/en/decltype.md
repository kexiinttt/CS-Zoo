# decltype introduction

`decltype` is a keyword introduced in C++11, which is used to obtain the type of expression at compile time. It doesn't evaluate expressions, it just **derives the type of the expression**.

* If expr is an id-expression, decltype does not deduce the value category of the expression
  * If expr contains only one variable name, decltype will return the type of this variable.
  * If expr only contains an expression that accesses a class data member, decltype will return the type of this data member.
* Otherwise, decltype will deduce both the type of expr (assumed to be T) and the value category
  * If expr is an lvalue, then decltype will return T&
  * If expr is an xvalue, then decltype will return T&&
  * If expr is a prvalue, then decltype will return T

> [!NOTE]
> ```cpp
> decltype(name)      // 拿声明类型
> decltype(lvalue)    // T&
> decltype(xvalue)    // T&&
> decltype(prvalue)   // T
> ```

---

# Basic usage

## Get variable/expression type
```cpp
int a = 10;
decltype(a) b = 20; // 🟰 int b = 20;
decltype(a + b) c; // 🟰 int c;
```

## Get function return value type
Generally used in templates. Because the type of the incoming parameter is unknown, the return type is also unknown. In this case, **Trailing Return Type** is used.
```cpp
template<typename T, typename U>
auto function(T t, U u) -> decltype(t + u) {
    return t + u;
}

// ❌ 不能写在前面，因为变量还未定义
// decltype(t + u) fucntion(T t, U u) {...};
```

## Reserve reference types
`decltype` retains the reference attribute of the expression.
```cpp
int a = 10;
int& ref = a;
decltype(ref) b = a; // b 的类型是int&
```

---

# auto vs decltype
| | auto | decltype |
| :---: | :---: | :---: |
| Whether to keep references | No | Yes |
| Whether to retain const | No | Yes |
| Whether initialization is required | Yes | No |

So `auto` looks at the value, `decltype` looks at the expression.

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

`decltype` has different judgment criteria for id-expressions and expressions. See the [introduction](#introduction) for a summary.

---

# `-> decltype(expr, void())`

The function of comma expressions in C++ is to ensure that each one is valid, and then use the value of the last one.

```cpp
int a = (1, 2);     // a = 2
int b = (1 / 0, 2); // 报错，因为 1/0 是invalid操作
```

So seeing `-> decltype(expr, void())` means: ensuring expr does not report an error, and the final inferred type is void.
```cpp
// 如果类型T有size()这个方法，那么返回类型是void
template<typename T>
auto print(T t) -> decltype(t.size(), void()) {
    std::cout << "has size()\n";
}
```

---

# `std::declval`

For use with `decltype`.

For example, `decltype(T().size())` needs to be used in `decltype`, but T may not have a constructor, so `T()` is wrong. And we only need to know whether `T` contains `size()`, we don't need to actually create and run it.

`decltype(std::declval<T>().size())` can pretend we have a T without actually creating anything.

> [!NOTE]
> Note that `std::declval` has no specific implementation, so it cannot be placed in running code, but can only be placed in expressions such as `decltype`, `sizeof`.
