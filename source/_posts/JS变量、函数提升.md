title: JS变量、函数提升
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-04-27 16:32:00
photos: http://p7wm7amg2.bkt.clouddn.com/js.png
---


## 变量提升
在ES6前，JS没有块级作用域，只有全局作用域和函数作用域。变量提升即将变量声明提升到它所在作用域的最开始的部分。

```javascript
console.log(global); // undefined
var global = 'global';
console.log(global); // global
```

**会先把声明提前，在对应的位置再赋值**


## 函数提升
函数创建的方式有两种：函数声明和函数字面量。只有函数声明才存在函数提升！

```javascript
console.log(f1);	// function f1() {}
console.log(f2);	// undefined
function f1() {}
var f2 = function() {}
```
执行结果是因为函数声明会被直接提到作用域开始地方。
