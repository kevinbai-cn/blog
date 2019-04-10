---
title: Python 中的匿名函数，你滥用了吗？
date: 2018-10-23 11:14:59
categories:
- Python
tags:
- Python
- 匿名函数
---

本文介绍一下 Python 中的匿名函数以及使用的误区。

# 概念

我们从一个例子引入。

这里有一个元素为非空字符串的列表，按字符串最后一个字母将列表进行排序。如果原列表是 ['abc', 'g', 'def']，则结果应该是 ['abc', 'def', 'g']。

很容易得到如下代码

```
source_list = ['abc', 'g', 'def']

def get_last_element(v):
    return v[len(v) - 1]

result = sorted(source_list,
				key=get_last_element)
```

我们发现，`get_last_element` 这个方法比较简单，并且只用了一次，但必须定义后得到一个名称才能使用。在上面的情境中，使用起来稍微麻烦了点，我们能不能直接定义了就用呢？当然可以。

```
source_list = ['abc', 'g', 'def']
result = sorted(source_list,
				key=lambda v:  v[len(v) - 1])
```

如上，我们使用了匿名函数来解决前面的需求。这一部分代码是核心

```
lambda v:  v[len(v) - 1]
```

使用很简单，有如下几个关键点

- 使用 lambda 关键字
- 自动 return，不需要你自己写
- 只有一行代码

知道了上面的内容，匿名函数的概念也大致清晰了。

<!-- more -->

# 使用误区

知道了匿名函数后，我们在开发的时候有时候不经意就把这个东西滥用了。

## 1 给匿名函数命名

PEP 8 中建议我们不要写类似下面的代码

```
f1 = lambda v:  v[len(v) - 1]
```

匿名函数可以直接当做变量一样传递，比如传给函数作为参数，并不要求它一定有个名字。

需要注意的是，其实上面的操作并没有真正起到给函数命名的作用。

如果需要给定义的函数命名，使用 def 关键字即可

```
def f2(v):
    return v[len(v) - 1]
```

通过 def 定义的函数才是真正有名称的，匿名函数的名称永远是 lambda

```
>>> f1 = lambda v:  v[len(v) - 1]
>>> f1
<function <lambda> at 0x10fb7b2f0>
>>> def f2(v):
...     return v[len(v) - 1]
...
>>> f2
<function f2 at 0x10fb7b378>
```

## 2 没有必要的匿名函数

某些时候，我们没有使用匿名函数的必要，但却无意中使用了。

一般有两种情况。一是使用无意义的调用，比如下面的代码

```
result = sorted(source_list,
				key=lambda v: len(v))
```

将列表按元素的长度进行排序。

其实，我们可以直接这样

```
result = sorted(source_list, key=len)
```

上面的一提出来大家马上就理解了，但是平时我们却或多或少的犯了类似的毛病。

另一方面，有很多函数，标准库中都已经实现了，我们不知道，所以做了多余的事情。

比如这里

```
from functools import reduce

data = [1, 2, 3, 4, 5]
result = reduce(lambda x, y: x * y,
				data, 1)
```

这里的匿名函数可以直接用 mul 函数替换

```
from functools import reduce
from operator import mul

data = [1, 2, 3, 4, 5]
result = reduce(mul, data, 1)
```

Python 的 operator 模块提供了很多常用的操作，熟悉了后，你会慢慢喜欢上它的。里面除了算术、比较等操作，关于对字典、对象的操作也值得一提。

itemgetter 函数，根据键获取字典的值

```
# 根据元素的 weight 对列表进行排序
result = sorted(elements,
				key=lambda e: e['weight'])
# 等同于
from operator import itemgetter
result = sorted(elements,
				key=itemgetter('weight'))
```

attrgetter 函数，根据属性获取对应值

```
# 根据元素的 weight 对列表进行排序
result = sorted(elements,
				key=lambda e: e.weight)
# 等同于
from operator import attrgetter
result = sorted(elements,
				key=attrgetter('weight'))
```

要了解 operator 的更多操作，请查阅官方文档

```
https://docs.python.org/3/library/operator.html
```

## 3 降低可读性的匿名函数

按元素的长度和字典序对列表进行排序

```
source_list = ['abc', 'g', 'def']
result = sorted(source_list,
				key=lambda v:  (len(v), v.upper()))
```

上面的代码能够实现功能，但是我觉得下面的可读性更强一些

```
def length_and_alphabetical(v):
    return len(v), v.upper()

source_list = ['abc', 'g', 'def']
result = sorted(source_list,
				key=length_and_alphabetical)
```

我们通过函数函数名就大概知道了函数的作用，如果是匿名函数的话，我们还得去看相应的逻辑。

## 4 可能根本不需要传递函数

对一个列表进行求和，我们可能会看到这样的代码

```
from functools import reduce

data = [1, 2, 3, 4, 5]
result = reduce(lambda x, y: x + y,
				data)
```

其实，直接使用 sum 函数就行了

```
data = [1, 2, 3, 4, 5]
result = sum(data)
```

对于一些特定的需求，很多时候 Python 可能已经有了现成的方案。我们要有这方面的意识，尽可能简单的去解决问题。

## 5 可以不使用 map/filter

Python 中的 map 和 filter 一般都结合匿名函数在使用，前者是在迭代过程中对元素做一些处理，后者是过滤掉一些元素。很多情况下，我们可以使用列表推导式或者生成器表达式代替它们。

用生成器表达式代替 map

```
data = [1, 2, 3, 4, 5]

result1 = map(lambda v: v**2, data)
# 功能等同于
result2 = (v**2 for v in data)
```

用生成器表达式代替 filter

```
data = [1, 2, 3, 4, 5]

result1 = filter(lambda v: v > 3, data)
# 功能等同于
result2 = (v for v in data if v > 3)
```

明显的可以看出，使用生成器表达式的代码可读性更强一些。

# 什么时候使用匿名函数

说了这么多匿名函数使用的误区，那么什么时候使用比较合理呢？我觉得满足下面的几个点，就可以考虑考虑了。

- 只用一次
- 函数逻辑简单
- 使用匿名函数前尽可能的确定 Python 没有自带类似功能的函数

# 参考

```
https://treyhunner.com/2018/09/stop-writing-lambda-expressions/
```
