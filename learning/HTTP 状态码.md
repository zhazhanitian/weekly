## 前言

本文从以下几个方面，循序渐进走进 HTTP 状态码

- 状态码用来做什么的
- 常见状态码有哪些
- 容易争论的点

<br >

<br >

## 状态码用来做什么的

HTTP 状态行中使用状态码（Status Code）和原因短语（Reason Phrase）来简单描述请求的结果

- Version：版本号，例如 HTTP/2
- Reason：是状态码的简短文字描述，例如“OK”“Not Found”等等，也可以自定义，它其实对状态码的解释说明
- Status Code：状态码，表示服务器对请求的处理结果

这里我们重点介绍状态码，状态码是用以表示 HTTP 响应状态的 3 位数字代码，由RFC 2616规范定义。 **合理的状态码不仅可以让用户或者浏览器做出更加合适的进一步操作（例如继续发送请求、切换协议，重定向跳转等），而且可以让客户端代码更加易于理解和维护** 

RFC 把状态码分成五类，分别是：

- 1××： 请求已被接受正被处理，表示目前是协议处理的中间状态，还需要后续的操作
- 2××： 请求成功处理，报文已经收到并被正确处理
- 3××： 代表需要客户端采取进一步的操作才能完成请求，例如重定向，通常，这些状态码用来重定向，后续的请求地址（重定向目标）在本次响应的Location域中指明
- 4××： 客户端错误，请求报文有误，服务器无法处理
- 5××： 服务器错误，服务器在处理请求时内部发生了错误

<br >

<br >

## 常见状态码有哪些

### 1xx

1xx 是很陌生的，代表请求已被接受，需要继续处理。这类响应是临时响应，标示客户应该等待服务器采取进一步行动

我们最常见的是 101（Switching Protocols）

<br >

#### 101（Switching Protocols）

在 http header怎么判断协议是不是websocket 我们提到过，http 发送请求给服务器，服务器通过判断 header 中是否包含 Connection: Upgrade 与 Upgrade: websocket 来判断当前协议是否要升级到 websocket ，如果服务器同意进行 WebSocket 连接时，返回响应码 101

<br >

### 2xx

表示请求已成功被服务器接收、理解、并接受

常见的有

- 200
- 204
- 206

<br >

#### 200（OK）

最常见的，表示请求成功

<br >

#### 204（NO Content）

与 200 基本相同，但响应头后没有 body 数据

<br >

#### 206（Partial Content）

分片传输，每次只返回了请求资源的 **部分** ，常用于实现断点续传或者将一个大文档分解为多个下载段同时下载

请求头中包含 Range 字段时，响应需要只返回 Range 指定的那一段。响应中应包含 Content-Range 来指示返回内容的范围

例如:

```js
'Range':byte=5001-10000 
// 表示本次要请求资源的5001-10000字节的部分
```

这种情况下，如果服务器接受范围请求并且成功处理，就会返回 206 ,并且在响应的头部返回

```js
'Content-Range':bytes 5001-10000/10000 
// 表示整个资源有10000字节，本次返回的范围为 5001-10000字节
```

<br >

### 3xx

这类状态码代表需要客户端采取进一步的操作才能完成请求。通常，这些状态码用来重定向， 重定向目标在本次响应的 Location 头字段中指明

主要有以下 9 种状态码

| 状态码 |      状态短语      |                           状态含义                           |
| :----: | :----------------: | :----------------------------------------------------------: |
|  300   |  Multiple Choices  | 当请求的 URL 对应有多个资源时（如同一个 HTML 的不同语言的版本），返回这个代码时，可以返回一个可选列表，这样用户可以自行选择。通过 Location 头字段可以自定首选内容 |
|  301   |  Moved Permanetly  | 当前请求的资源已被移除时使用，响应的 Location 头字段会提供资源现在的 URL。直接使用 GET 方法发起新情求 |
|  302   |       Found        | 与 301 类似，但客户端只应该将 Location 返回的 URL 当做临时资源来使用，将来请求时，还是用老的 URL。直接使用 GET 方法发起新情求 |
|  303   |     See Other      | 用于在 PUT 或者 POST 请求之后进行重定向，这样在结果页就不会再次触发重定向了 |
|  304   |    Not Modified    | 资源未修改，表示本地缓存仍然可用。产生这个状态的前提是：客户端本地已经有缓存的版本，并且在 Request 中告诉了服务端，当服务端通过时间或者 Etag，发现没有更新的时候，就会返回一个不含 body 的 304 状态 |
|  305   |     Use Proxy      | 用来表示必须通过一个代理来访问资源，代理的位置有 `Location` 头字段给出 |
|  306   |    Switch Proxy    | 在最新版的规范中，306 状态码已经不再被使用。最初是指“后续请求应使用指定的代理” |
|  307   | Temporary Redirect |          与 302 类似，但是使用原请求方法发起新情求           |
|  308   | Permanent Redirect |          与 301 类似，但是使用原请求方法发起新情求           |

这 9 种状态码可以分成 3 大类，分别是：永久重定向、临时重定向以及特殊重定向

- 永久重定向： 301 、 308
- 临时重定向： 302、303、307
- 特殊重定向： 300、304、305、306

<br >

#### 永久重定向

301 和 308 都属于永久重定向，301 本来在规范中是**不允许**重定向时改变请求方法的（将POST改为GET），但是许多浏览器却**允许重定向时改变请求方法**（这是一种不规范的实现）

308 的出现也是给上面的行为做个规范，不过是**不允许重定向时改变请求方法**

|                          Permanent                          | Temporary |      |
| :---------------------------------------------------------: | :-------: | :--: |
|     Allows changing the request method from POST to GET     |    301    | 302  |
| Does not allow changing the request method from POST to GET |    308    | 307  |

<br >

#### 临时重定向

302、303、307 都属于临时重定向，临时是指访问的资源可能暂时先用location的URI访问，但旧资源还在的，下次你再来访问的时候可能就不用重定向了

302 和 307 的关系类似于 301 和 308，303通常用来在创建、修改和删除时展示临时的进度页

<br >

#### 特殊重定向

除此之外，300/304/305/306 可以归属到特殊重定向类。这里重点说一下 304，304 是 HTTP 缓存中的一个重要内容，表示资源未修改，相当于将资源重定向到本地缓存

304 又是一个每个前端必知必会的状态，产生这个状态的前提是：客户端本地已经有缓存的版本，并且在 Request 中告诉了服务端，当服务端通过时间或者 Etag，发现没有更新的时候，就会返回一个不含 body 的 304 状态

<br >

### 4xx

表示客户端发送的请求报文有误，服务器无法处理，它就是真正的“错误码”含义了

<br >

#### 400（Bad Request）

由于明显的客户端错误（例如，格式错误的请求语法，太大的大小，无效的请求消息或欺骗性路由请求），服务器不能或不会处理该请求

<br >

#### 403（Forbidden）

表示服务器禁止访问资源。原因可能多种多样，例如信息敏感、法律禁止等，如果服务器友好一点，可以在 body 里详细说明拒绝请求的原因，不过现实中通常都是直接给一个“闭门羹”

<br >

#### 404（Not Found）

请求失败，请求所希望得到的资源未在服务器上发现，但允许用户的后续请求

<br >

### 5xx

服务器错误，服务器在处理请求时内部发生了错误

<br >

#### 500（Internal Server Error）

通用错误消息，服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。没有给出具体错误信息

<br >

#### 501（Not Implemented）

服务器不支持当前请求所需要的某个功能。当服务器无法识别请求的方法，并且无法支持其对任何资源的请求

<br >

#### 502（Bad Gateway）

作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应

<br >

#### 503（Service Unavailable）

表示服务器当前很忙，暂时无法响应服务，我们上网时有时候遇到的“网络服务正忙，请稍后重试”的提示信息就是状态码 503

<br >

<br >

## 容易争论的点

### 301、302 和 307区别（对 SEO 的影响）

- 301：可通知搜索引擎蜘蛛，表示某个网页或网站已被永久移动到新位置
- 302：搜索引擎蜘蛛会继续抓取原有位置并将其编入索引，因此某个页面或网站已被移动时，不要使用此代码来通知搜索引擎蜘蛛。
- 307：临时重定向，307 的定义实际上和 302 是一致的，唯一的区别在于，307 状态码不允许浏览器将原本为 POST 的请求重定向到 GET 请求上

<br >

### 401 和 404 的区别

- 401 unauthorized，表示发送的请求需要有通过 HTTP 认证的认证信息
- 403 forbidden，表示对请求资源的访问被服务器拒绝

<br >

打个生动的比方：

- 401：我去找个人，门卫说不认识我不让我进
- 403：我去找个人，门卫说认识我，但是我不能进，因为我不配





