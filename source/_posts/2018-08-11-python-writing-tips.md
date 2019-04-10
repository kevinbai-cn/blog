---
title: Python 代码编写时的一些小技巧
date: 2018-08-11 16:43:29
categories:
- Python
tags:
- Python
---

平时写 Python 代码的时候，如果注意一些小技巧的使用，可以使我们的代码更简洁好看。

# 1 条件表达式

很多情况下，我们需要用条件表达式对变量进行赋值。有些时候利用逻辑短路的原理来实现的话，可以让代码更简洁。

比如，端口为 0 时将其改为默认端口，可以这样

```
port = port if port else 6379
```

如果利用逻辑短路的原理，可以改写成如下

```
port = port or 6379
```

解释一下逻辑短路，或运算

```
a or b
```

当 a 为真时，逻辑运算直接返回 a。因为不需要继续判断，就已经能确定 a or b 已经是真了；当 a 为假返回 b。

与运算

```
a and b
```

当 a 为假时，逻辑运算直接返回 a。因为不需要继续判断，结果已经能确定为假了；当 a 为真返回 b。

简单的说逻辑短路就是，如果当前已经能确定逻辑运算的真假了，就不继续进行运算了，直接返回已计算的最后一个元素。

上面改写的例子中，当 port 非 0 时会直接返回 port，为 0 时会返回 6379，与条件表达式功能一致。

再举一个例子，求 a 的倒数（假设 a 为 0 时返回 0）。条件表达式可以这样写

```
b = 1 / a if a else 0
```

等同于这样写

```
b = a and (1 / a)
```

上面两个例子通过逻辑短路来实现相同的功能相对要简洁一些，不过并不是让你在任何情况都用这种方式。如果当判断条件太多的时候，反倒会降低可读性。根据情况合理选择就好。

<!-- more -->

# 2 连续比较

当判断某个值是否属于某个区间的时候，可以使用连续比较，这算是 Python 的一个特性吧。比如

```
if 0 < a < 10:
    print(a)
```

等同于传统的

```
if a > 0 and a < 10:
    print(a)
```

# 3 逻辑判断换行

当进行逻辑判断时，如果判断的变量名太长或是条件太多，使用反斜线换行很影响美观，这个时候我们可以使用括号。比如

```
if (variable_has_a_long_name_a > 0
        and variable_has_a_long_name_b > 0
        and variable_has_a_long_name_c > 0):
    print(variable_has_a_long_name_a
          + variable_has_a_long_name_b
          + variable_has_a_long_name_c)
```

相比

```
if variable_has_a_long_name_a > 0 \
        and variable_has_a_long_name_b > 0 \
        and variable_has_a_long_name_c > 0:
    print(variable_has_a_long_name_a
          + variable_has_a_long_name_b
          + variable_has_a_long_name_c)
```

看着要美观一些。

当然，不只是进行逻辑判断可能会碰到这种情况，在其他地方也可能碰到，那个时候也可以试着用用类似的方法。

# 4 多行字符串

除注释外，很多情况我们需要使用多行字符串。如果使用的是单引号或者双引号话，换行需要加反斜线，不但不好看，修改起来还不大方便。这个时候可以使用三单引号或者三双引号。

比如

```
sql = '''
    SELECT t.* FROM t
    WHERE t.status = 0 AND t.created_at >= 1525104000 AND t.created_at <= 1527782399
    ORDER BY id DESC
    '''
```

相比

```
sql = 'SELECT t.* FROM t \
    WHERE t.status = 0 AND t.created_at >= 1525104000 AND t.created_at <= 1527782399 \
    ORDER BY id DESC'
```

要简洁方便一些。

# 5 上下文管理器

善用上下文管理器，减少一些不必要代码的书写。比如读文件

```
file = open('file_name')
try:
    for line in file:
        print(line)
finally:
    file.close()
```

改用上下文管理器的方式来写

```
with open('file_name') as file:
    for line in file:
        print(line)
```

这样会自动给你管理文件的打开与关闭，看起来清爽很多。
