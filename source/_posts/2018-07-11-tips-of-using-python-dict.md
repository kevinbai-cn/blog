---
title: Python 中使用字典的几个小技巧
date: 2018-07-11 12:54:56
categories:
- Python
tags:
- Python
- 字典
---

# 1 解包

所谓解包，就是将字典通过 ** 操作符转为 Key=Value 的形式，这种形式可以直接传给函数作为关键字参数。
说说适用的几种情况。

## 1.1 搜索拼接条件

当应用中使用类似 SQLAlchemy 的 ORM 形式读取数据的时候，不同搜索条件，传入给 ORM 的搜索参数也随之改变。
下面是图书表的部分数据（只展示了部分字段）

```
+----+---------------+-------------------------+-------+
| id | category_name | book_name               | price |
+----+---------------+-------------------------+-------+
|  1 | 人文社科      | 人类简史                | 42.90 |
|  2 | 人文社科      | 世界简史                | 25.50 |
|  3 | 经济管理      | 极致产品                | 37.00 |
|  4 | 经济管理      | 史蒂夫·乔布斯传         | 44.20 |
|  5 | 经济管理      | 影响力                  | 41.20 |
+----+---------------+-------------------------+-------+
```

搜索时，我们会以这样的形式执行查询方法

```
books = Book.query.filter_by(id=1, book_name='影响力').all()
```

但是由于传入参数会根据搜索条件的变化而变化，无法直接写出有哪些参数，这个时候就可以使用字典解包

```
condition = {}
if book_id:
	condition['id'] = id
if book_name:
	condition['name'] = book_name
books = Book.query.filter_by(**condition).all()
```

这样就 OK 了

<!-- more -->

## 1.2 方法参数太多，为代码美观使用

```
new_book = Book(category_name='文学小说', book_name='解忧杂货店', price=28.8,
				...)
db.session.add(new_book)
```

改成这样的话，美观一些

```
book_param = {'category_name': '文学小说', 'book_name': '解忧杂货店', 'price': 28.8,
			  ...}
new_book = Book(**book_param)
db.session.add(new_book)
```

并且，在上述新增图书过程中，都会对提交的参数进行校验，而校验方法返回的结果（也就是 `book_param` 和其它信息）一般也都是字典，所以使用字典解包的方式更符合实际场景。

总之，适当使用字典解包对方法进行传参，可以让我们的代码更灵活。

# 2 setdefault() 的使用

先看下这个方法怎么使用

> dict.setdefault(key, default=None)

> 如果字典中包含有给定键，则返回该键对应的值，否则返回为该键设置的值。

很多时候我们需要对列表根据元素的某个 key 转化成一个包含列表的字典。比如，上面的数据中，我希望得到一个字典，字典的 key 是图书分类，value 是属于该分类的图书列表。我们通常会这样写

```
books_dict = {}
for book in book_list:
    if book['category_name'] not in books_dict.keys():
        books_dict[book['category_name']] = []
    books_dict[book['category_name']].append(book)
```

当然，这样写是正确的，能得到预期结果

```
{
	"人文社科": [{
		"id": 1,
		"category_name": "人文社科",
		"book_name": "人类简史",
		"price": 42.9
	}, {
		"id": 2,
		"category_name": "人文社科",
		"book_name": "世界简史",
		"price": 25.5
	}],
	"经济管理": [{
		"id": 3,
		"category_name": "经济管理",
		"book_name": "极致产品",
		"price": 37.0
	}, {
		"id": 4,
		"category_name": "经济管理",
		"book_name": "史蒂夫·乔布斯传",
		"price": 44.2
	}, {
		"id": 5,
		"category_name": "经济管理",
		"book_name": "影响力",
		"price": 41.2
	}]
}
```

但是如果使用字典的 setdefault() 方法话，可以少写几行代码，看起来也优雅一些

```
books_dict = {}
for book in book_list:
    books_dict.setdefault(book['category_name'], []).append(book)
```

# 3 字典合并

常用的合并方式

```
# new_dict = {**dict1, **dict2, ...}
# 合并多个字典，如果字典中存在相同的 key 的话，后面的会覆盖掉前面的
# 比如 dict2 会覆盖 dict1 中的 key 相同的值

>>> a = {'name': 'x', 'age': 13}
>>> b = {'name': 'y'}
>>> c = {**a, **b}
>>> c
{'name': 'y', 'age': 13}

# dict1.update(dict2)
# 合并两个字典，如果字典中存在相同的 key 的话，dict2 会覆盖 dict1 的对应值
# 理解为更新某个字典应该更合适

>>> a.update(b)
>>> a
{'name': 'y', 'age': 13}
```

有时我们碰到合并字典的情况也不少。比如，我们准备根据一本书的基本信息创建一本新书

```
# to_dict 将 ORM 对象转为字典，是自定义的，理解意思就好
base_book = Book.query.filter_by(id=1).first().to_dict()
# 提交的参数需要校验，校验成功后返回值包含 book_param ，内容和下面类似
book_param = {'book_name': '国家宝藏', 'price': 55.60}
# 同时需要更新新书的创建时间和更新时间
time_param = {'created_at': current_time, 'updated_at': current_time}
# 新增书籍
new_book = Book(**{**base_book, **book_param, **time_param})
db.session.add(new_book)
```

当然，如果只是合并两个字典的话，也可以使用 update() 方法。

假设我们只需要合并 `base_book` 和 `book_param`

```
base_book.update(book_param)
```

这也可以工作，不过要注意，这样会修改 `base_book` 中的值。

如果只是单纯的更新某个字典的信息的话，update() 方法显然最合适。对于当前需求的话，还是第一种方式更合适。
