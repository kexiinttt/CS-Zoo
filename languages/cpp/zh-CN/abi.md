# 什么是 ABI

ABI 是 **Application Binary Interface**，也就是“应用二进制接口”，解决的是“编译成二进制之后，不同模块能不能正确地连起来、跑起来”

ABI 是“编译器和链接器最终怎么约定这些代码在机器层面协作”。

ABI 关注的不是函数名字表面长什么样，而是更底层的二进制约定，比如：
- 函数参数怎么传
- 返回值怎么传
- 符号名如何修饰 (name mangling)
- 类对象在内存里的布局
- 虚函数表怎么组织
- 异常如何跨模块传播
- 不同编译器/标准库生成的二进制是否兼容

---

# 为什么 ABI 很重要

## 动态库/静态库兼容性

你写了一个 `.so` / `.dll` / `.a`，别人拿去链接时，除了头文件能编译通过，还必须保证二进制层面的约定一致，否则可能：

- 链接失败
- 运行时崩溃
- 对象布局错乱
- delete/free 错对象
- 异常跨边界出问题

## 编译器兼容性

同样的 C++ 源码，不同编译器可能：

- mangling 规则不同
- 类布局细节不同
- RTTI / exception 实现不同
- 标准库实现不同

所以很多 C++ 库虽然“API 一样”，但**不一定 ABI 兼容**。

## 版本升级兼容性

一个库升级版本后，即使源码接口没改太多，只要动了对象布局或符号，就可能破坏 ABI，导致旧程序不能直接替换新库。

- ABI stability
- backward binary compatibility

---

# ABI 包含哪些内容

## Calling Convention（调用约定）

函数调用时需要约定：

- 参数放寄存器还是栈上
- 谁负责清理栈
- 返回值放哪里
- this 指针怎么传

即使源码看起来一样，但底层调用约定可能不同。

## Name Mangling（名字修饰）

C++ 支持：

- 函数重载
- namespace
- 类成员函数
- template

所以编译器不能只保留源码里的原始函数名，必须把类型信息等编码到符号里。

例如：

```cpp
void func(int);
void func(double);
```

编译后不会都叫 `func`，而是不同的 mangled name。

这就带来两个后果：

1. 不同编译器的 mangling 规则可能不同  
2. C++ 符号通常不适合作为稳定跨编译器 ABI

因此很多对外暴露的二进制接口会使用：

```cpp
extern "C"
```

来关闭 C++ name mangling，使导出符号更稳定。

## Object Layout（对象布局）

类在内存中怎么排布，也是 ABI 的一部分，例如：

- 成员变量顺序
- padding / alignment
- base class placement
- vptr 放在哪
- multiple inheritance 布局
- virtual inheritance 布局

```cpp
struct A {
    int x;
    char y;
};
```

它的 `sizeof(A)` 往往不是 5，而可能是 8，因为有对齐和 padding。

如果库版本升级时改成：

```cpp
struct A {
    char y;
    int x;
};
```

源码层面看起来只是换了顺序，但 ABI 已经变了，因为内存布局变了。

## Virtual Table / RTTI

带虚函数的类一般会有：

- vptr
- vtable
- RTTI 信息

这些实现细节通常也受 ABI 约束。

例如：

```cpp
struct Base {
    virtual void f();
    virtual ~Base() = default;
};
```

对象里通常会多一个隐藏指针指向 vtable。  
如果你修改虚函数顺序、增加/删除虚函数，vtable 布局就可能变化，从而破坏 ABI。

## Exception Handling

异常跨模块传播时，依赖：

- 栈展开机制
- type info
- runtime support
- 标准库/编译器的异常实现

> 不要轻易让 C++ 异常跨动态库边界传播，尤其是编译器、标准库或编译选项不统一时。

## 标准库类型的 ABI

像这些类型：

- `std::string`
- `std::vector`
- `std::list`
- `std::function`

它们的内部实现细节不属于语言标准强制规定，而是由标准库实现决定。  
因此不同版本的 libstdc++ / libc++ / MSVC STL 之间，ABI 可能不同。

---

# Opaque Handle

很多大型 C++ 项目会采用：内部用 C++，对外导出 C 接口，原因是 C 的 ABI 通常更简单、更稳定：

- 没有函数重载
- 没有 name mangling（或规则更简单）
- 没有类对象布局兼容问题
- 没有模板实例化 ABI 问题
- 更适合跨语言调用

典型写法：

```cpp
extern "C" {
    void* create_handle();
    void destroy_handle(void* p);
}
```

但解决不了：

- 调用约定差异
- 对象布局差异
- 异常机制差异
- allocator/runtime 差异
- STL/RTTI/vtable 问题

---

# 哪些改动容易破坏 ABI

* 修改类成员变量
    - 新增成员
    - 删除成员
    - 调整顺序
    - 改类型
    - 改访问控制通常不影响 ABI，但改布局会影响

* 修改虚函数表
    - 增加虚函数
    - 删除虚函数
    - 调整虚函数声明顺序
    - 修改虚继承结构

* 修改函数签名
    - 参数类型变化
    - 返回类型变化
    - `noexcept` 变化在某些场景下也可能影响接口约定/符号
    - 改默认参数本身通常是 API 问题，不直接是 ABI 问题，因为默认参数在调用点展开

* 改变模板实例的暴露方式
* 暴露 STL 成员或返回 STL 类型

---

# 和 template / inline 的关系

template / inline 的实现写在 header 里，用户一旦编译，就会把这些实现实例化/编进自己的二进制。这意味着就算替换动态库，用户编译的运行代码仍然使用了旧的逻辑。

> 相当于普通函数是每次调用就使用动态库里那个；模版和inline是编译的时候就把代码“复制粘贴”了过来，调用的时候不需要查找动态库，所以只要不重新编译就永远用的旧的，不论动态库怎么改变。


## template 为什么对 ABI 不友好

普通函数库像“成品机器”（.so），你换新机器，旧程序还能接着用。模板更像“图纸”（在头文件），用户自己现场造机器。

所以你改了图纸（模板实现）后：
- 用户不重新编译，就还是旧机器
- 重新编译后，机器内部细节可能变了
- 不同工厂（编译器/标准库）造出来还可能不一样

## inline 为什么对 ABI 不友好

`inline` 函数常定义在头文件中。调用方可能直接把函数体编译进自己的二进制里。

因此即使你替换了动态库，如果调用方没有重新编译，旧 inline 逻辑还在。

---
# PImpl (Pointer to Implementation)

```cpp
// widget.h
class Widget {
public:
    Widget();
    ~Widget();

    void doWork();

private:
    struct Impl;
    std::unique_ptr<Impl> impl_;
};
```

```cpp
// widget.cpp
struct Widget::Impl {
    int x;
    std::string name;

    void doWorkImpl() {
        // real logic
    }
};

Widget::Widget() : impl_(std::make_unique<Impl>()) {}
Widget::~Widget() = default;

void Widget::doWork() {
    impl_->doWorkImpl();
}
```

用户看到的是 Widget，看不到 Impl 里面到底有什么成员和实现，这样用户能够得到非常稳定的public interface。相当于把 **"容易变化的实现"隔离到ABI边界后面**。

