# SFINAE (Substitution Failure Is Not An Error)

当模板参数替换（substitution）失败时，编译器不会报错，而是把这个模板从候选集合里移除。即**模板匹配失败 ≠ 编译错误**。

```cpp
template<typename T>
void foo(T t) {
    typename T::type x;
}

void foo(int t) {
    std::cout << "int version\n";
}

foo(10); // int没有type，但是模版函数不会编译报错，只会提示`substitution 失败，忽略这个模板`
```

其核心用途是在**编译期内选择合适的模版**。
```cpp
template<typename T>
void print(T t) {
    std::cout << "generic";
}

template<typename T>
auto print(T t) -> decltype(t.size(), void()) {
    std::cout << "has size()";
}

print(10); // "generic"
print(vector<int> {}); // "has size()"
```

# SFINAE 触发阶段

```
template call
     │
     ▼
template argument deduction
     │
     ▼
substitution
     │
     ├── failure → SFINAE → remove candidate
     │
     ▼
overload resolution
     │
     ▼
template chosen
     │
     ▼
template instantiation
     │
     ├── error → compile error
     │
     ▼
generated code
```

其中SFINAE是substitution阶段的术语。

> [!NOTE]
> 比如对于下面的例子，当我们使用`function<int>`的时候，只检查是否匹配`void function(T t)`以及其他也许存在的`decltype`，并不看具体实现，所以是没有出发SFINAE的。但是因为int没有type，所以会在 模版实例化 的阶段出错。
> ```cpp
> template<typename T>
> void function(T t) {
>     T::type x;
> }
> ```

# `decltype` 会触发SFINAE

对于下面的模版函数，如果类型不含size()方法则会SFINAE。具体请参考 [decltype](./decltype.md)。
```cpp
template<typename T>
auto function(T t) -> decltype(t.size(), void()) {...}
```

# `enable_if`

请参考 [enable_if](./enable-if.md)。

# `concept`

请参考 [concept](./concepts-and-requires.md)。
