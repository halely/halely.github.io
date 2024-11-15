---
title: TypeScrip-学习（七）命名空间、三斜线指令、声明文件、Mixins混入
date: 2023-05-27 16:50:03
cover: https://www.foodiesfeed.com/wp-content/uploads/2021/05/fresh-coconut.jpg
tags:
  - JavaScript
  - TypeScript
categories: 
  - 前端学习
---


# 命名空间 namespace

在工作中无法避免全局变量造成的污染，`TypeScript`提供了`namespace` 避免这个问题出现

- 内部模块，主要用于组织代码，避免命名冲突。
- 命名空间内的类默认私有
- 通过 export 暴露
- 通过 namespace 关键字定义

> **TypeScript与ECMAScript 2015一样，任何包含顶级import或者export的文件都被当成一个模块。相反地，如果一个文件不带有顶级的import或者export声明，那么它的内容被视为全局可见的（因此对模块也是可见的）**

`实例`

```ts
//命名空间中通过export将想要暴露的部分导出,否则无法读取其值
namespace a {
    export const Time: number = 1000
    export const fn = <T>(arg: T): T => {
        return arg
    }
    fn(Time)
}
namespace b {
     export const Time: number = 1000
     export const fn = <T>(arg: T): T => {
        return arg
    }
    fn(Time)
}
a.Time
```

`嵌套命名`

```ts
namespace a {
    export namespace b {
        export class Vue {
            parameters: string
            constructor(parameters: string) {
                this.parameters = parameters
            }
        }
    }
}
let v = a.b.Vue
new v('1')
```

`抽离命名空间`

```ts
//a.ts
export namespace A {
    export const a = 1
}
//b.ts

import {A} from './a.ts'
console.log(A.a)
//简化名称
import X=A.a;
console.log(x)
```

`合并命名空间`

```ts
//重名的命名空间会合并
export namespace A {
    export const a = 1
}
export namespace A {
    export const b =a
}
// 会合并
```

# 三斜线指令

三斜线指令是包含单个XML标签的单行注释。 注释的内容会做为**编译器指令使用**
> 三斜线指令仅可放在包含它的文件的最顶端。 一个三斜线指令的前面只能出现单行或多行注释，这包括其它的三斜线指令。 如果它们出现在一个语句或声明之后，那么它们会被当做普通的单行注释，并且不具有特殊的涵义。

`/// <reference path="..." />`: 是三斜线指令中最常见的一种，它用于声明文件间的 依赖

**三斜线引用**告诉编译器在编译过程中要引入的额外的文件，你也可以把它理解能`import`，它可以告诉编译器在编译过程中要**引入的额外的文件**

```ts
//a.ts
namespace A {
    export const fn = () => 'a'
}
//b.ts
namespace A {
    export const fn2 = () => 'b'
}
//c.ts
///<reference path="./a.ts" />
///<reference path="./b.ts" />
console.log(A);
```

当你想引入声明文件的时候可以使用，如引入`nodeJS` `/// <reference types="node" />`

# 声明文件 d.ts

在开发中我们现在会很频繁的使第三方库，但是我们如果使用TS开发，当使用第三方库时，我们需要引用它的声明文件，才能获得对应的代码补全、接口提示等功能。一般的声明我们需要使用关键字`declare`  。

```ts
declare var //声明全局变量
declare function //声明全局方法
declare class //声明全局类
declare enum //声明全局枚举类型
declare namespace //声明（含有子属性的）全局对象
interface 和 type //声明全局类型
/// <reference /> 三斜线指令
```

当我们使用第三方插件发现报错了，无法读取其值，那么有可能你的第三方没有提供声明文件列如`express` ，那么你可以尝试使用`npm install @types/node -D`去下载社区的声明文件，
当然也可以简单的手写

```ts
index.ts
import express from 'express'
const app = express()
const router = express.Router()
app.use('/api', router)
router.get('/list', (req, res) => {
    res.json({
        code: 200
    })
})
app.listen(9001,()=>{
    console.log(9001)
})
//express.d.ts
declare module 'express' {
    interface Router {
        get(path: string, cb: (req: any, res: any) => void): void
    }
    interface App {
 
        use(path: string, router: any): void
        listen(port: number, cb?: () => void): void
    }
    interface Express {
        (): App
        Router(): Router
 
    }
    const express: Express
    export default express
}
```

# Mixins混入

怎么说呢，就像是合并对象
`对象混入`
就可以使用 `Object.assign` 合并多个对象

```ts
interface Name {
    name: string
}
interface Age {
    age: number
}
interface Sex {
    sex: number
}
 
let people1: Name = { name: "小满" }
let people2: Age = { age: 20 }
let people3: Sex = { sex: 1 }
 
const people = Object.assign(people1,people2,people3);//people 会被推断成一个交差类型 Name & Age & sex;
```

`类的混入`

首先声明两个mixins类 **（严格模式要关闭不然编译不过）**

```ts
class A {
    type: boolean = false;
    changeType() {
        this.type = !this.type
    }
}
class B {
    name: string = '张三';
    getName(): string {
        return this.name;
    }
}
//首先应该注意到的是，没使用extends而是使用implements
class C implements A,B{
    type:boolean
    changeType:()=>void;
    name: string;
    getName:()=> string
}
//然后创建帮助函数，做混入操作。 它会遍历mixins上的所有属性，并复制到目标上去，把之前的占位属性替换成真正的实现代码
Mixins(C, [A, B])
function Mixins(curCls: any, itemCls: any[]) {
    itemCls.forEach(item => {
        Object.getOwnPropertyNames(item.prototype).forEach(name => {
            curCls.prototype[name] = item.prototype[name]
        })
    })
}
```
