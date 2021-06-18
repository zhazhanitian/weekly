## 引言

本文主要分为以下几个方面循序渐进走进 HTTP/3

- HTTP/2 和 TCP 的致命缺陷
- QUIC 协议为什么选择 UDP
- QUIC 和 HTTP/3 新特性
- QUIC 和 HTTP/3 前景发展展望

<br >

<br >

## HTTP/2 和 TCP 的缺陷

HTTP/2 使用二进制传输、Header 压缩（HPACK）、多路复用等，相较于 HTTP/1.1 大幅提高了数据传输效率，但它仍然存在着以下几个致命问题（主要由底层支撑的 TCP 协议造成）

- 建立连接时间长
- 队头阻塞问题相较于 HTTP/1.1 更严重

<br >

### 建立连接时间长

#### RTT 往返时间

如何定义建立连接时间喃？这里引入一个概念：RTT（Round-Trip Time），往返时间，表示从发送端发送数据开始，到发送端收到来自接收端的确认（接收端收到数据后便立即发送确认，不包含数据传输时间）总共经历的时间，即通信一来一回的时间

<br >

#### TCP 建立连接时间

TCP 通过三次挥手建立了 TCP 虚拟通道，它总共需要花费

<img src="https://qiniu-image.qtshe.com/63E8B1C8DA507.png" width="660px" style="float:left;" />

- 一去 （SYN）：客户端向服务端发送连接请求报文段。该报文段中包含自身的数据通讯初始序号。请求发送后，客户端便进入 SYN-SENT 状态
- 二回 （SYN+ACK）：服务端收到连接请求报文段后，如果同意连接，则会发送一个应答，该应答中也会包含自身的数据通讯初始序号，发送完成后便进入 SYN-RECEIVED 状态
- 三去 （ACK）：当客户端收到连接同意的应答后，还要向服务端发送一个确认报文。客户端发完这个报文段后便进入 ESTABLISHED 状态，服务端收到这个应答后也进入 ESTABLISHED 状态，此时连接建立成功

相当于一个半来回，故 TCP 建立连接时间 = 1.5 RTT

<br >

#### HTTP 交易时间

客户端在请求数据的时候，首先花费 1.5 RTT 建立 TCP 连接，然后 TCP 才开始传输 HTTP 请求，浏览器收到服务器的响应，又要等待的时间为

- 一去（HTTP Request）
- 二回 （HTTP Responses）

故 HTTP 交易时间 = 1 RTT

由于 TCP 在第三次握手的时候，不需要等待服务器端的响应，所以节省 0.5 RTT，那么基于 TCP 传输的 HTTP 通信，一共花费的时间总和

**HTTP 通信时间总和 = TCP 建立连接时间 + HTTP 交易时间 = 1 RTT + 1 RTT = 2 RTT** 

<br >

#### HTTPS 通信时间

HTTP/2 延续了 HTTP/1 的“明文”特点，可以像以前一样使用明文传输数据，不强制使用加密通信，但 HTTPS 已经是大势所趋，各大主流浏览器都公开宣布只支持加密的 HTTP/2，所以，真实应用中的 HTTP/2 是还是加密的

<img src="https://qiniu-image.qtshe.com/4B0484B9C3B9.png" style="float:left;" width="660px" />

<img src="https://qiniu-image.qtshe.com/D4FE489E6617.png" style="float:left;" width="660px" />

<br >

HTTPS 其实是 HTTP+SSL/TLS 的简称

所以，HTTPS 通信时间 = TCP建立连接时间 +  TLS 连接时间 + HTTP交易时间

<br >

#### TLS 连接时间

在 TLS 1.2 协议的握手，需要 2 个 RTT

<img src="https://qiniu-image.qtshe.com/097CF84CEB781C.png" style="float:left;" width="660px" />

- **一去：**客户端发送一个随机数 C，客户端的 TLS 版本号以及支持的密码套件列表给服务器端
- **二回：**服务端收到客户端的随机值，自己也产生一个随机值 S ，并根据客户端需求的协议和加密方式来使用对应的方式，并且发送自己的证书（如果需要验证客户端证书需要说明）
- **三去：**客户端收到服务端的证书并验证是否有效，验证通过会再生成一个随机值 pre-master，通过服务端证书的公钥去加密这个随机值并发送给服务端。如果服务端需要验证客户端证书的话会附带证书（双向认证，比如网上银行用 U 盾）
- **四回：** 服务端收到加密过的随机值并使用私钥解密获得第三个随机值，这时候两端都拥有了三个随机值，可以通过这三个随机值（C/S 加 pre-master 算出主密钥）按照之前约定的加密方式生成密钥，接下来的通信就可以通过该会话密钥来加密解密

HTTPS 通信时间总和 = TCP 建立连接时间 + TLS 连接时间 + HTTP交易时间 = 1 RTT + 2 RTT + 1 RTT = 4 RTT

如果服务器距离客户端很近，RTT 时间较短 < 10ms，那么 HTTPS 通信时间也不会超过 40 ms，用户不会感知，但如果距离较远，相隔上万公里，一个 RTT 时间通常在200ms以上，那么 HTTPS 通信将花费 800ms 甚至 1s 以上，这就严重影响到用户体验了

**注意：在 TLS 1.3 协议中，首次建立连接只需要一个 RTT，后面恢复连接不需要 RTT 了**

**HTTPS 通信时间总和（基于TLS1.2） = TCP 建立连接时间 + TLS1.2 连接时间 + HTTP交易时间 = 1 RTT + 2 RTT + 1 RTT = 4 RTT**

**HTTPS 通信时间总和（基于TLS1.3） = TCP 建立连接时间 + TLS1.3 连接时间 + HTTP交易时间 = 1 RTT + 1 RTT + 1 RTT = 3 RTT**

<br >

### 队头阻塞问题相较于 HTTP/1.1 更严重

因为 HTTP/2 使用了多路复用，一般来说同一域名下只需要使用一个 TCP 连接。当这个连接中出现了丢包的情况，那就会导致 HTTP/2 的表现情况反倒不如 HTTP/1 了

因为在出现丢包的情况下，整个 TCP 都要开始等待重传，也就导致了后面的所有数据都被阻塞了。但是对于 HTTP/1 来说，可以开启多个 TCP 连接，出现这种情况反到只会影响其中一个连接，剩余的 TCP 连接还可以正常传输数据

<br >

<br >

## QUIC 协议为什么选择 UDP

那么可能就会有人考虑到去修改 TCP 协议，其实这已经是一件不可能完成的任务了。因为 TCP 存在的时间实在太长，已经充斥在各种设备中，并且这个协议是由操作系统实现的，更新起来不大现实

基于这个原因，Google 就更起炉灶搞了一个基于 UDP 协议的 QUIC 协议

谷歌这样做看似出乎意料的，但我们对比一下 TCP 与 UDP 就会发现，这是很有道理的

- 基于 TCP 开发的设备和协议非常多，兼容困难
- TCP 协议栈是 Linux 内部的重要部分，修改和升级成本很大
- UDP 本身是无连接的、没有建链和拆链成本
- UDP 的数据包无队头阻塞问题
- UDP 改造成本小

从上面的对比可以知道，谷歌要想从 TCP 上进行改造升级绝非易事，但是 UDP 虽然没有 TCP 为了保证可靠连接而引发的问题，但是 UDP 本身不可靠，又不能直接用

<img src="https://qiniu-image.qtshe.com/74FD38C7B42A66.png" style="float:left;" width="660px" />

所以，谷歌决定在 UDP 基础上改造一个具备 TCP 协议优点的新协议也就顺理成章了，这个新协议就是 QUIC 协议（Quick UDP Internet Connection），并且使用在了 HTTP/3 上，当然 HTTP/3 之前名为 HTTP-over-QUIC，从这个名字中我们也可以发现，HTTP/3 最大的改造就是使用了 QUIC

<br >

<br >

## QUIC 和 HTTP/3 新特性

<img src="https://qiniu-image.qtshe.com/CF4B4DDE4D8F.png" style="float:left;" width="560px" />

QUIC 虽然基于 UDP，但是在原本的基础上新增了很多功能，比如多路复用、0-RTT、使用 TLS1.3 加密、流量控制、有序交付、重传等等功能。这里我们就挑选几个重要的功能学习下这个协议的内容

<br >

### 多路复用，解决队头阻塞问题

虽然 HTTP/2 支持了多路复用，但是 TCP 协议终究是没有这个功能的。QUIC 原生就实现了这个功能

QUIC 协议是基于 UDP 协议实现的，同一个 QUIC 连接上可以创建多个 stream（数据流） 来发送多个 HTTP 请求，并且，多个 stream 之间没有依赖，传输的单个 stream可以保证有序交付且不会影响其他的数据流

例如下图，stream2 丢了一个 UDP 包，不会影响后面跟着 Stream3 和 Stream4。这样的技术就解决了之前 TCP 存在的队头阻塞问题

<img src="https://qiniu-image.qtshe.com/26229FAAEFEE22.png" style="float:left;" width="660px" />

并且 QUIC 在移动端的表现也会比 TCP 好。因为 TCP 是基于 IP 和端口去识别连接的，这种方式在多变的移动端网络环境下是很脆弱的。但是 QUIC 是通过 ID 的方式去识别一个连接，不管你网络环境如何变化，只要 ID 不变，就能迅速重连上

<br >

### 0RTT

通过使用类似 TCP 快速打开的技术，缓存当前会话的上下文，在下次恢复会话的时候，只需要将之前的缓存传递给服务端验证通过就可以进行传输了

**0RTT 建连可以说是 QUIC 相比 HTTP2 最大的性能优势**。那什么是 0RTT 建连呢？

这里面有两层含义

- 传输层 0RTT 就能建立连接
- 加密层 0RTT 就能建立加密连接

<img src="https://qiniu-image.qtshe.com/56E8C7B45BB66.gif" style="float:left;" width="660px" />

上图左边是 HTTPS 的一次完全握手的建连过程，需要 2-3 个 RTT才开始传输数据，右边 QUIC 协议在第一个包就可以包含有效的应用数据

当然，QUIC 协议可以实现 0RTT ，但这也是有条件的，实际上是首次连接 1RTT，非首次连接 0RTT，首次连接过程

<img src="https://qiniu-image.qtshe.com/0904BB7AB492B.png" style="float:left;" width="660px" />

可以看到，首次连接的时候，在第 4 步时，就已经开始发送实际的业务数据了，而第 1 - 3 步正好一去一回花费了 1RTT 时间，所以，首次连接的成本是 1RTT

<img src="https://qiniu-image.qtshe.com/EE2D2E9C2C064.png" style="float:left;" width="660px" />

<br >

### 向前纠错机制

QUIC 协议有一个非常独特的特性，称为向前纠错 (Forward Error Correction，FEC)，每个数据包除了它本身的内容之外，还包括了部分其他数据包的数据，因此少量的丢包可以通过其他包的冗余数据直接组装而无需重传

向前纠错牺牲了每个数据包可以发送数据的上限，但是减少了因为丢包导致的数据重传，因为数据重传将会消耗更多的时间（包括确认数据包丢失、请求重传、等待新数据包等步骤的时间消耗）

假如说这次我要发送三个包，那么协议会算出这三个包的异或值并单独发出一个校验包，也就是总共发出了四个包

当出现其中的非校验包丢包的情况时，可以通过另外三个包计算出丢失的数据包的内容

当然这种技术只能使用在丢失一个包的情况下，如果出现丢失多个包就不能使用纠错机制了，只能使用重传的方式了

<br >

### 加密认证的报文

TCP 协议头部没有经过任何加密和认证，所以在传输过程中很容易被中间网络设备篡改，注入和窃听。比如修改序列号、滑动窗口。这些行为有可能是出于性能优化，也有可能是主动攻击

但是 QUIC 的 packet 可以说是武装到了牙齿。除了个别报文比如 PUBLIC_RESET 和 CHLO，所有报文头部都是经过认证的，报文 Body 都是经过加密的

这样只要对 QUIC 报文任何修改，接收端都能够及时发现，有效地降低了安全风险

<img src="https://qiniu-image.qtshe.com/041C51A3FC1BA6.png" style="float:left;" width="660px" />

如上图所示，红色部分是 Stream Frame 的报文头部，有认证。绿色部分是报文内容，全部经过加密

<br >

<br >

## QUIC 和 HTTP/3 前景发展展望

QUIC 协议虽然是基于 UDP 来实现的，但它将 TCP 的重要功能都进行了实现和优化，同时在加密传输方向的尝试也推动了TLS1.3的发展，未来还是可期的

只是现在 TCP 协议的势力过于强大，很多网络设备甚至对于UDP数据包做了很多不友好的策略，所以现在暂时还是 TCP 的天下，不过 QUIC 已经展现了强大的生命力，让我们拭目以待!



### 相关知识

- 图解|为什么HTTP3.0使用UDP协议：https://network.51cto.com/art/202009/625999.htm
- 如何看待 HTTP/3 ？：https://www.zhihu.com/question/302412059/answer/533223530

