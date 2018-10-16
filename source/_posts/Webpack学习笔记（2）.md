title: Webpack学习笔记（2）
tags:
  - 前端
categories:
  - web
author: 乔丁
date: 2018-10-16 01:09:09
description: 入门了解
photos: http://p7wm7amg2.bkt.clouddn.com/webpack.png
---

## 构建项目
webpack在执行构建时会从项目根目录下的webpack.config.js文件读取配置，需要在这里配置文件
```javascript
const path = require('path');

module.exports = {
  // JavaScript 执行入口文件
  entry: './main.js',
  output: {
    // 把所有依赖的模块合并输出到一个 bundle.js 文件
    filename: 'bundle.js',
    // 输出文件都放到 dist 目录下
    path: path.resolve(__dirname, './dist'),
  }
};
```
如一个项目结构这样：
|-- index.html
|-- main.js
|-- show.js
|-- webpack.config.js

具体代码：
页面入口文件 index.html
```html
<html>
  <head>
    <meta charset="UTF-8">
  </head>
  <body>
    <div id="app"></div>
    <!--导入 Webpack 输出的 JavaScript 文件-->
    <script src="./dist/bundle.js"></script>
  </body>
</html>
```

JS 工具函数文件 show.js
```javascript
// 操作 DOM 元素，把 content 显示到网页上
function show(content) {
  window.document.getElementById('app').innerText = 'Hello,' + content;
}
// 通过 CommonJS 规范导出 show 函数
module.exports = show;
```

JS 执行入口文件 main.js
```javascript
// 通过 CommonJS 规范导入 show 函数
const show = require('./show.js');
// 执行 show 函数
show('Webpack');
```

在项目目录下执行webpack命令运行构建，会生成相应构建后文件bundle.js。这是一个可执行文件，它包含所依赖的两个模块main.js和show.js以及内置的webpackbootstrap启动函数。打开index.html会看到hello,webpack

Webpack 是一个打包模块化 JavaScript 的工具，它会从 main.js 出发，识别出源码中的模块化导入语句， 递归的寻找出入口文件的所有依赖，把入口和其所有依赖打包到一个单独的文件中。 从 Webpack2 开始，已经内置了对 ES6、CommonJS、AMD 模块化语句的支持。

## 使用loader
基于之前的代码进行添加样式：
```css
#app{
  text-align: center;
}
```
webpack把一切文件看做模块，css文件也不例外，修改入口文件main.js如下：
```javascript
// 通过 CommonJS 规范导入 CSS 模块
require('./main.css');
// 通过 CommonJS 规范导入 show 函数
const show = require('./show.js');
// 执行 show 函数
show('Webpack');

```
这样会报错，因为webpack不支持原始css解析，要支持非js类型的文件，需要使用webpack的loader机制。修改webpack文件如下
```javascript
const path = require('path');

module.exports = {
  // JavaScript 执行入口文件
  entry: './main.js',
  output: {
    // 把所有依赖的模块合并输出到一个 bundle.js 文件
    filename: 'bundle.js',
    // 输出文件都放到 dist 目录下
    path: path.resolve(__dirname, './dist'),
  },
  module: {
    rules: [
      {
        // 用正则去匹配要用该 loader 转换的 CSS 文件
        test: /\.css$/,
        use: ['style-loader', 'css-loader?minimize'],
      }
    ]
  }
};
```
loader可以看做具有文件转换作用的翻译员，配置里的module.rule数组配置了一组规则，告诉webpack在遇到哪些文件时使用哪些loader去加载和转换。如上配置告诉webpack在遇到以.css结尾的文件时先使用css-loader读取css文件，仔给style-loader把css内容注入到js里。在配置loader时需要注意
- use属性的值需要一个由loader名称组成的数组，loader的执行顺序是由后到前
- 每一个loader都可以通过URL querystring的方式传入参数     