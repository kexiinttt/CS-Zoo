# 什么是 Reflection

Reflection 指的是：程序在编译期或运行时，能够检查、理解、甚至操作自身的结构信息。

这些“结构信息”通常包括：
- 类型名
- 成员变量
- 成员函数
- 继承关系
- 枚举值
- 模板参数
- 属性 / 元数据 (metadata)

> 你有一个对象，但你在运行的时候不知道它是哪个类，或者不知道它有哪些成员。反射就是让程序能够“照镜子”，看着自己说：“噢，我是一个叫 User 的类，我有 name 和 age 两个成员，我还可以调用 save() 方法。”


比如Python就可以很容易实现运行期的反射。
```py
class Person:
	...

p = Person(...)

# 内省（Introspection）： 获取类型信息（类名、成员变量、成员函数）
print(vars(p))

# 动态调用（Dynamic Invocation）： 根据名称字符串调用方法
method = getattr(p, "func")
method()

# 动态实例化（Dynamic Creation）： 根据类名字符串创建对象
cls = globals()["Person"]
pp = cls(...)
```

而C++作为静态语言，编译后很多类型名称等都变成了静态的内存地址，因此很难直接获得动态语言类似的反射。

---

# Reflection 的核心应用场景

## 通用日志 / 调试 / 打印

对于Python等而言非常简单，因为Python的class有metadata，知道这个class包含的成员和方法，所以可以直接print出来所有。
```py
class Person:
	def __init__(self, ...):
		self.name = ...
		self.age = ...

p = Person(...)
print(p) # 完全自动识别成员 Person(name=..., age=...)
```

而C++的class并没有办法知道所有的成员或者方法，只能人为控制输出的成员。

```cpp
class Person {
	string name;
	int age;
};

Person p(...);
cout << "name=" << p.name << ", age=" << p.age; // 没法自动识别有什么成员
```

## 序列化 / 反序列化

例如把对象转成：JSON，YAML，XML等。如果有 reflection，就可以自动遍历成员：
```cpp
// 理想情况下可以自动生成：{"name": "Alice", "age": 20}
struct User {
    std::string name;
    int age;
};
```

## ORM / 配置系统

比如把对象映射到：数据库表，配置文件，命令行参数等。
```cpp
// reflection 可以帮助自动：读取配置，字段校验，自动绑定CLI参数等
struct Config {
    std::string host;
    int port;
    bool enable_ssl;
};
```

---

# RTTI

详情参考[RTTI](./rtti.md)。

RTTI 不是完整 reflection
* typeid
	- 只能拿到非常有限的 type info 且通常是 mangled name
	- 不能遍历成员变量，函数
	- 不能动态列出字段名称
* dynamic_cast
	- 只能知道是不是派生类
	- 不能遍历成员变量，函数
---

# Compile-time Introspection

## type traits

```cpp
#include <type_traits>

template <typename T>
void foo(const T& value) {
    if constexpr (std::is_integral_v<T>) {
        // 整数逻辑
    } else if constexpr (std::is_floating_point_v<T>) {
        // 浮点逻辑
    } else {
        // 其他类型
    }
}
```

虽然不能“列出 T 的所有成员”，但可以：
- 判断类型类别
- 判断是否可拷贝
- 判断是否 trivially copyable
- 判断是否是 polymorphic
- 判断继承关系
- 判断是否是 enum / class / reference / pointer

详情参考[type traits](./type-traits.md)。

## detection idiom

```cpp
#include <type_traits>
#include <utility>

template <typename, typename = void>
struct has_begin : std::false_type {};

template <typename T>
struct has_begin<T, std::void_t<decltype(std::declval<T>().begin())>>
    : std::true_type {};

template <typename T>
inline constexpr bool has_begin_v = has_begin<T>::value;
```

比如上述代码检测是否有某个方法。

详情参考[concept](./concepts-and-requires.md)，[SFINAE](./sfinae.md)等。

## marco

在工程里，最常见的 reflection 方案之一其实是：宏 + 注册。

```cpp
#include <iostream>
#include <string>

// 把 PERSON_FIELDS(X) 的文本替换成 X(name) 和 X(age)
#define PERSON_FIELDS(X) \
    X(name)              \
    X(age)

struct Person {
    std::string name;
    int age;

	// PERSON_FIELDS(X)
	//
	// X(name)
	// X(age)
	//
	// std::cout << "name" << ": " << name << "\n";
	// std::cout << "age" << ": " << age << "\n";
    void print() const {
		// X(arg) 的文本等价于 std::cout 的一个文本
#define X(field) std::cout << #field << ": " << field << "\n";
        PERSON_FIELDS(X)
#undef X
    }
};

// name: Alice
// age: 30
Person p{"Alice", 30};
p.print();
```

其本质是把字段清单集中维护一次，再多处展开使用。这不是真 reflection，但能达到：“避免重复写字段列表”和“自动生成样板代码”的功能。

## 代码生成（code generation）

很多大型 C++ 项目里的 reflection，本质是外部工具生成代码。比如用户自定义schema，Domain Specific Language等，某些工具会自动转换并生成对应的代码。
