---
title: "网络知识点总结"
date: 2023-04-10T15:58:50+08:00
draft: false
categories: ["移动端"]
tags: ["HTTP"]
---

## 一、网络分层
网络分层就是将网络节点所要完成的数据的发送或转发、打包或拆包，以及控制信息的加载或拆出等 工作，分别由不同的硬件和软件模块来完成。这样可以将通信和网络互联这一复杂的问题变得较为简单。 网络分层有不同的模型，有的模型分7层，有的模型分5层。这里介绍分5层的，因为它更好理解。
![](/images/http_1.webp)

### 物理层

**该层负责比特流在节点间的传输，即负责物理传输**。该层的协议既与链路有关，也与传输介质有关。 其通俗来讲就是把计算机连接起来的物理手段。

### 数据链路层

**该层控制网络层与物理层之间的通信，其主要功能是如何在不可靠的物理线路上进行数据的可靠传递**。为了保证传输，从网络层接收到的数据被分割成特定的可被物理层传输的帧。
帧是用来移动数据的结构包，它不仅包括原始数据，还包括发送方和接收方的物理地址以及纠错和控制信息。其中的地址确定了帧将发送到何处，而纠错和控制信息则确保帧无差错到达。如果在传送数据时，接收点检测到所传数据中有差错，就要通知发送方重发这一帧。

### 网络层

**该层决定如何将数据从发送方路由到接收方**。网络层通过综合考虑发送优先权、网络拥塞程度、服务质量以及可选路由的花费来决定从一个网络中的节点 A 到另一个网络中节点 B 的最佳路径。

### 传输层

**该层为两台主机上的应用程序提供端到端的通信**。相比之下，网络层的功能是建立主机到主机的通信。传输层有两个传输协议：**TCP（传输控制协议）和UDP（用户数据报协议）**。其中，TCP是一个可靠的面向连接的协议，UDP是不可靠的或者说无连接的协议。

### 应用层

应用程序收到传输层的数据后，接下来就要进行解读。解读必须事先规定好格式，而应用层就是规定应用程序的数据格式的。它的主要协议有HTTP、FTP、Telnet、SMTP、POP3等。

## 二、TCP与UDP的区别

区别|TCP|UDP
--|--|--
连接|需要|不需要
速度|慢|快
占用资源|多|少
可靠性|可靠|不可靠
稳定性|稳定|不稳定
有序性|有序|不保证有序
准确性|准确无差错|可能存在丢包的情况

### 应用场景

TCP:文件传输、邮件、远程登录、数据库、分布式系统的数据传输
UDP:即时通讯、网络视频、网络语音电话、内部系统

## 三、TCP连接的建立流程

### TCP传输流程

通常我们进行HTTP连接网络的时候会进行TCP的三次握手，然后传输数据，之后再释放连接。

![](/images/http_2.webp)

### TCP3次握手流程

* 第一次握手：建立连接。客户端发送连接请求报文段，将 SYN 设置为 1、Sequence Number（seq）为 x；接下来客户端进入SYN_SENT状态，等待服务端的确认。
* 第二次握手：服务器收到客户端的 SYN 报文段，对 SYN 报文段进行确认，设置Acknowledgment Number（ACK）为 x+1（seq+1）；同时自己还要发送 SYN 请求信息，将SYN设置为1、seq为y。服务端将 上述所有信息放到SYN+ACK报文段中，一并发送给客户端，此时服务端进入SYN_RCVD状态。
* 第三次握手：客户端收到服务端的SYN+ACK报文段；然后将ACK设置为y+1，向服务端发送ACK报 文段，这个报文段发送完毕后，客户端和服务端都进入ESTABLISHED （TCP连接成功）状态，完成TCP的 三次握手。

### TCP4次挥手

当客户端和服务端通过三次握手建立了TCP连接以后，当数据传送完毕，断开连接时就需要进行TCP的 四次挥手。其四次挥手如下所示。
* 第一次挥手：客户端设置seq和ACK，向服务端发送一个FIN报文段。此时，客户端进入FIN_WAIT_1 状态，表示客户端没有数据要发送给服务端了
* 第二次挥手：服务端收到了客户端发送的FIN报文段，向客户端回了一个ACK报文段。
* 第三次挥手：服务端向客户端发送 FIN 报文段，请求关闭连接，同时服务端进入LAST_ACK状态。
* 第四次挥手：客户端收到服务端发送的FIN报文段，向服务端发送ACK报文段，然后客户端进入 TIME_WAIT状态。服务端收到客户端的ACK报文段以后，就关闭连接。此时，客户端等待2MSL（最大报 文段生存时间）后依然没有收到回复，则说明服务端已正常关闭，这样客户端也可以关闭连接了。

![](/images/http_3.webp)

## 四、HTTP协议

HTTP协议是基于TCP/IP协议之上的应用层协议，有以下几个特点

* **基于 请求-响应 的模式**
  HTTP协议规定,请求从客户端发出,最后服务器端响应该请求并 返回。换句话说,肯定是先从客户端开始建立通信的,服务器端在没有 接收到请求之前不会发送响应
  ![](/images/http_4.webp)

* **无状态保存**
  HTTP是一种不保存状态,即无状态(stateless)协议。HTTP协议 自身不对请求和响应之间的通信状态进行保存。也就是说在HTTP这个 级别,协议对于发送过的请求或响应都不做持久化处理。
  ![](/images/http_5.webp)

* **无连接**
  无连接的含义是**限制每次连接只处理一个请求**。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间，并且可以提高并发性能，不能和每个用户建立长久的连接，请求一次相应一次，服务端和客户端就中断了。
  但是无连接有两种方式，早期的http协议是一个请求一个响应之后，直接就断开了，但是现在的**http协议1.1版本不是直接就断开了**，而是等几秒钟，这几秒钟是等什么呢，等着用户有后续的操作，如果用户在这几秒钟之内有新的请求，那么还是通过之前的连接通道来收发消息，如果过了这几秒钟用户没有发送新的请求，那么就会断开连接，这样可以提高效率，减少短时间内建立连接的次数，因为建立连接也是耗时的。这边的等待时间是可以通过后端代码控制的。
  ![](/images/http_6.webp)

### HTTP状态码

状态码|类别|原因短语
--|--|--
1XX|指示信息|收到请求，需要请求者继续执行操作
2XX|请求成功|请求已被成功接收并处理
3XX|重定向|要完成请求必须进行更进一步的操作
4XX|客户端错误|请求有语法错误或请求无法实现
5XX|服务器错误|服务器不能实现合法的请求

### HTTP请求报文
HTTP 报文是面向文本的，报文中的每一个字段都是一些ASCII码串，各个字段的长度是不确定的。一 般一个HTTP请求报文由请求行、请求报头、空行和请求数据4个部分组成

![](/images/http_7.webp)
![](/images/http_8.webp)

### HTTP相应报文
HTTP 的响应报文由状态行、响应报头、空行、响应正文组成。
![](/images/http_9.webp)
![](/images/http_10.webp)

### HTTP协议与HTTPS协议的主要区别

区别|HTTP|HTTPS
--|--|--
CA证书|不需要|需要，进行连接时需要身份认证
传输协议|超文本传输协议，信息是明文传输|具有安全性的ssl/tls加密传输协议
端口|80|443

HTTPS是基于SSL/TLS协议安全层上的HTTP协议！

![](/images/http_11.webp)

下面是SSL握手的主要流程!

![](/images/http_12.webp)
