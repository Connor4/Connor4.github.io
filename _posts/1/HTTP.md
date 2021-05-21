---
layout: post
title:  "Integrating razorpay into your webapp"
date:   2019-03-23 21:03:36 +0530
categories: Javascript NodeJS

---

# HTTP

标签（空格分隔）： 基础知识

---

# 1. HTTP协议定义

HTTP协议是**超文本传输协议**的缩写，英文是Hyper Text Transfer Protocol。它是从WEB服务器传输超文本标记语言(HTML)到本地浏览器的传送协议。

设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法。

HTPP有多个版本，目前广泛使用的是HTTP/1.1版本。

# 2. HTTP的特点

HTTP 是基于 TCP/IP 协议的[**应用层协议**](http://www.ruanyifeng.com/blog/2012/05/internet_protocol_suite_part_i.html)，默认端口号是80。其工作会在TCP连接建立后，客户端向服务器请求网络数据，服务器就会回应数据。

1. 简单快速：客户端向服务器请求服务时，只需要请求方法和路径。
2. 灵活：HTTP允许传输任意类型的数据对象。传输的类型由Content-type加以标记。
3. 无连接：每次连接只处理一个请求，服务器处理完客户的请求，并收到客户的应答后，就会断开连接，节省服务请资源。
4. 无状态：HTTP协议是无状态协议，对之前的事务处理没有记忆能力。

# 3. URI和URL的区别

- URI：Uniform Resource Identifier 统一资源**标识**符 在某一规则下能把一个资源独一无二的标识出来
- URL：Uniform Resource Location 统一资源**定位**符 一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。 

**大白话，就是**URI是抽象的定义，不管用什么方法表示，只要能定位一个资源，就叫URI，本来设想的的使用两种方法定位：1，URL，用地址定位；2，URN 用名称定位。

举个例子：去村子找个具体的人（URI），如果用地址：某村多少号房子第几间房的主人 就是URL， 如果用身份证号+名字 去找就是URN了。

结果就是 目前WEB上就URL流行开了，平常见得URI 基本都是URL。

# 4. HTTP的请求

## 4.1 请求报文构成

1. 请求行：包括请求方法、URL、协议/版本
2. 请求头 Request Header
3. 请求正文 body

示例：

![](https://raw.githubusercontent.com/Connor4/Demo/master/%E7%AC%94%E8%AE%B0%E5%9B%BE%E7%89%87/http%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87%E7%BB%84%E6%88%90.jpg)



GET请求：

```xml
 GET /books/?sex=man&name=Professional HTTP/1.1
 Host: www.example.com
 User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)
 Gecko/20050225 Firefox/1.0.1
 Connection: Keep-Alive
```

POST请求：

```xml
 POST / HTTP/1.1
 Host: www.example.com
 User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)
 Gecko/20050225 Firefox/1.0.1
 Content-Type: application/x-www-form-urlencoded
 Content-Length: 40
 Connection: Keep-Alive

 sex=man&name=Professional  
```

- GET 可提交的数据量受到URL长度的限制，HTTP 协议规范没有对 URL 长度进行限制。这个限制是特定的浏览器及服务器对它的限制
- 理论上讲，POST 是没有大小限制的，HTTP 协议规范也没有进行大小限制，出于安全考虑，服务器软件在实现时会做一定限制
- 参考上面的报文示例，可以发现 GET 和 POST 数据内容是一模一样的，只是位置不同，一个在 URL 里，一个在 HTTP 包的包体里

### 4.1.1 POST的提交数据方式

HTTP协议规定POST提交必须在body中，服务端根据请求头中的Content-Type字段来得知消息主体是用何种方式编码，然后在对主体解析。

Content-Type类型：

``application/x-www-form-urlencoded``

浏览器原生的**<form>**表单，如果不设置enctype属性，最终就会以application/x-www-form-urlencoded方式提交数据。这种方式下，post提交body中内容与get提交是完全相同的。

``multipart/form-data``

我们使用表单上传文件时，必须让**<form>** 表单的 enctype 等于 multipart/form-data。

还有数据提交方式，例如 `application/json`，`text/xml`，乃至 `application/x-protobuf` 这种二进制格式，只要服务器可以根据 `Content-Type` 和 `Content-Encoding` 正确地解析出请求。

### 4.1.2 条件GET

客户端之前已经访问过某网站，并打算再次访问该网站就可以使用条件 GET 。

客户端向服务器发送一个包询问是否在上一次访问网站的时间后是否更改了页面，如果服务器没有更新，显然不需要把整个网页传给客户端，客户端只要使用本地缓存即可，如果服务器对照客户端给出的时间已经更新了客户端请求的网页，则发送这个更新了的网页给用户。

实例：

客户端发送请求：

```xml
 GET / HTTP/1.1  
 Host: www.sina.com.cn:80  
 If-Modified-Since:Thu, 4 Feb 2010 20:39:13 GMT  
 Connection: Close  
```

第一次请求时，服务器端返回请求数据，之后的请求，**服务器根据请求中的 `If-Modified-Since` 字段判断响应文件没有更新，如果没有更新，服务器返回一个 `304 Not Modified`响应，告诉浏览器请求的资源在浏览器上没有更新，可以使用已缓存的上次获取的文件。**

```xml
 HTTP/1.0 304 Not Modified  
 Date: Thu, 04 Feb 2010 12:38:41 GMT  
 Content-Type: text/html  
 Expires: Thu, 04 Feb 2010 12:39:41 GMT  
 Last-Modified: Thu, 04 Feb 2010 12:29:04 GMT  
 Age: 28  
 X-Cache: HIT from sy32-21.sina.com.cn  
 Connection: close 
```

如果服务器端资源已经更新的话，就返回正常的响应。

### 4.1.3 持久连接

- HTTP Keep-Alive 简单说就是保持当前的TCP连接，避免了重新建立连接。
- HTTP 长连接不可能一直保持，例如 `Keep-Alive: timeout=5, max=100`，表示这个TCP通道可以保持5秒，max=100，表示这个长连接最多接收100次请求就断开。
- HTTP 是一个无状态协议，这意味着每个请求都是独立的，Keep-Alive 没能改变这个结果。另外，Keep-Alive也不能保证客户端和服务器之间的连接一定是活跃的，在 HTTP1.1 版本中也如此。唯一能保证的就是当连接被关闭时你能得到一个通知，所以不应该让程序依赖于 Keep-Alive 的保持连接特性，否则会有意想不到的后果。
- 使用长连接之后，客户端、服务端怎么知道本次传输结束呢？两部分：1. 判断传输数据是否达到了Content-Length 指示的大小；2. 动态生成的文件没有 Content-Length ，它是分块传输（chunked），这时候就要根据 chunked 编码来判断，chunked 编码的数据在最后有一个空 chunked 块，表明本次传输数据结束，详见[这里](http://www.cnblogs.com/skynet/archive/2010/12/11/1903347.html)。

### 4.1.4 post和get的区别

- 都包含请求头请求行，post多了请求body。
- get多用来查询，请求参数放在url中，不会对服务器上的内容产生作用。**post用来提交，如把账号密码放入body中。**
- GET是直接添加到URL后面的，直接就可以在URL中看到内容，而POST是放在报文内部的（二进制），用户无法直接看到。然而，从传输的角度来说，他们都是不安全的，因为 HTTP 在网络上是明文传输的，只要在网络节点上捉包，就能完整地获取数据报文。
- GET提交的数据长度是有限制的，因为URL长度有限制，具体的长度限制视浏览器而定。而POST没有。

## 4.2 响应报文组成

1. 状态行
2. 响应头(Response Header)
3. 响应正文

实例：

![](https://raw.githubusercontent.com/Connor4/Demo/master/%E7%AC%94%E8%AE%B0%E5%9B%BE%E7%89%87/http%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87%E7%BB%84%E6%88%90.jpg)

### 4.2.1 状态码

分类：

- 1XX- 信息型，服务器收到请求，需要请求者继续操作。
- 2XX- 成功型，请求成功收到，理解并处理。
- 3XX - 重定向，需要进一步的操作以完成请求。
- 4XX - 客户端错误，请求包含语法错误或无法完成请求。
- 5XX - 服务器错误，服务器在处理请求的过程中发生了错误。

常见状态码：

- `200 OK` 客户端请求成功
- `301 Moved Permanently` 请求永久重定向
- `302 Moved Temporarily` 请求临时重定向
- `304 Not Modified` 文件未修改，可以直接使用缓存的文件。
- `400 Bad Request` 由于客户端请求有语法错误，不能被服务器所理解。
- `401 Unauthorized` 请求未经授权。这个状态代码必须和WWW-Authenticate报头域一起使用
- `403 Forbidden` 服务器收到请求，但是拒绝提供服务。服务器通常会在响应正文中给出不提供服务的原因
- `404 Not Found` 请求的资源不存在，例如，输入了错误的URL
- `500 Internal Server Error` 服务器发生不可预期的错误，导致无法完成客户端的请求。
- `503 Service Unavailable` 服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常。

## 4.3 HTTP工作原理

HTTP协议定义Web客户端如何从Web服务器请求Web页面，以及服务器如何把Web页面传送给客户端。HTTP协议采用了请求/响应模型。客户端向服务器发送一个请求报文，请求报文包含请求的方法、URL、协议版本、请求头部和请求数据。服务器以一个状态行作为响应，响应的内容包括协议的版本、成功或者错误代码、服务器信息、响应头部和响应数据。

以下是 HTTP 请求/响应的步骤：

###### 1、客户端连接到Web服务器

一个HTTP客户端，通常是浏览器，与Web服务器的HTTP端口（默认为80）建立一个TCP套接字连接。

###### 2、发送HTTP请求

通过TCP套接字，客户端向Web服务器发送一个文本的请求报文，一个请求报文由请求行、请求头部、空行和请求数据4部分组成。

###### 3、服务器接受请求并返回HTTP响应

Web服务器解析请求，定位请求资源。服务器将资源复本写到TCP套接字，由客户端读取。一个响应由状态行、响应头部、空行和响应数据4部分组成。

###### 4、释放连接[TCP连接](https://www.jianshu.com/p/ef892323e68f)

若connection 模式为close，则服务器主动关闭[TCP连接](https://www.jianshu.com/p/ef892323e68f)，客户端被动关闭连接，释放[TCP连接](https://www.jianshu.com/p/ef892323e68f);若connection 模式为keepalive，则该连接会保持一段时间，在该时间内可以继续接收请求;

###### 5、客户端浏览器解析HTML内容

客户端浏览器首先解析状态行，查看表明请求是否成功的状态代码。然后解析每一个响应头，响应头告知以下为若干字节的HTML文档和文档的字符集。客户端浏览器读取响应数据HTML，根据HTML的语法对其进行格式化，并在浏览器窗口中显示。

例如：在浏览器地址栏键入URL，按下回车之后会经历以下流程：

1、浏览器向 DNS 服务器请求解析该 URL 中的域名所对应的 IP 地址;

2、解析出 IP 地址后，根据该 IP 地址和默认端口 80，和服务器建立[TCP连接](https://www.jianshu.com/p/ef892323e68f);

3、浏览器发出读取文件(URL 中域名后面部分对应的文件)的HTTP 请求，该请求报文作为 [TCP 三次握手](https://www.jianshu.com/p/ef892323e68f)的第三个报文的数据发送给服务器;

4、服务器对浏览器请求作出响应，并把对应的 html 文本发送给浏览器;

5、释放 [TCP连接](https://www.jianshu.com/p/ef892323e68f);

6、浏览器将该 html 文本并显示内容;

# 5. HTTPS

**一般http中存在如下问题：**

- 请求信息明文传输，容易被窃听截取。
- 数据的完整性未校验，容易被篡改
- 没有验证对方身份，存在冒充危险

为了解决上述HTTP存在的问题，就用到了HTTPS。

HTTPS 协议（HyperText Transfer Protocol over Secure Socket Layer）：一般理解为HTTP+SSL/TLS，通过 SSL证书来验证服务器的身份，并为浏览器和服务器之间的通信进行加密。



## 5.1 HTTPS SSL过程

① 证书验证阶段
(1)浏览器发起HTTPS请求，要求与Web服务器建立SSL连接
(2)服务端返回HTTPS证书：Web服务器收到客户端请求后，会生成一对公钥和私钥，并把公钥放在证书中发给客户端浏览器
(3)客户端验证证书是否合法，如果不合法则提示告警

② 数据传输阶段
(1)当证书验证合法后，在本地生成随机数
(2)通过公钥加密随机数，并把加密后的随机数传输到服务端
(3)服务端通过私钥对随机数进行解密
(4)服务端通过客户端传入的随机数构造对称加密算法，对返回结果内容进行加密后传输

**所以最终是客户端与服务端用随机数构造的对称加密算法来进行加密，不是使用密钥公钥。**

## 5.2 中间人攻击过程和原理

在通信中，双方每次在写完消息之后，计算消息的散列值，并用自己的私钥加密生成数字签名，附在信件后面，接收者在收到消息和数字签名之后，先计算散列值，再使用对方的公钥解密数字签名中的散列值，进行对比，如果一致，就可以确保该消息确实是来自于对方，并且没有被篡改过。

过程原理：
1.本地请求被劫持（如DNS劫持等），所有请求均发送到中间人的服务器
2.中间人服务器返回中间人自己的证书
3.客户端创建随机数，通过中间人证书的公钥对随机数加密后传送给中间人，然后凭随机数构造对称加密对传输内容进行加密传输
4.中间人因为拥有客户端的随机数，可以通过对称加密算法进行内容解密
5.中间人以客户端的请求内容再向正规网站发起请求
6.因为中间人与服务器的通信过程是合法的，正规网站通过建立的安全通道返回加密后的数据
7.中间人凭借与正规网站建立的对称加密算法对内容进行解密
8.中间人通过与客户端建立的对称加密算法对正规内容返回的数据进行加密传输
9.客户端通过与中间人建立的对称加密算法对返回结果数据进行解密
**由于缺少对证书的验证**，所以客户端虽然发起的是HTTPS请求，但客户端完全不知道自己的网络已被拦截，传输内容被中间人全部窃取。



# 面试题
## 1. HTTP 1.x和2区别

- 协议使用二进制格式非文本格式
- 完全多路复用，客户端可以同时发送多个请求或回应，而且不用按照顺序一一对应，避免“对头拥塞”
- HTTP协议是没有状态的，导致每次请求都会需要附上所有信息。所以，请求的很多头字段都是重复的，比如Cookie，一样的内容会浪费带宽也影响速度；2.0对此做了优化，引入了头信息压缩机制，一方面头信息用gzip或compress压缩后再发送；另一方面，客户端和服务器同时维护一张头信息表，所有字段都会存入这个表，产生一个索引号，之后只需要发送索引号。
- 2.0服务器可以主动向客户端推送资源。

## 2 HTTP 1.1跟1.0区别

**HTTP / 1.0 和HTTP / 1.1 的一些区别：**

- 缓存处理，在HTTP / 1.0 中，使用`header`里的 `If-Modified-Since, Expires` 来做为缓存判断的标准，HTTP / 1.1 则引入了更多的缓存控制策略例如 `Entity tag`，`If-Unmodified-Since`, `If-Match`, `If-None-Match`等更多可供选择的缓存头来控制缓存策略；
- 带宽优化以及网络连接的使用，HTTP / 1.0 中存在一些浪费的现象，例如客户端只是需要某个对象一部分，而服务器却将整个对象送过来了，并且不支持断点续传的功能，HTTP / 1.1则在请求头中引入了 range 头域，它允许只请求资源的某一个部分，即返回 206。 这样开发者可以自由选择以便充分利用带宽和链接；
- 错误通知的管理，在 HTTP/1.1 中新增了24个错误状态码；
- Host 头处理，在 HTTP/ 1.0 中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP/ 1.1 的请求消息和响应消息都应支持 Host 头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）;
- 长连接，HTTP / 1.1 支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个 TCP 连接上可以传送多个 HTTP 请求和响应，减少了建立和关闭连接的消耗和延迟，在 HTTP/ 1.1 中默认开启`Connection： keep-alive`，一定程度上弥补了 HTTP/1.0 每次请求都要创建连接的缺点。

# 参考

https://zhuanlan.zhihu.com/p/72616216



 