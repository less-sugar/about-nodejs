## Express

使用 Express-generator 制作在线留言板。借用这个项目，了解 Express。

#### 安装

```shell
npm install express-generator -g
```

#### 使用 Express-generator 创建项目

​		使用 `express --help` 指令查看帮助信息得知，使用 `express -e <project name>` 创建项目。

#### 查看代码

ps: 以下代码部分来自 express-generator 的生成。

```js
var createError = require('http-errors');
var express = require('express');
var path = require('path');
var cookieParser = require('cookie-parser');
var logger = require('morgan');
var indexRouter = require('./routes/index');
var usersRouter = require('./routes/users');
var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');   // <== 设置默认的模板类型

app.use(logger('dev'));			// <== 输出有颜色区分的日志，以便于开发调试
app.use(express.json());		// <== 解析请求体
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));		//  <== 提供./public 下的静态文件

app.use('/', indexRouter);		// <== 指定程序路由
app.use('/users', usersRouter);

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  next(createError(404));
});


// error handler
app.use(function(err, req, res, next) {   // <== 设置错误处理，并且针对不同的模式设置不同的效果，非开发环境的情况下，不显示错误的具体信息
  // set locals, only providing error in development
  res.locals.message = err.message;

  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});

module.exports = app;
```



#### 整理需求

1. 用户应该给可以注册、登录、退出。
2. 用户应该可以发消息（条目）。
3. 站点的访问者应该可以分页浏览条目。
4. 应该有个支持认证的简单的 REST API。

针对这些需求，我们要存储数据和处理用户认证。还需要对用户的输入进行校验。必要的路由需要应该有以下两种

⭐ API 路由

- GET	/api/entries：获取条目列表。
- GET    / api/entries/page：获取单页条目。
- POST    /api/entry：创建新的留言条目。

⭐ Web UI 路由

- GET    /post：显示创建新条目的表单。
- POST    /post：提交新条目。
- GET    /register：显示注册表单。
- POST    /register：创建新的用户表单。
- GET    /login：显示登录表单。
- POST    /login：登录。
- GET    /logout：登出。



#### 渲染视图

​		视图渲染的过程大概可以表述为：把数据传递给视图，然后视图对数据进行转换，对 Web 程序来说，通常是转换成 HTML。

​		Express 有两种渲染视图的方法：程序层用：app.render()，在请求或者响应层用 res.render()，Express 内部是前一种。在 `./routers/index.js`  中会调用 `res.render('index')` 的函数，渲染的是 `./views/index.ejs` 模板，代码如下

```js
router.get('/', (req, res, next) => {
    res.render('index', {title: 'Express'}), // 渲染 index.ejs 文件，并传递数据处理。
})
```

​	

###### 配置视图系统

 		Express 视图系统的配置很简单。虽然 generator 已经生成好了，但还是应该了解一下底层的配置机制，以便在需要时进行修改。着重​介绍三个部分：

​		:ballot_box_with_check:：调整视图的查找；

​		:ballot_box_with_check:：配置默认的模板引擎；

​		:ballot_box_with_check:：启用视图缓存，减少文件 I/O。



**改变查找目录**

​		下面的代码片段是 Express 的可执行程序创建的 views 设定的：

​		`app.set('views', path.join(__dirname, 'views'));`	

这个配置项指明了 Express 查找视图的目录。



**使用默认的模板引擎**

​		用 Express 生成程序时，我们在命令行用 -e 指定模板引擎 ejs，所以 view engine 被设定成 ejs。Express 要靠扩展名确定用哪个模板引擎渲染文件，但有个这个配置项，我们可以省略文件名后缀。

​		Express 同样支持多引擎渲染，如下列代码所示：

```js
app.set('view engine', 'pug')
app.get('/', function(req, res) {
    res.render('index')
})
app.get('/ejs', function(req, res) {
    res.render('rss.ejs')
})
```



###### **视图缓存**

​		在成产环境中，view cache 是默认开启的，以防后续的 `render()` 从硬盘中读取模板文件。view cache 被禁用时，每 请求都会从硬盘上读取模板，启用了 view cache 后，模板只需要读取一次硬盘。

![](https://raw.githubusercontent.com/less-sugar/about-nodejs/master/images/exprees-view_cache.png)

