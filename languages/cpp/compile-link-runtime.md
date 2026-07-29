# C++ 常见错误总结

C++ 里的常见错误，按**发生阶段**来分类：

1. 编译期错误 (compile error)
2. 链接期错误 (linker error / linkage error)
3. 运行期错误 (runtime error)
4. 逻辑错误 (logic error)
5. 未定义行为 (Undefined Behavior)

---

# Compile Error

这是最早发生的一类错误：**代码连机器码都生不出来**。

也就是说，编译器在“翻译你的代码”时就发现有问题了。

## 常见原因

#### 语法错误
#### 类型不匹配
#### 没有声明 / 找不到名字
#### 头文件没包含
#### 模板约束不满足

```cpp
#include <vector>

int main() {
    std::vector<int> v;
    v.push_back("hello");  // const char* 不能直接当 int
}
```

## 编译错误的特点

- 程序无法生成可执行文件
- 错误通常定位得比较精确
- 通常是“代码写法不合法”或者“类型系统不允许”

---

# Linker Error / Linkage Error

编译器先把每个 `.cpp` 文件各自编译成目标文件 `.o`，然后linker再把这些目标文件拼起来，组成最终可执行文件。

如果“拼装阶段”出问题，就是 **linker error**。

## Undefined Reference / Unresolved External Symbol

声明了某个东西，编译器相信它存在，但链接时发现根本找不到定义。

#### 函数声明了但没定义

```cpp
#include <iostream>

void foo();

int main() {
    foo();
}
```

这里 `foo()` 只有声明，没有定义。

编译器会想：
- “好，foo 存在，先记着”
- 到链接阶段，链接器找不到 `foo` 的实现
- 报错：`undefined reference to foo`

#### 类的成员函数声明了但没实现

```cpp
class A {
public:
    void print();
};

int main() {
    A a;
    a.print(); // 没有A::print的定义
}
```

#### 头文件里写了 extern，但没定义

```cpp
// a.h
extern int g_value;
```

如果某个 `.cpp` 里用了它，但没有任何地方真正写：

```cpp
int g_value = 10;
```

也会链接失败。

## Multiple Definition

同一个符号被定义了多次。

#### 普通函数定义写在头文件里，被多个 cpp include

```cpp
// util.h
void foo() {
}
```

如果 `a.cpp` 和 `b.cpp` 都 include 了 `util.h`，那么 `foo()` 会在多个 translation unit 中都被定义一次。

链接器会报：`multiple definition of foo`

#### 全局变量在头文件直接定义

```cpp
// config.h
int g_count = 0;
```

多个 cpp include 后，就会产生多个 `g_count`。

正确做法：

```cpp
// config.h
extern int g_count;
```

```cpp
// config.cpp
int g_count = 0;
```

---

# Runtime Error

这个意思是：程序成功编译、成功链接、也启动了，但运行过程中崩了，或者出现非法行为。


## 常见运行时错误

### Segmentation Fault

程序访问了不该访问的内存区域。

操作系统给每个进程分配虚拟内存空间，不是所有地址都能随便访问。

如果你访问了：
- 根本不属于你的地址
- 没权限访问的地址
- 已经无效的地址

操作系统会发一个非法内存访问异常，程序就触发 segmentation fault。

#### 空指针解引用

```cpp
int main() {
    int* p = nullptr;
    *p = 10;
}
```

#### 数组越界

```cpp
int main() {
    int arr[3] = {1, 2, 3};
    arr[10] = 5;   // 未定义行为，可能 segfault
}
```

#### use after free

```cpp
int main() {
    int* p = new int(42);
    delete p;
    *p = 100;   // 已释放后继续使用，未定义行为
}
```

### Bus Error
和 segfault 有点像，也是非法内存访问，但更偏向：
- 非法对齐访问
- 访问不存在的物理地址映射

### Floating Point Exception
不一定真是浮点数问题，很多时候是：
- 整数除 0
- 某些非法算术操作

### Abort / Assertion Failure

```cpp
#include <cassert>

int main() {
    int x = -1;
    assert(x > 0);
}
```

### std::terminate
常见原因：
- 异常没被捕获
- `noexcept` 函数里抛异常
- `std::thread` 没有 join/detach 就析构

---


# Undefined Behavior

你的代码违反了 C++ 规则，后果完全不受保证。编译器和运行结果都可以“随便”。

## 常见 UB 例子

#### 越界访问

```cpp
int arr[3];
arr[5] = 10;
```

#### 解引用空指针

```cpp
int* p = nullptr;
*p = 1;
```

#### use after free

```cpp
int* p = new int(1);
delete p;
std::cout << *p;
```

#### 返回局部变量引用

```cpp
int& foo() {
    int x = 10;
    return x;   // UB
}
```

#### 有符号整数溢出
```cpp
int max(int a, int b) {
    return a < b ? a : b;
}
```