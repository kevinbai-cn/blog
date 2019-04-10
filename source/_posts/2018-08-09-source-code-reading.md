---
title: 说下自己看源码的一点经验
date: 2018-08-09 18:30:58
categories:
- 个人随笔
tags:
---

开发多了，工具用多了，难免会对一些库的源码感到好奇，想知道怎么里面都用了些什么技巧、知识。但是很多时候，我们会感觉到无从下手，下面我说下自己的一点点经验。

首先，要看的肯定是我们工作中经常用到的库，这样，你至少知道这个库到底有些什么功能。如果连有什么功能你都不清楚，那么你看源码肯定是没有目的性的。不带着问题去看源码，很容易迷失在其中，这样即使花时间了，效果也并不好。

然后，刚开始阅读一些库的源码的时候，最好选一些代码量少的先感受一下。不要认为代码量少就没必要看，其实很多小项目的代码风格和技巧，都是值得我们学习的。在后面的大项目中，我们可能也会碰到类似的技巧，这个时候我们相对有经验一些了。

最后，不管是小项目还是大项目，不管你是查资料还是通过其它方式，心里最好有一个大致结构。我们很容易碰到一个点读不懂，这个时候，我们没必要去死磕，先了解它的功能是啥。当把整个项目的点串起来的时候，你就对项目的设计思想有了进一步的认识。至于之前没看懂、落下的点，可能是某些技巧之前没接触过，也有可能是某些理论知识缺乏，这个时候我们可以慢慢去弥补。这样一来，我们才算是有一定提高。

<!-- more -->

具体该去看些什么库呢？我结合自身推荐几个，希望能触发大家的灵感。

看过我之前文章的都知道，支撑导出工作的时候，我比较喜欢用 kennethreitz/records 库。有个时候比较空，想看下究竟是怎么实现的，刚好，这个库代码也不多，核心逻辑还是一个单文件，就把它给看完了。看完后，感觉还是 Get 到了一些代码技巧。与这个库有关的另一个库 kennethreitz/tablib 代码量也不多，感兴趣的也可以去看下。还有我之前提过的 gleitz/howdoi 库，也是单文件，去看下也有好处。类似的项目也不是说看的越多越好，找几个公认的比较好的就行。

我主要是写后端的，所以用 Flask、SQLAlchemy、Celery 等比较多。当碰到的业务需求多了后，会慢慢的发现自己有很多细节不大确定。比如 Flask 可以用多线程跑，但是框架又是怎么保证当前的 request 全局变量就是当前线程的呢？看了源码后才知道有一个 Local 类，用了点小技巧（线程 ID）让我们看起来线程之间的数据是隔离着的。具体 Flask 怎么支持的 WSGI，我们可以去看 Werkzeug 库；怎么实现模版渲染的，可以去看下 Jinja 库。带着问题的时候去看代码，效果好很多。慢慢的，我们便清楚了 Flask 的大致结构，这个时候我们就可以去找一些之前可能没有理解的点进行深挖。如果你是做后端的，我推荐你可以先看下 Flask、Werkzeug、Requests 库，然后再结合自身情况看下 SQLAlchemy、Celery 等其它库。

知道了看哪些库，又怎么开始呢？拿 Flask 示范一下。

首先，将库克隆到本地，进入项目根目录，执行命令

```
pip install --editable .
```

将 Flask 安装在当前目录，方便我们修改测试。

最后，跑一个 Demo。进入 examples/flaskr 目录，按照 README 操作

（1）Edit the configuration

（2）pip install --editable .

（3）export FLASK_APP=flaskr

（4）flask initdb

（5）flask run

最后，打开链接 `http://localhost:5000` 可以看到程序已经正常启动。如果碰到问题，百度／谷歌一般都能解决。

到这，就去看源码中你感兴趣的地方吧。如果有必要，可以修改框架或 flaskr 中的代码后，重新安装运行，验证自己的想法。