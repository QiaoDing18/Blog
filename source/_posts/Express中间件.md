title: Express中间件
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-04-27 15:47:00
---

Express是一个基于Node平台的web应用开发框架，在Node基础上扩展了web应用开发所需要的基础功能

Express处理各种请求是通过Express执行函数去调用对应的方法，这样是不是更加方便和快捷了。

## 中间件结构
app.use(path, function)
path：是路由的URL，意义是路由到这个路径时使用这个中间件
function：中间件函数
这个中间件函数可以理解为就是function(request,response,next)

## 中间件分类
1、内置中间件
express.static 是Express目前唯一内置的一个中间件。用来处理静态资源文件。
app.use(express.static(__dirname + '/public'));

2、自定义中间件
在上面中间件结构中，中间件使用时第二个参数是一个function，这个回调有三个参数
(req, res, next)

**每个请求到达服务器时：**
（1）node会为请求创建一个请求对象(request)，该请求对象包含客户端提交上来的数据。
（2）同时也会创建一个响应(response)，响应对象主要负责将服务器的数据响应到客户端。
（3）最后一个参数next是一个方法，在应用中会用到多个中间件，想要运行下一个中间件，那么上一个中间件必须运行next()

3、第三方中间件
*body-parser* ：解析body中的数据，并将其保存为Request对象的body属性
*cookie-parser* ：解析客户端cookie中的数据，并将其保存为Request对象的cookie属性
*express-session* ：解析服务端生成的sessionid对应的session数据，并将其保存为Request对象的session属性
*query*：这个中间件将一个查询字符串从URL转换为JS对象，并将其保存为Request对象的query属性。这个中间件在第四个版本中已经内置了无需安装。

## 到底是什么
浏览器向服务器发送一个请求后，服务器直接通过request定位属性的方式得到通过request携带过去的数据（有用户输入的数据和浏览器本身的数据信息）*这中间就一定有一个函数将这些数据分类做了处理*，处理好后，让request对象调用使用。
这个处理数据的处理函数就是中间件，有以下几个特点
1、封装了一些处理一个完整事件的功能函数
2、非内置的中间件需要安装后，require到文件就可以运行
3、封装了一些或许复杂但肯定是通用的功能

