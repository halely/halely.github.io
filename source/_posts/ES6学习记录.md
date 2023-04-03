---
title: ES6学习记录
date: 2022-08-09 17:28:07
cover: 'https://w.wallhaven.cc/full/zy/wallhaven-zygeko.jpg'
tags:
 - JavaScript
---

# 前言

为了记录自己记得不熟或者使用的不多但是很实用的ES6的基础逻辑，纯记录和自我学习

## 字符串扩充

### 标签模板

在ES6中引入的`模板字符串`的概念，很好用，只需要使用灵活的使用 **``** 和 ${} 就能很好的实现之前很繁琐的功能，
但是在使用过程中，忽然发现了`标签模板`-就是指模板字符串跟在函数名后面，那么这个函数就会调用来处理这个模板字符串。

```js
alert`hello`
// 等同于
alert(['hello'])
```

这就成了函数后面跟着的模板字符串成了参数，函数就会处理这些参数，但是如果有变量的话就不一样了如下：

```js
let a = 5;
let b = 10;
//tag 是一个函数
tag`Hello ${ a + b } world ${ a * b }`;
// 等同于
tag(['Hello ', ' world ', ''], 15, 50);
```

字符串成了被解析的数组，而变量成了后面的参数，标签模板的一个重要功能就是过滤HTML字符串，防止恶意输入，还有就是多语言转换（国际化处理），还有就是JSX将DOM字符串渲染成React对象，这就是JSX的实现原理

```js
i18n`Welcome to ${siteName}, you are visitor number ${visitorNumber}!`
// "欢迎访问xxx，您是第xxxx位访问者！"

jsx`
<div>
    <input
    ref='input'
    onChange='${this.handleChange}'
    defaultValue='${this.state.value}' />
    ${this.state.value}
</div>
`
//模板处理函数的第一个参数（模板字符串数组），还有一个raw属性。raw保存的是转义后的原字符串
console.log`123`
// ["123", raw: Array[1]]
```

>标签模板也有限制，就是在使用注册证如果内嵌了其他预研比如说`LaTEX`,这其实是完全合法的，但是`JavaScript`引擎会报错，原因就是字符串转义问题，虽然在ES2018放松了限制，但是不合法的转义还是会返回`undefined`，且从raw属性上面可以得到原始字符串。所以在使用中,尽量不要使用其他语言嵌入。

### 字符串新增方法

1. **String.fromCodePoint()**

    ES5 提供`String.fromCharCode()`方法，用于从`Unicode`码点返回对应字符，但是这个方法不能识别码点大于`0xFFFF`的字符。

    ```js
    String.fromCharCode(0x20BB7)
    // "ஷ"
    ```

    如果大于`0xFFFF`，大于的就会被舍弃，所以ES6提供了`String.fromCodePoint()`方法，可以识别大于0xFFFF的字符，弥补了之前的不足，在作用上，正好与下面的`codePointAt()`方法相反。

    ```js
    String.fromCodePoint(0x20BB7)
    // "𠮷"
    String.fromCodePoint(0x78, 0x1f680, 0x79) === 'x\uD83D\uDE80y'
    // true
    ```
    上面代码中，如果`String.fromCodePoint`方法有多个参数，则它们会被合并成一个字符串返回。
    >注意，fromCodePoint方法定义在String对象上，而codePointAt方法定义在字符串的实例对象上。
