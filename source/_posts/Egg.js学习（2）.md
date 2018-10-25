title: Egg.js学习（2）
tags:
  - 前端
categories:
  - web
author: 乔丁
date: 2018-10-26 16:11:09
description: 毕设打算那这玩意弄后台
photos: http://img.bimg.126.net/photo/TNw-QmJvMUKKh8ozLku4xQ==/332703422488837224.jpg
---
## 渐进式开发
在Egg里，插件包括path和package两种加载模式，要如何选择

### 初始状态
假设有一段分析客户端的代码，实现功能：
- ctx.isAndroid
- ctx.isIOS
再假设一个项目结构
```
example-app
├── app
│   ├── extend
│   │   └── context.js
│   └── router.js
├── test
│   └── index.test.js
└── package.json
```
核心代码：
```javascript
// app/extend/context.js
module.exports = {
    get isIOS() {
        const iosReg = /iphone|ipad|ipod/i;
        return iosReg.test(this.get('user-agent'));
    }
}
```

### 插件雏形
上面这个逻辑是通用的，可以写成插件。但是功能还没完善，直接独立成插件维护起来比较麻烦。可以把代码写成插件的形式，但并不独立出去。

新的目录结构：
```
example-app
├── app
│   └── router.js
├── config
│   └── plugin.js
├── lib
│   └── plugin
│       └── egg-ua
│           ├── app
│           │   └── extend
│           │       └── context.js
│           └── package.json
├── test
│   └── index.test.js
└── package.json
```

核心代码：
- app/extend/context.js 移动到lib/plugin/egg-ua/app/extend/context.js。
- lib/plugin/egg-ua/package.json 声明插件。

```
{
    "eggPlgin": {
        "name": "ua"
    }
}
```
- config/plugin.js中通过path来挂载插件
```javascript
const path = require('path');
exports.ua = {
    enable: true,
    path: path.join(__dirname, '../lib/plugin/egg-ua')
}
```

### 抽成独立组件
模块功能成熟后，可以抽出来称为独立的插件。首先改造了原有应用
- 移除lib/plugin/egg-ua目录
- package.json中声明对egg-ua的依赖
- config/plugin.js中修改依赖声明为package方式
```javascript
exports.ua = {
    enable: true,
    package: 'egg-ua'
}
```
插件可以在本地使用npm link方式进行测试，测试号就上线呗。

## 从插件到框架
插件会大量用到，如果自己搞一个框架呢
抽出来一个example-framework框架，首先假设一个项目结构：
```
example-framework
├── config
│   ├── config.default.js
│   └── plugin.js
├── lib
│   ├── agent.js
│   └── application.js
├── test
│   ├── fixtures
│   │   └── test-app
│   └── framework.test.js
├── README.md
├── index.js
└── package.json
```

<img src="http://p7wm7amg2.bkt.clouddn.com/between.png">

把原来的egg-ua的依赖删掉，配置到框架的package.json和config/plugin.js里

然后改造原有应用：
- 移除