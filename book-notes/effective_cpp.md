# explicit

编译器会对**只有一个不确定参数**的**构造函数**采用隐式类型转换

```cpp
class Object {
public:
    int _val;
    int _size;
    Object(int val, int size = 10) : _val(val), _size(size) {};
}

Object o = 10;  // ok, 10 被隐式转换为Object类型
```

通过加`explicit`能避免这个问题

```cpp
class Object {
public:
    int _val;
    int _size;
    explicit Object(int val, int size = 10) : _val(val), _size(size) {};
}

Object o = 10;  // error
Object o(10);   // ok
```

---

# decltype & 返回类型后置（trailing-return-type)

`auto`必须要求右侧有内容且 cannot use in non-static member variable

```cpp
auto a; // error

class Object {
public:
    static auto size = 10;
    auto num = 20;  // error
}
```

`decltype(expr)`更加灵活，且可以和`auto`配合用于 **返回类型后置（trailing-return-type)**

> 注意如果expr是函数，那么需要包含函数名，括号以及内部的参数

```cpp
int& func(int a, int b);
int&& func(void);

decltype(10+20) a = 20;     // int a = 20
decltype(func(10, 20)) b;   // int& b
decltype(func()) c;         // int&& c

template<typename T, typename K>
auto add(T a, K b) -> decltype(a+b) {
    return a + b;
}

int a = 10;
double b = 10.0;
bool c = true;
add(a, b);  // decltype(10 + 10.0) -> double
add(a, c);  // decltype(10 + true) -> int
```

---

# enum

```cpp
enum Type {
    INT,
    DOUBLE,
    BOOL
}

// 根据 type 选择输出的 res 类型
switch (type) {
    case Type::INT {
        int res = val;
        break;
    }
    case Type::DOUBLE {
        double res = val;
        break;
    }
    case Type::BOOL {
        bool res = val;
        break;
    }
    default {
        string res = val;
        break;
    }
}
```

---

#  使用using代替typedef

```cpp
typedef uint unsigned int;
using uint = unsigned int;  // 通过赋值更容易看出结构

// ----

template<typename T>
using map_str_t = map<string, T>;

map_str_t<int> map1;

// ----

typedef void (*func_t)(int, int);
using func_t = void (*)(int, int);  // 通过赋值更容易看出结构
```

---

# 常使用 const !

1. 普通变量加const代表元素不可被更改
2. 函数参数加const代表该函数不可修改传入参数的内容
3. 类内方法加const代表该方法不会修改类的成员变量

## const 指针

> `const char* p` &rarr; const data
> `char* const p` &rarr; const pointer

## const 迭代器

> `const vector<int>::iterator` &rarr; 表明迭代器只能指向某一个位置,不可移动
> `vector<int>::const_iterator` &rarr; 表明迭代器指向的内容不可变,但是可以移动
> ```cpp
> const vector<int>::iterator it1 = nums.begin();
> *it1 = 10;    // ok
> it1++;        // error
>
> vector<int>::const_iterator it2 = nums.begin();
> *it2 = 10;    // error
> it2++;        // ok
> ```

## mutable

标记为mutable的变量可以在const函数中被改变

```cpp
class Object {
public:
    char* buffer;
    mutable size_t length;
    mutable bool is_valid;
    ...
    size_t size() const {
        if (!is_valid) {
            length = strlen(buffer);
            is_valid = true;
        }
        return length;
    }
}
```

---

# static

1. static变量的生命周期是整个程序, 且只会在第一次被执行时初始化
2. class 的 static 变量是类共享而非某个对象独有
3. class 的 static 方法只能使用 static 变量或其他 static 方法

```cpp
class Object {
public:
    static int common;
    static void update() {
        return ++common;
    }
}

int Object::common = 0;
```

当static变量在多个不同的作用域内, 且某些static变量需要使用其他static变量的时候, 会遇到问题. 因为多个程序编译和执行的顺序无法预计和固定, 所以可能出现某个static需要使用另一个没有初始化过的static变量, 那么就会报错.

```cpp
// file.cpp
class FileSystem {
    ...
}
extern FileSystem fs;
```

```cpp
// local.cpp
class Local {
    Local () {
        int size = fs.get_size();   // 要求一定要初始化过fs
    }
}
```

解决的方法就是把static变量变为non-static函数的返回值, 这样只有我们call函数的时候才会涉及到static的初始化, 但是函数调用的顺序我们是可以控制的, 因此我们可以控制static的初始化顺序.

```cpp
// file.cpp
class FileSystem {
    ...
}

FileSystem& init_fs() {
    static FileSystem fs;
    return fs;
} 
```

```cpp
// local.cpp
class Local {
    Local () {
        int size = init_fs().get_size();    // ok, 一定会调用函数, 从而保证fs一定被先初始化过
    }
}
```

---

# 构造函数

```cpp
class Object {
public:
    int _val;
    
    Object();                               // default constructor
    Object(int val);                        // converting constructor 
    Object(const Object& rhs);              // copy constructor
    Object& operator=(const Object& rhs);   // copy assignment
    Object(const Object&& rhs);             // move constructor
}
```

每当新创建一个对象的时候一定调用constructor.

```cpp
Object a;
Object b(a);    // copy constructor
Object c = b;   // copy constructor, 因为c是新的, 只能被构造
c = a;          // copy assignment, 因为c已经存在, 此处是赋值
```

> 在调用构造函数时, 更推荐使用 member initialization list 进行“初始化”而非“赋值”. 另一个有趣的事实是: 初始化的顺序是固定的, 先父类再子类, class内的变量按照定义顺序初始化.

## copy assignment

注意: operator= 中需要注意先判断是否和自身相等(证同)

## 子类copy函数需要记得copy父类的内容

在子类copy assignment或者copy constructor中, 由于某些成员是父类的, 所以我们也不能忘了他们.

```cpp
//  父类
class Base {    
public:
    ...
    Base(const Base& rhs);
    Base& operator=(const Base& rhs);
    ...
private:
    int base_var;
    struct base_struct;
};

// 子类
class Derive : Base {   
public:
    ...
    Derive(const Derive& rhs);
    Derive& operator=(const Derive& rhs);
    ...
private:
    int derive_var;
};
```

```cpp
Derive::Derive(const Derive& rhs) : Base(rhs) { // 一定要记得调用Base的copy constructor
    derive_var = ...;
}

Derive& Derive::operator=(const Derive& rhs) {
    if (&rhs == this) {     // 证同
        return *this;
    }
    Base::operator=(rhs);   // 一定要记得调用Base的copy assignment
    derive_var = ...;
}
```

---

# 不要在class内定义复杂函数(实现复杂函数)

class中定义的内容默认为inline, 对于复杂函数, inline会降低效率. 所以复杂函数可以在class中声明, 在外部定义.

```cpp
class Object {
public:
    // 对于构造函数, 析构函数, __str__, getter / setter可以在类内定义(默认为inline)
    Object() =default;
    Object(const Object& rhs) { num = rhs.num; }
    void complex_function();    // 复杂函数只声明
private:
    int num;
};

void Object::complex_function() {   // 在外面定义, 避免生成inline
    ...
}
```

## inline

* inline只对函数定义有效, 函数声明不需要加inline
* inline推荐使用在非常简易的函数上, 相当于把函数调用替换成函数体
  ```cpp
  inline void print(int num) { cout << num << endl; }
  void function() {
    print(10);  //  cout << 10 << endl;
    print(20);  //  cout << 20 << endl;
  }
  ```
* inline只是一个推荐, 编译器是否做优化是不一定的
  > 如果inline函数包含loop, recursion, static, switch等复杂内容, 编译器不会把其优化成inline

## 为什么需要inline?
函数的调用会涉及到从内存指定位置load到栈中, 执行然后弹出栈. 这些过程需要耗费额外的时间. 为了避免一些很简单的小函数频繁的入栈出栈, 我们使用inline去优化.

## 为什么不能滥用inline?
inline的优化是通过占用额外的空间实现的, 比如说有一个function, 如果不是inline的, 我们只需要存储一份定义即可, 但是如果是inline的, 我们需要将每个调用这个函数的地方都替换成函数定义, 会增加很多额外的空间.

如果inline函数很复杂, 其内部执行时间 > 入栈出栈的切换时间, 那么没有必要损耗额外的空间.

## inline vs #define
`#define`是预编译阶段直接用文本替换, inline是在编译的时候将函数体插在调用的地方.

---

# default函数 & delete函数

C++ 11新特性

## =default

只能用于没有参数的类特殊函数, 比如默认构造函数以及析构函数, 编译器将为显式声明的 "=default"函数自动生成函数体以获得更好的效率.

```cpp
class Object {
public:
    Object() = default;
    ...
    ~Object() = default;
}
```

## =delete

* 禁用类的某些转换构造函数, 从而避免不期望的类型转换
* 禁用copy相关
* 禁用某些用户自定义的类的 new 操作符

```cpp
class Uncopy {
public:
    ...
    // 不允许从int强制转换, 但是double可以
    Object(int) = delete;
    Object(double);
    // 禁止拷贝
    Object(const Object&) = delete;
    Object& operator=(const Object&) = delete;
    // 禁止new
    void* operator new(size_t) = delete;
    void* operator new[](size_t) = delete;
}
```

---

# virtual & =0

* 没有 virtual &rarr; 根据引用类型或指针类型选择方法; 有 virtual &rarr; 根据引用和指针指向的对象选择方法
* 在基类中将派生类会重新定义的方法设置为virtual, 这样在派生类中重定义时会自动生成virtual
* virtual关键字只用于类声明中, 而不用于定义, 在定义时通过className加以区分即可
* **如果某个类是基类, 其会有子类, 那么需要使用虚析构函数(类内至少有一个virtual函数的时候才需要定义virtual析构函数)**, 否则建议不要使用virtual析构函数, 因为会增加额外的虚函数表开销

> polymorphic base class 应该声明virtual析构函数, 如果class带有任何virtual函数, 他就应该拥有virtual析构函数; 如果某个 base class 不是为了 polymorphic, 就不需要virtual析构函数.

> 如果使用了virtual, 对象会额外保存一些信息 &rarr; vptr (virtual table pointer) 会指向一个函数指针构成的数组 vtbl (virtual table). 每个带有virtual的class都会有对应的vtbl. 当调用virtual函数的时候, 实际被调用的函数取决于vptr所指向的vtbl.

> 不要在构造函数或者析构函数中调用virtual方法

`virtual void func() = 0;`代表纯虚函数, 有纯虚函数的class不能被实例化, 只能被继承.

---

# override vs overload

override是子类继承父类的virtual函数, overload是同名但函数签名不同.

---

# 按值传递 vs 按引用传递

如果按值传递, 那么会调用copy constructor, 效率低, 所以对于用户自定义的class以及STL容器, 一般使用按引用传递.

但是对于**内置类型**(int, double, bool等), **函数对象**, **指针**以及**STL::iterator**, 一般推荐按值传递.

---

# 按值返回 vs 按引用返回

1. 如果返回的是class, 那么按值返回一个副本, 会默认调用copy constructor去赋值, 效率低
2. 按引用返回的时候一定保证返回的东西不会在函数退出后被解构, 比如static和放在heap里的
3. 按引用返回的内容可以作为左值, 按值返回的只能做右值. 所以如果需要连续调用, 一定要按引用返回

> * 比如对于string类而言, `string& operator=(const string& rhs)` 返回的是引用, 因为可能有 `str_a = str_b = str_c`
> * 但是 `string operator+(const string& rhs)` 返回的是值, 因为调用的方法是 `str_res = str_a + str_b`, 即会赋值给一个新的

```cpp
// return by value
vector<int> return_by_value(int a, int b) {
    vector<int> res = {a, b};
    return res;
}

// return by reference
vector<int>& return_by_reference(int a, int b) {
    vector<int>* res = new vector<int> {a, b};
    return *res;
}

// return by pointer
vector<int>* return_by_pointer(int a, int b) {
    vector<int>* res = new vector<int> {a, b};
    return res;
}

vector<int> value = return_by_value(1, 2);  // ok
return_by_value(1, 2)[0] = 3;               // error

vector<int> reference = return_by_reference(1, 2);  // ok
return_by_reference(1, 2)[0] = 3;                   // ok, [3, 2]

vector<int>* pointer = return_by_pointer(1, 2);     // ok
(*return_by_pointer(1, 2))[0] = 3;                  // ok, [3, 2]
```

> 对于assignment赋值函数(eg. =, +=, etc), 一定按引用返回

---

# smart pointer

smart pointer的好处在于被指向的资源不需要手动去delete, 会自动调用析构函数(意味着最好不要指向C风格内容, 否则没有正确的析构函数可以调用)

> 因为是指针, 使用 -> 去访问指向资源的方法和成员; 使用 . 去访问智能指针自带的方法.

## auto_ptr / unique_ptr

只能有一个智能指针指向同一个资源, 区别在于:
1. auto_ptr 可以copy, 结果就是老的变为null, 新的获得控制权
2. unique_ptr 不能copy只能move, move之后老的自然被销毁

```cpp
auto_ptr<Object> p1(new Object(1));
auto_ptr<Object> p2(new Object(2));
p1->show(); // Obj(1)
p2->show(); // Obj(2)

p2 = p1;

p1->show(); // null
p2->show(); // Obj(1)
```

```cpp
unique_ptr<Object> p1(new Object(1));
unique_ptr<Object> p2(new Object(2));
p1->show(); // Obj(1)
p2->show(); // Obj(2)

p2 = p1;    // error
p2 = move(p1);

p1->show(); // null
p2->show(); // Obj(1)
```

## shared_ptr

可以有多个智能指针指向同一个资源, 会有一个计数器记录某个资源被多少个指针指向, 当指针数量为0, 就会消除资源.

```cpp
shared_ptr<Object> p1(new Object(1));
shared_ptr<Object> p2(new Object(2));
shared_ptr<Object> p3 = p1;

cout << p1.use_count(); // 2 (p1, p3)
cout << p2.use_count(); // 1 (p2)
```

### 在函数中传递shared_ptr

因为本身是一个指针, 所以都是按值传递 (对于shared_ptr指向的内容, 相当于按指针传递)

### deleter

有时候shared_ptr某个资源利用率为0的时候, 我们并不想删除它, 而是做额外操作. 比如说我们有一个锁, 资源上没有锁的时候, 我们应该unlock而不是删除资源.

```cpp
void unlock(Object* obj) {
    cout << obj << " is unlocked" << endl;
};

int main() {
    ...
    shared_ptr<Object> p(new Object(1), unlock); // 计数器为0时调用unlock而非调用Object析构函数
}   // Object(1) is unlocked
```

这里要求删除器：
1. 可以是function object, 函数指针, Lambda表达式, bind/functor等等均可.
2. **返回值是void，参数是shared_ptr类型的指针**
3. 从形参看出, 删除器以传值的方式传入, 所以要求**删除器要是可拷贝的**
4. 删除器不要抛出异常

## 注意只会调用`delete`而非`delete[]`

因此使用智能指针指向C类型的数组是一个bad idea.

```cpp
unique_ptr<int> p(new int[10]); // bad
unique_ptr<vector<int>> p(new vector<int>);
```

---

# 注意member函数在二元运算符中的顺序

假设Object类允许做隐式类型转换, 且重载了operator*.

```cpp
class Object {
public:
    ...
    Object(int num) : _num(num) {}
    ...
    Object& operator*(const Object& rhs) {_num *= rhs._num;}
private:
    int _num;
}
```

当执行的时候会发生下面的情况:

```cpp
int num = 10;
Object obj(10);

obj * num;  // ok, 相当于 obj.operator*(Object(num))
num * obj;  // error, 2.operator*(obj), 但是int的operator*并不支持参数是Object
```

解决的方法有两种:
1. 加explicit彻底不允许隐式类型转换(但是很多时候这种简单的类型转换用于计算是ok的)
2. 额外定义一个non-member函数`const Object operator*(const Object& lhs, const Object& rhs)`

---

# 偏特化 & 全特化 (specialization)

**类模版可以全特化&偏特化, 函数模版只能全特化**.

## 全特化

全特化的模板参数列表应当是空的`template<>`, 并且应当给出"模板实参"列表`class Object<type>` or `function<type>()`

```cpp
template<>
class Object<int> {
    ...
}

template<>
void function<int> () {
    ...
}
```

## 偏特化

偏特化也是为了给自定义一个参数集合的模板，但偏特化后的模板需要进一步的实例化才能形成确定的签名.

```cpp
template<typename T>
class Object<T> {
    ...
}

// error
template<typename T>
void function(T a, T b) {

}
```

实际上函数只需要通过重载即可实现偏特化.

```cpp
void function(int, int);
void function(double, double);
```

但是也有例外, 就是std中的函数不允许重载, 比如说`std::swap`. 其实现非常直接, 就是利用一个temp变量实现交换. 但是对于用户自定义类型, 有时候不需要copy, 只需要交换指针即可, 那么用户需要自定义swap.

假设我们的class是模版类, 因为成员是private, 所以肯定需要在class中定义一个`Object<T>::swap`函数. 为了让用户能够仍然保持使用`swap(var1, var2)`的格式, 我们需要定义一个non-member的`swap(Object<T>, Object<T>)`函数, 但是我们又不能偏特化, 所以只能重载, 但是又不能在std重载, 我们只能通过namespace来实现.

```cpp
namespace MySpace{
    template<typename T>
    class ObjectImpl {
        ...
    };

    template<typename T>
    class Object {
    public:
        ...
        void swap(const Object<T>& rhs) {
            using std::swap;        // 为何使用这个? 看下面的分析
            swap(ptr, rhs.ptr);
        }
    private:
        ObjectImpl<T>* ptr;
    };

    // ok, 没有偏特化, 又是在非std上重载
    template<>
    void swap(const Object<T>& a, const Object<T>& b) {
        a.swap(b);
    }
}
```

> 为何使用`using std::swap`? 因为这样的话, 我们交换ptr的swap函数就能够根据传入的参数动态选择最合适的, 如果是内置类型, 那么调用`std::swap`, 如果是非内置类型, 那么调用`MySpace::swap`.
> 
> 如果我们直接使用`std::swap` / `MySpace::swap`会丧失编译器帮我们适配的机会, 容易造成问题.

---

# 转型(cast)

## `const_cast<T>`

 用来修改变量的const属性.

## `reinterpret_cast<T>`

较少使用, 本质是编译器的指令, 用于位的简单重新解释, 可以进行完全不相关的内容之间的转换.

```cpp
char* p = "hello world";                // 假设这个string的地址是0x00000011
int num = reinterpret_cast<int> (p);    // 把0x00000011用int表示成3
```

## `dynamic_cast<T>`

* 只能用于指针或者引用
* 只能用于含有虚函数的类, 即只能作用于子类父类转换
* dynamic_cast转换操作符在执行类型转换时首先将检查能否成功转换，如果能成功转换则转换之，如果转换失败，如果是指针则返回一个0值，如果是转换的是引用，则抛出一个bad_cast异常。

## `static_cast<T>`

1. 父类子类转换
   1. 向上转换 &rarr; 派生类转为基类是安全的
   2. 向下转换 &rarr; 基类转为派生类是不安全的
2. 基本数据类型转换 & 指针类型转换
3. 任何类型表达式转为void

---

# 悬垂指针(dangling pointer)

 如果返回的是handler(即pointer, reference, iterator)等, 然后这个handler指向某个类型的内部, 那么有可能出现对象被销毁, 但是该返回值仍然存在.