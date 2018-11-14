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
```javascript
// app/controller/user.js
class UserController entends Controller {
  async fetch() {
    this.ctx.body = this.ctx.ap.cache.get(this.ctx.query.id);
  }
}
```
在继承于Controller，Service基类的实例中，可以通过this.app访问到Application对象
```javascript
// app/controller/user.js
class UserController extends Controller{
  async fetch() {
    this.ctx.body = this.app.cache.get(this.ctx.query.id);
  }
}
```

### Context
Context是一个请求基本的对象，继承自Koa.Context。在每一次收到用户请求时，框架会实例化一个Context对象，这个对象封装了这次用户请求的信息，并提供了许多便捷的方法来获取请求参数或者设置响应的信息。框架会将所有的Service挂载到Context实例上，一些插件也会将一些其他的方法和对象挂载到它上面（egg-sequelize会将所有的model挂载在Context上）

#### 获取方式
最常见的Context实例获取方式是在Middleware，Controller以及Service中。Controller中的获取方式在上面有，在Service中获取和Controller中获取的方式一样，在Middleware中获取Context实例和Koa的一样，也是根据异步的支持情况来，就如Koa1和Koa2中的一样：
```javascript
// Koa v1
function* middleware(next) {
  // this is instance of Context
  console.log(this.query);
  yield next;
}

// Koa v2
async function middleware(ctx, next) {
  // ctx is instance of Context
  console.log(ctx.query);
}
```
除了在请求时可以获取Context实例之外，在有些非用户请求的场景下我们需要访问service/model等Context实例上的对象，我们可以通过Application.createAnonymouseContext()方法创建匿名Context实例：
```javascript
// app.js
module.exports = app => {
  app.beforeStart(async () => {
    const ctx = app.createAnonymousContext();
    // preload before app start
    await ctx.service.posts.load();
  });
}
```
在定时任务中的每一个task都接受一个Context实例作为参数，以便我们更方便的执行一些定时的业务逻辑：
```javascript
// app/schedule/refresh.js
exports.task = async ctx => {
  await ctx.service.posts.refresh();
};
```

### Request Response
Request是一个请求级别的对象，继承自Koa.Request。封装了Node.js原生的HTTP Request对象，提供了一系列辅助方法获取HTTP请求常用参数


Response是一个请求级别的对象，继承自Koa.Reponse。封装了Node.js原生的HTTP Response对象，提供了一系列辅助方法设置HTTP响应。

#### 获取方式
可以在Context的实例上获取到当前请求的Request(ctx.request)和Response(ctx.response)实例
```javascript
class UserController entends Controller {
  async fetch() {
    const { app, ctx } = this;
    const id = ctx.request.query.id;
    ctx.response.body = app.cache.get(id);
  }
}
```

- Koa会在Context上代理一部分Request和Response上的方法和属性
- 上面的ctx.request.query.id和ctx.query.id是等价的，ctx.response.body=和ctx.body=是等价的
- 获取POST的body应该使用ctx.request.body，而不是ctx.body

### Controller
框架提供了一个Controller基类，并推荐所有的Controller都继承与该基类实现。这个Controller基类有下列属性：
- ctx 当前请求的Context实例
- app 应用的Application实例
- config 应用的配置
- service 应用所有的Service
- logger 为当前Controller封装的logger对象

在Controller文件中，可以通过两种方式来引用Controller基类
```javascript
// app/controller/user.js
// 从egg上获取
const Contraller = require('egg').Controller;
class UserController extends Controller {
  // implement
}
module.exports = UserController;

// 从app实例上获取
module.exports = app => {
  return class UserController extends app.Controller {
    // implement
  }
}
```

### Service
框架提供了一个Service基类，并推荐所有Service都继承与该基类实现
Service基类的属性和Controller基类属性一致，访问方式也类似：
```javascript
// app/service/user.js
// 从egg上获取
const Service = reuqire('egg').Service;
class UserService extends Service {
  // implement
}
module.exports = UserService;

// 从app实例上获取
module.exports = app => {
  return class UserService extends app.Service{
    // implement
  }
}
```

### Helper
Helper用来提供一些实用的utility函数。它的作用在于我们可以将一些常用的动作抽离在helper.js里面成为一个独立的函数，这样可以用JavaScript来写复杂的逻辑，避免逻辑分散各处，同时可以更好的编写测试用例。

Helper自身是一个类，有和Controller基类一样的属性，会在每次请求时进行实例化，因此Helper上的所有函数也能获取到当前请求相关的上下文信息

#### 获取方式
可以在Context的实例上获取到当前请求的Helper(ctx.helper)实例
```javascript
class UserController extends COntroller {
  async fetch() {
    const { app, ctx } = this;
    const id = ctx.query.id;
    const user = app.cache.get(id);
    ctx.body = ctx.helper.formatUser(user);
  }
}

```
除此之外，Helper的实力I还可以在模板中获取到，例如可以在模板中获取到security插件提供的shtml方法

#### 自定义helper方法
应用开发中，我们可能经常要自定义一些helper方法，例如上面例子中的formatUser，我们可以通过框架扩展的形式来自定义Helper方法
```javascript
// ap/extend/helper.js
module.exports = {
  formatUser(user) {
    return only(user, ['name', 'phone'])
  }
}
```

### Config
我们推荐应用开发遵循配置和代码分离的原则，将一些需要硬编码的业务配置都放到配置文件中，同时配置文件支持各个不同的运行环境使用不同的配置，使用起来也非常方便，所有框架、插件和应用级别的配置都可以通过Config对象获取到。

#### 获取方式
可以通过app.config从Application实例上获取得到config对象，可以在Controller，Service，Helper的实例上通过this.config获取到config对象。

### Logger
框架内置了功能强大的日志功能，可以非常方便的打印各种级别的日志到对应的日志文件中，每一个Logger对象都提供了4个级别的方法：
- logger.debug()
- logger.info()
- logger.warn()
- logger.error()

在框架中提供了多个Logger对象：

#### App Logger
可以通过app.logger来获取到它，如果做一些应用级别的日志记录，如记录启动阶段的一些数据信息，记录一些业务上与请求无关的信息，都可以通过App Logger来完成。

#### App CoreLogger
可以通过app.coreLogger来获取到它，开发时不应该通过CoreLogger打印日志，而框架和插件则需要通过它来打印应用级别的日志，这样可以更清晰的区分应用和框架打印的日志，通过CoreLogger打印日志会放到和Logger不同的文件中。

#### Context Logger
可以通过ctx.logger从Context实例上获取到，从访问方式上可以看出，Context Logger一定是与请求相关的，打印的日志都会在前面上一些当前请求相关的信息，如（$userId/$ip/$traceId/${cost}ms $method $url），通过这些信息，可以从日志快速定位请求，并串联一次请求中的所有的日志。

#### Context CoreLogger
可以通过ctx.coreLogger获取到，和Context Logger区别是一般只有插件和框架会通过它来记录日志。

#### Controller Logger & Service Logger
可以在Controller和Service实例上通过this.logger获取到他们，本质上就是一个Context Logger。

### Subscription
订阅模型是一种比较常见的开发模式，譬如消息中间件的消费者或调度任务。因此我们提供了 Subscription 基类来规范化这个模式。

可以通过以下方式来引用 Subscription 基类：
```javascript
const Subscription = require('egg').Subscription;
class Schedule extends Subscription {
  // 需要实现此方法
  // subscribe 可以为 async function 或 generator function
  async subscribe() {}
}
```
插件开发者可以根据自己的需求基于它定制订阅规范，如定时任务就是使用这种规范实现的。