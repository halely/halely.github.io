---
title: 实战TS编写发布订阅模式
date: 2023-08-23 15:12:55
cover: https://www.foodiesfeed.com/wp-content/uploads/2023/08/shrimps-pad-thai.jpg
tags:
  - JavaScript
  - TypeScript
categories: 
  - 前端学习
---

# 前言

作为一个前端或多或少都知道**发布订阅模式**，那么他到底是什么呢？
其实我们在日常开发早已用到了发布订阅模式例如`addEventListener`，`Vue evnetBus`

>发布订阅模式又叫观察者模式，它定义了一种一对多的关系，让多个订阅者对象同时监听某一个发布者，或者叫主题对象，这个主题对象的状态发生变化时就会通知所有订阅自己的订阅者对象，使得它们能够自动更新自己。

**举例**：比如我们刷b站，关注了一个up主，他直播或者发新动向都会提醒呢，然后你就知道，我们关注就是订阅或者叫注册，当他有新动向的时候会提醒你，这就是发布，这里面有三个角色：发布者（up主）、订阅者（你自己）、调度者（平台）。

# 实现

我们定义一个对象，对象有四个方法：

- `on`:订阅/监听
- `emit`:发布/注册
- `once`:只执行一次
- `off`:解除绑定

```ts
//定义类接口
interface EventFace {
    on: (name: string, fn: Function) => void;
    emit: (name: string, ...arg: Array<any>) => void;
    off: (name: string, fn: Function) => void;
    once: (name: string, fn: Function) => void;
}
interface ListFn {
    [key: string]: Array<Function>
}
//创建实现类
class DisPatch implements EventFace {
    list: ListFn
    constructor() {
        this.list = {}
    }
    //订阅/监听
    on(name: string, fn: Function) {
        let callbackList = this.list[name] || [];
        callbackList.push(fn)
        this.list[name] = callbackList;
    }
    //发布/注册
    emit(name: string, ...arg: Array<any>) {
        let eventNames = this.list[name]
        if (eventNames) {
            eventNames.forEach(el => {
                el.apply(this, arg)
            })
        } else {
            console.error(`名称错误${name}`)
        }
    }
    //解除绑定
    off(name: string, fn: Function) {
        let eventNames = this.list[name];
        if (eventNames && fn) {
            let index = eventNames.findIndex(fns => fns === fn)
            eventNames.splice(index, 1)
        } else {
            console.error('该事件未监听');
        }
    }
    //只执行一次
    once(name: string, fn: Function) {
        let decor = (...args: Array<any>) => {
            fn.apply(this, args)
            this.off(name, decor)
        }
        this.on(name, decor)
    }

}
//创建实例
const o = new DisPatch();
//注册订阅/监听
o.on('post', (...res) => {
    console.log(1, res)
})
const fn2 = (...res) => {
    console.log(2, res)
}
o.on('post', fn2)
o.once('post',(...res) => {
    console.log(3, res)
})
//发布/注册
o.emit('post', 1, false, { name: 'hale' })
o.off('post', fn2)
```