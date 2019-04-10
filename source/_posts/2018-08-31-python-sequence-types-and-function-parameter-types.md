---
title: Python 序列类型以及函数参数类型
date: 2018-08-31 18:24:03
categories:
- Python
tags:
- Python
- 序列类型
- 函数参数类型
---

本文说说自己对 Python 序列类型和函数参数类型的理解。

**注意：内容基于 Python 3.6**

### 序列类型（Sequence Type）

我们先来看个例子

```
>>> x, y, z = [1, 2, 3]
>>> x
1
>>> y
2
>>> z
3
```

上面的操作叫做「多重赋值」，其实，只要是「序列类型」的，都可以有这种操作。

序列类型包括这几种：列表（list）、元组（tuple）、range、str、binary sequence type

```
>>> [1, 2, 3]  # 列表
>>> (1, 3, 3)  # 元祖
>>> range(1, 4)  # range
>>> 'text string: 文本字符串'  # str
>>> b'abc'  # byte (binary sequence type 的一种，这里了解就行)
```

所以，我们看到下面的用法就不奇怪了

```
>>> x, y = '你好'
>>> x
'你'
>>> y
'好'
```

这个也好理解

```
>>> 'abc' > 'aac'  # 字典序比较
True
>>> ['a', 'b', 'c'] > ['a', 'a', 'c']  # 同上
True
```

当然，「序列类型」还有很多类似的操作

```
>>> 1 in [1, 2, 3]  # x in s
True
>>> 'a' in 'abc'
True
>>> [1] * 5  # s * n
[1, 1, 1, 1, 1]
>>> 'a' * 3
'aaa'
>>> len([1, 2, 3])  # len(s)
3
>>> len('abc')
3
>>> # 等等
...
```

<!-- more -->

### 函数参数类型

共有三种：位置参数（Positional Arguments）、可变参数（Arbitrary Arguments）、关键字参数（Keyword Arguments）。

先有个整体的认识。函数定义如下

```
def introduce(name, *hobbies, **extra_info):
    print('Name:')
    print(name)
    print('Hobbies:')
    for hobby in hobbies:
        print(hobby)
    print('Extra info:')
    for key in extra_info:
        print(key, ':', extra_info[key])
```

调用

```
introduce('xiaoming',
          'movie', 'game',
          age=22, address='cn')
```

输出如下

```
Name:
xiaoming
Hobbies:
movie
game
Extra info:
age : 22
address : cn
```

其中，name 为「位置参数」，`*hobbies` 为「可变参数」，`**extra_info` 为「关键字参数」。

「位置参数」是指函数调用时根据参数位置进行赋值

```
>>> # arg1 ， arg2 为「位置参数」
...
>>> def f(arg1, arg2):
...     print(arg1, arg2)
...
>>> # 实参 1 对应形参 arg1 ，所以 arg1 = 1 。arg2 类似
...
>>> f(1, 2)
1 2
>>> f(1)  # 少参数会报错
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: f() missing 1 required positional argument: 'arg2'
>>> f(1, 2, 3)  # 多参数也会报错
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: f() takes 2 positional arguments but 3 were given
>>>
>>> # 但这可以通过设置参数默认值解决
...
>>> def f(arg1, arg2=2, arg3=3):
...     print(arg1, arg2, arg3)
...
>>> f(1)
1 2 3
>>> f(1, 3)
1 3 3
>>> f(1, 3, 5)
1 3 5
```

「可变参数」，个人理解就是用来解决不确定参数的问题的

```
>>> def my_join(sep, *args):
...     return sep.join(args)
...
>>> my_join(', ', 'apple', 'pear')
'apple, pear'
```

args 是一个元组，函数调用时，除掉「位置参数」用掉的参数，剩下的都会按顺序放到这个元组。

「关键字参数」是指以 `keyword=value` 形式定义的参数

```
>>> def f(name, age=22, address='cn'):
...     print('name:', name)
...     print('age:', age)
...     print('address:', address)
...
>>> # 只传一个参数
...
>>> f('xiaoming')
name: xiaoming
age: 22
address: cn
>>>
>>> # 三个都传
...
>>> f('xm', age=23, address='us')
name: xm
age: 23
address: us
>>>
>>> # 顺序可以随意
...
>>> f('xm', address='us', age=23)
name: xm
age: 23
address: us
>>>
>>> # 多了会出问题
...
>>> f('xm', age=23, address='us',  height='175cm')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: f() got an unexpected keyword argument 'height'
>>>
>>> # 但可以这样解决
...
>>> def f(name, age=22, address='cn', **extra_info):
...     print('name:', name)
...     print('age:', age)
...     print('address:', address)
...     print('extra_info:')
...     for kw in extra_info:
...             print(kw, ':', extra_info[kw])
...
>>> f('xm', age=23, address='us',  height='175cm')  # 搞定
name: xm
age: 23
address: us
extra_info:
height : 175cm
```

extra_info 是一个字典，由与前面参数对应不上的「关键字参数」组成。

相信很多朋友看到这，都有点疑惑，这「位置参数」和「关键字参数」怎么区分呢？我的理解是不用区分。理由如下

```
>>> def f(name, age=22, address='cn'):
...     print('name:', name)
...     print('age:', age)
...     print('address:', address)
...
>>> # 不传参会报错， name 像是一个「位置参数」
...
>>> f()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: f() missing 1 required positional argument: 'name'
>>>
>>> # name 当作「关键字参数」也没问题
...
>>> f(age=23, address='us', name='xm')
name: xm
age: 23
address: us
```

「位置参数」和「关键字参数」是一个相对的概念，不用去死磕。有的把三个参数都当作「位置参数」，有的把三个参数都当作「关键字参数」，有的把第一个当作「位置参数」，后面两个当作「关键字参数」。理解上其实都没问题，我们只要明白在各种情况下如何使用就好。（不过我个人倾向于最后一种理解）

另外需要注意下相关的两个操作

```
>>> def f(arg1, arg2, arg3):
...     print(arg1, arg2, arg3)
...
>>> # *s 相当于 s 中的元素解包，然后按顺序放到参数列表（s 可以是「序列类型」中的一种）
...
>>> f(*[1, 3, 5])  # 等同 f(1, 2, 3)
1 3 5
>>>
>>> # **d 相当于把 d 中的键值对解包，然后放到参数列表（d 可以是「字典」）
...
>>> f(**{'arg1': 2, 'arg2': 4, 'arg3': 6}) # 等同 f(arg1=2, arg2=4, arg3=6)
2 4 6
```

最后来个总结，放出本小节的第一个函数定义

```
def introduce(name, *hobbies, **extra_info):
    pass
```

当「位置参数」、「可变参数」和「关键字参数」同时存在时，「可变参数」在「关键字参数」之前，「位置参数」在最前。「位置参数」和「关键字参数」没必要强行去区分，有自己的合理理解即可。还有就是理解函数调用时，`*` 可以用于解包「列表」（或其它「序列类型」），`**` 可以用于解包「字典」。
