# Iterable vs Iterator vs Generator

## Iterable

可以被迭代的对象，可以简单理解为能不能被`for`调用。

- `list` / `tuple` / `str` / `dict`
- 文件对象
- 生成器对象
- 有`__iter__`的对象

> [!IMPORTANT]
> Iterator 一定是 Iterable 的，但是 Iterable 的不一定是 Iterator。
>
> 比如`list`可以被`for`迭代，但是`list`本身不是iterator。可以类比C++，vector是可迭代的，但是迭代器是`vector::iterator`。

> [!NOTE]
> `for` 实际上是类似如下的逻辑：
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

迭代器需要满足两个方法：`__iter__`和`__next__`。

- `__iter__()` 一般返回自身获取一个迭代器
- `__next__()` 从迭代器里取下一个元素
- 没有元素时抛 `StopIteration`

```python
it = iter([1, 2, 3])
print(next(it))  # 1
print(next(it))  # 2
print(next(it))  # 3
```

> [!IMPORTANT]
> Iterator 的特点：一次性消费，用过就往前走，不会自动重置
> ```python
> it = iter([1, 2, 3])
> print(list(it))  # [1, 2, 3]
> print(list(it))  # []
> ```

## Generator

最常见的生成器来自带 `yield` 的函数。

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

其中 `yield` 会：
- 产出一个值
- 暂停函数执行
- 下次再从暂停的位置继续

> [!NOTE]
> 普通函数执行到 `return` 就结束；生成器函数执行到 `yield` 会挂起并保存现场，下次继续执行。

> [!IMPORTANT]
> Generator 更省内存，因为它是**惰性计算**，即invoke一次输出一个结果，而不用一次性存储全部。
> ```python
> [x * x for x in range(1000000)] # 一次性生成全部，所需空间大小 = 1000000 * sizeof(int)
> (x * x for x in range(1000000)) # 生成器表达式，按需生成，所需空间大小 = sizeof(int)
> ```

---

# 高级用法：`send()`, `close()` & `throw()`

* send(x)：向生成器里发送一个值
* throw(e)：向生成器里抛一个异常
* close()：要求生成器立刻结束

## `send()`

```py
def gen():
    x = yield "start"
    yield f"got {x}"

g = gen()

print(next(g))        # "start"
print(g.send(10))     # "got 10"
```

其运行流程大概是：
1. 根据`next()`的调用每次执行一点，然后停在该处
2. `send(val)` 将 val 作为 yield 表达式的结果送回生成器并继续执行直到下一个 yield 停住

比如可以认为`next(g)` == `g.send(None)`。

## `throw()`

在生成器当前暂停的位置，抛出一个异常。

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

其流程如下：
1. 生成器暂停在 yield 1 那里
2. 外部调用 g.throw(ValueError(...))
3. 相当于异常在生成器内部当前暂停点被抛出来
4. 如果内部 try/except 能接住，它就继续运行
5. 如果接不住，异常就会传到外部

所以 throw() 常用于：

- 主动通知子生成器出错
- 中断当前处理流程
- 在协程/生成器协作里传递异常

## `close()`

通知生成器结束运行，不要再产出值了。

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

# 高级用法：`yield from`

`yield from subgen` 表示把另一个可迭代对象/生成器的产出“委托”出去。

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

它最大的特点在于除了能从subgen获取值，还会传递更完整的生成器协作语义(send / close / throw)。当外部对当前生成器调用协作语义，这些操作会自动转发给subgen。

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

# 高级用法：`return`

`return` 表示生成结束，本质上会触发 `StopIteration`。`42` 会附着在 `StopIteration.value` 上。普通 `for` 循环不会直接看到它。

```python
def gen():
    yield 1
    return 42

g = gen()
print(next(g))  # 1
print(next(g))  # StopIteration(42)
```

子生成器的 return value，会成为 yield from 表达式的结果。

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

# 高级用法：自定义 Iterable

* 对于自定义 iterable，需要实现 `__iter__`
* 对于自定义 iterator，需要实现`__next__`和`__iter__`

因此最常见的设计是：对于自定义 iterable container，其`__iter__` 返回一个自定义 iterator：

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

理论上可以直接在`MyIterable`中的`__iter__`返回self，即把容器本身即当作容器又当作迭代器。但是这样会带来一个问题：**无法重复遍历**。
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

通过把容器和迭代器分离能够很好的解决问题：**容器只负责数据储存，迭代器只负责遍历**。这样甚至可以支持多个迭代器遍历到不同的位置。

---

# Iterator / Generator 不是万能的

当容器需要随机访问的时候，iterator只能通过调用 `next()` 一次一次生成，不能跳跃，不能回头，有很多限制。

同时，如果需要反复多次遍历同一批数据，每一次都创建一个iterator非常不明智，尤其是数据量小且访问次数大的情况下。不如直接用list。
