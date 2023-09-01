---
title: TypeScrip-学习（二）
date: 2023-04-24 15:45:51
cover: https://www.foodiesfeed.com/wp-content/uploads/2022/11/tuna-poke-with-fresh-vegetables.jpg
tags:
  - JavaScript
  - TypeScript
---

# 前言

继续上级第一章的学习

# 类型断言 | 联合类型 | 交叉类型

## 联合类型

```ts
//例如我们的手机号通常是13XXXXXXX 为数字类型 这时候产品说需要支持座机
//所以我们就可以使用联合类型支持座机字符串
let myPhone: number | string  = '010-820'
//这样写是会报错的应为我们的联合类型只有数字和字符串并没有布尔值
let myPhone: number | string  = true
//函数使用联合类型
const fn = (something:number | boolean):boolean => {
     return !!something
}
```

## 交叉类型

多种类型的集合，联合对象将具有所联合类型的所有成员

```ts
interface People {
  age: number,
  height： number
}
interface Man{
  sex: string
}
const haleFn = (man: People & Man) => {
  console.log(man.age)
  console.log(man.height)
  console.log(man.sex)
}
haleFn({age: 18,height: 180,sex: 'male'});
```

## 类型断言

> 语法法：[值] `as` [类型]或`<类型>`[值]: `value as string`or`<string>value`

```ts
interface A {
       run: string
}
interface B {
       build: string
}
const fn = (type: A | B): string => {
       return (type as A).run
       //return (<A>type).run
}
//可以使用类型断言来推断他传入的是A接口的值
```

需要注意的是，类型断言只能够 **「欺骗」** TypeScript 编译器，无法避免运行时的错误，反而滥用类型断言可能会导致运行时错误：

**加餐**

- `使用any临时断言`

    ```ts
    (window as any).abc = 123
    //可以使用any临时断言在 any 类型的变量上，访问任何属性都是允许的。
    ```

- `as const`
   是对字面值的断言，与const直接定义常量是有区别的
   如果是普通类型跟直接const 声明是一样的

   ```ts
    const names = '小满'
    names = 'aa' //无法修改
    let names2 = '小满' as const
    names2 = 'aa' //无法修改

    // 数组
    let a1 = [10, 20] as const;
    const a2 = [10, 20];
    a1.unshift(30); // 错误，此时已经断言字面量为[10, 20],数据无法做任何修改
    a2.unshift(30); // 通过，没有修改指针
  ```

# 内置对象

## ECMAScript 的内置对象

`Boolean`、`Number`、`String`、`RegExp`、`Date`、`Error`

```ts
let b: Boolean = new Boolean(1)
let n: Number = new Number(true)
let s: String = new String('')
let d: Date = new Date()
let r: RegExp =/\[bc\]at/i; //new RegExp("\\[bc\\]at", "i")
let e: Error = new Error("error!")
```

**加餐**

`Promise`

```ts
function promise():Promise<number>{
   return new Promise<number>((resolve,reject)=>{
       resolve(1)
   })
}
promise().then(res=>{
    console.log(res)
})

```

## DOM 和 BOM 的内置对象

`Document`、`HTMLElement`、`Event`、`NodeList` 等

```ts
let body: HTMLElement = document.body;
let allDiv: NodeList = document.querySelectorAll('div');
//读取div 这种需要类型断言 或者加个判断应为读不到返回null
let div:HTMLElement = document.querySelector('div') as HTMLDivElement
document.addEventListener('click', function (e: MouseEvent) {});
//dom元素的映射表
interface HTMLElementTagNameMap {
    "a": HTMLAnchorElement;
    "abbr": HTMLElement;
    "address": HTMLElement;
    "applet": HTMLAppletElement;
    "area": HTMLAreaElement;
    "article": HTMLElement;
    "aside": HTMLElement;
    "audio": HTMLAudioElement;
    "b": HTMLElement;
    "base": HTMLBaseElement;
    "bdi": HTMLElement;
    "bdo": HTMLElement;
    "blockquote": HTMLQuoteElement;
    "body": HTMLBodyElement;
    "br": HTMLBRElement;
    "button": HTMLButtonElement;
    "canvas": HTMLCanvasElement;
    "caption": HTMLTableCaptionElement;
    "cite": HTMLElement;
    "code": HTMLElement;
    "col": HTMLTableColElement;
    "colgroup": HTMLTableColElement;
    "data": HTMLDataElement;
    "datalist": HTMLDataListElement;
    "dd": HTMLElement;
    "del": HTMLModElement;
    "details": HTMLDetailsElement;
    "dfn": HTMLElement;
    "dialog": HTMLDialogElement;
    "dir": HTMLDirectoryElement;
    "div": HTMLDivElement;
    "dl": HTMLDListElement;
    "dt": HTMLElement;
    "em": HTMLElement;
    "embed": HTMLEmbedElement;
    "fieldset": HTMLFieldSetElement;
    "figcaption": HTMLElement;
    "figure": HTMLElement;
    "font": HTMLFontElement;
    "footer": HTMLElement;
    "form": HTMLFormElement;
    "frame": HTMLFrameElement;
    "frameset": HTMLFrameSetElement;
    "h1": HTMLHeadingElement;
    "h2": HTMLHeadingElement;
    "h3": HTMLHeadingElement;
    "h4": HTMLHeadingElement;
    "h5": HTMLHeadingElement;
    "h6": HTMLHeadingElement;
    "head": HTMLHeadElement;
    "header": HTMLElement;
    "hgroup": HTMLElement;
    "hr": HTMLHRElement;
    "html": HTMLHtmlElement;
    "i": HTMLElement;
    "iframe": HTMLIFrameElement;
    "img": HTMLImageElement;
    "input": HTMLInputElement;
    "ins": HTMLModElement;
    "kbd": HTMLElement;
    "label": HTMLLabelElement;
    "legend": HTMLLegendElement;
    "li": HTMLLIElement;
    "link": HTMLLinkElement;
    "main": HTMLElement;
    "map": HTMLMapElement;
    "mark": HTMLElement;
    "marquee": HTMLMarqueeElement;
    "menu": HTMLMenuElement;
    "meta": HTMLMetaElement;
    "meter": HTMLMeterElement;
    "nav": HTMLElement;
    "noscript": HTMLElement;
    "object": HTMLObjectElement;
    "ol": HTMLOListElement;
    "optgroup": HTMLOptGroupElement;
    "option": HTMLOptionElement;
    "output": HTMLOutputElement;
    "p": HTMLParagraphElement;
    "param": HTMLParamElement;
    "picture": HTMLPictureElement;
    "pre": HTMLPreElement;
    "progress": HTMLProgressElement;
    "q": HTMLQuoteElement;
    "rp": HTMLElement;
    "rt": HTMLElement;
    "ruby": HTMLElement;
    "s": HTMLElement;
    "samp": HTMLElement;
    "script": HTMLScriptElement;
    "section": HTMLElement;
    "select": HTMLSelectElement;
    "slot": HTMLSlotElement;
    "small": HTMLElement;
    "source": HTMLSourceElement;
    "span": HTMLSpanElement;
    "strong": HTMLElement;
    "style": HTMLStyleElement;
    "sub": HTMLElement;
    "summary": HTMLElement;
    "sup": HTMLElement;
    "table": HTMLTableElement;
    "tbody": HTMLTableSectionElement;
    "td": HTMLTableDataCellElement;
    "template": HTMLTemplateElement;
    "textarea": HTMLTextAreaElement;
    "tfoot": HTMLTableSectionElement;
    "th": HTMLTableHeaderCellElement;
    "thead": HTMLTableSectionElement;
    "time": HTMLTimeElement;
    "title": HTMLTitleElement;
    "tr": HTMLTableRowElement;
    "track": HTMLTrackElement;
    "u": HTMLElement;
    "ul": HTMLUListElement;
    "var": HTMLElement;
    "video": HTMLVideoElement;
    "wbr": HTMLElement;
}
```

**加餐**

`代码雨`

```ts
let canvas = document.querySelector('#canvas') as HTMLCanvasElement
let ctx = canvas.getContext('2d') as CanvasRenderingContext2D
canvas.height = screen.availHeight; //可视区域的高度
canvas.width = screen.availWidth; //可视区域的宽度
let randomStr: string[] = 'XMZSHALEYZJXMFCLDD0011HS'.split('')
//创建宽度数量的默认填充数组获取宽度例如1920 / 10 192
let byteArr = Array(Math.ceil(canvas.width / 10)).fill(0)

const rain = () => {
    ctx.fillStyle = 'rgba(0,0,0,0.05)'//填充背景颜色
    ctx.fillRect(0, 0, canvas.width, canvas.height,)
    ctx.fillStyle = "#0f0"; //文字颜色
    byteArr.forEach((item, index) => {
        ctx.fillText(randomStr[Math.floor(Math.random() * randomStr.length)], index * 10, item + 10);
        byteArr[index] = item > canvas.height || item > (10000 * Math.random()) ? 0 : item + 10;
    })
}
setInterval(rain, 40)
```
