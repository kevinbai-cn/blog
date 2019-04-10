---
title: 说说 Python 3 中的数字处理
date: 2018-07-12 14:41:19
categories:
- Python
tags:
- Python
- 数字
---

最近在处理订单相关的问题，踩了数字的一些坑，在此记录下。

其中有问题的代码涉及金额比较，便于描述，假设了下面一段代码

```
def is_paid(pay_price, paid_price):
	return pay_price == paid_price

# 数据表中的记录类似这样
# id pay_price ...
# 1  12.3
# ...

# 操作如下
# 这里使用了 SQLAlchemy 的 ORM 形式读取数据
order = Order.query.filter_by(id=1).first()
if is_paid(order.pay_price, 12.3):
	print('paid')
else:
	print('unpaid')

# 最后打印的却是 unpaid
```

跟踪代码才发现 order.pay_price 是 Decimal 类型，而 12.3 是浮点类型，Python 是强类型语言，类型不一样当然不等。

```
>>> from decimal import Decimal as d
>>> a = d('12.3')
>>> b = 12.3
>>> type(a)
<class 'decimal.Decimal'>
>>> type(b)
<class 'float'>
>>> a
Decimal('12.3')
>>> b
12.3
>>> a == b
False
```

仔细想想，有点不对，你看 1 == 1.0 就成立啊，不也是不同类型（整型和浮点型）吗。

<!-- more -->

不管是不是强类型语言，数字之间作比较还是应该要能行的吧。

这里没有深挖，感觉就是 Python 设计的缘故吧。

所以，这里应该怎么作等值比较，试了下 math.isclose(a, b) ，嗯，行得通。

本应该在这里结束了，但，不小心玩了下

```
>>> m = 0.1 + 0.2
>>> n = 0.3
>>> m == n
False
```

纳尼

然后打印了下值

```
>>> m
0.30000000000000004
>>> n
0.3
```

我还有什么话可说呢，这下让我对 Python 浮点数的处理产生了怀疑。

也让自己对以前所写的数字比较产生了怀疑，天哪，全是 Bug 。不过后面想通了，不存在的 ：）

```
>>> x = 1.0
>>> y = Decimal('2')
>>> x + y
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'float' and 'decimal.Decimal'
```

这下，搞得我以后都不知道怎么处理数字了。程序中很多地方都没判断类型，呀，又是一堆隐患。(同样，还是想多了...)

为了理清自己的思路，又试了下

```
>>> z = 3
>>> y + 3
Decimal('5')
```

原来，Decimal 和整型是能进行算术运算的。

后面自己冷静了下，终于想通了。

程序中，我们用的很多库，涉及小数的基本上都是用的 Decimal 类型，比如 SQLAchemy ORM 取出来的小数数据都是此类型。

Decimal 之间比较一般精度的数字都是没问题的，并且程序中我们定义数字的初始值基本都是整型 0 ，和 Deciamal 运算没有问题，所以上面的疑虑都烟消云散了。

所以，在以后的数字处理中，自己尽量只使用整型和 Decimal 类型，来避免上面的隐形问题。

不过，要注意

```
>>> t1 = Decimal(0.123)
>>> t2 = Deciaml('0.123')
>>> t1 == t2
False
```

这是由于浮点数 0.123 在转为 Decimal 的时候失去了精度

```
>>> t1
Decimal('0.1229999999999999982236431605997495353221893310546875')
>>> t2
Decimal('0.123')
```

因此，定义 Decimal 类型的时候，我们尽量使用字符串来避免这个问题。
