---
title:      websocket介绍与使用
date:       2020-08-26
categories:
    - 知识整理
tags:
    - WebSocket
    - 网络原理
excerpt:  websocket介绍与使用
---
# websocket介绍与使用

其实关于websocket是一直想要写一篇文章介绍的内容，刚好前一阵子面试的时候有聊到，发现自己了解的还是不够深入，这里打算从概念到实现介绍一遍。

## 概念

WebSocket是一种在html5开始提供的协议。它实现了在一个TCP连接上的全双工通信。

## 为什么需要？

这个问题我们可以看作它到底优于http协议什么，有了http协议为什么还要用ws？关键有两点。

1. 它构建了一个真正的持久化连接，而现今流行的http1.1，虽然有keep-alive来保持，但实际上只是把多个请求放到了一起来发，具体而言就是我们在发送的时候仍然是一个Request+一个Response，每个请求都需要有header，都是一个完整的http请求，其实并非真正的“长链接”；而ws连接在建立之后发送的请求就可以不需要header，也正因如此，它与原本的http协议有不同之处，所以需要另外的websocket协议来“打补丁”。
2. “全双工”意味着支持服务器推送。而在http只能由客户端主动发送请求才能接受服务器的信息，所以现在的解决方案都是通过轮询来做到这种效果，而使用ws就会方便很多。

## 与http的关系？

上面也提到过，ws像是对http的一个补丁，也就是说ws是基于http的（一部分握手借用http），但ws也并不包含http所有内容，借用知乎回答上的一张图：

![http&ws](https://pic3.zhimg.com/80/6651f2f811ec133b0e6d7e6d0e194b4c_720w.jpg?source=1940ef5c)

而如果有了解过http2的话会发现http2像是把ws融到http协议中了，未来这会是一种发展趋势。

## 协议具体？

### 创建连接

上面我们提到ws一部分握手借用了http，其实在建立连接和结束连接的时候我们使用的仍然是http连接，进行一次握手后打通连接。发送请求如下所示：

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```

服务器返回：

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

这里握手的主要目的就是告诉服务器需要切换ws协议，以及发送对应的密钥。

### 数据格式

WebSocket使用了自定义的二进制分帧格式，把每个应用消息切分成一或多个帧，发送到目的地之后再组装起来，等到接收到完整的消息后再通知接收端。帧格式如图：

![ws帧格式](https://pic1.zhimg.com/v2-91b11ad46dba68b87a3daa6c185479d8_b.png)

- FIN： 1 bit 。表示此帧是否是消息的最后帧，第一帧也可能是最后帧。
- RSV1，RSV2，RSV3： 各1 bit 。必须是0，除非协商了扩展定义了非0的意义。
- opcode：4 bit。表示被传输帧的类型：x0 表示一个后续帧；x1 表示一个文本帧；x2 表示一个二进制帧；x3-7 为以后的非控制帧保留；x8 表示一个连接关闭；x9 表示一个ping；xA 表示一个pong；xB-F 为以后的控制帧保留。
- Mask： 1 bit。表示净荷是否有掩码（只适用于客户端发送给服务器的消息）。
- Payload length： 7 bit, 7 + 16 bit, 7 + 64 bit。 净荷长度由可变长度字段表示： 如果是 0~125，就是净荷长度；如果是 126，则接下来 2 字节表示的 16 位无符号整数才是这一帧的长度； 如果是 127，则接下来 8 字节表示的 64 位无符号整数才是这一帧的长度。
- Masking-key：0或4 Byte。 用于给净荷加掩护，客户端到服务器标记。
- Extension data： x Byte。默认为0 Byte，除非协商了扩展。
- Application data： y Byte。 在”Extension data”之后，占据了帧的剩余部分。
- Payload data： (x + y) Byte。”extension data” 后接 “application data”。

### 结束连接

通过客户端发送结束帧来提醒服务器已结束连接，通过opcode来标识帧类型：

- 关闭：操作码为0x8。关闭帧可能包含一个主体（帧的应用数据部分）指明关闭的原因，如终端关闭，终端接收到的帧太大，或终端接收到的帧不符合终端的预期格式。从客户端发送到服务器的关闭帧必须标记，在发送关闭帧后，应用程序必须不再发送任何数据。如果终端接收到一个关闭帧，且先前没有发送关闭帧，终端必须发送一个关闭帧作为响应。终端可能延迟发送关闭帧，直到它的当前消息发送完成。在发送和接收到关闭消息后，终端认为WebSocket连接已关闭，必须关闭底层的TCP连接。服务器必须立即关闭底层的TCP连接；客户端应该等待服务器关闭连接，但并非必须等到接收关闭消息后才关闭，如果它在合理的时间间隔内没有收到反馈，也可以将TCP关闭。如果客户端和服务器同时发送关闭消息，两端都已发送和接收到关闭消息，应该认为WebSocket连接已关闭，并关闭底层TCP连接。
- Ping：操作码为0x9。一个Ping帧可能包含应用程序数据。当接收到Ping帧，终端必须发送一个Pong帧响应，除非它已经接收到一个关闭帧。它应该尽快返回Pong帧作为响应。终端可能在连接建立后、关闭前的任意时间内发送Ping帧。注意：Ping帧可作为keepalive或作为验证远程终端是否可响应的手段。
- Pong：操作码为0xA。Pong 帧必须包含与被响应Ping帧的应用程序数据完全相同的数据。如果终端接收到一个Ping 帧，且还没有对之前的Ping帧发送Pong 响应，终端可能选择发送一个Pong 帧给最近处理的Ping帧。一个Pong 帧可能被主动发送，这作为单向心跳。对主动发送的Pong 帧的响应是不希望的。

## 具体实现？

下面以js和nodejs为例来看看前后端如何实现一个简单的即时聊天室。**具体的代码可以看[我的github仓库](https://github.com/YokingYoking/ws-chat)**，下面聊聊大致的思路。

http5有原生的websocket api，但在搜索资料的过程中发现有一个封装好的库socket.io供前后端来使用，恰好后端我也打算使用nodejs，会比较方便，故选择使用这个库，具体可以看[官方网站](https://socket.io/)。这里会用到的api基本上就是socket.on监听发送的事件和消息内容以及socket.emit发送一个事件以及数据，客户端和服务端是雷同的，这也很好理解，一个发一个收。

首先明确我们要做的这个聊天室大致有什么功能：

1. 客户端打开后自动进行连接，成功连接后客户端有提示，服务端发送消息告诉每个用户某个用户已上线；
2. 这样就需要一个用户名，笔者这里为了方便实现直接使用socket连接随机分配的id来代替了；
3. 用户端发送信息，服务端接收到之后广播出去。

### 前端实现

基本界面分为三部分，一个固定header标题，一个聊天内容的主体，一个footer装载输入框和发送按钮。

功能方面：

- js部分在创建socket对象的同时就会对目标url进行自动连接，用socket.on监听连接事件，若成功则发送一个alert提醒；
- 同时也监听message事件，当服务端有消息发过来的时候，接收信息，对于聊天界面来说，由于聊天内容是动态更新的，因此需要在js中动态创建一个DOM并append上去；
- 给发送按钮绑定事件，用socket.emit一个message事件到服务端，发送后输入框清空。
- 由于这里只做一个简单实现，其他的复杂交互提示就不加上去了。

### 后端实现

使用nodejs + Express，解决跨域加上cors中间件。

后端的处理就更简单了，监听connection事件和message事件，有用户连接和消息发送就广播出去（带上用户名），相当于做一个中转。

## 总结

日常开发中可能还是以http为主，但是ws是一个必须知道的知识点，某些场合下使用它还是很方便的，像demo中实现一个即时聊天室也十分有趣。

## 参考链接

[WebSocket 是什么原理？为什么可以实现持久连接？](https://www.zhihu.com/question/20215561)
[WebSocket 数据帧](https://blog.csdn.net/p312011150/article/details/79758068)
