# Summary of common errors in C++

Common errors in C++ are classified by **occurrence stage**:

1. compile error
2. Linker error / linkage error
3. Runtime error (runtime error)
4. logic error
5. Undefined Behavior

---

# CompileError

This is the earliest type of error: **The code cannot even generate machine code**.

In other words, the compiler discovers something is wrong when it "translates your code".

## Common reasons

#### Syntax error
#### Type mismatch
#### No declaration / name not found
#### The header file does not include
#### Template constraints are not satisfied

```cpp
#include <vector>

int main() {
    std::vector<int> v;
    v.push_back("hello");  // const char* 不能直接当 int
}
```

## Characteristics of compilation errors

- The program cannot generate an executable file
- Errors are usually located more precisely
- Usually "the code is illegal" or "the type system does not allow it"

---

# Linker Error / Linkage Error

The compiler first compiles each `.cpp` file into the target file `.o`, and then the linker puts these target files together to form the final executable file.

If something goes wrong during the "assembly phase", it's a **linker error**.

## Undefined Reference / Unresolved External Symbol

You declare something, the compiler believes it exists, but when you link it discovers that no definition can be found.

#### The function is declared but not defined

```cpp
#include <iostream>

void foo();

int main() {
    foo();
}
```

Here `foo()` is only declared, not defined.

The compiler will think:
- "Okay, foo exists, remember first"
- At the link stage, the linker cannot find the implementation of `foo`
- Error: `undefined reference to foo`

#### The member function of the class is declared but not implemented

```cpp
class A {
public:
    void print();
};

int main() {
    A a;
    a.print(); // 没有A::print的定义
}
```

#### extern is written in the header file, but it is not defined.

```cpp
// a.h
extern int g_value;
```

If it is used somewhere in `.cpp` but is not actually written anywhere:

```cpp
int g_value = 10;
```

The link will also fail.

## Multiple Definition

The same symbol is defined multiple times.

#### Ordinary function definitions are written in header files and included by multiple cpps

```cpp
// util.h
void foo() {
}
```

If both `a.cpp` and `b.cpp` include `util.h`, then `foo()` will be defined once in multiple translation units.

The linker will report: `multiple definition of foo`

#### Global variables are defined directly in the header file

```cpp
// config.h
int g_count = 0;
```

After multiple cpp include, multiple `g_count` will be generated.

Correct approach:

```cpp
// config.h
extern int g_count;
```


```cpp
// config.cpp
int g_count = 0;
```

---

# RuntimeError

This means: the program was compiled successfully, linked successfully, and started, but it crashed during operation, or illegal behavior occurred.


## Common runtime errors

### Segmentation Fault

The program accessed an area of memory it should not have accessed.

The operating system allocates virtual memory space to each process, and not all addresses can be accessed casually.

If you visited:
- an address that doesn't belong to you at all
- Addresses without permission to access
- Address that is no longer valid

The operating system will issue an illegal memory access exception, and the program will trigger a segmentation fault.

#### Null pointer dereference

```cpp
int main() {
    int* p = nullptr;
    * p = 10;
}
```

#### Array out of bounds

```cpp
int main() {
    int arr[3] = {1, 2, 3};
    arr[10] = 5;   // 未定义行为，可能 segfault
}
```

#### use after free

```cpp
int main() {
    int* p = new int(42);
    delete p;
    * p = 100;   // 已释放后继续使用，未定义行为
}
```

### Bus Error
It's a bit like segfault, it's also an illegal memory access, but it's more biased towards:
- Illegal alignment access
- Accessing a non-existent physical address mapping

### Floating Point Exception
It’s not necessarily a floating point number problem, in many cases it’s:
- Integer division by 0
- Certain illegal arithmetic operations

### Abort / Assertion Failure

```cpp
#include <cassert>

int main() {
    int x = -1;
    assert(x > 0);
}
```

### std::terminate
Common reasons:
- Exception not caught
- Exception thrown in `noexcept` function
- `std::thread` is destructed without join/detach

---


# Undefined Behavior

Your code violates C++ rules, and the consequences are completely unwarranted. Both the compiler and the running results can be "whatever" you want.

## Common UB examples

#### Cross-border access

```cpp
int arr[3];
arr[5] = 10;
```

#### Dereference null pointer

```cpp
int* p = nullptr;
* p = 1;
```

#### use after free

```cpp
int* p = new int(1);
delete p;
std::cout << *p;
```

#### Return local variable reference

```cpp
int& foo() {
    int x = 10;
    return x;   // UB
}
```

#### Signed integer overflow
```cpp
int max(int a, int b) {
    return a < b ? a : b;
}
```