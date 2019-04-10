---
title: 搞清楚 Python 的迭代器、可迭代对象、生成器
date: 2018-07-16 18:10:20
categories:
- Python
tags:
- Python
- 迭代器
- 可迭代对象
- 生成器
---

很多伙伴对 Python 的迭代器、可迭代对象、生成器这几个概念有点搞不清楚，我来说说我的理解，希望对需要的朋友有所帮助。

# 1 迭代器协议

迭代器协议是核心，搞懂了这个，上面的几个概念也就很好理解了。

所谓迭代器协议，就是要求一个迭代器必须要实现如下两个方法

> `iterator.__iter__()`

> Return the iterator object itself. 

> `iterator.__next__()`

> Return the next item from the container.

也就是说，一个对象只要支持上面两个方法，就是迭代器。`__iter__()` 需要返回迭代器本身，而 `__next__()` 需要返回下一个元素。

# 2 可迭代对象

知道了迭代器的概念，那可迭代对象又是啥呢？

这个更简单，只要对象实现了 `__iter__()` 方法，并且返回的是一个迭代器，那么这个对象就是可迭代对象。

比如我们常见的列表就是可迭代对象

```
>>> l = [1, 3, 5]
>>> iter(l)
<list_iterator object at 0x101a1d9e8>
```

使用 iter() 会调用对应的 `__iter__()` 方法，这里返回的是一个列表迭代器，所以说列表就是一个可迭代对象。

<!-- more -->

# 3 手写一个迭代器

迭代器的实现有不同的方式，相信大家首先能想到的就是自定义类，我们就从这个说起。

便于说明，我们手写一个迭代器，用于生成奇数序列。

按照迭代器协议，我们实现上述的两个方法。

```
class Odd:
    def __init__(self, start=1):
        self.cur = start

    def __iter__(self):
        return self

    def __next__(self):
        ret_val = self.cur
        self.cur += 2
        return ret_val
```

终端里，我们实例化一个 Odd 类得到一个对象 odd

```
>>> odd = Odd()
>>> odd
<__main__.Odd object at 0x101a1d9b0>
```

使用 iter() 方法会调用类里的 `__iter__` 方法，得到它本身

```
>>> iter(odd)
<__main__.Odd object at 0x101a1d9b0>
```

使用 next() 方法会调用对应的 `__next__()` 方法，得到下一个元素

```
>>> next(odd)
1
>>> next(odd)
3
>>> next(odd)
5
```

其实，odd 对象就是一个迭代器了。

我们可以用 for 来遍历它

```
odd = Odd()
for v in odd:
	print(v)
```

细心的伙伴可能会发现，这个其实会无限的打印下去，那怎么解决呢？

我们拿一个列表做做实验，先得到它的迭代器对象

```
>>> l = [1, 3, 5]
>>> li = iter(l)
>>> li
<list_iterator object at 0x101a1da90>
```

然后手动获取下一个元素，直到没有下一个元素为止，看下会发生什么

```
>>> next(li)
1
>>> next(li)
3
>>> next(li)
5
>>> next(li)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

原来列表迭代器会在没有下一个元素的时候抛出 StopIteration 异常，估计 for 语句就是根据这个异常来确定是否结束。

我们修改一下原来的代码，能生成指定范围内的奇数

```
class Odd:
    def __init__(self, start=1, end=10):
        self.cur = start
        self.end = end

    def __iter__(self):
        return self

    def __next__(self):
        if self.cur > self.end:
            raise StopIteration
        ret_val = self.cur
        self.cur += 2
        return ret_val
```

我们使用 for 试一下

```
>>> odd = Odd(1, 10)
>>> for v in odd:
...     print(v)
...
1
3
5
7
9
```

果然，和预期一致。

我们用 while 循环模拟 for 的执行过程

目标代码

```
for v in iterable:
	print(v)
```

翻译后的代码

```
iterator = iter(iterable)
while True:
	try:
		v = next(iterator)
		print(v)
	except StopIteration:
		break
```

事实上 Python 的 for 语句原理也就是这样，可以将 for 理解为一个语法糖。

# 4 创建迭代器的其它方式

生成器其实也是迭代器，所以可以使用生成器的创建方式创建迭代器。

## 4.1 生成器函数

和普通函数的 return 返回不同，生成器函数使用 yield。

```
>>> def odd_func(start=1, end=10):
...     for val in range(start, end + 1):
...         if val % 2 == 1:
...             yield val
...
>>> of = odd_func(1, 5)
>>> of
<generator object odd_func at 0x101a14200>
>>> iter(of)
<generator object odd_func at 0x101a14200>
>>> next(of)
1
>>> next(of)
3
>>> next(of)
5
>>> next(of)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

## 4.2 生成器表达式

```
>>> g = (v for v in range(1, 5 + 1) if v % 2 == 1)
>>> g
<generator object <genexpr> at 0x101a142b0>
>>> iter(g)
<generator object <genexpr> at 0x101a142b0>
>>> next(g)
1
>>> next(g)
3
>>> next(g)
5
>>> next(g)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

## 4.3 怎么选择

到现在为止，我们知道了创建迭代器的 3 种方式，那么该如何选择？

不用说也知道，最简单的就是生成器表达式，如果表达式能满足需求，那么就是它；如果需要添加比较复杂的逻辑就选生成器函数；如果前两者没法满足需求，那就自定义类实现吧。总之，选择最简单的方式就行。


# 5 迭代器的特点

## 5.1 惰性

迭代器并不是把所有的元素提前计算出来，而是在需要的时候才计算返回。

## 5.2 支持无限个元素

比如上面我们建立的第一个 Odd 类，它的实例 odd 表示大于 start 的所有奇数，而列表等容器没法容纳无限个元素的。

## 5.3 省空间

比如存 10000 个元素

```
>>> from sys import getsizeof
>>> a = [1] * 10000
>>> getsizeof(a)
80064
```

列表占用 80K 左右。

而迭代器呢？

```
>>> from itertools import repeat
>>> b = repeat(1, times=10000)
>>> getsizeof(b)
56
```

只占用了 56 个字节。

也正因为迭代器惰性的特点，才有了这个优势。

# 6 一些需要注意的细节

# 6.1 迭代器同时也是可迭代对象

因为迭代器的 `__iter__()` 方法返回了它自身，而正好它本身就是个迭代器，所以说迭代器也是可迭代对象。

# 6.2 迭代器遍历完一次就不能从头开始了

看一个奇怪的例子

```
>>> l = [1, 3, 5]
>>> li = iter(l)
>>> li
<list_iterator object at 0x101a1da90>
>>> 3 in li
True
>>> 3 in li
False
```

因为 li 是列表迭代器，第一次查找 3 的时候，找到了，所以返回 True，但是由于第一次迭代，已经跳过了 3 那个元素，第二次就找不到了，所以会出现 False。

因此，记得迭代器是「一次性」的。

当然，列表是可迭代对象，不管查找几次都是正常的。（不好理解的话，想想上面 for 语句的执行原理，每次都会从可迭代对象那通过 iter() 方法取到新的迭代器）

```
>>> 3 in l
True
>>> 3 in l
True
```

# 7 小节

* 实现了迭代器协议的对象都是迭代器
* 实现了 `__iter__()` 方法并返回迭代器的对象是可迭代对象
* 生成器也是一种迭代器
* 创建迭代器有三种方式，生成器表达式、生成器函数、自定义类，看情况选择最简单的就好
* 迭代器同时也是可迭代对象
* 迭代器是「一次性」的

前面 3 小项是重点，这 3 点理解了，其它的也都能领会。搞清楚标题的那几个名词的概念的自然也没有问题。

# 8 参考

* https://docs.python.org/3/library/stdtypes.html#iterator-types
* https://opensource.com/article/18/3/loop-better-deeper-look-iteration-python
* http://treyhunner.com/2018/06/how-to-make-an-iterator-in-python
