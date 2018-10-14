title: Vue中sync前世今生
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-05-27 12:12:12
description: Vue中sync前世今生
photos: http://p7wm7amg2.bkt.clouddn.com/vue.png
---

## Vue 1.x 里
在Vue 1.x版本中，强大的sync控制的数据使父组件的变量被子组件随意修改

```html
<div id="father">
	<child :need-msg.sync="msg"></child>
    <button @click="changeNeed">加加加</button>
</div>
<script>
Vue.component('child', {
	template:`<div>
									<p>{{ needMsg }}</p>
									<button @click.stop="reduceNeed">减减减	</button>
	    					</div>`,
	props: ['needMsg'],
	methods: {
    reduceNeed () {
     		this.needMsg--;
     		this.$emit('update:needMsg', this.needMsg);
    	}
	},
});
new Vue({
	el: '#father',
	data: {
		msg: 1
	},
	methods: {
		changeNeed () {
			this.msg++;
		}
	}
})
</script>
```
父组件控制加，子组件控制减。只需要加上sync就可以实现双向数据流

*在有些情况下，我们可能需要对一个 prop 进行“双向绑定”。不幸的是，真正的双向绑定会带来维护上的问题，因为子组件可以修改父组件，且在父组件和子组件都没有明显的改动来源。*

于是在Vue 2.0开始取消了sync的使用

在使用sync时会有报错
\[Vue warn]: Avoid mutating a prop directly since the value will be overwritten whenever the parent component re-renders. Instead, use a data or computed property based on the prop's value. Prop being mutated: "needMsg" 
(found in \<Child>)

报错意思是父组件再去修改prop值会导致覆盖

# Vue 2.0 开始
父组件依然可以通过prop向子组件传递值，但是子组件对于值的修改将不会直接影响到父组件。
sync消失后取而代之的是$emmit和$on这样通过自定义事件在父组件监听子组件的变动
这样依然可以实现双向数据绑定的效果，同时将子组件、父组件状态的代码区分
sync再见吧

## Vue 2.3 重出江湖
在父组件监听时需要这样写
```javascript
v-bind:title="doc.title"
v-on:update:title="doc.title = $event"
```
为了解决这种复杂的写法，同时为了方便Vue 1.x升级时不用做大量的代码修改，在Vue 2.3版本重新复出了sync，这次的sync只是一个语法糖，用来取代在父组件监听时如上复杂的写法

同时也要注意
*将 v-bind.sync 用在一个字面量的对象上，例如 v-bind.sync=”{ title: doc.title }”，是无法正常工作的，因为在解析一个像这样的复杂表达式的时候，有很多边缘情况需要考虑。*