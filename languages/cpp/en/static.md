What is #`static`

`static` is a keyword in C++, but its meaning is not exactly the same in different locations.  

- **Control life cycle**: Let the object not be destroyed when the scope ends, but will exist until the end of the program.
- **Control visibility/ownership**: Make a name visible only in the current file, or make a member belong to the "class itself" rather than an object.

Generally speaking, `static` has the following uses:
- Modify **local variables**
- Modify **global variables**
- Modify **function**
- Modify **member variables of the class**
- Modify **member functions of the class**

`static` variables are stored in the "static storage area", not the heap or the stack.

---

# static local variables

When `static` modifies local variables inside a function:

- The scope is still the current function / inside the current code block → cannot be accessed outside the function
- **The life cycle becomes the entire program running period** → will not be destroyed because the function ends
- **Initialize once** → will not be recreated every time the function is called

That is to say:

- Ordinary local variables: created every time you enter the function and destroyed when you leave the function
- `static` local variable: created when entering the function for the first time and retained thereafter

```cpp
#include <iostream>
using namespace std;

void test() {
    static int s_cnt = 0;
    int cnt = 0;
    s_cnt++;
    cnt++;
    cout << s_cnt << ", " << cnt << endl;
}

int main() {
    test(); // 1, 1
    test(); // 2, 1
    test(); // 3, 1
    return 0;
}
```
---

# static global variables

When `static` modifies a global variable, its focus is no longer on the life cycle (because it is already global, so the life cycle is the entire program), but on:

- **Link attribute becomes internal linkage** → Can only be used in **current.cpp** and cannot be accessed by other `.cpp` files through `extern`

```cpp
// file1.cpp
static int g_value = 100;
```
```cpp
// file2.cpp
extern int g_value;  // 不能正确链接到 file1.cpp 里的 static 全局变量
```

Its main function is to limit name pollution and avoid misuse or conflicts between different files. But now the more common way of writing is to use anonymous namespace. :

---

# static function

Like limiting global variables, it mainly limits the link scope and is only visible within this file.

```cpp
// file1.cpp
static void helper() {
}
```


```cpp
// file2.cpp
#include <file1>
helper(); // ❌
```

---

# static class member variables

If `static` modifies a **class member variable**, then this variable:

- **belongs to the class itself**
- does not belong to a specific object
- All objects share the same data

```cpp
#include <iostream>
using namespace std;

class Student {
public:
    Student() {
        ++count;
    }

    static int count;
};

int Student::count = 0;

int main() {
    Student s1;
    Student s2;
    Student s3;

    cout << Student::count << endl; // 3
    return 0;
}
```

## Initialize static member variables

### Ordinary static members

Usually only declarations can be made within a class, not definitions. It needs to be defined separately outside the class, and it can also be initialized outside the class.
```cpp
class A {
public:
    static int x;
};

int A::x = 10;
```

### static const static constant
Can be defined and initialized within a class, this is a special rule. After all, const must be initialized when defined (or initialized using a member initializer list), so there is no other choice.
```cpp
class A {
public:
    static const int x = 10;
};
```

### static constexpr members

It can be initialized within the class, which is the recommended way of writing in modern C++.
```cpp
class A {
public:
    static constexpr int x = 10;
};
```

### inline static
Can be initialized within the class.
```cpp
class A {
public:
    inline static int x = 10;
};
```

---

# static class member function

If `static` modifies a member function of a class, then it is also a function that "belongs to the class itself", not a method of an object.

- Can be called directly through the class name
- **No `this` pointer**
- Can only access:
  - `static` member variable
  - `static` member function
  - Other content unrelated to the object
- Ordinary member variables and ordinary member functions cannot be accessed directly because those require concrete objects

```cpp
#include <iostream>
using namespace std;

class Student {
public:
    static int count;

    static void printCount() {
        cout << count << endl;
    }
};

int Student::count = 0;

int main() {
    Student::printCount();
    return 0;
}
```


## How to access object members

If you really want to access the data of an object in the `static` member function, you need to explicitly pass in the object:

```cpp
class A {
public:
    int x = 10;

    static void print(const A& a) {
        cout << a.x << endl;
    }
};
```

---

# static initialization problem

* Ordinary static / global variables → This type of object is usually initialized during the program startup phase and destroyed at the end of the program
* Local static → initialized when the definition statement is executed for the first time
* Class static member variable initialization → outside class / `inline static` / `static constexpr` / `const static`