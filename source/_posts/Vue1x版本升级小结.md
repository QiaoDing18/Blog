title: Vue 1.x升级相关问题2.0
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-05-25 08:32:00
description: Vue 1.x升级相关问题2.0
photos: http://p7wm7amg2.bkt.clouddn.com/vuediff.png
---

在升级Vue 1.x项目时遇到了很多情况，小结一下

## 工具
[vue-migration-helper]('https://github.com/vuejs/vue-migration-helper')
在运行了vue-migration-helper之后会提供一些升级信息，根据提示信息修改相应代码

总结一下改动比较多的地方

## 双向数据绑定
`Replace :xxx.sync="ooo" with :visible="ooo", then $emit an event from the child component to trigger an update to xxx in the parent`
在Vue 2.3以下版本中，数据流被限制，取消了sync，来解决父组件受到子组件不必要的影响的问题。
解决的办法就是使用自定义事件向父组件传参
Vue 2.3中sync配合v-bind使用，被扩展成为一个自动更新父组件属性的v-on监听器


## 生命周期
![](http://p7wm7amg2.bkt.clouddn.com/vuediff.png)

主要的ready向mounted这两个钩子函数的转变。
1.x中的compiled感觉意义并不大

## transition
Vue 2.x中废弃掉了之前的transition属性，使用transition标签，并指定name属性实现效果

而且在Vue 2.x中的transition的钩子函数可以在methods中调用，废弃了之前的全局Vue.transition方法
```javascript
// Vue 1.x
Vue.transition('xxx', {
	beforeEnter: function( ){
		...
	},
	enter: function( ){
		...
	}
	...
})
// Vue 2.x
methods: {
	beforeEnter( ){
		...
	}
	enter( ){
		...
	}
	...
}
```

## 过滤器
在Vue 2.x中不再提供内置过滤器，需要引入外部库，比如moment.js来计算时间
```html
// Vue 1.x
<div>{{ item.appDate | date 'yyyy-MM-dd' }}</div>

// Vue 2.x
<div>{{ item.appDate | date('yyyy-MM-dd') }}</div>
```

## Vue Router
* router.go() 改为 router.push()
* router.map()被routes数组取代
* router-link 改为 v-link
* route.refresh 改为 route.meta.refresh

## 其他改动
* v-for中废弃了$index
* { { { xxoo } } } 改为 v-html
* v-el 和 v-ref 合并成ref属性