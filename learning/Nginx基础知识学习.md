## 前言

前面在弄活动配置平台的时候，需要给项目增加一个测试环境，过程中就涉及到了Nginx相关的配置，由于没有超过过相关流程，对于这一块也不熟悉，在最后一个配置步骤的时候和队友折腾了好一会儿才弄好，整得十分尴尬，基此对Nginx的基础知识学习一下

<br >

<br >

## Nginx

*Nginx* (engine x) 是一个高性能的 HTTP 和反向代理 web 服务器

Nginx 以事件驱动的方式编写，所以有非常好的性能，同时也是一个非常高效的反向代理、负载平衡服务器。在性能上，Nginx 占用很少的系统资源，能支持更多的并发连接，达到更高的访问效率；在功能上，Nginx 是优秀的代理服务器和负载均衡服务器；在安装配置上，Nginx 安装简单、配置灵活

Nginx 支持热部署，启动速度特别快，还可以在不间断服务的情况下对软件版本或配置进行升级，即使运行数月也无需重新启动。

在微服务的体系之下，Nginx 正在被越来越多的项目采用作为网关来使用，配合 Lua 做限流、熔断等控制

<img src="https://qiniu-image.qtshe.com/700762DB-6D47D48B.png" style="zoom:67%;float:left;" />

<br >

### 反向代理

Nginx 根据接收到的请求的端口，域名，url，将请求转发给不同的机器，不同的端口（或直接返回结果），然后将返回的数据返回给客户端

在 Java 设计模式中，代理模式是这样定义的：给某个对象提供一个代理对象，并由代理对象控制原对象的引用

<br >

### 反向代理: 客户端 一 > 代理 <一> 服务端

反向代理用一个租房的例子:

A（客户端）想租一个房子，B（代理）就把这个房子租给了他
 这时候实际上C（服务端）才是房东
 B(代理)是中介把这个房子租给了A（客户端）

这个过程中A（客户端）并不知道这个房子到底谁才是房东
 他都有可能认为这个房子就是B（代理）的

<br >

### 反向代理特点

- Nginx没有自己的地址，它的地址就是服务器的地址，如www.baidu.com，对外部来讲，它就是数据的生产者
- Ngxin明确的知道应该去哪个服务器获取数据（在未接收到请求之前，已经确定应该连接哪台服务器

<br >

### 正向代理

所谓正向代理就是顺着请求的方向进行的代理，即代理服务器他是由你配置为你服务，去请求目标服务器地址。正向代理最大的特点是客户端非常明确要访问的服务器地址；服务器只清楚请求来自哪个代理服务器，而不清楚来自哪个具体的客户端；正向代理模式屏蔽或者隐藏了真实客户端信息

<br >

### 正向代理:客户端 <一> 代理 一>服务端

正向代理也简单地打个租房的比方:

A（客户端）想租C（服务端）的房子,但是A（客户端）并不认识C（服务端）租不到。
 B(代理)认识C（服务端）能租这个房子所以你找了B（代理）帮忙租到了这个房子。

这个过程中C（服务端）不认识A(客户端)只认识B（代理）
 C（服务端）并不知道A（客户端）租了房子，只知道房子租给了B（代理）

<img src="https://qiniu-image.qtshe.com/A2CC5CAEE04D.png" style="zoom:100%;float:left;" />

<br >

### Nginx 应用场景

* http 服务器

  Nginx 是一个 http 服务可以独立提供 http 服务。可以做网页静态服务器

* 虚拟主机。可以实现在一台服务器虚拟出多个网站。例如个人网站使用的虚拟主机
  * 基于端口的，不同的端口
  * 基于域名的，不同域名

* 反向代理，负载均衡

  当网站的访问量达到一定程度后，单台服务器不能满足用户的请求时，需要用多台服务器集群可以使用 nginx 做反向代理。并且多台服务器可以平均分担负载，不会因为某台服务器负载高宕机而某台服务器闲置的情况

<br >

### 安装 Nginx

[官网地址](https://nginx.org/en/download.html)

<br >

### 命令

<img src="https://qiniu-image.qtshe.com/5D72B8532484DF7.png" style="zoom:60%;float:left;" />

<br >

<br >

##  Nginx 配置

Nginx 的主配置文件是：nginx.conf

<img src="https://qiniu-image.qtshe.com/917145CA5D67D2.png" style="zoom:90%;float:left;" />

<br >

里面的配置主要是这样

```nginx
# 全局区   有一个工作子进程，一般设置为CPU数 * 核数
worker_processes  1;

events {
  # 一般是配置nginx进程与连接的特性
  # 如1个word能同时允许多少连接，一个子进程最大允许连接1024个连接
  worker_connections  1024;
}

# 配置HTTP服务器配置段
http {

  # 配置虚拟主机段
  server {

  # 定位，把特殊的路径或文件再次定位。
  location  {

  }
}

server {
  ...
  }
}
```

nginx.conf 配置文件大致分为三部分

* 全局块

  从配置文件开始到 events 块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等

  比如上面第一行配置的

  ```nginx
  worker_processes  1;
  ```

  这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约

* events 块

  涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等

* http 块

  Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里

<br >

<br >

## 反向代理

上面已经提过过反向代理，这里来尝试写一个，先在 http 中添加一个 server

```nginx
server {
    listen            80;
    server_name       dev-customer.sdyxmall.com ;
    gzip              off;
    gzip_buffers      4 16k;
    gzip_comp_level   5;
    gzip_http_version 1.0;
    gzip_min_length   1k;
    gzip_types        text/plain text/css application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript image/x-icon image/bmp;
    gzip_vary         on;

    location / {
        proxy_pass   http://127.0.0.1:5000;
        proxy_set_header X-real-ip $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
    }
}
```

然后 配置 hosts 

```js
10.3.100.13 dev-qtshe.com
```

语法

```nginx
1 listen *:80 | *:8080 #监听所有80端口和8080端口
2 listen  IP_address:port   #监听指定的地址和端口号
3 listen  IP_address     #监听指定ip地址所有端口
4 listen port     #监听该端口的所有IP连接
5 server_name     #指令主要用于配置基于名称虚拟主机
6 gzip     #的作用是是否需要开启压缩传输
7 location     #指令用于匹配 URL
8 proxy_pass     #指令用于设置被代理服务器的地址
9 proxy_set_header     #用来设定被代理服务器接收到的 header 信息（请求头）
```

基本上我们了解 server_name，location，proxy_pass 就可以配置反向代理

<br >

<br >

## Nginx 管理虚拟主机

当配置时想在一台服务器虚拟出多个网站，就可以用虚拟主机来实现

虚拟主机使用的是特殊的软硬件技术，它把一台运行在因特网上的服务器主机分成一台台 “虚拟” 的主机，每台虚拟主机都可以是一个独立的网站，可以具有独立的域名，具有完整的 Intemet 服务器功能（WWW、FTP、Email 等），同一台主机上的虚拟主机之间是完全独立的。从网站访问者来看，每一台虚拟主机和一台独立的主机完全一样

<br >

### 基于域名的虚拟主机

* 在 http 大括号中添加如下代码段

  ```nginx
  	server {  
          #监听端口 80  
          listen 80;   
                                  
          #监听域名feng.com;  
          server_name feng.com;
            
          location / {              
              # 相对路径，相对nginx根目录。也可写成绝对路径  
              root    feng;  
              
              # 默认跳转到index.html页面  
              index index.html;                 
          }  
      }
  ```

<br >

* 切换安装目录：cd/usr/local/software/nginx

* 创建目录：mkdir feng

* 新建 index.html 文件：vi /usr/local/software/nginx/feng/index.html，文件内容

  ```html
  		<html>
          <head>
              <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
          </head>
          <body>
              <h2>枫</h2>
          </body>
      </html>
  ```

<br >

* 重新读取配置文件

  /usr/local/software/nginx/sbin/nginx-s reload

  kill -HUP 进程号

* 配置 windows 本机 host

  192.168.197.142  feng.com  // 服务器 IP 地址

* 访问：http ://feng.com:80/

<br >

### 基于端口的虚拟主机配置

```nginx
	  server {
        listen  2022;
        server_name     feng.com;
        location / {
           root    /home;
           index index.html;
        }
    }
```

<br >

### 基于 IP 地址虚拟主机配置

```nginx
    server {
      listen  80;
      server_name  192.168.197.142;
      location / {
              root    ip;
              index index.html;
      }
    }
```

<br >

<br >

## 负载均衡

负载均衡：由于目前现有网络的各个核心部分随着业务量的提高，访问量和数据流量的快速增长，其处理能力和计算强度也相应地增大，使得单一的服务器设备根本无法承担

针对此情况而衍生出来的一种廉价有效透明的方法以扩展现有网络设备和服务器的带宽、增加吞吐量、加强网络数据处理能力、提高网络的灵活性和可用性的技术就是负载均衡（Load Balance）

Nginx 实现负载均衡有几种方案

* 轮询

  轮询即 Round Robin，根据 Nginx 配置文件中的顺序，依次把客户端的 Web 请求分发到不同的后端服务器

  ```nginx
  upstream backserver {
      server 192.168.0.14;
      server 192.168.0.15;
  }
  ```

 <br > 

* weight

  基于权重的负载均衡即 Weighted Load Balancing，这种方式下，我们可以配置 Nginx 把请求更多地分发到高配置的后端服务器上，把相对较少的请求分发到低配服务器

  ```nginx
  upstream backserver {
      server 192.168.0.14 weight=3;
      server 192.168.0.15 weight=7;
  }
  ```

  权重越高，在被访问的概率越大，如上例，分别是 30%，70%

<br >

* ip_hash

  前述的两种负载均衡方案中，同一客户端连续的 Web 请求可能会被分发到不同的后端服务器进行处理，因此如果涉及到会话 Session，那么会话会比较复杂。常见的是基于数据库的会话持久化。要克服上面的难题，可以使用基于 IP 地址哈希的负载均衡方案。这样的话，同一客户端连续的 Web 请求都会被分发到同一服务器进行处理

  ```nginx
  upstream backserver {
      ip_hash;
      server 192.168.0.14:88;
      server 192.168.0.15:80;
  }
  ```

<br >

* fair

  按后端服务器的响应时间来分配请求，响应时间短的优先分配

  ```nginx
  upstream backserver {
      server server1;
      server server2;
      fair;
  }
  ```

<br >

* url_hash

  按访问 url 的 hash 结果来分配请求，使每个 url 定向到同一个（对应的）后端服务器，后端服务器为缓存时比较有效

  ```nginx
  upstream backserver {
      server squid1:3128;
      server squid2:3128;
      hash $request_uri;
      hash_method crc32;
  }
  ```

  在需要使用负载均衡的 server 中增加

  ```nginx
  proxy_pass http://backserver/; 
  upstream backserver{ 
      ip_hash; 
      server 127.0.0.1:9090 down; (down 表示单前的server暂时不参与负载) 
      server 127.0.0.1:8080 weight=2; (weight 默认为1.weight越大，负载的权重就越大) 
      server 127.0.0.1:6060; 
      server 127.0.0.1:7070 backup; (其它所有的非backup机器down或者忙的时候，请求backup机器) 
  } 
  ```

  max_fails ：允许请求失败的次数默认为 1. 当超过最大次数时，返回 proxy_next_upstream 模块定义的错误

  fail_timeout:max_fails 次失败后，暂停的时间

<br >

* 配置实例

  ```nginx
  #user  nobody;
  
  worker_processes  4;
  events {
  # 最大并发数
  worker_connections  1024;
  }
  http{
      # 待选服务器列表
      upstream myproject{
          # ip_hash指令，将同一用户引入同一服务器。
          ip_hash;
          server 125.219.42.4 fail_timeout=60s;
          server 172.31.2.183;
      }
  
      server{
          # 监听端口
          listen 80;
          # 根目录下
          location / {
          # 选择哪个服务器列表
              proxy_pass http://myproject;
          }
  
      }
  }
  ```

  
