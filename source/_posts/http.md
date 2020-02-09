---
title: HTTP 超文本运输协议（HyperText Transfer Protocol）
date: 2019-06-12 22:47:39
tags: HTTP
categories: TCP/IP
---

**用于客户端和服务端请求和响应超文本信息，请求访问资源的一端称之为客户端，响应返回资源的一端称之为服务端且都是通信过程中的应用进程。属于TCP/IP协议族的子集，处于TCP/IP五层网络模型的应用层。**

**请求报文是由请求方法、请求URI、协议版本、可选的请求首部字段和内容实体组成。**

## URI、URL、URN的区别
- URI（Uniform Resource Identifier，统一资源标识符)
- URL (Uniform Resource Locator，统一资源定位符
- URN（Uniform Resource Name，统一资源名称）

![uri](https://ruixiaojia-blog.oss-cn-hangzhou.aliyuncs.com/blog/http/uri.png?x-oss-process=style/compression)

### URL
![url](https://ruixiaojia-blog.oss-cn-hangzhou.aliyuncs.com/blog/http/url.png?x-oss-process=style/compression)

- protocol 协议部分 : 代表该网页使用的是HTTP协议，"//"为分隔符
- userinfo 用户信息 : 用于填写一些用户相关的信息，比如可能会填写"user:password"(不建议用于填写用户名及密码)
- host 域名部分 : 也可以使用IP地址作为域名使用
- port 端口部分 : 域名和端口之间使用“:”作为分隔符。端口不是一个URL必须的部分，如果省略端口部分，将采用默认端口80
- path 虚拟目录 : 定位主机上的具体文件位置。
- query 查询字符串 : 为请求提供参数，用&连接的key=value键值对
- fragment 位置标示 : 请求中不需要

## TCP/IP五层网络模型

** * 本文只对TCP/IP网络模型作简单说明，网络连接中的具体过程待我二刷《计算机网络》后详细讲解。**

| TCP/IP网络模型 | 协议 | 作用 |
| --- | --- | --- |
| 应用层 | DNS、HTTP、FTP、SMTP、SNMP| 决定了向用户提供应用服务时的通信活动 |
| 运输层 | TCP、UDP（建立连接、数据传送、连接释放） | 向应用层提供通信服务，属于面向通信部分的最高层，面向用户的最低层 |
| 网络层 | IP（ARP、ICMP、IGMP）、路由（RIP、OSPF）IPv4、IPv6、VPN | 向上层只提供简单灵活、无链接、尽最大努力交付的数据服务 |
| 数据链路层 | PPP、CSMA/CD、点对点信道、广播信道 | 将网络层交下来的数据构成**帧**发送到链路上，以及把接收到的**帧**取出交给网络层|
| 物理层  | 信道（单工、半双工、全双工） | 在计算机的传输媒体上传输比特流，确定与传输媒体的接口特性 |
| 物理介质 | 双绞线、光纤、同轴电缆 | 光信号或电信号 |

![tcp/ip](https://ruixiaojia-blog.oss-cn-hangzhou.aliyuncs.com/blog/http/tcp%3Aip.png?x-oss-process=style/compression)

**客户端在发送数据时，每经过一层都会打上一个该层所属的首部信息，反之，服务端在接收数据时，每经过一层会取消对等层的首部信息。**

这一复杂的过程对用户来说是透明的，就好像客户端直接将信息发送给了服务端的应用进程。同理，任何两个对等层之间也像虚线那样把数据直接传递给对方。

- **协议是“水平的”**，即控制对等层实体之间通信的规则。
- **服务是“垂直的”**，即服务是由下层向上层通过接口提供的。

> 类似于生活中收发快递，为了确保你所寄出去的邮件不会在运输的过程中损坏或被更换内容，你需要加上一层包装并在包装上做上记号，快递公司在运输之前也会加上一层包装并且填写上对应的地址，以防止运输过程中出现损坏或丢失，在快递运输的过程中经过哪些中转站由哪些快递员处理则是不需要知道的，收件方只需要在收到快递后需要确保包装完好、标记准确则可以确认收到的东西为正确的。

## HTTP请求的方法

- GET : 请求指定资源，只用于获取数据。
- POST : 发送数据给服务器. 请求主体的类型由 Content-Type 首部指定。
- HEAD : 和GET方法一样，但是只返回报文首部信息，用与确认URI的有效性,使用场景是在下载一个大文件之前先获取其大小再决定下载，节约带宽资源。
- OPTIONS : 用来查询指定的URI所支持的请求方法，可以发送一个预检请求，来检测实际请求能否被服务器接受。
- PUT : 用来传输文件，如果指定的URI上的资源已经在源服务器上存在，则更新文件内容并返回200，如果指定资源不存在，则根据URI的标识定义一个新资源并返回201。该方法不带任何验证机制，有很大的安全隐患。
- DELETE : 用来删除指定URI的文件，如果请求删除成功则返回200，如果请求删除的资源不存在则返回204，和PUT方法一样，不推荐使用。
- CONNECT : 要求使用隧道协议来连接代理服务器，让代理服务器去请求目标服务器并将数据返回给用户，说白了就是翻墙。可以用来访问使用了SSL（https）协议的站点。
- TRACE : 用来确认连接过程中发生的一系列操作，会将web服务器之前的请求通信返回给客户端。

**幂等 idempotent** : 方法执行一次或多次的效果是一样的，即没有副作用。POST、CONNECT方法为非幂等。

### * GET、POST请求的区别
一般我们在讨论GET、POST的区别包括面试的时候，讨论的都是specification而并不是implementation。看了很多很多关于GET、POST区别的文章几乎都在说implementation，包括[w3school](http://www.w3school.com.cn/tags/html_ref_httpmethods.asp)给出的答案也非常的粗浅，然鹅这些并不是我想要的答案。

- GET、POST都有各自的语义，GET是用来获取数据，POST是用来发送数据。方法是否安全、幂等、可缓存都需要根据客户端、服务端的具体操作去判断，协议中的规定不等于实现。总之在使用时必须考虑其语义，做出符合语义的行为。

> 推荐两篇不错的文章
> [HTTP协议中GET和POST方法的区别](https://sunshinevvv.coding.me/blog/2017/02/09/HttpGETv.s.POST/)
> [听说『99% 的人都理解错了 HTTP 中 GET 与 POST 的区别』？？](https://zhuanlan.zhihu.com/p/25028045)

## HTTP状态码 RFC7231
** http状态码的职责就是当客户端向服务端发起请求时，描述此次请求返回的结果，通过状态吗可以知道服务端是否正常处理请求，通知出现的错误。**

|  | 类别 | 描述 |
| --- | --- | --- |
| 1XX | Informational | 信息，接受的请求正在处理 |
| 2XX | Successful | 成功，操作被成功接收并处理 |
| 3XX | Redirection | 重定向，需要进一步的操作以完成请求 |
| 4XX | Client Error | 客户端错误，服务器无法完成请求 |
| 5XX | Server Error | 服务端错误，服务端处理请求出错 |

**200 ok** 请求已成功，响应的实体内容取决于请求方法。

**204 No Content** 服务器已成功处理请求，但是返回的实体主体中没有内容，用于只需要服务端通知是否成功，而不需要返回任何内容的情况，例如：保存。

**206 Partial Content** 服务器成功完成对目标资源的范围请求，响应请求Range中表示的一个或多个部分

**301 Moved Permanently** 目标资源及其资源的引用已被永久分配到一个新的URI，服务端响应Location字段跳转至新的URI，浏览器将会对此跳转进行缓存，使用此状态码需谨慎。

**302 Found** 目标资源已找到，但是存放在其它的URI下，由于历史原因，用户代理可将POST改为GET用于后续请求。

**303 See Other** 通常作为 PUT 或 POST 操作的返回结果，它表示重定向链接指向的不是新上传的资源，而是另外一个页面。而请求重定向页面的方法要必须使用 GET。

**307 Temporary Redirect** 目标资源临时存放在不同的URI下，如果用户代理执行自动重定向到该URI，则不得更改请求方法，因为POST方法不是幂等的，返回值可能和用户所期望的不同。与302相似，但不允许将POST改为GET。

**308 Permanent Redirect** 同307。与301相似，但不允许将POST改为GET。

**304 Not Modified** 已接收到GET或HEAD请求，如果不存在请求条件则返回200，否则根据此请求的条件If-Modified-Since或If-None-Match的值来判断客户端的资源是否是最新的，如果是则返回304且实体主体为空，否则返回200且实体主体为最新的内容。
一般来说if-modified-since的值来自于上一次这条请求的Response中的Last-Modified，if-none-match的值来自于上一次这条请求的Response中的Etag

**400 Bad Request** 客户端错误，请求的合适或语法错误导致请求无效。

**403 Forbidden** 服务端已理解请求，但拒绝授权。客户端访问权限不足。

**404 Not Found** 源服务器未找到目标资源或不愿透露目标资源。但不表示这种资源缺少是永久性的。

**410 Gone** 源服务器上不再可以访问该资源，且是永久性的。

**500 Internal Server Error** 服务器遇到意外的情况，无法完成请求。

**503 Service Unavailable** 服务器当前由于过载或维护无法完成请求。

## HTTP首部
首部的中分别存放客户端和服务端处理请求和响应请求所需要的信息。
本章只介绍几个常用的字段和指令，如想深入了解请出门右拐。➡️
[MDN HTTP headers](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)
[RFC2616](https://tools.ietf.org/html/rfc2616)

### HTTP通用首部字段 General
可以应用于请求和响应中的，但不能用于实体主体。

#### Cache-Control
控制缓存，缓存指令是单向的，这意味着在请求中设置的指令，不一定被包含在响应中。
- public：响应可以被任何用户或对象缓存（客户端、代理服务器等）。
- private：响应只对特定的用户或对象缓存，代理服务器不可缓存。
- no-cache：不缓存过期资源，在处理资源之前会强制向源服务器发起验证。
- no-store：不缓存任何内容。
- max-age：设置缓存的最大周期（秒）。
- s-maxage：覆盖max-age，设置代理服务器缓存的最大周期（秒）。
- no-transform：不得对资源进行转换或转变。如：非透明代理可能将图片进行格式转换或压缩。

#### Connection
控制不再转发给代理的首部字段、管理持久连接。
- keep-alive：TCP长连接，保持连接（HTTP1.1版本以前默认非持久连接，需要加上该指令。chrome单次并发连接数为6）。
- close：客户端或服务端想要关闭该网络连接。

#### Date
创建HTTP报文的日期和时间

#### Pragma
历史遗留字段，仅用于HTTP1.0版本向后兼容，与Cache-Control:no-cache效果一致

#### Traier
Trailer是一个响应首部，允许发送方在分块发送的消息后面添加额外的元信息。

#### Transfer-Encoding
规定传输实体主体时采用的编码方式。
chunked、compress、deflate、gzip、identity(不压缩)

#### Upgrade
切换协议，使用此字段时还需指定Connection：upgrade，服务端可返回101 Switching Protocols

#### Via
由代理服务器添加，适用于正向代理、反向代理。用于追踪报文的传输路径、防止循环请求。

### HTTP请求首部 Request Headers

#### Accept
客户端能够响应的媒体类型([MIME types](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types))及对应的优先级，q代表权重。      

    accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,\*/\*;q=0.8,application/signed-exchange;v=b3

#### Accept-Charset
客户端告诉服务端能够处理的字符集。q代表权重。

#### Accept-Encoding
客户端能够理解的内容编码方式。

    accept-encoding: gzip, deflate, br
br：Brotli

#### Accept-Language
客户端可以理解的自然语言集。q代表权重。
    
    accept-language: zh-CN,zh;q=0.9

#### Range
范围请求，告诉服务端返回文件的哪一部分。若成功则返回206 Partial Content，失败返回416 Range Not Satisfiable。

#### 条件请求
服务端在接收到此类条件后，只有判断条件满足时才会返回资源。
- If-Match ==> ETag
在GET和HEAD请求的情况下，请求的资源满足列出的ETag之一时才返回资源。
-If-None-Match ==> ETag
与If-Match作用相反，在不满足ETag时返回资源。与If-Modified-Since同时出现时优先级较高。

- If-Modified-Since ==> Last-Modified
如果服务端在指定日期之后有更新过资源则返回资源并且状态码为200，否则返回304 Not Modified，并在Last-modified上带上上次修改的日期。
- If-Unmodified-Since ==> Last-Modified
与If-Modified-Since作用相反，服务器在指定日期之后没有更新过资源才会返回。否则返回401

- If-Range
使Range头字段在一定条件下才会起作用。即可使用Last-Modified时间值验证，也可使用ETag标签来作为验证，但不能同时使用。
一般用于断点续传的过程中，验证资源是否发生改变。

- Referrer
包含了当前请求页面的来源页面的地址<URL>，即上一跳地址。一般用于统计访问来源。

[内容协商机制](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Content_negotiation)

### HTTP响应首部 Response Headers

#### Accept-Range
返回指令bytes则表示支持范围请求，none则不支持，等同于不返回该头部

#### Age
表示该信息在缓存中存贮的时间，以秒为单位

#### ETag
可用字符串对资源做唯一标示，当资源更新是ETag的值也需要更新，类似于指纹。
- 强ETag：不管实体发生多细微的变化都会改变值，如：ETag："value"
- 弱ETag：弱验证器，只有当实体主体发生根本的改变时才会改变值，如：ETag："W/value"; (W大写)

#### Location、Content-Location
Location：需要将页面重定向到新的地址，Location：<url>
Content-Location：报文主体返回对应资源经过内容协商后的URL

Location 与 Content-Location是不同的，前者（Location ）指定的是一个重定向请求的目的地址（或者新创建的文件的URL），而后者（ Content-Location） 指向的是可供访问的资源的直接地址，不需要进行进一步的内容协商。Location 对应的是响应，而Content-Location对应的是要返回的实体。

#### Vary
仅在Vary字段中指定字段的值相同时返回缓存，否则需要向源服务器重新获取资源。

#### Allow
枚举资源所支持的HTTP方法的集合

#### Expires
时间戳，在指定时间之后，响应过期，即资源失效日期。

#### Last-Modified
资源的修改日期。<week>, <day> <month> <year> <hour>:<minute>:<second> GMT

## HTTPS （HTTP over SSL）
** HTTP协议中没有加密机制，但是可以通过和SSL(安全套接层)、TLS(安全传输层协议，包括TLS记录协议和TLS握手协议，实际上TLS就是标准化的SSL3.0)组合使用达到加密HTTP的效果。**

互联网不是某个国家、组织、个人的私有物，在这种中间极有可能遭到恶意窥视。
HTTP协议是一个很单纯的协议，它只会傻傻的去请求并、响应，因此就有人利用它的傻白甜做一些不好的事情。
由于通信内容是未加密的，即使在通信的过程中使用了POST请求，但数据包在网络中是流动的，其内容很容易被一些抓包工具嗅探到，就会出现以下几种问题。

- 明文通信，内容可能被窃听（信息泄露）。
- 无法确保报文完整性，内容可能遭篡改（中间人攻击 MITM）。
- 不验证通信对方身份，身份遭伪装，请求和响应的客户端或服务器不是真实的，无法阻止海量请求（DDOS）。

> [HTTPS的DDoS攻击防护思路](http://blog.nsfocus.net/https-ddos-attack-defense-ideas/)

HTTP明文通信
![HTTP](https://ruixiaojia-blog.oss-cn-hangzhou.aliyuncs.com/blog/http/http.png?x-oss-process=style/compression)

TLS加密通信
![TLS](https://ruixiaojia-blog.oss-cn-hangzhou.aliyuncs.com/blog/http/TLS.png?x-oss-process=style/compression)

** 通常HTTP是直接和TCP通信的，SSL是独立的协议，处于HTTP之下TCP之上，由SSL来加密、认证传输内容。实际上HTTPS就是添加了身份验证、报文加密、报文完整性校验的HTTP，也可以理解为身披SSL的HTTP。**

** HTTPS采用对称加密和非对称加密，两种加密方式并用的混合加密机制，在交换秘钥环节使用对称加密，报文交换阶段使用非对称加密方式，交换秘钥环节使用对称加密很容易遭到中间人攻击（MITM）至秘钥泄露或者被替换，这时候就要用到证书，证书会对该公钥进行数字签名，并将这个已签名的公钥和证书绑定在一起，以保证非对称加密过程的安全。**

** 加密解密的过程会消耗CPU及内存资源，SSL通信也会占用部分网络资源。**

## HTTP的特点

- 一次链接只能发送一个请求
- 请求只能从客户端开始
- 首部未压缩且每次都要发送相同首部、未强制压缩
- 无法证明报文的完整性

### 灵活
简单、快速。
允许传输任意类型的数据对象。
客户端发起请求，服务端响应。

SPDY：基于TCP的会话层协议，多路复用、请求优先级、报头压缩、服务器推送，已被HTTP/2取代

### 无状态
是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。

cookie：客户端记录信息
session：服务端记录客户端状态

[理解cookie和session机制](https://www.cnblogs.com/andy-zhou/p/5360107.html)

### 无连接
每次连接只处理一个请求，服务端处理完请求并受到客户端应答后，即断开连接。

keep-alive：默认保持长连接。
WebSocket：在单个TCP连接上进行全双工通讯的协议。

[深入理解HTTP协议的特点](https://www.cnblogs.com/xuxinstyle/p/9813654.html)

#### 通信方式：
- 单工通信：只能有一个方向的通信，没有反方向交互。即只能我发给你，不能你发给我。
- 半双工通信：通信的双方都可以发送信息，但双方不能同时发送。
- 全双工通信：通信双方可以同时发送和接收信息。

