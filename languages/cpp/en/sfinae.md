# SFINAE (Substitution Failure Is Not An Error)

When template parameter substitution fails, the compiler does not report an error, but removes the template from the candidate set. That is, **template matching failure ≠ compilation error**.

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

Its core purpose is to select the appropriate template during compilation time.
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

# SFINAE trigger phase

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

Where SFINAE is the term for the substitution stage.

> [!NOTE]
> For example, in the following example, when we use `function<int>`, we only check whether it matches `void function(T t)` and other possible `decltype`, and do not look at the specific implementation, so SFINAE is not started. But because int has no type, an error will occur during the template instantiation stage.
> ```cpp
> template<typename T>
> void function(T t) {
>     T::type x;
> }
> ```

# `decltype` will trigger SFINAE

For the following template function, if the type does not contain a size() method, SFINAE will occur. Please refer to [decltype](./decltype.md) for details.
```cpp
template<typename T>
auto function(T t) -> decltype(t.size(), void()) {...}
```

# `enable_if`

Please refer to [enable_if](./enable-if.md).

# `concept`

Please refer to [concept](./concepts-and-requires.md).
