# What exactly are Python variables?

Python variables are more like "names bound to objects" rather than directly holding values in a fixed box.

```python
a = [1, 2, 3]
b = a
```

This is not a copy of the list, but:
- `a` is bound to a list object
- `b` is also bound to the same list object

So what Python passes is an object reference, or call by object reference / call by sharing.

---

# Object, identity, type, value

For a Python object, you can look at it from three perspectives:

- Identity: `id(obj)`, similar to address, unique identifier
- Type: `type(obj)`
- Value: the object content itself

## is vs ==
- `is` compare identities
- `==` comparison value (usually called `__eq__`)

```python
a = [1, 2]
b = [1, 2]

print(a == b)  # True
print(a is b)  # False
```

# Mutable vs Immutable

* Immutable objects: After the object is created, the content cannot be modified in place.
* Mutable object: After the object is created, the content can be modified in place (somewhat similar to C++ references)

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

# Function passing parameters

Python's function parameters are passed by reference by default.

## Pass in the immutable object
Some operations within the function will cause the local object to point to a new temporary object so it will not affect the caller.
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

## Pass in the mutable object

Some operations will operate on the object pointed to by the caller by passing in parameters.
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

However, it should also be noted that if the internal operation creates a new temporary object, the direction of the incoming parameters will be changed, so the external caller may not be affected.
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

# ⚠️Default parameter trap

Python's default parameters are only evaluated once when the function is defined and are not recreated each time it is called. Therefore **Don't set the default parameter to a mutable object**, otherwise the same mutable object will be shared every time the function is called.

```py
def f(x = []):
    x.append("x")
    return x

f() # ["x"]
f() # ["x", "x"]
f() # ["x", "x", "x"]
```

It can be simply understood that the program has the following settings:
```py
__default_list_for_f_x = [] # 共享的一个可变对象

def f(x=__default_list_for_f_x):
    ...
```

The correct way to write it is to use a flag to determine whether to use the default value.
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

`list` is an ordered, variable, repeatable sequence type.

- Supports O(1) access by index
- Tail append average O(1)
- Intermediate insertion/deletion is usually O(n)
- Will expand when there is not enough space, similar to vector in C++
- A whole block of space will be allocated, with good locality and sequential storage.

## tuple

`tuple` is an ordered, immutable, repeatable sequence type.

- cannot be modified
- Usually more lightweight than list
- **Can be used as dict key (provided that all elements inside can be hashed)**

## dict

`dict` is a key-value mapping structure, and the bottom layer is a **hash table**.

1. Find the hash value of key
2. Locate location based on hash value
3. If there is a conflict, handle it according to the conflict resolution strategy

> [!NOTE]
> key must be a **hashable object**, that is:
> - The hash value is stable over its lifetime
> - Equality objects have the same hash value

> [!IMPORTANT]
> For Python objects, as long as `__hash__` and `__eq__` are correctly defined, they can be considered "hashable objects"
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

### Hash collision
Two different keys may be mapped to the same slot, which is a hash conflict. Common solutions are "zippering" / rehash / "open addressing".

* Zipper method → Equivalent to each bucket being a linked list. If there is a conflict, it is appended to the linked list. When there are too many conflicts, the performance will be close to O(n).
* rehash → expand the number of buckets and re-establish the hash relationship
* Open addressing → If the current bucket conflicts, then look for the next free bucket

## set

`set` is an unordered, element-only collection structure, and the bottom layer is also based on a hash table.

- Remove duplicates
- Quickly determine whether a member exists
- Set operations: union (`|`), intersection (`&`), difference (`-`)

## str

`str` is an **immutable** character sequence.

> [!NOTE]
> Designed to be immutable because: (1) Easy to cache and reuse; (2) Hashable; (3) Semantic security.
>
> At the same time, because it is immutable, a new object is actually created every time `+=`.

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

Python's `heapq` implements **minimal heap**. For example, for the int type, the maximum heap can be achieved by storing negative numbers.

```python
import heapq

nums = [3, 1, 5]
heapq.heapify(nums)
heapq.heappop(nums)
heapq.heappush(nums, 2)
```

When heapq sorts, it actually calls `__lt__` to determine who is ranked at the top. Therefore, custom sorting can be achieved by overriding the object's `__lt__`.

## defaultdict and Counter

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

* sort is inplace sorting, not all types support it
* sotred is to generate a new list, which can be used as long as iterable objects

```py
sorted(arr, key = lambda x: ...) # 根据每个元素的某个属性进行排序

def cmp(lhs, rhs):
    ...
sorted(arr, key=cmp_to_key(cmp)) # 实现前后相邻两个元素间的复杂逻辑进行排序
```

---

# Deep copy and shallow copy

* Shallow copy → only copies the outer container, the internal objects are still shared
* Deep copy → Recursively copy all levels.

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
> `list[:]` is a shallow copy
