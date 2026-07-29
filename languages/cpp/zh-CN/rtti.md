# RTTI

RTTI (Run-Time Type Information，运行时类型信息) 是C++提供的一种机制，用于在程序运行时获取对象的实际类型信息。

在多态场景 (通过基类指针或引用操作派生类对象) 中，RTTI允许程序判断对象的真实类型。

---

# RTTI的核心组成

## typeid

`typeid` 用于获取对象的类型信息，返回 `std::type_info` 对象。

```cpp
#include <iostream>
#include <typeinfo>

class Base {virtual void foo() { <GO>} };
class Derived : public Base {};

int main() {
    Base* b = new Derived();
    std::cout << typeid(*b).name() << std::endl; // Derived (dynamic type)
    std::cout << typeid(b).name() << std::endl; // Base* (static type)
}
```
如果是多态类型(有virtual)，那么返回动态类型(运行时类型)；否则返回静态类型(编译期类型)。

> [!NOTE]
> 1. 需要有virtual才有动态类型 &rarr; 如果Base没有使用virtual，那么`typeid(*b).name()`仍然是Base类型
> 2. 可能会出现异常 &rarr; 假如`Base* b = nullptr`那么typeid可能会报错
> 3. `.name()`返回的是编译器相关字符串，不是human-readable &rarr; 比如`int -> i`，所以返回的不能用于逻辑判断

## dynamic_cast

`dynamic_cast` 用于安全的向下转型 (downcasting)，会在运行时检查类型是否合法。
> 向下转型：Base &rarr; Derived

```cpp
Base* b = new Derived();
Derived* d = dynamic_cast<Derived*>(b);
```

---

# RTTI的实现原理

RTTI通常通过虚函数表（vtable）实现：每个有虚函数的类都有一个vtable，里面包含类型信息指针。

```
对象内存：
[ vptr ]  --->  vtable
                ├── 虚函数指针
                ├── std::type_info (类型信息)
                └── 继承关系信息
```

其中`std::type_info`是一个对象，包含了一些额外信息，比如：类型标识符，helper function等。

## typeid

* 通过指针找到对象
* 读取对象中的 vptr
* 跳转到 vtable
* 从 vtable 中拿到 type_info
* 返回 Derived 类型信息

## dynamic_cast

* 通过vptr找到vtable从而得到类型信息
* 检查继承关系，比如 "当前对象是不是 Derived", "是否能转换"
* 处理复杂继承，比如遍历继承树来看到底哪一个是符合要求的类型

---

# RTTI的缺点
## 有一定运行时开销
比如 `dynamic_cast` 是O(N)的，因为他要遍历继承关系才能找到到底应该使用哪个类型。

## 增加程序体积
RTTI基于vtable，需要存储type_info等信息。

## 有时违背"用多态替代类型判断"的设计原则

```cpp
// ❌ 并不是一个好的代码
if (typeid(*b) == typeid(Derived)) {
    // do something
}

// ✅ 理论上最简单合适的代码
b->virtualFunction();
```