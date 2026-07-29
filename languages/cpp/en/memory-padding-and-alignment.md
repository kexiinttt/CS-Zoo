# `sizeof()`

Indicates the number of bytes occupied by the type. It is not simply adding up the sizes of all members, because we also need to consider:
- Each member's own alignment
- padding may be inserted between members
- There may be tail padding at the end of the structure

**The order of member declarations affects the overall size**! See [how declaration order affects size](#how-declaration-order-affects-size) for details.

---

# alignment

Each type usually has an alignment requirement, indicating that it wants to be placed at a certain multiple of addresses. For example:
- char is usually aligned by 1
- short is usually aligned by 2
- int is usually aligned by 4
- long is usually aligned by 8
- ptr is usually 4 / 8 aligned (32bit / 64bit)

**The starting offset of a member must usually be an integer multiple of its alignment value**.

---

# padding

In order to meet the alignment requirements of the next member, the compiler may insert null bytes between members. These null bytes are padding.
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

The structure may also be padded at the end, so that sizeof() is an integer multiple of the maximum alignment requirement of the structure. For example, let each element in the `A arr[10];` array be correctly aligned.

> [!NOTE]
> Note that this is "an integer multiple of alignment (`alignof()`)" rather than "an integer multiple of element size (`sizeof()`)". See [Example](#example-struct) below for details.

---

# union

The characteristic of union is that all members share the same memory, so look at the largest member:
- The size of the union = the maximum member size, and then padded upward to the maximum alignment requirement
- Alignment of union = maximum alignment requirement of all members

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
> *`alignof(A) = max(alignof(int), alignof(uinon)) = 8`.
> * `sizeof(A)=16`, not simply 4+8, because u's align=8, so its starting address must be a multiple of 8, so the first int requires padding.
> * If you change the declaration order of int and union, then the first union meets the requirements, and the second int also meets the requirements, then the extra 4 bytes become tail padding!

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
> `sizeof(B)=24`, because the maximum align is 8, so the tail padding must be a multiple of 8 in the end.

---

# The impact of declaration order on size

Place members with greater alignment requirements in the front and members with smaller alignment requirements in the back.
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

# Special case of class

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
When inheriting, it can be considered as a case of nested struct to calculate the memory size and layout.
