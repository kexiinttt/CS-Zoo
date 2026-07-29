# `inline`

The earliest meaning is: **It is recommended that the compiler expand the function call into the function body itself**, thereby reducing the function call overhead.

> Copy-paste the code into the code and run it directly instead of letting the program jump to execute the function

But note: In modern C++, the more important role of **`inline` is not to "force inline optimization", but to solve the problem of multiple definitions.**

---

# inline

`inline` is just a suggestion to the compiler, and it is up to the compiler to decide whether it is actually expanded.

The compiler may refuse inlining:

```cpp
inline void f() {
    // 函数体太大，编译器可能不内联
}
```

The compiler may also automatically inline functions that are not written `inline`:

```cpp
int square(int x) {
    return x * x;
}
```

---

# Multiple definitions

If a normal function is defined in a header file and included in multiple `.cpp` files, an error `multiple definition` will be reported during linking.

```cpp
// utils.h
int add(int a, int b) {
    return a + b;
}
```


```cpp
// a.cpp
#include "utils.h"
```


```cpp
// b.cpp
#include "utils.h"
```

Use `inline` to solve header file function definition problems. In this way, even if multiple `.cpp` include this header file, the ODR will not be violated.

```cpp
// utils.h
inline int add(int a, int b) {
    return a + b;
}
```

---

# Class member functions

## The default definition within the class is inline

When a function implementation is defined within a class, it is inline by default.
```cpp
class Person {
public:
    int getAge() const {
        return age;
    }

private:
    int age = 0;
};
```

## Out-of-class definitions need to be explicit

If the member function definition is written outside the class, explicit `inline` is required to achieve the inline effect.

```cpp
// Person.h
class Person {
public:
    int getAge() const;

private:
    int age = 0;
};

inline int Person::getAge() const {
    return age;
}
```

---

# `inline` variable

Generally speaking, static member variables should be defined outside the class, but `inline static` can be defined inside the class.

```cpp
// A.h
class A {
public:
    inline static int count = 0;
};
```

> The essence of the `inline` variable is that the linker will merge multiple inline definitions into the same variable.

---

# `inline` vs `static`

* inline → Multiple TUs can see the same x, and ultimately the same variable is shared throughout the entire program
* static → Each TU has its own independent copy of x


```cpp
// config.h
static int x = 1;
inline int y = 2;

// a.cpp
#include "config.h"

// b.cpp
#include "config.h"
```

Then in `a.cpp` and `b.cpp`
- `x` are two different variables
- `y` is the same variable

---

# Why is the `inline` function generally defined in the header file?

One use of inline is to allow multiple TUs to see the same function definition. Only when the function definition is placed in the header file and imported by multiple .cpp, can there be a situation of "spanning multiple TUs".

---

# `inline` Advantages and Disadvantages

| Advantages | Disadvantages |
| --- | --- |
| Allows functions or variables to be defined in header files without easily triggering `multiple definition` | Writing `inline` does not mean that the compiler will actually inline expansion |
| For very short functions, the compiler has the opportunity to expand them directly, reducing the function call overhead | If the function body is large, forcibly placing it in the header file may increase the compilation time |
| It is important for header-only libraries to facilitate writing the implementation directly in the header file | When multiple `.cpp` contain the same header file, each translation unit must see this definition, which may increase the burden of generating code and compilation |
| | Putting too many implementation details into header files will increase coupling and make changes more likely to trigger widespread recompilation |

---

# Why can’t we always use `inline`?

Use inline only if the function is very short, called extremely frequently, and if you are sure that inlining will actually bring performance improvements.

## Code Bloat

If an inline function has 10 lines of code and is called in 1,000 places, then the compiler will copy those 10 lines of code 1,000 times, resulting in 10,000 lines of executable code. This results in the final executable being too large.

## Increase compilation pressure and dependencies

The definition of an inline function must usually be placed in a header file (.h) so that the compiler can find the function body at each call site. This results in longer compilation times: once the inline function is modified, all .cpp files containing this header file must be recompiled.

## Function restrictions
Not all functions can be inlined:
- Recursive functions usually cannot be inlined
- The benefits of inlining a function that is too long are extremely low
