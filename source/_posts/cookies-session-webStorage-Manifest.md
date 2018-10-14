title: 'cookies,session,webStorage,Manifest'
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-04-27 15:33:01
---

## cookie
**cookie**的内容主要包括：名字、值、过期时间、路径、域。
* 路径和域一起构成cookie的作用范围，如不设置时间，则表示这个cookie的生命周期为浏览器会话期间，*关闭浏览器窗口，cookie就会消失*。这种生命周期为浏览器会话期的cookie被称为会话cookie。
* 会话cookie一般不存储在硬盘而是保存在内存里，*若果设置了过期时间，浏览器会把cookie保存在硬盘上*，关闭后再打开浏览器这些cookie仍然有效直到超过设定的过期设置的过期时间。对于保存在内存里的cookie，不同浏览器有不同的处理方式session机制。
* 当程序需要为某个客户端的请求创建一个session时，服务器首先检查这个客户端的请求里是否包含一个session表示（称为sessionID），如果已包含则说明以前已经为客户端创建过session，服务器就会按照sessionID把这个session检索出来使用（检索不到，会新建一个），如果客户端请求不包含sessionID，则为客户端创建一个session并且生成一个与此session相关联的sessionID，sessionID的值应该是一个既不会重复，又不容易找到规律以仿造的字符串，这个sessionID将被在本次响应中返回给客户端保存。保存这个sessionID的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个表示发给服务器。

#### 关于session机制
由于HTTP协议是无状态的，而出于种种考虑也不希望使之成为有状态的，因此，后面两种方案就成为现实的选择。具体来说cookie机制采用的是在客户端保持状态的方案，而session机制采用的是在服务器端保持状态的方案。同时我们也看到，由于采用服务器端保持状态的方案在客户端也需要保存一个标识，所以session机制可能需要借助于cookie机制来达到保存标识的目的，但实际上它还有其他选择。

## cookie和session的区别
* 1、cookie数据存放在客户端浏览器，session数据放在服务器
* 2、cookie不是很安全，*别人可以分析存放在本地的cookie并进行cookie欺骗*，考虑到安全应当使用session
* 3、*session会在一定时间内保存在服务器上*，当访问量增多，会比较占用你服务器性能，考虑到减轻服务器性能应当使用cookie
* 4、单个cookie保存的数据不能超过**4k**，肯多浏览器都限制一个站点最多保存20个cookie
* 5、建议将登录等重要信息存放在session中，其他信息如果需要保存，可以存放咋cookie中
* 6、session保存在服务器，客户端不知道其中的信息；cookie保存在客户端，服务器能够知道其中的信息
* 7、session中保存的是对象，cookie中保存的是字符串
* 8、session不能区分路径，同一个用户在访问一个网站期间，所有的session在任何一个地方都可以访问到，而cookie中如果设置了路径参数，那么同一个网站中不同路径下的cookie互相访问不到的

## web Storage和cookie
Web Storage的概念和cookie相似，区别是它是为了更大容量存储设计的，cookie的大小是受限的，并且每次请求一个新的页面的时候cookie都会被发送过去，这样无形中浪费了带宽，另外cookie还需要指定作用域，不可跨域调用。
web Storage拥有setItem,getItem,removeItem,clear等方法，不像cookie需要自己封装setCookie，getCookie
但是cookie不可或缺，*cookie的作用是与服务器进行交互*，作为http规范的一部分，而web Storage仅仅是为了在本地“存储”数据而生，sessionStorage、localStorage、cookie的作用都是在浏览器端存储的数据，其中sessionStorage的概念很特别，引入了一个“浏览器窗口”的概念，浏览器窗口没有关闭，即使刷新页面或进入同源另一个页面，数据仍在，*关闭窗口后sessionStorage就会被销毁*，同时“独立”打开的不同窗口，即使是同一页面，sessionStorage对象也是不同的

Web Storage带来的好处
1、减少网络流量：一旦数据保存在本地之后，就可以避免再向服务器请求数据，因此减少不必要的数据请求，减少数据在浏览器和服务器间不必要的来回传递
2、快速显示数据：性能好，从本地读数据比通过网络从服务器上获得数据快得多，本地数据可以即使获得，再加上网页本身也有缓存，因此整个页面和数据都在本地的话，可以立即显示
3、临时存储：很多时候数据只需要在用户浏览一组页面期间使用，关闭窗口后数据就可以丢弃了，这种情况使用sessionStorage非常方便

## 浏览器本地存储与服务端存储的区别
其实数据既可以在浏览器本地存储，也可以在服务器端存储。浏览器可以保存一些数据，需要的时候直接从本地读取。sessionStorage、localStorage和cookie都是由浏览器存储在本地的数据
服务器端也可以保存所有用户的所有数据，但需要的时候浏览器要向服务器请求数据。
1、服务器端可以保存用户的持久数据，如数据库和云存储将用户的大量数据保存在服务器端
2、服务器端也可以保存用户的临时会话数据，服务器端的session机制，如jsp的session对象，数据保存在服务器上
服务器端保存所有的用户的数据，所以服务器端的开销较大，而浏览器端保存则把不同用户需要的数据分别保存在用户各自的浏览器中，浏览器端一般只用来存储小数据，而非服务可以存储大数据或小数据服务器存储数据安全一些。浏览器只适合存储一般数据

## sessionStorage、localStorage和cookie的区别 
共同点：都是保存在浏览器端、且同源
区别：
1、cookie数据始终在同源的http请求中携带（即使不需要），即cookie在浏览器和服务器间来回传递，而sessionStorage和localStorage不会把数据发送给服务器，仅在本地保存。cookie数据还有路径的概念，可以限制cookie只属于某个路径下
2、*存储大小限制也不同*，cookie数据不能超过4k，同时因为每次http请求都会携带cookie、所以cookie只适合保存很小的数据，如会话标识。sessionStorage和localStorage虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大
3、*数据有效期不同*
sessionStorage：仅在当前浏览器窗口关闭之前有效；
localStorage：始终有效，窗口或浏览器关闭也一直保存，因此用作持久数据
cookie：只在设置的cookie过期时间之前有效，即使窗口关闭或浏览器关闭
4、*作用域不同*
**sessionStorage不在不同的浏览器窗口共享，即使是同一个页面**
**localStorage在所有同源窗口都是共享的**
cookie也是所有同源窗口中都是共享的
5、web Storage支持事件通知机制，可以将数据更新的通知发送给监听者 
6、web Storage的api接口使用更方便


## Manifest
Manifest是一个简单的文本文件，它的扩展名是任意的，定义需要缓存的文件、资源，当第一次打开时，浏览器会自动缓存相应的资源

### 特点
离线浏览：当网络断开时，可以继续访问页面
访问速度快：将文件缓存到本地，不需要每次都从网络上请求
稳定性：做了Manifest缓存，遇到突发网络故障或者服务器故障，继续访问本地缓存
![][1]

### 用法
创建一个和html同名的Manifest文件
如：《html lang="en" manifest="index.manifest"》
1、Manifest的引入可以是绝对路径也可以是相对路径，如果使用绝对路径，Manifest文件需要和站点挂在同一域名下
2、Manifest文件可以保存为任意的扩展名，但mine-type必须是text/cache-manifest
3、Manifest标签应该包含到你需要缓存资源的页面，当第一次打开该页面时，浏览器会解析该页面中的Manifest，并缓存里面列举的资源，同时该页面也会自动被浏览器缓存，即使该页面没有在Manifest中列出

### 内容
分为三段：CACHE、NETWORK、FALLBACK，其中NETWORK和FALLBACK为可选项
1、第一行为CACHE MANIFEST为固定格式，必须写在前面
2、CACHE（必须）标识出哪些文件需要缓存，可以是相对路径也可以是绝对路径。这里列举出来的文件，当第一次加载下来时，会被浏览器还存在本地。
3、NETWORK这部分是要绕过缓存直接读取文件，可以使用\*。使用\*时表示除了CACHE指定文件外其他所有页面都需要联网访问
4、FALLBACK（可选）当资源无法访问时，浏览器使用后备资源去替代。第二个表示后备页面。两个URL都必须使用相对路径并且与清单文件同源。可以使用通配符











  [1]: ./images/1523195088840.jpg