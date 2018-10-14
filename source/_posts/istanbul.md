title: istanbul是个啥
author: 乔丁
tags:
  - 闲聊
categories:
  - chat
date: 2018-05-23 10:32:00
description: vue源码中使用的一个测试工具，比较有意思
photos: http://www.ruanyifeng.com/blogimg/asset/2015/bg2015060802.jpg
---

## balabala
要学的真的是多 组里氛围很棒 压力甚大

## 正题
对于是否所有代码都测试到了有个指标，代码覆盖率。有四个测量维度
> 行覆盖率（line coverage）：是否每一行都执行了
> 函数覆盖率（function coverage）：是否每个函数都调用了
> 分支覆盖率（branch coverage）：是否每个if代码块都执行了
> 语句覆盖率（statement coverage）：是否每个语句都执行了

Istanbul是js代码覆盖率的计算工具
这个软件以土耳其最大城市伊斯坦布尔命名，因为土耳其地毯世界闻名，而地毯是用来覆盖的。
![](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015060802.jpg)

## exp
```javascript
let a = 6;
let b = 6;
if(a > b){
  console.log('a > b')
}else if(a < b){
  console.log('a < b')
}else{
  console.log('a = b')
}
```

```
============= Coverage summary =================
Statements   : 71.43% ( 5/7 )
Branches     : 50% ( 2/4 )
Functions    : 100% ( 0/0 )
Lines        : 71.43% ( 5/7 )
================================================
```
返回显示的：
7个语句执行5个
4个分支执行了2个
0个函数执行了0个
7行代码执行了5行

还剩成了coverage这个子目录，可以可视化的看情况

这个工具还可以设置覆盖率门槛
还可以与其他测试框架结合

## 忽略代码
在vue源码中用到的是istanbul的注释语法，允许某些代码不计入覆盖率
```javascript
var object = parameter || /* istanbul ignore next * /  {};
```
上面代码是为 object 指定默认值（一个空对象）。如果由于种种原因，没有为 object 为空对象的情况写测试，可以用注释，不将这种情况计入覆盖率。注意，注释要写在"或"运算符的后面。

```javascript
/* istanbul ignore if  */
if (a)) {
    return callback(b);
}
```
上面代码的if语句块，在计算覆盖率的时候会被忽略