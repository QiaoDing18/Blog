title: vue1x升级相关问题
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-05-20 13:20:06
description: 尝试升级v1x遇到的一些问题，很多还没解决，后面会一一搞出来
photos: http://p7wm7amg2.bkt.clouddn.com/vue.png
---

== 升级1x问题汇总 ==
1	twoway 双向绑定问题
2	生命周期ready替换为mounted
3	stylus和指令有关系？
4	v-bind 代替 \{ \{  \} \}
5	ref 代替 v-ref
6	sync问题 $emit解决 （未解决）
7	v-for ()中value和index的顺序反转
8	vue transition 升级  用钩子函数 
https://segmentfault.com/q/1010000007738500
9	这个没明白
￼refactor number to a v-model modifier: v-model.number="xx"
￼
10	路由的钩子函数升级 activate 升级为 beforeRouteEnter
11	未解决
￼check vue.directive('xxx') to make sure its syntax has been update and for anything beyond simple DOM manipulations
12	计算属性的缓存验证被移除了
https://cn.vuejs.org/v2/guide/migration.html#%E8%AE%A1%E7%AE%97%E5%B1%9E%E6%80%A7￼

13	vue.config.debug取消
不再需要，因为警告信息将默认在堆栈信息里输出
14	router.go => router.push
15	v-link指令更新为router-link标签 