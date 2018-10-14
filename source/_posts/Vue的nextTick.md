title: vue的nextTick机制
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-05-2 15:32:00
photos: http://p7wm7amg2.bkt.clouddn.com/vue.png
---

## nextTick用法
在下次DOM更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获得更新后的DOM

## 原理
1、JavaScript是单线程，时间点上只能做一个事情
2、任务分同步异步
3、异步任务大致分为：宏任务、微任务
4、所有同步任务都在主线程上执行，形成一个执行栈
5、主线程之外，有个任务队列。只要异步任务有了运行结果，就在任务队列之中放置一个事件
6、一旦执行栈中的所有同步任务执行完毕，轮询机制就会读取任务队列中的微任务，其次是宏任务，看看里面有哪些事件。对应哪些异步任务，于是不再等待，进入执行栈执行
7、事件轮询不断执行，看任务队列有没等着的

## vue实现
vue大多数情况下优先使用微任务，很少的地方使用宏任务

### vue对于nextTick的宏任务实现
*优先检测setImmediate，然后是MessageChannel支持情况*
```javascript
if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  macroTimerFunc = () => {
    setImmediate(flushCallbacks)
  }
}

else if (typeof MessageChannel !== 'undefined' && (
  isNative(MessageChannel) ||
  // PhantomJS
  MessageChannel.toString() === '[object MessageChannelConstructor]'
)) {
  const channel = new MessageChannel()
  const port = channel.port2
  channel.port1.onmessage = flushCallbacks
  macroTimerFunc = () => {
    port.postMessage(1)
  }
} 
```

*如果都不支持，会用setTimeout*
```javascript
else {
  /* istanbul ignore next */
  macroTimerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
```
### vue对于nextTick的微任务实现
*优先检测Promise*
```javascript
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  microTimerFunc = () => {
    p.then(flushCallbacks)
    // in problematic UIWebViews, Promise.then doesn't completely break, but
    // it can get stuck in a weird state where callbacks are pushed into the
    // microtask queue but the queue isn't being flushed, until the browser
    // needs to do some other work, e.g. handle a timer. Therefore we can
    // "force" the microtask queue to be flushed by adding an empty timer.
    if (isIOS) setTimeout(noop)
  }
}
```

如果不支持Promise，还是用宏任务
```javascript
else {
  // fallback to macro
  microTimerFunc = macroTimerFunc
}
```

## vue什么地方宏任务or微任务
DOM事件中用Vue.nextTick默认使用宏任务，其他地方使用Vue.nextTick默认是微任务

其实从源码中注释中可以看出Vue最开始都是使用微任务方式，后面出现了bug，才引入了宏任务方式


### 举个例子
```javascript
export default {
  data () {
    return {
      msg: 0
    }
  },
  mounted () {
    this.msg = 1
    this.msg = 2
    this.msg = 3
  },
  watch: {
    msg () {
      console.log(this.msg)
    }
  }
}
```
watch：对应一个对象，键是观察表达式，值是对应回调。值也可以是方法名，或者是对象，包含选项。在实例化时为每个键调用 $watch()

这段打印的是只输出一个3

*关于setImmediate、MessageChannel VS setTimeout*
为什么要优先创建macroTask而不是setTimeout？h5中规定setTimeout的最小时间时延是4ms，也就是说理想环境下异步回调最快也是4ms才能触发。Vue使用这么多函数来模拟异步任务，其目的只有一个，**就是让回调异步且尽早调用**。而MessageChannel和setImmediate的延迟明显是小于setTimeout的

因为Vue的事件机制是通过事件队列来调度执行，会等主进程执行空闲后进行调度，所以先回去等待所有的进程执行完成之后再去一次更新。这样的性能优势很明显，比如：
在mounted的时候变量值被循环加加1000次。每次加加时，都会根据响应式触发setter->Dep->Watcher->update->run
如果每次++都会直接操作DOM更新视图，会非常消耗性能。**所以Vue实现了一个queue队列，在下一个Tick（或者是当前Tick的微任务阶段）的时候会统一执行queue中的Watcher的run**。
**同时，拥有相同id的Watcher不会被重复加入到该queue中Watcher的run**。所以不会执行1000次Watcher的run。最终更新视图只会将test对应的DOM的0变成1000.保证更新视图操作DOM的动作是在当前执行栈完以后下一个Tick（或者是当前Tick的微任务阶段）的时候调用，大大优化了性能