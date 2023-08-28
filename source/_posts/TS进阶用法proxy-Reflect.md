---
title: TS进阶用法proxy & Reflect
date: 2023-08-24 10:46:10
cover: '/img/still.jpg'
tags:
  - JavaScript
  - TypeScript
---

# 前言

对于接触过`Vue3`的开发差不多都知道`ES6`添加了`Proxy`新特性,数据响应的技术就是运用了`Proxy`的代理拦截,但是和`Proxy`共生的还有`Reflect(反射)`.

# 开始

## proxy

>Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。

下面是 Proxy 支持的拦截操作一览，一共 13 种-[具体实例](https://es6.ruanyifeng.com/#docs/proxy)。

- `get(target, propKey, receiver)`：拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。
- `set(target, propKey, value, receiver)`：拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值。
- `has(target, propKey)`：拦截`propKey in proxy`的操作，返回一个布尔值。
- `deleteProperty(target, propKey)`：拦截`delete proxy[propKey]`的操作，返回一个布尔值。
- `ownKeys(target)`：拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for...in`循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而`Object.keys()`的返回结果仅包括目标对象自身的可遍历属性。
- `getOwnPropertyDescriptor(target, propKey)`：拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。
- `defineProperty(target, propKey, propDesc)`：拦截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。
- `preventExtensions(target)`：拦截`Object.preventExtensions(proxy)`，返回一个布尔值。
- `getPrototypeOf(target)`：拦截`Object.getPrototypeOf(proxy)`，返回一个对象。
- `isExtensible(target)`：拦截`Object.isExtensible(proxy)`，返回一个布尔值。
- `setPrototypeOf(target, proto)`：拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
- `apply(target, object, args)`：拦截 Proxy 实例作为**函数调用的操作**，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。
- `construct(target, args)`：拦截 Proxy 实例作为**构造函数调用**的操作，比如`new proxy(...args)`。

## Reflect

对象与`Proxy`对象一样，也是`ES6`为了操作对象而提供的新`API`。`Reflect`对象的设计目的有这样几个。

1. 将`Object`对象的一些明显属于语言内部的方法（比如`Object.defineProperty`），放到`Reflec`t对象上。现阶段，某些方法同时在`Object`和`Reflect`对象上部署，未来的新方法将只部署在Reflect对象上。也就是说，从Reflect对象上可以拿到语言内部的方法。
2. 修改某些`Object`方法的返回结果，让其变得更合理。比如，`Object.defineProperty(obj, name, desc)`在无法定义属性时，会抛出一个错误，而`Reflect.defineProperty(obj, name, desc)`则会返回false。
3. 让`Object`操作都变成函数行为。某些`Object`操作是命令式，比如`name in obj`和`delete obj[name]`，而`Reflect.has(obj, name)`和`Reflect.deleteProperty(obj, name)`让它们变成了函数行为。
4. `Reflect`对象的方法与Proxy对象的方法一一对应，**只要是`Proxy`对象的方法，就能在`Reflect`对象上找到对应的方法**。这就让`Proxy`对象可以方便地调用对应的`Reflect`方法，完成默认行为，作为修改行为的基础。**也就是说，不管Proxy怎么修改默认行为，你总可以在Reflect上获取默认行为**。

`Reflect`对象一共有 13 个静态方法-[具体实例](https://es6.ruanyifeng.com/#docs/reflect)。

- Reflect.apply(target, thisArg, args)
- Reflect.construct(target, args)
- Reflect.get(target, name, receiver)
- Reflect.set(target, name, value, receiver)
- Reflect.defineProperty(target, name, desc)
- Reflect.deleteProperty(target, name)
- Reflect.has(target, name)
- Reflect.ownKeys(target)
- Reflect.isExtensible(target)
- Reflect.preventExtensions(target)
- Reflect.getOwnPropertyDescriptor(target, name)
- Reflect.getPrototypeOf(target)
- Reflect.setPrototypeOf(target, prototype)

## 使用Proxy和Reflect实现观察者模式

```ts

/*
 * @proxy 代理13个方法 参数一模一样
 * @Reflect 反射13个方法 参数一模一样
 * @mobx  实现observable
*/


//proxy 代理：拦截
//支撑对象 数组 函数 set map
let person = { name: 'haleLy', age:27};
const listFns: Set<Function> = new Set();
const addAuthor = (cb: Function) => {
    if (!listFns.has(cb)) {
        listFns.add(cb)
    }
}

const observable = <T extends object>(params: T) => {
    return new Proxy(params, {
        set(target, key, value, receiver) {
            let result = Reflect.set(target, key, value, receiver);
            listFns.forEach(fn => fn())
            return result
        }
    })
}

const personObservable = observable(person);
console.log(personObservable)
addAuthor(() => {
    console.log('变化触发了')
})
personObservable.age = 28
```
