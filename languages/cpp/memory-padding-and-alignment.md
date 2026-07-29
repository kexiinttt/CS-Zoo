# `sizeof()`

表示类型占用的字节数。不是简单把所有成员大小相加，因为还要考虑：
- 每个成员自己的对齐 alignment
- 成员之间可能插入 padding
- 结构体结尾可能还有 tail padding

**成员声明的顺序会影响整体的size**！详情参考[声明顺序对size的影响](#声明顺序对size的影响)。

---

# alignment

每种类型通常都有一个对齐要求，表示它希望放在某个特定倍数的地址上。例如：
- char 通常按 1 对齐
- short 通常按 2 对齐
- int 通常按 4 对齐
- long 通常按 8 对齐
- ptr 通常按 4 / 8 对齐 (32bit / 64bit)

**一个成员的起始 offset，通常必须是它对齐值的整数倍**。

---

# padding

为了满足下一个成员的对齐要求，编译器可能会在成员之间插入空字节，这些空字节就是 padding。
```cpp
struct A {
    char c;
    int x;
};

// 0      : char
// 1 ~ 3  : padding
// 4 ~ 7  : int
sizeof(A); // 8
```

---

# tail padding

结构体最后也可能补 padding，使得 sizeof() 是该结构体**最大对齐要求的整数倍**。比如让`A arr[10];`数组里的每个元素都能正确对齐。

> [!NOTE]
> 注意，此处是“对齐(`alignof()`)的整数倍”而不是“元素大小(`sizeof()`)的整数倍”。详情请看下面的[Example](#example-struct)。

---

# union

union 的特点是所有成员共享同一块内存，因此看最大的成员：
- union 的大小 = 最大成员大小，再向上补齐到最大对齐要求
- union 的对齐 = 所有成员对齐要求的最大值

---

# Example: Struct

```cpp
// 0... ...7 
// xxxx oooo => x: int, o: padding
// llll llll => l: long
struct A {
    int x;          // size=4, align=4
    union {
        long l;     // 8 bytes
        short s;    // 2 bytes
    } u;            // size=8, align=8
};
```

> [!NOTE]
> * `alignof(A) = max(alignof(int), alignof(uinon)) = 8`。
> * `sizeof(A)=16`，不是简单的4+8，因为u的align=8，所以它的起始地址必须是8的倍数，因此第一个int需要padding。
> * 如果更改int和union的声明顺序，那么第一个union满足要求，第二个int也满足要求，那么多余的4个bytes就变成了tail padding！

```cpp
// 0... ...7
// aaaa aaaa
// aaaa aaaa
// cooo iiii
// oooo oooo
struct B {
    A a;        // size=16, align=8
    char c;     // size=1, align=1
    int i;      // size=4, align=4
};
```

> [!NOTE]
> `sizeof(B)=24`，因为最大align是8，所以最终要tail padding到8的倍数

---

# 声明顺序对size的影响

把对齐要求大的成员放前面，对齐要求小的成员放后面
```cpp
// 0      : c
// 1 ~ 3  : padding
// 4 ~ 7  : x
// 8      : d
// 9 ~11  : tail padding
struct Bad {
    char c;
    int x;
    char d;
};

sizeof(Bad); // 12
```
```cpp
// 0 ~ 3  : x
// 4      : c
// 5      : d
// 6 ~ 7  : tail padding
struct Good {
    int x;
    char c;
    char d;
};

sizeof(Good); // 8
```

---

# class 的特殊情况

## virtual

```cpp
class A {

};

class B {
public:
    void f();
};

class C {
public:
    virtual void f();
};

sizeof(A); // 1，空类大小不为0，因为要保证不同对象有不同的地址
sizeof(B); // 1，普通成员函数不占用对象内存
sizeof(C); // 4 or 8，因为有vptr
```

## inheritance
在继承的时候，可以认为是嵌套struct的情况计算内存大小和布局。