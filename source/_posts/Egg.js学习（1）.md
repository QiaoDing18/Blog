title: Egg.js学习（1）
tags:
  - 前端
categories:
  - web
author: 乔丁
date: 2018-10-25 10:11:09
description: 毕设打算那这玩意弄后台
photos: http://img.bimg.126.net/photo/TNw-QmJvMUKKh8ozLku4xQ==/332703422488837224.jpg
---
## Egg.js是什么
**Egg.js为企业级框架和应用而生**

### 设计原则
Egg 的插件机制有很高的可扩展性，一个插件只做一件事（比如 Nunjucks 模板封装成了 egg-view-nunjucks、MySQL 数据库封装成了 egg-mysql）。Egg 通过框架聚合这些插件，并根据自己的业务场景定制配置，这样应用的开发成本就变得很低。

Egg 奉行『约定优于配置』，按照一套统一的约定进行应用开发，团队内部采用这种方式可以减少开发人员的学习成本，开发人员不再是『钉子』，可以流动起来。没有约定的团队，沟通成本是非常高的，比如有人会按目录分栈而其他人按目录分功能，开发者认知不一致很容易犯错。但约定不等于扩展性差，相反 Egg 有很高的扩展性，可以按照团队的约定定制框架。使用 Loader 可以让框架根据不同环境定义默认配置，还可以覆盖 Egg 的默认约定。

### 和其他框架对比
Express 是 Node.js 社区广泛使用的框架，简单且扩展性强，非常适合做个人项目。但框架本身缺少约定，标准的 MVC 模型会有各种千奇百怪的写法。Egg 按照约定进行开发，奉行『约定优于配置』，团队协作成本低。

Sails 是和 Egg 一样奉行『约定优于配置』的框架，扩展性也非常好。但是相比 Egg，Sails 支持 Blueprint REST API、WaterLine 这样可扩展的 ORM、前端集成、WebSocket 等，但这些功能都是由 Sails 提供的。而 Egg 不直接提供功能，只是集成各种功能插件，比如实现 egg-blueprint，egg-waterline 等这样的插件，再使用 sails-egg 框架整合这些插件就可以替代 Sails 了。

### 特性
- 提供基于 Egg 定制上层框架的能力
- 高度可扩展的插件机制
- 内置多进程管理
- 基于 Koa 开发，性能优异
- 框架稳定，测试覆盖率高
- 渐进式开发

## Egg.js和Koa
### 异步！
异步发展：callback -> Promise -> Generator + co -> async

### Koa和Express
Koa 和 Express 的设计风格非常类似，底层也都是共用的同一套 HTTP 基础库，但是有几个显著的区别，除了上面提到的默认异步解决方案之外，主要的特点还有下面几个。

#### Middleware
Koa是经典的洋葱圈模型
<img src="https://camo.githubusercontent.com/d80cf3b511ef4898bcde9a464de491fa15a50d06/68747470733a2f2f7261772e6769746875622e636f6d2f66656e676d6b322f6b6f612d67756964652f6d61737465722f6f6e696f6e2e706e67">

一个收到的请求经过一个中间件的时候都会执行两次，对比Express形式的中间件，Koa的模型可以非常方便的实现后置处理逻辑。

提一下Express的中间件：Express内部维护一个函数数组，这个函数数组表示在发出响应之前要执行的所有函数，也就是中间件数组。在使用app.use(fn)时候，传进来的fn就在这个数组里，执行完后调用next()方法执行函数数组里的下一个函数，如果没有调用next的话，调用就会终止。<a href="https://www.jianshu.com/p/797a4e38fe77">这里简单实现了一个中间件</a>

我对鱼这两个中间件的理解是这样的。Express是类似回调的控制，Koa1是Generator函数yield控制，Koa2是async函数await的控制。

#### Content
Express只有Request和Response两个对象，Koa增加了一个content对象，作为这次请求的上下文对象（Koa1中为中间件的this，在Koa2中作为中间件的第一个参数传入）。可以将一次请求相关的上下文都挂载到这个对象上。类似traceld这种需要贯穿整个请求的属性就可以挂载上去。相较于request、response而言更加符合语义。

Content上挂载了Request和Response两个对象并提供了方法：
- get request.query
- get request.hostname
- set response.body
- set response.status

#### 异常处理
使用try...catch
```javascript
async function onerror(ctx, next) {
  try {
    await next();
  } catch (err) {
    ctx.app.emit('error', err);
    ctx.body = 'server error';
    ctx.status = err.status || 500;
  }
}
```
只要将这个中间件放在其他中间件前，就可以捕获所有同步或异步代码中抛出的异常

### Egg继承于Koa
Egg选择了Koa作为基础，进一步增强
#### 扩展
可以通过定义 app/extend/{application,context,request,response}.js 来扩展 Koa 中对应的四个对象的原型，通过这个功能，我们可以快速的增加更多的辅助方法，例如我们在 app/extend/context.js 中写入下列代码：
```javascript
// app/extend/context.js
module.exports = {
  get isIOS() {
    const iosReg = /iphone|ipad|ipod/i;
    return iosReg.test(this.get('user-agent'));
  },
};
```
在Controller中，可以用到刚才定义的属性：
```javascript
// app/controller/home.js
exports.handler = ctx => {
  ctx.body = ctx.isIOS
    ? 'Your operating system is iOS.'
    : 'Your operating system is not iOS.';
};
```
### 插件
在Express和Koa中有大量插件支持如koa-session提供Session的支持，引入koa-bodyparser来解析请求body。Egg提供了更加强大的插件机制，让这些独立领域功能模块可以更加容易编写。一个插件可以包含：
- extend：扩展基础对象的上下文，提供各种工具类、属性
- middleware：增加一个或多个中间件，提供请求的前置、后置处理逻辑
- config：配置各个环境下插件自身的默认配置项

一个独立领域下的插件实现，可以在代码维护性非常高的情况下实现非常完善的功能，而插件也支持配置各个环境下的默认配置，这样在使用的时候几乎可以不需要修改配置项。

### 版本问题
Egg1.x基于Koa1.x，Egg2.x基于Koa2.x，所以就嗯