# .pyc

.pyc is a Python compiled bytecode cache file.

When Python runs a .py file, it does not directly interpret the source code line by line every time. It will first compile the source code into bytecode bytecode, and then execute it by the Python virtual machine.

To make the next import faster, Python will cache this bytecode into a .pyc file.

Assuming that a `utils.py` is used by another module, a `__pycache__/utils.cpython-312.pyc` may be generated.

---

# .pyd / .so

These two are Python C/C++ extension modules:
* .pyd → Windows
* .so → Linux

It is essentially a dynamic link library that can be imported directly by Python.

For example, write a `utils.cpp` in C/C++, and then build it into `utils.pyd`/`utils.so`.
```py
# main.py
import utils
```
This extension module will be called directly.

---

# .pyi

Because for .pyd / .so, its essence is a bunch of binary code that humans cannot understand. If we want to call these C/C++ methods in Python, we do not know their function signatures and other information, making it very difficult to use.

`.pyi` is a Python type hint stub.

For example, we have a `math.pyd` and a `math.pyi`:
```py
# math.pyi
def add(a: int, b: int) -> int: ...
def function(a: str) -> str | None: ...
```
So when `import math` is called in our code:
* IDE can provide code prompts
* mypy can provide type checking
* Code jump can see declarations and comments

> [!NOTE]
> .pyi does not need to be implemented, only declarations and corresponding comments need to be provided.
