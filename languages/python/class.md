# 实例属性、类属性

实例属性绑定在具体对象上，不同对象通常不同。

```python
class Person:
    def __init__(self, name):
        self.name = name
```

类属性绑定在类本身上，所有实例共享访问入口。

```python
class Person:
    species = "human"
```

查找属性时，会先找实例，再找类，再沿继承链向上找。所以如果实例有和类属性重名的，则会使用实例的（更specific）而隐藏类的（更general）。

---

# self

`self` 只是一个普通名字，约定俗成表示“当前实例对象”。当实例调用的时候，相当于把实例作为第一个参数传入。

因此其实也可以使用任何名字来代替`self`，只要放第一个就是一个特殊参数用于代表实例。

```python
class A:
    def __init__(me):
        me.val = 10

    def equal(me, val):
        return me.val == val

a = A()
a.equal(10)  # 相当于 A.show(a, 10)
```

---

# `__init__` vs `__new__`

* `__init__` &rarr; 负责初始化对象
* `__new__` &rarr; 负责创建对象，返回实例

相当于`__new__` 先执行，用于创建实例，`__init__` 后执行，用于初始化实例。

> [!NOTE]
> 必须改 `__new__`的情况：
> * 继承 `tuple`、`str`、`int` 这类不可变类型
> * 实现单例等控制实例创建逻辑

---

# 实例方法、类方法、静态方法

## 实例方法
第一个参数通常是 `self`。它可以访问类属性或者是实例属性。

```python
class A:
    count = 0

    def __init__(self):
        self.val = ...

    def f(self):
        print(count)
        print(self.val)
```

## 类方法
第一个参数通常是 `cls`，用 `@classmethod` 标记。它无法访问实例属性（因为没有`self`）。

```python
class A:
    count = 0

    def __init__(self):
        self.val = ...

    @classmethod
    def f(cls):
        print(cls.count)
        # print(self.val)
```

## 静态方法
和类放在逻辑上相关，但不依赖 `self` 或 `cls`。它无法访问实例属性（除非显式作为参数传入），但是可以访问类属性（通过显式调用`Class.property`）。

```python
class A:
    count = 0

    def __init__(self):
        self.val = ...
    
    @staticmethod
    def f():
        print(A.count)
        # print(self.val)
```

---

# 继承

`super()` 会沿当前类的 **MRO**（Method Resolution Order，方法解析顺序），找到“下一个”实现。
- 在单继承里看起来像调用父类
- 在多继承里本质是协作式方法调用

> [!NOTE]
> 多继承里推荐 `super()` 而不是写死父类名。因为 `super()` 让多个父类能按统一 MRO 协作，而写死父类会破坏这一链条。

## MRO (Method Resolution Order)

MRO 决定属性/方法查找的顺序。Python 采用 **C3 linearization**。

```python
class A: ...
class B(A): ...
class C(A): ...
class D(B, C): ...

print(D.__mro__) # maybe (D, B, C, A, object)
```

比如一个方法在D中不存在，那么它的查找顺序就是根据MRO的优先次序，比如B&rarr;C&rarr;A。所以不能简单的说`super()`是找父类，而是按照MRO的顺序依次探索。

---

# 多态与 Duck Typing

Python 很强调 **duck typing**：

> If it walks like a duck and quacks like a duck, it’s a duck.

```python
class Dog:
    def speak(self):
        return "woof"

class NotAnimal:
    def speak(self):
        return "bkajnblafjf"

def make_sound(animal):
    return animal.speak()

make_sound(Dog())
make_sound(NotAnimal())
```

Python 更偏向行为兼容而不是类型层级兼容。只要对象提供所需接口，就可以被使用。比如上面的例子，即使`NotAnimal`不是Animal，但是只要它能`speak()`，就可以被调用。

---

# 抽象基类 ABC

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass
```

只要类里有任何一个abstract method没有被实现，就无法被实例化。

> [!NOTE]
> 和c++不同，即使加了`@abstractmethod`，方法还是可以有具体实现的，但是仍然视为一个抽象。

---

# Magic Methods（魔法方法）

魔法方法是形如 `__xxx__` 的特殊方法，用于让对象支持 Python 的内置语法和协议。

* `__str__` 和 `__repr__`
    - `__str__` &rarr; `str(obj)` / `print(obj)`，面向用户，可读性优先 
    - `__repr__` &rarr; 调试、解释器显示更常用，面向开发者，尽量准确、无歧义
* `__getitem__` / `__setitem__` &rarr; `obj[key] = val`
* `__iter__` / `__next__` &rarr; 实现迭代协议，详情参考[Iterator & Generator](./iterator-and-generator.md)
* `__call__` &rarr; 可以直接调用
    * 实现有状态的函数 &rarr; 比如计数器，如果直接使用函数可能需要配合全局变量等，通过包装成类就可以实现闭包，内部持有状态
    * 装饰器 &rarr; 相比于使用函数，可以包含更复杂的逻辑，比如preprocess/postprocess，
* `__eq__` / `__hash__` &rarr; 构建可哈希对象用作key
* `__lt__` 等 &rarr; 支持比较，自定义排序等
* `__enter__` / `__exit__` &rarr; 上下文管理，详情参考[Context Manage](./closure.md#context-manage)

---

# 属性控制 `@property`

通过只给属性加 `@property` 而不加 `@setter` 即可实现真正的 `private` 属性控制。

```python
class Object:
    __num = 10

    @property
    def num(self):
        return self.__num

obj = Object()
obj.__num           # ❌，但是实际上可以通过obj._Object__obj_num_val访问
obj._Object__num    # ✅
obj._Object__num = 0# ✅

obj.num             # ✅
obj.num = 0         # ❌
```

除此之外，可以后续加校验逻辑而不改调用方接口。比如下面例子中在set的时候进行check，从而避免每一次设置都在用户代码中加验证逻辑。

```python
class Person:
    def __init__(self, age):
        self._age = age

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        if value < 0:
            raise ValueError("age must be >= 0")
        self._age = value
```

---

# 封装与“私有”

Python 没有像 Java/C++ 那样真正强制的 private。  

- `_name`：表示“内部使用，不建议外部直接访问”
- `__name`：会触发名称重整（name mangling）
