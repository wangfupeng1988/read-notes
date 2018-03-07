# 《深入浅出 nodejs》读书笔记

本书购于 2015.9.30 ，至今快两年半的时间了。购书当时已经看完，但是现在再回头翻看，依然觉得很有收获。因为本书讲解的内容较为底层、原理，并且 nodejs 的基础的 API 一直也没有变过，因此就不过时。正好近期想写一篇关于 nodejs 的系列博客，正好拿本书做参考，重读并做读书笔记。

本书作者是阿里资深工程师朴灵，nodejs 步道师。书中前言部分就提到，作者作为一名 FE 转型成为 nodejs 后端工程师，所具有的优势仅仅是对 JS 语法熟悉而已，但是对后端开发的其他知识并不擅长，需要重新学习。我想这也是我等 FE 对 nodejs 的一个看不透彻的地方，总以为会了 JS 就很容易学会 nodejs 。

----

## 第一章 nodejs 介绍

**为什么叫`node`**？因为整个 server 端涵盖范围很大，由很多节点构成，而它可以作为 server 端的一个节点存在，因此叫`node`。另外，前两天曾经有这么一段分歧，有人 fork 除了 nodejs 的源码，新建了一个`IO.js`，和 nodejs 形成竞争。虽然现在没有了，但是可以看出 **nodejs 的核心竞争力在`异步 IO`**。

-----

## 第二章 Module

本章从原理上介绍 nodejs 如何实现 CommonJS 规范的。CommonJS 语法不在赘述，此处总结一些比较重要的（或者有趣的）地方

### require

CommonJS 中`require('....')`是 nodejs 启动的时候**同步加载**，会阻塞执行。加载过程会分为以下步骤：

- 尝试加载核心模块，如`http` `fs`等
- 尝试加载 npm 安装的第三方模块，名称都在 package.json 文件中
- 尝试通过相对路径或者绝对路径加载文件，此时 nodejs 会按照 `.js` `.json` `.node` 的次序不足扩展名，依次尝试

### exports

下面是一个比较有意思的事情，此前我都没仔细考虑过。如新建一个`a.js`文件，内容如下

```js
var fs = requre('fs')
module.exports = function () {
    console.log(__dirname)
    console.log(__filename)
}
```

这里有一个问题，按照 JS 语法来说，以上代码中 `require` `module` `__dirname` `__filename` 都是未定义的变量，应该会报错的。其实，**在编译过程中，nodejs 会获取这段代码，然后进行一个函数封装**，然后变成这样，问题就解决了。

```js
// 上面提出的未定义的变量，都是函数的参数，是执行时被传入的
(function (exports, require, module, __filename, __dirname) {
    var fs = requre('fs')
    module.exports = function () {
        console.log(__dirname)
        console.log(__filename)
    }
})
```

### 底层流程

书中又继续介绍了 nodejs 底层是如何实现模块化引入的，列出很多 c++ 代码，我一目十行看了下，了解甚少。

----

## 第三章 异步 I/O

nodejs 核心优势就在于异步 I/O ，这种基于事件驱动的方式，跟 Nginx 很相似（书中提及，我又询问后端同事确认）。相比而言，PHP 是从头到脚的同步执行（当然这样很简单），Apache 服务器针对每个请求都新建一个线程，请求过多内存爆满。

前端为了避免 DOM 渲染的冲突，JS 不得不单线程运行，解决单线程的方案就是异步，继而一并引用到 nodejs 中。单线程能避免多线程的死锁、状态同步的问题，还能通过异步来解决性能问题。 nodejs 也提供过了`Child_Process` `Cluster`这些模块来解决多进程的问题。

接下来书中详细介绍了异步 I/O 的实现过程，和 event-loop 一样，基于观察者模式的事件循环。至于太具体的底层实现，倒是不急于详细了解。

最后书中介绍了非 I/O 的异步 API 

- `setTimeout(fn, 0)` 回调函数存储在红黑树中，遍历查找比较浪费性能
- `process.nextTick()` 回调函数存储在数组中，每次轮询回调函数全部执行。**较轻量，推荐优先使用**
- `setImmediate()` 回调函数存储在链表中，每次轮询只执行链表中的一个

----

## 第四章 异步编程

异步是 JS 不变的话题，无论出现什么解决方案、工具，都无法改变 JS 单线程和异步的本质。**异步编程最大的优势就是 —— 非阻塞 IO**

### 问题

根据实际开发，总结两个最主要问题。

**第一个问题，异常捕获**。因为代码是异步执行，异常捕获肯定不像同步代码那么简单。针对这个问题，**nodejs 做了一个约定 —— callback 中第一个参数必须是 error ，如果没有错误，`error == null`即可**。例如

```js
fs.readFile('./xxx.log', function (err, data) {
    if (err == null) {
        // 正确读取了
    } else {
        // 发生了错误
    }
})
```

**第二个问题，callback 嵌套层次**，这个问题不再赘述。

### 解决方案

解决方案肯定少不了 Promise ，这里不再赘述，[《深入理解 JS 异步》](https://github.com/wangfupeng1988/js-async-tutorial) 中有详细讲解，当然书中也有详细讲解。

**书中提到的第二种方式是`next`的方式**，即 express koa 中的中间件。例如所有的中间件函数会存在`stack`数组中，那么`next`可以这样模拟：

```js
function next(err) {
    layer = stack[index++]
    layer.handle(req, res, next)
}
```

这样在`layer`函数体中手动调用`next()`函数，即可触发下一个`layer`，实现异步串联的功能。

**书中提到的第三种方式是借用一些 npm 库**，如 **[async](https://github.com/caolan/async)** （长期占据 npm 前几名）和 **[step](https://www.npmjs.com/package/step)** ，这俩库都可以实现异步函数的串行、并行流程控制。具体用法，可参见文档说明。

### 异步并发控制

这种大量并发的问题，我在日常开发中还真的没有仔细考虑过，应该引起重视。**以`readFile`为例，如果异步请求的并发量过大，OS 的文件描述符数量将会被瞬间光，会抛出错误**：

```js
var i = 0;
for (i = 0; i < 10000; i++) {
    fs.readFile(...)
}
// 并发量过大会抛出错误：Error: EMFILE, too many open files
```

这种问题在同步 I/O 就不会出现，但是对于**异步 I/O 就不得不考虑给予一定的过载保护**。书中推荐了作者自己写的 [bagpipe](https://www.npmjs.com/package/bagpipe) ，用起来也很简单。

```js
// 未用过载保护
for (var i = 0; i < 100; i++) {
  async(function () {
    // Asynchronous call
  });
}

// 使用 bagpipe
var Bagpipe = require('bagpipe');
var bagpipe = new Bagpipe(10);  // Sets the max concurrency as 100
for (var i = 0; i < 100; i++) {
  bagpipe.push(async, function () {
    // execute asynchronous callback
  });
}
```

至于内部实现，其实就是对于异步回调队列的一个管理，技术实现没有难度上的阻碍。

----

## 第五章 内存控制

### V8

nodejs 是采用 v8 作为 js 引擎，**但是只能使用部分内存（64 位最多使用 1.4GB ，32 位最多使用 0.7GB）**。原因：

- 表面原因：v8 最初为浏览器设置，即便内存限制，也绝对够用
- 底层原因：**v8 的垃圾回收机制限制**。内存限制放开，垃圾回收会变慢。具体可见书中 113 页。

nodejs 也开放了修改的接口，可自定义内存限制。在 nodejs 最初运行的时候，可以通过如下方式来修改。一旦修改，运行过程中就不能再次改变了。

- `node --max-old-space-size=1700 test.js` 单位是`MB`
- `node --max-new-space-size=1400 test.js` 单位是`KB`

接下来，书中详细写 v8 的垃圾回收机制，写的很底层，很复杂，日常工作接触不到，没仔细看。

### 高效使用内存

其实答案很简单 —— **及时释放用不到的变量，复制为`null`或者直接`delete`掉** 。JS 使用闭包时，变量会一直保留在内存中，因此闭包要谨慎使用。

做浏览器的环境的前端开发，不用太过苛刻的关注内存使用，但是做 server 端开发，对内存的关注必须到苛刻、极致。这是一种开发思想上的转变，也是最难的。

### 内存泄露

如果 **缓存过多、队列消费不及时、作用域未释放** ，都会导致内存泄露，书中讲了许多例子。其中重点提到，数据量过大的缓存不应该直接使用 nodejs 变量，而是应该使用专业的分布式缓存工具，例如`redis` `memcached`

### 大内存应用

使用 stream 读取大文件

-----

## 第六章 理解 Buffer

### 介绍

Buffer 是典型的 JS 和 C++ 结合的模块，将性能部分用 C++ 实现，非性能部分用 JS 实现。**Buffer 所占用的内存不是 V8 分配的，属于堆外内存**（何为“堆外内存”可参考 [《汇编语言入门 阮一峰》](https://news.cnblogs.com/n/587863/) ）

Buffer 对象类似于数组，每个元素都是一个 16 进制的两位数（换算成 10 进制即 0-255 之间的数字）。

```js
var str = '深入浅出nodejs'
var buf = new Buffer(str, 'utf-8')
console.log(buf)  // <Buffer e6 b7 b1 e5 85 a5 e6 b5 85 e5 87 ba 6e 6f 64 65 6a 73>
```

上面的示例，中文文字采用`utf-8`编码下占用 3 个元素，英文占用 1 个元素。即：`深`对应`e6 b7 b1`，`如`对应`e5 85 a5`……

**Buffer 和字符串本质就不一样，Buffer 是二进制数据，和字符串之间存在编码关系。**

### 拼接

在许多 nodejs 的示例中，往往这样拼接 buffer

```js
var fs = require('fs')
var rs = fs.createReadStream('test.log')
var data = ''
rs.on('data', function (chunk) {  // chunk 是 Buffer 对象
    data += chunk  // 拼接 buffer
})
rs.on('end', function () {
    console.log(data)
})
```

其实这样拼接在中文字符串情况下会有问题。上面提到过，中文文字采用`utf-8`编码下占用 3 个元素，通过 stream 方式读取内存，很有可能一段 chunk 中，将一个中文（占 3 个元素）截断，这样就会先乱码。因此，正确的拼接方式如下：

```js
var chunks = []
var size = 0
res.on('data', function (chunk) {  // chunk 是 Buffer 对象
    chunks.push(chunk)
    size += chunk.length
})
res.on('end', function () {
    var buf = Buffer.concat(chunks, size)  // 拼接 buffer
    var str = iconv.decode(buf, 'utf-8')
    console.log(str)
})
```

### 性能

Buffer 处理性能问题，最常见的就是通过 Stream 读取大文件的内容。不过，书中也介绍了 **在 http 请求中，使用 Buffer 对性能的提升**，如下代码：

```js
var http = require('http')
// 模拟一个字符串
var helloWorld = ''
for (var i = 0; i < 1024 * 10; i++) {
    helloWorld += 'a'
}

// helloWorld = new Buffer(helloWorld)  // 加上这一句代码，将对性能有极大提升（QPS 从 2.5k 到 4.8k）！！！

http.createServer(function (req, res) {
    res.writeHead(200)
    res.end(helloWorld)
}).listen(8080)
```

通过预先将返回内容转换为 Buffer 对象，可以有效介绍 CPU 的重复使用，不做额外的转换，提高性能。

----

## 第八章 构建 web 应用

第七章 讲述了 nodejs 网络编程的概述，网络编程不只有 http 协议。但是，我们工作常用的基本都是 http 协议，于是直接关注本章 —— web 应用。本章用到的 http 协议的知识，可参考[《图解 http》读书笔记](./图解http.md).

### 基本应用 

该部分，书中花了很大的篇幅讲解 cookie 和 session ，我也觉得这两块是最重要的。

**`cookie`**

cookie 是每次 http 请求都会携带的信息，其本质就是一堆字符串。前后端都可以对 cookie 进行设置，后端设置的几个选项。可通过`Set-Cookie`进行修改，即`res.setHeader('Set-Cookie', 'xxxx')`

- `path`
- `Expires`
- `HttpOnly` （是否允许前端 JS 修改 cookie 项）
- `Secure` （仅允许 https 协议）

对于 cookie 有两点需要注意：

- 网页中发起跨域的 http 请求（ajax、加载静态资源）时候，不会携带 cookie ，这是浏览器的同源策略保证的
- **但是要注意命中 CSRF (跨站请求伪造)的情况**：浏览器已经访问过`a.com`的请求（访问时肯定带了 cookie），此时黑客在`b.com`中诱导用户访问`a.com`的请求，同样会携带此前访问`a.com`的 cookie 。**注意，同源策略限制的是`b.com`中访问`a.com`请求时，不携带`b.com`的 cookie ，但是会正常携带`a.com`的 cookie** 。


**`session`**

介绍 session 一般都会使用登录的场景。用户登录了之后，server 端会往 cookie 中添加一个`session_id`的唯一不重复的值（**过期时间一般为 20 分钟、并且设置`HttpOnly`**），用以跟 server 端进行通讯。此时，server 端为这个`session_id`初始化一个 session 对象。

```js
// 全局变量，存储所有用户的 session
var SESSIONLIST = {}

// 处理请求的中间件
function (req, res) {
    // 处理请求时，创建一个 session （刚刚登录时，创建 session）
    var session_id = Date.now() + Math.random()
    SESSIONLIST[session_id] = {}
    var session = SESSIONLIST[session_id]

    // 可以挂在到 req 上
    req.session = session
    req.session.visited = true

    // Set-Cookie，设置 session_id，过期时间 20 分钟，HttpOnly
}
```

此后发送的请求， server 端都会检查 cookie 中是否有`session_id`，如果没有了（或者不合法）则证明身份失效。如果有合法的`session_id`，那证明身份合法，可以使用`session`中的信息。

```js
// 处理请求的中间件
function (req, res) {
    // 试图获取 cookie 中的 session_id ，看是否不存在或者非法
    // 如果存在且合法的话：
    req.session = SESSIONLIST[session_id]
    console.log(req.session.visited)
}
```

这样，将用户的敏感信息存储到 session 要比存储到 cookie 安全的多。

如上代码，将所有用户的 session 信息全部存储到内存中，会带来一些问题：

- 用户过多，会导致内存不够使用
- 启动多进程（cluster 集群等），内存无法共享

这个问题的解决方案 —— 将 session 存储到第三方缓存服务中（如 Redis Memcached 等）。虽然这样还要考虑网络访问的性能问题，但是总有一些解决方案，比命中上述两个问题要好多了。

### 数据上传

通过req header 中，**有没有`Transfer-Encoding`或`Content-Length`可判断是否带有内容**。

```js
function hasBody(req) {
    return 'transfer-encoding' in req.headers || 'content-length' in req.headers
}
```

报文内容可通过`data`事件触发（Stream 形式）

```js
function (req, res) {
    if (hasBody(req)) {
        var buffers = []
        req.on('data', function (chunk) {
            buffers.push(chunk)
        })
        req.on('end', function (chunk) {
            req.rawBoby = Buffer.concat(buffers).toString()
            handle(req, res)
        })
    } else {
        handle(req, res)
    }
}
```

**form**

表单提交时，req header 中 **`Content-Type`等于`application/x-www-form-urlencoded`**

```js
function handle(req, res) {
    if (req.headers['content-type'] === 'application/x-www-form-urlencoded') {
        req.body = queryString.parse(req.rawBody)
    }
    todo(req, res)
}
```

**JSON**

即一般的 ajax 请求。书中还介绍了 XML 格式，目前 XML 使用不多，就此略过。JSON 格式时，**`Content-Type`等于`application/json`**

```js
function handle(req, res) {
    if (req.headers['content-type'] === 'application/json') {
        req.body = JSON.parse(req.rawBody)  // 此处应 try-catch 格式错误的情况
    }
    todo(req, res)
}
```

**附件上传**

HTML 中普通表单和特殊表单的区别就在于是否有`<file>`标签。如果需要有`<file>`标签，就需要指定表单需求 **`enctype`为`multipart/form-data`**

```html
<form action="/upload" method="post" enctype="multipart/form-data">
    <input ... >
    <file ... >
</form>
```

浏览器遇到这样的表单提交时，构造的报文头部就和普通表单不同了

```
Content-Type: multipart/form-data; boundary=AaB03x
Content-Length: 18231
```

其中`AaB03x`是随时字符串，`boundary=AaB03x`是分解符，报文的内容将通过在它前面添加`--`进行分割，报文结束时再它前后都添加`--`表示结束（可参考书中 198 页的例子）。`Content-Length`表示报文主体的长度。

因此，**server 端可通过判断`Content-Type`是否是`multipart/form-data`来处理附件上传**。书中介绍了用 [formidable](https://www.npmjs.com/package/formidable) 插件来处理，formidable 能接收文件并写入到系统的临时文件夹，使用方式可参考其文档。

### 路由解析

路由解析是一个 web server 基本功能，nodejs 提供了 [url](http://nodejs.cn/api/url.html) 和 [querystring](http://nodejs.cn/api/querystring.html) 两个 API 来处理路由。常用的框架 express 和 koa 也都有很方便的路由处理接口，而且是基本功能。基本的处理逻辑比较简单，就此略过。

书中还提到了 RESTful API ，这个值得记录一下。其实 **RESTful 的思路就是：将每个 url 地址都抽象为一个资源，通过 method 来规定具体的操作**。我觉得这一点和 linux 中将一切输入输出都作为文件一样，是一种规范性的抽象。例如，普通的 API 设计如下：

```
POST /user/add?username=jack
GET  /user/remove?username=jack
POST /user/update?username=jack
GET  /user/get?username=jack
```

上面这种方式，就讲操作行为混合在了 url 中，无法做到每个 url 都对应一个资源。使用 RESTful API 将会这样设计：

```
POST   /user/jack
DELETE /user/jack
PUT    /user/jack
GET    /user/jack
```

书中总结了 RESTful 的三个特点：

- 通过 url 设计资源
- method 定义资源的操作
- 通过`Accept`决定资源的类型

### 中间件

体验中间件的最好方式就是立马做一个 express 和 koa 的例子，代码格式如

```js
app.use(function (req, res, next) {
    todo(req, res)
    next()
})
```

中间件用于将业务逻辑细分，以符合单一职责原则和开放封闭原则。中间件可分为通用的和业务的两种，其中通用的是每个业务都需要的，例如：

- logger
- body 处理
- querystring 处理
- cookie  session 处理
- ……

如果是完全同步的代码，中间件用起来是很顺畅的，但是 server 端处理请求基本都会有异步 IO 的情况，因此如何在中间件中使用异步，是一个重要问题。其中，express 是使用比较传统的 callback 方式，koa 1.x 使用了 ES6 的 Generator 特性，而在如今的 koa 2.x 又升级使用了 ES7 草案的 `async/await` 特性，算是终极的解决方案了。

### 页面渲染

nodejs 没有御用的模板引擎，这一点不像 php asp jsp 等，需要自己去选择，例如 artTemplate 。书中也简单讲解了实现一个模板引擎的逻辑，我之前了解过 vue 中模板的解析，因此对这块逻辑也不算陌生。另外，模板解析的逻辑，大概了解即可，也无需详细深入，毕竟是工具性的东西。这里先略过。

接下来书中介绍了 Bigpipe ，需详细记录一下。普通的页面渲染，即便是首屏渲染，也是拿到所有该拿的数据之后，一次性吐出给前端。**而 Bigpipe 是将页面内容分成了多个部分（pagelet），然后分批逐步输出**。

首先，要向前端输出模板和接收 pagelet 的方法，其实就是一个 JS 方法，该方法接收 DOM 选择器和内容，然后将内容渲染到 DOM 节点中。接下来，server 端异步请求数据，然后分批输出到前端去渲染，如下代码。nodejs 异步请求是部分顺序的，因此下面两个异步，哪个先输出不知道——也无需知道，先查询出来的先输出即可。

```js
app.get('/profile', function (req, res) {
    var num = 0
    db.getData('sql1', function (err, data) {
        res.write('<script>bigpipe.set("articles", "' + JSON.stringify(data) + '")</script>')
        num++
        if (num === 2) {
            res.end()
        }
    })
    db.getData('sql2', function (err, data) {
        res.write('<script>bigpipe.set("copyright", "' + JSON.stringify(data) + '")</script>')
        num++
        if (num === 2) {
            res.end()
        }
    })
})
```

这种多 pagelet 分批下发的方式，ajax 也可以办到。但是 ajax 每次都是一个独立的 http 请求，而 Bigpipe 共用相同的请求，开销十分小。

----

## 第九章 玩转进程

对应 nodejs API 中 [child_process](http://nodejs.cn/api/child_process.html) 和 [cluster](http://nodejs.cn/api/cluster.html) ，开源社区还有 [pm2](https://www.npmjs.com/package/pm2) 工具。

### 事件驱动

**Apache 是采用多线程/多进程模型实现的**，当并发连接数量达到 10k 级别时，内存耗用问题就会暴露出来，这就是著名的 [C10K](http://www.kegel.com/c10k.html) 问题。**而 node 和 Nginx 都是采用事件驱动的方式**，采用单线程避免不必要的内容开销和上下文切换开销。

基于事件驱动主要涉及两个问题：

- **CPU 利用率**。所有处理都是单线程的，影响时间驱动服务模型的根本在于 CPU 的计算能力，因此如何利用多核 CPU ？
- **进程的健壮性**。PHP 中没有线程的概念，它的健壮性是每次请求都建立独立的上下文（线程），而对于 nodejs 所有请求都是统一的上下文，健壮性必须保证（即能自动修复、容错）。

另外，**考虑 nodejs 的特殊情况（只能使用部分内存，64 位最多使用 1.4GB ，32 位最多使用 0.7GB），开启多进程还可充分利用内存**。因为每个进程都是一个单独的 v8 实例，会重新分配内存，和其他进程不冲突。

### 创建子进程

[child_process](http://nodejs.cn/api/child_process.html) 提供了创建子进程的方法

- `spawn`
- `exec`
- `execFile`
- `fork`

```js
var cp = require('child_process')
cp.spawn('node', ['worker.js'])
cp.exec('node worker.js', function (err, stdout, stderr) {
    // todo
})
cp.execFile('worker.js', function (err, stdout, stderr) {
    // todo
})
cp.fork('./worker.js')
```

进程之间的通讯，代码如下。跟前端`WebWorker`类似，使用`on`监听，使用`send`发送。

```js
// parent.js
var cp = require('child_process')
var n = cp.for('./sub.js')
n.on('message', function (m) {
    console.log('PARENT got message: ' + m)
})
n.send({hello: 'workd'})

// sub.js
process.on('message', function (m) {
    console.log('CHILD got message: ' + m)
})
process.send({foo: 'bar'})
```

### 句柄传递

书中通过一个场景很直接的引出了这个问题，干净利落。即：使用上面的`fork`来启动多个进程监听 http 请求，这样会报监听端口冲突的问题。例如，一旦启动了一个进程监听 8080 端口，再想启动另一个进程监听 8080 端口的话，就会报错。那该如何解决这个问题？

**node 从 v0.5.9 开始引入了进程之间发送句柄的功能**，`child.send(message, [sendHandle])`，后面那个可选的参数就是句柄。句柄可以是一个服务端 socket 对象、一个客户端 socket 对象，一个 UDP 套接字，一个管道等……（其实我没太看懂这里……）。具体的代码示例，可参考书中 243 页。

其实，这个问题通过后面的 Cluster 即可轻松解决。但是 **Cluster 也是借用的这里的句柄传递**，因此也要了解一点原理，虽然不会直接用。

### 稳定的集群

基本就是使用 Cluster ，cluster 模块允许设立一个主进程和若干个 worker 进程，由主进程监控和协调 worker 进程的运行。worker 之间采用进程间通信交换消息，**cluster模块内置一个负载均衡器，采用 Round-robin 算法协调各个 worker 进程之间的负载**。运行时，所有新建立的链接都由主进程完成，然后主进程再把 TCP 连接分配给指定的 worker 进程（即上文的句柄传递）。

```js
const cluster = require('cluster')
const os = require('os')
const http = require('http')

if (cluster.isMaster) {
    console.log('是主进程')
    const cpus = os.cpus() // cpu 信息
    const cpusLength = cpus.length  // cpu 核数
    for (let i = 0; i < cpusLength; i++) {
        // fork() 方法用于新建一个 worker 进程，上下文都复制主进程。只有主进程才能调用这个方法
        // 该方法返回一个 worker 对象。
        cluster.fork()
    }
} else {
    console.log('不是主进程')
    // 运行该 demo 之后，可以运行 top 命令看下 node 的进程数量
    // 如果电脑是 4 核 CPU ，会生成 4 个子进程，另外还有 1 个主进程，一共 5 个 node 进程
    // 其中， 4 个子进程受理 http-server
    http.createServer((req, res) => {
        res.writeHead(200)
        res.end('hello world')
    }).listen(8000)  // 注意，这里就不会有端口冲突的问题了！！！
}
```

维护进程健壮性，**通过 Cluster 能监听到进程退出，然后自动重启，即自动容错**。

```js
if (cluster.isMaster) {
    const num = os.cpus().length
    console.log('Master cluster setting up ' + num + ' workers...')
    for (let i = 0; i < num; i++) {
        // 按照 CPU 核数，创建 N 个子进程
        cluster.fork()
    }
    cluster.on('online', worker => {
        // 监听 workder 进程上线（启动）
        console.log('worker ' + worker.process.pid + ' is online')
    })
    cluster.on('exit', (worker, code, signal) => {
        // 兼容 workder 进程退出
        console.log('worker ' + worker.process.pid + ' exited with code: ' + code + ' and signal: ' + signal)
        // 退出一个，即可立即重启一个
        console.log('starting a new workder')
        cluster.fork()
    })
}
```

示例看似简单，但是实际应用还是尽量使用成熟的工具，例如 [pm2](https://www.npmjs.com/package/pm2)

----

## 第十章 测试

自动化单元测试，将单独整理，此处略过。

----

## 第十一章 产品化

按书中写的内容，我重新总结，应该分位三个层面:

- 流量处理：负载均衡，CDN 等
- 稳定监控：日志统计、监控内存 CPU、报警邮件等
- 上线部署：构建、部署、上线

-----

## 扩展 & 遗留问题

- linux 标准输入输出
- 异步流程控制库 [async](https://github.com/caolan/async) 和 [step](https://www.npmjs.com/package/step)
- 异步并发控制的过载保护 [bagpipe](https://www.npmjs.com/package/bagpipe) （本书作者提供）
- stream-handbook [英文原文](https://github.com/substack/stream-handbook) [中文翻译](https://github.com/jabez128/stream-handbook)
- [《汇编语言入门 阮一峰》](https://news.cnblogs.com/n/587863/) 
- nodejs 接收文件上传（并写入系统临时文件夹）[formidable](https://www.npmjs.com/package/formidable)
- nodejs 进程守候工具 [pm2](https://www.npmjs.com/package/pm2)
