# 什么是 Namespace

`namespace` 是 C++ 用来避免命名冲突（**name collision**）的机制。它其实只是 **符号前缀（symbol prefix）**。例如：

```cpp
math_utils::add // 编译后符号类似 -> `_ZN10math_utils3addEii`
```

不同 header 可以扩展同一个 namespace (**namespace extension**)。例如：

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

# 如何使用

⚠️ 不推荐在 header 使用`using namespace std;`，因为：
* namespace 污染
* 名字冲突
* 大型项目容易出 bug

---

# 在多个文件共享

在 header 声明，在 cpp 实现。

> [!NOTE]
> 如果在 header 中实现函数，会容易触发 ODR (One Definition Rule)。
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
> 编译会报错 `multiple definition of util::add`。
> 
> 因为 `#include` 本质是 **文本复制**。预处理后 *a.cpp* 和 *b.cpp* 都会包含下面的内容
> ```
> namespace util {
> int add(int a,int b){ return a+b; }
> }
> ```
> 
> Linker 发现两个相同的 `util::add` 违反了ODR所以报错。

当然有一些例外，某些函数可以/必须在 header 中实现：
* inline 允许多个定义
* template 必须放在header &rarr; template 在编译期实例化<sup>[*]</sup>

> [*] 详情参考[Template](./templates.md)

---

# Nested namespace

`namespace`可以嵌套，C++17引入下列语法，两者等同。
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

匿名 `namespace` 中定义的内容只在本 cpp 文件可见，相当于把一些内容 private 了。

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
