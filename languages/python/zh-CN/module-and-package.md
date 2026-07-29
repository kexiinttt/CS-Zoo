# Module vs Package

* module &rarr; 一个 `.py` 文件通常就是一个 module
* package &rarr; 一个 package 可以理解成“装着多个 module 的目录”

```text
mypkg/
    __init__.py
    a.py
    b.py
```

* `mypkg` 就是 package
* `mypkg/*.py` 就是 module

---

# import

`import foo`大致会做这些事：

1. 先检查 `sys.modules` 里是否已经加载过 `foo`
2. 如果没有，去按模块搜索路径查找
3. 找到模块文件后，加载并执行模块顶层代码
4. 创建模块对象
5. 把模块对象放进 `sys.modules` (可以视为一个key-value pair)
6. 把当前作用域里的名字 `foo` 绑定到这个模块对象

> [!NOTE]
> import 不仅仅是读文件，还会执行模块顶层代码，而且有缓存机制。
> ```py
> # utils.py
> print("utils.py")
> x = 10
> def func(): ...
> ```
> 这三个语句都会在导入的时候执行，且注意对于函数/类定义，其声明语句本身会被执行，但是内部的具体实现只会在被调用的时候才运行。
>
> 同时缓存机制保证了不会反复import已经load过的module（因为load意味着又要执行一遍顶层代码）。load过的会放在 `sys.modules` 所以可以直接读。

---

# `if __name__ == "__main__"`

- 如果该文件被直接运行，`__name__ == "__main__"`
- 如果该文件被当作模块导入，`__name__` 通常是模块名

把“可复用模块代码”和“脚本执行入口”分开，避免 import 时误执行入口逻辑。

---

# `__init__.py`

`__init__.py` 会在 package 被导入时执行。主要作用是可以做package级别的初始化，以及控制对外暴露的接口

```python
# mypkg/__init__.py
from .a import Foo
from .b import bar
```

```python
from mypkg import Foo, bar
```

---

# Relative / Absolute import

* 绝对导入 `from mypkg.subpkg.mod import func`
* 相对导入 `from .utils import func`

## 相对导入的错误

比如目录结构：

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

原因是：相对导入是**基于 package 上下文**解析的。`python mypkg/module.py` 会把这个文件当成一个顶层脚本直接运行，这时它不再是 `mypkg.module` 这个包内模块，而只是一个脚本，于是 `from .utils import helper` 里的 `.` 就找不到“父 package”了。

```bash
python -m mypkg.module
```

这样 Python 知道它是在以 `mypkg.module` 这个模块的身份运行，相对导入就能正常工作。

---

# Python 如何找模块

先查看`sys.modules`中有没有缓存已经load过的，如果没有则按`sys.path`的顺序查找。

注意`sys.path`的顺序很重要，理论上你可以通过`sys.path.insert(0, "/my/path")`来优先查找自定义path。

---

# Circular Import

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

因为模块导入时要执行顶层代码。如果 `a` 还没加载完，就去 import `b`；`b` 又回头 import `a`，此时 `a` 可能只是“部分初始化状态”，导致名字还不存在。

```text
ImportError: cannot import name ... from partially initialized module ...
```

可以通过以下方法进行解决：
- 重构模块边界，拆公共依赖到第三个模块
- 延迟导入（局部导入）&rarr; 比如函数内import
