---
title: Python 命名空间和作用域
date: 2018-09-04 10:31:25
categories:
- Python
tags:
- Python
- 命名空间
- 作用域
---

本文说下自己对 Python 命名空间和作用域的理解。

**注意：内容基于 Python 3.6**

# 命名空间

> A namespace is a mapping from names to objects.

命名空间，直译是名称到对象（比如数字、字符串等）的映射，（我的理解是）这些名称构成一个命名空间。一般有三种命名空间
* 内置名称（built-in names）， Python 语言内置的名称，比如函数名 abs、char 和异常名称 BaseException、Exception 等等。
* 全局名称（global names），模块中定义的名称。
* 局部名称（local names），函数中定义的名称。（类中定义的也是）

看个例子就清楚了

```
# A 是全局名称， object 是内置名称
class A(object):
    a = -1  # a 是局部名称（类中）

    # print_abs_a 是局部名称（类中）， self 是局部名称（函数中）
    def print_abs_a(self):
        # temp_str 是局部名称（函数中）， abs 是内置名称
        temp_str = '%s 的绝对值是 %s' % (self.a, abs(self.a))
        print(temp_str)  # print 是内置名称

t = A()  # t 是全局名称
```

命名空间的生命周期各不相同

* 内置命名空间在编译器启动开始建立，直到程序结束
* 全局命名空间，在模块文件读入后建立，直到程序结束。
* 局部命名空间，在函数被调用后建立，函数退出时结束。递归函数每次调用都会建立不同的命名空间。对于类来说类似，实例化后建立，实例销毁后结束。

<!-- more -->

# 作用域

> A scope is a textual region of a Python program where a namespace is directly accessible. “Directly accessible” here means that an unqualified reference to a name attempts to find the name in the namespace.

作用域，是 Python 代码中的一段文本区域，在这个区域里能「直接」访问一个命名空间中的名称。所谓「直接」，就是只要给出名称（如 `some_name`）就能找到命名空间中的对应的名称，而不需要使用类似 modulename.subname 或是 object.attribute 等这样的方式。

有四种作用域：

* local scope，最内层，包含 local names，（搜索名称时）最先被搜索
* nonlocal scope, 如果一个函数（或类） A 里面又包含了一个函数 B ，那么对于 B 中的名称来说 A 中的作用域就为 nonlocal
* global scope，包含 global names
* builtin scope, 包含 built-in names，最后被搜索

其中，需要强调的是 local scope 和 nonlocal scope 是一个相对的概念。如果一个模块中，函数 A 直接包含了函数 B ， B 又直接包含了函数 C 。如果以 C 中的名称作为参考，那么 C 中的作用域为 local scope，则 B 中的作用域就为 nonlocal scope。如果以 B 中的名称作为参考，那么 B 中的作用域是 local scope， 则 A 中的作用域为 nonlocal。如果以 A 中的名称作为参考，那么 A 中的作用域是 local scope，不过要注意，模块中的作用域始终为 global scope，这时并没有 nonlocal scope。

对于赋值操作，默认都是操作当前作用域中包含的名称。假设现在在 local scope，如果要对 nonlocal scope 包含的名称进行赋值，则要用 nonlocal 关键字。如果要对 global scope 中包含的名称赋值要用 global 关键字。需要注意的是，如果在某个作用域内没有对应的名称，则在对应的作用域中会新增。
下面的例子可以帮你理解赋值操作

```
def scope_test():
    def do_local():
        spam = "local spam"

    def do_nonlocal():
        nonlocal spam
        spam = "nonlocal spam"

    def do_global():
        global spam
        spam = "global spam"

    spam = "test spam"
    do_local()
    print("After local assignment:", spam)
    do_nonlocal()
    print("After nonlocal assignment:", spam)
    do_global()
    print("After global assignment:", spam)

scope_test()
print("In global scope:", spam)
```

输出为

```
After local assignment: test spam
After nonlocal assignment: nonlocal spam
After global assignment: nonlocal spam
In global scope: global spam
```

对应读值操作，都是由内到外进行搜索。即 local scope -> nonlocal scope -> global scope -> builtin scope，如果都找不到对应的名称，则报错。

```
>>> x = 1
>>> def t():
...     def tt():
...         print(x)
...     tt()
...
>>> t()
1
>>> del x
>>> t()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 4, in t
  File "<stdin>", line 3, in tt
NameError: name 'x' is not defined
```

如果要读取指定作用域的名称，则可以使用对应的 nonlocal 或 global 关键字，如果对应作用域找不到该名称，则直接报错。

```
>>> x = 1
>>> def t():
...     x = 2
...     def tt():
...         global x
...         print(x)
...     tt()
...
>>> t()
1
>>> del x
>>> t()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 6, in t
  File "<stdin>", line 5, in tt
NameError: name 'x' is not defined
```

当然，如果当前作用域已有同名的名称，就不能使用这 nonlocal 或 global 了，否则会报错。

```
>>> x = 1
>>> def t():
...     x = 2
...     global x
...
  File "<stdin>", line 3
SyntaxError: name 'x' is assigned to before global declaration
```

同时发现个有趣的地方

```
>>> def t():
...     def tt():
...         nonlocal x
...         print(x)
...     tt()
...
  File "<stdin>", line 3
SyntaxError: no binding for nonlocal 'x' found

>>> def t():
...     def tt():
...         global x
...         print(x)
...     tt()
...
>>> t()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 5, in t
  File "<stdin>", line 4, in tt
NameError: name 'x' is not defined
```

第一个报的是语法错误，而第二个是运行时报的错误。这说明了一个问题：局部名称的查找是编译时就确定的，而全局名称和内置名称的查找都是在运行时确定的。（这里只是指出来，了解下就行，暂时没必要深入）

# 小结

个人觉得，没必要太在意命名空间和作用域的定义，之所以有命名空间的说法，只是为了引入作用域的概念。

我们只需要清楚两个方面的内容：一是，哪个作用域包含哪些名称；二是，相反的，赋值和读值的时候它又是指向哪个作用域，并理解 nonlocal 和 global 的使用。

# 参考

```
https://docs.python.org/3/tutorial/classes.html#python-scopes-and-namespaces
```
