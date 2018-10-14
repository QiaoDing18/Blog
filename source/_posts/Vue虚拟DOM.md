title: Vue虚拟DOM
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-04-27 17:15:00
photos: http://p7wm7amg2.bkt.clouddn.com/vue.png
---
DOM是文档对象模型的简写，在浏览器中我们可以通过JS来操作DOM，但是这样的操作性能很差，于是虚拟DOM应运而生。

虚拟DOM就是在JS中模拟DOM对比树来优化DOM操作的一种技术或思路

## VNode虚拟DOM对象
包含了哪些属性
* tag：当前结点的标签名
* data：当前结点的数据对象，有一大堆
* children：数组类型，包含了当前结点的子节点
* text：当前结点的文本，一般文本节点或注释结点会有这个属性
* elm：当前虚拟结点对应的真实的DOM结点
* ns：结点的namespace
* context：编译作用域
* functionalContext：函数化组件的作用域
* key：节点的key属性，用于作为结点的标识，有利于patch的优化
* child: 当前节点对应的组件实例
* parent: 组件的占位节点
* isStatic: 静态节点的标识
* isRootInsert: 是否作为根节点插入，被transition包裹的节点，该属性的值为false
* isComment: 当前节点是否是注释节点
* isCloned: 当前节点是否为克隆节点
* isOnce: 当前节点是否有v-once指令

## VNode分类
VNode可以理解为vue框架的虚拟DOM的基类，通过new实例化的VNode大致可以分为几类
* EmptyVNode：没有内容的注释结点
* TextVNode：文本节点
* ElementVNode：普通元素节点
* ComponentVNode：组件节点
* CloneVNode：克隆节点，可以是以上任意类型的节点，唯一的区别在于isCloned属性为true

## patch原理
### patch函数接收6个参数
* oldVnode：旧的虚拟结点或旧的真实DOM节点
* vnode：新的虚拟结点
* hydrating：是否要跟真实DOM混合
* parentElm：父节点
* refElm：新节点将插入到refElm之前

### patch的策略
1、如果vnode不存在但是oldVnode存在，说明意图是要销毁老节点，那么就调用API来销毁旧的结点
2、如果oldVnode不存在但是vnode存在，说明意图是要创建新节点，那么就调用createElm来创建新节点
3、当vnode和oldVnode都存在时
	* 如果oldVnode和vnode是同一个节点，就调用patchVnode来进行patch
	* 当vnode和oldVnode不是同一个节点时，如果oldVnode是真实dom节点或hydrating设置为true，需要用hydrate函数将虚拟DOM和真实DOM进行映射，然后将oldVnode设置为对应的虚拟DOM，找到oldVnode.elm的父节点，根据vnode创建一个真实dom节点并插入到该父节点中oldVnode.elm的位置

**总结一下**
1、新虚拟DOM不存在，旧的存在，就说明要删除旧的了
2、旧虚拟DOM不存在，新的存在，就说明要创建新的了
3、新旧虚拟DOM都存在：
	（1）是同一个节点，调用patchVnode
	（2）不是同一个节点，如果是真实DOM，就将虚拟DOM映射和真实DOM进行映射，然后把oldVnode设置为对应的虚拟DOM，找到父节点，新的真实DOM节点并插入到该父节点中


重点是**patchVnode**函数，因为真正的patch算法是由它来实现的（patchVnode中更新子节点的算法其实是在updateChildren函数中实现的）

## patchVnode算法
1、如果oldVnode跟vnode完全一致，不做任何事情
2、如果oldVnode跟vnode都是静态节点，且具有相同的key，当vnode是克隆节点或是v-once指令控制的节点时，只需要把新的相关属性赋值到vnode上
3、如果不是文本或注释节点，两者都有子节点，且子节点不完全一致，就需要更新子节点的操作，更新操作通过比较oldStartVnode和newStartVnode的属性，属性进行patch操作
4、如果只是文本或注释结点，只需更新文本内容


# 虚拟DOM的生命周期
1、init：虚拟DOM初始化时
2、create：虚拟DOM创建时
3、prepatch：patch之前
4、insert：虚拟DOM插入后
5、update：虚拟DOM更新前
6、postpatch：patch之后
7、remove：虚拟DOM除移时
8、destroy：虚拟DOM销毁时

vue组件的声明周期底层其实就是依赖于虚拟DOM的生命周期，vue为自己的组件虚拟DOM已经写好了默认的init/prepatch/insert/destroy。而vue组件的mounted/activated就是在insert中触发的，deactivated就是在destroy中触发的