---
title: TypeScrip-学习（五）泛型
date: 2023-04-27 11:20:00
cover: https://www.foodiesfeed.com/wp-content/uploads/2023/03/colorful-gummy-bears-in-boxes.jpg
tags:
  - JavaScript
  - TypeScript
---

# 泛型

**泛型**在TypeScript 是很重要的东西,个人理解类似于类型入参

`函数泛型`

```ts
//普通类型函数
function num (a:number,b:number) : Array<number> {
    return [a ,b];
}
num(1,2)
function str (a:string,b:string) : Array<string> {
    return [a ,b];
}
str('独孤','求败')

// 函数优化
function Add<T>(a: T, b: T): Array<T>  {
    return [a,b]
}
 
Add<number>(1,2)
Add<string>('独孤','求败')
//也可以使用不同的泛型参数名，只要在数量上和使用方式上能对应上就可以
function Sub<T,U>(a:T,b:U):Array<T|U> {
    const params:Array<T|U> = [a,b]
    return params
}
Sub<Boolean,number>(false,1)
```

`泛型接口`
声明接口的时候 在名字后面加一个<参数>

```ts
interface Data<T> {
   str: T
}
interface MyInter<T> {
   (arg: T): T
}
//定义接口对象
let data: Data<string>={
    str:'张三'
}
function Inter<T>(num: T): T {
    return num
}
let result: MyInter<number> = Inter
result(121)
```

`对象字面量泛型`

```ts
let foo: { <T>(arg: T): T }
foo = function <T>(arg:T):T {
   return arg
}
foo(123)
```

`泛型约束`
我们期望在一个泛型的变量上面，获取其`length`参数，但是，有的数据类型是没有`length`属性的

```ts
function getLegnth<T>(arg:T) {
  return arg.length;
  //这会报错，可能不存在length
}
```

这时候我们就可以使用**泛型约束**

```ts
interface Len {
   length:number
}
 
function getLength<T extends Len>(arg:T) {
  return arg.length
}
 
getLength<string>('123')
```

`keyof 约束对象`

其中使用了TS泛型和泛型约束。首先定义了T类型并使用`extends`关键字继承object类型的子类型，然后使用`keyof`操作符获取`T`类型的所有键，它的返回类型是**联合类型**，最后利用`extends`关键字约束 K类型必须为`keyof` `T`联合类型的子类型

```ts
let objData = {
    key: '123456789',
    num: 12,
    isMan: true
}

function PropFn<T,K extends keyof T>(obj:T,key:K):T[K]{
   return obj[key]
}
let data=PropFn(objData,'isMan');//boolean
let num=PropFn(objData,'num');//number
let newStr=PropFn(objData,'bbt')//报错
```

`泛型类`
声明方法跟函数类似名称后面定义<类型>

```ts
class Sub<T>{
   attr: T[] = [];
   add (a:T):T[] {
      return [a]
   }
}
 
let s = new Sub<number>()
s.attr = [1,2,3]
s.add(123)
 
let str = new Sub<string>()
str.attr = ['1','2','3']
str.add('123')
```

高阶方式

```ts
interface Data {
    name: string,
    age: number,
    isMan: boolean
}

type Options<T extends Object> = {
    [Key in keyof T]?: T[Key]
}

type B=Options<Data>
```

例子:封装一个简单的`axios`

```ts
let axios = {
    get<T>(url: string): Promise<T> {
        return new Promise((resolve, rejects) => {
            //创建对象
            let xhr = new XMLHttpRequest();
            //连接接口
            xhr.open('GET', url);
            // 只要 readyState 属性发生变化，就会调用相应的
            xhr.onreadystatechange = () => {
                /*
                 * @description readyState 属性返回一个 XMLHttpRequest 代理当前所处的状态。一个 XHR 代理总是处于下列状态中的一
                 * @ 0 UNSENT 代理被创建，但尚未调用 open() 方法。
                 * @ 1  OPENED open() 方法已经被调用。
                 * @ 2  HEADERS_RECEIVED send() 方法已经被调用，并且头部和状态已经可获得。
                 * @ 3  LOADING 下载中；responseText 属性已经包含部分数据。
                 * @ 4  DONE 下载操作已完成。
                */
                if (xhr.readyState === 4 && xhr.status == 200) {
                    resolve(JSON.parse(xhr.responseText));
                }
            }
            xhr.send(null)
        })
    }
}
interface ResultData{
    data:string,
    num:number
}
axios.get<ResultData>('../data/json').then(res=>{
    res.num
})
```
