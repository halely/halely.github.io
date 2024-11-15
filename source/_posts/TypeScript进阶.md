---
title: TypeScript进阶
date: 2023-08-28 11:47:34
cover: '/img/background/backdrop2.png'
tags:
  - JavaScript
  - TypeScript
categories: 
  - 前端学习
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

# 内置进价用法

TypeScript内置高级类型`Partial Pick` ，我们学习一下,

## Partial

```ts
//=========源码=========

/**
 * Make all properties in T optional
  将T中的所有属性设置为可选
 */
type Partial<T> = {
    [P in keyof T]?: T[P];
};

//=========使用=========
type Person = {
    name:string,
    age:number
}
 
type p = Partial<Person>

//=========转换为=========

type p = {
    name?: string | undefined;
    age?: number | undefined;
}

```

- `keyof` 是将一个接口对象的全部属性取出来变成联合类型

- `in` 我们可以理解成for in P 就是key 遍历  keyof T  就是联合类型的每一项

- `?` 这个操作就是将每一个属性变成可选项

- `T[P]` 索引访问操作符，与 JavaScript 种访问属性值的操作类似

## Pick

**从类型定义T的属性中，选取指定一组属性，返回一个新的类型定义。**

```ts

//=========源码=========
/**
 * From T, pick a set of properties whose keys are in the union K
 */
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};


//=========使用=========
type Person = {
    name:string,
    age:number,
    text:string
    address:string
}
 
type Ex = "text" | "age"

type A = Pick<Person,Ex>

//=========转换成=========
type A = {
    age:number,
    text:string
}
```

## Readonly

```ts
/**
 * Make all properties in T readonly
 * 将T中的所有属性设置为只读
 */
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};

//=========使用=========
type Peron ={
    name:string,
    age:number
}

type p=Readonly<Peron>

//=========转换成=========
type p = {
    readonly name: string;
    readonly age: number;
}
```

##

```ts

//=========源码=========
/**
 * Construct a type with a set of properties K of type T
 */
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
//=========使用=========
type Peron = {
    name: string,
    age: number
}
type A = 'A' | 12 | 'N';
type p = Record<A, Peron>

//=========转换成=========
type p = {
    A: Peron;
    12: Peron;
    N: Peron;
}
```

## infer

infer 是TypeScript 新增到的关键字 充当占位符

定义一个类型 如果是数组类型 就返回 数组元素的类型 否则 就传入什么类型 就返回什么类型

```ts
//定义一个类型 如果是数组类型 就返回 数组元素的类型 否则 就传入什么类型 就返回什么类型

type TYPE<T> = T extends Array<any> ? T[number] : T;
type A=TYPE<(string | number)[]>
type B=TYPE<string>

//使用inter修改
type TYPE<T> = T extends Array<infer U> ? U : T;
type A=TYPE<(string | number)[]>
type B=TYPE<string>

```

**infer 类型提取**

```ts
type Arr = ['a', 'b', 'c', 'd', 'e', 'f'];
//提取头部元素
type FirstType<T extends any[]> = T extends [infer First, ...any[]] ? First : never;
type a = FirstType<Arr>

//提取尾部元素
type LastType<T extends any[]> = T extends [...any[], infer Last] ? Last : never;
type b = LastType<Arr>

//剔除第一个元素 
type EliminateFirst<T extends any[]> = T extends [unknown, ...infer Rest] ? Rest : never;
type c = EliminateFirst<Arr>

//剔除尾部元素
type EliminateLast<T extends any[]> = T extends [...infer Rest, unknown] ? Rest : never;
type d = EliminateLast<Arr>
```

**infer 梯归**

```ts
type a=[1,2,3,4]
type ReverseArr<T extends any[]>=T extends [infer First,...infer rest] ? [...ReverseArr<rest>,First]:T;
type b=ReverseArr<a>;//type b = [4, 3, 2, 1]
```
