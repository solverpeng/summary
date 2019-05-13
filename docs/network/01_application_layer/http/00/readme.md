# 超文本传输协议（HTTP）

HTTP是用于传输诸如HTML的超媒体文档的[应用层](https://en.wikipedia.org/wiki/Application_Layer)协议。被设计用于Web浏览器和Web服务器之间的通信。HTTP是[无状态协议](http://en.wikipedia.org/wiki/Stateless_protocol)，意味着服务器不会在两个请求之间保留任何数据（状态）。虽然通常基于TCP / IP层，但可以在任何可靠的[传输层](https://zh.wikipedia.org/wiki/传输层)上使用; 也就是说，一个不会静默丢失消息的协议，如UDP。

## HTTP请求

HTTP请求报文由3部分组成：请求行+请求头+空行+请求体。如下：

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/network/%E5%BA%94%E7%94%A8%E5%B1%82/http%E8%AF%B7%E6%B1%822.png)

### 请求行

由请求方法、请求URI、请求协议及版本组成了请求行。

### 请求头

#### Accept

用来告知服务器，客户端可以处理的内容类型，内容类型用[MIME类型](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)来表示。服务器可以从诸多备选项中选择一项进行应用，使用`Contnet-Type`响应头通知客户端它的选择。

##### 语法

```xml
Accept: <MIME_type>/<MIME_subtype>
Accept: <MIME_type>/*
Accept: */*

// Multiple types, weighted with the quality value syntax:
Accept: text/html, application/xhtml+xml, application/xml;q=0.9, */*;q=0.8
```

##### 指令

```xml
<MIME_type>/<MIME_subtype>：单一精确的 MIME 类型, 例如text/html
<MIME_type>/*             ：一类 MIME 类型, 但是没有指明子类
*/*                       ：任意类型的 MIME 类型

;q=						  ：权重
```

##### 示例

```html
Accept: text/html
Accept: image/*
Accept: text/html, application/xhtml+xml, application/xml;q=0.9, */*;q=0.8
```
#### Keep-Alive

是一个通用的消息头，允许消息的发送者暗示连接的状态，还可以用来设置超时时长和最大请求数。

> 需要将`Connection`的值设置为“keep-alive”这个消息头才有意义。在HTTP/2.0协议中，`Connection`和`Keep-Alive`被忽略，采用其他机制来管理连接。

##### 指令

```
timeout：一个空闲连接保持打开状态的最小时长，以秒为单位
max    ：在连接关闭前，此连接可以发送的请求的最大值。
```

##### 示例

```
Keep-Alive: timeout=5, max=1000
```

#### Connection

决定当前的事务完成后，是否会关闭网络连接。如果该值是“keep-alive”，表示一个长连接，网络连接就是持久的，不会关闭，使得对同一个服务器的请求可以继续在该连接上完成。

#### Content-Type

告诉服务器实际发送的数据类型。

##### 语法

```xml
Content-Type: text/html; charset=utf-8
Content-Type: multipart/form-data; boundary=something
```

##### 指令

```xml
media-type ：MIME类型
charset    ：字符编码标准
boundary   ：包括来自一组字符的1到70个字符，用于封装消息的多个部分的边界
```

#### Accept-Encoding

告诉服务器客户端可以理解的内容编码方式。服务器会选择一个客户端提议的方式，并在响应头通过`Content-Encoding`通知客户端它的选择。

#### Content-Encoding

对特定媒体类型的数据进行压缩。它的值表示消息主体进行了何种方式的内容编码转换。

#### Content-Length

用来指明发送给接收方的消息体的大小，用十进制数字表示的八位字节的数目。

#### Accept-Language

告诉服务器客户端可以理解的自然语言。服务器可以使用`Content-Language`响应头通知客户端它的选择。浏览器会基于用户界面语言来为这个请求头设置合适的值，用户尽量不要去修改它。

----

#### Host

指明了服务器的域名以及服务器监听的TCP端口号。如果没有给定端口号，会自动使用被请求服务器的默认端口。HTTP/1.1所有请求头中必须包含一个HOST头字段，若缺少或超出一个HOST头字段，会返回一个400的状态码。

#### Referer

表示当前请求页面的来源页面地址

#### User-Agent

标识发起请求的客户端软件应用类型、操作系统、软件开发商以及版本号。

##### 语法

```
Referer: <url>
```

#### Cache-Control

用于在http 请求和响应中，通过指定指令来实现缓存机制。缓存指令是单向的, 这意味着在请求设置的指令，在响应中不一定包含相同的指令。

## HTTP响应

HTTP响应也由四个部分组成，分别是：状态行、消息报头、空行和响应正文。

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/network/%E5%BA%94%E7%94%A8%E5%B1%82/httpmessage.jpg)

### HTTP响应头

#### Allow

服务器支持哪些请求方法（如GET、POST等）。

#### Server

服务器名字。Servlet一般不设置这个值，而是由Web服务器自己设置。

#### Expires

应该在什么时候认为文档已经过期，从而不再缓存它。

#### Refresh

表示浏览器应该在多少时间之后刷新文档，以秒计。

#### Set-Cookie

设置和页面关联的Cookie。

#### Content-Type

告诉客户端实际返回的内容的内容类型。

#### Content-Encoding

用来告知客户端应该如何解码才能获取在`Content-Type`中标示的媒体类型。



[菜鸟教程](https://www.runoob.com/http/http-messages.html)

[MDN web docs](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)