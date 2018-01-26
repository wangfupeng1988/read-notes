# 《深入浅出 nodejs》读书笔记

本书购于 2015.9.30 ，至今快两年半的时间了。购书当时已经看完，但是现在再回头翻看，依然觉得很有收获。因为本书讲解的内容较为底层、原理，并且 nodejs 的基础的 API 一直也没有变过，因此就不过时。正好近期想写一篇关于 nodejs 的系列博客，正好拿本书做参考，重读并做读书笔记。

本书作者是阿里资深工程师仆灵，nodejs 步道师。书中前言部分就提到，作者作为一名 FE 转型成为 nodejs 后端工程师，所具有的优势仅仅是对 JS 语法熟悉而已，但是对后端开发的其他知识并不擅长，需要重新学习。我想这也是我等 FE 对 nodejs 的一个看不透彻的地方，总以为会了 JS 就很容易学会 nodejs 。

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

待续……





-----

## 扩展 & 遗留问题

- linux 标准输入输出
- 异步流程控制库 [async](https://github.com/caolan/async) 和 [step](https://www.npmjs.com/package/step)
- 异步定法控制的过载保护 [bagpipe](https://www.npmjs.com/package/bagpipe) （本书作者提供）

