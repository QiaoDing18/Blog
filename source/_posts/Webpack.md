title: Webpack
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-04-27 17:16:00
photos: http://p7wm7amg2.bkt.clouddn.com/webpack.png
---

## Webpack概念
WebPack可以看做是模块打包机：它做的事情是，分析你的项目结构，找到JavaScript模块以及其它的一些浏览器不能直接运行的拓展语言（Scss，TypeScript等），并将其转换和打包为合适的格式供浏览器使用。

webpack可以实现 开发 和 测试 环境

四个核心：入口（entry）、出口（output）、module（模块加载器）、插件（plugins）

## entry入口
入口，指示webpack应该使用哪个模块 来作为构建内部依赖图的开始。进入入口起点后，webpack会找出哪些模块和库是入口起点（直接和间接）依赖的
在entry中可以设置一个或多个入口，多个入口时在entry对象里添加多个入口属性
```javascript
entry: {
  app: './src/app.js',
  vendors: './src/vendors.js'
}
```

## output出口
output属性告诉webpack在哪里输出它所创建的bundles，以及如何命名这些文件。
可以设置path、filename属性
```javascript
output: {
  path: path.resolve(__dirname, 'dist'),
  filename: 'my-first-webpack.bundle.js'
}
```

## loader(module)
loader让webpack能够去处理那些非js文件（webpack本身只理解js）。loader可以将所有类型的文件转换为webpack能够处理的有效模块
本质上，webpack loader将所有类型的文件，转换为应用程序的依赖图可以直接引用的模块
在更高层面，webpack的配置中loader有两个目标
>1、识别出应该被对应的 loader 进行转换的那些文件。(使用 test 属性)
>2、转换这些文件，从而使其能够被添加到依赖图中（并且最终添加到 bundle 中）(use 属性)

```javascript
module: {
  rules: [
    { test: /\.txt$/, use: 'raw-loader' }
  ]
}
```
在rule中必须包括test和use，告诉编译器：“嘿，webpack 编译器，当你碰到「在 require()/import 语句中被解析为 '.txt' 的路径」时，在你对它打包之前，先使用 raw-loader 转换一下。

## 插件 plugins
loader被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。
想要使用一个插件，需要require它，然后把它添加到plugins数组中。多数插件可以通过选项（option）自定义。你也可以在一个配置文件中因为不同的目的而多次使用同一个插件，这时需要使用new操作符来创建它的一个实例
```javascript
plugins: [
  new webpack.optimize.UglifyJsPlugin(),
  new HtmlWebpackPlugin({template: './src/index.html'})
]
```


### 实时预览
webpack的监听模式，当文件变化时重新执行，执行完后通知DevServer。devserver会让webpack在构建出的JS代码里注入一个代理客户端用于控制网页，网页和DevServer之间通过Websocket协议通信，以方便DevServer主动向客户端发送命令。DevServer在收到来自webpack的文件变化通知时通过注入的客户端控制网页刷新

### 模块热替换
模块热替换能做到在不重新加载整个网页的情况下，通过将被更新过的模块替换老的模块，再重新执行一次来实现实时预览。

### 核心概念
* *Entry：入口*，Webpack 执行构建的第一步将从 Entry 开始，可抽象成输入。
* *Module：模块*，在 Webpack 里一切皆模块，一个模块对应着一个文件。Webpack 会从配置的 Entry 开始递归找出所有依赖的模块。
* *Chunk：代码块*，一个 Chunk 由多个模块组合而成，用于代码合并与分割。
* *Loader：模块转换器*，用于把模块原内容按照需求转换成新内容。
* *Plugin：扩展插件*，在 Webpack 构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情。
* *Output：输出结果*，在 Webpack 经过一系列处理并得出最终想要的代码后输出结果。

### Entry
#### context
webpack在寻找相对路径的文件时会以context为根目录，context默认会执行启动webpack时所在的当前工作目录。