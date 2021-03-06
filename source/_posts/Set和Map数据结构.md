title: Set和Map数据结构
author: 乔丁
tags:
  - 前端
categories:
  - web
date: 2018-04-27 16:44:00
---


## Set
概念：类似于数组，成员的值都是唯一的，没有重复的值。Set本身是一个构造函数，用来生成一个Set数据结构。
```javascript
let s  = new Set();
[2, 3, 5, 4, 5, 2, 2].map(x => s.add(x))
for (i of s) { console.log(i); }      // 2 3 5 4
```
表明Set结构不会添加重复的值。Set函数也可以接收一个数组作为参数，用于初始化
特殊情况：
```javascript
let set = new Set();
set.add({});
set.size //1
set.add({});
set.size //2
```
NaN不等于自身，set内部的判断相等类似于精确相等（===）
>Set结构的实例属性
>Set.prototype.constructor：构造函数，默认就是set函数
>Set.prototype.size：返回Set实例的成员总数


>Set实例的方法
>add(value)：添加某个值，返回Set结构本身
>delete(value)：删除某个值，返回一个布尔值，表示是否成功
>has(value)：判断是否是Set的成员，返回布尔值
>clear()：清除所有成员，无返回

与Array.from将Set结构转化为数组
```javascript
function dedupe(array){
	return Array.from(new Set(array));
}
dedupe([1, 2, 3, 3])  //[1, 2, 3]
```

#### 四种遍历操作
>keys()：返回键名的遍历器
>values()：返回一个键值的遍历器
>entries()：返回一个键值对的遍历器
>forEach()：使用回调函数遍历每个成员

```javascript
let set = new Set(['red', 'green', 'blue']);

for(let item of set.keys()){
	console.log(item);
}//red		green		blue

for(let item of set.values()){
	console.log(item);
}//red		green		blue

for(let item of set.entries()){
	console.log(item);
}//["red", "red"]		["green", "green"]		["blue", "blue"]
```
也可以用for...of遍历
```javascript
for(let x of set){
	console.log(x)
}//red		green		blue
```
由于扩展运算符（...）内部使用for...of循环，所以也可以用于Set结构
```javascript
let arr = [...set];
//["red", "green", "blue"]
```
另一种数组去重的方法
```javascript
let arr = [1, 1, 2, 3];
let unique = [...new Set(arr)];//[1, 2, 3]
```
#### 扩展运算符：将数组转换成用逗号分隔的参数序列
```javascript
Math.max(...[14, 3, 77])
```
Set结构的forEach方法用于对每个成员执行某种操作，没有返回值
```javascript
let set = new Set([1, 2, 3]);
set.forEach((value, key) => console.log(value*2))
//2	 4  6
```
forEach方法本身的参数就是一个处理函数，处理函数参数依次是键值、键名、集合本身，forEach方法还可以有第二个参数，表示绑定的this对象

### WeakSet
>WeakSet结构与Set类似，也是不重复的值的集合。
>与Set有两个区别：
>1、WeakSet的成员只能是对象，而不能是其他类型的值
>2、WeakSet中的对象都是弱引用，即垃圾回收机制不考虑WeakSet对该对象
的引用。也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于WeakSet中。这个特点意味着无法引用WeakSet的成员，因此WeakSet是不可遍历的

## Map
Map结构的目的和基本用法：JS的对象本质上是键值对的集合（Hash结构），但是只能用字符串作为键。这给它的使用带来了很大的限制。比如[Object Object]
Map数据结构类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值(包括对象)都可以当作键。也就是说，Object结构提供了“字符串—值”的对应，Map结构提供了“值—值”的对应，是一种更为完善的Hash结构实现。
1、作为构造函数，Map也可以接受一个数组作为参数。该数组的成员是一个个表示键值对的数组

2、Map构造函数接受数组作为参数，实际上执行的是把key和value通过forEach，set进Map结构中

3、如果对同一个键多次赋值，后面的值将覆盖前面的值

4、如果读取一个未知的键，则返回undefined

5、只有对同一个对象的引用，Map结构才将其视为同一个键

6、Map的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键（解决了同属性名碰撞问题，扩展别人库时，使用对象作为键名就不用担心同名问题）

7、如果Map的键是一个简单类型的值（数字、字符串、布尔值），则只需要两个值严格相等，Map就将其视为一个键，包括0和-0。且NaN虽然不严格等于自身，但Map将其视为同一个键

相关代码：
```javascript
let map = new Map();

map.set(['a'], 555);
map.get(['a']); // undifined
// 表面是针对同一个键，但实际上这是两个值，内存地址不一样
```
>Map的属性方法
>1、size属性：返回Mao结构的成员数量
>2、set方法：set（key，value）
>3、get方法：get（key）返回key
>4、has方法：has（key）键是否在Map中
>5、delete方法：delete（key）删除键返回布尔
>6、clear方法：clear（）清除所有成员
>7、keys（）、values（）、entries（）、forEach（）遍历方法

>Map转化为其他数据结构
>#### 1、数组
>Map=>数组：...扩展运算符[...map]
>数组=>Map：数组直接传入Map
>#### 2、对象
>Map=>对象：Map所有键都是字符串
>```javascript
>let obj = Object.creat(null);
>for(let [k, v] of xxx]){
>  obj[k] = v;
>}
>```
>对象=>Map：
>```javascript
>for(let k of Object.key(obj)){
>  map.set(k, obj[k]);
>}
>```
>#### 3、JSON
>Map=>JSON
>```javascript
>fucntion strMapToJSON(strMap){
>  return JSON.stringify([strMapToJSON(strMap)]);//键名是字符串
>  return JSON.stringify([...map]);//键名非字符串
>}
>```
>JSON=>Map
>```javascript
>function jsonToStrMap(jsonstr){
>  return objToStrMap(JSON.parse(jsonstr));
>}
>```

### WeakMap
与Map类似，区别：只接受对象作为键名（null除外），而且键名所指向的对象不计入垃圾回收机制
