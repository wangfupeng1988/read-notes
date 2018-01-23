# 《第三方 javascript 编程》读书笔记

这并不是一本新书，阅读这本书的原因是由于我在工作中维护了几个 JSSDK ，感觉自己应该根据自身的精力去拜读一下这本书，毕竟本书的作者是大名鼎鼎的 [disqus](http://www.disqus.com) 工程师。disqus 在 2012 年 pv 就 50亿/月 ，可以说是世界上最具有技术话语权的 JSSDK。

本书讲解了开发一个外部使用的 JSSDK 将遇到的问题和解决方案。相比于 disqus 的实际情况，本书肯定只是简单的汇总了个“皮毛”，但是这对于 JSSDK 这一小众的技术方向，完全可以看成一个标杆性的书籍来阅读。

根据本书介绍，开发第三方使用的 JSSDK 面临的共性问题有：

- 应用的分发和加载
- HTML 和 CSS 的渲染
- 与服务器通讯
- 跨域 iframe 通讯
- 第三方 cookie
- 安全性

书中其他的内容并不特殊，即并不是 JSSDK 特有，属于共性问题，也没什么难点，就此略过。

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



----

## 与服务器通讯

----

## 跨域 iframe 通讯

----

## 第三方 cookie

----

## 安全性

----

## 扩展 & 遗留问题

- 跨域消息通讯工具 [easyXDM](https://github.com/oyvindkinsey/easyXDM)
