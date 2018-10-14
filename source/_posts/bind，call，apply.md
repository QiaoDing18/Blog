title: bind，call，apply
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-04-27 15:28:00
---
call()和apply()就是改变函数的执行上下文，也就是this值。他们两个是Function对象的方法，每个函数都能调用。

```javascript
let apply1 = (num1, num2) => {
	return sum.apply(this, [num1, num2]);
}

let call1 = (num1, num2) => {
	return sum.call(this, num1, num2);
}
```
第一个参数是要指定的上下文，这里传了当前的this，就说明没改变sum函数的执行上下文。
**第二个参数是函数需要的参数，call需要一个个传，apply需要把这些参数放到数组中**

bind是es5中的方法
**bind是新创建了一个函数，然后把它的上下文绑定到bind()括号中的参数上，然后将它返回**

所以，**bind函数不会立即执行，而是返回一个改变了上下问的函数副本**，而call和apply是直接执行函数

```javascript
var button = document.getElementById("button"),
    text = document.getElementById("text");
button.onclick = function() {
    alert(this.id);     // 弹出text
}.bind(text);
```

**模拟bind**
```javascript
if(!function(){}.bind){
	Function.prototype.bind = function(context){
		var that = this;
		var args = Array.prototype.slice.call(arguments);
		return function(){
			return that.apply(context, args.slice(1));
		}
	}
}
```
首先，我们判断是否存在bind方法，然后，若不存在，向Function对象的原型中添加自定义的bind方法。
this指向调用他的对象
Array.prototype.slice.call(arguments)把arguments转换成真的数组
后面用闭包，把参数给apply