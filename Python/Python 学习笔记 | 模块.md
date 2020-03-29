[TOC]

## 认识模块和包

在 Python 中，一个.py 文件就称之为一个模块（Module）。


使用模块有什么好处？

最大的好处是大大提高了代码的可维护性。其次，编写代码不必从零开始。当一个模块编写完毕，就可以被其他地方引用。我们在编写程序的时候，也经常引用其他模块，包括 Python 内置的模块和来自第三方的模块。

使用模块还可以避免函数名和变量名冲突。相同名字的函数和变量完全可以分别存在不同的模块中，因此，我们自己在编写模块时，不必考虑名字会与其他模块冲突。但是也要注意，尽量不要与内置函数名字冲突。点[这里](http://docs.python.org/3/library/functions.html)查看 Python 的所有内置函数。

你也许还想到，如果不同的人编写的模块名相同怎么办？为了避免模块名冲突，Python 又引入了按目录来组织模块的方法，称为包（Package）。

你也许还想到，如果不同的人编写的模块名相同怎么办？为了避免模块名冲突，Python 又引入了按目录来组织模块的方法，称为包（Package）。

举个例子，一个 `abc.py` 的文件就是一个名字叫 `abc` 的模块，一个 `xyz.py` 的文件就是一个名字叫 `xyz` 的模块。

现在，假设我们的 `abc` 和 `xyz` 这两个模块名字与其他模块冲突了，于是我们可以通过包来组织模块，避免冲突。方法是选择一个顶层包名，比如 `mycompany`，按照如下目录存放：

```ascii
mycompany
├─ __init__.py
├─ abc.py
└─ xyz.py
```

引入了包以后，只要顶层的包名不与别人冲突，那所有模块都不会与别人冲突。现在，`abc.py` 模块的名字就变成了 `mycompany.abc`，类似的，`xyz.py` 的模块名变成了 `mycompany.xyz`。

请注意，每一个包目录下面都会有一个`__init__.py` 的文件，这个文件是必须存在的，否则，Python 就把这个目录当成普通目录，而不是一个包。`__init__.py` 可以是空文件，也可以有 Python 代码，因为`__init__.py` 本身就是一个模块，而它的模块名就是 `mycompany`。

类似的，可以有多级目录，组成多级层次的包结构。比如如下的目录结构：

```ascii
mycompany
 ├─ web
 │  ├─ __init__.py
 │  ├─ utils.py
 │  └─ www.py
 ├─ __init__.py
 ├─ abc.py
 └─ utils.py
```

文件 `www.py` 的模块名就是 `mycompany.web.www`，两个文件 `utils.py` 的模块名分别是 `mycompany.utils` 和 `mycompany.web.utils`。

**自己创建模块时要注意命名，不能和 Python 自带的模块名称冲突。例如，系统自带了 sys 模块，自己的模块就不可命名为 sys.py，否则将无法导入系统自带的 sys 模块。**

## 使用模块

Python 本身就内置了很多非常有用的模块，只要安装完毕，这些模块就可以立刻使用。

我们以内建的 `sys` 模块为例，编写一个 `hello` 的模块：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

' a test module '

__author__ = 'Michael Liao'

import sys

def test():
    args = sys.argv
    if len(args)==1:
        print('Hello, world!')
    elif len(args)==2:
        print('Hello, %s!' % args[1])
    else:
        print('Too many arguments!')

# 当模块是被调用执行的，__name__ 变量取值为模块的名字；
# 当模块是直接执行的，则该变量取值为：__main__
if __name__=='__main__': 
    test()
```

第 1 行和第 2 行是标准注释，第 1 行注释可以让这个 `hello.py` 文件直接在 Unix/Linux/Mac 上运行，第 2 行注释表示.py 文件本身使用标准 UTF-8 编码；

第 4 行是一个字符串，表示模块的文档注释，任何模块代码的第一个字符串都被视为模块的文档注释；

第 6 行使用`__author__`变量把作者写进去，这样当你公开源代码后别人就可以瞻仰你的大名；

```python
import sys
```

导入 `sys` 模块后，我们就有了变量 `sys` 指向该模块，利用 `sys` 这个变量，就可以访问 `sys` 模块的所有功能。

`sys` 模块有一个 `argv` 变量，用 list 存储了命令行的所有参数。`argv` 至少有一个元素，因为第一个参数永远是该.py 文件的名称，例如：

运行 `python3 hello.py` 获得的 `sys.argv` 就是 `['hello.py']`；

运行 `python3 hello.py Michael` 获得的 `sys.argv` 就是 `['hello.py', 'Michael]`。

最后，注意到这两行代码：

```python
if __name__=='__main__':
    test()
```

当我们在命令行运行 `hello` 模块文件时，Python 解释器把一个特殊变量`__name__`置为`__main__`，而如果在其他地方导入该 `hello` 模块时，变量`__name__`的值是模块的名字，`if` 判断将失败，因此，这种 `if` 测试可以让一个模块通过命令行运行时执行一些额外的代码，最常见的就是运行测试。

### 作用域

正常的函数和变量名是公开的（public），可以被直接引用

类似`__xxx__`这样的变量是特殊变量，可以被直接引用，但是有特殊用途，比如上面的`__author__`，`__name__`就是特殊变量，`hello` 模块定义的文档注释也可以用特殊变量`__doc__`访问，我们自己的变量一般不要用这种变量名

类似`_xxx` 和`__xxx` 这样的函数或变量就是非公开的（private），不应该被直接引用。之所以我们说，private 函数和变量 “不应该” 被直接引用，而不是 “不能” 被直接引用，是因为 Python 并没有一种方法可以完全限制访问 private 函数或变量，但是，从编程习惯上不应该引用 private 函数或变量。

## import 和 from...import 

### 一、import 模块名

import 首次导入模块发生了 3 件事：

1. 以模块为准创造一个模块的名称空间
2. 执行模块对应的文件，将执行过程中产生的名字都丢到模块的名称空间
3. 在当前执行文件中拿到一个模块名

模块的重复导入会直接饮用之前创造好的结果，不会重复执行模块的文件

### 二、from 模块名 import 具体的功能

from...import... 首次导入模块发生了 3 件事：

1. 以模块为准创造一个模块的名称空间
2. 执行模块对应的文件，将执行过程中产生的名字都丢到模块的名称空间
3. 在当前执行文件的名称空间中拿到一个名字，该名字直接指向模块中的某一个名字，意味着可以不用加任何前缀而直接使用

- 优点：不用加前缀，代码更加精简
- 缺点：容易与当前执行文件中名称空间中的名字冲突

### demo

```shell
.
└── import_demo
    ├── pkg1
    │   ├── __init__.py
    │   ├── a.py
    │   ├── b.py
    │   └── c.py
    ├── pkg3
    │   ├── __init__.py
    │   ├── x.py
    │   └── y.py
    ├── pkg4
    │   ├── __init__.py
    │   ├── k.py
    │   └── m.py
    └── pkgtest
        ├── __init__.py
        └── test.py
```

```python
####################
# pkg1/__init__.py
####################
"""
请注意，每一个包目录下面都会有一个__init__.py 的文件，这个文件是必须存在的，否则，Python 就把这个目录当成普通目录，
而不是一个包。__init__.py 可以是空文件，也可以有 Python 代码，
因为__init__.py 本身就是一个模块，而它的模块名就是所在包的目录名
"""

init_int = 0

def init_func():
    print("this is init_func")


####################
# pkg1/a.py
####################
int_a = 1

def a_func():
    print("this is a_func")
    
# 当模块是被调用执行的，__name__ 变量取值为模块的名字；
# 当模块是直接执行的，则该变量取值为：__main__
print('__name__ = ', __name__)

if __name__ == '__main__':
    a_func()

####################
# pkg1/b.py
####################
int_b = 2

def b_func():
    print("this is b_func")

####################
# pkg1/c.py
####################
int_c = 3

def c_func():
    print("this is c_func")

####################
# pkg3/__init__.py
####################
__all__ = ['x', 'y']

####################
# pkg3/x.py
####################
def x_func():
    print("this is x_func")

####################
# pkg3/y.py
####################
def y_func():
    print("this is y_func")

####################
# pkg4/__init__.py
####################
from .k import k_func, Animal
from .m import m_func

####################
# pkg4/k.py
####################
def k_func():
    print("this is k_func")

class Animal(object):
    def run(self):
        print("run")

####################
# pkg4/m.py
####################
def m_func():
    print("this is m_func")

####################
# pkgtest/test.py
####################
import import_demo.pkg1 as pkg  # __init__.py 本身就是一个模块，而它的模块名就是所在的包名
import import_demo.pkg1.a
import import_demo.pkg1.b as b
from import_demo.pkg1.c import c_func  # from 模块名 import 具体的功能
from import_demo.pkg3 import *  # 在 import_demo.pkg3 模块设置 __all__ = ['x', 'y']
from import_demo.pkg4 import *  # 在 import_demo.pkg4 模块引入下面模块的具体功能

print(pkg.init_int)
pkg.init_func()

print(import_demo.pkg1.a.int_a)
import_demo.pkg1.a.a_func()

print(b.int_b)
b.b_func()

c_func()  # 不用加前缀，直接使用

x.x_func()
y.y_func()

k_func()
m_func()
a = Animal()
a.run()
```

运行test.py结果：

```
__name__ =  import_demo.pkg1.a
0
this is init_func
1
this is a_func
2
this is b_func
this is c_func
this is x_func
this is y_func
this is k_func
this is m_func
run
```



## 安装第三方模块

在 Python 中，安装第三方模块，是通过包管理工具 pip 完成的。

如果你正在使用 Mac 或 Linux，安装 pip 本身这个步骤就可以跳过了。

注意：Mac 或 Linux 上有可能并存 Python 3.x 和 Python 2.x，因此对应的 pip 命令是 `pip3`。

一般来说，第三方库都会在 Python 官方的 [pypi.python.org](https://pypi.python.org/) 网站注册，要安装一个第三方库，必须先知道该库的名称，可以在官网或者 pypi 上搜索，比如 Pillow 的名称叫 [Pillow](https://pypi.python.org/pypi/Pillow/)，因此，安装 Pillow 的命令就是：

```shell
pip install Pillow
```

### 模块搜索路径

当我们试图加载一个模块时，Python 会在指定的路径下搜索对应的.py 文件，如果找不到，就会报错：

```shell
>>> import mymodule
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: No module named mymodule
```

默认情况下，Python 解释器会搜索当前目录、所有已安装的内置模块和第三方模块，搜索路径存放在 `sys` 模块的 `path` 变量中：

```
>>> import sys
>>> sys.path
['', '/Library/Frameworks/Python.framework/Versions/3.6/lib/python36.zip', '/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6', ..., '/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages']
```

如果我们要添加自己的搜索目录，有两种方法：

一是直接修改 `sys.path`，添加要搜索的目录：

```
>>> import sys
>>> sys.path.append('/Users/michael/my_py_scripts')
```

**这种方法是在运行时修改，运行结束后失效。**

第二种方法是设置环境变量 `PYTHONPATH`，该环境变量的内容会被自动添加到模块搜索路径中。设置方式与设置 Path 环境变量类似。注意只需要添加你自己的搜索路径，Python 自己本身的搜索路径不受影响。