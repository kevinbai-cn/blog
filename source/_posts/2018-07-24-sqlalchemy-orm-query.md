---
title: 灵活使用 SQLAlchemy 中的 ORM 查询
date: 2018-07-24 18:36:03
categories:
- Python
tags:
- Python
- SQLAlchemy
- ORM
---

之前做查询一直觉得直接拼 SQL 比较方便，用了 SQLAlchemy 的 ORM 查询之后，发现也还可以，还提高了可读性。

这篇文章主要说说 SQLAlchemy 常用的 ORM 查询方式，偏实践。看了之后，对付开发中的查询需求，我觉得可以满足不少。

为方便说明，假设有如下数据

**图书表 books**

```
+----+--------+--------------------------+-------+
| id | cat_id | name                     | price |
+----+--------+--------------------------+-------+
|  1 |      1 | 生死疲劳                 | 40.40 |
|  2 |      1 | 皮囊                     | 31.80 |
|  3 |      2 | 半小时漫画中国史         | 33.60 |
|  4 |      2 | 耶路撒冷三千年           | 55.60 |
|  5 |      2 | 国家宝藏                 | 52.80 |
|  6 |      3 | 时间简史                 | 31.10 |
|  7 |      3 | 宇宙简史                 | 22.10 |
|  8 |      3 | 自然史                   | 26.10 |
|  9 |      3 | 人类简史                 | 40.80 |
| 10 |      3 | 万物简史                 | 33.20 |
+----+--------+--------------------------+-------+
```

**分类表 categories**

```
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 文学         |
|  2 | 人文社科     |
|  3 | 科技         |
+----+--------------+
```

ORM 对象定义如下

***

**注意：本文 Python 代码在以下环境测试通过**

* Python 3.6.0
* PyMySQL 0.8.1
* SQLAlchemy 1.2.8

***

```
# coding=utf-8

from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Numeric
from sqlalchemy.orm import sessionmaker

Base = declarative_base()
engine = create_engine('mysql+pymysql://username:password'
                       '@127.0.0.1:3306/db_name?charset=utf8')
Session = sessionmaker(bind=engine)

session = Session()


def to_dict(self):
    return {c.name: getattr(self, c.name, None)
            for c in self.__table__.columns}
Base.to_dict = to_dict


class Book(Base):
    __tablename__ = 'books'

    id = Column(Integer, primary_key=True)
    cat_id = Column(Integer)
    name = Column('name', String(120))
    price = Column('price', Numeric)


class Category(Base):
    __tablename__ = 'categories'

    id = Column(Integer, primary_key=True)
    name = Column('name', String(30))
```

好了，下面进入正题。

# 1 根据主键获取记录

当我们获取图书的详情时，很容易用到。

```
book_id = 1
book = session.query(Book).get(book_id)
print(book and book.to_dict())
```

直接 get(primary_key) 就得到结果

```
{'id': 1, 'cat_id': 1, 'name': '生死疲劳',
 'price': Decimal('40.40')}
```

当然，这样也可以

```
book_id = 1
book = session.query(Book) \
    .filter(Book.id == book_id) \
    .first()
print(book and book.to_dict())
```

不过，还是前一种方式简洁一些。

<!-- more -->

# 2 AND 查询

我们最常用到的就是这种查询，比如我要获取 `cat_id` 为 1 且价格大于 35 的书

```
books = session.query(Book) \
    .filter(Book.cat_id == 1,
            Book.price > 35) \
    .all()
print([v.to_dict() for v in books])
```

执行后，得到结果

```
[{'id': 1, 'cat_id': 1, 'name': '生死疲劳',
  'price': Decimal('40.40')}]
```

filter() 里面的条件默认是使用 AND 进行连接，毕竟这最常用嘛。所以说，换成这样用也是没有问题的

```
from sqlalchemy import and_
books = session.query(Book) \
    .filter(and_(Book.cat_id == 1,
                 Book.price > 35)) \
    .all()
print([v.to_dict() for v in books])
```

不过，通常来说，如果条件全是用 AND 连接的话，没必要显式的去用 `and_`。

如果条件都是等值比较的话，可以使用 `filter_by()` 方法，传入的是关键字参数。 

查询 `cat_id` 等于 1 且价格等于 31.8 的图书，可以这样

```
books = session.query(Book) \
    .filter_by(cat_id=1, price=31.8) \
    .all()
print([v.to_dict() for v in books])
```

结果

```
[{'id': 2, 'cat_id': 1, 'name': '皮囊',
  'price': Decimal('31.80')}]
```

这种方式相对于 filter() 来说，书写要简洁一些，不过条件都限制在了等值比较。

不同情况选择合适的就好。

# 3 常用方法

除了上面使用的 get()、first()、all() 外，还有下面的一些方法比较常用。

* one() 只获取一条记录，如果找不到记录或者找到多条都会报错

```
# 找不到记录会抛如下错误
# sqlalchemy.orm.exc.NoResultFound: No row was found for one()
book = session \
    .query(Book).filter(Book.id > 10) \
    .one()
print(book and book.to_dict())

# 找到多条记录会抛如下错误
# sqlalchemy.orm.exc.MultipleResultsFound: Multiple rows were found for one()
book = session \
    .query(Book).filter(Book.id < 10) \
    .one()
print(book and book.to_dict())

# 正常，得到如下结果
# {'id': 10, 'cat_id': 3, 'name': '万物简史',
#  'price': Decimal('33.20')}
book = session \
    .query(Book).filter(Book.id == 10) \
    .one()
print(book and book.to_dict())
```

* count() 返回记录条数

```
count = session \
    .query(Book) \
    .filter(Book.cat_id == 3) \
    .count()
print(count)
```

结果

```
5
```

* limit() 限制返回的记录条数

```
books = session \
    .query(Book) \
    .filter(Book.cat_id == 3) \
    .limit(3) \
    .all()
print([v.to_dict() for v in books])
```

结果

```
[{'id': 6, 'cat_id': 3, 'name': '时间简史',
  'price': Decimal('31.10')},
 {'id': 7, 'cat_id': 3, 'name': '宇宙简史',
  'price': Decimal('22.10')},
 {'id': 8, 'cat_id': 3, 'name': '自然史',
  'price': Decimal('26.10')}]
```

* distinct() 与 SQL 的 distinct 语句行为一致

```
books = session \
    .query(Book.cat_id) \
    .distinct(Book.cat_id) \
    .all()
print([dict(zip(v.keys(), v)) for v in books])
```

结果

```
[{'cat_id': 1}, {'cat_id': 2},
 {'cat_id': 3}]
```

* `order_by()` 将记录按照某个字段进行排序

```
# 图书按 ID 降序排列
# 如果要升序排列，去掉 .desc() 即可
books = session \
    .query(Book.id, Book.name) \
    .filter(Book.id > 8) \
    .order_by(Book.id.desc()) \
    .all()
print([dict(zip(v.keys(), v)) for v in books])
```

结果

```
[{'id': 10, 'name': '万物简史'},
 {'id': 9, 'name': '人类简史'}]
```

* scalar() 返回调用 one() 后得到的结果的第一列值

```
book_name = session \
    .query(Book.name) \
    .filter(Book.id == 10)\
    .scalar()
print(book_name)
```

结果

```
万物简史
```

* exist() 查看记录是否存在

```
# 查看 ID 大于 10 的图书是否存在
from sqlalchemy.sql import exists
is_exist = session \
    .query(exists().where(Book.id > 10)) \
    .scalar()
print(is_exist)
```

结果

```
False
```


# 4 OR 查询

通过 OR 连接条件的情况也多，比如我要获取 `cat_id` 等于 1 或者价格大于 35 的书

```
from sqlalchemy import or_
books = session.query(Book) \
    .filter(or_(Book.cat_id == 1,
                Book.price > 35)) \
    .all()
print([v.to_dict() for v in books])
```

执行，得到结果

```
[{'id': 1, 'cat_id': 1, 'name': '生死疲劳',
  'price': Decimal('40.40')},
 {'id': 2, 'cat_id': 1, 'name': '皮囊',
  'price': Decimal('31.80')},
 {'id': 4, 'cat_id': 2, 'name': '耶路撒冷三千年',
  'price': Decimal('55.60')},
 {'id': 5, 'cat_id': 2, 'name': '国家宝藏',
  'price': Decimal('52.80')},
 {'id': 9, 'cat_id': 3, 'name': '人类简史',
  'price': Decimal('40.80')}]
```

使用方式和 AND 查询类似，从 sqlalchemy 引入 `or_`，然后将条件放入就 OK 了。

# 5 AND 和 OR 并存的查询

现实情况下，我们很容易碰到 AND 和 OR 并存的查询。比如，我现在要查询价格大于 55 或者小于 25，同时 `cat_id` 不等于 1 的图书

```
from sqlalchemy import or_
books = session.query(Book) \
    .filter(or_(Book.price > 55,
                Book.price < 25),
            Book.cat_id != 1) \
    .all()
print([v.to_dict() for v in books])
```

结果

```
[{'id': 4, 'cat_id': 2, 'name': '耶路撒冷三千年',
  'price': Decimal('55.60')},
 {'id': 7, 'cat_id': 3, 'name': '宇宙简史',
  'price': Decimal('22.10')}]
```

又如，查询图书的数量，图书满足两个要求中的一个即可：一是 `cat_id` 大于 5；二是 `cat_id` 小于 2 且价格大于 40。可以这样

```
from sqlalchemy import or_, and_
count = session.query(Book) \
    .filter(or_(Book.cat_id > 5,
                and_(Book.cat_id < 2,
                     Book.price > 40))) \
    .count()
print(count)
```

结果

```
1
```

# 6 巧用列表或者字典的解包给查询方法传参

开发中，我们经常会碰到根据传入的参数构造查询条件进行查询。比如

* 如果接收到非 0 的 `cat_id`，需要限制 `cat_id` 等于 0
* 如果接收到非 0 的 price，需要限制 price 等于传入的 price
* 如果接收到非 0 的 `min_price`，需要限制 price 大于等于 `min_price`
* 如果接收到非 0 的 `max_price`，需要限制 price 小于等于 `max_price`

我们就可以编写类似的代码

```
# 请求参数，这里只是占位，实际由用户提交的请求决定
params = {'cat_id': 1}

conditions = []
if params.get('cat_id', 0):
    conditions.append(Book.cat_id == params['cat_id'])
if params.get('price', 0):
    conditions.append(Book.price == params['price'])
if params.get('min_price', 0):
    conditions.append(Book.price >= params['min_price'])
if params.get('max_price', 0):
    conditions.append(Book.price <= params['max_price'])
books = session.query(Book).filter(*conditions).all()

print([v.to_dict() for v in books])
```

结果

```
[{'id': 1, 'cat_id': 1, 'name': '生死疲劳',
  'price': Decimal('40.40')},
 {'id': 2, 'cat_id': 1, 'name': '皮囊',
  'price': Decimal('31.80')}]
```

OR 查询类似，将列表解包传给 `or_()` 即可。

如果需求更复杂，AND 和 OR 都可能出现，这个时候根据情况多建几个列表实现。这里只向大家说明大致的思路，就不举具体的例子了。

当然，如果都是等值查询的话，比如只有这两种情况

* 如果接收到非 0 的 `cat_id`，需要限制 `cat_id` 等于 0
* 如果接收到非 0 的 price，需要限制 price 等于传入的 price

可以使用字典的解包给 `filter_by()` 传参

```
# 请求参数，这里只是占位，实际由用户提交的请求决定
params = {'price': 31.1}

condition_dict = {}
if params.get('cat_id', 0):
    condition_dict['cat_id'] = params['cat_id']
if params.get('price', 0):
    condition_dict['price'] = params['price']
books = session.query(Book) \
    .filter_by(**condition_dict) \
    .all()

print([v.to_dict() for v in books])
```

结果

```
[{'id': 6, 'cat_id': 3, 'name': '时间简史',
  'price': Decimal('31.10')}]
```

# 7 其它常用运算符

除了上面看到的 ==、>、>=、<、<=、!= 之外，还有几个比较常用

* IN

```
# 查询 ID 在 1、3、5 中的记录
books = session.query(Book) \
        .filter(Book.id.in_([1, 3, 5])) \
        .all()
```

* INSTR()

```
# 查询名称包含「时间简史」的图书
books = session.query(Book) \
    .filter(Book.name.contains('时间简史')) \
    .all()
```

* `FIN_IN_SET()`

```
# 查询名称包含「时间简史」的图书
# 这里显然应该用 INSTR() 的用法
# FIND_IN_SET() 一般用于逗号分隔的 ID 串查找
# 这里使用 FIND_IN_SET()，旨在说明用法

from sqlalchemy import func
books = session.query(Book) \
    .filter(func.find_in_set('时间简史', Book.name)) \
    .all()
```

* LIKE

```
# 查询名称以「简史」结尾的图书
books = session.query(Book) \
        .filter(Book.name.like('%简史')) \
        .all()
```

* NOT

上面的 IN、INSTR、`FIN_IN_SET`、LIKE 都可以使用 ~ 符号取反。比如

```
# 查询 ID 不在 1 到 9 之间的记录
books = session.query(Book) \
    .filter(~Book.id.in_(range(1, 10))) \
    .all()
```

# 8 查询指定列

查询名称包含「简史」的图书的 ID 和名称。如下

```
books = session.query(Book.id, Book.name) \
    .filter(Book.name.contains('简史')) \
    .all()
print([dict(zip(v.keys(), v)) for v in books])
```

结果

```
[{'id': 6, 'name': '时间简史'},
 {'id': 7, 'name': '宇宙简史'},
 {'id': 9, 'name': '人类简史'},
 {'id': 10, 'name': '万物简史'}]
```

# 9 内连接、外连接

## 9.1 内连接

获取分类为「科技」，且价格大于 40 的图书

```
# 如果 ORM 对象中定义有外键关系
# 那么 join() 中可以不指定关联关系
# 否则，必须要	
books = session \
    .query(Book.id,
           Book.name.label('book_name'),
           Category.name.label('cat_name')) \
    .join(Category, Book.cat_id == Category.id) \
    .filter(Category.name == '科技',
            Book.price > 40) \
    .all()
print([dict(zip(v.keys(), v)) for v in books])
```

结果

```
[{'id': 9, 'book_name': '人类简史',
  'cat_name': '科技'}]
```

统计各个分类的图书的数量

```
from sqlalchemy import func
books = session \
    .query(Category.name.label('cat_name'),
           func.count(Book.id).label('book_num')) \
    .join(Book, Category.id == Book.cat_id) \
    .group_by(Category.id) \
    .all()
print([dict(zip(v.keys(), v)) for v in books])
```

结果

```
[{'cat_name': '文学', 'book_num': 2},
 {'cat_name': '人文社科', 'book_num': 3},
 {'cat_name': '科技', 'book_num': 5}]
```

# 9.2 外连接

为方便说明，我们仅在这一小节中向 books 表中加入如下数据

```
+----+--------+-----------------+-------+
| id | cat_id | name            | price |
+----+--------+-----------------+-------+
| 11 |      5 | 人性的弱点      | 54.40 |
+----+--------+-----------------+-------+
```

查看 ID 大于等于 9 的图书的分类信息

```
# outerjoin 默认是左连接
# 如果 ORM 对象中定义有外键关系
# 那么 outerjoin() 中可以不指定关联关系
# 否则，必须要
books = session \
    .query(Book.id.label('book_id'),
           Book.name.label('book_name'),
           Category.id.label('cat_id'),
           Category.name.label('cat_name')) \
    .outerjoin(Category, Book.cat_id == Category.id) \
    .filter(Book.id >= 9) \
    .all()
print([dict(zip(v.keys(), v)) for v in books])
```

结果

```
[{'book_id': 9, 'book_name': '人类简史',
  'cat_id': 3, 'cat_name': '科技'},
 {'book_id': 10, 'book_name': '万物简史',
  'cat_id': 3, 'cat_name': '科技'},
 {'book_id': 11, 'book_name': '人性的弱点',
  'cat_id': None, 'cat_name': None}]

```

注意最后一条记录。

# 10 打印 SQL

当碰到复杂的查询，比如有 AND、有 OR、还有连接查询时，有时可能得不到预期的结果，这时我们可以打出最终的 SQL 帮助我们来查找错误。

以上一节的外连接为例说下怎么打印最终 SQL

```
q = session \
    .query(Book.id.label('book_id'),
           Book.name.label('book_name'),
           Category.id.label('cat_id'),
           Category.name.label('cat_name')) \
    .outerjoin(Category, Book.cat_id == Category.id) \
    .filter(Book.id >= 9)

raw_sql = q.statement \
    .compile(compile_kwargs={"literal_binds": True})
print(raw_sql)
```

其中，q 为 sqlalchemy.orm.query.Query 类的对象。

结果

```
SELECT books.id AS book_id, books.name AS book_name, categories.id AS cat_id, categories.name AS cat_name 
FROM books LEFT OUTER JOIN categories ON books.cat_id = categories.id 
WHERE books.id >= 9
```

至此，SQLAlchemy ORM 常用的一些查询方法和技巧已介绍完毕，希望能帮助到有需要的朋友。
