title: vue编译
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-05-21 16:18:21
photos: http://p7wm7amg2.bkt.clouddn.com/vue.png
---


vue使用虚拟DOM作为view层，render方法返回一个vdom结构，compiler的作用就是将template编译成render

## 编译
parser：将template字符串解析为html语法树，并且根据parse的平台相关的modules对语法树中的节点进行修改
optimizer：遍历语法树，发现static静态部分，将其变为常量(存储为静态引用)，这样在更新vdom树时，可以迅速跳过他们
codegen：根据语法树，生成render和staticRenderFns函数

### parser
parse调用html-parser，传入start，end，chars，comments等选项，在html-parser解析到相应位置进行处理
start：标签开始，传入tag，attrs，unary，执行preTransformNode，postTransformNode两个钩子函数，还对attrs进行处理，比如对于attrs中包含的v-for属性进行v-for语句验证，以及解析v-for语句中的iterator的名字
end：标识一个标签结果，管理堆栈状态
chars：标签的textContent部分，判断是否可以裁剪，解析绑定。比如hello 解析为‘hello’ + ——s(person.name)，这样在后面生成代码阶段可以直接赋予text参数(VNode(tag, data, children, text))。_s为VDom helpers，将其参数转化为字符串
comments：插入comment类型的节点

## optimizer
optimizer旨在为树的节点标上static和staticRoot属性

遍历一轮，标记static属性：
	1、判断node是否为static
	2、标记node的children是否为static，若存在nonstatic子节点，父节点更改为static = false
	
遍历二轮，标记staticRoot
	1、标记static活节点为staticRoot，这个节点type === 1 （一般是含有tag属性的节点）
	2、具有v-once指令的节点同样被标记为staticRoot
	3、为了避免过度优化，只有static text为子节点的节点不被标记为staticRoot
	4、标记节点children的staticRoot
	
## codegen
代码生成器的逻辑没有想象中那么复杂，里面逻辑很多，但是非常清楚
对于v-for标记的vnode
```javascript
function getFor(el, state, altGen, altHelper) {
    // ...
    return `${altHelper || '_l'}((${exp}),` +
    `function(${alias}${iterator1}${iterator2}){` +
      `return ${(altGen || genElement)(el, state)}` +
    '})'
}

// core/instance/render-helpers/render-list.js
function renderList(val, render) {
    // v-for的exp可以传手写数组，字符串，number，或vm.$data数据

    // 根据exp自动生成vnode数组
}
```

_ altHelper为ssr所用，browser端使用_l helper，但是ssr会使用_ssrList helper_

Vue的v-for非常灵活，使用极其方便，从这里可以看到Vue的codegen为了做了非常多的工作，让我们摆脱了许多脏活。

events生成
codegen中对于事件代码处理同样精彩。只需要一个@click.prevent="onClick"，Vue变为我们写出$event.stopPropagation，如果是@click = "show = !show"，Vue还会自动编写包装show = !show函数。

events中有两个不熟的知识：

passive标识

passive event listeners，passive有助于提高滚动流畅度。touch事件默认行为是滚动页面，绑定touch事件回调可以阻止默认滚动行为，没有passive选项，浏览器需要等待touch事件回调是否调用了event.preventDefault来决定它是否应该做出滚动行为，有passive选项，浏览器则不理会有没有event.preventDefault，直接滚动。

passive在parser中被解析处理，Vue支持passive的modifier

event.composing

出现在platforms/web/runtime/directives/model，搜索关于event.composing的资料，正好是yyx大神的博客DOM COMPOSITION EVENTS COMPATIBILITY NOTES。event.composing代表输入法输入状态，用event.composing做guarding，可以避免输入触发input事件。


在渲染函数被调用的时候。一系列带下划线的VNode生成函数会调用VNode的函数，设置其属性并通过children记录其子VNode节点，从而形成一个VNode树
```javascript
function anonymous() {
  // this 指向当前 Vue 实例, 这样 todos, completed 才能指向 Vue 实例属性
  with (this) {
    return _c(
      'div',
      {
        attrs: { "id": "root" }
      }, 
      [
        _c('h1',
          { staticClass: "todo-list" },
          [
            _v("Here's something to do")
          ]),
        _v(" "),
        _c('ul',
          _l((todos), function (todo) { return _c('li', [_v(_s(todo))]) })),
        _v(" "),
        (completed) ?
          _c('p', [_v("Completed!")]) :
          _c('p', [_v("Not completed...")])
      ]
    )
  }
}
```


_ c定义在 core/instance/render中，用来构造元素节点
```javascript
vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
```
createElement 较为重要, 它最经常被调用, 还要负责维护 VNode 之间的父子关系.


_ v用来构造文本节点
```javascript
export function createTextVNode(val: string | number) {
  return new VNode(undefined, undefined, undefined, String(val))
}
```


_ l用来构造列表节点, 它的参数是一个数组和一个用于生成数组成员节点的回调函数.
```javascript
export function renderList(
  val: any,
  render: (
    val: any,
    keyOrIndex: string | number,
    index?: number
  ) => VNode
): ?Array<VNode> {
  let ret: ?Array<VNode>, i, l, keys, key
  if (Array.isArray(val) || typeof val === 'string') {
    ret = new Array(val.length)
    for (i = 0, l = val.length; i < l; i++) {
      ret[i] = render(val[i], i)
    }
  } else if (typeof val === 'number') {
    ret = new Array(val)
    for (i = 0; i < val; i++) {
      ret[i] = render(i + 1, i)
    }
  } else if (isObject(val)) {
    keys = Object.keys(val)
    ret = new Array(keys.length)
    for (i = 0, l = keys.length; i < l; i++) {
      key = keys[i]
      ret[i] = render(val[key], key, i)
    }
  }
  if (isDef(ret)) {
    ret._isVList = true
  }
  return ret
}
```
可以看到 v-if 并没有对应的函数, 我们可以从渲染函数中看到它是靠若干个三元组实现的.

当渲染函数执行完毕之后, VNode 树就构造好了, 并被交给 _ update 方法.
