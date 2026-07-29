# Instance attributes, class attributes

Instance properties are bound to specific objects and are usually different for different objects.

```python
class Person:
    def __init__(self, name):
        self.name = name
```

Class attributes are bound to the class itself, and all instances share access.

```python
class Person:
    species = "human"
```

When looking for attributes, we look for the instance first, then the class, and then up the inheritance chain. So if the instance has the same name as the class attribute, the instance's (more specific) will be used and the class's (more general) will be hidden.

---

# self

`self` is just a common name, which by convention means "the current instance object". When the instance is called, it is equivalent to passing the instance as the first parameter.

Therefore, you can actually use any name instead of `self`, as long as the first one is a special parameter used to represent the instance.

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

* `__init__` → Responsible for initializing the object
* `__new__` → Responsible for creating objects and returning instances

Equivalent to `__new__` executed first, used to create an instance, `__init__` executed later, used to initialize the instance.

> [!NOTE]
> Situations where `__new__` must be changed:
> * Inherit immutable types such as `tuple`, `str`, and `int`
> * Implement control instance creation logic such as singletons

---

# Instance methods, class methods, static methods

## Instance methods
The first parameter is usually `self`. It can access class properties or instance properties.

```python
class A:
    count = 0

    def __init__(self):
        self.val = ...

    def f(self):
        print(count)
        print(self.val)
```

## Class method
The first parameter is usually `cls`, marked with `@classmethod`. It cannot access instance properties (because there is no `self`).

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

## Static method
Logically related to the class, but does not depend on `self` or `cls`. It cannot access instance properties (unless explicitly passed in as parameters), but it can access class properties (by explicitly calling `Class.property`).

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

# Inheritance

`super()` will find the "next" implementation along the **MRO** (Method Resolution Order, method resolution order) of the current class.
- In single inheritance it looks like calling the parent class
- The essence of multiple inheritance is cooperative method calling

> [!NOTE]
> `super()` is recommended in multiple inheritance instead of hard-coding the parent class name. Because `super()` allows multiple parent classes to collaborate according to a unified MRO, and writing the parent class to death will destroy this chain.

## MRO (Method Resolution Order)

MRO determines the order in which property/method lookups are performed. Python uses **C3 linearization**.

```python
class A: ...
class B(A): ...
class C(A): ...
class D(B, C): ...

print(D.__mro__) # maybe (D, B, C, A, object)
```

For example, if a method does not exist in D, its search order is based on the priority of MRO, such as B→C→A. Therefore, we cannot simply say that `super()` is looking for the parent class, but is exploring it in the order of MRO.

---

# Polymorphism and Duck Typing

Python emphasizes **duck typing**:

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

Python prefers behavioral compatibility over type-level compatibility. As long as the object provides the required interface, it can be used. For example, in the above example, even if `NotAnimal` is not Animal, it can be called as long as it can be `speak()`.

---

# Abstract base class ABC

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass
```

As long as any abstract method in the class is not implemented, it cannot be instantiated.

> [!NOTE]
> Unlike c++, even if `@abstractmethod` is added, the method can still have a concrete implementation, but it is still regarded as an abstraction.

---

# Magic Methods

Magic methods are special methods of the form `__xxx__` that allow objects to support Python's built-in syntax and protocols.

* `__str__` and `__repr__`
    - `__str__` → `str(obj)` / `print(obj)`, user-oriented, readability first
    - `__repr__` → Debugging and interpreter display are more commonly used, for developers, try to be as accurate and unambiguous as possible
* `__getitem__` / `__setitem__` → `obj[key] = val`
* `__iter__` / `__next__` → implement iteration protocol, refer to [Iterator & Generator](./iterator-and-generator.md) for details
* `__call__` → can be called directly
    * Implement stateful functions → such as counters. If you use the function directly, you may need to cooperate with global variables, etc. By packaging it into a class, you can implement closures and hold the state internally.
    * Decorator → Compared with using functions, it can contain more complex logic, such as preprocess/postprocess,
* `__eq__` / `__hash__` → Construct a hashable object to use as key
* `__lt__`, etc. → Support comparison, custom sorting, etc.
* `__enter__` / `__exit__` → context management, please refer to [Context Manage](./closure.md#context-manage) for details

---

# Attribute control `@property`

Real `private` attribute control can be achieved by only adding `@property` to the attribute without adding `@setter`.

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

In addition, verification logic can be added later without changing the caller interface. For example, in the following example, check is performed during set to avoid adding verification logic in the user code for each setting.

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

# Encapsulation and "private"

Python doesn't have truly enforced private like Java/C++.  

- `_name`: means "internal use, direct external access is not recommended"
- `__name`: will trigger name mangling
