---
title: Python 网站应用从开发到部署
date: 2018-10-26 19:19:59
categories:
- Web
tags:
- Python
- Flask
- Git
- Pipenv
- Pyenv
- Gunicorn
- Nginx
---

本文大致说下一个网站从开发到部署的流程，里面会提及用到的或者可能会被用到的技术，这些东西在平时用到的也比较多，所以应该能给后端学习提供一些具体的参考。

<!-- more -->

# 1 网站开发

这里我建立一个简单的 Flask 应用，让它能处理简单的请求。

新建项目目录 flask_app

```
$ mkdir flask_app
```

切到项目目录

```
$ cd flask_app
```

使用 Pipenv 为项目初始化一个 Python 3.6 的环境

```
$ pipenv --python 3.6.0
```

如果报类似下面的错误

```
Warning: Python 3.6 was not found on your system...
```

可参考 3.3 小节的内容解决。

如果需要查看虚拟环境所在目录，可以使用下面的命令

```
$ pipenv --venv
```

安装 Flask

```
$ pipenv install flask==1.0.2
```

可能会出现类似下面的问题

```
TypeError: 'module' object is not callable
```

这是由于 pip 版本导致，安装这个版本就行了

```
$ pipenv run pip install pip==18.0
```

然后重新装下 Flask 就 OK 了。

新建文件 app.py

```
from flask import Flask
app = Flask(__name__)


@app.route('/', defaults={'query_path': 'index'})
@app.route('/<path:query_path>')
def match_path(query_path):
    return f'Query path: {query_path}'
```

到这里，应用就建立好了，我们运行测试一下。

为了方便，我们使用下面的命令进入虚拟环境

```
$ pipenv shell
```

如果后面需要退出该虚拟环境，可以这样

```
$ exit
```

然后

```
$ export FLASK_APP=app.py
$ flask run -h 0.0.0.0 -p 8080
```

最后在浏览器中访问 `http://127.0.0.1:8080/book/1`，看到如下内容

```
Query path: book/1
```

说明应用已经 OK。

**涉及技术：**

- Flask，基于 Werkzeug 和 Jinja 的微框架。
- Pipenv，整合了 virtualenv 和 pip，为虚拟环境和包管理提供了极大的方便，入门的话可以看下我之前的这篇文章《Pipenv 快速上手》。

**后续开发可能用到的**

- Requests，网络资源请求工具。
- SQLAlchemy，操作数据库的工具包。
- Celery，分布式任务队列。

# 2 代码管理

为了解决多人协作、版本回溯等问题，我们需要一个代码管理工具，这里我们使用 Git。

下面演示将本地代码保存到 GitHub，当然，如果是公司等私有项目的话，你可以使用 GitHub 付费服务或者自建托管服务。

首先在项目目录下初始化一个 Git 仓库

```
$ git init
```

添加忽略文件 `.gitignore`，将 IDE 配置文件忽略（根据你自己的情况进行配置）

```
.idea/
```

将当前代码提交到版本库

```
$ git add .
$ git commit -m "init"
```

然后登陆 GitHub 新建一个空的仓库 flask_app，复制其 SSH 地址，将其添加到本地客户端中

```
$ git remote add origin git@github.com:kevinbai-cn/flask_app.git
```

最后提交代码到仓库

```
$ git push origin master
```

如果其它项目成员需要使用项目或者要在服务器上进行部署，克隆一份代码即可

```
$ git clone git@github.com:kevinbai-cn/flask_app.git
```

**涉及的技术**

- Git，分布式版本控制软件。上面的例子只是让大家有个大致的印象，忽略了不少细节，比如提交代码到仓库时，可能会提示你没有权限，这时你就需要去进行 SSH Key 的配置。如果想将 Git 用得比较顺手的话，对版本库、分支、标签等概念要有一定理解，这不在本文的说明范围内，推荐大家去看下廖雪峰的 Git 教程，百度／谷歌相应关键字就能找到。

**后续开发可能用到的**

- Git 工作流程，其实就是对 Git 的使用做了一些规范建议，能让开发者更好的协同工作。目前有三种流程使用的比较多：Git flow、Github flow、Gitlab flow，有需要的可以看下阮一峰的 Git 工作流程相关的文章，同样，百度／谷歌相应关键字就能找到。

# 3 项目部署

这里我使用 Flask + Gunicorn（独角兽） + Nginx 的方式进行部署。

有的人可能会疑惑，Gunicorn 裸跑就能提供服务了，为什么还要加一层 Nginx 呢？我的考虑主要有负载均衡、静态文件缓存、IP 访问频率控制等，相对来说，Nginx 作为服务器支持得更全面一些。

**注意：当前服务器系统为新装的 Ubuntu 16.04，有些需要使用的软件会安装一次，所以记录得稍细一点**

## 3.1 Pipenv 安装

由于系统已经有了 pip，我们将 pipenv 安装至我们的用户目录

```
# pip install --user pipenv
```

查看用户基础目录

```
# python -m site --user-base
/root/.local
```
pipenv 安装后就在 `/root/.local/bin` 目录下，将其添加到 Path 中，打开 `/root/.bashrc`

```
# vim /root/.bashrc
```

在最后添加

```
export PATH=$PATH:/root/.local/bin
```

让配置生效

```
# source /root/.bashrc
```

测试一下 pipenv 是否能找到

```
# pipenv --version
pipenv, version 2018.7.1
```

OK，至此安装完成。

## 3.2 Git 安装与配置

使用 apt 进行安装

```
# apt update
# apt install git
```

Git 配置用户名与邮箱

```
# git config --global user.name "kevinbai"
# git config --global user.email "kevinbai.cn@gmail.com"
```

生成 SSH key，键入下面的命令，一路回车就行

```
# ssh-keygen -t rsa -C "kevinbai.cn@gmail.com"
```

执行时，你会看到 Key 保存的位置，一般是在 `~/.ssh` 目录下。

然后在 GitHub 上配置 SSH 公钥：settings -> SSH and GPG keys -> New SSH key，Title 可以随意填写，比如我填的 `server-ssh-rsa`，Key 填写 `.ssh/id_rsa.pub` 中的内容，之后保存就 OK 了。


## 3.3 克隆项目以及依赖包安装

克隆项目代码到服务器

```
# mkdir -p /data/htdocs
# cd /data/htdocs
# git clone git@github.com:kevinbai-cn/flask_app.git
```

依赖包安装

```
# cd flask_app
# pipenv install
```

发现报下面的错误

```
Warning: Python 3.6 was not found on your system...
```

我们需要先安装 Python 3.6，为了方便，我们使用 pyenv。

安装 pyenv

```
# curl -L https://raw.github.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```

打开 `~/.bashrc`

```
# vim ~/.bashrc
```

在最后添加

```
export PATH="/root/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

让配置生效

```
# source ~/.bashrc
```

安装 Python 3.6.0

```
# pyenv install 3.6.0
```

如果构建出错，一般是基础工具不全的问题，使用下面命令进行安装，然后再试一次就行了

```
# apt install make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev
```

重新安装依赖包

```
# pipenv install
```

## 3.4 Gunicorn 安装以及启动

这里我不打算将 gunicorn 写入 Pipfile，所以使用下面的命令进行安装

```
# pipenv run pip install gunicorn==19.9.0
```

而不用

```
# pipenv install gunicorn==19.9.0
```

测试一下，看能不能正常运行

```
# pipenv run gunicorn -w 5 -b 0.0.0.0:8080 app:app -D
```

回车后没有报错，说明启动成功。

## 3.5 Nginx 安装以及配置

安装 Nginx

```
# apt install nginx
```

备份默认配置

```
# cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak
```

打开配置文件

```
# vim /etc/nginx/sites-available/default
```

文件内容修改为

```
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_redirect     off;
        proxy_set_header   Host                 $http_host;
        proxy_set_header   X-Real-IP            $remote_addr;
        proxy_set_header   X-Forwarded-For      $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto    $scheme;
    }
}
```

启动 Nginx

```
# /etc/init.d/nginx start
```

如果需要关闭，使用下面的命令

```
# /etc/init.d/nginx stop
```

然后浏览器访问 `http://your_ip`，如果看到下面内容

```
Query path: index
```

则部署成功。

需要注意的是，如果是阿里云的 ECS 服务器，需要在控制台中去设置安全规则才能访问指定端口。

**涉及技术：**

- Linux 常用命令。不管是在服务器上进行操作还是在本地开发时，掌握一些常用的命令或工具是必要的，比如 chown、chmod、grep、find、ps、tar、vim 等。
- WSGI，为方便网站应用部署到服务器而定义的规范。我之前写了一篇相关的文章《说说我对 WSGI 的理解》，有兴趣的可以去看下。
- Gunicorn，一个 Python WSGI UNIX HTTP服务器。很多团队都在使用，最好熟悉一下常用的配置，如果能深入源码学习，那就更厉害了。
- Nginx，Web 服务器，可以用作负载均衡和 HTTP 缓存等。熟悉它的常用的一些配置是很有必要的。

**后续开发可能用到的**

- Docker，解决了开发和线上环境不统一以及虚拟机占用资源高的问题。现在基本上都在使用这种方式进行线上部署，对于一个厉害的开发者来说，这个也是需要掌握的。不过，前期的学习重心还是放在应用开发相关的库或者框架上较好，这一块先有一个大致的了解，有空再进一步学习。

# 4 参考

```
https://jiayi.space/post/flask-gunicorn-nginx-bu-shu
```
