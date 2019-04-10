---
title: Python 函数如何重载？
date: 2018-08-29 11:59:22
categories:
- Python
tags:
- Python
- 重载
---

什么是函数重载？简单的理解，支持多个同名函数的定义，只是参数的个数或者类型不同，在调用的时候，解释器会根据参数的个数或者类型，调用相应的函数。

重载这个特性在很多语言中都有实现，比如 C++、Java 等，而 Python 并不支持。这篇文章呢，通过一些小技巧，可以让 Python 支持类似的功能。

# 参数个数不同的情形

先看看这种情况下 C++ 是怎么实现重载的

```
#include <iostream>
using namespace std;

int func(int a)
{
	cout << 'One parameter' << endl;
}

int func(int a, int b)
{
	cout << 'Two parameters' << endl;
}

int func(int a, int b, int c)
{
	cout << 'Three parameters' << endl;
}
```

如果 Python 按类似的方式定义函数的话，不会报错，只是后面的函数定义会覆盖前面的，达不到重载的效果。

```
>>> def func(a):
...     print('One parameter')
... 
>>> def func(a, b):
...     print('Two parameters')
... 
>>> def func(a, b, c):
...     print('Three parameters')
... 
>>> func(1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: func() missing 2 required positional arguments: 'b' and 'c'
>>> func(1, 2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: func() missing 1 required positional argument: 'c'
>>> func(1, 2, 3)
Three parameters
```

但是我们知道，Python 函数的形参十分灵活，我们可以只定义一个函数来实现相同的功能，就像这样

```
>>> def func(*args):
...     if len(args) == 1:
...         print('One parameter')
...     elif len(args) == 2:
...         print('Two parameters')
...     elif len(args) == 3:
...         print('Three parameters')
...     else:
...         print('Error')
... 
>>> func(1)
One parameter
>>> func(1, 2)
Two parameters
>>> func(1, 2, 3)
Three parameters
>>> func(1, 2, 3, 4)
Error
```

<!-- more -->

# 参数类型不同的情形

同样，先看下当前情况下 C++ 的重载是怎么实现的

```
#include <iostream>
using namespace std;

int func(int a)
{
	cout << 'Int: ' << a << endl;
}

int func(float a)
{
	cout << 'Float: ' << a << endl;
}
```

代码中，func 支持两种类型的参数：整形和浮点型。调用时，解释器会根据参数类型去寻找合适的函数。Python 要实现类似的功能，需要借助 functools.singledispatch 装饰器。

```
from functools import singledispatch

@singledispatch
def func(a):
	print(f'Other: {a}')

@func.register(int)
def _(a):
	print(f'Int: {a}')

@func.register(float)
def _(a):
	print(f'Float: {a}')

if __name__ == '__main__':
	func('zzz')
	func(1)
	func(1.2)
```

func 函数被 functools.singledispatch 装饰后，又根据不同的参数类型绑定了另外两个函数。当参数类型为整形或者浮点型时，调用绑定的对应的某个函数，否则，调用自身。

执行结果

```
Other: zzz
Int: 1
Float: 1.2
```

需要注意的是，这种方式只能够根据第一个参数的类型去确定最后调用的函数。

关于 singledispatch 的更多细节请看官方文档

```
https://docs.python.org/3.6/library/functools.html#functools.singledispatch
```

**注意：函数返回值不同也是重载的一种情况，暂时没有比较好的 Python 实现方式，所以没有提及**

个人觉得，重载就是为了语言的灵活性而设计的，而 Python 函数本来就有不少巧妙的设计，这个时候去仿这个技术，其实没有多大必要，而且感觉有些违背 Python 的哲学。所以，本文更多的是在讲如何模仿，而对于重载的使用场景并没有作多少说明。
