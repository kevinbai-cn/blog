---
title: 写 Python 时的 5 个坏习惯
date: 2018-08-25 07:35:32
categories:
- Python
tags:
- Python
---

很多文章都有介绍怎么写好 Python，我今天呢，相反，说说写代码时的几个坏习惯。有的习惯会让 Bug 变得隐蔽难以追踪，当然，也有的并没有错误，只是个人觉得不够优雅。

**注意：示例代码在 Python 3.6 环境下编写**

# 1 用列表作函数的默认参数

看下面这个例子

```
def func(a, b=[]):
    b.append(a)
    print(f'a: {a}')
    print(f'b: {b}')

func(1)
func(2)
```

正常我们期望的结果应该是这样的

```
a: 1
b: [1]
a: 2
b: [2]
```

但当我们执行代码后，只会得到这样的结果

```
a: 1
b: [1]
a: 2
b: [1, 2]
```

与预期不一致。为什么呢？因为 Python 列表是可变对象，而且函数传参又是传的引用，所以当第二次调用 func 方法前，b 中已经有了元素 1，调用后 b 最终有两个元素 1 和 2。

示例中 func 方法比较简单，当发现问题的时候简单看下就能找到根源。但是，如果是在一个比较复杂的方法里面，你有可能会粗心的忽略这一点，从而会碰到一些莫名其妙的问题。

所以，当我们要为函数设置默认参数的时候，不要使用可变对象。

上面的代码改成这样就 OK 了

```
def func(a, b=None):
    if b is None:
        b = []
    b.append(a)
    print(f'a: {a}')
    print(f'b: {b}')
```

执行后得到预期结果

```
a: 1
b: [1]
a: 2
b: [2]
```

<!-- more -->

# 2 文件操作

很多刚接触 Python 的伙伴做文件操作的时候很容易写类似的代码

```
file = open('file_name')
try:
    for line in file:
        print(line)
finally:
    file.close()
```

这没有问题，不过文件资源我们没有必要手动去维护，像关闭这样的操作交给上下文管理器做就好。

```
with open('file_name') as file:
    for line in file:
        print(line)
```

这样看起来不是清爽很多。

# 3 捕获所有异常

```
try:
    pass  # 做一些操作
except Exception as e:
    print(f'Exception {e}')
```

就像上面一样，有时我们为了能够快速的完成功能，很容易不管三七二十一，就捕获 Exception 异常。这可能会捕捉到键盘中断（KeyboardInterrupt）（CTRL + C）或断言错误（AsstionError）等异常。捕获不确定的异常，有时也会让我们的程序出现莫名其妙的问题，我们应该避免这样做。

准确的做法是根据上下文捕获 ValueError 、AttributeError 、TypeError 等比较具体的异常，然后做适当的错误处理，比如打印日志等。

# 4 忽略 Python 的 for...else 语法

开发中我们很容易碰到类似的需求，在一个列表中，确定某个特定的元素是否存在。比如，下面的代码便是确定列表中有没有奇数存在

```
numbers = [1, 2, 3, 4, 5]
is_odd_exist = False
for n in numbers:
    if n % 2 == 1:
        is_odd_exist = True
        break
        
if is_odd_exist:
    print('Odd exist')
else:
    print('Odd not exist')
```

这里，我们使用了一个标识 `is_odd_exist`，默认为 False。当找到奇数时，将其置为 True，然后跳出循环。这样写并没有问题，但是我们可以换种方式

```
numbers = [1, 2, 3, 4, 5]
for n in numbers:
    if n % 2 == 1:
        print('Odd exist')
        break
else:
    print('Odd not exist')
```

先介绍下 Python 的 for...else 语法，当 for 循环是正常结束时（即不是通过 break 跳出结束的），会执行 else 中的语句。

这里，我们使用了相对于其他语言如 C、PHP 等不同的一种方式，完成了相同的功能，看起来代码也简洁了不少。

# 5 使用键遍历字典

初学 Python 的伙伴，可能容易写出这样的代码

```
member = {'name': 'xiaoming',
          'age': 18,
          'mobile': '18312341234'}
for key in member:
    print(f'{key}: {member[key]}')
```

同样，这也是没有问题的，但看起来并不直观。字典遍历的时候，其实可以直接取出键值信息，像这样

```
member = {'name': 'xiaoming',
          'age': 18,
          'mobile': '18312341234'}
for key, val in member.items():
    print(f'{key}: {val}')
```

这样的话，看起来要明了一些。

上面提到的几点有些带有自己一定的偏见，不要求大家都接受，选择合理的使用就好。
