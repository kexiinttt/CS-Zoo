# constexpr

constexpr represents that expressions/variables/functions can be evaluated at compile time (compile-time evaluation), which can be simply understood as compile time const.

##### constexpr constant
```cpp
constexpr int size = 10;
int arr[size];  // size在编译期就已知，所以可以作为数组大小
```

constexpr constants must be initialized with constexpr expressions. Reasonable constexpr expressions include:
| Type | Example |
| :---: | :---: |
| Literal | `10` |
| constexpr operation | `3 + 5` |
| constexpr variable | `a * 2` |
| constexpr function returns | `square(5)` |
| sizeof / alignof | `sizeof(int)` |


##### constexpr function
If the parameter is a compile-time constant, the function will be executed at compile time. But it can still be used as an ordinary function.
```cpp
constexpr int square(int x) { return x * x; };

// ✅ 编译期直接处理成 constexpr int x = 25
constexpr int x = square(5);

// ✅ 运行期进行计算，因为var不是 **编译期常量**
int var = 5;
int y = square(var);

// 🤔 可能是编译期处理，但是不保证
int z = square(5);

// ❌ 因为var不是 **编译期常量**
// constexpr int y = square(var);
```

##### constexpr object
Objects can also be created at compile time if the class constructor is constexpr.
```cpp
struct Point
{
    int x, y;
    constexpr Point(int a, int b) : x(a), y(b) {}
};

constexpr Point p(1, 2);
```

---

# internal linkage

Because constexpr is calculated at compile time, it has internal linkage by default. So we can define constexpr constants/functions, etc. in the header without affecting ODR. If we want it to have external linkage, we can use inline.

```cpp
constexpr int internal_val = 10;
inline constexpr int external_val = 10;
```

---

# template
```cpp
template<int N>
struct Array
{
    int data[N];
};

constexpr int size = 10;

Array<size> arr;
```
---

# const vs constexpr

| | const | constexpr |
| :---: | :---: | :---: |
| Whether to read only | ✔ | ✔ |
| Whether it is a compile-time constant | ❌ Not necessarily | ✔ |
| Can it be used as template parameter | ❌ Not necessarily | ✔ |

```cpp
const int x = rand();
constexpr double pi = 3.14;
```

---

# constexpr vs constinit vs consteval

| Keywords | Compile-time calculation | Run-time calculation | Whether const | Acts on |
| :---: | :---: | :---: | :---: | :---: |
| `constexpr` | ✔ | ✔ | ✔ | Function/Constant |
| `consteval` | ✔ Required | ❌ | Not necessarily | Function |
| `constinit` | ✔ Initialization | ✔ Usage | ❌ | Variables |

> [!NOTE]
> * `consteval` can only be used on functions, and the return of the function does not have to be const.
> * `constinit` can only act on variables

```cpp
consteval int square(int x) { return x * x; }

int var = 5;
int x = square(var); // ❌，var是runtime变量，而consteval必须编译期计算

int y = square(5); // ✅，且一定是编译期计算并替换成 int y = 25
```


```cpp
// 保证全局变量是编译期静态初始化的，而不是运行期动态赋值
constinit int global_counter = 0;
```