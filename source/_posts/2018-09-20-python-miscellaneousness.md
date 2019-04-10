---
title: Python 开发杂记
date: 2018-09-20 06:32:33
categories:
- Python
tags:
- Python
- 数字
- 字典
- 列表
---

这篇文章记录下最近遇到的问题或踩到的坑以及对应的解决方案。

**注意：文中内容基于 Python 3.6 编写**

# 四舍五入

Python 中处理小数时一般会用到两种类型：一个是 float，另一个是 Decimal。在开发时，我一般直接使用 Decimal，因为 Decimal 不会有 float 的精度问题，并且，很多库比如 SQLAlchemy 等读取出来的数据也是该类型。更详细的说明可以看下我之前的一篇文章《说说 Python3 中的数字处理》。

最近测试同学在比对数据的时候，发现有个位置的校验一直通不过。我看了下相关的数据以及脚本，发现问题出现在数据的四舍五入的地方。

Python 中的 round 方法，并不是严格的四舍五入。看个例子

```
>>> a = 5.195
>>> round(a, 2)
5.2
>>> b = 5.125
>>> round(b, 2)
5.12
```

使用 Decimal 也有类似的问题

```
>>> from decimal import Decimal as D
>>> a = D('5.195')
>>> round(a, 2)
Decimal('5.20')
>>> b = D(5.125)
>>> round(b, 2)
Decimal('5.12')
```

网上看了下相关的问题，最后使用下面的方式解决

```
import decimal


def my_round(number, ndigits=2):
    decimal.getcontext().rounding = \
        decimal.ROUND_HALF_UP
    d = decimal.Decimal(str(number),
                        decimal.getcontext())
    return d.__round__(ndigits)
```

其实就是将 decimal 取约数的规则改为了四舍五入。测试一下

```
>>> a = 5.195
>>> my_round(a, 2)
Decimal('5.20')
>>> b = 5.125
>>> my_round(b, 2)
Decimal('5.13')
```

问题解决。

<!-- more -->

# 对元素是字典的集合求交集

平时，我们经常碰到给两个列表求交集的情况。如果元素是整形、字符串等不可变的对象的时候，直接将列表转为集合处理就 OK 了。就像这样

```
>>> list1 = [1, 2]
>>> list2 = [1]
>>> list(set(list1) & set(list2))
[1]
```

但是，当其中的元素可变的时候，就不行了。比如

```
>>> list1 = [{'name': 'a', 'age': 10},
...          {'name': 'b', 'age': 20}]
>>> list2 = [{'name': 'a', 'age': 10}]
>>> list(set(list1) & set(list2))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'dict'
```

因为可变的对象没法确定哈希嘛，所以就有了上面的错误。

想想，列表可以转化为不可变的 tuple，集合又可以转化为不可变的 frozenset。类似的思路，也可以实现个不可变类型 FrozenDict，从而能进行集合运算。

```
import collections


class FrozenDict(collections.Mapping):
    def __init__(self, *args, **kwargs):
        self._d = dict(*args, **kwargs)
        self._hash = None

    def __iter__(self):
        return iter(self._d)

    def __len__(self):
        return len(self._d)

    def __getitem__(self, key):
        return self._d[key]

    def __hash__(self):
        if self._hash is None:
            self._hash = 0
            for pair in self.items():
                self._hash ^= hash(pair)
        return self._hash
```

测试下

```
list1 = [{'name': 'a', 'age': 10},
         {'name': 'b', 'age': 20}]
list2 = [{'name': 'a', 'age': 10}]

list1_new = [FrozenDict(**v) for v in list1]
list2_new = [FrozenDict(**v) for v in list2]

temp = list(set(list1_new) & set(list2_new))

result = [dict(v) for v in temp]
print(result)
```

运行，得到结果

```
[{'name': 'a', 'age': 10}]
```

与预期相符。

当然，开发时需要使用的话，可以自己封装下，这里不再赘述。

# 列表迭代时同时删除元素

假设有下面一个列表

```
students = [{'name': 'a', 'age': 10},
            {'name': 'b', 'age': 20}]
```

当其中某个元素的 age 小于 30 时，我们需要删掉这个元素。我们很容易写出这样的代码

```
for s in students:
    if s['age'] < 30:
        students.remove(s)

print(students)
```

students 列表预期应该是为空的。我们执行下

```
[{'name': 'b', 'age': 20}]
```

呐尼，怎么第二个元素还在里面，不是应该都删掉了吗。

原来是这样的，迭代过程中，第一次遍历后已经删掉第一个元素，这时列表中只有一个元素了。Python 认为自己已经迭代过一次，和迭代使用到的列表元素个数一致，所以就不迭代了，也就有了上面的结果。

因此，迭代时，我们应该使用列表的一个副本。修改后的代码

```
students = [{'name': 'a', 'age': 10},
            {'name': 'b', 'age': 20}]

for s in students.copy():
    if s['age'] < 30:
        students.remove(s)

print(students)
```

执行得到结果

```
[]
```

OK，搞定。

这种细节问题，我们应多加以关注，避免碰到又浪费比较多的时间去排查处理。
