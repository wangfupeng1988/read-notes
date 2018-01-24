# 《第三方 javascript 编程》读书笔记

这并不是一本新书，阅读这本书的原因是由于我在工作中维护了几个 JSSDK ，感觉自己应该根据自身的精力去拜读一下这本书，毕竟本书的作者是大名鼎鼎的 [disqus](http://www.disqus.com) 工程师。disqus 在 2012 年 pv 就 50亿/月 ，可以说是世界上最具有技术话语权的 JSSDK。

本书讲解了开发一个外部使用的 JSSDK 将遇到的问题和解决方案。相比于 disqus 的实际情况，本书肯定只是简单的汇总了个“皮毛”，但是这对于 JSSDK 这一小众的技术方向，完全可以看成一个标杆性的书籍来阅读。

根据本书介绍，开发第三方使用的 JSSDK 面临的共性问题有：

- 应用的分发和加载
- HTML 和 CSS 的渲染
- 与服务器通讯
- 跨域 iframe 通讯
- 验证和会话

书中其他的内容并不特殊，即并不是 JSSDK 特有，属于共性问题，也没什么难点，就此略过。另外，除了专门针对 SDK 以外，想要了解跨域、iframe 相关知识的同学，也应该阅读本书。

----

## 应用的分发和加载

读来总结以下要点。

### 无阻塞加载脚本

使用`defer`（html4 标准）和`async`（html5 标准）都可以无阻塞（不阻塞页面的加载）加载脚本，例如`<script defer src="xxxx"></script>`。**但是首推使用`async`，因为这样会是 js 的执行更快。** 总结一下两者的异同：

- 相同：不阻塞页面的加载
- 不同：`async`下载完立刻执行，`defer`加载的 js 要等待页面加载、解析完之后再顺序执行

**注意，使用`async`， js 执行如果有 DOM 操作，需待 DOM Ready 之后。**

```html
<!--无阻塞加载脚本-->
<script async src="xxxx"></script>
```

### 动态脚本插入

即“小拖大”、“静拖动”，直接看代码即可。这份代码在实际工作中用到了可直接拷贝过来使用，因此先整理总结下来。

```html
<script>
function loadScript(url, callback) {
    var script = document.creaateElement('script')
    script.async = true // 无阻塞加载，上文刚说过
    script.src = url

    var entry = document.getElementsByTagName('script')[0]  // 这段代码就在 <script> 中，因此肯定至少有一个 <script> 标签
    entry.parentNode.insertBefore(script, entry)

    script.onload = script.onreadystatechange = function () {
        var readyState = script.readyState
        if (!readyState || /complete|loaded/.test(script.readyState)) {
            // 加载完毕，执行回调
            callback()

            // 清理内存
            script.onload = null
            script.onreadystatechange = null
        }
    }
}
</script>
```

### 参数传递

书中总结了好几种，但是我觉得比较可用的就是两种：第一，传递 url 参数；第二，初始化时配置。

传递 url 参数例如加载 js 时直接在 url 中带有参数，例如`<script src="xxx.js?myapp-id=100"></script>`。这样，server 可以获取到`id`然后动态拼接 js 内容，不过这不是一个靠谱的方案，因为 js 一般都会作为静态文件放在 CDN 上，不会动态生成。还有一种方式就是前端识别

```js
function getMyAppId() {
    var scripts = document.getElementsByTagName('script')
    var el, src, i, length = scripts.length
    for (i = 0; i < length; i++) {
        el = scripts[i]
        src = el.src
        /*
        接下来，再分析 src 中是否有 myapp-id=100 格式的参数，即可
        */
    }
}
```

初始化配置就更加简单了，例如[微信 JSSDK](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115) 初始化的时候要传入`appid`一样。

----

## HTML 和 CSS 的渲染

### 定位 DOM

如何选中操作的 DOM 节点，书中介绍了一些常规方法，无非是通过 id 属性 class 等获取。还有一种是将`<script>`标签放在 DOM 节点里面（如果是单例的话），也可以，这部分没啥难点。

### 样式

针对渲染出来的 html 设置样式？可以使用内联方式（即用标签的 style 属性），但是样式代码过多的话就不推荐了。接下来，本文介绍了动态加载 css 文件的方式，跟动态加载 script 相似。**但是，这里没法和动态加载 script 一样，轻松判断是否加载完成**！文中指出了一个笨方法 —— **轮询页面，知道 css 文件中某个规则可用**。该方法小众怪异，且目前我没有应用场景，代码就不整理了，使用时可查阅本书 55 页代码。

如何防止 sdk 的 css 被父页面干扰？第一，个性化选择器名称，不重名；第二，提升选择器优先级；第三，使用 iframe 。无论使用哪种方式，都要遵循一个原则：**SDK 和父页面是共享 DOM 的，哪一方都不能为了自己的便捷而做出独占的行为**。

### iframe

为了不与外部页面的 css 冲突，可以将渲染内容放在 iframe 中。其实，用 iframe 也不一定非得是外链的页面，即不一定非得有`src`属性。JS 创建要给 iframe ，然后直接往里面写入内容即可

```js
var doc = iframe.contentWindow.document
doc.write(
    /* 要写入的 html 内容 */
)
doc.close()
```

以上代码**需要注意两点**：

- `document.write`是同步的，会阻塞。但是，iframe 是由浏览器异步处理的，这样`document.write`同步也仅仅是在 iframe 内部同步而已，不会影响父页面。
- 但是 iframe 仍然会阻塞父页面的 onload 事件。不过在 iframe 内部调用`document.close()`会强制触发其 onload 事件，这样就不会阻塞父页面的 onload 了。

接下来，书中提到了**样式继承，即通过 SDK 获取父页面最基本的 css 属性（字体、字号、背景色等），然后注入到 iframe 中**。为了能让 iframe 中的样式和父页面样式尽量保持一致，这是我一直都没有想过的一点。看书中代码，就是**通过`getComputedStyle`和`currentStyle`来获取样式的**，日常用不到就不整理了，在书中 70 页。

获取到基本样式之后，对于无`src`的 iframe 可直接`document.write`写入。**对于外链的 iframe ，可以将样式通过 url 参数传递，然后交给 iframe 内部的页面处理（前端或者 server 端）**，例如`<iframe src="xxx.html?font-size=20&color=%23333&font-style=italic">`这样。

### 选择

以上是书中提到的几种渲染 html 和 css 的方式，视实际情况选择一个使用

----

## 与服务器通讯

本章主要介绍跨域通讯的方式，JSSDK 被引用到外域的第三方页面，肯定是跨域通讯。

### 跨域资源共享

即现在 http 标准最新的跨域解决方案，server 端通过在 response header 中增加`Access-Control-Allow-Origin`。另外，如果想跨域传递 cookie ，客户端需要`xhr.withCredentials = true`，server 需要在 response header 中增加`Access-Control-Allow-Credentials: true`。

能通过标准来搞定的方案，就是最佳方案。

### JSONP

基于浏览器兼容性和历史代码原因， JSONP 还是现在最常用的解决方案，具体实现不过多赘述。书中提到了几个 JSONP 的问题，是我平时没有想过的：

- 仅能用于 Get 请求
- 缺乏错误处理机制。例如， JSONP 的请求如果返回 404 和 500 的话，就无法触发 JSONP 的 callback
- 借助 JSONP 可进行跨站点请求伪造（CSRF）。（书中讲的较少，没太看明白！！！）

### 子域名代理

如果这种跨域**仅仅是子域名跨域的关系，即父域名相同、协议相同、端口相同**，例如`a.com`和`sub.a.com`的关系，这样可以使用书中提到的子域名代理的技术。例如`a.com`的页面下有一个 iframe ，src 是`sub.a.com`的页面。

- 第一，需要在`a.com`页面上修改`document.domain = 'a.com'`
- 第二，需要在 iframe 的子页面中修改`document.domain = 'a.com'`，这样使两个页面都设置为共同的`document.domain`，这样就不受同源策略控制了。
- 第三，接下来即可在父页面进行跨域通信了，代码如下

```js
iframe.onload = function () {
    iframe.contentWindow.jQuery.ajax({
        url: 'http://sub.a.com',  // 注意，这里是 a.com 的页面发送 sub.a.com 的 ajax 请求，是跨域
        success: function (result) {

        }
    })
}
```

这种方式受限比较严重，因此应用场景比较少。

----

## 跨域 iframe 通讯

如果 iframe 是外链的页面，父页面无法通过访问到 iframe 页面的任何信息，通讯是一个问题。其实通讯问题也不仅仅在于父页面和 iframe 之间，也在于和通过`window.open`打开的页面之间。下面是一些相关代码

```js
// 父页面访问 iframe
var iframe = document.getElementById('my-iframe')
var win1 = iframe.documentWindow

// 从 iframe 中访问父页面
var win2 = window.parent

// 使用 window.open 打开页面
var win3 = window.open(...)
var win4 = window.opener
```

### postMessage

不过，以上这些情况的通讯，都可以通过 html5 标准中的`window.postMessage`来搞定。浏览器兼容性（可通过 [caniuse.com](https://caniuse.com/#search=postMessage) 来查看）可能会是一个问题，这里建议使用接下来要将的 [easyXDM](https://github.com/oyvindkinsey/easyXDM) 这个工具，工具有不兼容的降级方案。

使用 postMessage ，发送方写法如下

```js
// targetWindow 即接收方的 window 对象
// 第一个参数是要发送的内容，字符串格式
// 第二个参数是接收方的域
targetWindow.postMessage('xxxxx', 'http://target-doman.com/')
```

接收方的写法如下：

```js
// 在 window 上监听 message 方法
window.addEventListener('message', function (event) {
    event.data  // 发送过来的内容，字符串格式
    event.origin  // 发送方的域
    event.source  // 发送方的 window 对象
})
```

### easyXDM

如果浏览器不支持 postMessage ，降级方案书中提到了三个：

- 使用`window.name`
- 使用 url hash
- 使用 flash 

这些我都没有仔细看，**因为书中最后提到了使用 [easyXDM](https://github.com/oyvindkinsey/easyXDM) 工具，对各种方案就行了封装**。看 github 的提交记录，这是一个很老牌的工具了，又得作者推荐，稳定性应该没问题。文档先不看，后续有使用场景再看不迟。

----

## 验证和会话

- 介绍第三方 cookie
- 在第三方应用中验证和保持会话
- 禁用第三方 cookie 的解决方案
- 防止会话劫持的技术

本章没太看明白。第一，关于验证、会话、cookie 这些比较专业的事情，大公司一般都是有专门的团队负责，cookie 不是每个人都能修改的，登录也有专门的 SDK 和接口；第二，作者最后总结的时候也承认，该部分确实很难，针对不同浏览器进行不同的适配，可见针对这块领域目前还没有标准。

这块属于小众、专业的一部分，看不明白就先放着，遇到类似问题再来研究。这是本书第六章：验证和会话

----

## 扩展 & 遗留问题

- 跨域消息通讯工具 [easyXDM](https://github.com/oyvindkinsey/easyXDM)
- 遗留问题：如何借助 JSONP 进行 CSRF ？
- 遗留问题：本书第六章（验证和会话）没看明白

----

## 补充

### 上线前如何全面回归测试

我工作中做 SDK 主要是给公司内部的各个业务方来用，并不是给第三方。因此，每次修改 SDK 之后，如何回归测试是一个很蛋疼的事情。例如，这个 SDK 目前有 10 个接入方，按理说，每次需改之后上线之前，回归测试都得让这 10 个业务方的测试人员都参与进来，测试各自的相关功能有没有 bug 。

但是这是一个非常蛋疼的事情，特别是在大公司里。大家都号称很忙（无论是真的还是装的），想申请一个测试人力是非常难的，更别说同时申请 10 个业务方的测试人力。目前还没有简单有效的解决方案。
