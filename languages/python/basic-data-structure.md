# Python 变量到底是什么

Python 变量更像“名字绑定到对象”，而不是一个固定盒子里直接装值。

```python
a = [1, 2, 3]
b = a
```

这里不是复制了一份 list，而是：
- `a` 绑定到一个 list 对象
- `b` 也绑定到同一个 list 对象

因此 Python 传递的是对象引用，或说 call by object reference / call by sharing。

---

# 对象、身份、类型、值

对于一个 Python 对象，可以从三个角度看：

- 身份：`id(obj)`，类似于地址，唯一标识
- 类型：`type(obj)`
- 值：对象内容本身

## is vs ==
- `is` 比较身份
- `==` 比较值（通常调用 `__eq__`）

```python
a = [1, 2]
b = [1, 2]

print(a == b)  # True
print(a is b)  # False
```

# Mutable vs Immutable

* 不可变对象：对象创建后，内容不能原地修改
* 可变对象：对象创建后，内容可以原地修改（有点类似于c++的引用）

```py
a: str = "abc"
b: str = a      # b和a此时指向同一个不可变对象
b += "d"        # b指向一个新对象
print(a, b)     # "abc" "abcd"

x: list = [1, 2]
y: list = x     # 指向同一个可变对象
z: list = x     # 指向同一个可变对象
y += [3]        # y指向一个新对象，因为 y + [3] 创建了一个新对象
print(x, y, z)  # [1,2] [1,2,3] [1,2]
z.append(4)     # x和z还是指向同一个可变对象，此处通过z修改了对象
print(x, y, z)  # [1,2,4] [1,2,3] [1,2,4]
```

---

# 函数传参数

Python的函数传参数默认都是 **按引用** 的。

## 传入不可变对象
函数内的一些操作会导致局部对象指向新的临时对象所以不会影响caller。
```py
# 当传入时，x和y都指向同一个不可变对象
# 只不过内部操作导致局部变量y指向了新的不可变对象
# 当函数退出时局部变量y失效，外界的x指向的对象并不受影响
def f(y: int):
    y += 1

x = 1
f(x)
print(x) # 1
```

## 传入可变对象

一些操作会通过传入参数来操作caller指向的对象。
```py
# 当传入时，x和y都指向同一个可变对象
# 内部操作并未产生新的临时变量，所以list被通过y进行修改了
# 当函数退出时局部变量y失效，但是外界的x指向的对象已经被修改了
def f(y: list):
    y.append("y")

x = ["x"]
f(x)
print(x) # ["x", "y"]
```

但是也需要注意，如果内部的操作创建了新的临时对象，那么会更改传入参数的指向，因此外界caller可能也不受影响。
```py
# 当传入时，x和y都指向同一个可变对象
# 内部操作产生新的临时变量，所以y重新指向了新的对象
# 当函数退出时局部变量y失效，外界的x指向的对象并未被修改
def f(y: list):
    y += ["y"]

x = ["x"]
f(x)
print(x) # ["x"]
```

---

# ⚠️ 默认参数陷阱

Python的默认参数只在函数定义时求值一次，不是每次调用时重新创建。因此**不要设置默认参数为可变对象**，否则每次调用函数都会共享同一个可变对象。

```py
def f(x = []):
    x.append("x")
    return x

f() # ["x"]
f() # ["x", "x"]
f() # ["x", "x", "x"]
```

可以简单理解为程序做了如下的设置：
```py
__default_list_for_f_x = [] # 共享的一个可变对象

def f(x=__default_list_for_f_x):
    ...
```

正确的写法是使用一个标志来判断是否使用默认值。
```py
def g(x = None):
    if x is None:
        x = []
    x.append("x")
    return x

g() # ["x"]
g() # ["x"]
g() # ["x"]
```

---

# Data Structure

## list

`list` 是有序、可变、可重复的序列类型。

- 支持按索引 O(1) 访问
- 尾部追加平均 O(1)
- 中间插入/删除通常 O(n)
- 会在空间不够的时候扩容，类似c++中的vector
- 会分配一整块空间，局部性好，是顺序存储

## tuple

`tuple` 是有序、不可变、可重复的序列类型。

- 不可修改
- 通常比 list 更轻量
- **可作为 dict key（前提是里面元素都可哈希）**

## dict

`dict` 是键值映射结构，底层是 **哈希表**。

1. 对 key 求哈希值
2. 根据哈希值定位位置
3. 如果冲突，再按冲突解决策略处理

> [!NOTE]
> key 必须是 **可哈希的对象**，也就是：
> - 哈希值在生命周期内稳定
> - 相等对象哈希值一致

> [!IMPORTANT]
> 对于Python对象而言，只要正确定义了`__hash__`和`__eq__`就可以认为是“可哈希对象”
> ```py
> class Point:
>     def __init__(self, x, y):
>         self._x = x
>         self._y = y
> 
>     def __eq__(self, other):
>         return isinstance(other, Point) and self._x == other._x and self._y == other._y
> 
>     def __hash__(self):
>         return hash((self._x, self._y))
> ```

### 哈希冲突
两个不同 key 可能映射到相同槽位，这就是哈希冲突。常见的解决方法是“拉链法” / rehash / “开放寻址”。

* 拉链法 &rarr; 相当于每一个bucket都是一个链表，冲突了就append到链表上，当冲突特别多那么性能就逼近O(n)
* rehash &rarr; 扩充bucket数量然后重新建立hash关系
* 开放寻址 &rarr; 如果当前bucket冲突，那么就寻找下一个空闲的bucket

## set

`set` 是无序、元素唯一的集合结构，底层同样基于哈希表。

- 去重
- 快速判断成员是否存在
- 集合运算：并(`|`)、交(`&`)、差(`-`)

## str

`str` 是**不可变**的字符序列。

> [!NOTE]
> 设计成不可变是因为：（1）便于缓存和复用；（2）可哈希；（3）语义安全。
> 
> 同时因为不可变，所以每次`+=`其实都创建了新对象。

## deque

```python
from collections import deque
q = deque()
q.append(1)
q.appendleft(0)
q.popleft()
q.pop()
```

## heapq

Python 的 `heapq` 实现的是 **最小堆**。比如对于int类型，可以通过存入负数来实现最大堆。

```python
import heapq

nums = [3, 1, 5]
heapq.heapify(nums)
heapq.heappop(nums)
heapq.heappush(nums, 2)
```

heapq在排序的时候其实就是调用`__lt__`来决定谁排在顶上，因此通过override对象的`__lt__`可以实现自定义排序。

## defaultdict 和 Counter

```python
from collections import defaultdict
d = defaultdict(list)
d["a"].append(1)
```
```python
from collections import Counter
cnt = Counter("aabccc") # {"a": 2, "b": 1, "c": 3}
```

---

# sort & sorted

* sort 是inplace排序，并不是所有类型都支持
* sotred 是生成一个新的list，只要可迭代对象都可使用

```py
sorted(arr, key = lambda x: ...) # 根据每个元素的某个属性进行排序

def cmp(lhs, rhs):
    ...
sorted(arr, key=cmp_to_key(cmp)) # 实现前后相邻两个元素间的复杂逻辑进行排序
```

---

# 深拷贝和浅拷贝

* 浅拷贝 &rarr; 只拷贝外层容器，内部对象仍共享
* 深拷贝 &rarr; 递归拷贝所有层级。

```python
import copy

a = [[], []]
b = copy.copy(a)
c = copy.deepcopy(a)

b[0].append(1)
print(a, b, c)  # [[1],[]] ｜ [[1],[]] ｜ [[],[]]

c[1].append(2)
print(a, b, c)  # [[1],[]] ｜ [[1],[]] ｜ [[1],[2]]
```

> [!IMPORTANT]
> `list[:]` 是浅拷贝
