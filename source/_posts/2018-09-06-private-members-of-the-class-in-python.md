---
title: Python 类的私有成员
date: 2018-09-06 06:58:36
categories:
- Python
tags:
- Python
- 类的私有成员
---

大家都知道，一个类的私有成员只能在其所在的类里能够访问，而在外部是不行的。比如 C++ 或者 Java 的 private 关键字就是用来声明类的私有成员的，如果一个属性或者方法被 private 声明，那么这个成员就只能在类里面才能直接使用，这样做的目的是为了封装。和很多面向对象的语言一样，Python 也支持类的私有成员的使用，不过有些差别。

```
class T:
    __v = 'I am a private variable'
    v = 'I am a public variable'

    def get_private_v(self):
        return self.__v

    def get_public_v(self):
        return self.v
```

然后在终端下测试

```
>>> T.v
'I am a public variable'
>>> T.__v
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: type object 'T' has no attribute '__v'
>>> T().get_private_v()
'I am a private variable'
>>> T().get_public_v()
'I am a public variable'
```

上面的结果与预期相符，但是

```
>>> T._T__v
'I am a private variable'
```

这就有些神奇了。原来 Python 里面并没有真正的私有成员，只不过是在编译的时候直接将成员 `__variable` 替换成了 `_classname__variable`。解释器在碰到 `self.__v` 的时候直接将其解释为 `self._T__v`，这样一来，在类外面就不能通过 `T().__v` 的方式访问了。这在一定程度上体现的私有成员的意义，不过这就没法阻止外部通过 `T()._T__v` 访问私有成员了。

<!-- more -->

不过要注意, exec()、eval()、getattr()、setattr()、delattr() 在类里面无法通过 `self.__variable` 的方式访问私有变量

```
class A:
    __a = 1
    a = 2

    def get_private_a(self):
        return eval('self.__a')

    def get_public_a(self):
        return eval('self.a')

    def get_private_a2(self):
        return eval('self._A__a')

# 测试

>>> A().get_private_a()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/kevinbai/Downloads/test/test.py", line 16, in get_private_a
    return eval('self.__a')
  File "<string>", line 1, in <module>
AttributeError: 'A' object has no attribute '__a'
>>> A().get_public_a()
2
>>> A().get_private_a2()
1
```

我的理解是，在使用上面的几个方法访问成员变量时，解释器不会执行将 `self.__a` 解释为 `self._A__a` 的步骤，所以 `A().get_private_a()` 方法执行到 `eval('self.__a')` 时会报对象属性不存在的错误。但是，如果通过替换后的变量来访问，就不会有问题。

简单来说， Python 中私有成员其实只是在编译时通过名字的替换来实现的。当在类里访问相应的成员时，解释器会做转换，但是使用 eval() 等方法时，解释器不会执行这个步骤。
