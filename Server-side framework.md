# Server-side framework

- 使用热门的 **Node Web** 框架
- 选择合适的框架
- 用 **Web** 框架搭建 **Web** 程序

## 用户画像

为了避免在每个项目上都用同一个框架。能够做到兼收并蓄、针对每个问题组使用合适的工具则更好。用用户画像考虑设计问题是通用的做法，因为这在某种程度上能让设计师跟用户产生共鸣。



## 框架是什么？

从专业角度来看，一些服务器端框架根本不是框架。关于框架这个词不同的程序员对它有不同的理解。在 Node 社区，大部分的项目都应该叫做`模块`。



## Koa

`Koa`是以 `Express` 为基础开发的，但它用 `ES2015` 中的生成器语法来定义中间件。也就是说几乎可以编写异步的中间件。在`Koa`  中可以使用 `yield` 退出和重载中间件。

> koa的主要特点

| 库类型 | HTTP 服务器库 |
| -------- | ---------------------------------------------- |
| 功能特性 | 基于生成器的中间件，请求/相应模型              |
| 建议应用 | 轻型 Web 程序、不严格的HTTP API、单页 Web 程序 |
| 插件架构 | 中间件                                         |
| 文档     | http://koajs.com                               |
| 授权许可 | MIT                                            |



#### 定义路由

Koa定义路由采用 `koa-router` 中间件组件。它也是基于 `HTTP` 模块的，这点和 `Express` 一样，不同的是，它是链式 API。

```js
route.post('/pages', function*(nuxt) {
    // 创建页面
})
.get('/pages/:id', function*() {
    // 渲染页面
})
.delete('/pages/:id', function*() {
	// 删除页面
})
.put('/pages-update', function*() {
    // 更新页面
})
```

####  

#### REST API

Koa没有提供实现 RESTful API 所必须的工具，只能借助某种路由处理中间件。



#### advantage

Koa的主要优势在于它的精简，还有一些很奈斯的第三方模块。因为语法优雅，能根据项目的具体需求量身定制，所以Koa深受开发人员喜爱。



#### disadvantage

Koa的可配置水平让一些开发人员望而却步。除非有现成的代码共享策略，否则用Koa创建太多小项目会导致低下的代码复用率。



## Kraken

`Kraken` 是基于 `Express` 的，又通过 `Paypal` 开发的一些定制模块添了些新功能。为程序提供安全层的 `Lusca` 是其中特别使用的一个模块。虽然 `Lusca` 可以独立于 `Kraken` 使用，但 `Kraken` 还有一个优势是它预定义的项目结构。

> Kraken 的主要特性

| 库类型 | HTTP 服务器库 |
| -------- | ---------------------------------------------- |
| 功能特性 | 对象项目结构要求严格、模型、模板（Dust）、安全强化（Lusca）、配置管理、国际化 |
| 建议应用 | 企业 Web程序 |
| 插件架构 | Express 中间件                              |
| 文档     | http://www.kraken.com/help/api    |
| 授权许可 | Apache 2.0                                  |

⭐ kraken可以作为 Express 的中间件使用。



#### 定义路由

`Kraken` 中路由被定义为跟 **控制器** 在一起。这和 Express 路由定义和路由处理器分开的做法不同，kraken 采用了 **MVC** 的方式。

`Kraken` 的路由 API 是 `EXpress-enrouten` ，并且它会根据文件所在目录推断路由。

例：

​	controllers

​		|- user

​			|- create.js

​			|- list.js

那么 Kraken 会生路由 /user/create 和 /user/list



#### REST API

​		Kraken 可以做 REST API，但没有社么特别的支持。express-enrouten 可以跟解析 JSON 的中间件相结合，所以能实现 REST API。

​		Kraken 的路由器支持 **DELETE**，**GET**，**POST**，**PUT** 等 HTTP 动词，在实现 REST 时跟 Eexpress 类似。



#### advantage

​	Kraken 项目的从大体上差不多，Kraken 项目一般不会改变文件和目录的位置。

​	模板库（Dust）和国际化库（Makara）都是 Kraken 自带的，所以可以进行无缝集成。



#### disadvantage

相对而言学习难度较大。一些在 `Express` 中可以通过编程实现的任务，在 kraken 中要通过 JSON 配置文件来做，而且有时候很难确定到底要哪些 JSON 属性才能得到预期的结果。



## hapi

hapi 是一个服务器框架，重点在 **Web API** 的开发。hapi 有自己的插件API，完全没有客户端支持，也没有数据模型层。

> hapi 的主要特性

| 库类型   | HTTP 服务器库                      |
| -------- | ---------------------------------- |
| 功能特性 | 高层服务器容器抽象，安全的头部信息 |
| 建议应用 | 单页 Web 程序、HTTP API            |
| 插件架构 | hapi 插件                          |
| 文档     | http://hapijs.com/api              |
| 授权许可 | BSD 3条款                          |



#### 定义路由

​		hapi 有创建路由的 API 。要创建路由必须提供一个包含请求方法，URL，和回调函数的对象，其实回调函数就是 **路由处理器**。

​		例：

```js
const Hapi = require('hapi')
const server = new Hapi.Server()

server.connection({
    host: 'localhost',
    port: 8888
})

server.route({
    method: 'GET',
    path: '/hello',
    handler: (request, reply) => {
        return reply('hello world')
    }
})

server.start((err) => {
    if (err) throw err
    console.log('Server running at:', server.info.uri)
})
```



#### 插件

​		hapi 有自己的插件架构，并且大部分项目都需要靠插件完成认证和输入校验等功能。

​		想要把一个插件添加到hapi 项目中，需要先用 server.register 方法注册这个插件。然后根据实际业务情景去配置插件的使用。



#### REST API

​		根据路由的定义可知，如果想实现 REST API ,只需要按照定义路由的方式，给同一个请求地址配置不同的请求参数，从而处理不同的业务。



#### advantage

​		hapi 的插件 API 是它最大的优势。另外，由于 hapi 是基于HTTP服务器的，所以适合在某些部署场景中。如果要部署很多相互连接的服务器，或者需要做负载均衡时，hapi 基于服务器的 API 可能比 Express 或 Koa 好用。



#### disadvantage

​		hapi 的劣势和 Express 相同：极简，所以对项目结构没有把控。过度依赖插件可能会造成将来难以维护。



## Sails.js

​		`Sails` 是一个 模型-视图-控制器框架。Sails 不是全栈框架，所以可以和任何前端库或者框架配合使用。

> Sails 的主要特性

| 库类型   | MVC 框架                                    |
| -------- | ------------------------------------------- |
| 功能特性 | 有支持数据库的 ORM，生成REST API，websocket |
| 建议应用 | Rails风格的MVC程序                          |
| 插件架构 | express 中间件                              |
| 文档     | http://sailsjs.org/documentation/concepts   |
| 授权许可 | BSD 3条款                                   |

​		sails 有项目生成器，可以是用 sails 的生成器创建新项目会比较轻松。



#### 定义路由

​		sails 中将路由称为 **定制路由**，打开 config/route.js，在输出的路由中添加新的属性极客添加路由。属性的格式是HTTP动词加上部分 URL。



#### REST API

​		Sails 将数据库模型和控制器结合进了API中，可以用命令 `sails generate api resource-name` 生成REST API。要使用数据库，首先需要安装 **数据库适配器** 。找到 **Waterline MySQL** 包的名字，然后把它添加到项目中：

​		npm install --save waterline sails-mysql

​		接下来，打开 config/connection.js，将 MySQL 服务器的连接信息填好。Salis 模型文件中可以指定数据库连接，所以不用的模型可以使用不同的数据库。也就是说可以把用户会话数据放在 Radis 之类的数据库中，而把需要持久保存的数据放到 MySQL 这样的关系型数据库中。

​		Waterline 是 Salis 的数据库系统库，除了支持多个数据库，它还能定义表和列名，以支持一流的数据库模型。另外，它的查询 API 支持 promise。

  

#### advantage

​		可以快速设置项目，快速添加典型的 REST API。

​		因为 Salis 项目的文件系统都是一样的，所以有利于创建新项目和相互合作。



#### disadvantage

​		与其他MVC的框架一样：路由 API 意味着我们在设计程序时必须考虑到 Sails 的路由特性，并且由于 Waterline 的处理方式，可能很难将数据库模式调整为符合他的要求的样子。



## DerbyJS

​		DerbyJS 是全栈框架，支持数据同步和视图的服务器端渲染。它用到了 MongoDB 和 Redis，数据同步层是由 shareJS 提供的，支持冲突的自动解析。
> DerbyJS 的主要特性

| 库类型   | 全栈框架                          |
| -------- | --------------------------------- |
| 功能特性 | 有支持数据库的 ORM（Racer），同构 |
| 建议应用 | 由服务器端支持的单页 Web 程序     |
| 插件架构 | DerbyJS 插件                      |
| 文档     | http://derbyjs.com/docs/derby-0.6 |
| 授权协议 | MIT                               |
⭐：运行 DerbyJS 的例子需要安装 MongoDB 和 Redis。



#### 定义路由

​		DerbyJS 中的路由是用 derby-router 实现的。因为是基于 Express 的，所以 DerbyJS 的路由 API 跟服务器端路由类似，浏览器中用的也是这个路由模块。

​		因为 DerbyJS 是全栈框架，所以它添加路由的方式跟其他框架不太一样。对于基本的路由而言，最理想的添加方式是添加一个视图。



#### REST API

​		在 DerbyJS 创建 RESTful API 需要用 Express 添加路由和路由处理器。DerbyJS 项目中有个 server.js 文件，可以用 Express 创建服务器。

​		在服务器路由文件中，可以用 app.use 装载另外一个 EXpress 程序，所以可以将 REST API 作为一个完全独立的 Express 程序，然后作为主程序的DerbyJs 程序装载它。



#### advantage

​		DerbyJs 有数据库模型 API 和数据同步 API。你可以很方便的搭建单页 Web 程序和现代化的实时程序。因为它自对 WebSocket 和同步的支持，所以不同我们费心去选择 WebSocket 库，或者任何在服务器和客户端之间同步数据。



#### disadvantage

​		有服务器端或客户端相关经验的人不喜欢使用 DerbyJs。



## Flatiron.js

`Flatiron` 是一个 web 框架，有 URL 路由、数据管理、中间件、插件和日志功能。 Flatiron的模块在设计是就考虑了`耦合性`，所以可以分开使用。Flatiron 的模块不是用 Express 或 COnnect 写的，但是它的中间件可以和 Connect 兼容。

> Flatiron.jS 的主要特性

| 库类型   | 模块化MVC框架                                 |
| -------- | --------------------------------------------- |
| 功能特性 | 数据库管理层（Resourceful），解耦的可重用模块 |
| 建议应用 | 轻量的MVC程序，在其他框架中使用 Flatiron 模块 |
| 插件架构 | Broadway插件 API                              |
| 文档     | https://github.com/flatiron                   |
| 授权协议 | MIT                                           |

Flatiron 拥有自己的脚手架工具。安装Flatiron。

```shell
npm install -g flatiron
```

使用脚手架创建项目

```shel
flatiron create flatiron-app
```

⭐：通过 `this.res`  返回相应。



#### 定义路由

​		Flatiron 的路由库叫 `Dirctor`。它既能用于服务器端路由，也支持浏览器中的路由，所以可以用来制作单页程序。Director 使用 Express 风格的 HTTP 动词路由。相应数据时，可以用 `res.writeHead` 发送相应头部，用`res.end` 发送相应的主体部分。

```js
router.get('/', () => {
    this.res.writeHead(200, { 'content-type': 'text/plain' })
    this.res.end('Hello world')
})
```



​		亦可以定义一个路由表对象，把路由 API 当作类来用。这种方法需要 `初始化` 一个新的路由器，然后用 `dispatch` 方法来处理HTTP请求：

```js
const htpp = require('http')
const director = require('director')

const router = new director.http.Router({
    '/example': {
        get: () => {
            this.res.writeHeade(200, { 'content-type': 'text/plan' })
        }
    }
})

const server = http.createServer((req, res) => {
    router.dispatch(req, res)
})
```



#### REST API

​		在 Flatiron 中，可以用 EXpress 风格的标准 HTTP 动词方法创建 REST API，或者用 Director 的作用域路由功能。这个功能可以基于 URL 的组成和 URL 的参数对路由分组。

```js
router.path(/\/users\/(\w+)/, () => {
    this.get((id) => {});
    this.post((id) => {});
    this.delete(id => {});
    this.put(id => {})
})
```



#### advantage

​		Flatiron 的优势也是显而易见的，Flatiron 的解耦式设计就是它最大的优势所在。Flatiron 拥有自己的插件管理器，使用社区中的插件更加容易。



#### disadvantage

​		在大型的 MVC 项目中，Flatiron 不想其他框架那么好用。



## LoopBack

​		`LoopBack` 是一个 API 框架，但它的特性很适合跟数据库配合，也很适合跟 MVC 程序配合。甚至还有一个浏览和管理 REST API 的 web 界面。如果是要给移动端和桌面端程序创建 Web API 的框架，那就是`LoopBack`。

> LookBack 的主要特性

| 库类型   | API框架                                           |
| -------- | ------------------------------------------------- |
| 功能特性 | ORM、API用户界面、WebSocket、客户端SDK（包括IOS） |
| 建议应用 | 支持多客户端的API（移动端、桌面端、Web）          |
| 插件架构 | Express中间件                                     |
| 文档     | http://loopback.io/doc                            |
| 授权协议 | MIT                                               |

​		⭐：创建 LoopBack 项目需要 LoopBack 的命令行工具。



#### 定义路由

​		LoopBack 中路由可以在 Express 这个层面添加。创建 server/boot/routes.js，通过 LoopBack 路由实例添加一个新路由。

```js
module.exports = app => {
    const router = app.loopback.Router()
    router.get('/hello', (req, res) => {
        res.send('hello world')
    })
    app.use(router)
}
```



#### REST API

​		使用模型生成器创建 REST API 是最方便的。在你全局安装了 LoopBack 之后，可以使用 slc 命令。比如说，如果添加名为 project的新模型，则运行

```shell
$ slc loopback:model project
```

​		slc 命令会带着你一步一步创建，让你选择这个模型是否只用在服务器端，并设置一些属性和校验器。创建完成之后，可以查看对应的 JSON 文件。用这样的 JSON 文件来定义模型的行为更轻便，其中包括了你之前指定的所有属性。



#### advantage

- LoopBack 减少了堆砌重复代码的操作。
- 方面，一个命令行工具既可生成一个完整的 RESTful Web API 程序。
- LoopBack 对前端代码没有太多的限制。
- 可以与移动端相通信。集成 LoopBack 的SDK 就可以给 IOS 以及 Android 推送消息。



#### disadvantage

​		LoopBack 基于 JSON 的模型API 和大部分 JavaScript 数据库 API 都不同，有一定的学习成本。由于 HTTP 层是基于 Express 的，所以需要方面都限制与 Express。



## 如何选择

既然有这么多的 Node 框架，那么应该如何进行选择呢

![](D:\note-space\images\choose_Node_server-side-framework.png)

 

## 总结

- Koa 轻便、极简，在中间件中使用 ES2015生成器语法。适合依赖外部 Web API 的单页 Web 程序。
- hapi 的重点是 HTTP 服务和路由。适合由很多小服务器组成的轻便后台。
- Flatiron 是一组解耦的模块，既可以当作 Web MVC 框架来用，也可以当作更轻便的 Express 库。Flatiron 跟 Connect 中间件是兼容的。
- Kraken 是基于 Express 的，添加了安全特性，可以用于 MVC。
- Sails.js 是Rails / Django 风格的 MVC 框架。由 ORM 和模板系统。
- DerbyJs 是个同构框架，适合实时的程序。
- LoopBack 帮我们省去了写套路化代码的工作。它可以快速生成带有数据库支持的 REST API，并且有个 API 管理页面。