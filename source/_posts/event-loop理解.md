---
title: event loop理解
date: 2022-12-15 14:32:57
cover: 'https://w.wallhaven.cc/full/p9/wallhaven-p9op59.jpg'
tags:
 - javascript
---

# 前言

对于前端来说，`event loop` 是一个非常重要的知识点，因为这涉及到代码的执行顺序，如果不了解或者了解的不够，那么逻辑就可能出现纰漏。

# event loop 的由来

`js`是单线程的，如果某段程序需要等待一会再执行，后面的程序都会被阻塞，这样也就带来了一些问题。为了解决这个问题，js出现了**同步**和**异步**两种任务，两种任务的差异就在于执行的优先级不同。`event loop`就是对任务的执行顺序做了详细的规范

# 堆、栈、队列

在真正了解`event loop` 之前我们需要了解堆、栈、队列：

## 堆（Heap）

**堆**是一种数据结构，是利用完全二叉树维护的一组数据，堆分为两种，一种为最大堆，一种为最小堆
节点最大的堆叫做**最大堆或大根堆**，根节点最小的堆叫做**最小堆或小根堆**。
堆是**线性数据结构**，相当于一维数组，有唯一后继。

## 栈（Stack）

**栈**在计算机科学中是限定仅在**表尾**进行插入或删除操作的线性表。 栈是一种数据结构，它按照**后进先出(LIFO)**的原则存储数据，先进入的数据被压入栈底，最后的数据在栈顶，需要读数据的时候从栈顶开始弹出数据。
栈是只能在某一端插入和删除的特殊线性表。

## 队列（Queue）

特殊之处在于它只允许在表的前端`（front）`进行删除操作，而在表的后端`（rear）`进行插入操作，和栈一样，队列是一种操作受限制的线性表。
进行插入操作的端称为队尾，进行删除操作的端称为队头。 队列中没有元素时，称为空队列。

队列的数据元素又称为队列元素。在队列中插入一个队列元素称为入队，从队列中删除一个队列元素称为出队。因为队列只允许在一端插入，在另一端删除，所以只有最早进入队列的元素才能最先从队列中删除，故队列又称为先进先出`（FIFO—first in first out）`

# 同步和异步任务

异步任务：异步任务分为**宏任务**和**微任务**。
常见的微任务有：`Promise.then()`，`.then`中的逻辑是微任务；`process.nextTick（node环境）`。
常见的宏任务有：`setTimeout`、`setInterval`、`setImmediate（node环境)`、`xhr(发送网络请求)`，`callback`。
同步任务：除了上面的这些情况，都属于同步任务。

>执行顺序是:同步任务 -> 微任务 -> 宏任务。
同步任务和异步任务都是在主线程中执行，除非使用`web Worker`

# 什么是 event loop

事件循环（event loop）就是 任务在主线程**不断进栈出栈**的一个循环过程。任务会在将要执行时进入主线程，在执行完毕后会退出主线程。

**下面就是这个循环的步骤：**

1. 把同 **步任务队列** 或者 **微任务队列** 或者 **宏任务队列**中的任务放入主线程。
2. **同步任务** 或者 **微任务** 或者 **宏任务** 在执行完毕后会全部退出主线程。

**在实际场景下大概是这么一个顺序：**

1. 把同步任务相继加入同步任务队列。
2. 把同步任务队列的任务相继加入主线程。
3. 待主线程的任务相继执行完毕后，把主线程队列清空。
4. 把微任务相继加入微任务队列。
5. 把微任务队列的任务相继加入主线程。
6. 待主线程的任务相继执行完毕后，把主线程队列清空。
7. 把宏任务相继加入宏任务队列。无`time`的先加入，像网络请求。有`time`的后加入，像`setTimeout(()=>{},time)`，在他们中`time`短的先加入。
8. 把宏任务队列的任务相继加入主线程。
9. 待主线程的任务相继执行完毕后，把主线程队列清空。

# 实践

```js
setTimeout(()=>{console.log(6)},2000)    // 任务a
setTimeout(()=>{console.log(4)})     // 任务 b
setTimeout(()=>{console.log(5)})     // 任务 c
new Promise((rel)=>{              
console.log(1)                         // 任务 d
rel(3)                            
})
.then((res)=>{console.log(res)})     // 任务 e
console.log(2)                    // 任务 f
// 打印结果：1,2,3,4,5,6
```

下面使用`event loop`的概念来讲解上面的题目，总共四次循环：

```js
const tasks=[]        // 主线程任务队列
const syncTasks=[]    // 同步任务队列
const tinyTasks=[]    // 微任务队列
const macroTasks=[]   // 宏任务队列

// 第一次
syncTasks.push('d','f')  // 首先同步任务d,f相继加入同步任务队列
for(let t of syncTasks){tasks.push(syncTasks.pop())}   // 然后把同步任务队列的任务相继加入主线程   
console.log(tasks)   // ['d','f']
for(let t of tasks){tasks.pop()}  // 在主线程相继执行完毕后，会相继出栈    
console.log(tasks)    // []

// 第二次
tinyTasks.push('e')  // 首先微任务e加入微任务队列
for(let t of tinyTasks){tasks.push(tinyTasks.pop())}   // 然后把微任务队列的任务相继加入主线程   
console.log(tasks)  // ['e']
for(let t of tasks){tasks.pop()}  // 在主线程相继执行完毕后，会相继出栈    
console.log(tasks)    // []

// 第三次
macroTasks.push('b','c')   // 首先宏任务'b','c'加入宏任务队列
for(let t of macroTasks){tasks.push(macroTasks.pop())}   // 然后把宏任务队列的任务相继加入主线程   
console.log(tasks)   // ['b','c']
for(let t of tasks){tasks.pop()}  // 在主线程相继执行完毕后，会相继出栈    
console.log(tasks)    // []

// 第四次
macroTasks.push('a')   //  首先宏任务a加入宏任务队列
for(let t of macroTasks){tasks.push(macroTasks.pop())}   // 然后把宏任务队列的任务相继加入主线程   
console.log(tasks)   // ['a']
for(let t of tasks){tasks.pop()}  // 在主线程相继执行完毕后，会相继出栈    
console.log(tasks)    // []
```
