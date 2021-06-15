## Node Web

- 创建一个新的 Web 服务
- 搭建 RESTful 服务
- 持久化数据
- 使用模板

### 创建一个 web 服务

#### 使用 express 创建一个  web 服务

**why?**

相比于node原有的 http 模板，express 更便捷，不需要做很多套路化的开发工作

**核心code**

```js
const express = require('express')
const app = express()

const port = process.env.PROT || 3000

app.get('/', (req, res) => {
    res.send('Hello World')
})

app.listen(port, () => {
    console.log(`Express web app available at localhost:${port}`)
})
```



### 搭建 restful 服务

**what is restful?**

相同的接口，使用不同的请求方式，以达到不同的效果。

#### 使用 express 搭建 restful 服务

experss 自身的拥有相当丰富的 api 可以很便捷的搭建出 restful 服务

```js
const express = require('express')
const app = express()

app.get('/article', (req, res) => {
    // do something by get
    // get something
})

app.post('/article', (req, res) => {
	// do something by post
    // create something
})

app.put('/article', (req, res) => {
    // do something by put
    // edit something
})

app.delete('/article', (req, res) => {
    // do someting by delete
    // delete something
})

app.listen(port, () => {
    console.log('express server is runing')
})

```

#### 添加消息体（请求体）解析器

⭐ body-parser 已经被 **废弃**，不推荐使用，在 express@4.16.0之后，express就可以拓展支持解析，请求体。

```js
const express = require('express')
cosnt app = express()

app.use(express.json())
app.use(express.urlencoded({ extended: true }))

app.set('PORT', process.env.PORT || 3000)

// ... 
app.post('/article', (req, res, next) => {
    const url = req.body.url // 直接获取requestBody的
})

app.listen(app.get('PORT'), () => {
  console.log('App srarted on port', app.get('PORT'))
})
```



### 持久化数据

选择何是的数据库系统和数据库模块需要你根据自己的需求做一些前期的调研。

对于小项目来说，SQLite很好用。

#### 添加数据库

就往 Node 程序添加数据库而言，并没有一定之规，但一般会设计下面几个步骤。

- 决定想要用的数据库系统。
- 在 npm 上看看耐饿是心啊了数据库驱动或对象--关系映射（ORM）的热门模块。
- 用 npm --save 将模块添加到项目中。
- 创建模型，封装数据库访问 API。
- 把这些模型添加到 Express 路由中

#### 制作自己的模型 Api

介于此项目用于演示，使用轻量级的 `sqltie3` 来使用，sqlite3 不需要安装，是一个运行在内存中的数据库。

```js
const sqlite3 = require('sqlite3').verbose()
const dbName = 'later.sqlite'
const db = new sqlite3.Database(dbName)

db.serialize(() => {
  const sql = `
    CREATE TABLE IF NOT EXISTS articles
      (id integer primary key, title, content TEXT)
  `;
  db.run(sql)
})

class Article {
  static all(cb) {
    db.all('SELECT * FROM articles', cb)
  }

  static find(id ,cb) {
    db.get('SELECT * FROM articles WHERE id = ?', id, cb)
  }

  static create(data, cb) {
    const sql = `INSERT INTO articles(title, content) VALUES(?, ?)`
    db.run(sql, data.title, data.content, cb)
  }

  static delete(id, cb) {
    if (!id) return cb(new Error('Please provide an id'))
    db.run('DELETE FROM articles WHERE id = ?', id, cb)
  }
}


module.exports = {
  Article,
  db,
}

```



#### 将 Articles 模块添加到 HTTP 路由中

```js
const express = require('express')
const Article = require('./db').Article
const app = express()
const read = require('node-readability')
const url = 'http://www.manning.com/cantelon2'


app.set('PORT', process.env.PORT || 3000)

app.use(express.json())
app.use(express.urlencoded({ extended: true }))

/**
 * @description 获取全部的文章
 */
app.get('/articles', (req, res, next) => {
  Article.all((err, articles) => {
    if (err) return next(err)
    res.send(articles)
  })
})

/**
 * @description 根据id，查询某一条文章
 */
app.get('/articles/:id', (req, res, next) => {
  const id = req.params.id
  Article.find(id, (err, article) => {
    if (err) return next(err)
    res.send(article)
  })
})

app.post('/articles', (req, res, next) => {
  const url = req.body.url
  read(url, (err, data) => {
    if (err || !data) res.status(500).send('Error downloading article')
    Article.create(
      { title: data.title, content: data.content },
      (err, article) => {
        if (err) return next(err)
        res.send('ok')
      }
    )
  })
})

app.delete('/articles/:id', (req, res) => {
  const id = req.params.id
  Article.delete(id, (err) => {
    if (err) return next(err)
    res.send({message: 'success'})
  })
})

app.listen(app.get('PORT'), () => {
  console.log('App srarted on port', app.get('PORT'))
})

module.exports = app

```



### 添加模板

用 `curl` 发送请求时， JSON很方便，因为在控制台里看起来很清晰。但在现时应用中，这个程序还需要支持 HTML。

####  format方法

使用 `res.format` 方法。它可以`根据请求`发送相应格式的响应。

```js
res.format({
    html: () => {
        res.render('articles.ejs', {articles: articles})
    },
    json: () => {
        res.send(articles)
    }
})
```

#### 渲染模板

模板引擎有很多，EJS属于简单易学那种。`res.render` 可以渲染 EJS 格式的 html 文件。回去默认寻找，项目目录下的 `view` 文件夹中的 相对应的ejs文件。