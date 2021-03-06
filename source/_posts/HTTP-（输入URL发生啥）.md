title: HTTP （输入URL发生啥）
author: 乔丁
tags:
  - 计算机网络
categories:
  - net
date: 2018-04-27 16:30:00
---


# *http请求报文由3部分组成：请求行、请求头、请求体*

## 请求行
请求方法 请求URL HTTP协议及版本

## 请求头
1、Accept：请求报文可通过一个“Accept”报文头属性告诉服务端 客户端接受什么类型的响应。 

2、Cookie：客户端的cookie通过这个报文头属性传给服务端。通过cookie中的sessionid与session服务关联起来。

3、Referer：表示这个请求是从哪个URL过来的。

4、Cache-Control：对缓存进行控制，如一个请求希望响应返回的内容在客户端要被缓存一年，或不希望被缓存就可以通过这个报文头达到目的。 

## 请求体
请求的参数啥了

----

# *http响应报文由3部分组成：响应行、响应头、响应体*

## 请求行
报文协议  状态码及描述

状态码：
1xx 消息，一般是告诉客户端，请求已经收到了，正在处理，别急...
2xx 处理成功，一般表示：请求收悉、我明白你要的、请求已受理、已经处理完成等信息.
3xx 重定向到其它地方。它让客户端再发起一个请求以完成整个处理。
4xx 处理发生错误，责任在客户端，如客户端的请求一个不存在的资源，客户端未被授权，禁止访问等。
5xx 处理发生错误，责任在服务端，如服务端抛出异常，路由出错，HTTP版本不支持等。

200 OK 成功    
304 Not Modified 请求的这个资源至你上次取得后，并没有更改，可以直接用本地的缓存
404 Not Found 页面不存在
500 Internal Server Error 服务器问题

## 响应头
1、Cache-Control：响应输出到客户端后，服务端通过该报文头属告诉客户端如何控制响应内容的缓存。 

2、ETag：一个代表响应*服务端资源（如页面）版本*的报文头属性，如果某个服务端资源发生变化了，这个ETag就会相应发生变化。它是Cache-Control的有益补充，可以让客户端“更智能”地处理什么时候要从服务端取资源，什么时候可以直接从缓存中返回响应。 

3、Set-Cookie：服务端可以设置客户端的Cookie，其原理就是通过这个响应报文头属性实现的



# 输入URL到页面加载完发生了什么
URL统一资源定位符，俗称网页地址
URL中包括：
1、传送协议protocol：最常用的是HTTP协议
2、主机host：通常为域名或者IP地址
3、端口号port：以数字形式表示
4、路径path：以“/”字元区别路径中的每一个目录名称
5、查询query：以“?”字元为起点，每个参数以“&”隔开，再以“=”分开参数名称与其对应的值
6、片段fragment：也就是在浏览器环境下location的hash值，用于指定网络资源中的片断，一般用于定位到某个位置

浏览器发生了什么：
1、浏览器查询缓存
2、浏览器询问操作系统服务器的IP地址
3、操作系统做DNS查询，返回IP地址给浏览器
4、浏览器与服务器建立TCP连接
5、浏览器通过TCP连接发送HTTP请求
6、浏览器接收HTTP响应
7、浏览器检查响应，并解码，并根据类型处理

如果您在cookie中设置了HttpOnly属性，那么通过js脚本将无法读取到cookie信息

页面加载完成有两种事件
一是ready，表示文档结构已经加载完成（不包含图片等非文字媒体文件）
二是onload，指示页面包含图片等文件在内的所有元素都加载完成

HTTP数据 + TCP首部 + IP首部 + 以太网首部
 应用层     传输层   网络层     链路层

## IP协议
IP协议的作用就是把各种数据包传送给对方。传输时需要满足条件是：IP地址和MAC地址
IP地址指明了结点被分配到的地址，MAC地址是指网卡所属的固定地址。IP地址可以和MAC地址进行配对。IP可变，MAC不可变

IP间通信依赖MAC地址。在网络上，通信的双方在同一局域网内的情况很少，通常需要中转设备，中转时，会利用下一站中转设备的MAC地址来搜索下一中转目标。
这时，会采用ARP协议来解析地址，根据通信方的IP地址就可以反查出对应的MAC地址。
（ARP协议：IP地址解析成对应的MAC地址）

## TCP协议
TCP提供可靠字节流服务
字节流服务：为了方便传输，将大块数据分割成报文段为单位的数据包进行管理
可靠服务：能够把数据准确可靠地传给对方，能够确认数据最终是否送达到对方

为了准确送达，TCP采用了三次握手策略。用TCP协议把数据包送出去，TCP不会置之不理，会向对方确认是否成功送达。握手时采用了TCP的标志————SYN和ACK

发送端首先发送一个带SYN标志的数据包给对方
接收端收到后，回传一个带有SYN/ACK标志的数据包以示传达确认信息
发送端再回传一个带ACK标志的数据包，代表握手结束

如果握手过程中某个阶段莫名中断，TCP协议会再次以相同的顺序发送相同的数据包

## DNS服务
DNS服务和HTTP协议都在应用层。它提供域名到IP地址之间的解析服务
DNS作用：提供通过域名查找IP地址，或者逆向从IP地址反向查域名的服务

## HTTP和各种协议的关系
1、客户端向DNS发个域名，DNS服务返回对应的IP
2、HTTP向目标服务器发送请求
3、TCP的协议将HTTP请求报文分割成报文段，按序号分为多个报文段，把每个报文段可靠地传给对方
4、IP协议搜索对方的地址，一边中转一边传送
5、TCP协议收到了报文段，再按照序号以原来的顺序重组请求报文
6、HTTP协议拿到内容

## 持久连接
1、多个HTTP请求，需要建立多次TCP连接，开销太大
2、出现了HTTP keep-alive的方法
3、持久连接的特点：只要任意一端没有明确提出断开连接，则保持TCP连接状态
4、在HTTP1.1中，所有连接默认都是持久连接
5、持久连接使多数请求可以管线化。不用等服务器响应就可以继续发请求


## 内容协商机制
1、服务器驱动协商
	由服务器端进行内容协商。以请求的首部字段为参考，在服务端自动处理。但对用户来说，以浏览器发送的信息作为判定的依据。并不一定能筛选出最优内容。

2、客户端驱动协商
	由客户端进行内容协商的方式。用户从浏览器显示的可选项列表中手动选择。还可以利用JS脚本在Web页面上自动进行上述选择。比如按OS的类型或浏览器类型，自行切换成PC版页面或手机版页面。
	
3、透明协商
	是服务器驱动和客户端驱动的结合体，是由服务器端和客户端各自进行内容协商的一种办法
	
	
http1.0中的缓存控制：

Pragma：严格来说，它不属于专门的缓存控制头部，但是它设置no-cache时可以让本地强缓存失效（属于编译控制，来实现特定的指令，主要是因为兼容http1.0，所以以前又被大量应用）
Expires：服务端配置的，属于强缓存，用来控制在规定的时间之前，浏览器不会发出请求，而是直接使用本地缓存，注意，Expires一般对应服务器端时间，如Expires：Fri, 
If-Modified-Since/Last-Modified：这两个是成对出现的，属于协商缓存的内容，其中浏览器的头部是If-Modified-Since，而服务端的是Last-Modified，它的作用是，在发起请求时，如果If-Modified-Since和Last-Modified匹配，那么代表服务器资源并未改变，因此服务端不会返回资源实体，而是只返回头部，通知浏览器可以使用本地缓存。Last-Modified，顾名思义，指的是文件最后的修改时间，而且只能精确到1s以内
http1.1中的缓存控制：

Cache-Control：缓存控制头部，有no-cache、max-age等多种取值
Max-Age：服务端配置的，用来控制强缓存，在规定的时间之内，浏览器无需发出请求，直接使用本地缓存，注意，Max-Age是Cache-Control头部的值，不是独立的头部，譬如Cache-Control: max-age=3600，而且它值得是绝对时间，由浏览器自己计算
If-None-Match/E-tag：这两个是成对出现的，属于协商缓存的内容，其中浏览器的头部是If-None-Match，而服务端的是E-tag，同样，发出请求后，如果If-None-Match和E-tag匹配，则代表内容未变，通知浏览器使用本地缓存，和Last-Modified不同，E-tag更精确，它是类似于指纹一样的东西，基于FileEtag INode Mtime Size生成，也就是说，只要文件变，指纹就会变，而且没有1s精确度的限制。