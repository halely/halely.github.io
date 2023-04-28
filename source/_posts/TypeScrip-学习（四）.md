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
