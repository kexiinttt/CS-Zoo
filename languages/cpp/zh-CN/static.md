# `static` 是什么

`static` 是 C++ 里的一个关键字，但它在**不同位置**上含义不完全一样。  

- **控制生命周期**：让对象不随作用域结束而销毁，而是一直存在到程序结束。
- **控制可见性 / 归属关系**：让某个名字只在当前文件可见，或者让某个成员属于“类本身”而不是某个对象。

一般而言 `static` 有以下几种用法：
- 修饰**局部变量**
- 修饰**全局变量**
- 修饰**函数**
- 修饰**类的成员变量**
- 修饰**类的成员函数**

`static` 变量存储在 **“静态存储区”**，不是堆也不是栈。

---

# static 局部变量

当 `static` 修饰一个函数内部的局部变量时：

- 作用域仍然是当前函数 / 当前代码块内部 &rarr; 函数外无法访问
- **生命周期变成整个程序运行期间** &rarr; 不会因为函数结束就被销毁
- **初始化一次** &rarr; 不会每调用一次函数就重新创建一次

也就是说：

- 普通局部变量：每次进入函数创建，离开函数销毁
- `static` 局部变量：第一次进入函数时创建，之后一直保留

```cpp
#include <iostream>
using namespace std;

void test() {
    static int s_cnt = 0;
    int cnt = 0;
    s_cnt++;
    cnt++;
    cout << s_cnt << ", " << cnt << endl;
}

int main() {
    test(); // 1, 1
    test(); // 2, 1
    test(); // 3, 1
    return 0;
}
```
---

# static 全局变量

当 `static` 修饰全局变量时，它的重点不再是生命周期(因为已经是全局的了，所以生命周期就是整个程序)，而是：

- **链接属性变成内部链接（internal linkage）** &rarr; 只能在**当前.cpp**中使用，不能被其他 `.cpp` 文件通过 `extern` 访问

```cpp
// file1.cpp
static int g_value = 100;
```
```cpp
// file2.cpp
extern int g_value;  // 不能正确链接到 file1.cpp 里的 static 全局变量
```

其主要作用是：限制名字污染，避免不同文件之间误用或冲突。不过现在更常见的写法是使用匿名namespace。：

---

# static 函数

和限制全局变量一样，主要是**限制链接范围，只在本文件内可见**。

```cpp
// file1.cpp
static void helper() {
}
```

```cpp
// file2.cpp
#include <file1>
helper(); // ❌
```

---

# static 类成员变量

如果 `static` 修饰的是**类的成员变量**，那么这个变量：

- **属于类本身**
- 不属于某个具体对象
- 所有对象**共享**同一份数据

```cpp
#include <iostream>
using namespace std;

class Student {
public:
    Student() {
        ++count;
    }

    static int count;
};

int Student::count = 0;

int main() {
    Student s1;
    Student s2;
    Student s3;

    cout << Student::count << endl; // 3
    return 0;
}
```

## 初始化 static 成员变量

### 普通 static 成员

类内通常只能声明，不能完成定义。需要在类外单独定义，类外也可以顺便初始化。
```cpp
class A {
public:
    static int x;
};

int A::x = 10;
```

### static const 静态常量
可以定义和初始化在类内，这是特殊的规定。毕竟const必须在定义时初始化（或者使用member initializer list初始化），所以没有别的选择。
```cpp
class A {
public:
    static const int x = 10;
};
```

### static constexpr 成员

可以在类内初始化，是现代C++推荐的写法。
```cpp
class A {
public:
    static constexpr int x = 10;
};
```

### inline static
可以在类内初始化。
```cpp
class A {
public:
    inline static int x = 10;
};
```

---

# static 类成员函数

如果 `static` 修饰的是**类的成员函数**，那么它也是“属于类本身”的函数，而不是某个对象的方法。

- 可以通过类名直接调用
- **没有 `this` 指针**
- 只能访问：
  - `static` 成员变量
  - `static` 成员函数
  - 其他和对象无关的内容
- 不能直接访问普通成员变量和普通成员函数，因为那些需要具体对象

```cpp
#include <iostream>
using namespace std;

class Student {
public:
    static int count;

    static void printCount() {
        cout << count << endl;
    }
};

int Student::count = 0;

int main() {
    Student::printCount();
    return 0;
}
```


## 如何访问对象成员

如果确实要在 `static` 成员函数中访问某个对象的数据，需要显式传入对象：

```cpp
class A {
public:
    int x = 10;

    static void print(const A& a) {
        cout << a.x << endl;
    }
};
```

---

# static 的初始化问题

* 普通 static / 全局变量 &rarr; 这类对象通常在程序启动阶段初始化，程序结束时销毁
* 局部 static &rarr; 第一次执行到定义语句时初始化
* 类 static 成员变量初始化 &rarr; 类外 / `inline static` / `static constexpr` / `const static`