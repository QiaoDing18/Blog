title: new时候发生了什么
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-04-27 16:34:00
---


```javascript
function Animate(name){
	this.name = name;
}
Animate.prototype.dance = function(){
	console.log(this.name + "在跳舞");
}

var dog = new Animate('旺财');
dog.dance();
```

new Animate('旺财')时候：
1、创建了一个新的对象 var obj = {}
2、将Animate中的this关键字指向obj
3、将Animate的prototype原型指向obj原型，这样obj就拥有了Animate中的方法
4、执行Animate构造函数的代码（this.name=name）

```javascript
var obj = {
	name: "旺财",
	dance: function(){
		console.log(this.name + "在跳舞");
	}
}
```