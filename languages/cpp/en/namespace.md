# What is Namespace

`namespace` is a mechanism used by C++ to avoid name collision. It's actually just a symbol prefix. For example:

```cpp
math_utils::add // 编译后符号类似 -> `_ZN10math_utils3addEii`
```

Different headers can extend the same namespace (**namespace extension**). For example:

```cpp
// file1.cpp
namespace util {
    void foo() {}
}
```
```cpp
// file2.cpp
namespace util {
    void bar() {}
}
```


```cpp
#include "file1.h"
#include "file2.h"

int main() {
    util::foo();
    util::bar();
}
```

---

# How to use

⚠️ It is not recommended to use `using namespace std;` in header because:
* namespace pollution
* Name conflict
* Large projects are prone to bugs

---

# Share across multiple files

Declared in header and implemented in cpp.

> [!NOTE]
> If you implement the function in the header, it will easily trigger ODR (One Definition Rule).
>
> ```cpp
> // math.h
> namespace util {
> 
> int add(int a, int b) {   // ❌
>     return a + b;
> }
> }
> ```
> 
> ```cpp
> // a.cpp
> #include "utils.h"
> ```
> 
> ```cpp
> // b.cpp
> #include "utils.h"
> ```
>
> Compilation will report error `multiple definition of util::add`.
>
> Because `#include` is essentially **text copy**. After preprocessing, *a.cpp* and *b.cpp* will contain the following content
> ```
> namespace util {
> int add(int a,int b){ return a+b; }
> }
> ```
>
> Linker found two identical `util::add` that violated ODR and reported an error.

There are of course some exceptions, and certain functions can/must be implemented in the header:
* inline allows multiple definitions
* template must be placed in the header → template is instantiated at compile time <sup>[*]</sup>

> [*] For details, please refer to [Template](./templates.md)

---

# Nested namespace

`namespace` can be nested, and C++17 introduces the following syntax, which is equivalent to both.
```cpp
namespace Company::Org::Team {
    void function();
}
```


```cpp
namespace Company {
namespace Org {
namespace Team {
    void function();
}
}
}
```

---

# Anonymous namespace

The content defined in anonymous `namespace` is only visible in this cpp file, which is equivalent to making some content private.

```cpp
// mod.cpp
namespace mod {
namespace {
    int private_attr = ...;
    int private_function() {...};
}

    int public_function() {...};
}
```


```cpp
#include "mod.h"

int main() {
    mod::public_function(); // ✅
    mod::private_function(); // ❌
}
```
