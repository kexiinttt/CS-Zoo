  # **C++ Prime Plus 教程**
  基于《C++ Prime Plus》 6th
  
  <a href="https://sm.ms/image/sahKZY69UGSquc2" target="_blank"><img src="https://i.loli.net/2020/10/17/sahKZY69UGSquc2.jpg" width="300" height="450" div align=center></a>
  
  
- [**C++ Prime Plus 教程**](#c-prime-plus-教程)
- [**基本概念**](#基本概念)
  - [名称空间](#名称空间)
  - [泛型 Vs 面向对象OOP](#泛型-vs-面向对象oop)
  - [类 Vs 对象](#类-vs-对象)
  - [函数原型 Vs 函数定义](#函数原型-vs-函数定义)
  - [初始化](#初始化)
  - [强制类型转换](#强制类型转换)
  - [数组](#数组)
  - [String类](#string类)
  - [结构体Structure](#结构体structure)
  - [联合Union Vs 枚举Enum](#联合union-vs-枚举enum)
  - [动态联编 Vs 静态联编](#动态联编-vs-静态联编)
  - [堆Heap Vs 栈Stack](#堆heap-vs-栈stack)
  - [vector、array和数组](#vectorarray和数组)
  - [基于范围的for循环](#基于范围的for循环)
- [**指针**](#指针)
  - [创建/消除指针](#创建消除指针)
  - [指针 Vs 数组名](#指针-vs-数组名)
  - [指针数组 Vs 数组指针](#指针数组-vs-数组指针)
  - [浅复制 Vs 深复制](#浅复制-vs-深复制)
  - [const指针](#const指针)
- [**函数**](#函数)
  - [函数中传递数组](#函数中传递数组)
  - [返回指针](#返回指针)
  - [const函数](#const函数)
  - [函数指针](#函数指针)
  - [内联函数](#内联函数)
  - [默认参数](#默认参数)
  - [函数重载(多态)](#函数重载多态)
  - [函数模板](#函数模板)
        - [显示具体化](#显示具体化)
        - [实例化](#实例化)
  - [decltype](#decltype)
- [**引用\&**](#引用)
- [**内存模型和名称空间**](#内存模型和名称空间)
  - [编译](#编译)
  - [作用域和链接性](#作用域和链接性)
  - [名称空间](#名称空间-1)
- [**类Class**](#类class)
  - [类定义](#类定义)
  - [构造函数](#构造函数)
  - [析构函数](#析构函数)
  - [const成员函数](#const成员函数)
  - [this指针](#this指针)
  - [运算符重载](#运算符重载)
  - [友元函数](#友元函数)
  - [类的自动类型转换和强制类型转换](#类的自动类型转换和强制类型转换)
  - [动态内存分配](#动态内存分配)
        - [复制构造函数](#复制构造函数)
          - [浅复制](#浅复制)
          - [深复制](#深复制)
          - [赋值函数 Vs 复制构造函数 Vs 构造函数](#赋值函数-vs-复制构造函数-vs-构造函数)
  - [静态成员函数](#静态成员函数)
  - [成员初始化列表](#成员初始化列表)
  - [继承](#继承)
        - [派生类和基类的特殊关系:](#派生类和基类的特殊关系)
        - [虚函数](#虚函数)
        - [隐藏](#隐藏)
        - [public、private、protected继承](#publicprivateprotected继承)
  - [抽象基类(ABC)](#抽象基类abc)
  - [继承和动态内存分配](#继承和动态内存分配)
        - [派生类不使用new](#派生类不使用new)
        - [派生类使用new](#派生类使用new)
        - [友元](#友元)
  - [类设计小结](#类设计小结)
        - [三种默认函数](#三种默认函数)
        - [按值传递和按引用传递](#按值传递和按引用传递)
        - [返回引用和返回值](#返回引用和返回值)
        - [继承](#继承-1)
        - [类函数](#类函数)
- [**代码复用**](#代码复用)
  - [valarray类](#valarray类)
  - [包含 Vs 私有继承](#包含-vs-私有继承)
  - [多重继承(MI)](#多重继承mi)
        - [虚基类](#虚基类)
  - [类模板](#类模板)
        - [表达式参数](#表达式参数)
        - [模板递归（嵌套）](#模板递归嵌套)
        - [模板具体化](#模板具体化)
        - [将模板作为参数](#将模板作为参数)
        - [模板类和友元](#模板类和友元)
        - [模板别名](#模板别名)
- [**友元和异常**](#友元和异常)
  - [友元](#友元-1)
  - [嵌套类](#嵌套类)
  - [异常](#异常)
        - [abort()](#abort)
        - [throw](#throw)
        - [try](#try)
        - [catch](#catch)
        - [将对象用作异常类型](#将对象用作异常类型)
        - [栈解退](#栈解退)
        - [其他属性](#其他属性)
        - [exception类](#exception类)
  - [RTTI（运行阶段类型识别）](#rtti运行阶段类型识别)
        - [dynamic\_cast\<\>](#dynamic_cast)
        - [typeid和type\_info](#typeid和type_info)
  - [static\_cast、const\_cast和reiterpret\_cast](#static_castconst_cast和reiterpret_cast)
- [**智能指针模板类**](#智能指针模板类)
- [**STL类（基础）**](#stl类基础)
  - [vector模板类](#vector模板类)
  - [泛型编程](#泛型编程)
        - [容器](#容器)
        - [迭代器](#迭代器)
        - [函数对象(函数符)](#函数对象函数符)
          - [函数符概念](#函数符概念)
          - [预定义函数符](#预定义函数符)
          - [自适应函数符](#自适应函数符)
          - [函数适配器](#函数适配器)
        - [算法](#算法)
- [**C++11新标准**](#c11新标准)
  - [右值引用](#右值引用)
  - [移动语义](#移动语义)
        - [移动语义](#移动语义-1)
        - [std::move()](#stdmove)
        - [C++11新的类功能](#c11新的类功能)
          - [delete和default关键字](#delete和default关键字)
          - [委托构造函数](#委托构造函数)
          - [继承构造函数](#继承构造函数)
          - [override \& final](#override--final)
  - [Lambda](#lambda)
  - [包装器](#包装器)
  - [可变参数模板](#可变参数模板)


# **基本概念**
## 名称空间
* 使用名称空间可以将不同文件中同名的函数，变量区分开来，以免冲突
* 在不同的地方使用该定义，具有不同的生命周期

*(注意：如果使用头文件，在include<xxx.h>时不需要使用名称空间，只有使用include<xxx>时需要使用)*

```c++
using namespace std
// using std::cout
```
## 泛型 Vs 面向对象OOP
* 泛型——忽略不同数据类型之间的差异
* OOP——对某些数据定义统一操作方式
## 类 Vs 对象
* 类Class——定义某些数据和操作
* 对象Object——创建某一个Class的实体
## 函数原型 Vs 函数定义
- 原型——只针对函数的返回值、形式参数等进行定义
- 定义——针对函数的具体实现
```c++
int max(int a, int b);  # 原型
int max(int a, int b)   # 定义
{
  return a>b?a:b;
}
```
## 初始化

使用列表初始化能很好地避免隐式类型转换，即数据无法被截断，如 float->int

```c++
int a = 10;  // traditional init
int b(10);   // C++ type init
int c = {10};// list init
```
## 强制类型转换
```c++
(type) value;
type (value);
static_cast<tpye> (value)
```
## 数组
* 只能在定义的时候对数组初始化，且不能将一个数组赋值给另一个数组
* 可以使用列表初始化
* 只对一部分初始化将会将剩余部分置为0
* 定义数组的时候，数组元素个数必须是常量
```c++
int list[5] = {1, 2, 3};  // list = 1, 2, 3, 0, 0
// int list[5]; list = {1,2,3};  是错误的方式
// int list2[5] = list;          也是错误的方式
```
* 字符串数组中不要忘记最末位有个'\0'也是算入个数的
## String类
- 不用指定长度
- 可以相互赋值
- 可以使用"+"进行拼接
- 可以使用"=="判断是否相同

*(注意：C风格字符串即char[]不能使用"=="判断是否相同，而要用strcmp()!!)*

## 结构体Structure
* 不像C语言，不使用typedef也可以省去structure关键字
* 可以使用列表初始化{}
* 可以将结构体相互赋值
## 联合Union Vs 枚举Enum
- union——存储不同数据类型，且每个类型只有一个
```c++
union Brand
{
  int id_int;
  string id_string;
}
Brand brand;
std::cout<<brand.id_int;
```
- enum
```c++
enum color{red, orange, blue}; // red:0 orange:1 blue:2
// red = 100; 不能对enum进行赋值
// color(4);  不能越界
enum size{small=1, medium=10, large=100};
// size(50);  不报错，但是没有意义。只要idx在定义的[min, max]都不会报错
enum name{tom, alice=0, bob=1};
```
## 动态联编 Vs 静态联编
- 动态联编——在程序运行后才进行内存分配等操作(new操作，虚函数等)
- 静态联编——在程序编译时即进行内存分配
## 堆Heap Vs 栈Stack
- heap——为new的内存来源
- stack——为用户定义的内存来源
## vector、array和数组
- vector == 整合了new&delete的可边长数组
- array == 可直接赋值的定长数组
```c++
vector<int> v(n);  // n 可以是cin>>n;动态分配
array<int, 10> a;  // 若还存在array<int, 10> b;则可以a=b
int i[10];
```
## 基于范围的for循环
```c++
double points[5] = {0.1, 1.2, 6.5, 7.2, 9.0};
for(double x: points)  // 使用该方法只能读出数据 
  cout<< x<<endl;
for(double &y: points)  // 使用&可以修改数据
  y = y*0.1;
```
:top:[Back to Top](#c基础知识):top:

---


# **指针**
## 创建/消除指针
* *delete只能消除用new创建的指针!!*
```c++
type* pointer = new type;
type i;
type* pi = &i;
delete pointer;    // ok
delete pi;         // not allowed
```
## 指针 Vs 数组名
- 指针是变量(未使用const时)，而数组名是常量，所以指针可以+1操作
- int * p;那么p+1为在p的地址上移动sizeof(int*)的距离
- 数组名是第一个元素的地址，而对数组取地址得到的是整个数组的地址
```c++
short tell[10];
cout<< tell<< endl;  // display the addr of tell[0], maybe 0xFF
cout<< &tell<< endl; // display the addr of tell, maybe 0xFF
cout<< tell+1<<endl;  // 地址增加2
cout<< &tell+1<<endl; // 地址增加2*10
cout<< sizeof(&tell[0])<<endl;    // display 2
cout<< sizeof(tell)<<endl;        // display 2*10
```
## 指针数组 Vs 数组指针
- 指针数组——指针类型的数组
- 数组指针——指向数组的指针
```c++
int* p[10];    // 指针数组，p[10]为数组，内部为int*指针
int (*p)[10];  // 数组指针，*p作为一个指针，指向int[10]
```
## 浅复制 Vs 深复制
* *不要用赋值运算给数组输入数据，而是用strcpy()，但是string可以用=*
```c++
string s1 = "hello C++";
char *p = new char(strlen(s1)+1);
// p = s1; 此时为深复制，即p与s1指向同一个地方，此时创建的原来new的区域失去指针，且更改p或s1会同时更改另一个
// strcpy(p, s1); 此时为浅复制，p与s1地址不同，但是内容相同，修改s1不会更改p
```
## const指针
* *不能将const地址赋给非const指针*
```c++
int a = 10;
const int * p = &a;  // 此时，*p为const，不能通过p改变a的值，但是可以直接改变a的值，如a = 11(√)； *p = 11(×)
int* const t = &a;   // 此时，*t可以更改a的值，但是不能指向别的地址
```

:top:[Back to Top](#c基础知识):top:

---


# **函数**
## 函数中传递数组
* 数组名 == 数组第一个元素的地址
```c++
int max(int arr[]);    // 此处使用 int arr[] == int* arr
max(myArr, ArrSize);   // 调用时，只要传入数组名，不需要取地址
```
- 推荐显示传递数组的元素个数，而不是在函数中再去计算个数，因为传入的只有第一个元素的地址，无法得到总个数
```plain
int max(int arr[][4], int arrSize);
```
- 传递二维数组时，要指明列数，行数可以不指明。而且此时，列数传去的参数是可以直接当做数据使用的。
- 传递常规变量时，修改的是临时变量；但传递数组时，修改的是原数据
## 返回指针
```c++
int main()
{
  char * p = build(...);
  cout<< p<<endl;
  delete [] p;    // 在build中使用了new[]，所以需要在此处使用delete[]
}
char * build(...)
{
  char * ps = new char[n+1];
  ...    
  return ps;  //退出时，ps作为局部变量被消除，但是ps指向的空间是new的，所以仍然存在，main中通过返回的地址，
}
```
## const函数
```c++
int max(const int arr[]); // 此时该函数只能读arr而不能修改 
```
## 函数指针
- 在函数声明中，将函数名更换为(*函数名)即可，然后再代码中将别的函数名替换进来即可
- 对于单个函数指针，如果定义其类型过于复杂，可以使用auto自动推断其类型
```c++
int max(int arr[]);
int (*p)(int arr[]);
p = max;
int a = max(arr);
int b = (*p)(arr); 
```
- 使用typedef来简化代码
```c++
typedef double* (*p_func)(double);
p_func p1 = f;
double* (*p2)(double) = f;
```
## 内联函数
- 一般直接将定义书写在原本声明处，并加上inline关键字
- 不需要在不同地址间互相跳跃，直接将内容增加到调用的代码内存空间后面
```c++
inline int square(int a) {return a*a;}  //一般内联函数都比较短小
```
## 默认参数
- 只需要在函数声明的时候写出默认即可，函数定义时可以不用管
- 默认参数只能位于最右侧
```c++
int max(int a, int b, int c=0);
int max(int a, int b, int c) {...} 
```
## 函数重载(多态)
- 多个同名函数起到不同作用
- 函数重载的关键是"函数特征标"（参数引用和参数本身是同样的类型！！）
- 函数的返回值可以不同，但是特征标一定要不同！
- 名称修饰(名称矫正)——在编译阶段，针对函数重载的不同格式进行不同的编码，以区分
```c++
int max(int, int);
double max(double, double);
// int max(int &, int &); 并不是重载，和第一个函数冲突 
```
- 在类中，是否const是不同的特征标
```c++
char& operator [](int i);
char& operator [](int i) const;  // 这两个函数是不同的！！
```
## 函数模板
* 对于多种不同类型的数据，进行同样的操作，则使用模板
```c++
template <typename type>    // 当传入的a,b为int时，type==int
void swap(type& a, type& b)
{
  type temp;
  temp = a;
  a = b;
  b = temp;
}
```
* 模板也可以进行重载，只要其函数特征标不同即可
* 模板可以含有多种数据类型
```c++
template <typename t1, typename t2>
void add(t1 a, t2 b) {...}
```
##### 显示具体化
* 当模板无法完成某一类型的操作时，可以使用显示具体化，对某一个类型单独设计操作
```c++
// 在函数声明和定义时，前方加入template<>，并在函数名后加入<类型>即可
template<> void Swap<job>(job& a, job& b);
// 可以缩写成 template<> void Swap(job&, job&);
```
* 优先级：非模板函数 > 显示具体化 > 模板函数
##### 实例化
* 实例化的函数定义仍然满足同名函数的模板，只不过是实例化了对应的参数类型
* 具体化的函数定义是不满足函数模板的，需要进行重载
* 实例化不需要定义，只需要声明；具体化需要声明+定义
```c++
template<> void Swap(job&, job&);  // 具体化
template void Swap(int&, int&);    // 实例化
```
## decltype
* 在使用多数据类型模板中，可能无法确定数据类型
* 使用decltype(x+y) xpy = x + y;即可完成不同类型x,y相加
* 使用decltype((xx)) r = xx;则r为xx类型的引用
* 如果无法确定返回值，则可以使用后置返回(auto func_name(paras) -> type)
```c++
template <typename t1, typename t2>
auto add(t1 a, t2 b) -> decltype(a+b)
{
  return a + b;
}
```

:top:[Back to Top](#c基础知识):top:

---


# **引用&**
* *被引用的两个变量会指向同样的地址，值也是共享的*
* *必须在声明时就初始化*
```c++
int rat;
int & rodent = rat;
// "int & rodent; rodent=rat;" is invalid
```
+ 可以通过初始化声明来设置引用，如果是const，则不能更改引用；如果不是const，则可以通过直接对引用或者引用内容进行修改
```c++
// 最后p指向bunnies，而rodent仍为rats的引用，为100
int rats = 100;
int * p = &rats;
int & rodent = *p; // rodent与*p共用地址和内容，此后p改变并不影响rodent
int bunnies = 50;
p = &bunnies;

rodent = *p; // 此时rodent和rats都是50，但是此时rodent和rats的地址仍然没变
```
+ 可以通过指针或者引用来实现函数对main中数据的改变
+ *在返回引用的时候要注意，千万不能返回一个临时变量！要么返回new过的，要么返回传入的引用参数*
+ 由于返回的是引用，所以可以作为左值和右值，如果不想函数作为左值，则需要在返回值处加入const
+ *数组->指针； 结构体->指针/引用； 类->引用*

:top:[Back to Top](#c基础知识):top:

---


# **内存模型和名称空间**
## 编译
* 使用自己编写的头文件时，用""，用系统自带的头文件时，用<>
* 头文件一般包括：#define、函数声明、内联函数、const常量、结构体声明、类声明
* 在编写头文件时保留好习惯
```c++
#ifndef NAME_H_
#define NAME_H_
// place include file contents here
#endif
```
- 源文件(.cpp) ---编译---> 目标文件(.obj) ---链接---> 可执行文件(.exe)
    - 编译：将C++代码翻译成机器代码
    - 链接：将目标文件、操作系统启动代码和库文件整合成可执行文件
## 作用域和链接性
- 作用域：定义的变量有效的范围
    - 自动持续变量：常见的定义的变量，执行到代码结尾则失去活性，存储在栈中
    - 静态持续变量
        - 未初始化的静态变量均为0
        - static变量无论是否使用，都驻留在内存中，更新时只会更新对应位置的变量
        - 结构体中可用mutable关键词代表const结构体中仍可以改变的成员
    - 函数使用static将被赋予单文件的作用域；不使用关键字或external都被赋予多文件作用域
```c++
int global = 1000;    // 外部链接性，其他文件可通过external访问
static int global_static = 100;   // 内部链接性，只在该文件里有用该文件该文件文件
void function()
{
  void static int local_static = 10;  // 只在函数内有用
}
```
```c++
struct student
{
  string name;
  mutable int age;
}
const student tom;
tom.age ++;        //仍被允许
```
* 链接性
## 名称空间
- 名称空间接名称空间可以是全局的，也可以位于另一个名称空间中，但不能位于代码块中
- using声明——指出能够使用namespace中特定值; using编译——使用namespace所有内容
- using声明更加安全
```c++
namespace tom
{
  string name = "tom";
}
int main()
{
  using namespace tom;  // 使用using编译不报错，但是使用using tom::name则会因为冲突而报错
  string name = "jack";
  cout<< name;          // 输出为jack，局部变量会隐藏全局变量
}
```
- 名称空间可以嵌套
- 可以使用=对名称空间进行命名
```c++
namespace student
{
  namespace tom
  {
    ...
  }
}
```
- namespace TOM = student::tom;则TOM为tom的别名

:top:[Back to Top](#c基础知识):top:

---


# **类Class**
## 类定义
```c++
class Stock
{
  private:
    char company[30];
    int shares;
    double share_val;
    double total_val;
    void set_tot() {total_val = shares * share_val;}
  public:
    void show();
    void update(double price);
    ...
}
```
* 实现类函数——通过className::funcName来标识类函数，同时类函数可以访问private组件
```c++
void Stock::show()
{
  ...  // 可以使用Stock内的private成员或方法
}
inline void Stock::set_tot()  // 内联函数除了在类声明时定义，也可以在后面定义
{
  ...
}
```
* 不同的对象占用不同的内存空间，但调用的方法是公共的。比如tom.show()和jack.show()都共用Stock::show()，但是两者调用的数据位于不同的内存区域
## 构造函数
- 构造函数与类名相同
- 构造函数没有返回类型，用于初始化类中的private成员，当代码中创建对象的时候会自动调用合适的构造函数进行初始化
- 构造函数的形式参数不能和类的成员同名，否则会有歧义，建议以_结尾作为类的成员名，如name_, age_
- 构造函数的使用
```c++
Stock garment = Stock("Furry Mason", 50, 2.5);
Stock food("World Cabbage", 250, 1.25);
Stock *pstock = new Stock("Electro Games", 18, 1.9);
```
- 在对一个对象进行赋值操作时，会先创建一个临时对象，然后进行值的复制，再将临时变量删除。如果既可以初始化生成对象，又可以赋值生成时，建议使用初始化。
```c++
Stock s1 = Stock("JC", 50, 0.5);  // 使用初始化
Stock s2;                         // 调用默认初始化
// 下面都是赋值操作，不推荐使用
// s2= Stock("KTV", 100, 1.8);
// s2 = s1;
```
- 在构造的时候如果是new不定长度的变量，那么在遇到只需要一个长度的变量时，请使用new[]，即如下，这样可以只需要再析构函数中写delete[]即可
```c++
str = new char[1];    // 而不是str = new char
```
- 如果有多种构造函数，一定要注意使用相同的new的方式，如都为new或都为new[]
- **默认构造函数**
    - 在不传入任何参数的时候，将自动default一些值(*隐式调用默认构造函数时不要加括号！！！！*)
    - 有参数的构造函数进行default设置
    - 新建无参数的构造函数
```c++
Stock::Stock()
{
  company = "";
  share = 0;
  share_val = 0.0;
}
// 或者是在已有的有参数构造函数中给所有形参设置default
// 正确使用默认构造函数
Stock first;
Stock first = Stock();
Stock *pfirst = new Stock;
// 错误使用默认构造函数，下面语句意为second函数返回值为Stock
Stock second();
```
- 列表初始化(*推荐使用，这样调用默认构造函数时不会出错*)
```c++
Stock first {};                  // 调用默认构造函数
Stock second = {"", 100, 0.5};
```
## 析构函数
- 在类名前加上~，即~ClassName()即为析构函数
- 析构函数在对象从内存上被删除时自动调用，不需要用户手动调用
- 如果构造函数使用了new，则析构函数要记得使用delete / delete[]
## const成员函数
```c++
private: 
  const int i;
  ...
public:
  // void show(){std::cout<<i;}此时会报错，因为不能保证函数不修改const int
  void show() const;  // 此时不会报错，这样是const成员函数，表明不会更改成员
```
## this指针
* this指向调用对象的地址
```c++
const double topval(const Stock& s) const
{
  if(s.val>val) return s.val;
  else          return this->val;    // 时刻注意this是指针而不是值
}
```
## 运算符重载
* 运算符重载格式
```c++
returnType operator op(Type params);
```
* 在使用op时，左侧的是调用的对象，右侧的为被作为参数的对象，如果使用成员函数重载，则左侧必为用户定义的类，即a op b == a.operator(b)
- 当使用友元函数重载时，a op b == operator(a, b)
- 运算符重载限制
    - 不能修改优先级
    - 不能创建新的操作符
    - 至少有一个操作数是用户定义的类型(否则会使原有的含义出现歧义)
    - =、[]、->、()这四个只能通过成员函数进行重载(即不能通过友元函数)~~
## 友元函数
- 友元函数能够访问类内数据，但是不属于类作用域
- 使用关键字friend，首先在类声明中声明，*再在类外进行定义，注意定义时不需要加上className::funcName与friend关键字!!*
```c++
class Stock
{
  ...
  public:
    friend Stock operator *(double a, const Stock& t);
}
Stock operator *(double a, const Stock& t)
{
  return t*a;    // 此处已经使用*运算符成员函数重载
}
```
- 如果操作数是一个类元素一个非类元素，通常要使用成员函数重载运算符+友元函数翻转运算符顺序
- 如果友元函数访问两个不同的类，并不意味着需要同时成为两个类的友元，该函数访问哪个类的数据，就需要成为哪个类的友元
- <<操作符重载(*注意该函数为用户类的友元*)
```c++
// 返回值为ostream&能实现多个<<连续使用
ostream& operator <<(ostream& os, className& C)
{
  os<<输出内容;
  return os;
}
```
## 类的自动类型转换和强制类型转换
- 只有一个参数的构造函数可以作为转换函数(*如果有多个参数，但其他参数均有default也可以*)
- 使用explicit作为限制，转换函数只能显式转换而不能隐式转换
```c++
// 有构造函数Stone(int, double) Stone(double) Stone()
Stone myCat;    // 调用隐式构造函数
myCat = 15.4;   // 隐式类型转换调用Stone(double)，在explicit下不行
myCat = Stone(18.5);  // 显式类型转换
myCat = (Stone) 16.4; // 显式类型转换
```
- 如果存在Stone(long)，则上述代码报错，因为存在二义性
- 如果要实现将类类型强制转换成普通类型，则需要使用转换函数
```c++
operator double() const;  // 转换函数在类内声明，无返回值和形参
double a = double(Stone);
```
## 动态内存分配
- 如果在类声明中定义了static类型成员变量，则需要在调用该类的文件的公共区域进行初始化(注意，初始化时并未使用static关键字，且使用了类作用域符)
```c++
/* filename = mystring.h */
class String
{
  ...
  static int num;    // 此处不能初始化，因为声明并不分配内存空间
}
/* file2 */
#include "mystring.h"
int String::num = 0;
int main(){...}
```
##### 复制构造函数
* 当新建一个对象并将其初始化为同类现有对象的时候调用复制构造函数
```c++
// 如果motto是StringBad类，则下列均会调用复制构造函数
StringBad ditto(motto);
StringBad ditto = motto;
StringBad ditto = StringBad(motto);
StringBad* p = new StringBad(motto);
```
* 由于按值传递会引起复制构造函数，所以一般使用按引用传递
* 引起复制构造函数的三种情况：
    * 以传值的方式送入函数作为参数
    * 以传值的方式return
    * 以赋值的方式初始化另一个对象
###### 浅复制
* 逐个复制非静态成员，复制的是成员的值
* 对于new过的部分，浅复制会导致两个类指向同一个内存空间，而当调用析构函数后，该内存空间会被释放，此时另一个类将指向内容不定的区域

![浅复制.png](https://i.loli.net/2020/10/07/lhVa9xTMS6Pq3vH.png)

###### 深复制
* 创建一个字符串的备份，这样两个类指向的内存空间不同

![深复制.png](https://i.loli.net/2020/10/07/Lid1sJ92e7ZPtwa.png)

* 如果类中使用了new初始化成员，则要定义复制构造函数，进行深复制
* 对赋值运算符也要进行重新定义，但是与复制构造函数有差别——(1) 由于被赋值对象可能已有值，所以要先delete[]; (2) 避免赋值给自身，否则先delete[]后就没法赋值了; (3) 函数要返回对象的引用，便于连续赋值.
```c++
StringBad& StringBad::operator =(StringBad& s)
{
  if(this == &s)    // 检查是否是对自身赋值
    return *this;
  delete[] str;     // 删除原有的指向内存 
  len = s.len;
  str = new char[len+1];
  std::strcpy(str, s.str);    // 进行深复制
  return *this;
}
```
###### 赋值函数 Vs 复制构造函数 Vs 构造函数
* 如果对象不存在，且没有使用别的对象进行初始化 -> 构造函数
* 如果对象不存在，用别的对象进行初始化 -> 复制构造函数
* 如果对象存在，用别的对象进行赋值 -> 赋值函数
## 静态成员函数
- 函数声明包括static关键字，但是定义中不能出现static
- 不能通过对象调用静态函数，因为静态函数是属于这个类的所有对象的，而不是属于这个类
- 静态函数只能访问类中的静态变量
```c++
class String
{
  ...
  static int num;
  int len;
  ...
  static int Howmany() {return num;}  // 只能访问num不能访问len
}
int main()
{
  String a;
  String b;
  int n = String::Howmany();  // 不能使用a::Howmany()这种
}
```
## 成员初始化列表
* 只能用于构造函数
* 必须用这种方法初始化非静态常量(非static的const)
* 必须用这种方式初始化引用数据成员
* 数据成员初始化的顺序与出现在类声明的顺序一致，与初始化器的排列顺序无关
```c++
Queue::Queue(int qs): qsize(qs), front(NULL), rear(NULL)
{
  ...
}
```
* 在C++11标准中，可以再声明const和引用变量时直接初始化，如果没有进行成员初始化，则使用该值，否则使用成员初始化列表中的值
```c++
class Queue
{
  ...
  const int qs = 0;    // 如果没使用Queue(int qs)初始化，则qs=0
  ...
}
Queue::Queue(int qs): qs(qs)  // 调用后qs = qs
{
  ...
}
```
## 继承
- 从已有的类派生出新的方法，但同时继承原有类(基类)的特征和方法
```c++
class TableTennisPlayer {...}
class RatedPlayer : public TableTennisPlayer  // 公有继承
{...}
```
- 继承的类需要添加属于自己的成员变量和函数，其可以继承基类的数据，但是只能通过基类函数来访问； 同时需要加入构造函数，同时初始化派生类成员和基类成员
```c++
class RatedPlayer : public TableTennisPlayer
{
  private:
    int rating;
  public:
    RatedPlayer(int r=0, const string& s="none");  // 两种方法同等
    RatedPlayer(int r, const TableTennisPlayer& tp);
}
// 要通过成员列表初始化才能进行对父类的初始化
RatedPlayer::RatedPlayer(int r=0, const string& s="none") : TableTennisPlayer(s)
{
  ...
}
// 如果不使用成员列表初始化，相当于调用父类的默认构造函数
//RatedPlayer::RatedPlayer(int r=0, const string& s="none"){...}
//RatedPlayer::RatedPlayer(int r=0, const string& s="none") : TableTennisPlayer(){...}
```
##### 派生类和基类的特殊关系:
* 派生类可以使用基类方法，条件是方法不是基类私有的
* 基类指针可以指向派生类；基类引用可以是派生类(基类能用的派生类都能用，反之不行)
##### 虚函数
* 由于基类（指针、引用）可作用于派生类，所以会产生歧义，而使用虚函数可以消除这些歧义
* 没有virtual——根据引用类型或指针类型选择方法；有virtual   ——根据引用和指针指向的对象选择方法
* 在基类中将派生类会重新定义的方法设置为virtual，这样在派生类中重定义时会自动生成virtual，但仍建议在派生类中协商virtual
* virtual关键字只用于类声明中，而不用于定义，在定义时通过className加以区分即可
* 如果某个函数在基类和派生类都定义了，则在派生类中运用基类方法时，一定要加入className加以区分，否则会产生递归
* 可以使用基类的指针数组，完成对基类和派生类的访问，通过虚函数自动根据对象进行函数选择，这就是多态性
* 使用虚析构函数，可以正确根据对象调用基类或派生类的析构函数，保证按照正确的顺序进行删除
* 虚函数表的工作原理:

![虚函数.png](https://i.loli.net/2020/10/11/FczQTEbfsDOHqSx.png)

* 构造函数不能是虚函数；基类析构函数应当为虚函数；友元函数不能为虚函数
##### 隐藏
* 在同一个class中定义同名但参数不同的函数是重载
* 在子类和父类中定义同名函数并不是重载（无论参数是否相同），而是会用子类隐藏父类
* 子类和父类中同名且参数相同的函数可以使用virtual来声明
* 返回类型协变——返回值中子类和父类的指针、引用是等价的，并不是隐藏
* 消除隐藏的方法是在子类中把父类被隐藏的函数再声明一遍（或virtual，保证参数完全相同）
##### public、private、protected继承
|    |public|private|protected|
|:----|:----|:----|:----|
|公有成员|公有|私有|保护|
|私有成员|不可直接访问|不可直接访问|不可直接访问|
|保护成员|保护|私有|保护|

* 保护继承——如果想让派生类了解基类，但又不想让外接知道类内成员，则使用保护
* 对数据采用private，对函数采用protected
## 抽象基类(ABC)
* 通常在ABC中声明暂时无法实现的函数为纯虚函数，类似于python中的pass
* 使用纯虚函数的原因就是为了在派生类中重新定义它
* 纯虚函数使用virtual关键字和=0表征
```c++
class Circle
{
  private:
    double x;
    double y;
    double r;
    ...
  public:
    ...
    void Move(int nx, int ny){x=nx; y=ny;}
    virtual double Area() const = 0;    // pure virtual function
    ...
}
```
* 使用纯虚函数将A->B变为了A->B，A->C的关系，即ABC中存储BC公共的数据和纯虚方法，再在BC内定义所需的数据，并覆盖纯虚方法。可以将ABC看作一种必须实施的接口。
## 继承和动态内存分配
##### 派生类不使用new
* 由于不使用new，则不存在深复制和浅复制，也无需特殊的析构函数，赋值、复制时都能正确调用默认函数，所以基本上不需要做任何显式定义和声明
##### 派生类使用new
* 由于需要释放new的成分，需要显式定义析构函数；同时此时会出现浅复制，需要显式定义复制构造函数和赋值函数
* 析构函数——需要delete派生类的new的东西
* 复制构造函数——需要使用基类（复制）构造函数，可以使用成员列表初始化，将基类成员数据先初始化，再将派生的成员初始化
```c++
hasDMA::hasDMA(const hasDMA& hs) : baseDMA(hs)
{
  style = new char[std::strlen(hs.style)+1];  // style is char* in class
  std::strcpy(style, hs.stley);
}
```
* 赋值函数——需要使用基类的赋值函数，将基类成员赋值后，再赋值派生的成员
```c++
hasDMA& hasDMA::hasDMA(const hasDMA &hs)
{
  if(this==&hs)
    return *this;
  baseDMA::operator=(hs);    // new line
  delete[] style;
  style = new char[std::strlen(hs.style)+1];
  std::strcpy(style, hs.stley);
  return *this;
}
```
##### 友元
* 如果一个函数是子类的友元，如何访问父类成员？（使用baseDMA的同名函数）
* 由于友元无法使用className，如何使用baseDMA函数？（使用强制类型转换）
## 类设计小结
##### 三种默认函数
* 默认构造函数
* 默认复制构造函数（在使用new时需要显式定义）
* 默认赋值函数（在使用new时需要显式定义）
##### 按值传递和按引用传递
* 推荐按引用传递
* 按值传递会引起复制构造函数，可能导致浅复制；在派生类中使用虚函数，父类的引用可以运用到派生类
##### 返回引用和返回值
* 如果返回临时变量，则返回值
* 如果返回传入的引用、new的空间等，使用返回引用，避免调用复制构造函数或者赋值函数
##### 继承
* 构造函数、析构函数、赋值函数不能被继承
* 友元函数无法继承，但可以使用强制类型转换进行子类、父类的访问
* 可以将派生类赋给基类，但反之不一定可以，除非有转换函数
##### 类函数
|**函数**|**能否继承**|**成员还是友元**|**是否默认生成**|**能否为virtual**|**是否有return**|
|:----|:----|:----|:----|:----|:----|
|构造函数|否|成员|是|否|否|
|析构函数|否|成员|是|是|否|
|=|否|成员|是|是|是|
|&|是|任意|是|是|是|
|转换函数|是|成员|否|是|否|
|() [] ->|是|成员|否|是|是|
|友元|否|友元|否|否|是|

:top:[Back to Top](#c基础知识):top:

---


# **代码复用**
## valarray类
* 模板类，在声明时需要指明数据类型，如：valarray<int> a;
* 使用valarray
```c++
double gpa[5] = {3.1, 2.5, 3.9, 3.4, 3.7};
valarray<double> v1;
valarray<int> v2(8);  // an array of 8 elements
valarray<int> v3(10,8);  // an array of 8 elements, which are 10
valarray<double> v4(gpa, 4);  // an array of 4 elements, init as {gpa[0], ..., gpa[3]}
valarray<int> v5 = {1,2,3};  // use list init
```
## 包含 Vs 私有继承
- 使用继承时，类可以继承接口，还可能有实现（纯虚函数提供接口没有实现）；包含可以获得实现，而无法获得接口
- 初始化被包含的对象时，因为初始化的是成员对象而不是继承的对象，所以使用成员名而不是类名
- *继承->使用父类的构造函数，即使用类名进行初始化，同时使用className::进行访问；包含->使用包含声明中的成员名称进行初始化，同时使用成员名称进行访问*

```c++
// 包含
class Student 
{
  private:
    string name;
    valarray<int> scores;
    ...
   public:
     Student(const char* str, const double* pd, int n) : name(str), scores(pd, n) {...};  // 注意，此处用的是name(str)而不是string(str)
}
// 继承
Sclass Student : private std::string, private std::valarray<double>
{
  private:
    ...
  public:
    Student(const std::string8 & str, const double* pd, int n) : std::string(str), std::valarray(pd, n) {...};  // 注意，此处用的是类名
}
```
* 在继承中，由于没有父类的对象，所以无法通过对象名字来实现调用，此时使用强制类型转换
```c++
const string& Student::Name() const
{
  return (const string&) *this;  // this是string的子类，强制类型转换进行子类->父类变化
}
```
* 在私有继承中，子类对象不能赋给父类指针或引用，除非使用强制类型转换
* 在大多数情况下使用包含；如果需要重构虚函数或者访问原有类的protected成员，则应使用私有继承
* 私有继承中，如果A->B->C，那么C无法访问A的成员，除非B中有函数访问了A，那么这样就相当于套娃访问，可以摆脱私有继承的束缚
```c++
#include <iostream>
class A
{
private:
    int a_;
public:
    A() : a_(10){}
    void show() {std::cout<<a_<<std::endl;}
};
class B : private A
{
private:
    int b_;
public:
    B() : A(), b_(20) {}
    void show_B() {show();}  // 此处访问了A，并为C提供了访问A的渠道
};
class C : private B
{
private:
    int c_;
public:
    C() : B(), c_(30) {}
    void show_C() {show_B();}  // B的成员成了private，但是B提供访问A的方法，所以可以C->A
};
int main()
{
    C c;
    c.show_C();
}
```
* 如果使用protected继承，则不需要有上述的束缚
```c++
#include <iostream>
class A
{
private:
    int a_;
public:
    A() : a_(10){}
    void show() {std::cout<<a_<<std::endl;}
};
class B : protected A
{
private:
    int b_;
public:
    B() : A(), b_(20) {}
};
class C : protected B
{
private:
    int c_;
public:
    C() : B(), c_(30) {}
    void show_C() {show();}
};
int main()
{
    C c;
    c.show_C();
}
```
* 可以使用using摆脱私有继承的束缚(*using只需要成员名，不需要圆括号、特征标和返回类型*)
```c++
#include <iostream>
class A
{
private:
    int a_;
public:
    A() : a_(10){}
    void show() {std::cout<<a_<<std::endl;}
};
class B : private A
{
private:
    int b_;
public:
    using A::show;      // 相当于提出了一个特例，可以摆脱私有继承 
    B() : A(), b_(20) {}
};
class C : private B
{
private:
    int c_;
public:
    C() : B(), c_(30) {}
    void show_C() {show();}
};
int main()
{
    C c;
    c.show_C();
}
```
## 多重继承(MI)
* 即一个派生类源自多个基类，在声明时要将每个基类的派生类型指明，默认为private
* 由于同时派生多个基类，而这些基类可能共用基类，即A->B,A->C,B+C->D，那么当使用基类指针A指向D时，会出现歧义，不知道该使用B的基类还是C的基类。最好的方法是使用类型转换
```c++
SingingWaiter sw;
Worker* p = (Singer *) &sw; // 此时使用的是singer的基类
```
##### 虚基类
* 为了避免上述的问题，使用虚基类将使得B,C共用同一个基类

![两个基类.png](https://i.loli.net/2020/10/12/5zdLeci71a2vUEp.png)
![虚基类.png](https://i.loli.net/2020/10/12/wnWuxpyrQilIS1z.png)

* 在BC中使用virtual public或者public virtual进行派生
* 虚基类构造时特殊操作——在列表初始化的时候，并不会出现"D调用B的构造函数，B的构造函数里调用A的构造函数"这样的操作，因为会从BC两个不同道路初始化同一个虚基类。此时需要"分别调用A,B,C的构造函数"
```c++
SingingWaiter(Worker& wk, int p=0, int v=Singer::other)
    : Worker(wk), Waiter(wk,p), Singer(wk, v){...}
```
* 如果BC中均有Show()函数，则在D中使用Show()的时候要注明是B::Show()还是C::Show()
* 当同名函数是继承与被继承的两个类中，则不会出现二义性，派生类将覆盖父类；当同名函数不是继承与被继承关系，则将会产生二义性
## 类模板
* 使用template<typename T>放在类开头，声明为模板类，同时，类的限定符从className::变为className<T>::
* 在实例化（具体化）类模板时，要显式提供类型
##### 表达式参数
* 如果使用template<class T, int n>形式，则这种指定类型而不是泛型的参数称为非类型或表达式参数。
    * 表达式参数只能为int，enum，&或*，即double m是非法的
    * 表达式参数不能修改，也不能取地址，即不能出现n++或者&n操作
    * 实例化模板时必须是常量表达式，不能等待用户输入等操作
* 使用表达式参数法相较于使用构造函数法，优点是使用自动变量维护内存，效率高；缺点是每种不同大小的数组都要产生新的实例化
```c++
ArrayTP<int, 12> a;
ArrayTP<int, 13> b;    // 将产生两个不同的模板实例化
Stack<int> c(12);      // 只会产生一个模板实例化
Stack<int> d(13);
```
##### 模板递归（嵌套）
* 可以使用ArrayTP< ArrayTP<int, 5>, 10 >，其表示有10个数据，每个数据都是5个int组成，其等价于int [10][5]
* 模板语法里，维的顺序和二维数组相反
##### 模板具体化
* 实例化——可以用模板实现的，只不过是指定一下具体的类型；具体化——模板实现不了的，要针对模板处理不了的特例进行设计
* 显示具体化：使用如下操作进行显示具体化
```c++
template<> class Classname<specialized-type-name> {...};
```
* 部分具体化
```c++
template<class T1> class Pair<T1, int> {...}
template<class T1, class T2, class T3> class Trio{...}
template<class T1, class T2> class Trio<T1, T2, T1>{...}
template<class T1> class Trio<T1, T1*, T1&>{...}
```
* 如果模板定义在类外，而类内使用了不同的模板函数，则需要嵌套定义
```c++
template<class T>
class beta
{
    private:
        template<class V>    // nested class template member
        class hold
        {
        private:
            ...
        public:
           hold(V v=0):val(v){}  //将hold在内部定义
        }
    public:
        beta(T t, int i);
        template<class U>
        U blab(U u, T t) {...}
}
```
```c++
template<class T>
class beta
{
    private:
        template<class V>    // nested class template member
        class hold;
    public:
        beta(T t, int i);
        template<class U>
        U blab(U u, T t);
}
template<class T>
template<class V>    //嵌套调用
class beta<T>::hold
{
    ...
}
template<class T>
template<class U>
U beta<T>::blab(U u, T t)
{
    ...
}
```
##### 将模板作为参数
```c++
template< template<typename T> class Thing>
class Crab
{
private:
    Thing<int> s1;
    Thing<double> s2;
public:
    Crab(){};
}
int main()
{
   Crab<Stack> ne;  // s1和s2被定义为Stack<int>和Stack<double>类型  
   ...
}
```
* 上述代码中，Thing是一个模板类，用的T代表各种数据类型，然后Crab也是一个模板类，用的Thing代表各种数据类型
##### 模板类和友元
* 非模板友元函数
    * 友元函数不属于类，所以无法使用className::进行解析，每当调用一种数据类型时，都需要为友元函数进行显示实例化
```c++
template <typename T>
class HasFriend
{
    ...
    friend void cnt(HasFriend<T>& hf);
}
// 当使用了T==int的
void cnt(HasFriend<int>& hf)    {...}
// 当使用了T==double的
void cnt(HasFriend<double>& hf)    {...}
```
* 约束模板友元函数
    * 在类定义前声明每个模板函数
    * 在类中声明为友元
    * 对友元进行函数定义，实现功能
    * *避免了换一种数据结构就要显示实例化一次*
```c++
template <typename T>
void cnt(T &);
template <typename TT>
class HasFriend
{
    ...
    friend void cnt<>(HasFriend<TT>& hf);  // 显示具体化
}
// 适用于所有的T类型
template <typename T>
void cnt(T& hf)    {...}
```
* 非约束模板友元函数
    * 在类内声明模板函数，创建非约束友元函数，每个函数具体化都是每个类具体化的友元
```c++
template<typename T>
class ManyFriend
{
    ...
    template <typename C, typename D> friend void show(C&, D&);
}
// 是T==int的具体化的友元
void show<>(ManyFriend<int>& c, ManyFriend<int>& d);
// 是T==double和T==int的具体化的友元
void show<>(ManyFriend<double>& c, ManyFriend<int>& d);
```
##### 模板别名
* 使用using命令进行别名设置
```c++
template<typename T>
using month = array<T, 12>;
// 两者等价
array<int, 12> a;
month<int> b;
```

:top:[Back to Top](#c基础知识):top:

---


# **友元和异常**
## 友元
* 类友元
    * 比如遥控器和TV就是友元类的关系：TV可以被遥控器遥控，即遥控器可以访问TV，反之不行。所以TV的友元类是遥控器。
```c++
class TV
{
  friend class Remote;  // 由于Remote要访问TV，所以必须先知道TV有什么
                        // 所以TV定义在Remote前面
  ...
}
class Remote {...}
```
* 类成员友元
    - 仅让特定的函数称为另一个类的友元
    - 前向声明——因为Remote中只有一个函数访问了TV的private，所以在TV中定义friend Remote::func，但是这又必须知道Remote，那么Remote应该在TV前面定义，如此会产生循环嵌套。只有使用前向声明才能避免
    - 注意，*在前向声明中，最好不要直接定义函数*，如在Remote中使用TV的函数。只能在上述代码后面进行函数定义
```c++
class TV;    // forward declaration
class Remote
{
  void set_channel(TV& t, int c);
}
class TV
{
  friend void Remote::set_channel(TV& t, int c);
}
```
* 彼此成为友元
    * 两者可以任意顺序，但是，前一个定义的类中函数不能直接定义，需要在两个类都声明后才能定义函数；而第二个类中的函数可以直接类内定义
* 共用友元
```c++
class Analyzer;
class Probe
{
  friend void sync(Analyzer& , Prob& );
  friend void sync(Prob& , Analyzer&);
}
class Analyzer
{
  friend void sync(Analyzer& , Prob& );
  friend void sync(Prob& , Analyzer& );
}
```
## 嵌套类
* 包含：在类内实例化某个类；嵌套：在类内定义新的类
```c++
class Queue
{
  class Node
  {
    ...  // 可以定义构造函数实现初始化
  }
  ...
} 
// 使用Queue::Node::对Node类进行访问，如果Node定义在public则可以在Queue类外访问，如果定义在private则只能在Queue中访问
```
* 嵌套类的作用域，访问权限和类内普通的public，private和protected一样
* 注意：如果Node类内有private或protected，则Queue也无法访问Node内这些内容，所以建议Node类内全定义为public，然后把Node定义在Queue的private或protected中
## 异常
##### abort()
* 调用该函数将直接终止程序，不会返回到main
##### throw
* 引发异常，其后的值代表了异常的特征
* throw语句的执行类似于return，但并不是将控制权返回给调用程序，而是沿函数调用的顺序后退，直到找到try语句块对应的catch块
##### try
* 可能出现异常，程序尝试调用的语句模块
* 如果try中没有异常，则跳过catch，否则调用合适的catch
##### catch
* 如果try中出现异常，catch会捕获相应的throw，并根据throw的类型选择合适的catch语句块，执行相应的函数
* 如果没有对应的catch，则默认调用abort()
```c++
int main()
{
  try
  {
    z = hmean(x, y);  // 尝试调用可能产生异常的函数
  }
  catch(char * s)    // 如果try出错，则s==throw的内容并执行catch中代码
  {
    std::cout<<s<<endl;
  }
  std::cout<<z<<endl;
}
double hmean(int a, int b)
{
  if(a == -b)
    throw "wrong";
  return 2.0*a*b/(a+b)   
}
// x=3, y=6 -> 4
// x=3, y=-3 -> wrong
```
##### 将对象用作异常类型
* 可以使用不同的异常类型来区分不同的异常
* 对象可以携带信息，可以根据信息确定异常的原因
```c++
class bad_hmean
{
private:
    int v1;
    int v2;
public:
    bad_hmean(int a, int b):v1(a),v2(b){}
    void print_hmean();
}
class bad_gmean
{
    private:
    int v1;
    int v2;
public:
    bad_gmean(int a, int b):v1(a),v2(b){}
    void print_gmean();
}
int main()
{
    try
    {
        z = hmean(x, y);
        w = gmean(x, y);
    }
    catch(bad_gmean& bg)
    {
        bg.print_gmean();
    }
    catch(bad_hmean& bh)
    {
        bh.print_hmean();
    }
}
double hmean(int a, int b)
{
    if(...)
        throw bad_hmean(a, b);
    ...
}
double gmean(int a, int b)
{
    if(...)
        throw bad_gmean(a, b);
    ...
}
```
##### 栈解退
* 如果try没有直接调用引发异常的函数，而是调用了对引发异常的函数进行调用的函数，则会引发栈解退
* throw后，并不会返回一个函数就停止，而是继续释放，直到到达try处
* 同时，不会直接执行出错的下一句，而是执行最邻近的catch，不断比较合适的catch块
* 在throw时，某些临时变量无法从栈中删除，就要靠栈解退实现
##### 其他属性
* 为什么要使用引用？—— 基类引用可以针对派生类，使用基类引用即可以捕获所有派生类异常；如：文件异常(读异常，写异常，not found异常...)，那么只需要catch一个文件异常的引用，即可以检查所有读写异常
* catch块排列顺序——按照先派生类，再基类，由小到大，由具体到笼统的顺序。因为catch是按照顺序进行排查异常的，如果基类写前面，则无法检查更细致的问题
* 捕获任何异常——catch(...)
##### exception类
* 可以视为是各种异常的基类
```c++
class bad_hmean : public std::exception  // 继承
{
  public:
    const char* what(){return "wrong";}
    ...
}
```
* 使用exception&可以捕获各种派生类型的异常
```c++
catch(exception& e)  // 利用基类引用，处理所有派生类异常
{
  ...
}
```
## RTTI（运行阶段类型识别）
* RTTI只能用于包含虚函数的类层次结构
##### dynamic_cast<>
* 如果转换不合法，则返回为NULL，否则进行类型转换
```c++
class Grand {...};
class Superb : public Grand {...};
class Magnificent : public Superb{...};
Grand * pg = new Grand;
Grand * ps = new Superb;
Grand * pm = new Magnificent;
Magnificent* p1 = (Magnificent*) pm;    // valid, Magn -> Magn
Magnificent* p2 = (Magnificent*) pg;    // invalid, Magn -> Grand
Superb* p3 = (Magnificent*) pm;         // valid, Sup -> Magn
Superb* p4 = dynamic_cast<Superb*> pg
```
* 可以利用返回为0或指针的特点，放入if语句进行条件判断。即如果可以转化成对应数据类型，则执行if内容，否则不执行。
* 可以使用引用，但是返回不会为0，而是会引发exception异常
```c++
Superb& s = dynamic_cast<Superb&> rg;
```
##### typeid和type_info
* 获取内容的类型，可用于类型是否相同的比较
* 如果在if else语句中使用了typeid，则应考虑更改成虚函数或dynamic_cast<>
```c++
typeid(ps) == typeid(pm);
```
## static_cast、const_cast和reiterpret_cast
* const_cast用于const和volatile两种类型的转换，typename和expression类型必须一致
* static_cast用于typename和expression可以相互隐式转换的情形
* dynamic_cast用于在类层次结构中进行向上转换
```c++
xxx_cast<typename> (expression)
```

:top:[Back to Top](#c基础知识):top:

---

# **智能指针模板类**
* 使用智能指针时，当指针过期，会自动调用析构函数进行内存释放
* 包含头文件memory才能使用智能指针
* 使用智能指针时不需要使用delete或delete[]
* 智能指针有三种：auto_ptr，shared_ptr，unique_ptr
```c++
auto_ptr<double> p(new double);
```
* 智能指针的转换函数和赋值函数是含有explicit的，即必须显式定义
```c++
shared_ptr<double> pd;
double* p_reg = new double;
pd = p_reg;                             // invalid
pd = shared_ptr<double> (p_reg);       // valid
shared_ptr<double> pshare = p_reg;     // invalid
shared_ptr<double> pshare(p_reg);      // valid
```
* 智能指针只能指向new过的堆，不能指向栈
```c++
string s = "hello, f5";
shared_ptr<string> ps(&s);  // invalid
```
* shared_ptr使用引用计数，即*可以有多个指针指向同一个内存*，仅当最后一个指向内存的指针销毁时，才调用析构函数；而unique_ptr使用控制权的概念，即每次*只能有一个指针指向一个内存区*
```c++
auto_ptr<string> s(new string("Hello, f5"))
auto_ptr<string> t;
t = s;        //    出现问题，当t和s销毁时，字符串内存销毁了两次
unique_ptr<string> s(new string("Hello, f5"))
unique_ptr<string> t;
t = s;        //    不允许，因为两者冲突，指向同一个内存
// 可以使用t = std::move(s)来移交控制权
// 由于t和s都是长期存在，所以不允许；如果s是一个临时变量，则被允许，因为s完成操作后即消失。即unique<string>p = unique<string>(new string("OK"))被允许 
shared_ptr<string> s(new string("Hello, f5"))
shared_ptr<string> t;
t = s;        //    允许，删除时仅在最后一个指针销毁时调用析构函数
```
* *使用new分配内存时，才能使用auto_ptr和shared_ptr；使用new[]分配时，只能使用unique_ptr*

:top:[Back to Top](#c基础知识):top:

---

# **STL类（基础）**
* STL提供了一组表示容器、迭代器、函数对象和算法的模板
* 容器：同质的，类似于数组，只能保存相同数据类型的数据(数据结构之一)
* 迭代器：能够用于遍历容器中的数据(容器使用的特殊指针)
* 函数对象：类似于函数的对象，可以是类对象和函数指针
* 算法：完成特定任务的方法
* STL不是面向对象编程，而是泛型编程
* 容器是一种数据结构，模板是算法；但是容器是由模板实现的
## vector模板类
* 使用动态分配，可以由用户声明长度
* vector< type, num> vecName;
* 迭代器可以视为是一种广义指针，通过声明，可以访问vector中的各个元素，进行增删查改等
```c++
vector<string> s(10);
vector<string>::iterator ps;  // 声明一个迭代器
ps = s.begin();               // ps指向迭代器开头
*ps = 'c';                    // 开头设置为c  
ps++;                         // 指向第二个元素
ps.insert(...);               // 插入
ps.erase(...);                // 删除
ps.push_back('d');            // 在尾部加入一个d元素
...
```
* 定义了非成员函数实现对各种不同容器类进行相同操作。比如for_each()，random_shuffle()，sort()等
## 泛型编程
##### 容器
* 容器是由模板类实现的，是存储数据的数据结构的一种
* 所有容器都具有的基本特征，如赋值，构造函数，复制构造函数，析构函数，实例化等
* 容器有三种复杂度——编译时间、固定时间、线性时间
* **序列容器**——指容器中数据有顺序，存在第一个元素和最后一个元素，中间的元素左右两边各有一个元素，允许进行随机访问，常见的有vector, deque, list ,queue, stack, array, forward_list, priority_queue
* **关联容器**——键值对，类似于python的字典。常见的有set(其值==键), multset(set中可以有重复键), map(python中字典，键唯一，每个键一个值), multimap(一个键可以多个值)
* **无序关联容器**——关联容器使用树结构，而无序关联容器使用哈希表，常见的是在关联容器前加上unordered,如unordered_map
##### 迭代器
* 模板使算法独立于数据的类型，而迭代器使算法独立于使用的容器类型。如：模板使得不论是int数组还是double数组，都可以进行操作；而迭代器使得不论是数组，还是链表，都可以进行操作
* 迭代器基本操作包括：解引用（*），递增递减（++/--），赋值（=），判断（==/!=）
* STL定义了5种迭代器——输入/输出/双向/随机访问/正向迭代器
    * 输入 / 输出迭代器：无序，只能++递增，分别只能读/写
    * 正向迭代器：有序，可以++递增，能够读写
    * 双向迭代器：有序，可以++和--，能够读写
    * 随机访问迭代器：有序，可以++和--，还能够进行±n的跨越式访问，能够读写
* *迭代器相当于指针，而容器相当于数据类型*，要访问容器（数据结构）的值，就要定义迭代器（指针）
* iterator头文件中，除了常规的iterator外，还定义了如reverse_iterator，对该迭代器进行++操作将会访问前一个单元
```c++
copy(words.rbegin(), words.rend(), show);  // rbegin和end指向相同，但前者是reverse_iterator，后者是iterator
vector<string>::reverse_iterator ri;      // 显示声明了一个指针（迭代器）
for(ri=words.rbegin(); ri!=words.rend(); ++ri)
    cout<< *ri;
```
* 这些新加入的迭代器，会以容器类型作为模板参数，容器对象作为构造函数参数
```c++
back_insert_iterator<vector<int>> back(vectorName);
```
```c++
#include <iterator>
#include <iostream>
void show(const std::string& s) {std::cout<<s<<std::endl;}
int main()
{
    using namespace std;
    string s1[4] = {"a", "b", "c", "d"};
    string s2[2] = {"e", "f"};
    vector<string> words(4);
    copy(s1, s1+4, words.begin());
    for_each(words.begin(), words.end(), show);        // a b c d
    copy(s2, s2+2, back_insert_iterator<vector<string>> (words));
    for_each(words.begin(), words.end(), show);        // a b c d e f
}
```
##### 函数对象(函数符)
* 函数符是可以以函数方式与()结合使用的任意对象。这包括函数名、函数指针、重载了()的类对象
```c++
template<typename Inputiter, typename Func>
Func for_each(Inputiter first, Inputiter second, Func f);
// 如果f处是一个函数名，则使用函数指针；如果f处是一个类对象，则使用重载()的对象
```

> Q:以for_each为例，为什么要返回一个Func类型呢？<br/>
> A:因为传入的f可能是一个类，那么这个类可能在处理过程中被改变，所以需要返回更改过后的f方便用户访问。
>> ```c++
>> #include <iostream>
>> #include <iterator>
>> #include <algorithm>
>> #include <list>
>> class myprint
>> {
>>     private:
>>         int cnt_;
>>     public:
>>         myprint(int cnt=0):cnt_(cnt){}
>>         int show(){return cnt_;}
>>         void operator()(int x){std::cout<< x<< std::endl; cnt_++;}
>> };
>>
>> int main()
>> {
>>     using namespace std;
>>     list<int> arr;
>>     for(size_t i=0; i<5; i++)
>>         arr.push_back(i);
>>     //myprint a = for_each(arr.begin(), arr.end(), myprint());
>>     myprint a;
>>     a = for_each(arr.begin(), arr.end(), a); // 此处要a=for_each(...)，否则a.show()仍为0
>>     cout<< a.show()<< endl;
>>     system("pause");
>> }
>> ```

> Q：为什么要重载()或者说为什么for_each的第三个参数一定要是一元函数呢？<br/>
> A：这与for_each内部的实现有关，可以看下面的解析：
>> ```c++
>> template<class InIt, class Func>
>> Func for_each(InIt first, InIt second, Func f)
>> {
>>    _For_Each(first, second, f);
>> }
>> 
>> template<class InIt, class Func> inline
>> void _For_Each(InIt first, InIt second, Func& f)
>> {
>>   for(; first!= second; first++)
>>   {
>>     f(*first);   // 函数指针
>>   } 
>> }
>> ```
> 由此可以看出，重载()操作符后，f(*first)即可以按照重载的内容进行操作；传入函数指针，也可以按照正常流程运行。

* 重载()的类可以如下所示
```c++
class Linear
{
  private:
    double y0_;
    double slope_;
  public:
    Linear(double y=0.0, double slope=1.0) : y0_(y), slope_(slope){}
    double operator()(double x) {return y0_ + slope_ * x;}
};

int main()
{
  Linear L1(1, 2);    // 调用构造函数生成y和slope
  double out = L1(3); // out = 1 + 2 * 3
}
```
###### 函数符概念
* 生成器——不同参数就可以调用的函数符
* 一元函数——调用一个参数
* 二元函数——调用两个参数
* 谓词——返回bool的一元函数
* 二元谓词——返回bool的二元函数
```c++
// 使用函数指针
bool tooBig(int n)
{return n>100;}

list<int> scores;
scores.remove_if(tooBig); // 只能移除大于100的
```
```c++
// 使用重载运算符()
template<typename T>
class tooBig
{
  private:
    T cutoff;
  public:
    tooBig(T t): cutoff(t){}
    bool operator()(T n) {return n>cutoff;}
}

list<int> scores;
tooBig<int> f200(200);
scores.remove_if(f200)
// scores.remove_if(tooBig<int> (300))
```
* 以for_each为例，第三个参数需要为一元函数，如果某个函数需要两个参数，则可以使用类的方法缩减为一个参数
```c++
template<class T>
bool tooBig(T a, T b)
{
  return a>b;
}

template<class T>
class tooBig2
{
  private:
    T b;
  public:
    tooBig2(T t): b(t) {}
    bool operator()(T& a) {return tooBig<T>(a, b);}
}

/*
tooBig2<int> tB100(100);
for_each(..., ...,tB100(x)); // 等价于tooBig(x, 100);
*/
```
###### 预定义函数符
* 如果要实现两个迭代器相加，则不能使用"+"，而要创建一个函数执行加法，然后把函数名作为函数指针传入迭代器中
* 为了避免这么麻烦，STL为所有算术运算、逻辑运算与关系运算创建了模板函数符

运算符 | 相应函数符
:---: | :---:
  '+'   | plus<>
  '-'   | minus<>
  ... | ...
  '==' | equal_to
  ... | ...

###### 自适应函数符
* 让函数符成为自适应的：携带了标识参数类型和返回类型的typedef成员，分别是result_type, first_argument_type, second_argument_type
* 这些typedef让函数适配器对象知道有这些成员，并可以使用

###### 函数适配器
* 如果某个函数需要两个参数，而需要放入一元函数的函数符中，那么有两种方法
  * 使用tooBig2的方法
  * 使用适配器binder1st和binder2nd自动完成
* 适配器binder1st使用方法
```c++
binder1st(f2, val) f1;
f1(x);
f2(val, x);   // 两者等价
for_each(list.begin(), list.end(), binder1st(plus<double>(), 1.5)); // 将区间内每个值都加上1.5
```
* 适配器binder2nd使用方法类似，只不过val是第二个参数
```c++
binder1st(f2, val) f1; // f1(x) == f2(val, x)
binder2nd(f2, val) f1; // f1(x) == f2(x, val)
```

##### 算法
* 均使用模板提供泛型+迭代器提供数据通用访问
* 基本上接受表示范围的两个迭代器+对数据操作的算法
* 就地算法：新结果被保存在原来的位置<br/>
  赋值算法：新结果被保存到新的位置
* 同样的功能，可以选择使用“容器方法”和“函数”，但要注意两者区别。容器方法由于是类内成员，所以可以更改如长度相关的数据，而函数只能处理数据，如果不经过赋值，可能会出现数据被删除但是长度没变的问题
```c++
lsit<int> score;
score.remove()  // 容器方法，移除数据后，长度会变
remove(score, ...) //函数，移除数据，但是长度信息等不会变
// score = remove(score, ...)只有这样才与容器方法等效
```

:top:[Back to Top](#c基础知识):top:

---

# **C++11新标准** 
## 右值引用
* 左值——在内存中有空间的量<br/>
  右值——在寄存器中存储的临时变量或不容更改的值，如函数返回值，表达式，const常量
* 右值可以关联字面常量（C风格数组除外，其表示地址），x+y等表达式，返回值的函数（返回引用的除外）
> 返回引用的函数在return时，需要返回new过的内容或者传入的引用值，即不能返回右值，所以不能赋给右值参数
>> ```c++
>> int& f1(int& x){x=x+1; return x;}  // valid，因为x是左值
>> int& f2(int& x){return x+1;} // invalid，因为x+1是右值
>> const int& f3(int& x){x=x+1; return x;} // valid，因为x是左值
>> const int& f4(int& x){return x+1;} // valid，因为const type&可以接受左值或右值，但是此时会生成临时变量temp，进行完返回后再删除
>> ```
* 对右值使用&可以获得其地址
```c++
int x = 10;
int& lx = x;
int&& rx = 10;
// &10 is invalid
cout<< &x<<endl;  // 0xff00
cout<< &lx<<endl; // 0xff00
cout<< &rx<<endl; // 0xff04
```
* *const type&（常量左值引用）可以被赋予右值，因为const代表了该值不能被修改，则可以认为不能作为左值使用，那么可以作为右值使用（注意：如果在函数参数列表中使用，会产生临时变量!!）*
```c++
void printL(string& s){cout<< s;}
void printLR(const string& s){cout<< s;}
int main()
{
  printL("hello");  // error
  printLR("hello"); // right
}
```
* 右值引用关联的是当前语句的值，即后面右值发生改变，但是右值引用仍不变
```c++
int x=10;
int y=20;
int&& sum=x+y;  // sum为30
x=20;           // sum仍为30
```

## 移动语义
##### 移动语义
* 移动语义实际上避免了数据的大规模移动，而是在原地等待修改
* 复制构造函数可以使用深复制，进行数据的转移；移动构造函数只会在原有位置调整数据
* 在下列代码中，生成three的时候调用了移动构造函数。先由one+two生成临时变量，同时通过移动语义，将three和该临时变量进行浅复制，不用new新的空间，节省操作和空间（比如考虑n非常大）
```c++
class Useless
{
  private:
    int n;
    char* pc;
    ...
  public:
    Useless(const Useless& u){...}
    Useless(Useless&& u){...}
}

Useless::Useless(const Useless& u):n(u.n)
{
  pc = new char[n];
  ...
}

Useless::Useless(Useless&& u):n(u.n)
{
  u.n = 0;
  pc = u.pc;        // 夺取控制权，成为窃取(pilfering)
  u.pc = nullptr;   // 由于临时变量u会被析构函数调用，同时当类过期时，也会调用析构函数，如果不将临时变量的指针设为nullptr，则退出该函数时，该内存区域就会被清空
}

int main()
{
  Useless one;
  Useless two;
  Useless three(one+two);
}
```
* 如果上述代码不使用移动构造函数，而是使用复制构造函数，那么生成three的时候，会由one+two先生成临时变量，再由临时变量new一块区域并初始化该区域，再将这块区域复制给three，再把临时变量生成的区域delete[]，增加了时间和空间复杂度
> ![IMG_20201018_181311.jpg](https://i.loli.net/2020/10/18/La2RBznKfPSsuZl.jpg)
* 移动语义也可以用于operator =()的重载
##### std::move()
* 对于左值，其实也可以使用移动语义，比如可以摒弃复制构造函数，直接使用移动构造函数
```c++
Useless one;
Useless two(one); // 如何在此处使用移动构造函数而非复制构造函数呢？
```
* std::move(left value)可以将左值转换成右值
* 如果没有定义移动构造函数，则move仍将调用复制构造函数
* 如果move(x)中x不是左值，而是右值，也不会发生任何变化

##### C++11新的类功能
###### delete和default关键字
* C++原本默认提供4种函数——构造函数、析构函数、复制构造函数、赋值函数
* C++11引入移动语义后新加入2种默认函数——移动复制构造函数和移动赋值函数
> 当提供了复制XX函数时，不会自动生成移动XX函数<br/>
> 同理，当提供了移动XX函数时，不会自动生成复制XX函数
* 如果想禁用或者开启默认方法，则可以使用delete和default关键字
> default只有6个函数可能使用(默认函数)<br/>delete可用于任何函数，使用delete限制某些类型的操作
```c++
class Use
{
  Use() = default;    // 使用默认构造函数
  Use(Use&) = delete; // 禁用默认复制构造函数
}
```
###### 委托构造函数
* 允许在构造函数中使用之前定义的构造函数进行初始化等操作
```c++
class Use
{
  private:
    int n_;
    int k_;
  public:
    Use();
    Use(int n);
    Use(int n, int k);
}
// 注意定义的顺序，被委托的函数要放前面
Use::Use(int n) : n_(n) {k_ = 0;}
Use::Use(int n, int k): Use(10){}
Use::Use() : Use(10, 20)  // 委托了Use(int n, int k)函数
```
###### 继承构造函数
* 使派生类能够使用基类的构造函数(默认构造函数、复制构造函数和移动构造函数除外)
* 在子类的private中使用baseClass::baseClass的方式，前一个baseClass代表名称空间，后一个baseClass代表构造函数名
```c++
class base
{
  private:
    int n_;
  public:  
    base() {n_ = 0;}
    base(int n): n_(n) {};
}

// 注意，不能使用父类的默认构造函数，复制构造函数和移动构造函数
class child: public base
{
  private:
    int k_;
  public:
    using base::base;
    child() {k_=0; n_=0;} 
    child(int n): base(n) {k_=0;}
    child(int n, int k): base(n), k_(k){}
}
```
###### override & final
* 在子类使用override，将检查是否正确的重载了父类的虚方法
```c++
class base
{
  ...
  virtual int add(int, int);
  virtual double add(double, double);
}
class child: public base
{
  ...
  virtual int add(int, int) override {...}  // 不报错
  virtual double add(int, double) override {...} // 如果base没有该函数，则会报错，因为此时并非重载虚函数，而是会隐藏父类函数
}
```
* 在父类使用final，声明该函数已经是最终版本，不允许子类进行重载
```c++
class base
{
  ...
  virtual int add(int, int) final {...}
}
```


## Lambda
```c++
[] (int x) {return x%3==0;}
```
* 上述为一个用于判断是否整除3的lambda函数
* []表示无需函数名，()为参数列表，{}为操作
* 当{}内只有一句return xx时，lambda无需返回值，因为可以通过decltype自动推断；但是当{}内有多句语句，则需要使用后置类型推断
```c++
[] (int x) -> int {int y=x; return x-y;}
```
* lambda可以作为函数对象，使用于迭代器算法中
* 可以使用auto name为lambda函数命名，这样可以实现简洁调用
```c++
auto mod3 = [](int x){return x%3==0;}
```
* []使得lambda可以访问{}任何值
> 1. [i, j, ...]使得可以访问lambda中的i,j,...参数的值<br/>
> 2. [&i, &j, ...]使得可以按引用访问i,j,...<br/>
> 3. [=]使得可以访问所有参数的值<br/>
> 4. [&]使得可以按引用访问所有参数<br/>
>> ```c++
>> // 实现求取被3整除的数据个数和被13整除的数据个数
>> int cnt3 = cnt13 = 0;
>> auto func = [&](int x){cnt3+=(x%3==0); cnt13+=(x%13==0);}
>> for_each(list.begin(), list.end(), func);
>> cout<<cnt3<<cnt13<<endl;
>> ```

## 包装器
* 调用特征标——返回类型和形式参数列表定义的
* 具有相同调用特征标的函数对象、函数指针、lambda可视为是一样的function
```c++
template<class T, typename F>
T use_f(T v, F f) {return f(v);}

class Fp
{
  double z_;
  Fp(double z=1.0):z_(z){}
  double operator()(double p){return z_*p;}
}

auto Fq = [](double x){return x*x;}

double dub(double x){return 2.0*x;}

// use_f(2.0, dub)
// use_f(2.0, Fp())
// use_f(2.0, Fq)
// 上述三种方式会生成三种不同的模板实例化
```
> 1. 实例化各种函数指针、lambda、函数对象 
>> ```c++
>> // 包装器使用方法——function< returnType (params List) >
>> function<double(double)> ef1 = dub;
>> function<double(double)> ef2 = Fp();
>> function<double(double)> ef3 = Fq;
>> // use_f(2.0, ef1)
>> // use_f(2.0, ef2)
>> // use_f(2.0, ef3)
>>// 上述三种方式会生成一种模板实例化
>> ```
> 2. 直接传入生成包装器临时变量
>> ```c++
>> // 包装器使用方法——function< returnType (params List) >
>> function<double(double)> func;
>> // use_f(2.0, func(dub))
>> // use_f(2.0, func(Fp()))
>> // use_f(2.0, func(Fq))
>> // 上述三种方式会生成一种模板实例化
>> ```
> 3. 直接在模板声明中定义成包装器
>> ```c++
>> // 包装器使用方法——function< returnType (params List) >
>> template<class T>
>> T use_f(T v, function<T(T)> f){return f(v);}
>> // use_f(2.0, dub)
>> // use_f(2.0, Fp())
>> // use_f(2.0, Fq)
>> // 上述三种方式会生成一种模板实例化
>> ```

## 可变参数模板
* 使用typename...来表示长度不定且数据类型各异的参数列表；使用...name来展开参数列表
```c++
// 会导致无限递归
template<typename... Args>
void show1(Args... args)
{
  show1(args...);
}

// 不会永远递归，但是会打印格式错误，最后一个参数会多打印一个逗号
template<typename T, typename... Args>
void show2(T value, Args... args)
{
  cout<< value<< ",";
  show2(args...);
}

// 正确调用方式，通过重载一个单参数的同名函数，既能解决打印格式错误，又能终止递归
template<typename T>
void show2(T value)
{
  cout<< value<< endl;
}
```
:top:[Back to Top](#c基础知识):top:
