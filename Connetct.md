## Connect

target

- 了解 Connect 是用来做什么的。
- 中间件的使用以及创建。

```tex
 Express 就是在 Connect 的基础上，通过高层糖衣拓展和搭建出来的。
```



#### 创建 Connect 程序

Connect 以前是 Express 的基础，但实际上只用 Connect 也能做出完整的 Web 程序。

安装：

$ npm install connect@3.4.0

最简单的程序

```js
const app = require('connect')()

app.use((req, res, next) => {
    res.end('Hello world')
})
app.listen(3000)
```



#### 了解 Connect 中间件的工作机制

​		Connect 的中间件就是 JavaScript 函数。这个函数一般会有三个参数：请求对象、响应对象，以及一个名为 `next` 的回调函数。一个中间件完成自己的工作，要执行后续的的中间件时，可以调用这个回调函数。

​		在中间件执行之前，Connect 会用分派器接管请求对象，然后交给程序中的第一个中间件。

![](https://raw.githubusercontent.com/less-sugar/about-nodejs/master/images/Connect-MiddleWare.png)

- 分派器收到请求，把它传给第一个中间件。
- 记录请求日志，并用 next 传递给下一个中间件。
- 如果有请求体会被解析，传递给下一个中间件。
- 如果是静态文件，用那个文件做响应，不再调用 next()；否则请求进入下一个中间件。
- 请求被一个定制的中间件处理好，请求结束。



#### 组合中间件

​		Connect 的 use 方法就是用来组合中间件的。 

​		组合 hello 函数和 logger

```js
const connect = require('connect')
function logger(req, res ,next) {
  console.log(req.method, req.url)
  next()
}

function hello(req, res, next) {
  res.setHeader('Content-type', 'text/plan')
  res.end('Hello world')
}

connect()
  .use(logger)
  .use(hello)
  .listen(3000)
```



##### 中间件的顺序。

​		next() 函数表示执行下一个中间件，如果中间件的顺序出现问题，那么可能会导致程序错过，漏过某些中间件的执行。如果不调用 next()，就会阻碍后续的中间件的执行。

​		将上述例子，hello 中间件和 logger 中间件顺序倒过来，就会出现问题。

```js
const connect = require('connect')
function logger(req, res ,next) {
  console.log(req.method, req.url)    <==== 总是调用 next(), 所以后续中间件总会被调用
  next()
}

function hello(req, res, next) {
  res.setHeader('Content-type', 'text/plan')
  res.end('Hello world')			  <==== 不会调用 next()，因为组件响应了请求
}

connect()
  .use(hello)				<==== 因为 hello 不会调用 next(), 所以，logger永远不会被调用
  .use(logger)
  .listen(3000)
```

​		执行完 next() 之后，控制权会移交给分派器。如果某个中间件不调用 next() 那链在它后面的中间件就不会调用。



##### 创建可配置的中间件

​		为了做到可配置，中间件一般会遵循一个简单的惯例：用一个函数返回另一个函数（闭包）。基本结构如下所示：

```js
function setup(options) {
    // 设置逻辑
    
    return function(req, res, next) {
        // 中间件逻辑
    }
}
 
app.use(setup({some: 'options'}))
```



​		现有的 logger 中间件是不可配置的，只能输出 req.method 和 req.url。但是在实际工作中这往往是不合理的，如果要创建可配置的 logger 要怎么做？

```js
function setup(format) {			<=== setup 函数可以用不同的配置，调用多次
    const regexp = /:(\w+)/g;		<=== logger 组件用正则表达式匹配请求属性
    
    return function createLogger(req, res, next) {  	<== Connect 使用的真实 logger组件
        const str = format.replace(regexp, (mathc, property) => {
            return req[property]		<=== 用正则表达式格式化请求的日志条目
        })
        console.log(str)		<=== 将日志条目输出到控制台
        next()			<=== 将控制权交给下一个中间件组件
    }
}

module.expores = setup		<=== 直接导出 logger 的 setup 函数
```



##### 使用错误处理中间件

​		任何程序都有出错的可能。不管是在用户层面还是在系统层面，面对错误，朕只是无法预料的错误，应当事先将错误提前抛出，处理错误才是正确的选择。Connect有一种用来处理错误的中间体变体，和常规的中间件不同的是，多了请求、响应对象外还多一个错误处理对象。Connect处理中间件的工作机制以及一些实用的模式。

- 用 Connect 的默认错误处理器
- 自行处理

###### 用Connect的默认错误处理器

```js
const connect = require('connect')
connect()
	.use((req, res) => {
        foo()
    	res.setHeader('Content-type': 'text/plan')
    	res.end('hello world')
    })
	.listen(3000)
```

​		在上述的代码中，foo()没有定义，所以这个中间件会抛出错误 ReferenceError。

​		Connect 默认的处理是返回响应状态码 500，响应主体是文本 Internal Server Error 和错误的详细信息。这无可厚非，但在真正的程序中一般还会对这些错误做些特殊处理，比如将它们发送给守护进程。

###### 自行处理程序错误

​		错误处理中间件必须有四个参数：err，req，res 和 next，而常规的中间件只有 req，res，next 三个参数。

```js
const env = process.env.NODE_ENV || 'devolopment'

function errHeander(err, req, res, next) {
    res.status = 500
    switch(env) {
        case 'development':
            console.error('Error')
            console.error(err)
            res.setHeader('Content-type', 'application/json')
            res.end(JSON.stringify(err))
        default: 
            res.end('Server Error')
    }
}
```

​		当 Connect 遇到错误时，它会切换，只去调用错误处理中间件。

