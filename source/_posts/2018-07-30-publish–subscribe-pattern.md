---
title: 聊聊发布订阅模式
date: 2018-07-30 10:47:31
categories:
- 设计模式
tags:
- 发布订阅模式
---

发布订阅模式在开发中还比较常用，今天我们就来聊聊这个模式。

<!-- more -->

# 1 概念

怎么理解？说到发布订阅的时候，很多人会以邮件订阅的例子来说明大致的思路，但我觉得还可以找一个更贴近人们生活的例子。虽说现在已经是网络时代了，但是订报的人还是有很多，发报订报这个过程和发布订阅模式特别类似，我画了张图说明了大致的过程。

{% fi 1.png,,, 80% %}

**注意：报社名称是我随便填的**

如图所示，发报订报这个过程涉及 3 个对象，分别是报社、订报人、邮局。小明如果要订时代周报和环球时报的话，他就要去邮局登记自己订阅的报纸名称和住址。在固定时间，时代周报报社或者环球时报报社会向邮局送报纸过去。邮局收到报纸后，又会分发到对应订报人的手里。小军呢，过程与小明类似。

上面发报订报的图，稍作修改，就可以当作一个发布订阅模式的结构图了。

{% fi 2.png,,, 80% %}

分为 3 个部分，发布者（Publisher）、订阅者（Subscriber）、中间人（Broker）。中间人一般能处理多个频道（Channel），订阅者订阅某个频道后，当中间人收到发布者发布的对应频道的信息后，会将信息转发给相应的订阅者。发布者、订阅者的数量都可以是一个或多个。

# 2 实践

***

**注意：本文 Python 代码在以下环境测试通过**

* Python 3.6.0
* redis 2.10.6（这里指的的包版本）

***

这里我们使用 Redis 作为中间人来实现一个简单发布订阅应用，来模仿一下订报发报的过程。

首先是小明需要订阅时代周报和环球时报

**subscriber_xiaoming.py**

```
# coding=utf-8

import redis

rd = redis.Redis()
sub = rd.pubsub()
sub.subscribe(['时代周报', '环球时报'])
for msg in sub.listen():
    if msg['type'] == 'message':
        channel = msg['channel'].decode('utf-8')
        data = msg['data'].decode('utf-8')
        print(f'{channel}: {data}')
```

小军订阅环球时报和南方周末类似

**subscriber_xiaojun.py**

```
# coding=utf-8

import redis

rd = redis.Redis()
sub = rd.pubsub()
sub.subscribe(['环球时报', '南方周末'])
for msg in sub.listen():
    if msg['type'] == 'message':
        channel = msg['channel'].decode('utf-8')
        data = msg['data'].decode('utf-8')
        print(f'{channel}: {data}')
```

然后是报社发行报纸，时代周报报社

**publisher_sdzb.py**

```
# coding=utf-8

import redis

rd = redis.Redis()
channel = '时代周报'.encode('utf-8')
message = f'123 期 Facebook 在华子公司' \
          f'遭撤销 ...'.encode('utf-8')
rd.publish(channel, message)
```

环球时报报社

**publisher_hqsb.py**

```
# coding=utf-8

import redis

rd = redis.Redis()
channel = '环球时报'.encode('utf-8')
message = f'123 期 习近平会见毛里求斯总理' \
          f'贾格纳特 ...'.encode('utf-8')
rd.publish(channel, message)
```

我们分别在两个终端中启动订阅者

小明

```
python subscriber_xiaoming.py
```

小军

```
python subscriber_xiaojun.py
```

然后我们再开一个终端发行报纸

```
python publisher_hqsb.py
python publisher_hqsb.py
```

结果，和预期一致，小明收到了时代周报和环球时报的报纸

```
时代周报: 123 期 Facebook 在华子公司遭撤销 ...
环球时报: 123 期 习近平会见毛里求斯总理贾格纳特 ...
```

小军收到了环球时报的报纸

```
环球时报: 123 期 习近平会见毛里求斯总理贾格纳特 ...
```

# 3 应用场景

这个模式的可以应用的地方很多，这里说几个常见的。

（1）视频和电商等网站都有类似「更新了通知我」的功能，就可以使用发布订阅的模式来完成。比如某视频网站的某个电视剧，如果选中了「更新时及时通知我」，就相当于订阅了某个电视剧，当后台更新电视剧时，订阅者们都能收到相应消息。电商网站上的「降价了通知我」的功能也类似。

（2）聊天室。一个聊天室作为一个频道，聊天的人们同时是订阅者和发布者。

（3）微服务。微服务为了完成最终一致性，常常都会维护一些任务表，其中的任务处理也涉及订阅发布模式。这个场景中，订阅者都是一些任务处理程序，比如库存处理等，它们会订阅特定的频道。发布者是某些监控程序，当它监听到任务表的更新或创建时，会通过表中的信息找到合适的频道然后发布消息。这样一来，对应的任务处理也就被触发了。当然，这里面还包括失败重试等其它机制，这里就不详述了。

# 4 延伸

Redis 作为发布订阅的中间人有如下缺点

（1）不可靠。如果一个订阅者掉线，不巧发布者这个时候发了一个消息，这个时候即使订阅者重新连接，同样也接收不到消息了。

（2）耗资源。每个订阅者都要占用一个 Redis 连接。

所以，很多时候都用消息队列来作为中间人，比如 ZeroMQ、RabbitMQ 等。当然，如果你的需求能容忍上面的缺点，那么选择 Redis 也没问题，还更简单粗暴一些。
