---
title: 关于抓包（packet capture）
date: 2019-05-15 23:13:17
tags: Packet Capture
categories: HTTP
---

看标题也知道咯，此篇分享呢主要是向大家分享一些我在日常工作中使用到的一些工具以及如何去使用它帮助我们解决开发中截获、编辑、分析网络数据包的问题。

### Proxy SwitchyOmega
在开始之前呢先分享一个chrome的拓展程序 Proxy SwitchyOmega。非常轻量级的一个应用程序，可以轻松快捷地管理和切换多个代理设置，操作之简便，使用之方便。

首先下载它 — 进入程序 — 点击新建情景模式 — 选择代理的协议、服务器、端口 — 点击工具栏的图标选择对应的情景模式就可以了。
![proxy-switchyOmega](2019/05/15/packet-capture/proxy.png)

***

## Charles
Charles其实是一款代理服务器，通过将自己设置成系统（电脑或者浏览器）的网络访问代理服务器，然后截取请求和请求结果达到分析抓包的目的。该软件是用Java写的，能够在Windows，Mac，Linux上使用,但是它是收费的，只能免费使用30分钟，超时必须重新打开，自行查找破解版。

### Charles的主要功能:
- 截取Http 和 Https 网络封包。
- 重发网络请求，方便后端调试。
- 修改网络请求参数。
- 网络请求的截获并动态修改。
- 模拟慢速网络。

### Charles抓包:
这时候就需要使用到Proxy SwitchyOmega了，将其打开，添加情景模式，将服务器设置为127.0.0.1，代理端口为8888，当然也可以将端口设置为其他，需要在charles工具栏中点击poxy — 勾选macOS Proxy/windows Proxy — Proxy Settings — 更改端口号后保存即可。
![charles](2019/05/15/packet-capture/charles1.png)

### Charles抓取https:
![charles](2019/05/15/packet-capture/charles2.png)
打开charles可以看到上图中左边的红框里面已经抓到一些数据了，但是https前面多了一把锁，表示https是没有抓到数据包的，这时候你的电脑需要装charles的证书，同时要信任，且charles要开启https代理。

- 开启https代理如下图所示：
其中*:* 代表所有域名和协议下面所有的端口
![charles](2019/05/15/packet-capture/charles3.png)

- 电脑安装证书：
![charles](2019/05/15/packet-capture/charles4.png)

- 点击install Charles Root Certificate后会在你的系统钥匙串里面装一个证书，但证书是不被信任的，此时我们要信任此证书。
![charles](2019/05/15/packet-capture/charles5.png)

- 手机安装证书:（以iPhone为例）
1、确保手机和电脑在同一局域网内
2、打开手机设置 — 选择已连接的无线网络 — 找到配置代理 — 填写代理服务器为电脑的IP,端口为8888
3、然后在手机浏览器里面输入chls.pro/ssl这时候会提示你下载描述文件，然后安装，iPhone手机记得在关于本机里面信任证书，安卓手机，如果用原生的浏览器装不上，那就下个Chrome在Chrome里面打开chls.pro/ssl这个地址

***

## Whistle
基于Node实现的跨平台web调试代理工具,所以下载前首先确保你已安装node。功能之强大，操作之简便，比较适合前端开发同学使用的工具，也是我个人比较喜欢的一款工具。
官文比较详细，还配有中文版的教程，如何下载使用我这里就不赘述了,直接附上地址。
[github地址](https://github.com/avwo/whistle)
[中文文档](http://wproxy.org/whistle/)

***

## Wireshark
我们先将其设置为中文，这样的话使用会比较方便
![wireshark](2019/05/15/packet-capture/wireshark1.png)

选择对应的网卡后进入主界面
![wireshark](2019/05/15/packet-capture/wireshark2.jpg)

这个时候你已经可以看到界面刷刷刷不停的在加载了，我的马鸭怎么回事😱
好了不要慌。。。点左上角的红色方块就可以让它停下来。

### 对应网络层级
点击分组后我们可以看到下方的封包详细信息，它对应TCP/IP五层网络模型中的每一层，以HTTP为例：
![wireshark](2019/05/15/packet-capture/wireshark3.jpg)

### TCP包内容
![wireshark](2019/05/15/packet-capture/wireshark4.jpg)

### TCP建立连接（三次握手🤝）
![wireshark](2019/05/15/packet-capture/tcp1.jpg)
我们可以看到这次TCP连接的完整过程，连接完成后建立TLS隧道连接。
此例子中30.38.36.123为客户端，52.231.18.241为服务端。

![wireshark](2019/05/15/packet-capture/tcp2.jpg)
客户端向服务端发出请求连接的报文，其中同步位SYN置为1，并选择了序号seq为x，TCP规定SYN报文段（即SYN为1的报文）不能携带数据（可以看到LEN=0），并且需要消耗一个序号,此时客户端进入SYN-SENT同步已发送状态。

![wireshark](2019/05/15/packet-capture/tcp3.jpg)
服务端在收到请求报文段后如果同意连接，则向客户端发送确认。在确认报文段中将SYN、ACK都置为1，确认号Ack为x+1，同时也为自己选择一个序号y（客户端和服务端的序号不相同），此报文段也不携带数据且也需要消耗一个序号。此时服务器进入SYN-RCVD同步收到状态。

![wireshark](2019/05/15/packet-capture/tcp4.jpg)
客户端在收到服务端的确认后，还要向服务端再次发出确认，将确认ACK置为1，确认号Ack为y+1，同时将自己的序号seq置为x+1。此时客户端进入ESTABLISHED已建立连接状态。
服务端在收到确认后也进入ESTABLISHED状态。

**序号seq** ： 本段报文中所发送的数据的第一个字节的序号，例如：本段报文的序号为0，本段报文长度100字节，下段报文的seq就为101。
**确认号Ack** ： 期望收到对方下一报文段的第一个数据字节的序号。仅在控制位中确认ACK为1时确认号才有效。
**确认ACK** ： 在连接建立后所有传送的报文段都必须将ACK置为1。
**同步SYN** ： 在建立连接时用来同步序号。TCP规定SYN报文段不得携带数据。

### 过滤表达式
使用正确的表达式可以快速的从茫茫数据中找到你想要的那一堆。
- 逻辑运算符 ： and、or
- http ：只看http协议或其它协议名
- ip.src == 127.0.0.1 ： 源IP地址为127.0.0.1的封包
- ip.dst == 0.0.0.0 ： 目标地址为0.0.0.0的封包
- tcp.port == 80 ：端口为80的
- tcp.srcport == 443 ：源端口为443的
- http.request.method == "GET" ：只显示GET方法的请求。


> 分享两篇文章，如果想深入学习Wireshark可以看看
> http://www.cnblogs.com/TankXiao/archive/2012/10/10/2711777.html
> https://www.cnblogs.com/dragonir/p/6219541.html