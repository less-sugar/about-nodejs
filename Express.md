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

