# What is closure

Closure can be simply understood as: internal function + references variables in the scope of the external function + these variables can still be accessed by the internal function after the external function returns.

```python
def outer(x):
    def inner(y):
        return x + y
    return inner

add10 = outer(10)
print(add10(3))   # 13
```

---

# Why closure is established
  
Not only can functions be called, they can also:

- Assign value to variable
- passed as parameter
- Returned as a return value

At the same time, the function will "carry" the environment required when it is defined. So even if the outer function ends, the inner function can still access the captured variables.

---

# `nonlocal` & `global`

If the closure only reads external variables, just use it directly. If you want to modify it, you usually need `nonlocal`.

```python
def counter():
    count = 0

    def inc():
        nonlocal count
        count += 1
        return count

    return inc
```

If there is no `nonlocal`, Python will treat `count` as a local variable, causing an error when assigning.

> [!NOTE]
> `nonlocal` and `global` both indicate that a variable is not a local variable, but has different scopes:
> * `nonlocal` → The variable comes from the outer function scope and is not a global variable
> * `global` → global variable

---

# LateBinding

Late binding → The "value" of the variable is not copied during definition, but the current value of the variable is checked during actual execution.

```python
funcs = []
for i in range(3):
    funcs.append(lambda: i)

print([f() for f in funcs])   # [2, 2, 2]
```
These three f do not each save the value of i at that time, they just "remember the same name i". When f() is finally called, the loop ends early. At this time, the current value of i is 2, so 2 is returned.

```python
funcs = []
for i in range(3):
    funcs.append(lambda i=i: i)

print([f() for f in funcs])   # [0, 1, 2]
```

The specific details involve [free var & cell object](#free-variable--cell-object).

It can be understood that the closure makes both inner and outer point to the same cell, and this cell stores a free variable (`i`). This cell will only be read when invoked, but at this time the value is already the latest.

And when setting the default parameters `lambda i=i: i`, `i` at this time is not the nonlocal of the closure, but an incoming local variable, so it has nothing to do with the free variable / cell object.

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

It is used in the current function, but it is not a local variable of the current function, but a variable from the outer scope. For example, `x` in the example.

## cell object

In order to allow the inner function to continue to access these variables after the outer function ends, Python will not simply throw away the local variables, but will put such variables captured by the closure into a special container, which is the cell object.

> cell object is a box used by Python to store closure variables

---

# Closures vs classes

Sometimes closures and classes can solve the problem.

- Closure: simple state and clear logic
- Class: complex status, many behaviors, group needs to be expanded

---

# Decorator

The most common low-level implementation of decorators is closures.

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

## Decorator with parameters

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

⚠️ Pay attention to the order. In fact, `@repeat(3) def hi()` is equivalent to `repeat(3)(hi)()`, so:
* Outermost → decorator parameter
* Middle layer → decorated function
* Innermost layer → parameters of function

## Execution order

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
> `@deco` in the above code is equivalent to `deco_func = deco(func)`, so A will be output anyway. This behavior occurs in the **function definition** stage and will be executed without a call. Therefore, in theory any function using `@deco` will be executed once.
>
> You can use this feature to register decorated content into global. In this way, regardless of whether it is called or not, it will be executed during the definition phase.
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

When writing a decorator, it is best to retain the original function meta-information, otherwise some information of the decorated function will be overwritten by the wrapper.
- `__name__`
- `__doc__`
- Other meta information

This can lead to some reflection-related functional errors or failures.

```python
from functools import wraps

def deco(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

---

# ContextManage

The most common way to use a context manager is `with`:

```python
with open("a.txt", "r") as f:
    data = f.read()
```

Its core goal is to bind resource application and resource release, so that even if an exception occurs in the middle, the cleanup logic can be executed.

## Based on class

Need to achieve:

- `__enter__`: What to do when entering the context, return an object for internal operations of the context
- `__exit__`: What to do when exiting the context

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

Three parameters of `__exit__`:
- `exc_type`: exception type
- `exc_val`: Exception object
- `exc_tb`: traceback

If `__exit__` returns `True`, it means that the exception is handled/suppressed and will not continue to propagate.

## Based on `contextlib.contextmanager`

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

- Before `yield`: Equivalent to `__enter__`
- The value output by `yield`: equivalent to the object bound by `as`
- `yield` followed by `finally`: equivalent to `__exit__`

In order to ensure that exceptions can be handled and cleaned up even if there are exceptions, it is best to use try-except-finally.
