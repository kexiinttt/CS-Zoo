# Module vs Package

* module → A `.py` file is usually a module
* package → A package can be understood as "a directory containing multiple modules"

```text
mypkg/
    __init__.py
    a.py
    b.py
```

* `mypkg` is package
* `mypkg/*.py` is module

---

# import

`import foo` will roughly do the following:

1. First check whether `foo` has been loaded in `sys.modules`
2. If not, search according to the module search path.
3. After finding the module file, load and execute the top-level code of the module
4. Create module object
5. Put the module object into `sys.modules` (can be regarded as a key-value pair)
6. Bind the name `foo` in the current scope to this module object

> [!NOTE]
> Import not only reads files, but also executes the top-level code of the module and has a caching mechanism.
> ```py
> # utils.py
> print("utils.py")
> x = 10
> def func(): ...
> ```
> These three statements will be executed when imported, and note that for function/class definitions, the declaration statements themselves will be executed, but the internal specific implementation will only run when called.
>
> At the same time, the caching mechanism ensures that modules that have already been loaded will not be imported repeatedly (because loading means executing the top-level code again). The loaded ones will be placed in `sys.modules` so they can be read directly.

---

# `if __name__ == "__main__"`

- If the file is run directly, `__name__ == "__main__"`
- If the file is imported as a module, `__name__` is usually the module name

Separate "reusable module code" and "script execution entry" to avoid mistaken execution of entry logic during import.

---

# `__init__.py`

`__init__.py` will be executed when the package is imported. Its main function is to perform package-level initialization and control externally exposed interfaces.

```python
# mypkg/__init__.py
from .a import Foo
from .b import bar
```


```python
from mypkg import Foo, bar
```

---

# Relative/Absolute import

* Absolutely import `from mypkg.subpkg.mod import func`
* Relative import `from .utils import func`

## Relative import errors

For example, directory structure:

```text
project/
    mypkg/
        __init__.py
        utils.py
        module.py
```
```py
# module.py
from .utils import helper
```


```bash
$> python mypkg/module.py

ImportError: attempted relative import with no known parent package
```

The reason is: relative imports are resolved based on the package context. `python mypkg/module.py` will run this file directly as a top-level script. At this time, it is no longer the module in the package `mypkg.module`, but just a script. Therefore, `.` in `from .utils import helper` cannot find the "parent package".

```bash
python -m mypkg.module
```

This way Python knows that it is running as the module `mypkg.module`, and the relative import will work normally.

---

# How to find modules in Python

First check whether there is any cache in `sys.modules` that has been loaded. If not, search in the order of `sys.path`.

Note that the order of `sys.path` is important. In theory, you can use `sys.path.insert(0, "/my/path")` to first search for custom paths.

---

# CircularImport

```python
# a.py
from b import func_b

def func_a():
    ...
```


```python
# b.py
from a import func_a

def func_b():
    ...
```

Because the top-level code is executed when the module is imported. If `a` has not been loaded, import `b`; `b` will go back and import `a`. At this time, `a` may only be in a "partial initialization state", causing the name to not exist yet.

```text
ImportError: cannot import name ... from partially initialized module ...
```

This can be solved by:
- Reconstruct module boundaries and split public dependencies into the third module
- Delayed import (local import) → For example, import within a function
