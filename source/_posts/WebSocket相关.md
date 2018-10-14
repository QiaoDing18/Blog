title: WebSocket相关
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-04-27 17:17:00
photos: http://p7wm7amg2.bkt.clouddn.com/websocket.png
---


首先WebSocket是**全双工的**，通信双方建立起一个保持在活动状态的连接通道

WebSocket协议是借用http协议的**101状态码 协议更换**
请求头的**upgrade**实现协议更换

WebSocket协议是基于Frame而非Stream的，也就是说，数据的传输不是像传统数据流读写一样按字节发送，而是采用一帧一帧的Frame，每个Frame都定义了严格的数据结构，因此所有的信息就在这个Frame载体中


#### 特点
* 基于TCP协议
* 具有命名空间
* 可以和HTTP Server共享同一port

## 连接过程
1、首先需要浏览器主动发起一个HTTP请求，请求头中Upgrade内容为WebSocket，
2、一旦服务器收到来自客户端的upgrade请求，便会将请求头中的“Sec-WebSocket-Key”字段提取出来，追加一个固定的串，
3、之后将字段内容（编码的随机字符串）加上那个串，加密并生成一个新的key，作为响应头中的“Sec-WebSocket-Accept”字段的内容返回给浏览器
4、浏览器接收到来自服务器的响应，便会解析响应中的“Sec-WebSocket-Accept”字段，与自己加密后的串比对，匹配成功就建立连接


## 安全性
连接建立时：WebSocket协议规定在连接建立时检查Upgrade请求中的某些字段（如Origin），对于不符合要求的请求立即截断

通信过程中：也对Frame中的控制做了很多限制，以便禁止异常连接

## 和TCP、HTTP协议的关系
WebSocket是基于TCP的独立的协议
和HTTP的唯一关联就是HTTP服务器需要发送一个Upgrade请求，即101 Switching Protocol到HTTP服务器，然后由服务器进行协议转换。

## 关闭连接
主动关闭端会发送Close Frame关闭帧，一端接收到Close Frame，并也发送过Close Frame时，就可以关闭连接了
然后底层TCP连接关闭，连接状态变为closed
关闭成功后状态是clean close



