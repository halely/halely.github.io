---
title: TypeScrip-学习（四）
date: 2023-04-26 16:37:00
cover: https://www.foodiesfeed.com/wp-content/uploads/2023/04/female-farmer-holding-sign-board.jpg
tags:
  - JavaScript
  - TypeScript
---

# 枚举

在`JavaScript`中是没有枚举的概念的TS帮定义了枚举这个类型,定义通过`enum`关键字

- `数字枚举`

    ```ts
    enum Types{
        Red,
        Green,
        BLue
    }
    //简单的定义默认为数字枚举，那么什么是数字枚举呢，即：枚举值是数字，如果你不设置默认值，那么每一个组员默认都是从0开始的自增数
    console.log(Types.Red);//0
    console.log(Types.Green);//1
    console.log(Types.BLue);//2
    ```

    **增长枚举**：细心的我们能发现数字枚举是会`自增`的，那么如果我们给上一个设置了一个默认值，下一个是自动增加

    ```ts
    enum Types{
        Red = 1,
        Green,
        BLue
    }
    console.log(Types.Red);//1
    console.log(Types.Green);//2
    console.log(Types.BLue);//3
    ```

- `字符串枚举`
   字符串枚举的概念很简单。 在一个字符串枚举里，每个成员都必须用字符串字面量，或另外一个字符串枚举成员进行初始化。

   ```ts
   enum Types{
    Red = 'red',
    Green = 'green',
    BLue = 'blue'
   }
   ```

   由于字符串枚举没有自增长的行为，字符串枚举可以很好的序列化。 换句话说，如果你正在调试并且必须要读一个数字枚举的运行时的值，这个值通常是很难读的 - 它并不能表达有用的信息，字符串枚举允许你提供一个运行时有意义的并且可读的值，独立于枚举成员的名字。
- `异构枚举`
    枚举可以混合字符串和数字成员

    ```ts
    enum Types{
        No = "No",
        Yes = 1,
    }
    ```

# 类型别名 type

`type` 关键字（可以给一个类型定义一个名字）多用于复合类型

```ts
//定义类型别名
type str = string
let s:str = "我是字符串"
console.log(s);
// 定义函数别名
type fn = () => string
let f: fn = () => "我是函数"
console.log(f);
type str = string | number
//定义联合类型别名
type strOrNmu = string | number
let text1: strOrNmu = 123
let text2: strOrNmu = '123'
console.log(text1,text2);
//定义值的别名
type value = boolean | 0 | '213'
let text3:value = true
//变量text3的值只能是上面value定义的值
```

`type` 和 `interface`是有区别的

1. `interface`是定义对象类型的，`type`是类型别名,他只是一个别名！！
2. `interface`可以继承,`type`只能通过 & 交叉类型合并
3. `type`可以定义 联合类型 和 可以使用一些操作符 `interface`不行
4. `interface` 遇到重名的会合并 type 不行,而且不能重复定义

**type高级用法**

```ts
//右边是否包含左边
type a1 = 1 extends number ? 1 : 0 //1
type a2 = 1 extends Number ? 1 : 0 //1
type a3= 1 extends Object ? 1 : 0 //1
type a4 = 1 extends any ? 1 : 0 //1
type a5 = 1 extends unknow ? 1 : 0 //1
type a6 = 1 extends never ? 1 : 0 //0
```

**TS类型层级图**
![GIF](/img/postImg/tsLevel.png)

# never类型

`TypeScript` 将使用 never 类型来表示不应该存在的状态(很抽象是不是)

```ts
// 返回never的函数必须存在无法达到的终点
// 因为必定抛出异常，所以 error 将不会有返回值
function error(message: string): never {
    throw new Error(message);
}
// 因为存在死循环，所以 loop 将不会有返回值
function loop(): never {
    while (true) {
    }
}
```

**never 与 void 的差异**

```ts
    //void类型只是没有返回值 但本身不会出错
    function Void():void {
        console.log();
    }
    //只会抛出异常没有返回值
    function Never():never {
    throw new Error('aaa')
    }
    //差异2   当我们鼠标移上去的时候会发现 只有void和number    never在联合类型中会被直接移除
    type A = void | number | never
```

>任何类型都不能赋值给`never` 类型

# symbol类型

`symbol`成为了一种新的原生类型，就像`number`和`string`一样。`symbol`类型的值是通过`Symbol`构造函数创建的。可以传递参做为唯一标识 只支持 `string` 和`number`类型的参数

Symbol的值是唯一的

```ts
const s1 = Symbol(1)
const s2 = Symbol(1)
// s1 === s2 =>false
```

>一般来说`Symbol`是创建唯一值，但是可以调用`for`来创建`Symbol`,`console.log(Symbol.for('key')==Symbol.for('key'))`为`true`，因为for是查找`Symbol`有没有注册对应的`key`值(**key为string类型**)，如果有就返回，如果没有就创建，故是相同的

**使用symbol定义的属性，是不能通过如下方式遍历拿到的**

```ts
const symbol1 = Symbol('hale')
const symbol2 = Symbol('ton')
const obj= {
   [symbol1]: 'hale',
   [symbol2]: 'ton',
   age: 27,
   sex: 'man'
}
// 1 for in 遍历
for (const key in obj) {
   console.log(key);//// 注意在console看key,是不是没有遍历到symbol1
}
// 2 Object.keys 遍历
console.log(Object.keys(obj))
// 3 getOwnPropertyNames
console.log(Object.getOwnPropertyNames(obj))
// 4 JSON.stringify 克隆是无法克隆symbol的数据的
console.log(JSON.stringify(obj))

/* 获取方式 */
// 1 拿到具体的symbol 属性,对象中有几个就会拿到几个
console.log(Object.getOwnPropertySymbols(obj))
// 2 es6 的 Reflect 拿到对象的所有属性
console.log(Reflect.ownKeys(obj))
```

**Symbol.iterator 迭代器 和 生成器 for of**

在es6中有`iterator`(迭代器)概念，遍历大部分类型迭代器 `arr` `nodeList` `函数的 arguments 对象` `set` `map` 等

```ts
var arr:number[] = [1,2,3,4];
let iterator = arr[Symbol.iterator]();
 
console.log(iterator.next());  //{ value: 1, done: false }
console.log(iterator.next());  //{ value: 2, done: false }
console.log(iterator.next());  //{ value: 3, done: false }
console.log(iterator.next());  //{ value: 4, done: false }
console.log(iterator.next());  //{ value: undefined, done: true }
```

`ES6` 规定，默认的 `Iterator` 接口部署在数据结构的`Symbol.iterator`属性，或者说，一个数据结构只要具有`Symbol.iterator`属性，就可以认为是**可遍历的**
象 **（Object）** 之所以没有默认部署 Iterator 接口，是因为对象的哪个属性先遍历，哪个属性后遍历是不确定的，需要开发者手动指定。本质上，遍历器是一种线性处理，对于任何非线性的数据结构，部署遍历器接口，就等于部署一种线性转换。不过，严格地说，对象部署遍历器接口并不是很必要，因为这时对象实际上被当作 `Map 结构使用，ES5` 没有 `Map` 结构，而 `ES6` 原生提供了。下面是es6对应的知识点:

1. **生成器**

    ```ts
    function* getApp() {
        yield 'hale'
        yield '24'
        yield Promise.resolve('你好')
        yield 'man'
    }
    let app = getApp()
    console.log(app.next())
    console.log(app.next())
    console.log(app.next())
    console.log(app.next())
    console.log(app.next())
    ```

2. **迭代器利用**

    ```ts
    let map: Map<any, any> = new Map();
    map.set([1, 2, 3], 'hale')
    let set = new Set([1, 2, 2, 1, 4, 3, 5, 4])
    let arr = [1, 2, 3, 1, 2, 3]
    let each = (value: any) => {
        let It: any = value[Symbol.iterator]()
        let next: any = { done: false }
        while (!next.done) {
            next = It.next();
            if (!next.done) console.log(next.value)
        }
    }
    each(map)
    each(arr)
    each(set)
    ```

3. **`for of` 就是迭代器的语法糖，就是上面内置的`each`方法**

   ```ts
    let map: Map<any, any> = new Map();
    map.set([1, 2, 3], 'hale')
    let set = new Set([1, 2, 2, 1, 4, 3, 5, 4])
    let arr = [1, 2, 3, 1, 2, 3]
    for(let value of set){
        console.log(value)
    }
   ```

4. **数组的解构**

    数组的解构或者`[...]`利用的就是迭代器，如果需要数组方法解构对象，可以自己封装

    ```ts
    let obj = {
        currentIndex: 0,
        maxIndex: 5,
        [Symbol.iterator]() {
            return {
                currentIndex: this.currentIndex,
                maxIndex: this.maxIndex,
                next() {
                    if (this.maxIndex == this.currentIndex) {
                        return {
                            value: undefined,
                            done: true,
                        }
                    } else {
                        return {
                            value: this.currentIndex++,
                            done: false,
                        }
                    }
                }
            }
        }
    }
    let ad=[...obj]
    console.log(ad) //[ 0, 1, 2, 3, 4 ]
    ```

    >注意这是数组的解构原理，对应的解构有自己的原理
