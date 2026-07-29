# `inline`

最早的含义是：**建议编译器把函数调用展开成函数体本身**，从而减少函数调用开销。

> 把代码实现copy-paste到代码里直接运行，而不用让程序进行跳转来执行函数

但注意：现代 C++ 中，**`inline` 更重要的作用不是“强制内联优化”，而是解决多重定义问题。**

---

# 内联

`inline` 只是给编译器一个建议，最终是否真的展开，由编译器决定。

编译器可能拒绝内联：

```cpp
inline void f() {
    // 函数体太大，编译器可能不内联
}
```

编译器也可能自动内联没有写 `inline` 的函数：

```cpp
int square(int x) {
    return x * x;
}
```

---

# 多重定义

普通函数如果定义在头文件里，并被多个 `.cpp` 文件包含，链接时会报错 `multiple definition`。

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

用 `inline` 解决头文件函数定义问题。这样即使多个 `.cpp` 都包含这个头文件，也不会违反 ODR。

```cpp
// utils.h
inline int add(int a, int b) {
    return a + b;
}
```

---

# 类成员函数

## 类内定义默认是 inline

当在类内定义函数实现的时候，默认是inline的。
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

## 类外定义需要显式

如果成员函数定义写在类外，需要显式 `inline` 才能实现 inline 的效果。

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

# `inline` 变量

一般来说 static 的成员变量要在类外定义，但是 `inline static` 可以在类内。

```cpp
// A.h
class A {
public:
    inline static int count = 0;
};
```

> `inline` 变量的本质是链接器会把多个 inline 定义合并成同一个变量。

---

# `inline` vs `static`

* inline &rarr; 多个TU可以看到同一个 x，最终整个程序中共享同一个变量
* static &rarr; 每个TU都有自己独立的一份 x


```cpp
// config.h
static int x = 1;
inline int y = 2;

// a.cpp
#include "config.h"

// b.cpp
#include "config.h"
```

那么 `a.cpp` 和 `b.cpp` 中的
- `x` 是两份不同的变量
- `y` 是同一个变量

---

# 为什么一般 `inline` 函数定义在头文件

inline 其中一个用途，是让多个 TU 都能看到同一个函数定义。而只有把函数定义放在头文件里并被多个 .cpp 导入，才有“跨多个TU”的情况。

---

# `inline` 的优缺点

| 优点 | 缺点 |
| --- | --- |
| 允许在头文件中定义函数或变量，而不容易触发 `multiple definition` | 写了 `inline` 也不等于编译器一定会真的内联展开 |
| 对很短小的函数，编译器有机会直接展开，减少函数调用开销 | 如果函数体很大，强行放在头文件里可能增加编译时间 |
| 对头文件库（header-only library）很重要，便于把实现直接写在头文件里 | 多个 `.cpp` 都包含同一个头文件时，每个翻译单元都要看到这份定义，可能增大生成代码和编译负担 |
|  | 把太多实现细节放进头文件，会增加耦合，也让改动更容易触发大范围重新编译 |

---

# 为什么不能一直用 `inline`

只有在函数非常短、被极其频繁调用，且能确定内联确实能带来性能提升时才使用 inline。

## 代码体积膨胀（Code Bloat）

如果一个内联函数有 10 行代码，在 1000 个地方调用，那么编译器会将这 10 行代码复制 1000 次，产生 10000 行可执行代码。这导致最终的可执行文件过大。

## 增加编译压力与依赖

内联函数的定义通常必须放在头文件（.h）中，以便编译器在每个调用点都能找到函数体。而这会导致编译时间变长：一旦该内联函数修改，所有包含该头文件的 .cpp 文件都必须重新编译。

## 函数限制
不是所有函数都能内联：
- 递归函数通常不能内联
- 过长的函数内联后收益极低
