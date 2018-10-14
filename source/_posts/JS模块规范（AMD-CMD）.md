title: JS模块规范（AMD CMD）
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-02-27 16:33:00
photos: http://p7wm7amg2.bkt.clouddn.com/js.png
---


## CommonJS
Node采用的是CommonJS，**同步**加载模块
```javascript
var clock = require('clock.js');
clock.start();

// 导入
const moduleA = require('./moduleA');
// 导出
module.exports = moduleA.someFunc;
```
这种写法适合服务端，因为在服务器读取模块都是在本地磁盘，加载速度很快。**但是如果在客户端**，加载模块的时候有可能出现“假死”状况。比如上面的例子中clock的调用必须等待clock.js请求成功，加载完毕。那么，能不能异步加载模块呢？

## AMD
AMD，即 (Asynchronous Module Definition)，这种规范是**异步**的加载模块，requireJs应用了这一规范。**先定义所有依赖**，然后在加载完成后的回调函数中执行
```javascript
require([module], callback);
require(['clock.js'],function(clock){
  clock.start();
});

// 定义一个模块
define('module', ['dep'], function(dep) {
  return exports;
});
// 导入和使用
require(['module'], function(module) {
});
```
AMD虽然实现了异步加载，但是**开始就把所有依赖写出来**是不符合书写的逻辑顺序的，能不能像commonJS那样用的时候再require，而且还支持异步加载后再执行呢？
AMD 的缺点在于JavaScript 运行环境没有原生支持 AMD，需要先导入实现了 AMD 的库后才能正常使用。

## CMD
CMD (Common Module Definition), 是seajs推崇的规范，CMD则是**依赖就近**，**用的时候再require**。它写起来是这样的：
```javascript
define(function(require, exports, module) {
   var clock = require('clock.js');
   clock.start();
});
```

AMD和CMD最大的区别是对依赖模块的执行时机处理不同，而不是加载的时机或者方式不同，二者皆为异步加载模块。
AMD依赖前置，js可以方便知道依赖模块是谁，立即加载；而CMD就近依赖，需要使用把模块变为字符串解析一遍才知道依赖了那些模块，这也是很多人诟病CMD的一点，牺牲性能来带来开发的便利性，实际上解析模块用的时间短到可以忽略。
