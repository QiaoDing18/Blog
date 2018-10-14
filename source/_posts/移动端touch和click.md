title: 移动端touch和click
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-04-27 19:30:00
---



## touch
1、touch事件在某些场景下存在点击穿透的问题
2、touchstart事件是触摸屏幕就会触发，touchend事件是手指离开屏幕就会触发，而有时候我们仅仅只是想滑动屏幕，但却会触发这两个事件。

移动端事件触发的事件顺序为 touchstart -> touchout -> click 


## click
click会有300ms的延迟，是因为苹果Safari中最早为了确定用户是双击还是单机而将ckick事件加了300ms的延迟，来等待用户的二次点击，但之后发展中300ms就显得卡顿。
FastClick的实现原理是在检测到touchend事件的时候，会通过DOM自定义事件立即出发模拟一个click事件，并把浏览器在300ms之后的click事件阻止掉。

解决300ms的方法
1、在html文档中添加meta标签
```html
<meta name="viewport" content="user-scalable=no"/>
<meta name="viewport" content="init-scale=1.0,maximum-scale=1.0" />
```
表明这个页面是可放缩的，那么双击放缩的功能就没有意义了，此时浏览器可以禁用默认的双击放缩行为并且去掉300ms的点击延迟。

2、改变默认的视口宽度
```html
<meta name="viewport" content="width=device-width" />
```
我们通过上述标签设置移动浏览器的宽度等于设备的视窗宽度。随着响应式的普及，很多站点都已经对移动端做过适配了，这时候就不需要双击缩放了。如果能识别出一个网站是响应式网站，那么浏览器就可以认为已经对移动端做了优化和适配，那么久无需双击操作了。
如果页面比较简单，可以用touch事件，如果页面比较复杂，直接用click事件，同意用click的事件的好处是不应担心页面点击穿透的问题。

**点击穿透：**
使用touchstart去代替click事件有两个不好的地方。
第一：touchstart是手指触摸屏幕就触发，有时候用户只是想滑动屏幕，却触发了touchstart事件，这不是我们想要的结果；
第二：使用touchstart事件在某些场景下可能会出现点击穿透的现象。
假如页面上有两个元素A和B。B元素在A元素之上。我们在B元素的touchstart事件上注册了一个回调函数，该回调函数的作用是隐藏B元素。我们发现，当我们点击B元素，B元素被隐藏了，随后，A元素触发了click事件。

这是因为在移动端浏览器，事件执行的顺序是touchstart > touchend > click。而click事件有300ms的延迟，当touchstart事件把B元素隐藏之后，隔了300ms，浏览器触发了click事件，但是此时B元素不见了，所以该事件被派发到了A元素身上。如果A元素是一个链接，那此时页面就会意外地跳转。

### 解决办法
1、只用touch
	把页面的所有click全部换成touch事件。a标签的href也是click，需要去掉换成js控制的跳转，或者不用a标签
	
2、只用click
	会带来300ms延迟

3、使用fastclick
	用这个取消300ms延迟，随后只用click
	
4、遮挡
	给上层的消失做一个fade效果，并设置动画大于300ms，这样当延迟的click触发时，就不会穿透到下方元素了
	
5、禁掉放缩/更改默认视口宽度
	不能放缩就不用等待300ms