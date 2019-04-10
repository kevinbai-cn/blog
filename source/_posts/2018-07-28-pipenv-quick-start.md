---
title: Pipenv 快速上手
date: 2018-07-28 23:22:23
categories:
- Python
tags:
- Python
- Pipenv
---

做过 Python Web 项目的伙伴可能都有体会，每次新建一个项目，自己得手动建一个虚拟环境，把包装好了之后，还得自己把装的包导入到文件中，方便部署。时间久了之后，感觉重复劳动太多，应该改变一下了，这不就找到 Pipenv 了。

Pipenv 是 Kenneth Reitz 开发的又一个 for Humans 项目，于 2017 年 1 月份创建，仅仅用了一年左右的时间便成了官方推荐工具。Kenneth Reitz，作为一个 Python 开发者应该都知道吧，不知道可以去面壁了，哈哈。就算你不知道他，你也一定用过他的一个库 requests/requests，它写的库基本上都有 for Humans 标签，用起来，也确实像标签说的那样，简单好用。

Pipenv 个人觉得主要解决了下面几个麻烦事：

（1）整合了 pip 和 virtualenv，现在不必将这两个工具分开使用了

（2）不用自己新建 virtualenv 了

（3）不用自己导出包依赖到 requirements.txt 了


# 1 安装

如果是 Mac 的话，直接

```
brew install pipenv
```

就搞定了。如果没有 Python 它会自动给你安装好。

其它平台的话，首先确保你有安装 Python，才能安装 Pipenv。如果没装的话，可以自己去百度／谷歌／官网寻找安装方法。

如果你之前装过 pip，可以使用如下命令将 Pipenv 装到你的用户目录

```
pip install --user pipenv
```

如果之前没装过 pip 的话，可以使用下面的命令，简单粗暴

```
curl https://raw.githubusercontent.com/kennethreitz/pipenv/master/get-pipenv.py | python
```

然后看下是否安装成功，终端执行

```
➜  ~ pipenv --version
pipenv, version 2018.7.1
```

看来已经 OK，如果你的提示命令不存在之类的问题，说明没有安装成功，可以去看看官网的安装文档。


好了，下面我们来说说 Pipenv 的简单使用。便于说明，我们打算新建一个基于 Flask 的 hello_world 应用并运行起来。

<!-- more -->

# 2 开始

Pipenv 的虚拟环境是基于项目的，一个项目便会创建一个虚拟环境，不过没问题，我们之前开发也是这样的啊。

创建项目文件夹

```
➜  ~ mkdir hello_world
```

好，我们现在准备创建一个 Python 3.6.0 的虚拟环境

```
➜  ~ cd hello_world
➜  hello_world pipenv --python 3.6.0
```

等待一小会儿就安装成功了。

看下虚拟环境在什么目录

```
➜  hello_world pipenv --venv
/Users/kevinbai/.local/share/virtualenvs/hello_world-qymNYcuh
```

看下当前目录多了什么文件

```
➜  hello_world ls
Pipfile
```

看下它的内容

```
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]

[dev-packages]

[requires]
python_version = "3.6"
```

其中，[[source]] 小节记录安装源信息，[packages] 记录依赖包信息，[dev-packages] 记录开发依赖包信息，[requires] 记录依赖的环境信息，这里要求 Python 版本必须等于 3.6。

好了，我们安装一个包试试

```
➜  hello_world pipenv install requests
```

执行成功后，Pipfile 会在 [packages] 小节下会添加一行 requests 信息

```
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
requests = "*"

[dev-packages]

[requires]
python_version = "3.6"
```

同时生成一个 Pipfile.lock 文件，该文件记录了安装包的具体版本。线上部署时执行命令

```
➜  hello pipenv install
```

会自动给你生成相应的虚拟环境并安装 Pipfile.lock 中的包，线上环境和开发环境的包的版本可以达到完全一致。

如果没有 Pipfile.lock，会根据 Pipfile 生成虚拟环境（如果没有的话）并安装相应的包。

如果两个文件都没有的话，会生成一个默认的虚拟环境与两个带默认信息的 Pipfile 和 Pipfile.lock 文件。

这里我们删除 requests 包

```
➜  hello_world pipenv uninstall requests
```

安装 Flask

```
➜  hello_world pipenv install flask
```

安装好后，我们可以看下包的依赖信息

```
➜  hello_world pipenv graph
certifi==2018.4.16
chardet==3.0.4
Flask==1.0.2
  - click [required: >=5.1, installed: 6.7]
  - itsdangerous [required: >=0.24, installed: 0.24]
  - Jinja2 [required: >=2.10, installed: 2.10]
    - MarkupSafe [required: >=0.23, installed: 1.0]
  - Werkzeug [required: >=0.14, installed: 0.14.1]
idna==2.7
urllib3==1.23
```

# 3 编写代码并执行应用

新建文件 hello_world/main.py

```
# coding=utf-8

from flask import Flask

app = Flask(__name__)


@app.route('/', methods=['GET'])
def index():
    return 'Hello, World!'


if __name__ == '__main__':
    app.run()
```

终端激活虚拟环境

```
➜  hello_world pipenv shell
```

然后运行应用

```
(hello_world-qymNYcuh) ➜  hello_world python main.py
 * Serving Flask app "main" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

访问 `http://127.0.0.1:5000`，看到 Hello, World! 信息，搞定。

终端如果要退出虚拟环境，执行如下命令即可。

```
(hello_world-qymNYcuh) ➜  hello_world exit
```

# 4 其它常用命令或选项

* pipenv run 执行当前项目对应的虚拟环境中的已安装的脚本

如果需要调用虚拟环境中的 pip 命令，除了使用 pipenv shell 进入环境执行以外，也可以使用 pipenv run。比如

```
➜  hello_world pipenv run pip freeze
certifi==2018.4.16
chardet==3.0.4
click==6.7
Flask==1.0.2
idna==2.7
itsdangerous==0.24
Jinja2==2.10
MarkupSafe==1.0
urllib3==1.23
Werkzeug==0.14.1
```

* --where 当前项目路径

```
➜  hello_world pipenv --where
/Users/kevinbai/hello_world
```

* --py 当前项目的虚拟环境的解释器所在路径

```
➜  hello_world pipenv --py
/Users/kevinbai/.local/share/virtualenvs/hello_world-qymNYcuh/bin/python
```

* --rm 删除当前项目的虚拟环境目录

```
➜  hello_world pipenv --rm
Removing virtualenv (/Users/kevinbai/.local/share/virtualenvs/hello_world-qymNYcuh)...
```

* -h 帮助，查看可用的命令以及选项介绍
