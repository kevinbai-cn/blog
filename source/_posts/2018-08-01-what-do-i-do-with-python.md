---
title: 我用 Python 做些什么？
date: 2018-08-01 18:35:31
categories:
- Python
tags:
- Python
---

我主要工作是后端，所以这一点我就不说了。本文主要说下我用 Python 做的其它的一些事。

# 1 数据导出支撑

一个月可能有一两次需要导出一些数据，每次的需求都有些不同。刚开始的时候还好，用数据库的一些连接工具，比如 Sequel Pro 和 DataGrip 等，直接导出就能满足要求。后面呢，有些字段需要自定义又或者是要连表，用那些工具不大方便，于是就只能自己写脚本了。网上找了找，看到 Kenneth Reitz 大牛写的一个 records 库，用起来超级爽。

执行 SQL 简单粗暴

```
import records

db = records.Database('mysql+pymysql://'
                      'username:password@host:port'
                      '/database?charset=utf8')
rows = db.query('select * from books') 
```

读出的结果使用也比较方便，迭代时使用字典或对象的方式可以直接获取相应值，相比一些类库需要自己转换方便得多

```
for r in rows:
    print(r['name'], r['price'])
    # 或者
    # print(r.name, r.price)
```

导出格式支持也比较丰富，有 CSV、YAML、xls 等。比如要导出 xls 文件

```
with open('books.xls', 'wb') as f:
    f.write(rows.export('xls'))
```

当然，由于这个库使用起来简单、灵活，除了导出之外，还做了一些其它的事。比如，某个表某些数据有异常，要找出是哪几条，纯写 SQL 太麻烦的时候，使用这个库也能解决问题。

平时做一些支撑性工作的时候，通过这个库还是省了不少时间。

<!-- more -->

# 2 简单的代码自动生成

开发时我们经常发现某些代码结构特别类似，自己一行一行的敲感觉太浪费时间，这个时候我们应该考虑用脚本来处理。

比如，新增一条数据时，如果用的是 SQLAlchemy 的话，可能会碰到类似的语句

```
def create(book):
	new_book = Book()
	new_book.name = book.get('name', '')
	new_book.price = book.get('price', 0)
	# ...
	db.session.add(new_book)
	db.session.flush()
```

如果当表的字段特别多的时候，自己一行一行的敲完太痛苦。所以，我们可以写个脚本自动生成一下

```
# coding=utf-8

import records

db = records.Database('mysql+pymysql://'
                      'username:password@host:port'
                      '/database?charset=utf8')

books = db.query('select * from books limit 1')
columns = list(books.as_dict()[0].keys())

for column in columns:
    print(f"new_book.{column} ="
          f" book.get('{column}', '')")
```

执行后得到下面的代码

```
new_book.id = book.get('id', '')
new_book.cat_id = book.get('cat_id', '')
new_book.name = book.get('name', '')
new_book.price = book.get('price', '')
```

将上面的复制到需要的地方，然后把 get() 的默认值自己根据类型手动调下，就搞定了。

当然，如果在字段不多的情况下，这样做反而更浪费时间了。自己根据需要选择就好。

这里，只是举个例子，工作中碰到重复性的任务多的时候，不妨考虑写个脚本。刚开始可能费点时间，后面可是效率的大大提升。

# 3 消息推送

某些时候，我可能会关注些什么东西，想在合适的时候提醒我下。比如，我同时关注了好几个电商平台上的同一款手机，我希望在这几个平台的最低价低于多少的时候提醒我下，我就会写个脚本去解决。

思路很简单，写个简单的爬虫每隔一段时间获取一下这几个平台的这款手机的价格，取最小值，当最小值低于期望值的时候，就告诉我，可以发微信、短信或推送消息到 APP 等。使用微信比较繁琐，使用短信的话免费的又不靠谱，所以我喜欢推送消息到 APP。最后，将脚本放到服务器上一直跑着。

这里，我推荐个网站 IFTTT（`https://ifttt.com`），正如其名字（IF This Then That）那样，推送工具其实就是在这个网站上新建一个应用 （ Applet ）并定义好事件（ Event ）与消息，当相应事件发生时推送消息给手机上的 App，当然这个 App 是网站提供的，目前安卓、IOS 都支持。具体怎么使用，可以看下这篇文章 `https://realpython.com/python-bitcoin-ifttt` 的这个小节 `Sending a Test IFTTT Notification`。

# 4 其它

Python 能解决很多日常的小问题，上面只是部分。不同方向、不同兴趣的开发者，弄出的小玩意也不尽相同。比如，之前看《东京喰种》的漫画，在网页上看加载太慢了，一怒，写个脚本开下多线程一分钟不到几百话全下好了，在本地流畅的看。之前在 GitHub 上也看到一个下载羞羞视频的库，都不用自己去找资源了，不过不好意思，库的名字搞忘了。总之，了解一下 Python，即使你不是做相关开发的，它也能帮你解决不少问题。
