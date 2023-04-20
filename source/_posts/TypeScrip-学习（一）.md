---
title: TypeScrip 学习（一）
cover: 'https://www.foodiesfeed.com/wp-content/uploads/2023/04/green-spring-salad-with-a-turquoise-background.jpg'
date: 2023-04-20 16:05:32
tags:
  - typeScript
---

# 前言

在目前的前端方向，typeScript占的重要性越来越大，记录和分享自己的学习旅程
那么什么是typeScript？
TS是JS的超集，所以JS基础的类型都包含在内

# 起步

## 安装

``` cmd
npm install typescript -g 
tsc -v 
```

以上为安装和查看安装的版本

我们创建一个ts文件，然后使用 `tsc [文件名]` 就可以执行生成一个解析的js文件
当我们需要配置ts我们可以执行`tsc -init`去创建 `tsconfig.json` 文件
例如我们取消严格模式的时候可以在该文件中修改`strict:false`

## 基础类型

基础类型：`Boolean`、`Number`、`String`、`null`、`undefined` 以及 ES6 的 `Symbol`和 ES10 的 `BigInt`。

1. **字符串类型**

    ```js
    //字符串是使用string定义的
    let a: string = 'hale'
    //普通声明

    //也可以使用es6的字符串模板
    let str: string = `ly${a}`
    ```

2. **数字类型**

    ```js
    let notANumber: number = NaN;//Nan
    let num: number = 123;//普通数字
    let infinityNumber: number = Infinity;//无穷大
    let decimal: number = 6;//十进制
    let hex: number = 0xf00d;//十六进制
    let binary: number = 0b1010;//二进制
    let octal: number = 0o744;//八进制s
    //支持十六进制、十进制、八进制和二进制；
    ```

3. **布尔类型**

    ```js
    let booleand: boolean = true //可以直接使用布尔值

    //注意 使用构造函数 Boolean 创造的对象不是布尔值：
    let createdBoolean: boolean = new Boolean(1) 
    //这样会报错 应为事实上 new Boolean() 返回的是一个 Boolean 对象 
    //所以需要写成
    let createdBoolean1: Boolean = new Boolean(1) 
    //or
    let createdBoolean2: boolean = Boolean(1) 
    ```

4. **空值类型**

   `JavaScript` 没有空值（Void）的概念，在 `TypeScript` 中，可以用 `void` 表示没有任何返回值的函数

    ```js
    function voidFn(): void {
       console.log('test void')
    }
    //void 类型的用法，主要是用在我们不希望调用者关心函数返回值的情况下，比如通常的异步回调函数
    ```

    **void也可以定义undefined 和 null类型,但是这需要取消严格模式**

5. **Null和undefined类型**

     ```js
    let u: undefined = undefined;//定义undefined
    let n: null = null;//定义null
    ```

    >void 和 undefined 和 null 最大的区别:undefined 和 null 是所有类型的子类型。也就是说 undefined 类型的变量，可以赋值给 string 类型的变量：

     ```js
     //非严格模式
    let u: undefined = undefined;//定义undefined
    let n: null = null;//定义null
    let v:void=undefined
    let str='hale';
    str=u;
    str=n;
    str=v; // TODO 不支持
    ```

## 任意类型

