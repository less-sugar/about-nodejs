## 基础部分

### 异步编程

nodejs中 IO 都属异步进行的（也可以使用同步API），因此对于异步的结果，通常使用回调函数进行接受或者处理。因此在多重操作的时候，就会产生回调之内套用另外一个回调的情况。

#### 通过调用 function 的方式来减少回调的产生

```js
http.createServer((req, res) => {
  getTitles(res)
}).listen(8000, '127.0.0.1')

function getTitles(res) {
  fs.readFile('./titles.json', (err, data) => {
    if (err) {
      hadError(err, res)
    } else {
      getTemplate(JSON.parse(data.toString()), res)
    }
  })
}

function getTemplate(titles, res) {
  fs.readFile('./template.html', (err ,data) => {
    if (err) {
      hadError(err, res)
    } else {
      formatHtml(titles, data.toString(), res)
    }
  })
}

function formatHtml(titles, tmpl, res) {
  const html = tmpl.replace('%', titles.join('</li><li>'))
  res.writeHead(200, {'Content-Type': 'text.html'})
  res.end(html)
}

function hadError (err, res) {
  console.log(err)
  res.end('Server Error')
}
```



#### 通过提前 return 减少 if...else

在nodejs中，大部分情况都是错误优先。因此，每次执行代码之前都需要判断是否存在错误，所以，可以在发生错误的时候提前 return 来结束函数的进行。

```js
function getTitles(res) {
  fs.readFile('./titles.json', (err, data) => {
    if (err) return hasError(err, res)
    getTemplate(JSON.parse(data), res)
  })
}

function hasError(err, res) {
  res.end('Server Error')
  throw err
}
```



### 事件发射器

事件发射器会触发事件，并且在那些事件被触发时能处理它们。

#### net 模板实现 echo 服务器

echo服务器，当有客户端链接上来时，就会创建一个socket。socket就是事件发射器。

```js
const net = require('net')						读取到新数据时，处理的data 事件
const server =  net.createServer(socket => {		|
    socket.on('data', data => {		  <——————-------|
        socket.write(data)  <---|
    })							|
})						 数据被写回到客户端
server.listen(8888)
                               
// 通过 once 只触发一次
const net = require('net')
const server = net.createSetver(socket => {
    socket.once('data', data => {
        socket.write(data)
    })
})
server.listen(8888)
```

#### 通过 events 模板和 net 模块实现简易聊天室

原理：events 模块的 EventEmitter 可以创建一个事件发射器，net 模块可以监听用户链接，然后通过接收用户的输入，并将信息传递给其他的连接用户。

现阶段存在的问题：

- 中文乱码问题
- 每一次输入都会触发事件发送

实现步骤：

- 添加 join 事件的监听器，保存用户的 clinet 对象，以便程序可以将数据发送给用户
- 忽略发出这一条广播的用户自身
- 添加一个专门针对当前用户的broadcast事件监听器
- 当有用户连接到服务器上时，发出一个 join 事件，知名用户 id 和 clinet 对象
- 当有用户发送数据时，发出一个频道 broadcast 事件，指明用户 id 和 消息
- 当有用户断开连接时，发出一个频道 leave 事件，指明用户 id
  - 如果想停止服务又不想关闭服务器，可以使用 removeAllListeners，去掉指定类型的全部监听器	
- 开启服务端口

```js
const events = require('events')
const net = require('net')
const channel = new events.EventEmitter()

channel.clients = {}
channel.subscriptions = {}

channel.on('join', function (id, client) {
  this.clients[id] = client
  this.subscriptions[id] = (senderId, message) => {
    if (senderId !== id) {
      this.clients[id].write(message)
    }
  }
  this.on('broadcart', this.subscriptions[id])
})
channel.on('leave', function (id) {
  channel.removeListener('broadcart', this.subscriptions[id])
  channel.emit('broadcart', id, `${id} has left the chatroom. \n`)
})
channel.on('shutdown', () => {
  channel.emit('broadcart', '', 'the server has shut down. \n')
  channel.removeAllListeners('broadcart')
})


const server = net.createServer(client => {
  const id = `${client.remoteAddress}:${client.remotePort}`
  channel.emit('join', id, client)
  client.on('data', data => {
    data = data.toString()
    if (data === 'shutdown') {
        channel.emit('shutdown')
    }
    channel.emit('broadcart', id, data)
  })
  client.on('close', () => {
    channel.emit('leave', id)
  })
})

server.listen(8888)
```

### 

### 实现串行控制

串行：顾名思义，就是让代码想一串一样，一个一个的进行

#### 何时使用串行

回调函数可以让异步任务同步进行，但是如果任务比较多，就必须组织一下，否则过多的回到嵌套会把代码搞得很乱。或者说，当后者需要前者的结果的时候，就需要使用串行。

#### 如何控制串行

**how?**

需要事先把这些任务按与其的执行顺序放到一个数组中，完成一个任务后按照顺序从数组中取下一个。

![](.\images\control-series.png)

**核心code**

通过回调调用next，next内部调用下一个任务，保证任务的有序进行。

```js
// step:3 定义相关任务函数
function checkForRssFile() {
  if (!fs.existsSync(configFilename)) {
    return next(new Error(`Missing Rss File: ${configFilename}`))
  }
  next(null, configFilename)
}

function readRssFile(configFilename) {
  fs.readFile(configFilename, (err, feedList) => {
    if(err) return next(err)
    feedList = feedList.toString().replace(/^\s+|\s+$/g, '').split('\n')
    const random = Math.floor(Math.random() * feedList.length)
    next(null, feedList[random])
  })
}

function downloadRSSFeed(feedUrl) {
  request({uri: feedUrl}, (err, res, body) => {
    if (err) return next(err)
    if ( res.statusCode !== 200) return next (new Error('Abnormal response status code'))
    next(null, body)
  })
}

// step1: 定义一个预先执行顺序的数组
const tasks = [
  checkForRssFile,
  readRssFile,
  downloadRSSFeed,
  parseRSSFeed
]

// step2: 定义一个执行顺序函数
function next(err, result) {
  if (err) throw err
  const currentTask = tasks.shift()
  if (currentTask) {
    currentTask(result)
  }
}

// step4: 执行
next()

```

#### 如何控制并行任务

**what？**

并行，顾名思义就是并列执行，不考虑先后顺序，当所有任务执行完成之后，主程序才执行完。

**how？**

首先将所有执行任务放置于一个数组当中，后续遍历数组，执行任务。

![](.\images\control-parallel.png)

**code**

```js
const fs = require('fs');
const tasks = []
const wordCounts = {}
const filesDir = './text'
let completedTasks = 0

function checkIfComplete() {
  completedTasks++
  if (completedTasks === tasks.length) {
    for(let index in wordCounts) {
      console.log(`${index}: ${wordCounts[index]}`)
    }
  }
}

function addWordCount(word) {
  wordCounts[word] = wordCounts[word] ? wordCounts[word] + 1 : 1
}

function countWordsInText(text) {
  const words = text.toString()
                    .toLowerCase()
                    .split(/\W+/)
                    .sort()

  words.filter(word => word)
      .forEach(word => addWordCount(word))
  console.log(text)
}

fs.readdir(filesDir, (err, files) => {
  if (err) throw err
  files.forEach(file => {    <====== 将所有 task 放置于 tasks 中
    const task = (file => {
      return () => {
        fs.readFile(file, (err, text) => {
          if (err) throw err
          countWordsInText(text)
          checkIfComplete()
        })
      }
    })(`${filesDir}/${file}`)
    tasks.push(task)
  })
  tasks.forEach(task => task()) <====== 遍历任务队列，直接执行所有任务
})
```

