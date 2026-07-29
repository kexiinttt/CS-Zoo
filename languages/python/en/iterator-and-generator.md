# Iterable vs Iterator vs Generator

## Iterable

Objects that can be iterated can be simply understood as whether they can be called by `for`.

- `list` / `tuple` / `str` / `dict`
- file object
- generator object
- There are objects of `__iter__`

> [!IMPORTANT]
> Iterator must be Iterable, but Iterable is not necessarily Iterator.
>
> For example, `list` can be iterated by `for`, but `list` itself is not an iterator. It can be compared to C++, vector is iterable, but the iterator is `vector::iterator`.

> [!NOTE]
> `for` is actually similar to the following logic:
> ```python
> for x in data:
>     print(x)
> ```
> ```python
> it = iter(data)
> while True:
>     try:
>         x = next(it)
>         print(x)
>     except StopIteration:
>         break
> ```


## Iterator

The iterator needs to satisfy two methods: `__iter__` and `__next__`.

- `__iter__()` generally returns itself to obtain an iterator
- `__next__()` removes an element from the iterator
- Throw `StopIteration` when there is no element

```python
it = iter([1, 2, 3])
print(next(it))  # 1
print(next(it))  # 2
print(next(it))  # 3
```

> [!IMPORTANT]
> Features of Iterator: one-time consumption, move forward after use, will not automatically reset
> ```python
> it = iter([1, 2, 3])
> print(list(it))  # [1, 2, 3]
> print(list(it))  # []
> ```

## Generator

The most common generators come from functions with `yield`.

```python
def gen(n):
    for i in range(n):
        print(i)
        yield i

g = gen(3) # 不是普通函数返回值，而是一个生成器对象

next(g) # 0
next(g) # 1
next(g) # 2
next(g) # StopIteration
```

where `yield` will:
- produces a value
- Pause function execution
- Continue from where you left off next time

> [!NOTE]
> When the ordinary function is executed to `return`, it will end; when the generator function is executed to `yield`, it will be suspended and the scene will be saved, and the execution will continue next time.

> [!IMPORTANT]
> Generator saves more memory because it is **lazy calculation**, that is, invoke outputs one result at a time instead of storing all at once.
> ```python
> [x * x for x in range(1000000)] # 一次性生成全部，所需空间大小 = 1000000 * sizeof(int)
> (x * x for x in range(1000000)) # 生成器表达式，按需生成，所需空间大小 = sizeof(int)
> ```

---

# Advanced usage: `send()`, `close()` & `throw()`

* send(x): Send a value to the generator
* throw(e): Throw an exception into the generator
* close(): requires the generator to end immediately

## `send()`

```py
def gen():
    x = yield "start"
    yield f"got {x}"

g = gen()

print(next(g))        # "start"
print(g.send(10))     # "got 10"
```

Its operation process is roughly as follows:
1. Execute a little at a time according to the call of `next()`, and then stop there
2. `send(val)` returns val to the generator as the result of the yield expression and continues execution until the next yield stops.

For example, it can be considered that `next(g)` == `g.send(None)`.

## `throw()`

Throws an exception at the location where the generator is currently paused.

```py
def gen():
    try:
        yield 1
    except ValueError:
        yield "caught error"
    yield 2

g = gen()

print(next(g))                    # 1
print(g.throw(ValueError("x")))   # "caught error"
print(next(g))                    # 2
```

The process is as follows:
1. The generator pauses at yield 1
2. External call g.throw(ValueError(...))
3. It is equivalent to an exception being thrown at the current pause point inside the generator.
4. If the inner try/except can catch it, it continues to run
5. If it cannot be caught, the exception will be transmitted to the outside.

So throw() is often used for:

- Proactively notify sub-generators of errors
- Interrupt current processing flow
- Pass exceptions in coroutine/generator collaboration

## `close()`

Tells the generator to finish running and not produce any more values.

```py
def gen():
    try:
        yield 1
        yield 2
    finally:
        print("cleanup")

g = gen()
print(next(g))   # 1
g.close()        # cleanup
```

---

# Advanced usage: `yield from`

`yield from subgen` means "delegate" the output of another iterable object/generator.

```python
def subgen():
    yield 1
    yield 2

def main():
    # for x in sub():
    #     yield x
    yield from subgen()
    yield 3
```

Its biggest feature is that in addition to getting values ​​from subgen, it also delivers more complete generator collaboration semantics (send/close/throw). When collaboration semantics are called externally on the current generator, these operations are automatically forwarded to subgen.

```py
def subgen():
    try:
        x = yield 1
        print("subgen got:", x)
        y = yield 2
        print("subgen got:", y)
    finally:
        print("subgen cleanup")

def gen():
    yield from subgen()

g = gen()

print(next(g))      # 1
print(g.send(10))   # subgen got: 10, 2
g.close()           # subgen cleanup
```

---

# Advanced usage: `return`

`return` indicates the end of the generation, essentially triggering `StopIteration`. `42` will be attached to `StopIteration.value`. The normal `for` loop won't see it directly.

```python
def gen():
    yield 1
    return 42

g = gen()
print(next(g))  # 1
print(next(g))  # StopIteration(42)
```

The return value of the subgenerator becomes the result of the yield from expression.

```py
def subgen():
    yield 1
    return "done"

def gen():
    result = yield from sub()
    print("sub return:", result)
    yield 2

g = main()

print(next(g))   # 1
print(next(g))   # 打印 sub return: done，然后产出 2
```

---

# Advanced usage: Custom Iterable

* For custom iterable, `__iter__` needs to be implemented
* For custom iterator, `__next__` and `__iter__` need to be implemented

So the most common design is: for a custom iterable container, its `__iter__` returns a custom iterator:

```python
class MyIterable:
    def __init__(self, n):
        self.n = n

    def __iter__(self):
        return MyIterator(self.n)

class MyIterator:
    def __init__(self, n):
        self.n = n

    def __iter__(self):
        return self

    def __next__(self):
        if self.n <= 0:
            raise StopIteration
        current = self.n
        self.n -= 1
        return current
```

Theoretically, self can be returned directly from `__iter__` in `MyIterable`, that is, the container itself is used as both a container and an iterator. But this will bring about a problem: **cannot be traversed repeatedly**.
```py
class Bad:
    def __init__(self):
        self.data = [1, 2, 3]
        self.i = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.i >= len(self.data):
            raise StopIteration
        val = self.data[self.i]
        self.i += 1
        return val

b = Bad()
print(list(b))  # [1, 2, 3]
print(list(b))  # []
```

The problem can be solved well by separating the container and the iterator: the container is only responsible for data storage, and the iterator is only responsible for traversal. This even supports multiple iterators traversing to different locations.

---

# Iterator / Generator is not omnipotent

When the container requires random access, the iterator can only be generated once by calling `next()`. It cannot jump or look back, and there are many restrictions.

At the same time, if you need to traverse the same batch of data multiple times, it is very unwise to create an iterator each time, especially when the amount of data is small and the number of accesses is large. It is better to use list directly.
