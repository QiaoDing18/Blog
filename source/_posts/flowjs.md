title: flow.js
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-05-22 19:31:00
description: facebook出品
photos: http://p7wm7amg2.bkt.clouddn.com/js.png
---
## balabala
js对于类型检测不是很好，很难保证一些变量和函数和预期的一样
flow.js是facebook出品，它的的类型系统可以保证代码的可靠性，在后期维护的时候能够传达出有效的信息
要在js前的注释里写上    //@flow
[git](https://github.com/facebook/flow)
[文档](https://flow.org/en/docs/types/)

## 简单类型
flow.js中定义了5种最简单的类型，都是小写，其中void对应js的undefined
	* boolean
	* number
	* string
	* null
	* void
just like this
```javascript
var num:number = 1;
var str:string = 'a';
```

## 复杂类型
主要有
	* Object
	* Array
	* 函数
	* 自定义Class
exp：
### 对象
```javascript
//Object大写的O
var o:Object = {
  hello:'h'
};

//声明了Object的key
var o2:{key:string} = {
  key:'z233'
};
```
### 数组
```javascript
//基于基本类似的数组，数组内都是相同类型
var numberArr:number[] = [12,3,4,5,2];
//另一个写法
var numberAr2r:Array<number> = [12,3,2,3];

var stringArr:string[] = ['12','a','cc'];
var booleanArr:boolean[] = [true,true,false];
var nullArr:null[] = [null,null,null];
var voidArr:void[] = [ , , undefined,void(0)];
```
### 函数
```javascript
/**
 * 声明带类型的函数
 * 这里是声明一个函数fn，规定了自己需要的参数类型和返回值类型。
 */
function fn(arg:number,arg2:string):Object{
  return {
    arg,
    arg2
  }
}
//同理，ES2015箭头函数的写法
var fn2 = (arg:number,arg2:string):Object => {
  return {
    arg,
    arg2
  }
}

/**
 * 这里是声明变量fn2，规定了它所需的函数的特征:
 * 参数： (arg:string,arg2:number)
 * 返回值：Object
 */
var fn3:(arg:string,arg2:number)=>Object = function(){
  return {}
}

/**
 * 对比下面这种写法,
 * 两者的声明的地方不一样，造成的意义也不同。
 */
var fn4 = function(arg:string,arg2:Object):number{
  return 1;
}
```

### 自定义class
```javascript
class MyClass{
  name:string;
  constructor(n){
    this.name = n;
  }
}

var myClass : MyClass = new MyClass('abc');
```

## 引入
如果是正经的带webpack + babel 的前端项目,可以无缝集成，加入babel插件即可：babel-plugin-transform-flow-strip-types

个人觉得flow解决了代码在封装时候对于参数未知的情况的处理问题，嗯