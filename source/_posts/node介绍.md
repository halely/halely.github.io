---
title: node介绍
date: 2023-10-07 11:01:34
cover:
tags:
 - node
 - JavaScript
---

# 前言

1. [nodejs](https://nodejs.org/zh-cn) 并不是`JavaScript`应用，也不是编程语言，因为编程语言使用的`JavaScript`,`nodejs` 是 `JavaScript`的运行时环境。

  ![](/img/postImg/node.png)

2. `Nodejs`是构建在V8引擎之上的，V8引擎是由C/C++编写的，因此我们的JavaSCript代码需要由C/C++转化后再执行。

3. `NodeJs`使用异步 I/O 和[事件驱动](https://so.csdn.net/so/search?q=%E4%BA%8B%E4%BB%B6%E9%A9%B1%E5%8A%A8)的设计理念，可以高效地处理大量并发请求，提供了非阻塞式 I/O 接口和事件循环机制，使得开发人员可以编写高性能、可扩展的应用程序,异步I/O最终都是由libuv 事件循环库去实现的。

4. `NodeJs`使用npm 作为包管理工具类似于python的pip，或者是java的Maven，目前npm拥有上百万个模块。
<https://www.npmjs.com/>

5. `nodejs`适合干一些IO密集型应用，不适合CPU密集型应用，nodejsIO依靠libuv有很强的处理能力，而CPU因为nodejs单线程原因，容易造成CPU占用率高，如果非要做CPU密集型应用，可以使用C++插件编写 或者nodejs提供的`cluster`。(CPU密集型指的是图像的处理 或者音频处理需要大量数据结构 + 算法)


# nodeJs 大致架构图

