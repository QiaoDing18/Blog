title: Egg.js学习（3）
tags:
  - 前端
categories:
  - web
author: 乔丁
date: 2018-11-10 22:11:09
description: 毕设打算那这玩意弄后台
photos: http://img.bimg.126.net/photo/TNw-QmJvMUKKh8ozLku4xQ==/332703422488837224.jpg
---

## 从目录结构开始入门
先了解下目录：
```
egg-project
├── package.json
├── app.js (可选) -- 用于自定义启动时的初始化工作
├── agent.js (可选) -- 同app.js
├── app
|   ├── router.js -- 用于配置URL路由规则
│   ├── controller -- 用于解析用户的输入，处理后返回相应的结果
│   |   └── home.js
│   ├── service (可选) -- 用于编写业务逻辑层
│   |   └── user.js
│   ├── middleware (可选) -- 用于编写中间件
│   |   └── response_time.js
│   ├── schedule (可选) -- 用于定时任务
│   |   └── my_task.js
│   ├── public (可选)  -- 用于放置静态资源
│   |   └── reset.css
│   ├── view (可选) -- 用于放置模板文件
│   |   └── home.tpl
│   ├── model (可选) -- 用于放置领域模型
│   |   └── home.tpl
│   └── extend (可选) -- 用于框架的扩展
│       ├── helper.js (可选)
│       ├── request.js (可选)
│       ├── response.js (可选)
│       ├── context.js (可选)
│       ├── application.js (可选)
│       └── agent.js (可选)
├── config -- 用于编写配置文件
|   ├── plugin.js
|   ├── config.default.js
│   ├── config.prod.js
|   ├── config.test.js (可选)
|   ├── config.local.js (可选)
|   └── config.unittest.js (可选)
└── test -- 用于单元测试
    ├── middleware
    |   └── response_time.test.js
    └── controller
        └── home.test.js
```

## 内置对象
首先是基础对象：Application、Context、Request、Response，以及矿建扩展的一些对象：Controller，Service，Helper，Config，Logger

### Application
Application是一个全局对象，在一个应用中，只会实例化一个，继承自Koa.Application，在它上面可以挂载一些全局的方法和对象。可以在插件或应用中扩展Application对象。

#### 事件
在框架运行时，会在Application实例上出发一些事件，应用开发者或插件开发者可以监听这些事件做一些操作，一般会在启动自定义脚本中进行监听：
- server：该事件一个worker进程只会触发一次，在HTTP服务完成启动后，会将HTTP server通过这个事件暴露出来给开发者
- error：运行时有任何的异常被onerror插件捕获后，都会触发error事件，将错误对象和关联的上下文暴露给开发者，可以自定义的日志记录上报等处理
- request和response：应用受到请求和响应请求时，分别会触发request和Response事件，并将当前请求上下文暴露出老，开发者可以监听这两个事件来进行日志记录。

```javascript
// app.js

module.exports = app => {
  app.once('server', server => {
    // websocket
  });
  app.on('error', (err, ctx) => {
    // report error
  });
  app.on('request', ctx => {
    // log receive request
  });
  app.on('response', ctx => {
    // ctx.starttime is set by framework
    const used = Date.now() - ctx.starttime;
    // log total cost
  });
};
```

#### 获取方式
Application对象可以在编写应用时任何一个地方获取到，几个经常用到的方式：
几乎所有被框架Loader加载的文件，都可以export一个函数，这个函数会被Loader调用，并使用app作为参数：
- 启动自定义脚本
```javascript
// app.js
module.exports = app => {
  app.cache = new Cache();
};
```
- Controller文件
```javascript
// app/contraller/user.js
class UserController extends Controller {
  async fetch() {
    this.ctx.body = this.ap.cache.get(this.ctx.query.id);
  }
}
```


和Koa一样，在Content对象上，可以通过ctx.app访问到Application对象。以上的Controller为例：
