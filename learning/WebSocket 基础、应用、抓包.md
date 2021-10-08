## 前言

WebSocket 是为了满足基于 Web 的日益增长的实时通信需求而产生的

在传统的 Web 中，要实现实时通信，通用的方式是采用 HTTP 协议不断发送请求，即轮询（Polling）（例如：腾讯即时通讯）

但这种方式既浪费带宽（HTTP HEAD 是比较大的），又导致服务器 CPU 占用（没有信息也要接受请求）

<img src="https://qiniu-image.qtshe.com/1DCFA02E03F13A.png" width="600" style="float:left;" />

<br />

而使用 WebSocket 技术，则能大幅优化上面提到的问题

<img src="https://qiniu-image.qtshe.com/2DCFA02E03F13A.png" width="600" style="float:left;" />

<br />

<br />

## WebSocket 简介

WebSocket 协议在 2008 年诞生，2011 年成为国际标准。所有浏览器都已经支持了

它是从 HTML5 开始提供的一种浏览器与服务器进行全双工通讯的网络技术，属于应用层协议。它基于 TCP 传输协议，并复用 HTTP 的握手通道

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种

其他特点包括

1. 建立在 TCP 协议之上，服务器端的实现比较容易
2. 与 HTTP 协议有着良好的兼容性。默认端口也是 80 和 443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器
3. 较少的控制开销。连接创建后，ws 客户端、服务端进行数据交换时，协议控制的数据包头部较小。在不包含头部的情况下，服务端到客户端的包头只有 2~10 字节（取决于数据包长度），客户端到服务端的的话，需要加上额外的 4 字节的掩码。而 HTTP 协议每次通信都需要携带完整的头部
4. 可以发送文本，也可以发送二进制数据
5. 没有同源限制，客户端可以与任意服务器通信
6. 协议标识符是 ws（如果加密，则为 wss），服务器网址就是 URL
7. 支持扩展。ws 协议定义了扩展，用户可以扩展协议，或者实现自定义的子协议。（比如支持自定义压缩算法等）

<br />

### WebSocket、HTTP、TCP 之间的关系

在下图中，我们只需要知道，HTTP、WebSocket 等协议都是处于 OSI 模型的最高层：应用层。而 IP 协议工作在网络层（第 3 层），TCP 协议工作在传输层（第 4 层）

HTTP、WebSocket 等应用层协议，都是基于 TCP 协议来传输数据的。我们可以把这些高级协议理解成对 TCP 的封装。既然大家都使用 TCP 协议，那么大家的连接和断开，都要遵循 TCP 协议中的三次握手和四次挥手 ，只是在连接之后发送的内容不同，或者是断开的时间不同

<img src="https://qiniu-image.qtshe.com/3DCFA02E03F13A.png" width="800" style="float:left;" />

<br />

### HTML5 与 WebSocket

WebSocket API 是 HTML5 标准的一部分， 但这并不代表 WebSocket 一定要用在 HTML 中，或者只能在基于浏览器的应用程序中使用。

实际上，许多语言、框架和服务器都提供了 WebSocket 支持，例如

- 基于 C 的 libwebsocket.org
- 基于 Node.js 的 Socket.io
- 基于 Python 的 ws4py
- 基于 C++ 的 WebSocket++
- Apache 对 WebSocket 的支持：Apache Module mod_proxy_wstunnel
- Nginx 对 WebSockets 的支持：NGINX as a WebSockets Proxy 、 NGINX Announces Support for WebSocket Protocol 、WebSocket proxying
- lighttpd 对 WebSocket 的支持：mod_websocket

<br />

<br />

## 例子与抓包分析

### 入门例子

先来看一个简单的例子，有个直观感受。例子包括了 WebSocket 服务端（ Node.js ）、WebSocket 客户端

#### 服务端

```js
// 导入WebSocket模块:
const WebSocket = require('ws');

// 引用Server类:
const WebSocketServer = WebSocket.Server;

// 实例化:
const wss = new WebSocketServer({
    port: 3000
});
wss.on('connection', function (ws) {
    console.log([SERVER] connection());
    ws.on('message', function (message) {
        console.log([SERVER] Received: ${message});
        ws.send(message from server: ${message}, (err) => {
            if (err) {
                console.log([SERVER] error: ${err});
            }
        });
    })
});
```

#### 客户端

```js
const WebSocket = require('ws');
const ws = new WebSocket('ws://localhost:3000/');

ws.on('open', function open() {
  console.log('[CLIENT]: open')
  ws.send('something');
});

ws.on('close', function close(){
  console.log('[CLIENT]: close');
});
ws.on('message', function incoming(data) {
  console.log('[CLIENT]: Received:',data);
});
ws.on('ping', function(){
  console.log('[CLIENT]: ping')
})
```

#### 服务端输出

```java
[SERVER] connection()
[SERVER] Received: something
```

#### 客户端输出

```java
[CLIENT]: open
[CLIENT]: Received: message from server: something
```

<br />

### 从抓包看如何建立连接

#### 工具准备

1. 安装 Wireshark 抓包软件

2. 在 Capture 中选择本机回环网络

   <img src="https://qiniu-image.qtshe.com/4DCFA02E03F13A.png" width="700" style="float:left;" />

3. 在 filter 中写入过滤条件 tcp.port == 3000 (ws 服务端口)

这样就可以抓到想要的包啦

<img src="https://qiniu-image.qtshe.com/5DCFA02E03F13A.png" width="700" style="float:left;" />

为了更好的对比 WebSocket 的连接及数据传输与 TCP 和 HTTP 有什么区别，我们再抓一下 TCP 和 HTTP 的包

<br />

### TCP 抓包

#### 服务端代码

```js
const net = require('net');

const server = net.createServer();

server.on('connection', (socket) => {
  socket.on('data', (data) => {
    console.log('Receive from client:', data.toString('utf8'));
  });
  socket.write('Hello, I am from server.');
});

server.listen(3000, () => {
  console.log('Server is listenning on 3000');
});
```

#### 客户端代码

```js
const net = require('net');

const client = new net.Socket();

client.setEncoding('utf8');

client.connect(3000, () => {
  console.log('Connected to server.');
  client.write('Hello, I am from client.');
  client.on('data', (data) => {
    console.log('Receive from server:', data);
  });
});
```

#### 抓包结果

<img src="https://qiniu-image.qtshe.com/6DCFA02E03F13A.png" width="700" style="float:left;" />

简单理解一下 TCP FLAGS

在 TCP 层，有个 FLAGS 字段，这个字段有以下几个标识：SYN (synchronous 建立联机)、ACK (acknowledgement 确认)、PSH (push 传送)、FIN (finish 结束)、RST (reset 重置)、URG (urgent 紧急)

其中，对于我们日常的分析有用的就是前面的五个字段

它们的含义分别是

- SYN 表示建立连接
- FIN 表示关闭连接
- ACK 表示响应
- PSH 表示有 DATA 数据传输
- RST 表示连接重置

用一张图清楚表示一下 TCP 3 次握手及 4 次挥手的过程

<img src="https://qiniu-image.qtshe.com/7DCFA02E03F13A.png" width="600" style="float:left;" />

<br />

### HTTP 抓包

#### 服务端代码

```js
const http = require('http');

const server = http.createServer();

server.on('request', (req, res) => {
  console.log('request ...');
  req.on('data', (data) => {
    console.log('data from client ', data.toString('utf-8'));
  });
  res.write('Hello, I am Server');
  res.end();
});

server.listen(3000);
```

#### 客户端代码

```js
const request = require('request');

request('http://127.0.0.1:3000?param=1', (err, response, body) => {
  console.log('Response:', body);
});
```

可以看到连接和断开连接和 TCP 都是一样的，中间的数据传输换成了 HTTP 协议数据

<img src="https://qiniu-image.qtshe.com/8DCFA02E03F13A.png" width="700" style="float:left;" />

<br />

### 再回来看 WebSocket 的抓包：如何建立连接

WebSocket 复用了 HTTP 的握手通道。具体指的是，客户端通过 HTTP 请求与 WebSocket 服务端协商升级协议。协议升级完成后，后续的数据交换则遵照 WebSocket 的协议

<img src="https://qiniu-image.qtshe.com/9DCFA02E03F13A.png" width="700" style="float:left;" />

#### 客户端：申请协议升级

首先，客户端发起协议升级请求。可以看到，采用的是标准的 HTTP 报文格式，且只支持 GET 方法

<img src="https://qiniu-image.qtshe.com/10DCFA02E03F13A.png" width="700" style="float:left;" />

Connection: Upgrade：表示要升级协议

Upgrade: websocket：表示要升级到 websocket 协议

Sec-WebSocket-Version: 13：表示 websocket 的版本。如果服务端不支持该版本，需要返回一个

Sec-WebSocket-Key：与后面服务端响应首部的 Sec-WebSocket-Accept 是配套的，提供基本的防护，比如恶意的连接，或者无意的连接

#### 服务端：响应协议升级

服务端返回内容如下，状态代码 101 表示协议切换

<img src="https://qiniu-image.qtshe.com/11DCFA02E03F13A.png" width="700" style="float:left;" />

到此完成协议升级，后续的数据交互都按照新的协议来

#### Sec-WebSocket-Accept 的计算

`Sec-WebSocket-Accept` 根据客户端请求首部的 `Sec-WebSocket-Key` 计算出来

计算公式为

将 `Sec-WebSocket-Key` 跟 258EAFA5-E914-47DA-95CA-C5AB0DC85B11 拼接。通过 SHA1 计算出摘要，并转成 base64 字符串。伪代码如下

```js
toBase64( sha1( Sec-WebSocket-Key + 258EAFA5-E914-47DA-95CA-C5AB0DC85B11 )  )
```

验证下前面的返回结果

```js
const crypto = require('crypto');

const magic = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11';
const secWebSocketKey = '8cyP/EvUjJHSMbkOIHFU/w==';

const secWebSocketAccept = crypto.createHash('sha1')
  .update(secWebSocketKey + magic)
  .digest('base64');

console.log(secWebSocketAccept);
// EiaKGKO0E/pC8vnArob263aS3XY=
```

<br />

<br />

## 数据帧格式

客户端、服务端数据的交换，离不开数据帧格式的定义。因此，在实际讲解数据交换之前，我们先来看下 WebSocket 的数据帧格式。

WebSocket 客户端、服务端通信的最小单位是帧（frame），由 1 个或多个帧组成一条完整的消息（message）

发送端：将消息切割成多个帧，并发送给服务端；接收端：接收消息帧，并将关联的帧重新组装成完整的消息

<br />

### 数据帧格式概览

下面给出了 WebSocket 数据帧的统一格式 从左到右，单位是比特。比如 FIN、RSV1 各占据 1 比特，opcode 占据 4 比特。内容包括了标识、操作代码、掩码、数据、数据长度等

Frame format

```html
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-------+-+-------------+-------------------------------+
 |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
 |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
 |N|V|V|V|       |S|             |   (if payload len==126/127)   |
 | |1|2|3|       |K|             |                               |
 +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
 |     Extended payload length continued, if payload len == 127  |
 + - - - - - - - - - - - - - - - +-------------------------------+
 |                               |Masking-key, if MASK set to 1  |
 +-------------------------------+-------------------------------+
 | Masking-key (continued)       |          Payload Data         |
 +-------------------------------- - - - - - - - - - - - - - - - +
 :                     Payload Data continued ...                :
 + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
 |                     Payload Data continued ...                |
 +---------------------------------------------------------------+
```

抓包例子

<img src="https://qiniu-image.qtshe.com/12DCFA02E03F13A.png" width="700" style="float:left;" />

<br />

### 数据帧格式详解

#### FIN

1 个比特

如果是 1，表示这是消息（message）的最后一个分片（fragment），如果是 0，表示不是是消息（message）的最后一个分片（fragment）

#### RSV1, RSV2, RSV3

各占 1 个比特

一般情况下全为 0。当客户端、服务端协商采用 WebSocket 扩展时，这三个标志位可以非 0，且值的含义由扩展进行定义。如果出现非零的值，且并没有采用 WebSocket 扩展，连接出错

#### Opcode

4 个比特

操作代码，Opcode 的值决定了应该如何解析后续的数据载荷（data payload）。如果操作代码是不认识的，那么接收端应该断开连接（fail the connection）。可选的操作代码如下

- % x0：表示一个延续帧。当 Opcode 为 0 时，表示本次数据传输采用了数据分片，当前收到的数据帧为其中一个数据分片
- % x1：表示这是一个文本帧（frame）
- % x2：表示这是一个二进制帧（frame）
- % x3-7：保留的操作代码，用于后续定义的非控制帧
- % x8：表示连接断开
- % x8：表示这是一个 ping 操作
- % xA：表示这是一个 pong 操作
- % xB-F：保留的操作代码，用于后续定义的控制帧

#### Mask

1 个比特

表示是否要对数据载荷进行掩码操作。从客户端向服务端发送数据时，需要对数据进行掩码操作；从服务端向客户端发送数据时，不需要对数据进行掩码操作

<img src="https://qiniu-image.qtshe.com/13DCFA02E03F13A.png" width="700" style="float:left;" />

如果服务端接收到的数据没有进行过掩码操作，服务端需要断开连接

如果 Mask 是 1，那么在 Masking-key 中会定义一个掩码键（masking key），并用这个掩码键来对数据载荷进行反掩码。所有客户端发送到服务端的数据帧，Mask 都是 1

#### Payload length

数据载荷的长度，单位是字节。为 7 位，或 7+16 位，或 1+64 位

假设数 Payload length === x，如果

- x 为 0~126：数据的长度为 x 字节
- x 为 126：后续 2 个字节代表一个 16 位的无符号整数，该无符号整数的值为数据的长度
- x 为 127：后续 8 个字节代表一个 64 位的无符号整数（最高位为 0），该无符号整数的值为数据的长度。此外，如果 payload length 占用了多个字节的话，payload length 的二进制表达采用网络序（big endian，重要的位在前

```js
		// 来自 ws库 sender.js frame函数
    let payloadLength = data.length;

    if (data.length >= 65536) {
      offset += 8;
      payloadLength = 127;
    } elseif (data.length > 125) {
      offset += 2;
      payloadLength = 126;
    }
```

#### Masking-key

0 或 4 字节（32 位）

所有从客户端传送到服务端的数据帧，数据载荷都进行了掩码操作，Mask 为 1，且携带了 4 字节的 Masking-key。如果 Mask 为 0，则没有 Masking-key

备注：载荷数据的长度，不包括 mask key 的长度

```js
//mask key的生成
//每个数据帧都会生成一次
const mask = Buffer.alloc(4);
randomFillSync(mask, 0, 4);
```

#### Payload data

(x+y) 字节

载荷数据：包括了扩展数据、应用数据。其中，扩展数据 x 字节，应用数据 y 字节

扩展数据：如果没有协商使用扩展的话，扩展数据数据为 0 字节。所有的扩展都必须声明扩展数据的长度，或者可以如何计算出扩展数据的长度。此外，扩展如何使用必须在握手阶段就协商好。如果扩展数据存在，那么载荷数据长度必须将扩展数据的长度包含在内

应用数据：任意的应用数据，在扩展数据之后（如果存在扩展数据），占据了数据帧剩余的位置。载荷数据长度 减去 扩展数据长度，就得到应用数据的长度

<br />

### 掩码算法

掩码键（Masking-key）是由客户端挑选出来的 32 位的随机数。掩码操作不会影响数据载荷的长度。掩码、反掩码操作都采用如下算法

首先，假设

- original-octet-i：为原始数据的第 i 字节
- transformed-octet-i：为转换后的数据的第 i 字节
- j：为 i mod 4 的结果
- masking-key-octet-j：为 mask key 第 j 字节

算法描述为：original-octet-i 与 masking-key-octet-j 异或后，得到 transformed-octet-i

```js
j = i MOD 4
transformed-octet-i = original-octet-i XOR masking-key-octet-j
```

ws 库中的 mask 和 unmask 函数

```js
// mask 的生成
// const mask = crypto.randomBytes(4);
// <Buffer 54 63 0c 77>
/**
 * Masks a buffer using the given mask.
 *
 * @param {Buffer} source The buffer to mask
 * @param {Buffer} mask The mask to use
 * @param {Buffer} output The buffer where to store the result
 * @param {Number} offset The offset at which to start writing
 * @param {Number} length The number of bytes to mask.
 * @public
 */
function _mask(source, mask, output, offset, length) {
  for (var i = 0; i < length; i++) {
    output[offset + i] = source[i] ^ mask[i & 3];
  }
}

/**
 * Unmasks a buffer using the given mask.
 *
 * @param {Buffer} buffer The buffer to unmask
 * @param {Buffer} mask The mask to use
 * @public
 */
function _unmask(buffer, mask) {
  // Required until https://github.com/nodejs/node/issues/9006 is resolved.
  const length = buffer.length;
  for (var i = 0; i < length; i++) {
    buffer[i] ^= mask[i & 3];
  }
}

/**
  * Frames a piece of data according to the HyBi WebSocket protocol.
  *
  * @param {Buffer} data The data to frame
  * @param {Object} options Options object
  * @param {Number} options.opcode The opcode
  * @param {Boolean} options.readOnly Specifies whether `data` can be modified
  * @param {Boolean} options.fin Specifies whether or not to set the FIN bit
  * @param {Boolean} options.mask Specifies whether or not to mask `data`
  * @param {Boolean} options.rsv1 Specifies whether or not to set the RSV1 bit
  * @return {Buffer[]} The framed data as a list of `Buffer` instances
  * @public
  */
function frame(data, options) {
  const merge = data.length < 1024 || (options.mask && options.readOnly);
  let offset = options.mask ? 6 : 2;
  let payloadLength = data.length;

  if (data.length >= 65536) {
    offset += 8;
    payloadLength = 127;
  } elseif (data.length > 125) {
    offset += 2;
    payloadLength = 126;
  }

  const target = Buffer.allocUnsafe(merge ? data.length + offset : offset);

  target[0] = options.fin ? options.opcode | 0x80 : options.opcode;
  if (options.rsv1) target[0] |= 0x40;

  if (payloadLength === 126) {
    target.writeUInt16BE(data.length, 2);
  } elseif (payloadLength === 127) {
    target.writeUInt32BE(0, 2);
    target.writeUInt32BE(data.length, 6);
  }

  if (!options.mask) {
    target[1] = payloadLength;
    if (merge) {
      data.copy(target, offset);
      return [target];
    }

    return [target, data];
  }
//验证一下
  // const mask = crypto.randomBytes(4);
  const mask = Buffer.from('32fd435f', 'hex'); //为了还原例子，这里直接指定mask

  target[1] = payloadLength | 0x80;
  target[offset - 4] = mask[0];
  target[offset - 3] = mask[1];
  target[offset - 2] = mask[2];
  target[offset - 1] = mask[3];

  if (merge) {
    _mask(data, mask, target, offset, data.length);
    return [target];
  }

  _mask(data, mask, data, 0, data.length);
  return [target, data];
}

const str = 'something';
const source = Buffer.from(str);
const target = frame(source, {
  fin: true, rsv1: false, opcode: 1, mask: true, readOnly: false,
});
console.log('Payload:', source);
console.log('Masked payload:', target);
```

```nginx
Payload: <Buffer 73 6f 6d 65 74 68 69 6e 67>
Masked payload: [ <Buffer 81 89 32 fd 43 5f 41 92 2e 3a 46 95 2a 31 55> ]
```

可以看到结果与下图一致。

原始 payload

<img src="https://qiniu-image.qtshe.com/14DCFA02E03F13A.png" width="700" style="float:left;" />

masked payload

<img src="https://qiniu-image.qtshe.com/15DCFA02E03F13A.png" width="700" style="float:left;" />

<br />

<br />

## 数据传递

一旦 WebSocket 客户端、服务端建立连接后，后续的操作都是基于数据帧的传递

WebSocket 根据 opcode 来区分操作的类型。比如 0x8 表示断开连接，0x0-0x2 表示数据交互

断开连接

<img src="https://qiniu-image.qtshe.com/16DCFA02E03F13A.png" width="700" style="float:left;" />

<br />

### 数据分片

WebSocket 的每条消息可能被切分成多个数据帧。当 WebSocket 的接收方收到一个数据帧时，会根据 FIN 的值来判断，是否已经收到消息的最后一个数据帧

FIN=1 表示当前数据帧为消息的最后一个数据帧，此时接收方已经收到完整的消息，可以对消息进行处理。FIN=0，则接收方还需要继续监听接收其余的数据帧

此外，opcode 在数据交换的场景下，表示的是数据的类型。0x01 表示文本，0x02 表示二进制。而 0x00 比较特殊，表示延续帧（continuation frame），顾名思义，就是完整消息对应的数据帧还没接收完

<br />

### 数据分片例子

下面例子来自 MDN，可以很好地演示数据的分片。客户端向服务端两次发送消息，服务端收到消息后回应客户端，这里主要看客户端往服务端发送的消息

#### 第一条消息

FIN=1, 表示是当前消息的最后一个数据帧。服务端收到当前数据帧后，可以处理消息。opcode=0x1，表示客户端发送的是文本类型

#### 第二条消息

1. FIN=0，opcode=0x1，表示发送的是文本类型，且消息还没发送完成，还有后续的数据帧
2. FIN=0，opcode=0x0，表示消息还没发送完成，还有后续的数据帧，当前的数据帧需要接在上一条数据帧之后
3. FIN=1，opcode=0x0，表示消息已经发送完成，没有后续的数据帧，当前的数据帧需要接在上一条数据帧之后。服务端可以将关联的数据帧组装成完整的消息

```nginx
Client: FIN=1, opcode=0x1, msg="hello"
Server: (process complete message immediately) Hi.
Client: FIN=0, opcode=0x1, msg="and a"
Server: (listening, new message containing text started)
Client: FIN=0, opcode=0x0, msg="happy new"
Server: (listening, payload concatenated to previous message)
Client: FIN=1, opcode=0x0, msg="year!"
Server: (process complete message) Happy new year to you too!
```

<br />

<br />

## 连接保持 + 心跳

WebSocket 为了保持客户端、服务端的实时双向通信，需要确保客户端、服务端之间的 TCP 通道保持连接没有断开。然而，对于长时间没有数据往来的连接，如果依旧长时间保持着，可能会浪费包括的连接资源

但不排除有些场景，客户端、服务端虽然长时间没有数据往来，但仍需要保持连接。这个时候，可以采用心跳来实现

发送方 -> 接收方：ping 

接收方 -> 发送方：pong 

ping 、pong 的操作，对应的是 WebSocket 的两个控制帧，opcode 分别是 0x9、0xA

举例，WebSocket 服务端向客户端发送 ping，只需要如下代码：（采用 ws 模块）

```js
ws.ping('', false, true);
```

<img src="https://qiniu-image.qtshe.com/17DCFA02E03F13A.png" width="700" style="float:left;" />

<img src="https://qiniu-image.qtshe.com/18DCFA02E03F13A.png" width="700" style="float:left;" />
