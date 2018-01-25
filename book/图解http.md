# 《图解 http 》读书笔记

看书中记录是 2015.7.10 购买的，那会儿我还没来北京，那本书就是专门从济南带到北京来的，说明这本书比较重要。书的内容在当初买了就很详细的看过，那会儿我看书不像现在这么快，很仔细。近期准备写一个 nodejs 的系列博客，其中会涉及到很多 http 的知识，因此再翻看一遍，顺便记录读书笔记。

本书是一位日本人写的，我很喜欢这本书的表达方式 —— 突出重点、浅显易懂！很符合我经常说的 2/8 原则。如果感觉看着不过瘾，还可以去阅读更加高大上的《 HTTP 权威指南》，厚厚的枕头书。不过，我感觉作为一名前端程序猿，仔细看看这本书就够了。

----

## 什么是 http 协议

网络（包括互联网）都是建立在 **TCP/IP 协议族** 基础上的，其中常见的具体协议有 http ftp ssh dns TCP IP等，http 协议仅仅是其中的一个子集。先了解出处和来源。

### 三次握手

书中介绍了 **TCP 协议的“三次握手”**。详细内容可参见网络资料，用简单的话语总结就是：

- 第一次，客户端发送一些数据到 server 端，要求建立连接
- 第二次，server 端收到，然后发送一些数据回去，表明可以建立连接
- 第三次，客户端收到并检查返回的信息，若信息正确，再发送数据到 server 端，告知建立连接成功

PS：**http 1.1 版本开始支持持久连接`Connection: keep-alive`**，只要连接双方不明确终点连接，连接就会持续保持，即每次发送数据无需再进行“三次握手”

### 临时想到的

> 按照 TCP/IP 协议族的层次划分，TCP 位于传输层，提供可靠的字节流服务。所谓字**节流服务（Byte Stream Service）**是指，为了方便传输，将大块数据分割成以报文段（segment）为代为的数据包就行管理。

读到书中这段文字（12页），想到了一个关于数据**结构化和非结构化**的话题，需要分开来说：

- **存储 & 传输：非结构化数据**，例如上文提到的传输过程中的字节流服务（在 nodejs 中的 Stream 会有更贴切的体现），再如存储为文件的文本、图片、视频等。非结构化的数据无法用户复杂计算。
- **计算：结构化数据**，进行复杂的逻辑运算，前提是需要结构化的数据，因此学习算法之前要先学习数据结构。这样就涉及到非结构和结构化相互转换的一个问题，最简单的示例就是`JSON.parse`和`JSON.stringify`

----

## http 报文构成内容

<img src="https://camo.githubusercontent.com/e8c3eac293061d79cfda2a1b94d638fdc03118c4/68747470733a2f2f696d61676573323031372e636e626c6f67732e636f6d2f626c6f672f3133383031322f3230313830312f3133383031322d32303138303132343137323033393935392d34333539353632372e706e67" style="width:500px"/>

上图是 request 的报文图解，内容有：

- **Method**
- URI
- 协议版本
- **Head 头部字段**
- 内容实体

<img src="https://camo.githubusercontent.com/bf100a806db4631da3db00139c15613953cdf24c/68747470733a2f2f696d61676573323031372e636e626c6f67732e636f6d2f626c6f672f3133383031322f3230313830312f3133383031322d32303138303132343137323034383439302d313535343832323530332e706e67" style="width:500px"/>

上图是 response 的报文图解，内容有：

- 协议版本
- **状态码**
- 状态码的原因短语
- **Head 头部字段**
- 主体内容

PS：报文内容中，头部内容和主体内容是通过 **空行（CR+LF）** 区分开来的。

----

## Method

> 和 Method 比较相关的新概念是 RESTful API ，可去查阅相关的资料，本文不展开。

常用的 Method 有两种：

- `GET` 获取信息
- `POST` 传输信息

其他 Method

- `PUT` 传输文件。自身不带验证机制，存在安全问题，一般的 web 网站不推荐使用
- `HEAD` 获取 Head 信息。不返回报文主体。用于确认 URI 有效性即资源更新的日期时间等
- `DELETE` 删除文件。跟`PUT`一样，不安全

还有 `OPTIONS` `TRACE` `CONNECT` 等，不常用。

----

## http 状态码

- `1xx` 请求正在处理
- `2xx` 成功
- `3xx` 重定向
- `4xx` 客户端错误
- `5xx` 服务端错误

以上是状态码的分类。书中提到，全部的状态码总共有 60 多种，但实际常用的就 14 种（其实常用的不到 14 种）。下面分类别简述：

### `1xx`

无

### `2xx`

- `200` 成功
- `204` 成功，但没有返回实体。一般用于只往客户端发送信息，而不需要客户端返回内容的情况（不常见）。
- `206` 范围请求，不是全部。用 Head 中的`Content-Range`来指定范围。

### `3xx`

- `301` 永久重定向。如`http://xxx.com`这个 Get 请求（最后没有`/`），就会被`301`到`http://xxx.com/`（最后是`/`）
- `302` 临时从定向。临时的，不是永久的。
- `304` 资源找到但是不符合请求条件，不会返回任何主体。如发送 GET 请求时，head 中有`If-Modified-Since: xxx`（要求返回更新时间是`xxx`时间之后的资源），如果此时 server 端资源未更新，则会返回`304`，即不符合要求。

### `4xx`

- `401` 请求需认证，登录
- `403` 被拒绝，例如外域图片盗链
- `404` 请求资源未找到

### `5xx`

- `500` 服务器执行请求期间发生错误，如程序出现 bug
- `503` 超负载或者维护停机

----

## http head

分类：

- 通用首部字段（即请求和相应都会用到）
- 请求首部字段
- 响应首部字段
- 实体首部字段

下面按照分类介绍一些常用的字段，不常用的略过。

### 通用首部字段（即请求和相应都会用到）

- **`Cache-Control` 缓存控制**

字段值非常多。工作中一个例子，加载 CDN 一个 JS 的时候，request 时候`Cache-Control:no-cache`，response 时候`Cache-Control:max-age=3600`（资源缓存周期是 1h）

- **`Connection`**

一般用于管理持久连接，request 和 response 都是`Connection: keep-alive`

- **`Date` 时间**

即 request 和 response 的时间。记得很早之前有人问到，像那种定时秒杀抢购这种场景，肯定需要用服务器端时间，**那么客户端如何以最小的代价获取服务器端时间？** 答案就是用这里的`Date`字段，如果再弄一个 API 专门返回时间，那就慢很多了。

### 请求首部字段

客户端发出 http 请求时，这些信息将被 server 端接收。

- **`Accept` 可接收类型**

客户端接收的媒体类型，如加载 html 时`Accept: text/html,application/xhtml+xml,application/xml`，加载图片时`Accept: image/webp,image/apng,image/*,*/*;q=0.8`，无要求就直接`Accept:*/*`。

上面的这些可选值，如`text/html` `image/webp` 都是标准的 MIME 类型，全部类型参考 [这里](http://www.w3school.com.cn/media/media_mimeref.asp) 或者网上搜索“MIME 类型”。

- **`Accept-Encoding`**

如`Accept-Encoding: gzip, deflate, br`，其中`gzip`是常用的压缩格式，表明客户端支持`gzip`压缩。`gzip`压缩能记得缩减资源大小，以 JS 为例，可压缩至原大小的 1/3 左右。

- **`Accept-Language` 语言**

如`Accept-Language:zh-CN,zh;q=0.9,en;q=0.8,und;q=0.7`，优先中文、英文次之，在多语言系统中可用。

- **`Host` 域**

如`Host: m.baidu.com`，包括域和端口号，不包括协议。

- **`If-Modified-Since`**

`If-Modified-Since: xxx`即要求资源必须是`xxx`时间之后修改过的，返回`200`。如果是找到资源，但是资源修改时间是`xxx`时间之前的，返回`304`（没有任何实体内容）

- **`Referer` 来源**

如`Referer:https://m.baidu.com/`，即该请求来自于哪里。

- **`Range` 范围**

如`Range: bytes:5001-10000`

- **`User-Agent` UA**

每次 http 请求，都将 UA 传递到 server 。注意，**UA 没有跨域限制，请求第三方 http ，也会传递 UA** 。

- **`Cookie`**

每次 http 请求，都将 cookie 传递给 sever 。注意，**cookie 有跨域限制，只有同源才会传递 cookie**  。

### 响应首部字段

server 端返回 http 请求时，这些信息将被客户端接收。

- **`Age` 缓存持续时长**

如`Age: 300`，即说明该资源缓存是在 300s 之前创建的。可以拿来和`Cache-Control:max-age=3600`进行对比。

- **`ETag`**

如`ETag: xxx`。`ETag`是服务端为每个资源分配的唯一的值，资源更新`ETag`也会更新。`ETag`并没有统一的算法，由 server 端自行控制。

`ETag`会用于 request Head 中的`If-Match`字段，不过我在实际工作中尚未用过该字段。

- **`Location` 重定向地址**

当返回`3xx`状体码时，要配合返回重定向的地址。

- **`Server` 服务信息**

如`Server: Apache`，可能是服务器名称，可能是名称 + 版本号，也可能是自定义的某个值

- **`Set-Cookie`**

是本次 http 请求中 server 端设置的 cookie 信息。除了 cookie 内容本身之外，还有`expires` `patch` `domain` `Secure`（仅用于 https） `HttpOnly`（JS 不能访问）这些信息。

如本次请求是申请登录，如果登录成功的话，server 端肯定要使用`Set-Cookie`在 cookie 中增加`sessionid=xxxx`，并且使用`HttpOnly`和`expires`。

注意，**设置`domain`可能会造成主域名的 cookie 污染**。如果指定`domain=example.com`时，`sub.example.com`的页面也会有效。

### 实体首部字段

request 和 response 中，实体部分所使用的首部字段。

- **`Content-Encoding`**

如`Content-Encoding: gzip`，实体使用 gzip 压缩。

- **`Content-Length`**

如`Content-Length: 967`，实体部分的大小，单位是 字节 。

- **`Content-Type`**

如`Content-Type: application/x-javascript`，其他信息可参考`Accept`字段。

- **`Expires`**

如`Expires: Thu, 22 Feb 2018 03:03:12 GMT`，即该资源即将失效的时间。

----

## https

http 请求是明文传输，因此内容被窃听是分分钟的事，因此要谨慎使用那些免费的 wifi 和 vpn 等。解决方式就是使用 https 。https 传输的数据是经过加密保护的，没有正式将无法被阅读，这也给 debug 时候抓包带来了一定成本。

**`HTTP + 加密 + 认证 + 完整性保护 = HTTPS`**

日常工作中，https 的配置和运维都由 server 端专门的运维人员来做，因此不再详细展开，先明白原理即可。

----

## 扩展 & 遗留问题

- [MIME 类型列表](http://www.w3school.com.cn/media/media_mimeref.asp)
