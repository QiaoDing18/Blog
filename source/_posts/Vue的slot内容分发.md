title: Vue的slot内容分发
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-05-23 10:11:09
description: 官网上关于slot解释的很生硬
photos: http://p7wm7amg2.bkt.clouddn.com/vue.png
---
## 概述
假如父组件需要在字组件内放一些DOM，那么这些DOM是显示、不显示、在哪显示、如何显示，就是由slot分发负责

## 默认情况下
父组件在子组件内直接写的的内容，是不显示的
```html
<div id="app">  
    <children>  
        <span>3g666</span>  
        <!--上面这行不会显示-->  
    </children>  
</div>  
<script>  
    var vm = new Vue({  
        el: '#app',  
        components: {  
            children: {    //这个无返回值，不会继续派发  
                template: "<button>为了明确作用范围，所以使用button标签</button>"  
            }  
        }  
    });  
</script> 
```
显示一个button，不包含span标签里面的内容


## 单个slot
简单来说，只使用这个标签，可以将父组件放在子组件的内容，放到让他显示的地方
```html
<div id="app">  
    <children>  
        <span>3g666</span>  
        <!--上面这行不会显示-->  
    </children>  
</div>  
<script>  
    var vm = new Vue({  
        el: '#app',  
        components: {  
            children: {    //这个无返回值，不会继续派发  
                template: "<button><slot></slot>为了明确作用范围，所以使用button标签</button>"  
            }  
        }  
    });  
</script>  
```
这样的结果就是
< button>< span>12345< / span>为了明确作用范围，所以使用button标签< / button>
即父组件放在字组件里的内容，插到了子组件的< slot>< / slot>位置
注意，即便有多个标签，会一起被插入，相当于用父组件放在子组件里的标签，替换了< slot>< / slot>这个标签

## 具名slot
将放在子组件里的不同html标签放在不同的位置
父组件在要分发的标签里添加slot='name名'属性
子组件在对应分发的位置的slot标签里，添加name='name名'属性
然后就会将对应的标签放在对应的位置
```html
<div id="app">  
    <children>  
        <span slot="first">12345</span>  
        <span slot="second">56789</span>  
        <!--上面这行不会显示-->  
    </children>  
</div>  
<script>  
    var vm = new Vue({  
        el: '#app',  
        components: {  
            children: {    //这个无返回值，不会继续派发  
                template: "<button><slot name='first'></slot>为了明确作用范围，<slot name='second'></slot>所以使用button标签</button>"  
            }  
        }  
    });  
</script>
```
显示结果：
< button>
< span slot="first">12345< / span>
为了明确作用范围，
< span slot="second">56789< / span>
所以使用button标签
< / button>

## 分发内容的作用域
被分发的内容的作用域，根据其所在模板决定，以上标签，在其父组件模板中，虽然其被子组件的children标签所包括，但由于他不再子组件的template属性中，因此不属于子组件，还是受父组件的控制
```html
<div id="app">  
    <children>  
        <span slot="first" @click="tobeknow">12345</span>  
        <span slot="second">56789</span>  
        <!--上面这行不会显示-->  
    </children>  
</div>  
<script>  
    var vm = new Vue({  
        el: '#app',  
        methods: { 
            tobeknow: function () {  
                console.log("It is the parent's method");  
            } 
        },  
        components: {  
            children: {    //这个无返回值，不会继续派发  
                template: "<button><slot name='first'></slot>为了明确作用范围，<slot name='second'></slot>所以使用button标签</button>"  
            }  
        }  
    });  
</script>
```
这个只有在点击文字12345时候，才会触发父组件的tobeknow
但是点击其他地方则没有影响

父组件模板的内容在父组件作用域内编译；子组件模板的内容在子组件作用域内编译

## 当没有分发内容时的提示
加入父组件没有子组件中放置有标签，或者是父组件在子组件中防止标签，但是有slot属性，而子组件中没有该slot属性的标签
那么子组件的slot就会失效没用
除非该slot标签内有内容，那么在无分发内容的时候，会显示该slot标签内的内容
```html
<div id="app">  
    <children>  
        <span slot="first">【12345】</span>  
        <!--上面这行不会显示-->  
    </children>  
</div>  
<script>  
    var vm = new Vue({  
        el: '#app',  
        components: {  
            children: {    //这个无返回值，不会继续派发  
                template: "<div><slot name='first'><button>【如果没有内容则显示我1】</button></slot>为了明确作用范围，<slot name='last'><button>【如果没有内容则显示我2】</button></slot>所以使用button标签</div>"  
            }  
        }  
    });  
</script> 
```
第一个slot因为父组件有对应的标签会被替换，第二个父组件中没用对应的slot标签，会直接显示slot中的内容

## 加入想控制子组件根标签的属性
1、由于模板标签是属于父组件的，因此，将子组件的指令绑定在模板标签里，是不可以的，他终归还是属于父组件
2、加入需要通过父组件控制子组件是否显示，比如v-if，v-show，那么这个指令显然是属于父组件的（例如放在父组件的data下面）可以将标签卸载子组件的模板上
```html
<div id="app">  
    <button @click="toshow">点击让子组件显示</button>  
    <children v-if="abc">  
    </children>  
</div>  
<script>  
    var vm = new Vue({  
        el: '#app',  
        data: {  
            abc: false  
        },  
        methods: {  
            toshow: function () {  
                this.abc = !this.abc;  
            }  
        },  
        components: {  
            children: {    //这个无返回值，不会继续派发  
                template: "<div>这里是子组件</div>"  
            }  
        }  
    });  
</script>  
```
说明：通过父组件（点击按钮，切换v-if指令的值）控制子组件是否显示
3、假如需要通过子组件是否显示，比如是否让他隐藏，那么这个指令显然是属于子组件的，这个控制显示的值应该放在子组件的data属性下，那么就不能像上面这么写，而是必须防止在子组件的根标签中。
```html
<div id="app">  
    <button @click="toshow">点击让子组件显示</button>  
    <children>  
        <span slot="first">【12345】</span>  
        <!--上面这行不会显示-->  
    </children>  
</div>  
<script>  
    var vm = new Vue({  
        el: '#app',  
        methods: {  
            toshow: function () {  
                this.$children[0].tohidden = true;  
            }  
        },  
        components: {  
            children: {    //这个无返回值，不会继续派发  
                template: "<div v-if='tohidden' @click='tohide'>这里是子组件</div>",  
                data: function () {  
                    return {  
                        tohidden: true  
                    }  
                },  
                methods: {  
                    tohide: function () {  
                        this.tohidden = !this.tohidden;  
                    }  
                }  
            }  
        }  
    });  
</script>  
```
这样，点击子组件会让子组件消失
点击父组件的按钮，通过更改子组件的控制隐藏的data属性，让子组件重新显示
子组件的指令绑定在子组件的模板之中