# 什么是闭包

闭包（closure）可以简单理解成：内部函数 + 引用了外部函数作用域中的变量 + 外部函数返回后这些变量仍然能被内部函数访问。

```python
def outer(x):
    def inner(y):
        return x + y
    return inner

add10 = outer(10)
print(add10(3))   # 13
```

---

# 为什么闭包成立
  
函数不仅能被调用，也能：

- 赋值给变量
- 作为参数传递
- 作为返回值返回

同时，函数会“携带”定义时需要的环境。所以即使外层函数结束了，内部函数仍然能访问被捕获的变量。

---

# `nonlocal` & `global`

如果闭包内部只是读取外部变量，直接用就行。如果要修改，通常需要 `nonlocal`。

```python
def counter():
    count = 0

    def inc():
        nonlocal count
        count += 1
        return count

    return inc
```

如果没有 `nonlocal`，Python 会把 `count` 当成局部变量，导致赋值时报错。

> [!NOTE]
> `nonlocal`和`global`都是表明某个变量不是局部变量，只不过作用域不同：
> * `nonlocal` &rarr; 变量来自于外层函数作用域，并不是全局变量
> * `global` &rarr; 全局变量

---

# Late Binding

晚绑定 &rarr; 没有在定义时把变量的“值”拷贝下来，而是在真正执行时再去查这个变量现在的值。

```python
funcs = []
for i in range(3):
    funcs.append(lambda: i)

print([f() for f in funcs])   # [2, 2, 2]
```
这 3 个 f 并没有各自保存当时的 i 的值，它们只是都“记住了同一个名字 i”。等最后调用 f() 时，循环早结束了，这时 i 的当前值是 2，所以都返回 2。

```python
funcs = []
for i in range(3):
    funcs.append(lambda i=i: i)

print([f() for f in funcs])   # [0, 1, 2]
```

具体的细节涉及到[free var & cell object](#free-variable--cell-object)。

可以理解为闭包使得inner和outer都指向同一个cell，而这个cell存储的是free variable (`i`)。当invoke的时候才会去读这个cell，但是此时这个值已经是最新的了。

而当设置默认参数的时候`lambda i=i: i`，此时的 `i` 就不是闭包的nonlocal了，而是一个传入的局部变量，那么和free variable / cell object无关了。

---

# free variable & cell object

```py
def outer():
    x = 10

    def inner():
        return x
    return inner

f = outer()

print(f.__closure__)                    # (<cell at 0x...: int object at 0x...>,)
print(f.__code__.co_freevars)           # ('x',)
print(f.__closure__[0].cell_contents)   # 10
```

## free variable

在当前函数里被使用，但不是当前函数的局部变量，而是来自外层作用域的变量。比如例子里的 `x`。

## cell object

Python 为了让外层函数结束后，内部函数还能继续访问这些变量，不会只是简单把局部变量丢掉，而是会把这类被闭包捕获的变量放进一个特殊容器里，这个容器就是 cell object。

> cell object 是 Python 用来保存闭包变量的盒子

---

# 闭包 vs 类

有时候闭包和类都能解决问题。

- 闭包：状态简单、逻辑清晰
- 类：状态复杂、行为较多、组要扩展

---

# Decorator

装饰器底层最常见的实现方式就是闭包。

```python
def deco(func):
    def wrapper(*args, **kwargs):
        print("before")
        result = func(*args, **kwargs)
        print("after")
        return result
    return wrapper

@deco
def greet(name):
    print(f"hello {name}")
```

## 带参数装饰器

```python
def repeat(n):
    def deco(func):
        def wrapper(*args, **kwargs):
            for _ in range(n):
                func(*args, **kwargs)
        return wrapper
    return deco

@repeat(3)
def hi():
    print("hi")
```

⚠️ 注意顺序，其实`@repeat(3) def hi()`相当于`repeat(3)(hi)()`，所以：
* 最外层 &rarr; 装饰器参数
* 中间层 &rarr; 被装饰的函数
* 最内层 &rarr; 函数的参数

## 执行顺序

```python
def deco(func):
    print("A") # ⚠️ 第一个输出
    def wrapper(*args, **kwargs):
        print("C")
        func(*args, **kwargs) # ⚠️ 此处返回一个生成器，但是不调用就不会执行内部
        # 如果想打印func内部
        # g = func(*args, **kwargs)
        # next(g)
        # next(g)
    return wrapper

@deco
def func():
    print("never print")
    yield
    print("never print")

print("B")
func()
print("D")
```
> [!IMPORTANT]
> 上述代码中`@deco`相当于是`deco_func = deco(func)`，所以 A 无论如何都会被输出，该行为发生在 **函数定义** 阶段，并不需要call就会执行。因此理论上任何函数使用`@deco`都会执行一次。
> 
> 可以利用这个特性将被装饰的内容注册到global。这样无论是否调用，在定义阶段就会都执行。
> ```python
> REGISTER_TABLE = {}
> 
> def register_decorator(func):
>    REGISTER_TABLE[func.__name__] = func
>    def wrapper(*args, **kwargs):
>        ...
>    return wrapper
> ```

## `functools.wraps`

写装饰器时最好保留原函数元信息，否则被装饰函数的一些信息会被 wrapper 覆盖。
- `__name__`
- `__doc__`
- 其他元信息

这会导致一些反射相关的功能错误或失效。

```python
from functools import wraps

def deco(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

---

# Context Manage

上下文管理器最常见的使用方式是 `with`：

```python
with open("a.txt", "r") as f:
    data = f.read()
```

其核心目标是**把资源申请和资源释放绑定起来**，这样即使中间发生异常，也能执行清理逻辑。

## 基于类

需要实现：

- `__enter__`：当进入上下文时要做的事，返回一个对象用于上下文内部操作
- `__exit__`：退出上下文时要做的事

```python
class MyContext:
    def __enter__(self):
        print("enter")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("exit")

with MyContext() as c:
    print("inside")
```

`__exit__` 的三个参数：
- `exc_type`：异常类型
- `exc_val`：异常对象
- `exc_tb`：traceback

如果 `__exit__` 返回 `True`，表示异常被处理/抑制，不再继续传播。

## 基于 `contextlib.contextmanager`

```python
from contextlib import contextmanager

@contextmanager
def my_context():
    print("enter")
    try:
        yield "inside"
    finally:
        print("exit")

with my_context() as r:
    print(r)
```

- `yield` 前：相当于 `__enter__`
- `yield` 产出的值：相当于 `as` 绑定的对象
- `yield` 后 `finally`：相当于 `__exit__`

为了保证能处理异常，以及即使有异常也要做清理，最好使用 try-except-finally 来实现。
