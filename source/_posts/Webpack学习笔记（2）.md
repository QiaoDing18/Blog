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
- 每一个loader都可以通过URL querystring的方式传入参数，例如css-loader?minimize中的minimize告诉css-loader要开启css压缩

在安装了style-loader后，bundle.js文件被更新了，里面注入了在main.css中写的css代码，而不是额外生成一个css文件。但刷新index.html会发现样式生效了。这是style-loader的功劳，他的工作原理大概是把css内容用js里的字符串储存起来，在网页执行js时通过DOM操作动态地往html head标签里面插入htnl style标签。其实可以使用webpack的Plugin机制来实现。

给loader传入属性的方式除了querystring外，还可以通过Object传入，以上的Loader配置可以修改如下:
```javascript
use: [
  'style-loader', 
  {
    loader:'css-loader',
    options:{
      minimize:true,
    }
  }
]
```
这样比?minimize可读性高一些

还可以在源码中指定用什么loader去处理文件：
```javascript
require('style-loader!css-loader?minimize!./main.css');
```
这样就能指定对./main.css这个文件先采用css-loader再采用style-loader转换


## 使用Plugin
Plugin是用来扩展webpack功能的，通过在构建流程里注入钩子实现，它给webpack带来了很大的灵活性。

使用loader加载了css文件，本节将通过Plugin把注入到bundle.js文件里的css提取到单独的文件中，配置修改如下：
```javascript
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  // JavaScript 执行入口文件
  entry: './main.js',
  output: {
    // 把所有依赖的模块合并输出到一个 bundle.js 文件
    filename: 'bundle.js',
    // 把输出文件都放到 dist 目录下
    path: path.resolve(__dirname, './dist'),
  },
  module: {
    rules: [
      {
        // 用正则去匹配要用该 loader 转换的 CSS 文件
        test: /\.css$/,
        use: ExtractTextPlugin.extract({
          // 转换 .css 文件需要使用的 Loader
          use: ['css-loader'],
        }),
      }
    ]
  },
  plugins: [
    new ExtractTextPlugin({
      // 从 .js 文件中提取出来的 .css 文件的名称
      filename: `[name]_[contenthash:8].css`,
    }),
  ]
};
```
安装新引入的插件需要安装新的插件： extract-text-webpack-plugin.

重新构建项目，会生成一个main_1a87a56a.css 文件，bundle.js里也没有css代码了，再把该css文件引入到index.html里就完成了。

从以上的配置中可以看出，webpack是通过plugin属性来配置需要使用的插件列表的。plugins属性是一个数组，里面的每一项都是插件的一个实例，在实例化一个组件时可以通过构造函数传入这个组件支持的配置属性。

例如ExtractTextPlugin插件的作用是提取出js代码里的css到一个单独的文件。对此可以通过插件的filename属性，告诉插件输出的css文件名称是通过\[name\]_\[contenthash:8\].css字符串模板生成的，里面的\[name\]代表文件名称，\[contenthash:8\]代表根据文件内容算出的8位hash值


## 使用DevServer
现在可以让webpack运行起来了，但在实际开发中还可以：
> 1、提供http服务而不是使用本地文件预览
> 2、监听文件的变化并自动刷新网页，实时预览
> 3、支持Source Map，方便调试

Webpack原生提供2、3原生支持，跟官方提供的开发工具DevServer也可以很方便的做到1。DevServer会启动一个http服务器用于服务网页请求，同时会帮助启动Webpack，并接收Webpack发出的文件变更信号，通过WebSocket协议自动刷新网页做到实时预览。

安装后执行webpack-dev-server，Devserver就启动了，会看到控制台输出：
```
Project is running at http://localhost:8080/
webpack output is served from /
```
这意味着 DevServer 启动的 HTTP 服务器监听在 http://localhost:8080/ ，DevServer 启动后会一直驻留在后台保持运行，访问这个网址就能获取项目根目录下的 index.html。 

会报错./dist/bundle.js 加载404了。 同时会发现并没有文件输出到 dist 目录，原因是 DevServer 会把 Webpack 构建出的文件保存在内存中，在要访问输出的文件时，必须通过 HTTP 服务访问。 由于 DevServer 不会理会 webpack.config.js 里配置的 output.path 属性，所以要获取 bundle.js 的正确 URL 是 http://localhost:8080/bundle.js，对应的 index.html 应该修改为：
```html
<html>
  <head>
    <meta charset="UTF-8">
  </head>
  <body>
    <div id="app"></div>
    <!--导入 DevServer 输出的 JavaScript 文件-->
    <script src="bundle.js"></script>
  </body>
</html>
```

