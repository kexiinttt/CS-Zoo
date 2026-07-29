# What are lvalues and rvalues?

You can first remember it in one sentence:

- **lvalue**: an expression that has identity, can be located, and can usually take an address
- **rvalue**: temporary value, which is about to disappear, can be moved, and is usually a temporary expression that should not be referenced for a long time.
  - pvalue: pure rvalue, temporary value, ratio `1+1`, `x + y`
  - xvalue: expring value, has object identity but can be moved

## lvalue lvalue

lvalues usually have these characteristics:

- Have a name, or at least a clear, stable memory location
- Can appear to the left of the assignment number
- Usually you can get the address
- Lifecycle usually does not disappear immediately when the current expression ends

Example:

```cpp
// `x` 都是左值
int x = 10;
int y = x;
x = 20;
int* p = &x;

// `s[0]` 也是左值
std::string s = "hello";
s[0] = 'H';
```

## rvalue rvalue

Rvalues usually have these characteristics:

- Often temporary results
- Generally cannot be placed on the left side of the assignment number
- Life cycle is usually short
- Suitable for being "moved" rather than "copied"

Example:

```cpp
// `10` 是右值
int x = 10;
// `x + 1` 只是一个计算结果，没有独立名字，也是右值
int y = x + 1;
// `make_name()` 返回出来的那个临时 `std::string`，通常也看作右值
std::string make_name() {
    return "Alice";
}
```

---

# lvalue reference and rvalue reference

## lvalue reference `T&`

An lvalue reference can usually only be bound to an lvalue.

```cpp
int x = 10;
int& ref = x; // ✅

int& ref = 10; // ❌
```

## rvalue reference `T&&`

Rvalue references usually bind rvalues:

```cpp
int&& r = 10;
```

Its meaning is not primarily to "give an rvalue a name", but to support:

- **move semantics**
- **perfect forwarding**

For example:

```cpp
std::string s1 = "hello";
std::string s2 = std::move(s1);
```

Here `std::move(s1)` converts `s1` into an rvalue expression, allowing `s2` to directly "steal" the resources held by `s1` instead of deep copying.

## const lvalue reference `const T&`

`const T&` is very special. It can bind both lvalues and rvalues.

```cpp
int x = 10;
const int& a = x;   // ✅
const int& b = 20;  // ✅
```

Why can rvalues ​​be bound?

Because the compiler will generate a temporary object for the rvalue and extend its life cycle until the end of the scope of this reference.

This is why `const T&` is often used for:

- Avoid copying
- Receive various types of actual parameters

For example:

```cpp
void print(const std::string& s) {
    std::cout << s << '\n';
}

std::string s = "abc";
print(s); // 左值
print("abc"); // 右值
```

> [!NOTE]
> **A named rvalue reference variable is itself an lvalue**.
>
> ```cpp
> int&& r = 10;
> ```
>
> Although the type of `r` is `int&&`, the expression `r` itself is an lvalue because it has a name and a stable identity.
>
> ```cpp
> void f(int&)  { std::cout << "lvalue\n"; }
> void f(int&&) { std::cout << "rvalue\n"; }
>
> int main() {
>     int&& r = 10;
>     f(r);            // 调用 lvalue 版本
>     f(std::move(r)); // 调用 rvalue 版本
> }
> ```

## const rvalue reference `const T&&` (basically not used)

The main value of rvalue reference is "you can modify this object that is about to be destroyed and move the resources away." But const limits changes.

---

# `std::move`

`std::move` is a **cast** that converts an expression into an rvalue expression (more precisely, an xvalue).

> xvalue → eXpiring value (dying value), it has an "identity/address" (like an lvalue), but the resource "can be moved" (like an rvalue).

It can be roughly understood as:

```cpp
template <typename T>
constexpr std::remove_reference_t<T>&& move(T&& t) noexcept {
    return static_cast<std::remove_reference_t<T>&&>(t);
}
```


```cpp
// 资源转移并不是move触发的，而是那个"="
// 相当于调用了std::string的移动构造或者是移动赋值
std::string ss = std::move(s);
```

> - `std::remove_reference_t<T>` is used to remove references, such as `int& -> int`, `int&& -> int`
> - `std::remove_reference_t<T>&&` means removing the reference to T and adding an rvalue reference, that is, `int& -> int&&`, `int -> int&&`.

## Move semantics

If there is no move, then you can only rely on copying, but copying can be expensive, for example:

- `std::string` to allocate heap memory
- `std::vector` To copy a large number of elements
- File handles, locks, and sockets cannot even be copied normally

The introduction of rvalues allows compilers and libraries to know that this object is a temporary object that is about to be destroyed, and you can safely move its resources.

Example:

```cpp
class Buffer {
public:
    Buffer(size_t n) : size_(n), data_(new int[n]) {}

    ~Buffer() {
        delete[] data_;
    }

    // copy
    Buffer(const Buffer& other) : size_(other.size_), data_(new int[other.size_]) {
        std::copy(other.data_, other.data_ + size_, data_);
    }

    // move，不用复制内容，直接接管指针即可
    Buffer(Buffer&& other) noexcept : size_(other.size_), data_(other.data_) {
        other.size_ = 0;
        other.data_ = nullptr;
    }

private:
    size_t size_{};
    int* data_{};
};
```

## Can the original object still be used after `std::move`?

Works, but only in valid but unspecified state.

For example:

```cpp
std::string s = "hello";
std::string t = std::move(s);
```

Thereafter:

- `s` is still a legal object
- You can reassign it
- you can destroy it
- But it cannot be assumed that it retains the original content `"hello"`

---

# `std::forward`

> [!IMPORTANT]
> ⚠️ `forward` must cooperate with `template`/`auto&&` in type deduction. For the determined type `int&&`, lvalue references cannot be passed in.

In the template, the parameters are forwarded "according to their original value categories". If the lvalue is passed in, the lvalue will be passed out, and if the rvalue is passed in, the rvalue will be passed out.

```cpp
void process(int& x) {
    std::cout << "lvalue\n";
}

void process(int&& x) {
    std::cout << "rvalue\n";
}

template<typename T>
void wrapper(T arg) {
    process(arg);
}

int x = 10;
wrapper(x); // lvalue
wrapper(10); // lvalue
```

Both functions are called lvalue versions. Because in the wrapper, arg is a named variable. Any named variable is an lvalue in expressions.

If `std::move` is used, the rvalue version is called regardless of the lvalue or rvalue passed.

```cpp
template<typename T>
void wrapper(T arg) {
    process(std::move(arg));
}
```

## Perfect Forwarding

In a **template function**, receive parameters of any type and any value category, and then pass them unchanged to another function.

> Intact generally includes:
>
> - type unchanged
> - const unchanged
> - lvalue and rvalue unchanged

```cpp
template<typename T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg));
}
```

## `T&&` in template

When `T&&` appears in template type deduction, it may not be an ordinary rvalue reference, but a forwarding reference.

```cpp
template <typename T>
void func(T&& x) {
}
```

Reference folding occurs here, that is, if the lvalue is passed in, the lvalue will be passed out, and if the rvalue is passed in, the rvalue will be passed out.

## Reference Collapsing

**As long as an lvalue reference is involved, the result is an lvalue reference**.

- `T&&` → `T&&`
- `T& &` → `T&`
- `T& &&` → `T&`
- `T&& &` → `T&`
- `T&& &&` → `T&&`

---

# FAQ

#### ❌ rvalue means that the address cannot be taken

Certain rvalues may also have addresses under language rules or be materialized by the compiler. A more secure statement is:

- lvalue emphasizes identity
- rvalues emphasize temporaryness and portability

#### ❌ `std::move` will move automatically

`std::move` is just a conversion. Whether it is actually moved depends on whether the move constructor/move assignment is subsequently called.

#### ❌ `T&&` must be an rvalue reference

In a template deduction scenario, it might be a forward reference.

#### ❌ The named `T&&` variable is an rvalue

Having a name is an lvalue expression.

#### ❌ The object cannot be used after being `std::move`

It still works, just the value is unspecified.

#### ✅ Mobile structures are usually marked `noexcept`?

Because standard containers (especially `std::vector`) tend to use the move structure of `noexcept` when expanding and relocating elements. If the move may throw an exception, the container may use copying instead to ensure strong exception safety.

#### 🔍 Is the function return value an lvalue or an rvalue?

- Return `T`: the result is a temporary value, usually regarded as an rvalue
- Return `T&`: get lvalue
- Return `T&&`: get dying value/xvalue

---

# auto&&

Similar to `T&&` in the template function, it also does reference folding and can bind both lvalues and rvalues.

```cpp
int x = 10;
auto&& a = x;   // a 是 int&
auto&& b = 20;  // b 是 int&&
```

For example, using `auto&&` instead of `auto` during looping can ensure the reference and cv attributes while avoiding element copying.

```cpp
for (auto&& x : vec_x) {...}
```

However, according to reference folding, **`auto&&` must represent a reference type and must not point to a basic type**.

---

# decltype(auto)

The biggest difference from ordinary auto is:

- auto will lose some references and cv information
- decltype(auto) can preserve the exact type of the expression

```cpp
int x = 1;
int& rx = x;

decltype(auto) a = x; // int
decltype(auto) b = rx; // int&
```


```cpp
// 如果 f(...) 返回引用，那么 decltype(auto) 也返回引用
// 如果返回值类型，那么它也返回值
template<typename F, typename... Args>
decltype(auto) call(F&& f, Args&&... args) {
    return std::forward<F>(f)(std::forward<Args>(args)...);
}
```

---

# Copy Elision and RVO/NRVO

This is a mechanism to optimize "don't copy/move randomly when returning objects".

The compiler can construct the object directly at the final location, skipping the copy/move of intermediate temporary objects.

```
// 代码中
1. 先构造临时对象
2. 再拷贝/移动到目标对象

// 编译器
直接在目标对象所在内存上构造
```

- RVO (Return Value Optimization) → When a function returns a temporary object, it is constructed directly to the receiver location.
- NRVO (Named Return Value Optimization) → Optimize when returning a named local variable.

```cpp
std::string rvo_func() {
    return "Hello World";
}

std::string nrvo_func() {
    std::string s = "Hello World";
    return s;
}

// 相当于不会先通过函数创建一个临时变量，再移动/拷贝到s1和s2
// 而是相当于直接对s1和s2调用了一次构造函数
//
// old: std::string(const char *) + std::string(std::string&&)
// new: std::string(const char *)
std::string s1 = rvo_func();
std::string s2 = nrvo_func();
```

> [!NOTE]
> As can be seen from the following example, the program can still be compiled even if the move and copy constructors are deleted.
>
> ```cpp
> struct A {
>     A() = default;
>     A(const A&) = delete;
>     A(A&&) = delete;
> };
>
> A make() {
>     return A{};
> }
> ```

---

# Life cycle extension rules

The life cycle extension mechanism mainly targets "temporary objects/rvalue temporary quantities that are destroyed quickly". An lvalue usually has an independent life cycle (determined by its definition location) and generally does not need to be "extended".

- `const T&`

```cpp
const std::string& s = std::string("abc");
```

⚠️ When the function returns a reference, it cannot return a temporary variable, that is, the life cycle of the temporary variable cannot be extended in the following ways

```cpp
const std::string& func() {
    return std::string("abc");
}

const std::string& ref = func(); // ref 引用的是已经被销毁的内容！
```

- `T&&`

```cpp
std::string&& ref = std:;string("abc");
```


