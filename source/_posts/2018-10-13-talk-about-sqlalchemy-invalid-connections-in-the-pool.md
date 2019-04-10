---
title: 聊聊 SQLAlchemy 连接池中的连接失效问题
date: 2018-10-13 16:42:52
categories:
- Python
tags:
- Python
- SQLAlchemy
- MySQL
- 连接池
---

最近项目中事情比较多，也遇到了一些问题，其中有一个是关于连接池的，比较有意思，这里分享下。

一天早上，进入业务系统，点击了一个功能按钮，页面上突然弹出个 MySQL gone away 的错误，我擦，数据库挂了吗，上服务器一看正常的。又点击了一下，又报事务未正常关闭的错误，有点懵。当然，是在测试环境上 :)。仔细想了想，发现是连接池的问题，下面我重现下这个错误并说下自己的一些解决办法。

<!-- more -->

**注意：文中代码测试环境**

* Python 3.6.0
* PyMySQL 0.9.2
* SQLAlchemy 1.2.12
* Flask 1.0.2

便于说明，假设应用程序代码如下

```
# coding=utf-8

# ====== SQLAlchemy ======

from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Numeric
from sqlalchemy.orm import sessionmaker

Base = declarative_base()
engine = create_engine('mysql+pymysql://username:password'
                       '@host:port/db_name?charset=utf8')
Session = sessionmaker(bind=engine)

session = Session()


class Book(Base):
    __tablename__ = 'books'

    id = Column(Integer, primary_key=True)
    cat_id = Column(Integer)
    name = Column('name', String(120))
    price = Column('price', Numeric)

# ====== Flask ======

from flask import Flask

app = Flask(__name__)


@app.route('/')
def index():
    book_num = session.query(Book).count()
    return f'Book num: {book_num}'


if __name__ == '__main__':
    app.run(port=8080, host='0.0.0.0', debug=True)
```

数据表 books 内容如下

```
+----+--------------------------+
| id | name                     |
+----+--------------------------+
|  1 | 生死疲劳                 |
|  2 | 皮囊                     |
|  3 | 半小时漫画中国史         |
|  4 | 耶路撒冷三千年           |
|  5 | 国家宝藏                 |
|  6 | 时间简史                 |
|  7 | 宇宙简史                 |
|  8 | 自然史                   |
|  9 | 人类简史                 |
| 10 | 万物简史                 |
+----+--------------------------+
```

应用中，我们使用 SQLAlchemy ORM 操作数据库，当 create_engine 使用默认参数的时候，连接池是打开着的。对大部分数据库来说，poolclass 默认为 QueuePool。当一个请求进来，SQLAlchemy 会创建一个数据库连接，执行结束后把连接放回池子里。下一个请求来的时候，就可以直接使用之前的连接。当然，如果同时进来多个不够分配的时候，会创建另外的连接用于使用，执行结束后又放回池子里。池子里的最大连接数是可以配置的。这种方式可以避免频繁创建、销毁连接，从而提高执行效率。

但是，这带来另一个问题。当数据库突然挂掉或者数据库过一定时间清理未活动连接的时候，SQLAlchemy 是不知道的。当一个请求进来时，会被分配一个失效的连接，自然会抛出一些异常。

**备注：**MySQL 中使用如下命令查看未活动连接过期时间

```
show variables like "interactive_timeout";
```

结果类似这样

```
+---------------------+-------+
| Variable_name       | Value |
+---------------------+-------+
| interactive_timeout | 28800 |
+---------------------+-------+
```

单位是秒。

下面，我们重现下错误。

执行应用程序后，我们首先访问下这个链接

```
http://127.0.0.1:8080
```

此时，连接池里有一个连接了。

在继续下一步之前，我们先关闭或者重启下数据库，然后请求，抛出了无法连接数据库的错误

```
sqlalchemy.exc.OperationalError:
(pymysql.err.OperationalError)
(2013, 'Lost connection to MySQL server during query')
```

因为被分配的连接由于数据库的重启已经失效了嘛。

**注意：**这里和我之前在公司测试环境上碰到的 MySQL gone away 的错误不一致，估计是由于示例应用与实际项目的结构、类库版本等方面的差异原因，不过错误的意思都差不多。

我们再执行第三次请求，发现又报了事务相关的错误，这是由于上一个错误导致连接未正常关闭引起的。

```
sqlalchemy.exc.StatementError:
(sqlalchemy.exc.InvalidRequestError)
Can't reconnect until invalid transaction is rolled back
```

碰到这种问题，我们该如何解决呢？究其原因，就是连接池的连接在不知情的情况下在数据库服务器上被关掉了。这个大致分为两种情况：

一、数据库异常或者误操作，比如数据库挂掉、重启、有的连接被误操作给 kill 掉了等等。这种情况下只好重启应用程序，让应用重新维护连接池；

二、连接长时间未活动。这又有两种情况：

1、create_engine 时指定的连接池中的连接的回收时间大于数据库配置的未活动连接过期时间，由于连接的回收时间一般都是设置的一两个小时，而数据库的未活动连接过期时间默认是八个小时，所以这种情况一般不会出现；

2、应用程序里的逻辑代码有问题，在进行数据库的写入操作后，缺少 commit、rollback、close 的操作将连接放回连接池，连接池没法管理，当这个连接被数据库回收后，也就出现了上面的异常。这种情况，有以下几个解决办法：

（1）查看相关的业务，补充缺失的连接维护操作（推荐）；

（2）可以通过重启应用程序解决，不过指标不治本，运行一段时间后，问题依旧会出现；

（3）SQLAlchemy 开启 autocommit，不过这样就不能手动 rollback 了，在很多插入、更新场景中，不大实用；

（4）彻底一点的，直接不用连接池，在 create_engine 时指定参数 poolclass 为 NullPool。不过连接的使用效率就不如之前了，自己权衡。
