# What is `std::invoke`

`std::invoke` is a universal calling tool introduced by the C++17 standard, which is used to call various "callable objects" in a unified manner. No matter what form the call target is, `std::invoke` can help you complete the call using unified syntax.

The "callable objects" here include:

* Ordinary functions
* Function pointer
* lambda
* Functor (object that implements `operator()`)
* Member function pointer
* Member data pointer

---

# Why do we need `std::invoke`

This code looks like it can call a lot of things, but it's actually not complete.

```cpp
template <typename F, typename... Args>
auto call(F&& f, Args&&... args) {
    return f(std::forward<Args>(args)...);
}
```
If `f` is an ordinary function, lambda, or functor, there is usually no problem; but if `f` is a member pointer, it cannot be called/deaddressed directly.

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
> ⚠️ For member pointers, the second parameter of `std::invoke(f, obj, args...)` is instance, because member functions/properties actually have a `this` pointer by default (similar to `self` in Python), so it must be passed in!

> [!NOTE]
> **Member Pointer** and **Ordinary Pointer** are different:
> * Ordinary pointers point to the address of an object/value; member pointers point to a variable/function in the class
> * Ordinary pointers can directly decode addresses, but member pointers cannot, because they are not an address and must be used with an object.
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

# Typical writing methods in generic programming

```cpp
#include <functional>
#include <utility>

template <typename F, typename... Args>
decltype(auto) call(F&& f, Args&&... args) {
    return std::invoke(std::forward<F>(f), std::forward<Args>(args)...);
}
```

* `F&&` and `Args&&...` are universal references
* `std::forward` for perfect forwarding
* `decltype(auto)` retains the return value category
* `std::invoke` ensures that all callables can be processed uniformly

> ⚠️ `decltype(auto)` is used here instead of `auto` because it retains ref and cv.


---

# invocable traits

## `std::invoke_result_t`

Used to deduce the call result type.

```cpp
template <typename F, typename... Args>
using invoke_result_t = std::invoke_result_t<F, Args...>;
```

## `std::is_invocable_v`

Used to determine whether this call can be made.
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
