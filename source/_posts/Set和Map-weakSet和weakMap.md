---
title: Set和Map & weakSet和weakMap
date: 2023-08-28 15:03:45
tags:
 - Javascript
---

# 前言

在我们`ES5`的开发过程中，我们很多时间都在使用`array`和`object`,数组的去重对象的遍历等等都是为手熟尔，在`ES6`又添加了`Set和Map`和弱类型`weakSet和weakMap`

# 详情

**1. Set**
集合是由一组无序且唯一(即不能重复)的项组成的，可以想象成集合是一个既没有重复元素，也没有顺序概念的数组
操作方法：

- add(value)：添加某个值，返回 Set 结构本身。
- delete(value)：删除某个值，返回一个布尔值，表示删除是否成功。
- has(value)：返回一个布尔值，表示该值是否为 Set 的成员。
- clear()：清除所有成员，无返回值。
- size: 返回set数据结构的数据长度

```ts
let setH=Set<number> =new Set([1,2,3,4]);
setH.add(5);//Set(5) {1, 2, 3, 4, 5}
setH.has(5) //true
setH.delete(5);//true
setH.size //4
setH.clear(); //清除
//去重操作
let arr = [...new Set([1,1,1,2,2,3,4,5,5,5,5])]
console.log(arr); //[ 1, 2, 3, 4, 5 ]
```

**2. Map**
它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键，是一种更完善的 Hash 结构实现。如果你需要“键值对”的数据结构，Map 比 Object 更合适

```ts
let obj = { name: 'haleLy' }
let map: Map<object, Function> = new Map()
map.set(obj, () => 123)
map.get(obj)
map.has(obj);//true
map.delete(obj)//true
map.size;//0
```

 >操作方法同`Set`,区别是`Map`的添加为`set()`操作方法

**3. weakSet和weakMap**

weak 在英文中是弱的异常，而weakSet和WeakMap的**键**都是**弱引用**，前提也是**必须使用引用类型去定义**，不会被计入垃圾回收机制。

>首先obj引用了这个对象 `+ 1`，aahph也引用了 `+ 1`，wmap也引用了，但是**不会  + 1**，应为他是弱引用，不会计入垃圾回收，因此 obj 和 aahph 释放了该引用 weakMap 也会随着消失的，但是有个问题你会发现控制台能输出，值是取不到的，应为V8的GC回收是需要一定时间的，你可以延长到500ms看一看，并且为了避免这个问题不允许读取键值，也不允许遍历，同理weakSet 也一样

```ts
let obj:any = {name:'孙亚龙'} //1
let aahph:any = obj //2
let wMap:WeakMap<object,string> = new WeakMap()
 
wMap.set(obj,'爱安徽潘慧') //2 他的键是弱引用不会计数的
 
obj = null // -1
aahph = null;//-1
//v8 GC 不稳定 最少200ms
 
setTimeout(()=>{
    console.log(wmap)
},500)
```
