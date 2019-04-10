---
title: Python 中的 __str__ 和 __repr__ 有什么差别
date: 2018-11-27 09:22:25
categories:
- 翻译
tags:
- Python
- __str__
- __repr__
---

使用 Python 还是有一段时间了，很多大的概念都比较清楚，但是对于某些细节，其实并不熟悉。之前，看了相关的一些文章，觉得写的不错的就翻译了一些，这里分享下。当然，翻译并非逐字逐句的，我对某些段落进行了精简或补充，争取能把某些东西说的更清楚一些。

<!-- more -->

这篇讲 Python 中的 `__str__` 和 `__repr__` 的用法和区别，正文开始。

---

很多时候我们自己编写一个类，在将它的实例在终端上打印或查看的时候，我们往往会看到一个不太满意的结果。

```
class Car:
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage

>>> my_car = Car('red', 37281)
>>> print(my_car)
<__console__.Car object at 0x109b73da0>
>>> my_car
<__console__.Car object at 0x109b73da0>
```

类默认转化的字符串基本没有我们想要的一些东西，仅仅包含了类的名称以及实例的 ID（理解为 Python 对象的内存地址即可）。虽说这总比没有好，但确实是没什么用处啊。

所以，我们可能会手动打印对象的一些属性或者是在类里自己实现一个方法来返回我们需要的信息。

```
>>> print(my_car.color, my_car.mileage)
red 37281
```

这没有什么不对的地方，但是我们可以使用更 Pythonic 的方式来解决这个问题。

**1 使用 `__str__` 实现类到字符串的转化**

不用自己另外定义一个方法，和 JAVA 的 toString() 方法类似，你可以在类里实现 `__str__` 和 `__repr__` 方法从而自定义类的字符串描述，这两种都是比较 Pythonic 的方式去控制对象转化为字符串的方式。

下面我们通过做实验慢慢的来看这两种方式是怎么工作的。首先，我们先加一个 `__str__` 方法到前面的类中看看情况。

```
class Car:
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage

    def __str__(self):
        return f'a {self.color} car'
```

当你重新打印和查看这个类的实例的时候，你会看到一个稍微不同的结果

```
>>> my_car = Car('red', 37281)
>>> print(my_car)
'a red car'
>>> my_car
<__console__.Car object at 0x109ca24e0>
```

查看 `my_car` 的时候的输出仍然和之前一样，不过打印 `my_car` 的时候返回的内容和新加的 `__str__` 方法的返回一致。类的 `__str__` 方法会在某些需要将对象转为字符串的时候被调用。比如下面这些情况

```
>>> print(my_car)
a red car
>>> str(my_car)
'a red car'
>>> '{}'.format(my_car)
'a red car'
```

有了 `__str__` 这个方法，你就不用手动去打印对象的一些信息或者添加额外的方法去达到目的。类到字符串的转化使用 `__str__` 这种 Pythonic 的方式实现即可。

**2 使用 `__repr__` 也有类似的效果**

有的朋友可能发现，上面我们查看 `my_car` 对象的时候，输出的仍是类似 `<__console__.Car object at 0x109ca24e0>` 这样比较奇怪的结果。这是因为 Python 3 中一共有 2 种方式控制类到字符串的转化，第一种就是我们前面提到的 `__str__` 方法，另一个就是 `__repr__` 方法。后者的工作方式与前者类似，但是它被调用的时机不同。

Python 2 中还有一个 `__unicode__` 方法，后面我会说明，暂时跳过。

这里有个简单的例子，同样是在之前的类上作改动

```
class Car:
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage

    def __repr__(self):
        return '__repr__ for Car'

    def __str__(self):
        return '__str__ for Car
```

我们通过下面的操作来感觉下什么时候调用 `__str__` ，什么时候调用的 `__repr__` 。

```
>>> my_car = Car('red', 37281)
>>> print(my_car)
__str__ for Car
>>> '{}'.format(my_car)
'__str__ for Car'
>>> my_car
__repr__ for Car
```

从上面可以看出，当我们查看对象的时候（上面的最后一个操作）调用的是 `__repr__` 方法。

另外，列表以及字典等容器总是会使用 `__repr__` 方法。即使你显式的调用 str 方法，也是如此。

```
str([my_car])
'[__repr__ for Car]'
```

如果我们需要显示的指定以何种方式进行类到字符串的转化，我们可以使用内置的 str() 或 repr() 方法，它们会调用类中对应的双下划线方法。（当然，上面的情况除外）

```
>>> str(my_car)
'__str__ for Car'
>>> repr(my_car)
'__repr__ for Car'
```

当然，如果你直接调用 `__str__` 或 `__repr__` 方法，也能达到同样的方法，但是不推荐这么做。

**3 那么 `__str__` 和 `__repr__` 的差别是什么**

现在你可能在想，`__str__` 和 `__repr__` 的差别究竟在哪里，它们的功能都是实现类到字符串的转化，它们的特定并没有体现出用途上的差异。

带着这个这个问题，我们试着去 Python 的标准库中找找答案。我们就来看看 datetime.date 这个类是怎么在使用这两个方法的。

```
>>> import datetime
>>> today = datetime.date.today()
>>> str(today)
'2017-02-02'
```

因此，我们有个初步的答案。

`__str__` 的返回结果可读性强。也就是说，`__str__` 的意义是得到便于人们阅读的信息，就像上面的 '2017-02-02' 一样。

`__repr__` 的返回结果应更准确。怎么说，`__repr__` 存在的目的在于调试，便于开发者使用。细心的读者会发现将 `__repr__` 返回的方式直接复制到命令行上，是可以直接执行的。

上面应该就是这两个方法的意义所在吧（便于描述，后面我称这为通常的原则吧）。

但是于个人来说，如果按照通常的原则去编写代码会做很多额外的工作，两个方法的返回结果只需要对开发者友好就可以了，并不一定需要存储某个对象的完整状态。后面我会根据这一点，写部分有实践意义的代码实例，并不完全按照通常的原则。

**4 为什么每个类都最好有一个 `__repr__` 方法**

如果你没有添加 `__str__` 方法，Python 在需要该方法但找不到的时候，它会去调用 `__repr__` 方法。因此，我推荐在写自己的类的时候至少添加一个 `__repr__` 方法，这能保证类到字符串始终有一个有效的自定义转换方式。

我们为 Car 类添加一个 `__repr__` 方法

```
def __repr__(self):
    return f'Car({self.color!r}, {self.mileage!r})'
```

注意，我们这里用了 !r 标记，是为了保证 self.color 与 self.mileage 在转化为字符串的时候使用 repr(self.color) 和 repr(self.mileage) ，而不是 str(self.color) 和 str(self.mileage) 。

这个能正常工作，不过有个缺点，就是我们把类的名称写死了。这有一个小技巧可以改进这种方式，就是使用对象的 `__class__.__name__` 属性，该属性总代表着类的名称。

这样做的话，当类名被修改的时候，我们不需要修改 `__repr__` 方法，这也符合软件开发的 DRY 原则（Don’t Repeat Yourself）。

```
def __repr__(self):
   return (f'{self.__class__.__name__}('
           f'{self.color!r}, {self.mileage!r})')
```

这种写法也有一个不好的地方，就是格式化字符串太长了。当然，我们好好调整一个格式也能符合 PEP 8 的代码规范。

实现了 `__repr__` 方法后，当我们查看类的实例或者直接调用 repr() 方法，就能得到一个比较满意的结果了。

```
>>> repr(my_car)
'Car(red, 37281)'
```

打印或直接调用 str() 方法也能得到相同的结果，因为 `__str__` 的默认实现就是调用 `__repr__` 方法。

```
>>> print(my_car)
'Car(red, 37281)'
>>> str(my_car)
'Car(red, 37281)'
```

这样就能以比较少的工作量，让两个方法都能工作，并且也有一定的可读性。所以一般情况下，我都推荐至少添加一个 `__repr__` 方法。

下面是比较全的代码示例

```
class Car:
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage

    def __repr__(self):
       return (f'{self.__class__.__name__}('
               f'{self.color!r}, {self.mileage!r})')

    def __str__(self):
        return f'a {self.color} car'
```

**5 Python 2 中的 `__unicode__` 方法**

Python 3 中字符串用 str 类型表示，代表 unicode 字符串。而 Python 2 中字符串有两种类型，一是 str ，只能存储 ASCII 码，另一种是 unicode ，与 Python 3 中的 str 等同。

通常来说，用 `__unicode__` 来控制类到字符串的转化更容易被大家接受。和 `__str__` 和 `__repr__` 类似，你可以使用内置的 unicode() 来显示调用 `__unicode__` 方法。

打印语句和 str() 会调用 `__str__` 方法，unicode() 会先找 `__unicode__` 方法，找不到的话会调用 `__str__` 方法，并将其结果按当时的编码方式解码返回。

相对于Python 3 ，Python 2 中的类到字符串的转化，显得稍微复杂一些。不过，下面我给了个便于实践的思路。由于使用 unicode 处理字符串更方便，这也是趋势，所以我们总会实现自己的 `__unicode__` 方法。同时，`__str__` 方法的实现则依靠于 `__unicode__`，主要逻辑是调用 `__unicode__` 方法并将其结果使用 UTF-8 编码后返回。

```
def __str__(self):
    return unicode(self).encode('utf-8')
```

所以，大部分情况下，`__str__` 方法都不需要做修改，对于新建的类，可以直接把这个 `__str__` 方法复制进去，而把关注点只放在 `__unicode__` 方法的实现上。

下面是在 Python 2 中一段比较完整的示例

```
class Car(object):
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage

    def __repr__(self):
       return '{}({!r}, {!r})'.format(
           self.__class__.__name__,
           self.color, self.mileage)

    def __unicode__(self):
        return u'a {self.color} car'.format(
            self=self)

    def __str__(self):
        return unicode(self).encode('utf-8')
```

**6 小结**

（1）我们可以使用 `__str__` 和 `__repr__` 方法定义类到字符串的转化方式，而不需要手动打印某些属性或是添加额外的方法。

（2）一般来说，`__str__` 的返回结果在于强可读性，而 `__repr__` 的返回结果在于准确性。

（3）我们至少需要添加一个 `__repr__` 方法来保证类到字符串的自定义转化的有效性，`__str__` 是可选的。因为默认情况下，在需要却找不到 `__str__` 方法的时候，会自动调用 `__repr__` 方法。

（4）在 Python 2 中，我们可能更在意类的 `__unicode__` 方法的实现。

**7 原文**

`https://dbader.org/blog/python-repr-vs-str`
