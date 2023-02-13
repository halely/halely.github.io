---
title: JavaScript实用方法
cover: 'https://w.wallhaven.cc/full/l3/wallhaven-l3xk6q.jpg'
date: 2022-12-07 15:19:23
tags:
  - JavaScript
---

# 前言

在我日常开发过程总，会编写诸如排序、搜索、查找唯一值、传递参数、交换值等等功能，一些实用和便捷的方法做一个记录

## 1.找出数组总和、最小值和最大值

利用`reduce`方法实现

```js
const array  = [5,4,7,8,9,2];
array.reduce((a,b) => a+b);//和 35
array.reduce((a,b) => a>b?a:b);//最大 9
array.reduce((a,b) => a<b?a:b);//最小 2
```

## 2.从数组中过滤出虚假值

Falsy值喜欢0，undefined，null，false，""，''可以很容易地通过以下方法省略

```javascript
const array = [3, 0, 6, 7, '', false];
array.filter(Boolean);
// 输出(3) [3, 6, 7]
```

## 3.删除重复值

```js
const array  = [5,4,7,8,9,2,7,5];
const nonUnique =[... new Set(array)]
// or
array.filter((item,index,arr)=>arr.indexOf(item)=== index)
```

## 3.将十进制转换为二进制或十六进制

在解决问题的同时，我们可以使用一些内置的方法，例如`.toPrecision()`或`.toFixed()`来实现许多帮助功能。

```js
const num = 10;
num.toString(2);
// 输出: "1010"
num.toString(16);
// 输出: "a"
num.toString(8);
// 输出: "12"
```

## 4.对象检查属性是否存在

```js
const person = { name: '前端小智', salary: 1000 };
console.log('salary' in person); // true
console.log('age' in person); // false
console.log('toString' in person); // true  -- 因为toString是对象的原型方法是纯在的  同理的还有Reflect.has({x: 0}, "toString");  也是true

//针对这种方法可以使用 Object.prototype.hasOwnPerporty()方法，来判断自身
const obj = {x: 1}; 
console.log(obj.hasOwnProperty('toString')); // false
// 这种方法不能判断Object.create(null) 创建的对象 和对象设有 hasOwnProperty属性方法的情况
//原因是这对js来说不是敏感词，可以被本身替换。如果使用的话就需要从对象原型找
Object.prototype.hasOwnProperty.call(obj,'p');//true
//当着这样过于麻烦所以es2022提出了Object.hasOwn(),所以当然这是有兼容性的，仁者见仁
//如果指定的对象具有作为其自身属性的指定属性，则hasOwn()方法返回true;如果该属性是继承的或不存在，则该方法返回false。
const object1 = {prop: 'exists'};
console.log(Object.hasOwn(object1, 'prop')); // true
console.log(Object.hasOwn(object1, 'toString'));// false

```

## 5.扁平化数组

在原型 Array 上有一个方法 `flat`，可以从一个数组的数组中制作一个单一的数组。
```js
const myArray = [{ id: 1 }, [{ id: 2 }], [{ id: 3 }]];

const flattedArray = myArray.flat(); 
//[ { id: 1 }, { id: 2 }, { id: 3 } ]
//指定一个嵌套的数组结构应该被扁平化的深度
const arr = [0, 1, 2, [[[3, 4]]]];
console.log(arr.flat(2)); // returns [0, 1, 2, [3,4]]
```

## 6.Object.entries
大多数开发人员使用 `Object.keys` 方法来迭代对象,但是只返回键，不返回值，这时候就需要`Object.entries`.
```js
const person = {
  name: '前端小智',
  age: 20
};

Object.keys(person); // ['name', 'age']
Object.entries(data); // [['name', '前端小智'], ['age', 20]]
```
## 7.快捷替换replaceAll 方法

在 JS 中，要将所有出现的字符串替换为另一个字符串，我们需要使用如下所示的正则表达式：
```js
const str = 'Red-Green-Blue';
// 只规制第一次出现的
str.replace('-', ' '); // Red Green-Blue
// 使用 RegEx 替换所有匹配项
str.replace(/\-/g, ' '); // Red Green Blue。
```
## 7.数字分隔符
可以使用下划线作为数字分隔符，这样可以方便地计算数字中0的个数。

```js
// 难以阅读
const billion = 1000000000;
// 易于阅读
const readableBillion = 1000_000_000;
console.log(readableBillion) //1000000000
//下划线分隔符也可以用于BigInt数字，如下例所示
const trillion = 1000_000_000_000n;
console.log(trillion); // 1000000000000
```

## 8.document.designMode 
与前端的JavaScript有关，设计模式让你可以编辑页面上的任何内容。只要打开浏览器控制台，输入以下内容即可。
```js
document.designMode = 'on';//怎么说呢，就是页面直接编辑
```