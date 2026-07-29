# RTTI

RTTI (Run-Time Type Information) is a mechanism provided by C++ to obtain the actual type information of an object when the program is running.

In polymorphic scenarios (operating derived class objects through base class pointers or references), RTTI allows the program to determine the true type of the object.

---

# Core components of RTTI

## typeid

`typeid` is used to obtain the type information of the object and returns the `std::type_info` object.

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
If it is a polymorphic type (with virtual), then the dynamic type (runtime type) is returned; otherwise, the static type (compilation time type) is returned.

> [!NOTE]
> 1. Virtual is required to have dynamic types → If Base does not use virtual, then `typeid(*b).name()` is still the Base type
> 2. An exception may occur → If `Base* b = nullptr`, typeid may report an error
> 3. `.name()` returns a compiler-related string, which is not human-readable → such as `int -> i`, so the returned string cannot be used for logical judgment.

## dynamic_cast

`dynamic_cast` is used for safe downcasting and will check whether the type is legal at runtime.
> Downward transformation: Base → Derived

```cpp
Base* b = new Derived();
Derived* d = dynamic_cast<Derived*>(b);
```

---

# Implementation principle of RTTI

RTTI is usually implemented through a virtual function table (vtable): each class with a virtual function has a vtable, which contains type information pointers.

```
对象内存：
[ vptr ]  --->  vtable
                ├── 虚函数指针
                ├── std::type_info (类型信息)
                └── 继承关系信息
```

Among them, `std::type_info` is an object that contains some additional information, such as type identifier, helper function, etc.

## typeid

* Find the object through the pointer
* Read the vptr in the object
* Jump to vtable
* Get type_info from vtable
* Return Derived type information

## dynamic_cast

* Find vtable through vptr to get type information
* Check the inheritance relationship, such as "whether the current object is Derived", "whether it can be converted"
* Handle complex inheritance, such as traversing the inheritance tree to see which type meets the requirements

---

# RTTI Disadvantages
## There is a certain runtime overhead
For example, `dynamic_cast` is O(N) because it needs to traverse the inheritance relationship to find out which type should be used.

## Increase program size
RTTI is based on vtable and needs to store information such as type_info.

## Sometimes it violates the design principle of "use polymorphism instead of type judgment"

```cpp
// ❌ 并不是一个好的代码
if (typeid(*b) == typeid(Derived)) {
    // do something
}

// ✅ 理论上最简单合适的代码
b->virtualFunction();
```