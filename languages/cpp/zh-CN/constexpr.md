# constexpr

constexpr 代表了 表达式/变量/函数 可以在编译期计算 (compile-time evaluation)，可以简单理解为compile time const。

##### constexpr 常量
```cpp
constexpr int size = 10;
int arr[size];  // size在编译期就已知，所以可以作为数组大小
```

constexpr常量 必须通过 constexpr表达式 来初始化。其中合理的 constexpr expression 包含：
| 类型 | 示例 |
| :---: | :---: |
| 字面量 | `10` |
| constexpr 运算 | `3 + 5` |
| constexpr 变量 | `a * 2` |
| constexpr 函数返回 | `square(5)` |
| sizeof / alignof | `sizeof(int)` |


##### constexpr 函数
如果参数是编译期常量，那么函数会在编译期执行。但是其本身还是可以当作普通函数来使用的。
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

##### constexpr 对象
如果类构造函数是 constexpr，对象也可以在编译期创建。
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

因为constexpr是在编译期就计算完成，其默认具有internal linkage。所以我们可以在header中定义constexpr的常量/函数等，且并不影响ODR。如果我们想让其有external linkage，我们可以使用inline。

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
| 是否只读 | ✔ | ✔ |
| 是否编译期常量 | ❌ 不一定 | ✔ |
| 能否用于模板参数 | ❌ 不一定 | ✔ |

```cpp
const int x = rand();
constexpr double pi = 3.14;
```

---

# constexpr vs constinit vs consteval

| 关键字 | 编译期计算 | 运行期计算 | 是否 const | 作用于 |
| :---: | :---: | :---: | :---: | :---: |
| `constexpr` | ✔ | ✔ | ✔ | 函数/常量 |
| `consteval` | ✔ 必须 | ❌ | 不一定 | 函数 |
| `constinit` | ✔ 初始化 | ✔ 使用 | ❌ | 变量 |

> [!NOTE]
> * `consteval`只能作用于函数，且函数的返回并不要求一定是const
> * `constinit`只能作用于变量

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