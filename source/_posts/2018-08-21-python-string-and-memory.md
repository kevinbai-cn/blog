---
title: Python 存储字符串时是如何节省空间的？
date: 2018-08-21 16:56:33
categories:
- 翻译
tags:
- Python
- 字符串
---

> 本文为翻译文章，已得到 @rushter 的许可
> 原文链接：https://rushter.com/blog/python-strings-and-memory
> 转载请注明出处

从 Python 3 开始，str 类型代表着 Unicode 字符串。取决于编码的类型，一个 Unicode 字符可能会占 4 个字节，这个有些时候有点浪费内存。

出于内存占用以及性能方面的考虑，Python 内部采用下面 3 种方式来存储 Unicode 字符：

* 一个字符占一个字节（Latin-1 编码）
* 一个字符占二个字节（UCS-2 编码）
* 一个字符占四个字节（UCS-4 编码）

使用 Python 进行开发的时候，我们会觉得字符串的处理都很类似，很多时候根本不需要注意这些差别。可是，当碰到大量的字符处理的时候，这些细节就要特别注意了。

<!-- more --> 

我们可以做一些小实验来体会下上面三种方式的差别。方法 sys.getsizeof 用来获取一个对象所占用的字节，这里我们会用到。

```
>>> import sys
>>> string = 'hello'
>>> sys.getsizeof(string)
54
>>> # 1-byte encoding
... sys.getsizeof(string + '!') - sys.getsizeof(string)
1
>>> # 2-byte encoding
... string2  = '你'
>>> sys.getsizeof(string2 + '好') - sys.getsizeof(string2)
2
>>> sys.getsizeof(string2)
76
>>> # 4-byte encoding
... string3 = '🐍'
>>> sys.getsizeof(string3 + '💻') - sys.getsizeof(string3)
4
>>> sys.getsizeof(string3)
80
```

如上所示，当字符串的内容不同时，所采用的编码也会不同。需要注意的是，Python 中每个字符串都会另外占用 49-80 字节的空间，用于存储额外的一些信息，比如哈希、字符串长度、字符串字节数和字符串标识。这么一来，一个空字符串会占用 49 个字节，也就好理解了。

我们可以通过 cbytes 直接获取一个对象的编码类型：

```
import ctypes


class PyUnicodeObject(ctypes.Structure):
    # internal fields of the string object
    _fields_ = [("ob_refcnt", ctypes.c_long),
                ("ob_type", ctypes.c_void_p),
                ("length", ctypes.c_ssize_t),
                ("hash", ctypes.c_ssize_t),
                ("interned", ctypes.c_uint, 2),
                ("kind", ctypes.c_uint, 3),
                ("compact", ctypes.c_uint, 1),
                ("ascii", ctypes.c_uint, 1),
                ("ready", ctypes.c_uint, 1),
                # ...
                # ...
                ]


def get_string_kind(string):
    return PyUnicodeObject.from_address(id(string)).kind
```

然后测试

```
>>> get_string_kind('Hello')
1
>>> get_string_kind('你好')
2
>>> get_string_kind('🐍')
4
```

如果一个字符串中的所有字符都能用 ASCII 表示，那么 Python 会使用 Latin-1 编码。简单说下，Latin-1 用于表示前 256 个 Unicode 字符。它能支持很多拉丁语言，比如英语、瑞典语、意大利语等。不过，如果是汉语、日语、西伯尔语等非拉丁语言，Latin-1 编码就行不通了。因为这些语言的文字的码位值（编码值）超过了 1 个字节的范围（0-255）。

```
>>> ord('a')
97
>>> ord('你')
20320
>>> ord('!')
33
```

大部分语言文字使用 2 个字节（UCS-2）来编码就已经足够了。4 个字节（UCS-4）的编码在保存特殊符号、emoji 表情或者少见的语言文字的时候会用到。

设想有一个 10GB 的 ASCII 文本文件，我们准备将其读到内存里面去。如果你插入一个 emoji 表情到文件中，文件占用空间将会达到 4 倍。如果你处理 NLP 问题较多的话，这种差别你应该能经常体会到。

# Python 内部为什么不直接使用 UTF-8 编码

最常见的 Unicode 编码是 UTF-8，但是 Python 内部并没有使用它。

UTF-8 编码字符的时候，取决于字符的内容，占的空间在 1-4 个字节内发生变化。这是一种特别省空间的存储方式，但正因为这种变长的存储方式，导致字符串不能通过下标直接进行随机读取，只能遍历进行查找。比如，如果采用的是 UTF-8 编码的话，Python 获取 string[5] 只能一个一个字符的进行扫描，直至找到目标字符。如果是定长编码的话也就没有问题了，要用一个下标定位一个字符，只需要用下标乘以指定长度（1、2 或者 4）就能确定。

# 字符串驻留

Python 中的空字符串和 ASCII 字符都会使用到字符串驻留（string interning）技术。怎么理解？你就把这些字符（串）看作是单例的就行。也就是说，两个相同内容的字符串如果使用了驻留的技术，那么内存里面其实就只开辟了一个空间。

```
>>> a = 'hello'
>>> b = 'world'
>>> a[4],b[1]
('o', 'o')
>>> id(a[4]), id(b[1]), a[4] is b[1]
(4567926352, 4567926352, True)
>>> id('')
4545673904
>>> id('')
4545673904
```

正如你看到的那样，a 中的字符 o 和 b 中的字符 o 有着同样的内存地址。Python 中的字符串是不可修改的，所以提前为某些字符分配好位置便于后面使用也是可行的。

使用到字符串驻留的除了 ASCII 字符、空窜之外，字符长度不超过 20 的串也使用到了同样的技术，前提是这些串的内容在编译的时候就能确定。

这包括：

* 方法名、类型
* 变量名
* 参数名
* 常量（代码中定义的字符串）
* 字典的键
* 属性名

当你在交互式命令行中编写代码的时候，语句同样也会先被编译成字节码。所以说，交互式命令行中的短字符串也会被驻留。

```
>>> a = 'teststring'
>>> b = 'teststring'
>>> id(a), id(b), a is b
(4569487216, 4569487216, True)
>>> a = 'test'*5
>>> b = 'test'*5
>>> len(a), id(a), id(b), a is b
(20, 4569499232, 4569499232, True)
>>> a = 'test'*6
>>> b = 'test'*6
>>> len(a), id(a), id(b), a is b
(24, 4569479328, 4569479168, False)
```

因为必须是常量字符串会使用到驻留，所以下面的例子不能达到驻留的效果：

```
>>> open('test.txt','w').write('hello')
5
>>> open('test.txt','r').read()
'hello'
>>> a = open('test.txt','r').read()
>>> b = open('test.txt','r').read()
>>> id(a), id(b), a is b
(4384934576, 4384934688, False)
>>> len(a), id(a), id(b), a is b
(5, 4384934576, 4384934688, False)
```

字符串驻留技术，减少了大量的重复字符串的内存分配。Python 底层通过字典实现的这种技术，这些暂存的字符串作为字典的键。如果想要知道某个字符串是否已经驻留，使用字典的查找操作就能确定。

Python 的 unicode 对象的实现（`https://github.com/python/cpython/blob/master/Objects/unicodeobject.c`）大约有 16,000 行 C 代码，其中有很多小优化在本文中未提及。如果你想更多的了解 Python 中的 Unicode，推荐你去看一下字符串相关的 PEPs（`https://www.python.org/dev/peps/`），同时查看下 unicode 对象的源码。
