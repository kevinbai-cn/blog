---
title: 采访 Python 大佬 Kenneth Reitz
date: 2018-11-22 11:22:35
categories:
- 翻译
tags:
- Kenneth Reitz
---

Kenneth Reitz，我想做 Python 开发的应该都知道这个名字。他负责了多个很受 Python 社区欢迎的类库，比如 requests/requests、pypa/pipenv 等。本文是 Real Python 社区的管理人员 Ricky White 对 Kenneth 采访内容的翻译，主要聊聊 Kenneth 最新的项目以及他迄今为止编写的最具挑战性的代码。

<!-- more -->

**Ricky：**
让我们从头开始......你是怎么接触编程，什么时候开始使用 Python 的？

**Kenneth：**
我从小就开始编程了。我的父亲是一名程序员，我在 9 岁时自学了 BASIC 和 C。当我在大学上第一节计算机课程的时候，我开始使用 Python。不久之后，我退学并学习了许多其他编程语言，但我总是回到 Python。

**Ricky：**祝贺您在 Digital Ocean 作为团队中的高级成员开始新的工作。您觉得你现在和在 Heroku 工作时相比，角色上有些什么变化？Digital Ocean 对促进 Python 发展可能会做些什么吗？

**Kenneth：**
谢谢！我很享受现在的新角色和服务整个开发社区的机会，而不仅仅是 Python 社区。我最近开发的一个网站服务框架 Responder 刚好是 Digital Ocean 的一个项目，你可以期待下这个项目对 Python 发展的影响 😊。

**Ricky：**
你很优秀，编写了极受欢迎的 Requests 和新的 Pipenv 库，并且 python.org 现在也倡议使用 Pipenv 进行包依赖管理。你觉得社区对 Pipenv 的接受程度如何？您是否碰到过社区的阻力，有些开发者更愿意坚持使用 venv 或更旧的依赖管理方法？

**Kenneth：**
社区已经很好地接受了 Pipenv，甚至像 GitHub 这样的公司也在使用其标准进行安全漏洞扫描。除了 Reddit 的一些用户的反对之外，我没有碰到社区的太多阻力。不过，我花了一段时间才意识到 /r/python 并不代表 Python 社区，它仅仅代表了使用了 Python 的 Reddit 的部分用户。

**Ricky：**现在你的 Requests 库已经被下载超过 3 亿次，这是多么酷炫的一件事啊。不过，作为一个吉他手，我更兴奋的是你的最新项目 PyTheory。你能告诉我们一些关于它和你未来项目的计划吗？

**Kenneth：**
PyTheory 是一个非常有趣的库，试图将所有已知的音乐系统封装到库中。目前，有一个系统：Western。它可以以编程方式呈现 Western 系统的所有不同比例，并告诉您音符的音高（以十进制或符号表示）。此外，还有指板和和弦图表，因此您可以为吉他指定自定义调音，并使用它生成和弦图表。这很抽象。

这绝对是我写过的最具挑战性的东西。

**Ricky：**嗯！最近你好像在从 Mac 转向 PC 工作，并且使用微软的 VS Code 进行 Python 开发。作为一个 Windows 用户，你是否感到高兴和自豪？对于那些正在看这篇文章却没使用过 Windows 95 及其以后系统的用户，他们错过了些什么？

**Kenneth：**相比 Windows，其实我还是更喜欢 Mac。最近也只是无聊，想挑战一下自己。不过，用起来还是比较愉快、高效，在家里感觉很好。

Windows 不像以前那样，它现在是一个真正可靠的操作系统。我在我的 iMac Pro 上运行它，它像一个梦想一样嗡嗡作响。

**Ricky：**
我知道你是一个富有热情的摄影师，你接触摄影多久了？你拍过的最喜欢的一张照片是什么？除了 Python 之外，你还有其它兴趣爱好吗？

**Kenneth：**
我已经认真对待摄影大约 10 年了，我拍过的最喜欢的照片可能就是这张。这是我在患有偏头痛几周之后，第一次能够走出去时拍的，当时使用的是胶片相机。

{% fi 1.jpg, https://500px.com/photo/54603002/seasonal-harmonies-by-kenneth-reitz, https://500px.com/photo/54603002/seasonal-harmonies-by-kenneth-reitz, 40% %}

**Ricky：**采访就快结束了，你还有什么想对大家说的吗？

**Kenneth：**Responder！我最近新写的 Python Web 服务框架。它遵循我们所熟悉的 ASGI 规范，速度快，而且比 Flask 更容易使用！非常适合用来构建 API。

**原文**

`https://realpython.com/interview-kenneth-reitz`

---

另外，Kenneth Reitz 还有一段从胖子到男神的励志故事，感兴趣的可以去看下这篇文章《谁说程序员不是潜力股？让这位世界前五名的天才程序员来颠覆你三观！》。

`https://mp.weixin.qq.com/s/Nc8CFRA_HLurIYHXw-KWEA`
