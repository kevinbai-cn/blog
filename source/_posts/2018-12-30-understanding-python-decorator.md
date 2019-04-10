---
title: 一文搞懂 Python 装饰器
date: 2018-12-30 20:48:48
categories:
- Python
tags:
- Python
- 装饰器
---

Python 的装饰器允许你在不修改原方法的情况下，扩展可调用对象的行为。这里的可调用对象理解为函数或是实现了 `__call__` 方法的类的对象。为方便后文的描述，我直接使用函数一词。

有了装饰器，你可以为现有的函数附加各种有用的行为，包括但不限于记录日志、权限认证、缓存。

<!-- more -->

**在理解装饰器之前，你需要明白 Python 的以下几个特点：**

- 函数都是对象，它们可以被赋值给某个变量，也可以作为某个函数的参数或返回结果。
- 函数可以在某个函数内部定义，子函数可以存储父函数某次调用的状态（这也是通常闭包的定义）。

关于以上的点我举个例子说明

```
>>> def father(a):
...     def child():
...         return a
...     return child
...
>>> func = father(5)
>>> func()
5
```

father 函数返回了 child 函数作为结果，这印证了上述的第一点。另外，child 函数将 father 函数的某次调用状态（这里是 5）记录了下来，这印证了上述的第二点。

好了，了解了上面的点，我们进入正题吧。

# 1 一个最简单的装饰器

装饰器在代码实现上长什么样子呢？其实很直观

```
>>> def simple_decorator(func):
...     return func
```

这就是一个装饰器了。

简单的理解，一个装饰器就是一个函数，它的参数和返回结果也是函数。下面我们来看看这个装饰器怎么用

```
>>> def say_hello():
...     return 'Hello!'
...
>>> say_hello = simple_decorator(say_hello)
>>> say_hello()
'Hello!'
```

其实就是将需要装饰的函数作为参数传给装饰器，装饰器会将原函数做一些处理（当然，你喜欢的话也可以不作任何处理，就像本例）后，然后返回，这时我们调用返回的函数就可以了。看到这里有朋友会问，我们平时看到装饰器不是这么用的啊。其实，也只是语法糖而已。

```
>>> @simple_decorator
... def say_hello():
...     return 'Hello!'
...
>>> say_hello()
'Hello!'
```

这下熟悉了吧。当然，这个例子中的装饰器没有添加任何附加的行为，所以看起来好像没什么意义，但是至少让我们明白了装饰器是怎么工作的。

另外，需要注意的是，使用这个 @ 这个语法后，Python 会立即将被装饰的函数传给装饰器，装饰器的返回结果也直接赋值给被装饰函数同名的变量，这个例子中是 `say_hello` 。如果我们某些时候想使用被装饰前的函数的话，我们需要使用之前的那种方式。

# 2 让装饰器能修改一些行为吧

```
>>> def to_upper(func):
...     def inner():
...         primitive_result = func()
...         final_result = primitive_result.upper()
...         return final_result
...     return inner
```

不同于之前的 `simple_decorator` 装饰器，`to_upper` 装饰器定义了一个子函数用于将被装饰函数的返回结果变为大写，然后将该子函数返回。下面我们来看看将 `to_upper` 装饰器用在 `say_hello` 函数上会发生什么。

```
>>> @to_upper
... def say_hello():
...     return 'Hello!'
...
>>> say_hello()
'HELLO!'
```

哈哈，与预期相符，我们成功修改了被装饰函数的行为。

# 3 多个装饰器作用在一个函数上面，结果如何？

如果一个函数使用多个装饰器，会怎么样呢，我们来做个实验。我们先定义两个装饰器

```
>>> def bold(func):
...     def inner():
...         return '<b>' + func() + '</b>'
...     return inner
...
>>> def italic(func):
...     def inner():
...         return '<i>' + func() + '</i>'
...     return inner
```

同样，我们定义一个 `say_hello` 函数，并使用装饰器

```
>>> @bold
... @italic
... def say_hello():
...     return 'Hello!'
```

如果是先使用 bold 装饰器的话，结果会是 `'<i><b>Hello!</b></i>'`；如果是先使用 italic 装饰器的话，结果会是 `'<b><i>Hello!</i></b>'`。

下面我们看看结果

```
>>> say_hello()
'<b><i>Hello!</i></b>'
```

好吧，是先执行的 italic 装饰器，可理解为这样

```
decorated_say_hello = bold(italic(say_hello))
```

也就是说，一个函数有多个装饰器的时候使用顺序是从下到上的。

# 4 怎么让装饰器更方便调试

看了上面的内容，其实装饰器的工作原理就是使用处理过的函数代替之前的函数。这样的话有个弊端，那就是之前函数的一些信息丢失了。我们通过下面的例子简单的说下这个情况

```
>>> def say_hello():
...     """Return string Hello!"""
...     return 'Hello!'
...
>>> decorated_say_hello = to_upper(say_hello)
```

这时我们查看 `decorated_say_hello` 的一些信息，它和原函数 `say_hello` 不一致

```
>>> say_hello.__name__
'say_hello'
>>> say_hello.__doc__
'Return string Hello!'
>>>
>>> decorated_say_hello.__name__
'inner'
>>> decorated_say_hello.__doc__
>>>
```

这样我们在某些情况下调试的话可能就不太方便。还好，标准库中的 functools.wraps 的装饰器可以帮我们解决这个麻烦。

```
>>> import functools
>>>
>>> def to_upper(func):
...     @functools.wraps(func)
...     def inner():
...         return func().upper()
...     return inner
```

使用 functools.wraps 装饰器后，我们仍能看到到和原函数一致的信息

```
>>> @to_upper
... def say_hello():
...     """Return string Hello!."""
...     return 'Hello!'
...
>>> say_hello.__name__
'say_hello'
>>> say_hello.__doc__
'Return string Hello!.'
```

建议在写自己的装饰器的时候，把这个技巧用上，免得在调试部分问题的时候不方便。

# 5 小结

- 装饰器可以在不修改原函数的基础上增加额外的行为。
- @ 语法可以很方便的给某个函数加上装饰器。
- 一个函数有多个装饰器的时候使用顺序是从下到上的。
- 为了方便调试，建议在写自定义的装饰器的时候使用 functools.wraps 这个技巧。
