---
title: 理解 Python 中的 import
date: 2018-09-28 18:06:42
categories:
- Python
tags:
- Python
---

当项目代码逐渐多起来的时候，我们会使用包、模块、类的方式去组织程序。先简单的理解下这几个概念：

* 类：使用 class 关键字定义的结构
* 模块：一个 Python 文件
* 包：包含 Python 文件的目录，在 Python 2 的时候规定这个目录必须要有一个文件 `__init__.py`，Python 3 中没有这个限制

这样组织项目的话，也就涉及到如何导入其他包、模块、类、方法、变量，这就是本文的内容。

# 怎么导入

如果我们要导入一个名称 xyz，这样就可以了

```
import xyz
```

其中，xyz 可以是包、模块、类等对象的名称。

知道了怎么使用，我们还得清楚解释器查找名称的过程，不然以后碰到导入失败的时候完全摸不着头脑。以下是查找的步骤，依次检索下去，如果找到，便导入；如果最后一步也没有找到，抛 ModuleNotFoundError 异常。

（1）在 sys.modules 中查找。这个字典中缓存了目前为止导入的模块名称

（2）在 Python 标准库中查找。标准库中包含 sys、datetime 等，具体还有哪些可以参考下面的链接

```
https://docs.python.org/3/library/
```

（3）在 sys.path 中查找。一般情况下，这个列表的第一个元素便是指的是当前目录

当然，如果你想从指定包中导入指定模块，或者从指定模块中导入某个类，等等，可以使用 from 关键字。比如

```
from a_package import a_module
from a_package.a_module import a_class
```

如果导入的名称和当前作用域中的名称发生冲突，可以给导入的名称取个别名。就像这样

```
from a_package import a_module as m
```

之后，你就可以将 m 当作 a_module 使用了。

<!-- more -->


# 绝对导入

导入分绝对导入和相对导入两种方式，这节介绍前一种。

假设有如下项目结构

```
├── main.py
├── module1.py
├── package_a
│   ├── module1.py
│   └── package_aa
│       ├── module1.py
│       └── module2.py
└── package_b
    ├── __init__.py
    └── module1.py
```

其中，module1.py 和 module2.py 中分别定义有 func1 和 func2 方法。

在 main.py 中，导入根目录下的 module1 中的 func1

```
from module1 import func1
```

导入 `package_b` 中的 module1

```
from package_b import module1
```

导入 `package_b/module1` 中的 func1

```
from package_b.module1 import func1
```

导入 `package_a/package_aa/module1` 中的 func1

```
from package_a.package_aa.module1 import func1
```

正如上面展示那样，根据绝对位置导入名称的方式就称为绝对导入。

# 相对导入

在 `package_a/package_aa/module1.py` 中，导入同级的 module2

```
from . import module2
```

导入同级 module2 中的 func2

```
from .module2 import func2
```

导入上一级中 module1.py 中的 func1，即 `package_a/module1.py` 中的 func1

```
from ..module1 import func1
```

导入项目根目录下的 module1 中的 func1

```
from ...module1 import func1
```

同理，如果需要访问更上级的目录，可以增加 `.` 号实现。

这种使用相对位置导入包的方式就叫相对导入。

事实上，上面的方式为显式相对导入。隐式的由于在 Python 3 中已废弃，所以就不介绍了。

# 优缺点

绝对导入能清楚的展示导入名称的位置，只是如果层级太深，导入代码会比较长。而相对导入相关代码相对简洁一些，但层级上的展示没有绝对导入那么清晰。个人倾向于使用绝对导入，这也是 PEP 8 推荐的。

# 让导入语句好看一些

PEP 8 中对导入语句也有一些规范，下面简单总结下

* 导入语句总应该在文件头部，除非有模块注释
* 对导入语句分组，并用空行隔开。一般可以分为以下三组：标准库、第三方库、当前程序的库

举个例子

```
"""导入语句规范 Demo
"""

# 标准库
import sys
import datetime

# 第三方库
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

# 当前程序的库
from package_a import module1
from package_a import package_aa
```
