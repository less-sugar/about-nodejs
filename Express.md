## Express

使用 Express-generator 制作在留言板。借用这个项目，了解 Express。

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



###### 视图查找

​		查找视图的过程跟 `require()` 查找模块的过程差不多。在程序中调用了 res.render() 或者 app.render() 后， express 会先检查有没有这样的绝对路径，接着找到视图目录的相对路径。最后会尝试找目录中的 index  文件。如果还是为找到就会抛出异常。



###### 将数据传递给视图的方法

​		在 Express 中，要给被渲染的驶入传递数据有几种办法，其中最常用的是将要传递的数据作为 res.render() 的参数。此外，还可以在路由处理器之前的中间件设定一些变量，比如用 app.locals 传递程序层面的数据，用 res.locals 传递请求层面的数据。

​		默认情况下，Express 只会向视图中传递一个程序级变量 —— settings，这个对象中包含所有用 app.set() 设定的值。



#### Express 路由入门

- 用特定的路由中间件校验用户提交的内容；
- 实现特定路由的校验。·

###### 前期分析：

1. 创建数据库连接（此时方便使用，选用redis）
2. 需要一个用于提交的表单页面
3. 创建消息的动作对应的操作
4. 访问全部消息的列表页



###### 集体实现：

**step1：创建redis连接**

​		准备工作：

​			1、新建文件 models/entry.js

​			2、安装 node-redis `yarn add redis --save`

​		书写代码：

​			在 models/entry.js 中写入下列代码

```js
const redis = require('redis')
const db = redis.createClient()

class Entry {
  constructor(obj) {
    for (const key in obj) {
      this[key] = obj[key]
    }
  }

  save(cb) {
    const entryJSON = JSON.stringify(this)
    db.lpush(
      'entries',
      entryJSON,
      err => {
        if (err) return cb(err)
        cb()
      }
    )
  }

  static getRange(from, to, cb) {
    db.lrange('entries', from, to ,(err, items) => {
      if (err) return cb(err)
      let entries = []
      items.forEach(item => {
        entries.push(JSON.parse(item))
      })
      cb(null, entries)
    })
  }

}

module.exports = Entry
```



**step2：创建表单页面**

​		新建文件 views/post.ejs，创建 post 视图。并写入以下代码

```ejs
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title><%= title %></title>
  <link rel="stylesheet" href="/stylesheets/style.css">
</head>
<body>
    <h1><%= title %></h1>
    <p>Fill in the form bellow to add a new post.</p>
    <form action="/post" , method="POST">
      <p><input type="text" name="entry[title]" placeholder="Title"></p>
      <p><textarea name="entry[body]" placeholder="Body"></textarea></p>
      <p><input type="submit" value="post"></p>
    </form>
</body>
</html>
```

​		渲染输出 Web UI 页面。新建文件 routers/entries.js。在 entryis.js 中写入现阶阶段需要的功能

```js
exports.form = (req, res) => {
  res.render('post', {title: 'POST'})
}
```



**step3：创建消息的动作对应的操作**

​		将用户创建的消息，保存至 redis 中，来做到模拟数据库存储的效果。在 views/entries.js 中继续写入以下代码。

```js
const Entry = require("../models/entry")

exports.submit = (req, res, next) => {
  const data = req.body.entry
  const user = res.locals.user
  const username = user ? user.name : null
  const entry = new Entry({
    username: username,
    title: data.title,
    body: data.body
  })
  entry.save(err => {
    if (err) return next(err)
    res.redirect('/')
  })
}
```



**step4：访问全部消息的列表页**

​		创建 views/etries.ejs 。写入下列代码：

```ejs
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title><%= title %></title>
</head>
<body>
  <% entries.forEach((entry) => { %>
    <div class="entry">
      <h3><%= entry.title %></h3>
      <p><%= entry.body %></p>
      <p> Posted by <%= entry.username %> </p>
    </div>
  <% }) %>
</body>
</html>
```

​		在 routers/entries 文件中设置对应的路由：

```js
exports.list = (req, res, next) => {
  Entry.getRange(0, -1, (err, entries) => {
    if (err) next(err)
    res.render('entries', {
      title: 'Entries',
      entries: entries
    })
  })
}
```



最后将所有的路由都注册到 app.js 中

```js
const entries = require('./routes/entries')

app.get('/post', entries.form)
app.post('/post',entries.submit)
app.get('/', entries.list)
```



**亿点点优化**

​		我们应该构建灵活的检验中间件来保证程序的正常运行。程序在运作过程中会出现各种意外，可能是自身的，也可能是外在的23，那么使用中间件来保证使用是不可或缺的。

- 定义公共方法

定义公共的方法，用于中间件的使用。

```js
function parseField(filed) {
  return filed
    .split(/\[|\]/)
    .filter(s => s)
}

function getField(req, filed) {
  let val = req.body
  filed.forEach(prop => {
    val = val[prop]
  })
  return val
}
```



- 校验必填项

定义一个校验必填项的中间件来保证必填项的输入。要求可以灵活使用，可以重复使用。

```js
exports.required = (filed) => {
  filed = parseField(filed)
  return (req, res, next) => {
    if (getField(req, filed)) {
      next()
    } else {
      res.end(`${filed.join(' ')} is required`)
      res.redirect('back')
    }
  }
}
```



- 判断长度是否符合提交的条件

定义一个用于校验是否合规的中间件，减少脏数据的产生。

```js
exports.lengthAbove = (field, len) => {
  field = parseField(field)
  return (req, res, next) => {
    if (getField(req, field).length > len) {
      next()
    } else {
      const fields = field.join(' ')
      res.end(`${fields} must have more than ${len} chartacters`)
      res.redirect('back')
    }
  }
}
```



#### 用户认证

从新创建一个认证系统，改系统包括：

- 存储和认证已注册用户；
- 注册功能；
- 登录功能；
- 加载用户信息的中间件。

还是用 Redis 作为用户账号的存储。



###### 保存和加载用户记录

本部分要实现用户加载、保存、和认证。任务清单是：

- 用 packages.json 定义程序的依赖项；
- 创建用户模型；
- 用 redis 加载和保存用户信息；
- 用 bcrypt 增强用户密码的安全性；
- 实现用户认证。

Bcrypt 是一个加盐的哈希函数。可作为专门模块对密码做哈希处理。



###### 创建用户模型

创建 model/user.js 文件作为用户模型，并写入以下代码。

- 创建用户模型。
- 添加 save 方法，将用户保存至 redis。
- 添加 update 方法，更新 redis 中的用户。
- 添加加密函数，hashPassword 进行加盐加密处理密码函数。
- 静态方法 `getId` 查询 id，静态方法 `getByName` 根据 用户名查询用户。 静态方法 `get`  获取一个新的对象模型。
- 静态方法 `authenticate` 进行用户名和密码校验。

```js
const redis = require('redis')
const bcrypt = require('bcrypt')
const db = redis.createClient()

class User {
  constructor(obj) {
    for (const key in obj) {
      this[key] = obj[key]
    }
  }

  save(cb) {
    if (this.id) {
      this.update(cb)
    } else {
      db.incr('user:id', (err, id) => {
        if (err) return cb(err)
        this.id = id
        this.hashPassword((err) => {
          if(err) return cb(err)
          this.update(cb)
        })
      })
    }
  }

  update(cb) {
    const id = this.id
    db.set(`user:id:${this.name}`, id ,err => {
      if (err) return cb(err)
      db.hmset(`user:${id}`, this, err => {
        cb(err)
      })
    })
  }

  hashPassword(cb) {
    bcrypt.genSalt(12, (err, salt) => {
      if (err) return cb(err)
      this.salt = salt
      bcrypt.hash(this.pass, salt, (err, hash) => {
        if (err) return cb(err)
        this.pass = hash
        cb()
      })
    })
  }

  static getByName(name, cb) {
    User.getId(name, (err, id) => {
      if (err) return cb(err)
      User.get(id, cb)
    })
  }

  static getId(name, cb) {
    db.get(`user:id:${name}`, cb)
  }

  static get(id, cb) {
    db.hgetall(`user:${id}`, (err, user) => {
      if (err) return cb(err)
      cb(null, new User(user))
    })
  }

  static authenticate(name, pass, cb) {
    User.getByName(name, (err, user) => {
      if (err) return cb(err)
      if (!user.id) return cb()
      bcrypt.hash(pass, user.salt, (err, hash) => {
        if (err) return cb(err)
        if (hash === user.hash) return cb(null, user)
        cb()
      })
    })
  }

}

module.exports = User
```



#### 注册新用户

注册功能的实现：

:ballot_box_with_check:：将注册和登录路由映射到URL地址上；

:ballot_box_with_check:：添加显示注册表单的注册路由处理器；

:ballot_box_with_check:：实现用户数据存储功能，存储从表单提交上来的数据。



1. 添加注册路由
		在 app.js 中添加相关的路由。并且在 router/register 定义相关的路由逻辑。先处理渲染注册模板的路由。
	
	```js
	exports.form = (req, res) => {
	  res.render('register', {title: 'Register'})
	}
	```



2. 创建注册表单

   ​	创建 views/reigster.ejs 提供注册的 HTML 页面。此时事先创建好消息模块，后续在创建消息提醒的 ejs 文件。

   ```ejs
   <!DOCTYPE html>
   <html>
     <head>
       <title><%= title %></title>
       <link rel="stylesheet" href="/stylesheets/style.css">
     </head>
     <body>
       <h1><%= title %></h1>
       <p>Fill in the form below to sign up!</p>
       <% include messages %>
       <form action="/register" method="POST">
         <p>
           <input type="text" name="user[name]" placeholder="Username">
         </p>
         <p>
           <input type="password" name="user[pass]" placeholder="Password">
         </p>
         <p><input type="submit" value=" Sign Up"></p>
       </form>
     </body>
   </html>
   ```

   

3. 把反馈消息传达给用户。

   ​	在大多数应用场景中，将反馈消息传达给用户都是必须要做的工作。此程序中的 messages.ejs 就是用来做消息处理的。它会被嵌入到许多模板之中。

   ​	在 views 目录下创建一个名为 messages.ejs 的文件，把下面的代码写入到这个文件里。折断代码会检查 locals.messages 是否存在，如果有，就会循环遍历这个变量以显示消息对象，根据 type 属性来定义消息的类型。调用完消息后要通过 removeMessage 来清空消息。

   ```ejs
   <% if (locals.messages) { %>
     <% messages.forEach((message) => { %>
       <p class='<%= message.type %>'><%= message.message %></p>
     <% }) %>
     <% removeMessages() %>
   <% } %>
   ```



4. 在会话中存放临时的消息。

   **PRG**

   ​	Post/Redirect/Get（PRG）是一种常用的 Web 程序设计模式。这种模式是指，用户请求表单，表单数据作为 `HTTP POST` 请求被提交，然后用户被重定向到另外一个 Web 页面上。用户被重定向到哪里取决于表单数据是或否有效。如果表单数据无效，程序会让用户回到表单页面。如果表单数据有效，程序会让用户到新的 Web 页面中。PRG 模式主要是为了防止表单的重复提交。

   **Express中如何处理**

   ​	在 Express 中，用户被重定向后，res.locals 中的内容也会被重置。如果把发给用户的消息存在 res.locals 中，这些消息在显示之前就已经丢失了。把消息存在会话变量中可以解决这个问题。确保消息在重定向后的页面上仍然可以显示。

   **维护用户消息队列**

   ​	创建文件 ./middleware/messages.js ，写入下列代码：

   ```js
   const express = require('express')
   
   function message(req) {
     return (msg, type) => {
       type = type || 'info'
       let sess = req.session
       sess.message = sess.message || []
       sess.message.push({type: type, message: msg})
     }
   }
   
   module.exports = (req, res, next) => {
     res.message = message(req)
     res.error = msg => {
       return res.message(msg, 'error')
     }
     res.locals.messages = req.session.message || []
     res.locals.removeMessages = () => {
       req.session.messages = []
     }
     next()
   }
   ```

   ​	这个功能需要会话（session）支持，我们可以使用官方支持的包：express-session。用 `yarn add express-session` 安装，然后添加到 app.js 中。

   ```js
   const session = require('express-session')
   ...
   app.use(session({
     secret: 'secret',
     resave: false,
     saveUninitialized: true,
   }))
   ...
   ```

   ​	这个中间件最好放在 cookie 之后。保证了 session 的正常使用之后，我们需要把 messages 也添加到 app.js 中。

   ```js
   const messages = require('./middleware/messages');
   ...
   app.use(messages)
   ...
   ```



5. 实现用户注册

   ​	之前已经在前端进行了表单提交的操作，我们需要对表单提交的操作做额外的处理。表单提交路由处理器的代码很少，我们只需要处理校验，比如确保用户名被占用；还有保存新用户。

   ​	注册已完成，就会把 user.id 赋值给会话变量，稍后还要通过检查它来判断用户是否通过了认证。如果检验失败，消息会作为 message 变量通过 res.locals.messages 输出到模板中，并且用户会被重定向回注册表单。

   ```js
   const User = require("../models/user")
   
   exports.submit = (req, res, next) => {
     const data = req.body.user
     User.getByName(data.name, (err, user) => {
       if (err) return next(err)
   
       if (user.id) {
         res.error('Username already taken!')
         res.redirect('back')
       } else {
         let user = new User({
           name: data.name,
           pass: data.pass
         })
         user.save(err => {
           if (err) return next(err)
           req.session.uid = user.id
           res.redirect('/')
         })
       }
     })
   }
   ```




#### 总结

- Connect 是一个HTTP框架，可以处理请求前和之后堆叠中间件。
- Connect 中间件是个函数，它的参数包括 Node 的请求和响应对象，一个调用下一个中间件的函数，以及一个可选的错误对象。
- Express Web 程序也是用中间件搭建的。
- 在用 Express 实现 REST API 时，可以用 HTTP 谓定义路由。
- Express 路由的响应可以是 JSON、HTML 或其他格式的数据。
- Express 有个简单的模板引擎 API，支持很多种引擎。

