---
title: TypeScrip 学习（一）
cover: 'https://www.foodiesfeed.com/wp-content/uploads/2023/04/green-spring-salad-with-a-turquoise-background.jpg'
date: 2023-04-20 16:05:32
tags:
  - JavaScript
  - TypeScript
---

# 前言

目前，JavaScript已经成为编写网页和应用程序的主要语言，前端也在一个产品中的权重越来越大，但是因为JavaScript设计之初的先天问题，导致avaScript创建大型复杂Web应用系统很困难，也很难维护。

比如 `JavaScript`的数据类型是不确定的，可以进行**隐式转换**，导致很多错误在运行的时候才能发现，这就需要开发使用更多的精力去检查代码；

而TypeScript能解决这些问题，甚至他是JS的超集，JS有的我都有，JS没有的我也有，总之我是前端版*西厂对东厂*。

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

为了方便我们学习我们需要借用一些工具
全局安装 ts 的编译工具，使用 ts-node 可以将 ts 文件执行

- `npm i ts-node -g`
  - 使用：`ts-node index.ts`

   这时候我们就可以直接执行 `ts-node [文件名]` 且获取返回值了

- 安装 `ts-node` 依赖包：`npm install tslib @types/node -g`
- 安装声明文件: `npm i @types/node --save-dev`（node环境支持的依赖必装）

## TS类型

- TS 出现弥补的 JS 的类型缺失
- 众所周知，代码错误越早发现越好，代码编写 > 代码编译 > 代码运行   开发 > 测试 > 上线
- Vue2 使用 Flow 进行类型检查，后续 Vue3 也使用 Typescript 重写
- TS 代码要运行在浏览器，需要进行类型擦除，转换为 JS 代码
- TS 类型包含所有 JS 类型 `null`、`undefined`、`string`、`number`、`boolean`、`bigInt (ES10)`、`Symbol(ES6)`、`object`（数组，对象，函数，日期）
- 还包含 `void`、`never`、`enum`、`unknown`、`any` 以及 自定义的 `type` 和 `interface`

### **字符串类型**

```ts
//字符串是使用string定义的
let a: string = 'hale'
//普通声明
//也可以使用es6的字符串模板
let str: string = `ly${a}`
```

### **数字类型**

```ts
let notANumber: number = NaN;//Nan
let num: number = 123;//普通数字
let infinityNumber: number = Infinity;//无穷大
let decimal: number = 6;//十进制
let hex: number = 0xf00d;//十六进制
let binary: number = 0b1010;//二进制
let octal: number = 0o744;//八进制s
//支持十六进制、十进制、八进制和二进制；
```

### **布尔类型**

```ts
let booleand: boolean = true //可以直接使用布尔值

//注意 使用构造函数 Boolean 创造的对象不是布尔值：
let createdBoolean: boolean = new Boolean(1) 
//这样会报错 应为事实上 new Boolean() 返回的是一个 Boolean 对象 
//所以需要写成
let createdBoolean1: Boolean = new Boolean(1) 
//or
let createdBoolean2: boolean = Boolean(1) 
```

### **空值类型**

`JavaScript` 没有空值（Void）的概念，在 `TypeScript` 中，可以用 `void` 表示没有任何返回值的函数

```ts
function voidFn(): void {
    console.log('test void')
}
//void 类型的用法，主要是用在我们不希望调用者关心函数返回值的情况下，比如通常的异步回调函数
```

**void也可以定义undefined 和 null类型,但是这需要取消严格模式**

### **Null和undefined类型**

```ts
let u: undefined = undefined;//定义undefined
let n: null = null;//定义null
```

>void 和 undefined 和 null 最大的区别:undefined 和 null 是所有类型的子类型。也就是说 undefined 类型的变量，可以赋值给 string 类型的变量：

```ts
    //非严格模式
let u: undefined = undefined;//定义undefined
let n: null = null;//定义null
let v:void=undefined
let str='hale';
str=u;
str=n;
str=v; // TODO 不支持
```

### **Any 类型 和 unknown （顶级类型|任意类型）**

- 1.没有强制限定哪种类型，随时切换类型都可以 我们可以对 `any` 进行任何操作，不需要检查类型
- 2.声明变量的时候没有指定任意类型默认为`any`
- 3.弊端如果使用`any` 就失去了TS类型检测的作用
- 4.`TypeScript 3.0`中引入的 `unknown` 类型也被认为是 `top type` ，但它更安全。与 `any` 一样，所有类型都可以分配给`unknown`.

> `unknow`:  `unknow`类型比`any`更加严格当你要使用`any` 的时候可以尝试使用`unknow`,因为`unknow`只能赋值自身或者`any`，且`unknow`不能读到自身任何属性

### **接口和对象类型**

对于对象类型容易搞混的是`object`、`Object` 以及`{}`，那么他们有什么区别呢。

- `Object`
`Object`类型是所有`Object`类的实例的类型。 由以下两个接口来定义：
  - `Object` 接口定义了 `Object.prototype`原型对象上的属性；
  - `ObjectConstructor` 接口定义了 `Object` 类的属性， 如上面提到的 `Object.create()`。

    >这个类型是跟原型链有关的原型链顶层就是Object，所以值类型和引用类型最终都指向Object，他包含所有类型。

- `{}`
    看起来很别扭的一个东西 你可以把他理解成new Object 就和我们的上面Object基本一样 包含所有类型

    ```ts
    let a1: {} = {name:'hale'} //正确
    let a2: {} =  () => 123//正确
    let a3: {} = 123//正确
    ```

- `object`

    代表所有非值类型的类型，例如 数组 对象 函数等，常用于泛型约束

    ```ts
    let a1: {} = {name:1} //正确
    let a2: {} =  () => 123//正确
    let a3: {} = 123//正确
    ```

    在TS中我们定义对象的方式要用关键字`interface`（接口），我的理解是使用`interface`来定义一种约束，让数据的结构满足约束的格式

    ```Ts
    interface A {
        d:number 
    }
    interface Person {
        b:string,
        a:string,
        c?:number,//可选属性?,该属性可以不存在
        [propName: string]: any;//任意属性定义了任意属性，那么确定属性和可选属性的类型都必须是它的类型的子集
    }
    //如果重名，interface合并
    interface Person extends  {
        readonly id:string, //只读属性是不允许被赋值的只能读取
        cb:()=>void
    }
    const person:Person  = {
        a:"hale",
        b:'man',
        d:25
        cb:()=>{
            console.log(123)
        }
    }
    //定义一个函数类型
    interface Fn{
        (name:string):number[]
    }
    const fnHandle:Fn=function (name:string){
        return [12,13]
    }
    ```

### **数组类型**

```ts
// 类型数组
    let arr1:number[] = [1,2,3]// 数组中不能存在非number类型
    //如果是混合数组
    let arr2:any[]=[1,'2',true]
    //数组泛型
    let arr3:Array<number> = [1,2,3,4,5]
    //接口数组
    interface ObjArray {
    id:string,
    num:number
    }
    let arr4:ObjArray[] = [{id:'1',num:12}]
    //多维数组
    let arr5:number[][] = [[1,2], [3,4]];
    let arr6:Array<Array<number>> = [1,2,3,4,5]
```

### **元组 Tuple**

>元组就是数组的变种
 元组（Tuple）是固定数量的不同类型的元素的组合

 ```ts
 //元组与集合的不同之处在于，元组中的元素类型可以是不同的，而且数量固定
let arr:[number,string] = [1,'hale']
//设置只读
let arr2: readonly [number,boolean,string,undefined] = [1,true,'hale',undefined]
//设置可选
let a:[x:number,y?:boolean] = [1]
 ```

 应用场景，前端导出excel文件的时候，后端返回数据

 ```ts
 let excel: [string, string, string, number][] = [
   ['title', 'name', 'man', 1]
]
 ```

### **函数类型**

**基本函数类型**

```ts
//注意，参数不能多传，也不能少传 必须按照约定的类型来
const fn1 = (name: string, age:number): string => {
    return name + age
}
fn1('张三',18)
//如果要少传可以给参数加上可选属性？或者 加上默认值，两者不能作用到同一个参数
function fn2(name:string='hale',age?:number):string{
    return {
    name,
    age
    }
}
fn2('张三',13)
fn2('李四')
fn2()
//接口定义函数
//定义参数 num 和 num2  ：后面定义返回值的类型
interface Add {
    (num:  number, num2: number): number
}
const fn3: Add = (num: number, num2: number): number => {
    return num + num2
}
fn3(5, 5)
interface User{
    name: string;
    age: number;
}
function getUserInfo(user: User): User {
    return user
}
```

**扩展函数类型**

```ts
//函数this类型
interface User {
    user: number[],
    add: (this:User,num: number) => void;
}
//ts 可以定义this的类型，且必须的第一个参数定义this,在js中是无法使用的，
let Obj: User = {
    user: [1, 2, 3],
    add(this: User, num:number) {
        this.user.push(num)
    }
}
Obj.add(4)

/*
 * @description  函数重载
 * @1. 重载是方法名字相同，而参数不同，返回类型可以相同也可以不同。
 * @2. 如果参数类型不同，则参数类型应设置为 any。
 * @3 参数数量不同你可以将不同的参数设置为可选。
*/

let num: number[] = [1, 2, 3];
function findNum(add: number[]): number[];//如果是number类型数组.则是添加
function findNum(is: number): number;//如果是number字段.则是查找
function findNum(): number[];//如果是空，则返回全部
function findNum(args?: number | number[]) {
    if (Array.isArray(args)) {
        num.push(...args);
        return num
    } else if (typeof args == 'number') {
        return args
    } else {
        return num
    }
}
```
