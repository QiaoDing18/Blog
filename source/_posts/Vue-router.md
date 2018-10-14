title: 'Vue-router '
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-04-27 16:47:00
---


## 概述
使用router-link组件来导航
通过to属性指定链接
还有一个路由出口，把路由匹配到的组件渲染在里面
```html
<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```
如果使用模块化机制编程，導入Vue和VueRouter，要调用 Vue.use(VueRouter)
```javascript
// 1. 定义（路由）组件。
// 可以从其他文件 import 进来
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
// 我们晚点再讨论嵌套路由。
const routes = [
    { path: '/foo', component: Foo },
    { path: '/bar', component: Bar }
]

// 3. 创建 router 实例，然后传 `routes` 配置
// 你还可以传别的配置参数, 不过先这么简单着吧。
const router = new VueRouter({
  routes // （缩写）相当于 routes: routes
})

// 4. 创建和挂载根实例。
// 记得要通过 router 配置参数注入路由，
// 从而让整个应用都有路由功能
const app = new Vue({
  router
}).$mount('#app')

```

## 还是概述
路由的三个基本概念route，routes，router
1、route它是一条路由，它是单数， Home按钮  => home内容， 这是一条route,  about按钮 => about 内容， 这是另一条路由。
2、routes是一组路由，把上面的每一条路由组合起来，形成一个数组，[{home 按钮 =>home内容 }， { about按钮 => about 内容}]
3、router是一个机制，相当于一个管理者，它来管理路由。因为routes只是定义了一组路由，它放在哪里是静止的，当真正来了请求，就是当用户点击home 按钮的时候，怎么办？这时router 就起作用了，它到routes 中去查找，去找到对应的 home 内容，所以页面中就显示了 home 内容
4、客户端中的路由，实际上就是DOM元素的显示和隐藏。当页面中显示home内容的时候，about中的内容全部隐藏，反之也一样。客户端路由有两种实现方式：基于hash和基于history api

vue-router中实现路由还是相对简单的。因为我们页面中所有内容都是组件化的，我们只要把路径和组件对应起来就可以了，然后在页面中把组件渲染出来

## hash模式和history模式
### hash模式
hash模式背后的原理是*onhashchange*事件，可以在window对象上监听这个事件
hash发生变化的URL都会被浏览器记录下来，从而使浏览器的前进后退都可以用。
这样尽管浏览器没有请求服务器，但是页面状态和URL一一关联起来，这就是前端路由，单页面应用的标配
**只能改变#后的url，在请求时也不会带上#后的内容**

### history路由
随着history api的到来，前端路由开始进化了，前端的hashchange，只能改变#后面的URL片段，history api则给了前端完全的自由

利用了 HTML5 History Interface 中新增的 pushState() 和 replaceState() 方法
只是当它们执行修改时，虽然改变了当前的 URL，但浏览器不会立即向后端发送请求
#### 切换历史状态
包括back，forward，go三个方法，对应浏览器的前进，后退，跳转操作
history.go(-2) 后退两次
history.go(2)  前进两次
history.back() 后退
history.forward() 前进

#### 修改历史状态
包括pushState，replaceState两个方法，这两个方法接受三个参数stateObj，title，url

通过history.pushstate把页面的状态保存在第一个参数state对象中，当页面的url再变回这个url时，可以通过event.state取到这个state对象，其实滚动条的位置，阅读进度，组件的开关的这些页面状态都可以存储到state的里面。
history丢掉了#，但如果刷新，就回去请求服务器

**在hash模式下，前端路由修改的是#中的信息，而浏览器请求时不会携带**。
**在history下，可以自由修改path，当刷新时，如果服务器中没有响应或者资源，会导致404**