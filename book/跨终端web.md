# 《跨终端web》读书笔记

本书购于 2016.8.3 ，已经早早读完放在书架上。现在再拿出来重新翻看一下当时记录的重点内容，做读书笔记。

本书前两章主要以介绍为主，引出跨终端 web 和 Mobile Web 的一些概念。然后在接下来的很大篇幅中，讲了这样一个**开发过程**

- 第一，基准
- 第二，检测
- 第三，接口
- 第四，定位
- 第五，预览

篇幅很多估计这部分也很重要，但是我就是没看明白它想表达的什么意思（现在翻看也是没看懂）。连个介绍、总结都没有，上来就从“基准”开始讲。至于看不懂的原因，我觉得可能是作者表达的过于抽象，没有找到一个适合落地的例子。

**不过本书还是推荐购买的，因为书中有一章讲到了 hybrid** （对，只要书中有一章是高价值的，就值得购买，不要指望书中全部都是干货）。这也是驼子里拔将军，因为专门讲解 hybrid 的书籍市面上貌似还没有。（PS：慕课网有一个 [hybrid 设计](https://www.imooc.com/learn/850) 的视频课程，免费的，值得推荐）

下面总结一下我看本书觉得比较重要的内容：viewport 和 hybrid

----

## viewport

viewport 是移动 web 发展壮大之后，被重新提及重视的一个新概念，内容还挺复杂，想详细了解也不容易，书中推荐阅读《viewport 双城记》（最后有链接）。这些资料我也都阅读过，书中提到的重点也都看过，但是后来还是忘记了大部分。

重要内容之一，移动端网页 html 代码要加

```html
<meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no">
```

另外，和 viewport 比较相关的（书中没有讲到的）还有。就是视觉设计同事给图的时候，一定要让它给尺寸宽度是`1242`（对应三倍屏，如 iphone7p）或者`750`（对应两倍屏，如 iphone7）的图，这样才能将图中标注的尺寸轻松计算出来，写到 css 代码中。

----

## hybrid

书中第 8 章讲 hybrid ，也是我觉得书中最值得看的内容。关于 hybrid 的一些介绍、历史、优点缺点等就不在赘述，自己看书。**最重要的是 Native 和 Web 的双向通讯机制**。

### Native 调用 Web

双端都比较统一，就是直接在 NA webview 中执行 JS 全局的函数，具体的客户端的 API 不必关心。

### Web 调用 Native

书中介绍，安卓有以下常见方式：

- 重写`WebViewClient.shouldOverrideUrlLoading`，即劫持 url 变化。可将 web 调用 NA 的数据封装到 url 中。
- 重写`WebChromeClient.onJsPrompt`（或`JsConfirm` `JsAlert`），即劫持 prompt 弹框。可将 web 调用 NA 的数据封装到 prompt 中。

ios 只有一种常用方式：

- 类似安卓的第一种方式，劫持 url 变化。

### Bridge

所谓的 Bridge 其实就是统一封装了 Native 和 Web 相互通讯的 API ，Native 也能调用，web 也能调用。例如封装成一个`invoke.js`文件，将其内置到客户端中，webview 加载页面时默认运行`invoke.js`，即 web 就默认可用其中的 API 了。

封装不是问题，关键是上面提到的调用方式，书中 149 页列出了封装的代码示例。

### 实际工作中使用 schema 方式

我实际工作中，Native 和 web 通讯都是用 schema 方式，通讯的原理就是 webview 劫持 url 变化。只不过这个 url 是一种 schema 协议的地址，而非 http 协议的地址。百度搜索“微信 schema”，可看到什么是 schema 协议的地址。

-----分割线-----

最后书中介绍了 hybrid 的框架 [PhoneGap](https://github.com/sintaxi/phonegap) ，没了解过这个框架，也暂时没有这个需求。

-----

## PS：扩展

**关于 hybrid 和 RN/WEEX** ：

慕课网 [hybrid 设计](https://www.imooc.com/learn/850) 中讲过，RN/WEEX 不是一般公司能玩得转的。需要大量的人力财力和时间，而且需要一个精通客户端和前端的架构师来带头做，就这样，能做好也非短期实现的。阿里的 weex 都玩了近两年了，还是有各种问题需要优化，并且需要前端和客户端配合完成。

不可否认，RN/WEEX 比 hybrid 要快、体验要好，而且它们肯定是未来的替代方案，是未来的发展方向。但是，从目前上处于研究而非稳定状态的情况下，还是要谨慎使用。

再者，对比来说 hybrid 速度慢、体验差，但是我觉得还是能被用户所接受的。

----

## 扩展 & 遗留问题

- 《viewport 双城记》 [part one](https://www.quirksmode.org/mobile/viewports.html), [part two](https://www.quirksmode.org/mobile/viewports2.html), [中文翻译](http://ju.outofmemory.cn/entry/73169)
- hybrid 框架 [PhoneGap](https://github.com/sintaxi/phonegap) 和 [ionic](https://ionicframework.com/)
- 慕课网视频教程 [hybrid 设计](https://www.imooc.com/learn/850) 
- 慕课网视频教程 [When iOS loves JS](https://www.imooc.com/learn/92)
