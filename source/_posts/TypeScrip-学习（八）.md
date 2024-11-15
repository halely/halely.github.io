---
title: TypeScrip-学习（八）装饰器Decorator
date: 2023-04-28 16:07:38
cover: https://www.foodiesfeed.com/wp-content/uploads/2023/05/bowl-of-raspberries.jpg
tags:
  - JavaScript
  - TypeScript
categories: 
  - 前端学习
---
# 装饰器 Decorator

它是一项**实验性特性**，在未来的版本中可能会发生改变
它们不仅增加了代码的可读性，清晰地表达了意图，而且提供一种方便的手段，增加或修改类的功能

启用实验性的装饰器特性，必须在命令行或`tsconfig.json`里启用编译器选项

```json
{
//...
"experimentalDecorators": true,    
"emitDecoratorMetadata": true, 
//...
}

 ```

**装饰器**是一种特殊类型的声明，它能够被附加到**类声明，方法， 访问符，属性或参数上**。
**学习点**

1. 类装饰器：`ClassDecorator`
2. 方法装饰器：`MethodDecorator`
3. 属性装饰器：`PropertyDecorator`
4. 参数装饰器：`ParameterDecorator`
5. 装饰器工厂

## 类装饰器

定义一个类装饰器函数 他会把`Http`的构造函数传入你的`Base`函数当做第一个参数,注意是**构造函数不是原型**
> 优势：当原先的类有很多业务代码，无法全部理清，但是又需要添加新的方法或属性。因为迭代器返回的是构造函数，可以在回调方法中加入你所需要的业务，这是不影响原先业务

```ts
const Base:ClassDecorator=(target)=>{
  console.log(target)
  target.prototype.Name='hale';
  target.prototype.fn=()=>{
   console.log('本人单身')
  }
}
@Base
class Http {
}
const http=new Http() as any;
http.fn()
console.log(http.Name)
```

## 方法装饰器

```ts
import axios from 'axios';
const Get = (url: string) => {
  const fn: MethodDecorator = (target, key, descriptor: PropertyDescriptor) => {
    console.log(target, key, descriptor)
    /*
    * @description 它的返回参数有三个：
    * @target 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
    * @key 成员的名字。
    * @descriptor 成员的属性描述符。
    */
    axios.get(url).then(res => {
      descriptor.value(res.data)
    })
  }
  return fn
}

class Http {
  @Get('https://api.apiopen.top/api/getUserInfoForId/1')
  getList(data: any) {
    console.log(data)

  }
}

const http = new Http()
```

## 装饰器工厂

其实也就是一个高阶函数 外层的函数接受值,里层的函数最终接受类的构造函数

```ts
const Base = (params: string) => {
  const fn: ClassDecorator = (target:Function) => {
    console.log(target)
    console.log(params)
    target.prototype.Name = 'hale';
    target.prototype.getFn = () => {
      console.log('本人单身')
    }
  }
  return fn
}

@Base('张三')
class Http {

}
const http=new Http() as any;
http.fn()
console.log(http.Name)
```

## 参数装饰器

```ts

import axios from 'axios';
import 'reflect-metadata'
const Get = (url: string) => {
  const fn: MethodDecorator = (target, key, descriptor: PropertyDescriptor) => {
    const metadata=Reflect.getMetadata('key',target);//获取元数据
    console.log(metadata)
    axios.get(url).then(res => {
      descriptor.value(metadata?res[metadata]:res)
    })
  }
  return fn
}
const Result = () => {
  const fn: ParameterDecorator = (target, propertyKey, parameterIndex) => {
    /*
    * @description 它的返回参数有三个：
    * @target 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
    * @key 成员的名字。
    * @parameterIndex 参数对应的索引
    */
    //设置元数据
    Reflect.defineMetadata('key', 'data', target)
  }
  return fn
}
class Http {
  @Get('https://api.apiopen.top/api/getUserInfoForId/1')
  getList( @Result() data: any) {
    console.log(data)

  }
}

const http = new Http()
```

> 参数装饰器的优先级大于方法装饰器

## 属性装饰器

```ts
const Name: PropertyDecorator = (target, propertyKey)=>{
 /*
    * @description 它的返回参数有两个：
    * @target 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
    * @propertyKey 成员的名字。
    */
  console.log(target, propertyKey)
}
class Http {
  @Name
  name: string
  constructor() {
    this.name = '张三'
  }
}
```
