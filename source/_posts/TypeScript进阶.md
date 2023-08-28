---
title: TypeScript进阶
date: 2023-08-28 11:47:34
cover: '/img/background/backdrop2.png'
tags:
  - JavaScript
  - TypeScript
---

# 类型兼容

所谓的`类型兼容性`，就是用于确定一个类型是否能赋值给其他的类型。**TypeScript中的类型兼容性是基于结构类型的（也就是形状），如果A要兼容B 那么A至少具有B相同的属性**。

## 协变

```ts
interface A {
    name: string,
    age: number,
    isMan: boolean
}

interface B {
    name: string,
    age: number
}

let a: A = {
    name: 'hale',
    age: 22,
    isMan: true
}

let b: B = {
    name: 'yzj',
    age: 20
}

// a=b; //异常 类型 "B" 中缺少属性 "isMan"，但类型 "A" 中需要该属性。
b=a;
```

>A B 两个类型完全不同但是竟然可以赋值并无报错B类型充当A类型的子类型，当子类型里面的属性满足A类型就可以进行赋值，也就是说**不能少可以多，这就是协变**。

## 逆变

 **逆变一般发生于函数的参数上面**

```ts
interface A {
    name: string,
    age: number,
    isMan: boolean
}

interface B {
    name: string,
    age: number
}

let fnA = (paramA: A) => { }
let fnB = (paramB: B) => { }

//fnB = fnA //异常不能将类型“(params: A) => void”分配给类型“(params: B) => void”。参数“paramA”和“paramB” 的类型不兼容。类型 "B" 中缺少属性 "isMan"，但类型 "A" 中需要该属性。

fnA=fnB
```

这里是和协变不一样的，协变是只能多不能少，但是**逆变是只能少不能多**，那么具体是什么原因呢？在我的理解下，不管这个函数如果赋值，其实执行的都是等号后面的函数，那么当`fnB = fnA`,执行的是`fnA`,而`fnB`对应的`类型B`不能完全覆盖`fnA`的`类型A`,所以TS认定这是不安全的，反之，`类型A`能够覆盖`类型B`,所以这样是安全的


> 那么如何双向协变呢？`tsconfig => strictFunctionTypes` 设置`为false` 支持双向协变 `fnA` `fnB` 随便可以来回赋值.