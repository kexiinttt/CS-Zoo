# .pyc

.pyc 是 Python 编译后的字节码缓存文件。

Python 运行 .py 文件时，不是每次都直接一行一行解释源码。它会先把源码编译成 bytecode 字节码，然后由 Python 虚拟机执行。

为了下次导入更快，Python 会把这个字节码缓存成 .pyc 文件。

假设有一个`utils.py`在被另一个module使用，那么可能会生成一个`__pycache__/utils.cpython-312.pyc`。

---

# .pyd / .so

这两个是 Python C/C++ 扩展模块：
* .pyd &rarr; Windows 
* .so &rarr; Linux

它本质上是一个动态链接库，可以被 Python 直接 import。

比如用C/C++写了一个`utils.cpp`，然后build之后成`utils.pyd`/`utils.so`。
```py
# main.py
import utils
```
会直接去调用这个扩展模块。

---

# .pyi

因为对于 .pyd / .so，其本质是一堆人类看不懂的二进制代码。如果想在Python中调用这些 C/C++ 方法，我们并不知道他们的函数签名等各种信息，导致非常难使用。

`.pyi` 就是一个 Python 的类型提示 stub。

比如我们有一个 `math.pyd`，同时有一个 `math.pyi`：
```py
# math.pyi
def add(a: int, b: int) -> int: ...
def function(a: str) -> str | None: ...
```
那么当我们的代码里调用`import math`的时候：
* IDE 能提供代码提示
* mypy 可以提供类型检查
* 代码跳转能看到声明和注释

> [!NOTE]
> .pyi 不需要实现，只需要提供声明和对应的注释即可。
