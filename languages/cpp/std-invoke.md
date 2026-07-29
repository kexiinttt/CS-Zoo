# 什么是 `std::invoke`

`std::invoke` 是 C++17 标准引入的一个通用调用工具，用来**统一调用各种“可调用对象”**。无论调用目标是什么形式，`std::invoke` 都可以用统一的语法帮你完成调用。

这里的“可调用对象”包括：

* 普通函数
* 函数指针
* lambda
* 仿函数（实现了 `operator()` 的对象）
* 成员函数指针
* 成员数据指针

---

# 为什么需要 `std::invoke`

这段代码看起来可以调用很多东西，但其实**并不完整**。

```cpp
template <typename F, typename... Args>
auto call(F&& f, Args&&... args) {
    return f(std::forward<Args>(args)...);
}
```
如果 `f` 是普通函数、lambda、仿函数，通常没问题；但如果 `f` 是**成员指针**，就不能直接调用/解地址。

```cpp
struct Person {
    void hello() const {}
    int age = 18;
};

auto pmf = &Person::hello;
auto pmd = &Person::age;

Person p;

// ❌ 不能直接这样调用成员指针
// pmf();
// pmd;

std::invoke(pmf, p);   // ✅ 等价于 p.hello()
std::invoke(pmd, p);   // ✅ 等价于 p.age
```

> [!IMPORTANT]
> ⚠️ 对于成员指针，`std::invoke(f, obj, args...)`的第二个参数是instance，因为成员函数/属性其实是默认有一个`this`指针(类似Python里的`self`)，所以必须传入！

> [!NOTE]
> **成员指针** 和 **普通指针** 是不同的：
> * 普通指针指向某个对象/值的地址；成员指针指向类里的某个变量/函数
> * 普通指针可以直接解地址，但是成员指针不能，因为其不是一个地址，必须配合某个对象一起使用
> ```cpp
> // 成员指针可以定义在instance前面，因为其并不指向实际的地址
> std::string User::* mp = &User::name;
> 
> User u{"Alice", 24};
>
> // 普通指针必须定义在instance后面，因为其指向具体的地址
> std::string* p = &u.name; 
>
> std::cout << *p << u.*mp; 
> ```

---

# 泛型编程中的典型写法

```cpp
#include <functional>
#include <utility>

template <typename F, typename... Args>
decltype(auto) call(F&& f, Args&&... args) {
    return std::invoke(std::forward<F>(f), std::forward<Args>(args)...);
}
```

* `F&&` 和 `Args&&...` 是万能引用
* `std::forward` 用于完美转发
* `decltype(auto)` 保留返回值类别
* `std::invoke` 保证各种 callable 都能统一处理

> ⚠️ 此处使用`decltype(auto)`而非`auto`是因为其会保留ref和cv。


---

# invocable traits

## `std::invoke_result_t`

用来推导调用结果类型。

```cpp
template <typename F, typename... Args>
using invoke_result_t = std::invoke_result_t<F, Args...>;
```

## `std::is_invocable_v`

用来判断是否可以这样调用。
```cpp
#include <functional>
#include <type_traits>

struct X {
    int foo(double) { return 1; }
};

static_assert(std::is_invocable_v<decltype(&X::foo), X&, double>);
static_assert(std::is_invocable_v<decltype(&X::foo), X*, double>);
static_assert(!std::is_invocable_v<decltype(&X::foo), int, double>);
```
