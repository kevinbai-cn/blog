---
title: 使用 Vue + Flask 搭建单页应用
date: 2018-12-14 12:06:46
categories:
- Web
tags:
- Single Page Application
- Vue
- Vuetify
- Flask
- RESTful API
- Flask-RESTful
---

单页应用，只加载一个主页面，然后通过 AJAX 无刷新加载其它页面片段。表面上看，就只有一个 HTML 文件，所谓单页。开发上，做到了前后端分离，前端专注于渲染模板，而后端只要提供 API 就行，不用自己去套模板了。效果上，页面和共用的 JS、CSS 文件都只加载一次，能减轻服务器压力和节省一定的网络带宽。另外，由于不需要每次都加载页面以及共用的静态文件，响应速度也有一定提高，用户体验比较好。当然，也有一些缺点，比如 SEO 优化不大方便，不过也有相应的解决方案。总的来说，使用单页应用的好处还是远多于坏处，这也是越来越多的人使用单页应用的原因。

构建单页应用的方式有很多，这里我们选择 Flask + Vue 实现。本文以实现一个 CRUD 的 Demo 为主线，在其中穿插必要的技术点进行讲述。里面可能涉及了一些你没接触或者不熟悉的概念，不过不要紧，我会给出相应的参考文章帮助你理解。当然，大牛可忽略这些 :)。看完这篇文章后，相信你也能搭建自己的单页应用了。

<!-- more -->

# 1 前端

这里我们会用到 Vue 框架。**如果你之前没有接触过，推荐去看下官方文档的「基础」一节。也可以先直接向下看，Demo 用的都是一些基础的东西，大致看下应该就能理解。即使暂时不理解，照着例子实践一遍后，去看下文档收获也应该更多。**

为了更便捷的创建基于 Vue 的项目，我们可以使用 Vue Cli 脚手架。通过脚手架创建项目的时候，它会辅助我们做一些配置，省去我们手动配置的时间。刚接触的伙伴前期会用它创建项目就行了，至于更深的一些东西后期再去了解。

安装脚手架

```
$ npm install -g @vue/cli
```

这里我们安装的是最新的 3 版本。

基于 Vue 的 UI 组件库很多，比如 iView、Element、Vuetify 等。国内使用 iView、Element 的特别多，而使用 Vuetify 的人相对要少很多，不知道是大家看不惯它的 Material Design 风格还是它的中文文档稀缺的缘故。不过我个人挺喜欢 Vuetify 的风格的，所以我会使用这个组件库搭建前端页面。

**如果你没使用过这个组件库，照着本文一步步实践下去，也能对 Vuetify 的用法有个大致的了解。如果这个过程中，感觉碰到的疑问太多，可以看下 YouTube 上的这个视频教程。**

`https://dwz.cn/lxMHF4bY`

**也不要到处去找类似的资源了，就是这个系列的视频看完再加上官方文档，掌握常用的点基本没问题。**

不过，还是建议先照着本文实现一下 Demo，再去学习，我觉得这样效果更好。

新建目录 spa-demo，然后切换到该目录下新建前端项目 client

```
$ vue create client
```

创建项目时会让你手动选择一些配置，这里贴下我当时的设置

```
? Please pick a preset: Manually select features
? Check the features needed for your project: Babel, Router, Linter
? Use history mode for router? (Requires proper server setup for index fallback in production) Yes
? Pick a linter / formatter config: Basic
? Pick additional lint features: (Press <space> to select, <a> to toggle all, <i> to invert selection)Lint on save
? Where do you prefer placing config for Babel, PostCSS, ESLint, etc.? In package.json
? Save this as a preset for future projects? (y/N) N
```

回车安装完成后，我们切换到 client 目录下，执行命令

```
$ npm run serve
```

上述命令执行完成后会有类似这样的输出

```
...

App running at:
- Local:   http://localhost:8080/
- Network: http://172.20.10.3:8080/

...
```

在浏览器中访问

`http://localhost:8080/`

如果看到包含下面文字的页面

`Welcome to Your Vue.js App`

说明项目安装成功。

安装 Vuetify

```
$ vue add vuetify
```

同样会提示你选择一些配置，这里我选择的 Default

```
? Choose a preset: Default (recommended)
```

回车安装完成后，重新开下服务器

```
$ npm run serve
```

执行完毕后，我们在浏览器中访问

`http://localhost:8080/`

会看到页面内容又些改变，有这么一行文字

`Welcome to Vuetify`

这里说明 Vuetify 安装成功。

看下此时的目录结构

```
spa-demo
└── client
    ├── README.md
    ├── babel.config.js
    ├── package-lock.json
    ├── package.json
    ├── node_module
    │   └── ...
    ├── public
    │   ├── favicon.ico
    │   └── index.html
    └── src
        ├── App.vue
        ├── assets
        │   ├── logo.png
        │   └── logo.svg
        ├── components
        │   └── HelloWorld.vue
        ├── main.js
        ├── plugins
        │   └── vuetify.js
        ├── router.js
        └── views
            ├── About.vue
            └── Home.vue
```

简化 `spa-demo/client/src/App.vue`，将其修改为

```
<template>
  <v-app>
    <v-content>
      <router-view></router-view>
    </v-content>
  </v-app>
</template>

<script>
  export default {
    name: 'App',
    data () {
      return {
        //
      }
    }
  }
</script>
```

修改 `spa-demo/client/src/views/Home.vue`，在页面放入一个 Data table

```
<template>
  <div class="home">

    <v-container class="my-5">

      <!-- 对话框 -->

      <!-- 表格 -->
      <v-data-table
        :headers="headers"
        :items="books"
        hide-actions
        class="elevation-1"
      >
        <template slot="items" slot-scope="props">
          <td>{{ props.item.name }}</td>
          <td>{{ props.item.category }}</td>
          <td class="layout px-0">
            <v-icon small class="ml-4" @click="editItem(props.item)">
              edit
            </v-icon>
            <v-icon small @click="deleteItem(props.item)">
              delete
            </v-icon>
          </td>
        </template>
        <template slot="no-data">
          <v-alert :value="true" color="info" outline>
            无数据
          </v-alert>
        </template>
      </v-data-table>
    </v-container>

  </div>
</template>

<script>
  export default {
    data: () => ({
      headers: [
        { text: '书名', value: 'name', sortable: false, align: 'left'},
        { text: '分类', value: 'category', sortable: false },
        { text: '操作', value: 'name', sortable: false }
      ],
      books: [],
    }),
    created () {
      this.books = [
        { name: '生死疲劳', category: '文学' },
        { name: '国家宝藏', category: '人文社科' },
        { name: '人类简史', category: '科技' },
      ]
    },
  }
</script>
```

我们使用了数据 headers 和 books 控制表的头部和数据，并在创建的时候，给 books 填充了一些临时数据。

**这个页面中涉及到了 Data table 的使用，相关代码不用记，在 Vuetify 文档中搜索 Data table 有很多例子，看了几个之后你就知道怎么使用了。对于新手来说，不好理解的可能就是那个 slot-scope（作用域插槽 ），这个看下 Vue 官方文档这些内容**

- **「基础」一节的「组件基础」**
- **「深入了解组件」一节的「组件注册」、「Prop」、「自定义事件」、「插槽」**

**静下心来读读就明白了，不难，这里我不再赘述。**

同样，这里你也可以先照葫芦画瓢，可以先暂时忽略掉一些不好理解的地方，待实践一遍之后再去搞清楚。

打开

`http://localhost:8080/`

看到的页面是这样的

{% fi 1.png,,, 60% %}

就是一个图书列表。

现在我们要做个可以弹出的对话框，用于新增书籍。我们在 `<!-- 对话框 -->` 位置新增如下代码

```
<v-toolbar flat class="white">
  <v-toolbar-title>图书列表</v-toolbar-title>
  <v-spacer></v-spacer>
  <v-dialog v-model="dialog" max-width="600px">
    <v-btn slot="activator" class="primary" dark>新增</v-btn>
    <v-card>
      <v-card-title>
        <span class="headline">{{ formTitle }}</span>
      </v-card-title>
      <v-card-text>
        <v-alert :value="Boolean(errMsg)" color="error" icon="warning" outline>
          {{ errMsg }}
        </v-alert>
        <v-container grid-list-md>
          <v-layout>
            <v-flex xs12 sm6 md4>
              <v-text-field label="书名" v-model="editedItem.name"></v-text-field>
            </v-flex>
            <v-flex xs12 sm6 md4>
              <v-text-field label="分类" v-model="editedItem.category"></v-text-field>
            </v-flex>
          </v-layout>
        </v-container>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="blue darken-1" flat @click="close">取消</v-btn>
        <v-btn color="blue darken-1" flat @click="save">保存</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</v-toolbar>
```

对应的，要在 `<script></script>` 之间添加一些 JS

```
export default {
  data: () => ({
    dialog: false, // 是否展示对话框
    errMsg: '', // 是否有错误信息
    editedIndex: -1, // 当前在对话框中编辑的图书在列表中的序号
    editedItem: { // 当前在对话框中编辑的图书内容
      id: 0,
      name: '',
      category: ''
    },
    defaultItem: { // 默认的图书内容，用于初始化新增对话框内容
      id: 0,
      name: '',
      category: ''
    }
  }),
  computed: {
    formTitle () {
      return this.editedIndex === -1 ? '新增' : '编辑'
    }
  },
  watch: {
    dialog (val) {
      if (!val) {
        this.close()
        this.clearErrMsg()
      }
    }
  },
  methods: {
    clearErrMsg () {
      this.errMsg = ''
    },
    close () {
      this.dialog = false
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      }, 300)
    }
  }
}
```

为了让文章简洁一些，贴代码的时候我将之前已有的片段进行了省略，你写的时候可以将上面的代码根据位置添加到合适的地方。

我们使用了 Toolbar、Dialog 在表格上面添加对话框相关的东西，同样，不必记代码，不知道怎么写的时候查阅下文档就行。

数据 dialog 表示当前对话框是否展示，errMsg 控制错误信息的展示，监听 dialog 当它变化为 false 的时候关闭对话框并清空 errMsg。计算属性 formTitle 用于控制对话框的标题。然后添加了两个表单元素用于填写书籍的名称以及分类。

当我们点击新增后，页面是这样的

{% fi 2.png,,, 60% %}

其实，到这里，我们的前端页面差不多就 OK 了，后面便是增删改的实现。这个我们先在前端单方面的实现下，后面再和后端进行整合。这样，会让前端的 Demo 更完整一些。

实现保存方法，在 methods 新增 save

```
save() {
  if (this.editedIndex > -1) { // 编辑
    Object.assign(this.books[this.editedIndex], this.editedItem)
  } else { // 新增
    this.books.push(this.editedItem)
  }
  this.close()
}
```

编辑的时候，要展示弹框，我们需要添加 editItem 方法

```
editItem (item) {
  this.editedIndex = this.books.indexOf(item)
  this.editedItem = Object.assign({}, item)
  this.dialog = true
}
```

保存方法和新增时的一致。

实现删除方法 deleteItem

```
deleteItem (item) {
  const index = this.books.indexOf(item)
  confirm('确认删除?') && this.books.splice(index, 1)
}
```

至此，前端项目告一段落。

# 2 后端

后端，我们只需要提供增删改查的接口供前端使用就行。RESTful API 是目前比较成熟的一套互联网应用程序设计理论，我也会基于此实现图书的相关操作接口。

**考虑到有对 RESTful API 不大熟悉的伙伴，我列了几个我之前学习的文章，供大家参考。**

- **《理解RESTful架构》**
	- `https://dwz.cn/eXu0p6pv`
- **《RESTful API 设计指南》**
	- `https://dwz.cn/8v4B0twY`
- **《RESTful API 最佳实践》**
	- `https://dwz.cn/2aSnI8fF`
- **知乎问题《怎样用通俗的语言解释REST，以及RESTful？》**
	- `https://dwz.cn/bVxrSsf4`

**看完上面的相关资料，你对这种设计理论应该就有一定掌握了。**

同样，你暂时可不必对 RESTful API 了解得很全面，暂时像下面这样理解它就行

> 就是用 URL 定位资源，用 HTTP 描述操作。

这个是在刷上面知乎问题看到的一个回答，作者是 @Ivony。写得很简洁，但确实有道理。

等到自己实践一次后，再回头看看理论的一些东西，印象更深。

首先列下我们需要实现的接口

| 序号 | 方法 | URL | 描述 |
| ------ | ------ | ------ | ------ |
| 1 | GET | http://domain/api/v1/books | 获取所有图书 |
| 2 | GET | http://domain/api/v1/books/123 | 获取主键为 123 的图书 |
| 3 | POST | http://domain/api/v1/books| 新增图书 |
| 4 | PUT | http://domain/api/v1/books/123 | 更新主键为 123 的图书 |
| 5 | DELETE | http://domain/api/v1/books/123 | 删除主键为 123 的图书 |

我们可以直接使用 Flask 实现上面的接口，不过当资源多的时候，我们写代码时会写很多重复的片段，违反了 DRY(Don't Repeat Yourself) 原则，后面维护起来比较麻烦，所以我们借助 Flask-RESTful 扩展实现。

另外，本节的重心是放在接口的实现上，也为了行文更简洁，我们将数据直接存在字典里，就不涉及数据库相关的操作了。

在 spa-demo 目录下新建 server 目录，并切换到该目录下，初始化 Python 环境

```
$ pipenv --python 3.6.0
```

Pipenv 是当前官方推荐的虚拟环境和包管理工具，我之前写过一篇文章《Pipenv 快速上手》介绍过，没接触过的可以去看下。

安装 Flask

```
$ pipenv install flask
```

安装 Flask-RESTful

```
$ pipenv install flask-restful
```

新建 `spa-demo/server/app.py`

```
# coding=utf-8

from flask import Flask, request
from flask_restful import Api, Resource, reqparse, abort


app = Flask(__name__)
api = Api(app)


books = [{'id': 1, 'name': 'book1', 'category': 'cat1'},
         {'id': 2, 'name': 'book2', 'category': 'cat2'},
         {'id': 3, 'name': 'book3', 'category': 'cat3'}]


# 公共方法区


class BookApi(Resource):
    def get(self, book_id):
        pass

    def put(self, book_id):
        pass

    def delete(self, book_id):
        pass


class BookListApi(Resource):
    def get(self):
        return books

    def post(self):
        pass


api.add_resource(BookApi, '/api/v1/books/<int:book_id>', endpoint='book')
api.add_resource(BookListApi, '/api/v1/books', endpoint='books')

if __name__ == '__main__':
    app.run(debug=True)
```

上面就是一个标准的整合了 Flask-RESTful 的代码结构，在 Flask-RESTful 的官方文档中可以看到相似的例子。对于每一种资源，我们都可以用类似的结构实现接口。BookApi 类中的 get、put、delete 方法对应接口 2、4、5，BookListApi 类中的 get、post 方法对应接口 1、3。之后便是注册路由。看到这，有的伙伴可能会有疑问，为什么同一个资源需要定义两个类呢？其实就是方便给一个资源注册带主键和不带主键的路由。

此时，项目结构为

```
spa-demo
├── client
│   └── ...
└── server
    ├── Pipfile
    ├── Pipfile.lock
    └── app.py
```

切换到 `spa-demo/server` 目录，运行 `app.py`

```
$ pipenv run python app.py
```

然后测试获取所有图书接口是否可用。由于是 API 测试，不建议直接使用浏览器，毕竟有时构造参数和看 HTTP 信息不大方便，推荐使用 Postman，当然简单的测试的话可以直接使用命令 curl。

请求接口 1，获取所有图书信息

```
$ curl -i http://127.0.0.1:5000/api/v1/books
```

得到结果

```
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 249
Server: Werkzeug/0.14.1 Python/3.6.0
Date: Thu, 13 Dec 2018 15:21:56 GMT

[
    {
        "id": 1,
        "name": "book1",
        "category": "cat1"
    },
    {
        "id": 2,
        "name": "book2",
        "category": "cat2"
    },
    {
        "id": 3,
        "name": "book3",
        "category": "cat3"
    }
]
```

成功获取所有图书，说明接口 1 已经 OK。

然后实现接口 2，获取指定 ID 的图书。由于根据 ID 获取图书以及图书不存在时抛 404 的操作后面会频繁使用到，所以这里提两个方法到「公共方法区」。

```
def get_by_id(book_id):
    book = [v for v in books if v['id'] == book_id]
    return book[0] if book else None


def get_or_abort(book_id):
    book = get_by_id(book_id)
    if not book:
        abort(404, message=f'Book {book_id} not found')
    return book
```

然后实现 BookApi 中 get 方法

```
def get(self, book_id):
    book = get_or_abort(book_id)
    return book
```

取 ID 为 1 的图书测试下

```
$ curl -i http://127.0.0.1:5000/api/v1/books/1
```

结果

```
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 61
Server: Werkzeug/0.14.1 Python/3.6.0
Date: Thu, 13 Dec 2018 15:31:48 GMT

{
    "id": 1,
    "name": "book1",
    "category": "cat1"
}
```

取 ID 为 5 的图书测试下

```
$ curl -i http://127.0.0.1:5000/api/v1/books/5
```

结果

```
HTTP/1.0 404 NOT FOUND
Content-Type: application/json
Content-Length: 149
Server: Werkzeug/0.14.1 Python/3.6.0
Date: Thu, 13 Dec 2018 15:32:47 GMT

{
    "message": "Book 5 not found. You have requested this URI [/api/v1/books/5] but did you mean /api/v1/books/<int:book_id> or /api/v1/books ?"
}
```

ID 为 1 时，成功获取到图书信息；ID 为 5 时，由于图书不存在，所以会返回 404 的响应。测试结果与预期一致，说明这个接口也 OK 了。

实现接口 3，新增图书。新增图书的时候，我们应该校验参数是否符合要求。Flask-RESTFul 给我们提供了比较优雅的实现，不需要我们使用多个 if 判断的硬编码的形式去检测参数是否有效。

由于图书名称和分类都是不能为空的，我们需要自定义规则，我们可以在「公共方法区」新增一个方法

```
def not_empty_str(s):
    s = str(s)
    if not s:
        raise ValueError("Must not be empty string")
    return s
```

重写 BookListApi 的初始化方法

```
def __init__(self):
    self.reqparse = reqparse.RequestParser()
    self.reqparse.add_argument('name', type=not_empty_str, required=True, location='json')
    self.reqparse.add_argument('category', type=not_empty_str, required=True, location='json')
    super(BookListApi, self).__init__()
```

然后实现 post 方法

```
def post(self):
    args = self.reqparse.parse_args()
    book = {
        'id': books[-1]['id'] + 1 if books else 1,
        'name': args['name'],
        'category': args['category'],
    }
    books.append(book)
    return book, 201
```

方法中，首先检测参数是否有效，然后取最后一本书的 ID 加上 1 作为新书的 ID 保存，最后返回添加的图书信息和状态码 201（表示已创建）。

测试下参数校验是否 OK

```
$ curl -i \
    -H "Content-Type: application/json" \
    -X POST \
    -d '{"name":"","category":""}' \
    http://127.0.0.1:5000/api/v1/books
```

结果

```
HTTP/1.0 400 BAD REQUEST
Content-Type: application/json
Content-Length: 70
Server: Werkzeug/0.14.1 Python/3.6.0
Date: Thu, 13 Dec 2018 15:46:18 GMT

{
    "message": {
        "name": "Must not be empty string"
    }
}
```

返回 400 的错误，说明参数校验有效。

看下新增接口是否可用

```
$ curl -i \
    -H "Content-Type: application/json" \
    -X POST \
    -d '{"name":"t_name","category":"t_cat"}' \
    http://127.0.0.1:5000/api/v1/books
```

结果

```
HTTP/1.0 201 CREATED
Content-Type: application/json
Content-Length: 63
Server: Werkzeug/0.14.1 Python/3.6.0
Date: Thu, 13 Dec 2018 15:53:54 GMT

{
    "id": 4,
    "name": "t_name",
    "category": "t_cat"
}
```

说明创建成功。我们通过获取指定 ID 的图书接口确认下

```
$ curl -i http://127.0.0.1:5000/api/v1/books/4
```

结果

```
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 63
Server: Werkzeug/0.14.1 Python/3.6.0
Date: Thu, 13 Dec 2018 15:54:18 GMT

{
    "id": 4,
    "name": "t_name",
    "category": "t_cat"
}
```

获取成功，说明确实创建成功，说明接口 3 也好了。

接口 4、5 的实现与上面类似，这里贴下代码，就不详细说明了。

和 BookListApi 类似，首先重写 BookApi 的初始化方法

```
def __init__(self):
    self.reqparse = reqparse.RequestParser()
    self.reqparse.add_argument('name', type=not_empty_str, required=True, location='json')
    self.reqparse.add_argument('category', type=not_empty_str, required=True, location='json')
    super(BookApi, self).__init__()
```

然后实现 put 和 delete 方法

```
def put(self, book_id):
    book = get_or_abort(book_id)
    args = self.reqparse.parse_args()
    for k, v in args.items():
        book[k] = v
    return book, 201

def delete(self, book_id):
    book = get_or_abort(book_id)
    del book
    return '', 204
```

至此，后端项目基本完毕。

当然，这是不完整的，比如这里面都没有实现对 API 的认证，这个可以通过 Flask-HTTPAuth 或者其它方式实现。限于篇幅，这里就不展开说明了，有兴趣的可以看下这个这个扩展的文档或者自己研究实现下。

# 3 整合

单独的前端或后端都有了雏形，就差整合这一步了。

前端需要请求数据，这里我们使用 axios，切换到 `spa-demo/client` 目录下进行安装

```
$ npm install axios --save
```

修改 `spa-demo/client/src/views/Home.vue`，在 `script` 标签之间引入 axios，并初始化 API 地址

```
import axios from 'axios'

const booksApi = 'http://localhost:5000/api/v1/books'

export default {
  ...
}
```

修改钩子 created 的逻辑，从后端获取数据

```
created () {
  axios.get(booksApi)
    .then(response => {
      this.books = response.data
    })
    .catch(error => {
      console.log(error)
    })
}
```

运行前端项目后，查看首页，会发现没有数据。查看开发者工具，我们会发现这么一个错误

```
Access to XMLHttpRequest at 'http://localhost:5000/api/v1/books' from origin 'http://localhost:8080' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

就是说当前项目不支持 CORS（Cross-Origin Resource Sharing，即跨域资源访问）。这个我们可以在前端添加代理的形式实现，也可以在后端通过 Flask-CORS 实现。这里，我使用的后者。

切换到 `spa-demo/server` 目录，安装 Flask-CORS

```
$ pipenv install flask-cors
```

修改 `spa-demo/server/app.py`，在头部引入 CORS

```
from flask_cors import CORS
```

在代码

```
app = Flask(__name__)
```

和

```
api = Api(app)
```

之间添加一行

```
CORS(app, resources={r"/api/*": {"origins": "*"}})
```

然后重新运行 app.py，刷新首页，我们会看到列表有数据了，说明 CORS 的问题成功解决。

在 `spa-demo/client/src/views/Home.vue` 中，修改 save 方法，同时新增辅助方法 setErrMsg

```
setErrMsg (errResponse) {
  let errResMsg = errResponse.data.message
  if (typeof errResMsg === 'string') {
    this.errMsg = errResMsg
  } else {
    let errMsgs = []
    let k
    for (k in errResMsg) {
      errMsgs.push('' + k + ' ' + errResMsg[k])
    }
    this.errMsg = errMsgs.join(',')
  }
},
save() {
  if (this.editedIndex > -1) { // 编辑
    axios.put(booksApi + '/' + this.editedItem.id, this.editedItem)
    .then(response => {
      Object.assign(this.books[this.editedIndex], response.data)
      this.close()
    }).catch(error => {
      this.setErrMsg(error.response)
      console.log(error)
    })
  } else { // 新增
    axios.post(booksApi, this.editedItem)
      .then(response => {
        this.books.push(response.data)
        this.close()
      }).catch(error => {
      this.setErrMsg(error.response)
      console.log(error)
    })
  }
}
```

此时，图书新增、保存搞定。

修改 deleteItem 方法

```
deleteItem (item) {
  const index = this.books.indexOf(item)
  confirm('确认删除?') && axios.delete(booksApi + '/' + this.books[0].id)
    .then(response => {
      this.books.splice(index, 1)
    }).catch(error => {
      this.setErrMsg(error.response)
      console.log(error)
    })
}
```

此时，删除方法也搞定了。

至此，整合完毕，基于 Vue + Flask 的前后端分离的一个 CRUD Demo 就完成了。

看完本文，你可以按着步骤自己实现下。刚接触的伙伴在看的过程中在某些地方可能有疑惑，我也在我能想到的地方提供了一些资料，你可以试着看下。如果没能提供全，你需要自己百度／谷歌下解决。**不过，我还是建议不要妄求每个点都了解的特别清楚，先明白关键点，试着实现一下，回头去看相关资料的时候，也更有感触一些。**

完整代码可到 GitHub 查看

`https://github.com/kevinbai-cn/spa-demo`

# 4 参考

- 《Full-stack single page application with Vue.js and Flask》
    - `https://bit.ly/2C9kSiG`
- 《Developing a Single Page App with Flask and Vue.js》
    - `https://bit.ly/2ElaXrB`
- 《Vuetify Documents》
    - `https://bit.ly/2QupMzx`
- 《Designing a RESTful API with Python and Flask》
    - `https://bit.ly/2vqq3Y1`
- 《Designing a RESTful API using Flask-RESTful》
    - `https://bit.ly/2nGDNtL`
