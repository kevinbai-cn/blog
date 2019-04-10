---
title: 说说我对 WSGI 的理解
date: 2018-07-20 07:48:58
categories:
- Web 开发
tags:
- Python
- WSGI
---

先说下 WSGI 的表面意思，Web Server Gateway Interface 的缩写，即 Web 服务器网关接口。

之前不知道 WSGI 意思的伙伴，看了上面的解释后，我估计也还是不清楚，所以下面结合实际场景说明，先让大家有个大致的认识。最后我们再自己实现一个，加深对 WSGI 的理解。

我们现在使用 Python 编写 Web 应用，可以用比较流行的 Flask、Django 框架，也可以按自己的想法直接写一个。可选的服务器软件也特别多，比如常见的有 Apache、Nginx、IIS 等，除此外，也有很多小众的软件。但是，现在问题来了，我该怎么部署？在没有 WSGI 规范之前，一个服务器调度 Python 应用是用这种方式，另一款服务器使用的是那种方式，这样的话，编写出来的应用部署时只能选择局限的某个或某些服务器，达不到通用的效果。

**注意：下文中的代码基于 Python 3.6 编写。**

假如有这么一个服务器

**wsgi/server.py**

```
# coding=utf-8

import socket

listener = socket.socket()
listener.setsockopt(socket.SOL_SOCKET,
                    socket.SO_REUSEADDR, 1)
listener.bind(('0.0.0.0', 8080))
listener.listen(1)
print('Serving HTTP on 0.0.0.0 port 8080 ...')

while True:
    client_connection, client_address = \
        listener.accept()
    print(f'Server received connection'
          f' from {client_address}')
    request = client_connection.recv(1024)
    print(f'request we received: {request}')

    response = b"""
HTTP/1.1 200 OK

Hello, World!
"""
    client_connection.sendall(response)
    client_connection.close()
```

实现比较简单，就是监听 8080 端口，如果有请求在终端进行打印，并返回 Hello, World! 的响应。

终端中启动服务器

```
➜  wsgi python server.py
Serving HTTP on 0.0.0.0 port 8080 ...
```

再开一个终端，请求下

```
➜  ~ curl 127.0.0.1:8080
HTTP/1.1 200 OK

Hello, World!
```

说明服务器工作正常。

另外有一个 Web 应用

**wsgi/app.py**

```
# coding=utf-8


def simple_app():
    return b'Hello, World!\r\n'
```

现在要部署（也就是让这个整体跑起来），简单粗暴的做法就是在服务器里面直接调用 app 中相应的方法。就像这样

**wsgi/server2.py**

```
# coding=utf-8

import socket

listener = socket.socket()
listener.setsockopt(socket.SOL_SOCKET,
                    socket.SO_REUSEADDR, 1)
listener.bind(('0.0.0.0', 8080))
listener.listen(1)
print('Serving HTTP on 0.0.0.0 port 8080 ...')

while True:
    client_connection, client_address = \
        listener.accept()
    print(f'Server received connection'
          f' from {client_address}')
    request = client_connection.recv(1024)
    print(f'request we received: {request}')

    from app import simple_app
    response = 'HTTP/1.1 200 OK\r\n\r\n'
    response = response.encode('utf-8')
    response += simple_app()

    client_connection.sendall(response)
    client_connection.close()
```

运行脚本

**注意：因为使用端口相同的缘故，请先关闭上次的脚本，然后再执行，不然会由于端口冲突而报错。**

```
➜  wsgi python server2.py
Serving HTTP on 0.0.0.0 port 8080 ...
```

然后请求一下看看效果

```
➜  ~ curl 127.0.0.1:8080
Hello, World!
```

嗯，可以了。但是，上面的服务器和应用整体是跑起来了，那么我换一个服务器或者应用呢。由于服务器与应用之间怎么交互完全没有规范，比如服务器应该如何把请求信息传给应用，应用处理完毕后又怎么告诉服务器开始返回响应，如果都是各搞各的，服务器需要定制应用，应用也要定制服务器，这要一个应用能跑起来也太麻烦了点吧。

所以，WSGI 的出现就是为了解决上面的问题，它规定了服务器怎么把请求信息告诉给应用，应用怎么把执行情况回传给服务器，这样的话，服务器与应用都按一个标准办事，只要实现了这个标准，服务器与应用随意搭配就可以，灵活度大大提高。

<!-- more -->

WSGI 规范了些什么，下图能很直观的说明。

{% fi 1.jpg,,, 80% %}

首先，应用必须是一个可调用对象，可以是函数，也可以是实现了 `__call__()` 方法的对象。

每收到一个请求，服务器会通过 application_callable(environ, start_response) 调用应用。

应用在处理完毕准备返回数据的时候，先调用服务传给它的函数 start_response(status, headers, exec_info)，最后再返回可迭代对象作为数据。（不理解可迭代对象的伙伴可以看下我之前的一篇文章《搞清楚Python的迭代器、可迭代对象、生成器》）

其中，environ 必须是一个字典，包括了请求的相关信息，比如请求方式、请求路径等等，start_response 是应用处理完毕后，需要调用的函数，用于告诉服务设置响应的头部信息或错误处理等等。

status 必须是 `999 Message here` 这样的字符串，比如 `200 OK`、`404 Not Found` 等，headers 是一个由 (header_name, header_value) 这样的元祖组成的列表，最后一个 exec_info 是可选参数，一般在应用出现错误的时候会用到。

知道了 WSGI 的大致概念，下面我们来实现一个。

首先是应用

**wsgi/wsgi_app.py**

```
# coding=utf-8


def simple_app(environ, start_response):
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    return [f'Request {environ["REQUEST_METHOD"]}'
            f' {environ["PATH_INFO"]} has been'
            f' processed\r\n'.encode('utf-8')]
```

这里定义了一个函数（可调用对象），它可以使用服务器传给它的请求相关的内容 environ，这里使用了 REQUEST_METHOD 和 PATH_INFO 信息。在返回之前调用了 start_response，方便服务器设置一些头部信息。

然后是服务器

**wsgi/wsgi_server.py**

```
# coding=utf-8

import socket

listener = socket.socket()
listener.setsockopt(socket.SOL_SOCKET,
                    socket.SO_REUSEADDR, 1)
listener.bind(('0.0.0.0', 8080))
listener.listen(1)
print('Serving HTTP on 0.0.0.0 port 8080 ...')

while True:
    client_connection, client_address = \
        listener.accept()
    print(f'Server received connection'
          f' from {client_address}')
    request = client_connection.recv(1024)
    print(f'request we received: {request}')

    headers_set = None

    def start_response(status, headers):
        global headers_set
        headers_set = [status, headers]

    method, path, _ = request.split(b' ', 2)
    environ = {'REQUEST_METHOD': method.decode('utf-8'),
               'PATH_INFO': path.decode('utf-8')}
    from wsgi_app import simple_app
    app_result = simple_app(environ, start_response)

    response_status, response_headers = headers_set
    response = f'HTTP/1.1 {response_status}\r\n'
    for header in response_headers:
        response += f'{header[0]}: {header[1]}\r\n'
    response += '\r\n'
    response = response.encode('utf-8')
    for data in app_result:
        response += data

    client_connection.sendall(response)
    client_connection.close()
```

服务器监听相关代码没怎么变化，主要是处理请求的时候有些不同。

首先定义了 start_response(status, headers) 函数，自身并不会调用。

然后调用应用，将当前的请求信息 environ 和上面的 start_response 函数传给它，让其自己决定使用什么请求信息以及在处理完成准备返回数据之前调用 start_response 设置头部信息。

好了，启动服务器后（即执行服务器代码，和之前的类似，这里不赘述），然后请求看看结果

```
➜  ~ curl 127.0.0.1:8080/user/1
Request GET /user/1 has been processed
```

嗯，程序是正常的。

上面为了说明，代码耦合性较大，如果服务器需要更换应用的话，还得修改服务器代码，这显然是有问题的。现在原理差不多说清楚了，我们把代码优化下

**`wsgi/wsgi_server_oop.py`**

```
# coding=utf-8

import socket
import sys


class WSGIServer:
    def __init__(self):
        self.listener = socket.socket()
        self.listener.setsockopt(socket.SOL_SOCKET,
                                 socket.SO_REUSEADDR, 1)
        self.listener.bind(('0.0.0.0', 8080))
        self.listener.listen(1)
        print('Serving HTTP on 0.0.0.0'
              ' port 8080 ...')
        self.app = None
        self.headers_set = None

    def set_app(self, application):
        self.app = application

    def start_response(self, status, headers):
        self.headers_set = [status, headers]

    def serve_forever(self):
        while True:
            listener = self.listener
            client_connection, client_address = \
                listener.accept()
            print(f'Server received connection'
                  f' from {client_address}')
            request = client_connection.recv(1024)
            print(f'request we received: {request}')

            method, path, _ = request.split(b' ', 2)
            # 为简洁的说明问题，这里填充的内容有些随意
            # 如果有需要，可以自行完善
            environ = {
                'wsgi.version': (1, 0),
                'wsgi.url_scheme': 'http',
                'wsgi.input': request,
                'wsgi.errors': sys.stderr,
                'wsgi.multithread': False,
                'wsgi.multiprocess': False,
                'wsgi.run_once': False,
                'REQUEST_METHOD': method.decode('utf-8'),
                'PATH_INFO': path.decode('utf-8'),
                'SERVER_NAME': '127.0.0.1',
                'SERVER_PORT': '8080',
            }
            app_result = self.app(environ, self.start_response)

            response_status, response_headers = self.headers_set
            response = f'HTTP/1.1 {response_status}\r\n'
            for header in response_headers:
                response += f'{header[0]}: {header[1]}\r\n'
            response += '\r\n'
            response = response.encode('utf-8')
            for data in app_result:
                response += data

            client_connection.sendall(response)
            client_connection.close()


if __name__ == '__main__':
    if len(sys.argv) < 2:
        sys.exit('Argv Error')
    app_path = sys.argv[1]
    module, app = app_path.split(':')
    module = __import__(module)
    app = getattr(module, app)

    server = WSGIServer()
    server.set_app(app)
    server.serve_forever()
```

基本原理没变，只是使用了面向对象的方式修改了下原来的代码，同时 environ 添加了一些必要的环境信息。

可以使用以前的应用

```
➜  wsgi python wsgi_server_oop.py wsgi_app:simple_app
Serving HTTP on 0.0.0.0 port 8080 ...
```

请求

```
➜  ~ curl 127.0.0.1:8080/user/1
Request GET /user/1 has been processed
```

得到和之前相同的结果。

Flask 应用能行吗？来试一试，先新建一个

**wsgi/flask_app.py**

```
# coding=utf-8

from flask import Flask
from flask import Response

flask_app = Flask(__name__)


@flask_app.route('/user/<int:user_id>',
                 methods=['GET'])
def hello_world(user_id):
    return Response(
        f'Get /user/{user_id} has been'
        f' processed in flask app\r\n',
        mimetype='text/plain'
    )
```

重新启动服务器

```
➜  wsgi python wsgi_server_oop.py flask_app:flask_app
Serving HTTP on 0.0.0.0 port 8080 ...
```

请求

```
➜  ~ curl 127.0.0.1:8080/user/1
Get /user/1 has been processed in flask app
```

因为 Flask 也是遵守 WSGI 规范的，所以执行也没有问题。

至此，一个粗略的 WSGI 规范就实现了，虽说代码不优雅，一些核心的东西还是体现出来了。不过毕竟忽略了很多东西，比如错误处理等，要在生产环境中使用的话还远远不够，想知道得更全面的伙伴可以去看看 PEP 3333。

目前流行的 Web 应用框架比如 Django、Bottle 等，服务器 Apahce、Nginx、Gunicorn 等也都支持这个规范。因此，框架和应用随意搭配基本没什么问题。

**参考**

* https://www.python.org/dev/peps/pep-3333
* http://ruslanspivak.com/lsbaws-part2
* https://www.toptal.com/python/pythons-wsgi-server-application-interface
* https://gist.github.com/thomasballinger/5807241
