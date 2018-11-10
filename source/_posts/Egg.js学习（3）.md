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
首先是基础对象：Application、Context、Request、Response，以及矿建扩展的一些对象