---
title: 使用 Python 发送邮件
date: 2018-11-09 08:30:31
categories:
- Python
tags:
- Python
- 邮件发送
---

开发中，使用到邮件发送的场景特别多，比如用户注册、找回密码、推广等等。本文简单介绍下使用 Python 发送邮件的方法，不涉及协议那些理论知识，基本上看完就能使用。

<!-- more -->

# 1 准备

和自己手动发电子邮件一样，首先要准备好这些东西：邮箱账号、密码、邮件主题、内容、收件人邮箱。

需要注意的是，我们发送邮件需要使用邮件服务商的 SMTP 服务，而这个一般默认是关闭的，我们需要先开启。

这里以 163 邮箱为例进行说明，首先登陆并找到 SMTP 的设置位置

{% fi 1.png,,, 60% %}

开启 SMTP

{% fi 2.png,,, 60% %}

设置授权码

{% fi 3.png,,, 60% %}

{% fi 4.png,,, 60% %}

确定后会提示你是否另外开启 POP3 和 IMAP 服务，这两个服务都是是用于第三方客户端接收邮件使用的。发送邮件并不会使用到这两个服务，如果你有接收邮件的需求则需要开启其中的一个或两个，看你情况自行选择。

{% fi 5.png,,, 60% %}

至此 SMTP 成功开启。

这里也要注意下，大部分平台邮箱账号登陆第三方客户端的时候，使用的是和登录密码不同的授权码，有小部分是相同的，根据你的邮件平台自行确定。

到这里我们有了如下信息

```
# 邮箱账号
sender = 'xiaoming@163.com'
# 授权码
password = 'xiaoming123'
# SMTP 服务器地址（上面的设置页面可以查看）
smtp_server = 'smtp.163.com'
# SMTP 默认端口是 25
smtp_port = 25
# 邮件主题
subject = '小明测试'
# 内容
conent = '你好！...'
# 收件人地址
receiver = 'xiaogang@qq.com'
```

# 2 发送文本邮件

Python 内置了对 SMTP 的支持，email 包用来构造邮件，smtplib 包用来发送邮件。

```
# coding=utf-8

import smtplib

from email.mime.text import MIMEText
from email.header import Header

msg = MIMEText(conent, 'plain', 'utf-8')
msg['Subject'] = Header(subject, 'utf-8')
msg['From'] = sender
msg['To'] = receiver

server = smtplib.SMTP(smtp_server, smtp_port)
server.login(sender, password)
server.sendmail(sender, [receiver], msg.as_string())
server.quit()
```

首先我们创建了一个 MIMEText 对象 msg，第一个参数指定邮件内容，第二个参数指定 MIME 的子类型为 plain，第三个指定编码类型为 uft-8。

然后填充 msg 的主题、发件人、收件人，如果填充的主题包含中文，需要将其通过 Header 进行编码。

再后面，我们使用服务器地址和端口创建了一个 SMTP 对象 server，使用邮箱名称和授权码登陆，然后发送邮件，最后关闭。

一般情况下，通过 25 端口连接 SMTP 服务器时使用的是明文传输，内容很容易被劫取。如果服务器支持 SSL，推荐使用这种加密传输的方式。由于 163 的 SMTP 服务器是支持 SSL 的，这里我们对上面代码进行两个小改动就能更安全的发送邮件了。

一是端口

```
smtp_port = 25
```

改为

```
smtp_port = 465
```

二是创建 SMTP 对象的方式

```
server = smtplib.SMTP(smtp_server, smtp_port)
```

改为

```
server = smtplib.SMTP_SSL(smtp_server, smtp_port)
```

这样就搞定了。

# 3 发送 HTML 邮件

我们发送文本邮件创建 MIMEText 对象时，我们指定的 MIME 子类型为 plain。如果要发送 HTML 邮件，将 plain 改为 html 就行。就像这样

```
msg = MIMEText('<html><body><h1>你好！...</h1></body></html>',
               'html', 'utf-8')
```

# 4 带附件发送，比如 PDF

我们先看下 MIME 相关类的继承关系

```
Message
	|--MIMEBase
		|-- MIMENonMultipart
		    |-- MIMEApplication
		    |-- MIMEAudio
		    |-- MIMEImage
		    |-- MIMEMessage
		    |-- MIMEText
		|-- MIMEMultipart	
```

可以看出，一个邮件对象就是一个 Message 类的实例。其中，由 MIMEMultipart 创建的对象可以由多个部分组成，它可以使用 attach 方法将 MIMEText、MIMEApplication 等的实例包含进来；而 MIMENonMultipart 没有 attach 方法。所以，如果要能发送附件的话，我们需要创建一个 MIMEMultipart 对象。

邮件内容，MIMEText 能处理；音频附件，MIMEAudio 能处理；图片附件，MIMEImage 能处理。但是 PDF 怎么处理呢？我们不要忽略了这个类 MIMEApplication，它的默认类型是 application/octet-stream，邮件客户端遇到这样的文件一般会根据文件扩展名来猜测文件类型。

清楚了上面的思路以及相关问题的解决方法后，下面的代码就不难理解了

```
# coding=utf-8

import smtplib

from email.mime.multipart import MIMEMultipart
from email.header import Header
from email.mime.text import MIMEText
from email.mime.application import MIMEApplication
from os.path import basename

msg = MIMEMultipart()
msg['Subject'] = Header(subject, 'utf-8')
msg['From'] = sender
msg['To'] = receiver

text = MIMEText(conent, 'plain', 'utf-8')
msg.attach(text)

file_path = 'test.pdf'
part = MIMEApplication(open(file_path, 'rb').read())
part.add_header('Content-Disposition', 'attachment',
                filename=basename(file_path))
msg.attach(part)

server = smtplib.SMTP_SSL(smtp_server, smtp_port)
server.login(sender, password)
server.sendmail(sender, [receiver], msg.as_string())
server.quit()
```

需要注意的是，MIMEApplication 能办到的，MIMEBase 肯定也能实现。只是文件的 MIME 类型太多了，我们使用 MIMEBase 去自定义的话，要花费不少时间去搜索去尝试，过于麻烦，所以 MIMEApplication 一般会用的多一些。

# 5 参考

```
https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432005226355aadb8d4b2f3f42f6b1d6f2c5bd8d5263000
https://blog.csdn.net/handsomekang/article/details/9811355
https://docs.python.org/3.6/library/email.mime.html#email.mime.base.MIMEBase
```
