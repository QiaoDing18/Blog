title: React笔记新
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-04-27 16:36:00
photos: http://p7wm7amg2.bkt.clouddn.com/react.png
---


# 生命周期
React组件可以经历三个阶段
1、挂载阶段
2、更新阶段
3、卸载阶段

（1）挂载：这个阶段发生在组件被插入DOM时
（2）更新：这个阶段发生在组件被重新渲染成虚拟DOM并决定实际DOM是否需要更新
（3）卸载：这个阶段发生在组件从DOM中被删除时

------------------------------------------

## 挂载时
**1、getInitalState()
2、componentWillMount()
3、render()
4、componentDidMount()**

### getInitalState
第一个被调用的。它会在React将组件插入DOM之前被调用。如果想让组件有一个state，可以使用这个方法返回初始的组件state。

### componentWillMount
第二个被调用的。它会在React将组件即将插入DOM时被调用。
调用this.setState()方法会触发组件的render()方法，但这时调用React知道还没有开始渲染，所以仅仅调用render一次

### render
这时候第一次render

### compomentDidMount
第三个被调用的。它会在React将组件插入到DOM之后立即被调用。更新后的DOM现在可以被访问
可以在这里整合jQuery，Ajax请求，setTimeout定时器

------------------------------------------

## 卸载时
### componentWillUnmount
这个阶段React仅提供这一个方法
在React即将从DOM中删除并销毁组件之前被调用。
用于清理组件在安装或更新阶段创建的所有数据。

------------------------------------------

## 更新时
这时候React组件已经被插入到了DOM中。
这个DOM代表组件的当前状态，并且当状态改变，React需要评估一个新的状态如何改变先前渲染的DOM

### 生命周期更新阶段的方法
5个 按顺序执行
1、componentWillReceiveProps()
2、shouldComponentUpdate()
3、componentWillUpdate()
4、render()
5、componentDidUpdate()

### componentWillReceiveProps
更新阶段第一个被调用，就是当组件从父组件接收到新属性时被调用
在这个阶段调用多次this.setState多少次，都不会触发组件额外的渲染。React会优化下，把状态更新操作放在一起批量执行
最好在这个阶段操作state

### shouldComponentUpdate
第二个被调用
通过shouldComponentUpdate方法，可以决定组件的下一个状态是否触发组件的重新渲染。这个方法返回一个布尔值
一般为true，但是如果为false，之后的componentWillUpdate、render、componentDidUpdate就不会被调用了
如果跳过了render，就会阻止对该组件的重新渲染，这将提高应用程序的性能，因为DOM没有额外的改变

### componentWillUpdate
在React即将更新DOM之前被调用，
有两个参数，nextProps：下一个属性对象
		   nextState：下一个状态对象
不能在componentWillUpdate中使用this.setState


### componentDidUpdate
在React更新DOM之后会被立即调用。有以下两个参数
prevProps：上一个属性对象
prevState：上一个状态对象
使用这个方法来操作更新后的DOM或者执行渲染后的操作。


componentDidUpdate被调用之后，更新阶段结束。
当组件的状态更新或者父组件传来一个新属性时，一个新的阶段又开始了


# diff
React 通过制定大胆的策略，将 O(n^3) 复杂度的问题转换成 O(n) 复杂度的问题。
### diff策略
1、web UI中DOM节点*跨层级的移动操作特别少*，可以忽略不计
2、拥有*相同同类的两个组件将会生成相似的树形结构*，拥有不同类的两个组件将会生成不同的树形结构
3、对于同一层级的一组子节点，它们可以*通过唯一id进行区分*
基于以上三个前提策略，React 分别对 **tree diff、component diff 以及 element diff** 进行算法优化，事实也证明这三个前提策略是合理且准确的，它保证了整体界面构建的性能。

### tree diff
React对树的算法进行了简介明了的优化，即对树进行分层比较，**两棵树只会对同一层次的节点进行比较**
React 只会简单的考虑同层级节点的位置变换，而对于不同层级的节点，只有创建和删除操作。当根节点发现子节点中 A 消失了，就会直接销毁 A；当 D 发现多了一个子节点 A，则会创建新的 A（包括子节点）作为其子节点。此时，React diff 的执行情况：create A -> create B -> create C -> delete A。
![][1]

### component diff
React是基于组件构建应用，对于组件间的比较采取的策略也是简洁高效
1、如果是同一类型的组件，按照原策略继续比较虚拟DOM树
2、如果不是，则该组件判断为dirty component
3、对于同一类型的组件，有可能其虚拟DOM没有变化，如果能够确切的知道这点可以节省大量的diff运行时间，因此React允许用户通过shouldComponentUpdate来判断组件是否需要diff
![][2]


当D组件改为G组件时，即使这两个组件结构相似，一旦React判断D和G是不同类型的组件，就不会比较二者的结构，而是直接删除D组件，重新创建G组件以及其子节点。虽然两个组件是不同类型但结构相似。React diff会影响性能，但正如React官方博客所言：不同类型的组件是很少存在相似DOM树的机会，因此这种极端因素在开发中很难造成重大影响

### element diff
当节点处于同一层级时，Reactdiff提供了三种节点操作：插入、移动、删除
插入：新的组件类型不在之前的集合里，即是全新的结点，需要对新的结点执行插入操作
移动：在老集合有新组件类型，且结点是可更新的类型，就可以移动操作，可以复用以前的DOM结点
删除：老的组件类型，新的集合里也有，但对应的结点不同则不能直接复用和更新，需要执行删除操作，或者老的组件不在新集合里的，也需要执行删除操作



# 组件通信
![][3]
几种通信情况
1、父组件向子组件通信
2、子组件向父组件通信
3、跨级组件通信
4、没有嵌套关系组件之间的通信

## 父组件向子组件通信
父组件通过props向子组件传递需要的信息

## 子组件向父组件通信
1、利用回调函数
2、利用自定义事件机制

回调函数：在子组件中使用this.props.函数名   这个函数定义在父组件中，然后执行相应的操作，如配合setState

## 跨级组件通信
1、层层组件传递props
如A、B组件通信，先找到A和B的公共父组件，Ａ先向父组件通信，然后父组件通过props和B组件通信，此时父组件起的就是中间件的作用

2、使用context
context是一个全局变量，像是一个大容器，在任何地方都可以访问到，我们可以把要通信的信息放在context上，然后再其他组件中可以随意取到
但是React官方不建议大量使用context，尽管他可以减少逐层传递，但是当组件结构复杂的时候，我们并不知道context是从哪里传过来的；而且context是一个全局变量，全局变量会导致应用混乱

## 没有嵌套关系的组件通信
*使用自定义事件机制*
在componentDidMount事件中，如果组件挂载完成，再订阅事件；
在组件卸载的时候，在componentWillUnmount事件中取消事件订阅

--------------
### 总结
* 父组件向子组件通信: props
* 子组件向父组件通信: 回调函数/自定义事件
* 跨级组件通信: 层层组件传递props/context
* 没有嵌套关系组件之间的通信: 自定义事件

# Mixin
React的主流构建组件的方法是利用createClass创建，就是创造一个包含React方法Class类。这种实现，官方提供了非常有用的mixin属性。mixins接受一个数组

Mixin 就是用来定义一些方法，使用这个 mixin 的组件能够自由的使用这些方法（就像在组件中定义的一样），所以 mixin 相当于组件的一个扩展，在 mixin 中也能定义“生命周期”方法。

虽然组件的原则就是模块化，彼此之间相互独立，但是有时候不同的组件之间可能会共用一些功能，共享一部分代码。所以 React 提供了 mixins 这种方式来处理这种问题。







  