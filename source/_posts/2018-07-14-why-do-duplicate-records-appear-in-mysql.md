---
title: 数据库存数据时，逻辑上防重了为啥还会出现重复记录？
date: 2018-07-14 19:08:01
categories:
- 数据库
tags:
- MySQL
- 事务
- 隔离级别
---

在很多异常情况下，比如高并发、网络糟糕的时候，数据库里偶尔会出现重复的记录。

假如现在有一张书籍表，结构类似这样

```
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
+----+--------------+
```

在异常情况下，可能会出现下面这样的记录

```
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
|  2 | 人类简史     |
|  3 | 人类简史     |
+----+--------------+
```

但是，想了想，自己在处理相关数据的时候也加了判重的相关逻辑，比如，新增时当图书 name 相同时，会提示图书重复而返回。

初次遇到这个情况的时候，感觉有点摸不着头脑，后面想了想，还是理清了，其实这和数据库的事务隔离级别有一定关系。

先简单说下数据库事务的 4 个隔离级别，然后重现下上述问题，最后说说解决办法。

# 1 数据库事务的 4 个隔离级别

# 1.1 未提交读

顾名思义，当事务隔离级别处于这个设置的时候，不同事务能读取其它事务中未提交的数据。

便于说明，我开了两个客户端（A 以及 B），并设置各自的隔离级别为未提交读。（并没有全局设置）

设置隔离级别命令

```
SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}
```

好了，开始。

**Client A**

```
mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@session.tx_isolation;
+------------------------+
| @@session.tx_isolation |
+------------------------+
| READ-UNCOMMITTED       |
+------------------------+
1 row in set (0.00 sec)

mysql> start transaction;
Query OK, 0 rows affected (0.01 sec)

mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
+----+--------------+
1 row in set (0.00 sec)

mysql> insert into books(name) values('人类简史');
Query OK, 1 row affected (0.01 sec)

mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
|  4 | 人类简史     |
+----+--------------+
2 rows in set (0.00 sec)
```

当 A 中的事务没有关闭的时候，我们去 B 中看下数据

**Client B**

```
mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@session.tx_isolation;
+------------------------+
| @@session.tx_isolation |
+------------------------+
| READ-UNCOMMITTED       |
+------------------------+
1 row in set (0.00 sec)

mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
|  4 | 人类简史     |
+----+--------------+
2 rows in set (0.00 sec)
```

B 中可以读取 A 未提交的数据，所谓未提交读就是这样。

最后，记得把各个事务提交。

**Client A & Client B**

```
mysql> commit;
```

<!-- more -->

# 1.2 提交读

不能事务可以读取其它事务中已经提交的数据。

篇幅问题，这里我就不贴出设置隔离级别的语句，测试某个隔离级别的时候，默认已经设置好该级别。

**Client A**

```
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
+----+--------------+
1 row in set (0.00 sec)

mysql> insert into books(name) values('人类简史');
Query OK, 1 row affected (0.00 sec)

mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
|  5 | 人类简史     |
+----+--------------+
2 rows in set (0.00 sec)
```

A 没提交，在 B 里面去看下数据

**Client B**

```
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
+----+--------------+
1 row in set (0.00 sec)
```

和预期一样，A 中未提交的数据在 B 中看不到。

A 中提交事务

**Client A**

```
mysql> commit;
```

在 B 中看下

**Client B**

```
mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
|  5 | 人类简史     |
+----+--------------+
2 rows in set (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)
```

B 中能看到 A 中提交的数据。

# 1.3 可重复读

细心的朋友可能会发现一个问题，那就是在 B 中的同一个事务读同一个表，得到的结果却不一致，开始只有 1 条，后面有 2 条，而如果没有这个问题的话，也就是可重复读了。

我们来验证下

**Client A**

```
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
+----+--------------+
1 row in set (0.00 sec)

mysql> insert into books(name) values('人类简史');
Query OK, 1 row affected (0.01 sec)

mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
|  6 | 人类简史     |
+----+--------------+
2 rows in set (0.00 sec)
```

**Client B**

```
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
+----+--------------+
1 row in set (0.00 sec)
```

**Client A**

```
mysql> commit
Query OK, 0 rows affected (0.00 sec)
```

**Client B**

```
mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
+----+--------------+
1 row in set (0.00 sec)
```

和预期一致。B 中事务没有受到 A 中事务的提交影响，读取的数据和事务刚开始的时候一致，books 中都只有一条数据，这就是可重复读。

当然，B 在自己的事务中做修改，肯定是可见的。

**Client B**

```
mysql> insert into books(name) value ('时间简史');
Query OK, 1 row affected (0.00 sec)

mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
|  8 | 时间简史     |
+----+--------------+
2 rows in set (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)
```

# 1.4 串行化

这是隔离级别最严格的一级，在该级别中，不同事务中的读写会相互阻塞。

**Client A**

```
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
+----+--------------+
1 row in set (0.00 sec)
```

当 A 未提交的时候在 B 中对同一个表进行写

**Client B**

```
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from books;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
+----+--------------+
1 row in set (0.00 sec)

mysql> insert into books(name) value ('人类简史');
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```

由于不同事务中的读写相互阻塞，所以出现了上面超时的情况。

如果 A 中提交事务

**Client A**

```
mysql> commit;
Query OK, 0 rows affected (0.00 sec)
```

那么在 B 中就能正常写了

**Client B**

```
mysql> insert into books(name) value ('人类简史');
Query OK, 1 row affected (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.01 sec)
```

同理，在 A 中开启事务并向 books 中插入一条记录后不提交，B 中开启事务并对该表进行读操作，也会超时。当 A 中的事务提交后，B 中对 books 的读操作就没有问题了。

# 2 重现问题

由于 MySQL 的 Innodb 的默认事务隔离级别为可重复读，也就导致了判重逻辑可能会出现问题，我们来重现一下。

现在，数据库的数据是这样的

```
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
+----+--------------+
```

后端逻辑类似这样的

```
try:
	book_name = '人类简史'
	book = get_by_name(book_name)
	if book:
		raise Exception(f'图书 {book_name} 已存在')

	# 新增操作
	# 其它操作

	db.session.commit()
	return {'success': True}
except Exception as e:
	db.session.rollback()
	return {'success': False, 'msg': f'新增图书失败 {e}'}
```

当两个用户输入书名「人类简史」并提交后，同时有两个线程执行这段逻辑，也就相当于上面两个客户端同时开启了事务，我们以这两个客户端来说明问题

**Client A**

```
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from books where name = '人类简史';
Empty set (0.00 sec)

mysql> insert into books(name) values('人类简史');
Query OK, 1 row affected (0.00 sec)
```

A 中检测图书不存在，然后插入，但是由于「其它操作」由于网络或者其它原因太费时间，导致事务提交延迟。

这时在 B 中执行类似操作

**Client B**

```
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from books where name = '人类简史';
Empty set (0.00 sec)

mysql> insert into books(name) values('人类简史');
Query OK, 1 row affected (0.00 sec)
```

由于事务隔离级别是可重复读的，B 中无法读取 A 中未提交的数据，所以判重逻辑顺利通过，也插入了同一本书。（也就是说隔离级别在提交读及以上都有可能出现这个问题）

最后 A 和 B 都提交后

**Client A & Clinet B**

```
mysql> commit;
Query OK, 0 rows affected (0.01 sec)
```

就出现了重复记录了

```
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 世界简史     |
| 12 | 人类简史     |
| 13 | 人类简史     |
+----+--------------+
```

# 3 怎么解决

# 3.1 数据库层面

从底层进行限制，对 name 添加唯一索引后，插入重复记录会报错，简单粗暴的解决了这个问题。

# 3.2 代码层面

加唯一索引能解决，但是总觉得代码不够完整，其实在代码层面也可以解决这个问题。

如果我们在接收请求的时候如果碰到关键参数相同的请求，我们可以直接拒绝，返回类似「操作进行中」的响应，这样也就从源头上解决了这个问题。

实现上面的思路也很简单，借助 redis 的 setnx 即可。

```
book_name = request.form.get('book_name', '')
if not book_name:
	reutrn json.dumps({'success': False, 'msg': '请填写书名'})

redis_key = f'add_book_{book_name}'
set_res = redis_client.setnx(redis_key, 1)
if not set_res:
	reutrn json.dumps({'success': False, 'msg': '操作进行中'})

add_res = add_book(book_name)  # 添加操作

redis_client.delete(redis_key)
return json.dumps(add_res)
```

如果类似场景比较多，可以考虑把 redis 的操作封装成一个装饰器，让代码能复用起来，这里不再赘述。

# 4 小结

由于数据库隔离级别的原因，一些数据就算是逻辑上进行防重了，也有可能出现重复记录。解决这个问题，可以在数据库层面加唯一索引解决，也可以在代码层面进行解决。
