---
title: JavaScript 内存空间详解
date: 2023-05-16 10:43:51
tags:
 - JavaScript
categories: 
  - 前端学习
---

# 引入

我们先看下面简单的代码

```js
function foo() {
    foo();
}
foo();
```

这样就会报错
![](http://resource.muyiy.cn/image/2019-07-24-060211.png)

某些情况下，调用堆栈中函数调用的数量超出了调用堆栈的实际大小，浏览器会抛出一个错误终止运行。上面的就是无限循环调用导致。


