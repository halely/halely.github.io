---
title: 了解Web Worker
date: 2023-02-10 14:21:38
cover: 'https://w.wallhaven.cc/full/kx/wallhaven-kx36mq.png'
copyright_author: BEFE团队
copyright_author_href: https://juejin.cn/user/976828486917245
copyright_url: https://juejin.cn/post/7139718200177983524#comment
copyright_info: 本文章来自稀土掘金
tags:
  - JavaScript
---

# 前言

先来聊聊单线程的Javascript

众所周知，js最初设计是运行在浏览器中的，为了防止多个线程同时操作DOM，带来渲染冲突问题，所以js执行器被设计成单线程。但随着前端技术的发展，js能力远不止如此，当我们遇到需要大量计算的场景时（比如图像处理、视频解码等），js线程往往会被长时间阻塞，甚至造成页面卡顿，影响用户体验。为了解决单线程带来的这一弊端，Web Worker 应运而生。

# Web Worker

##  `Web Worker` 是什么

`Web Worker` 是 HTML5 标准的一部分，这一规范定义了一套 API，允许我们在 js 主线程之外开辟新的 Worker 线程，并将一段 js 脚本运行其中，它赋予了开发者利用 js 操作多线程的能力。
因为是独立的线程，Worker 线程与 js 主线程能够同时运行，互不阻塞。所以，在我们有大量运算任务时，可以把运算任务交给 Worker 线程去处理，当 Worker 线程计算完成，再把结果返回给 js 主线程。这样，js 主线程只用专注处理业务逻辑，不用耗费过多时间去处理大量复杂计算，从而减少了阻塞时间，也提高了运行效率，页面流畅度和用户体验自然而然也提高了。

## `Web Worker` 能干些什么

虽然 `Worker` 线程是在浏览器环境中被唤起，但是它与当前页面窗口运行在不同的全局上下文中，我们常用的顶层对象 `window`，以及 `parent` 对象在 Worker 线程上下文中是不可用的。另外，在 Worker 线程上下文中，操作 DOM 的行为也是不可行的，document对象也不存在。但是，`location`和`navigator`对象可以以可读方式访问。除此之外，绝大多数 Window 对象上的方法和属性，都被共享到 Worker 上下文全局对象 `WorkerGlobalScope` 中。同样，Worker 线程上下文也存在一个顶级对象 `self`。

#  Web Worker 使用

##  创建 `worker`

创建 `worker` 只需要通过 `new` 调用 `Worker()` 构造函数即可，它接收两个参数

```js
const worker = new Worker(path, options);
```

参数     | 说明  |
-------- | ----- |
path|有效的js脚本的地址，必须遵守同源策略。无效的js地址或者违反同源策略，会抛`SECURITY_ERR` 类型错误 |
options.type | 可选，用以指定 worker 类型。该值可以是 `classic` 或 `module。` 如未指定，将使用默认值`classic`   |
options.credentials| 可选，用以指定 worker 凭证。该值可以是 `omit`, `same-origin`，或`include`。如果未指定，或者 type 是 classic，将使用默认值 omit (不要求凭证) |
options.name | 可选，在 `DedicatedWorkerGlobalScope` 的情况下，用来表示 `worker` 的 scope 的一个 `DOMString` 值，主要用于调试目的。 |

##  js 主线程与 worker 线程数据传递

主线程与 worker 线程都是通过 `postMessage` 方法来发送消息，以及监听 `message` 事件来接收消息。如下所示

```js
// main.js（主线程）

const myWorker = new Worker('/worker.js'); // 创建worker

myWorker.addEventListener('message', e => { // 接收消息
    console.log(e.data); // Greeting from Worker.js，worker线程发送的消息
});

// 这种写法也可以
// myWorker.onmessage = e => { // 接收消息
//    console.log(e.data);
// };

myWorker.postMessage('Greeting from Main.js'); // 向 worker 线程发送消息，对应 worker 线程中的 e.data
// worker.js（worker线程）
self.addEventListener('message', e => { // 接收到消息
    console.log(e.data); // Greeting from Main.js，主线程发送的消息
    self.postMessage('Greeting from Worker.js'); // 向主线程发送消息
});

```

好了，一个简单 worker 线程就创建成功了。

好了，一个简单 worker 线程就创建成功了。
postMessage() 方法接收的参数可以是字符串、对象、数组等。具体我们在#3.7讨论。
主线程与 worker 线程之间的数据传递是传值而不是传地址。所以你会发现，即使你传递的是一个Object，并且被直接传递回来，接收到的也不是原来的那个值了。

```js
// main.js（主线程）
const myWorker = new Worker('/worker.js');

const obj = {name: '小明'};
myWorker.addEventListener('message', e => { 
    console.log(e.data === obj); // false
});
myWorker.postMessage(obj);
```

```js
// worker.js（worker线程）
self.addEventListener('message', e => {
    self.postMessage(e.data); // 将接收到的数据直接返回
});
```

##  监听错误信息

web worker 提供两个事件监听错误，error 和 messageerror。这两个事件的区别是:

事件|描述
-|-
`error`|当worker内部出现错误时触发
`messageerror`|当 `message` 事件接收到无法被反序列化的参数时触发

监听方式跟接收消息一致：

```js
// main.js（主线程）
const myWorker = new Worker('/worker.js'); // 创建worker

myWorker.addEventListener('error', err => {
    console.log(err.message);
});
myWorker.addEventListener('messageerror', err => {
    console.log(err.message)
});

```

```js
// worker.js（worker线程）
self.addEventListener('error', err => {
    console.log(err.message);
});
self.addEventListener('messageerror', err => {
    console.log(err.message);
});
```

##  关闭 worker 线程

worker 线程的关闭在主线程和 worker 线程都能进行操作，但对 worker 线程的影响略有不同。

```js
// main.js（主线程）
const myWorker = new Worker('/worker.js'); // 创建worker
myWorker.terminate(); // 关闭worker
```

```js
// worker.js（worker线程）
self.close(); // 直接执行close方法就ok了
```

无论是在主线程关闭 worker，还是在 worker 线程内部关闭 worker，worker 线程当前的 Event Loop 中的任务会继续执行。至于 worker 线程下一个 Event Loop 中的任务，则会被直接忽略，不会继续执行。
区别是，在主线程手动关闭 worker，主线程与 worker 线程之间的连接都会被立刻停止，即使 worker 线程当前的 Event Loop 中仍有待执行的任务继续调用 `postMessage()` 方法，但主线程不会再接收到消息。
在 worker 线程内部关闭 worker，不会直接断开与主线程的连接，而是等 worker 线程当前的 Event Loop 所有任务执行完，再关闭。也就是说，在当前 Event Loop 中继续调用 `postMessage()` 方法，主线程还是能通过监听`message`事件收到消息的。
如下两个例子可以很好说明这一点：

如下两个例子可以很好说明这一点：
在主线程关闭 worker

worker 线程在接受到消息后，立即向主线程回复一条消息。然后利用计时器添加一个宏任务；利用 Promise 添加一个微任务；执行一个 for 循环。目的都是向主线程回复一条消息。
主线程在接收到消息后立即关闭 worker 线程。

大家可以思考一下，主线程会接收到哪些消息呢，控制台会打印出哪些信息呢？

```js
// main.js（主线程）
const myWorker = new Worker('/worker.js'); // 创建 worker

myWorker.addEventListener('message', e => {
    console.log(e.data);
    myWorker.terminate(); // 关闭 worker
});

myWorker.postMessage('Greeting from Main.js');

```

```js
// worker.js（worker线程）

self.addEventListener('message', e => {

    postMessage('Greeting from Worker');
    
    setTimeout(() => {
        console.log('setTimeout run');
        postMessage('Greeting from SetTimeout');
    });
    
    Promise.resolve().then(() => {
        console.log('Promise run');
        postMessage('Greeting from Promise');
    })
    
    for (let i = 0; i < 1001; i++) {
        if (i === 1000) {
            console.log('Loop run');
            postMessage('Greeting from Loop');
        }
    }
    
});

```

运行结果如下

```js
//--- loop run
//--- Promise run 
//--- Greeting from Worker
```

- 主线程只会接收到 worker 线程第一次通过 postMessage() 发送的消息，后面的消息不会接收到；
- worker 线程当前 Event Loop 里的任务会继续执行，包括微任务；
- worker 线程里 setTimeout 创建的下一个 Event Loop 任务队列没有执行。

**在 worker 线程内部关闭 worker**

对上述例子稍作修改，将关闭 worker 的事件放到 worker 线程内部，大家觉得又会打印出什么呢

```js
// main.js（主线程）
const myWorker = new Worker('/worker.js'); // 创建 worker

myWorker.addEventListener('message', e => {
    console.log(e.data);
});

myWorker.postMessage('Greeting from Main.js');
```

```js
// worker.js（worker线程）

self.addEventListener('message', e => {

    postMessage('Greeting from Worker');
    
    self.close(); // 关闭 worker
    
    setTimeout(() => {
        console.log('setTimeout run');
        postMessage('Greeting from SetTimeout');
    });
    
    Promise.resolve().then(() => {
        console.log('Promise run');
        postMessage('Greeting from Promise');
    })
    
    for (let i = 0; i < 1001; i++) {
        if (i === 1000) {
            console.log('Loop run');
            postMessage('Greeting from Loop');
        }
    }
    
});
```

运行结果如下

```js
//--- loop run
//--- Promise run 
//--- Greeting from Worker
//--- Greeting from Loop
//--- Greeting from Promise
```
与在主线程关闭不同的是，worker 线程当前的 `Event Loop` 任务队列中的 `postMessage()` 事件都会被主线程监听到。

##  Worker 线程引用其他js文件

总有一些场景，需要放到 worker 进程去处理的任务很复杂，需要大量的处理逻辑，我们当然不想把所有代码都塞到`worker.js `里，那样就太糟糕了。不出意料，web worker 为我们提供了解决方案，我们可以在 worker 线程中利用 `importScripts()` 方法加载我们需要的js文件，而且，通过此方法加载的js文件不受**同源策略约束！**

```js
// utils.js
const add = (a, b) => a + b;
```
```js
// worker.js（worker线程）
// 使用方法：importScripts(path1, path2, ...); 

importScripts('./utils.js');

console.log(add(1, 2)); // log 3
```

##  ESModule 模式

还有一些场景，当你开启一个新项目，正高兴的用 `importScripts()` 导入js文件时发现， `importScripts()` 方法执行失败。仔细一看，原来是新项目的 js 文件都用的是 ESModule 模式。难道要把引用到的文件都改一遍吗？当然不用，还记得上文提到初始化 worker 时的第二个可选参数吗，我们可以直接使用 module 模式初始化 worker 线程！

```js
// main.js（主线程）
const worker = new Worker('/worker.js', {
    type: 'module'  // 指定 worker.js 的类型
});
```

```js
// utils.js
export default add = (a, b) => a + b;
```

```js
// worker.js（worker线程）
import add from './utils.js'; // 导入外部js

self.addEventListener('message', e => { 
    postMessage(e.data);
});

add(1, 2); // log 3

export default self; // 只需把顶级对象self暴露出去即可
```

##  主线程和 worker 线程可传递哪些类型数据

很多场景，在调用某些方法时，我们将一些自定义方法当作参数传入。但是，当你使用 `postMessage()` 方法时这么做，将会导致 `DATA_CLONE_ERR` 错误。


```js
// main.js（主线程）
const myWorker = new Worker('/worker.js'); // 创建worker

const fun = () => {};

myWorker.postMessage(fun); // Error：Failed to execute 'postMessage' on 'Worker': ()=>{} could not be cloned.
```

那么，使用 `postMessage()` 方法传递消息，可以传递哪些数据？
`postMessage()` 传递的数据可以是由结构化克隆算法处理的任何值或 JavaScript 对象，包括循环引用。
结构化克隆算法不能处理的数据：

- Error 以及 Function 对象；
- DOM 节点
- 对象的某些特定参数不会被保留
    - RegExp 对象的 lastIndex 字段不会被保留
    - 属性描述符，setters 以及 getters（以及其他类似元数据的功能）同样不会被复制。例如，如果一个对象用属性描述符标记为 read-only，它将会被复制为 read-write
    - 原形链上的属性也不会被追踪以及复制


结构化克隆算法**支持**的数据类型：

类型 | 说明
-|-
[所有的原始类型](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures#%E5%8E%9F%E5%A7%8B%E5%80%BC)|symbols 除外
[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean) 对象|
String 对象|
[Date](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date)|
[RegExp](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp)|`lastIndex` 字段不会被保留。
[`Blob`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)|
[`File`](https://developer.mozilla.org/zh-CN/docs/Web/API/File)|
[`FileList`](https://developer.mozilla.org/zh-CN/docs/Web/API/FileList)|
[ArrayBuffer](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)|
[ArrayBufferView](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBufferView)| 这基本上意味着所有的 [类型化数组](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Typed_arrays) ，如 Int32Array 等。
[`ImageData`](https://developer.mozilla.org/zh-CN/docs/Web/API/ImageData)|
[Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)|
[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)|仅包括普通对象（如对象字面量）
[Map](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map)|
[Set](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)|


# SharedWorker

**SharedWorker** 是一种特殊类型的 Worker，可以被多个浏览上下文访问，比如多个 windows，iframes 和 workers，但这些浏览上下文必须同源。它们实现于一个不同于普通 worker 的接口，具有不同的全局作用域：`SharedWorkerGlobalScope` ，但是继承自`WorkerGlobalScope`
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da793b904d904910bf4f3b37558510ea~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

`SharedWorker` 线程的创建和使用跟 `worker` 类似，事件和方法也基本一样。
不同点在于，主线程与 `SharedWorker` 线程是通过`MessagePort`建立起链接，数据通讯方法都挂载在`SharedWorker.port`上。
值得注意的是，如果你采用 `addEventListener` 来接收 `message` 事件，那么在主线程初始化`SharedWorker() `后，还要调用 `SharedWorker.port.start()` 方法来手动开启端口。


```js
// main.js（主线程）
const myWorker = new SharedWorker('./sharedWorker.js');

myWorker.port.start(); // 开启端口

myWorker.port.addEventListener('message', msg => {
    console.log(msg.data);
})
```
但是，如果采用 `onmessage` 方法，则默认开启端口，不需要再手动调用`SharedWorker.port.start()`方法

```js
// main.js（主线程）
const myWorker = new SharedWorker('./sharedWorker.js');

myWorker.port.onmessage = msg => {
    console.log(msg.data);
};
```

以上两种方式效果是一样的，具体信息请参考[MessagePort](https://developer.mozilla.org/en-US/docs/Web/API/MessagePort)。
由于 `SharedWorker` 是被多个页面共同使用，那么除了与各个页面之间的数据通讯是独立的，同一个`SharedWorker` 线程上下文中的其他资源都是共享的。基于这一点，很容易实现不同页面之间的数据通讯。

**一个利用`SharedWorker`实现多页面数据共享的例子**

1. index 页面的 add 按钮，每点击一次，向 sharedWorker 发送一次 add 数据，页面 count 增加1

```html
// index.html

<!DOCTYPE html>
<html lang="zh-CN">
    <head>
        <meta charset="utf-8">
        <title>index page</title>
    </head>
    <body>
        <p>index page: </p>
        count: <span id="container">0</span>
        <button id="add">add</button>
        <br>
        // 利用iframe加载
        <iframe src="./iframe.html"></iframe>
    </body>
    <script type="text/javascript">
        if (!!window.SharedWorker) {
            const container = document.getElementById('container');
            const add = document.getElementById('add');
            
            const myWorker = new SharedWorker('./sharedWorker.js');
            
            myWorker.port.start();

            myWorker.port.addEventListener('message', msg => {
                container.innerText = msg.data;
            });

            add.addEventListener('click', () => {
                myWorker.port.postMessage('add');
            });
        }
    </script>
</html>
```
2. iframe 页面的 reduce 按钮，每点击一次，向 sharedWorker 发送一次 reduce 数据，页面count 减少1

```html
<!DOCTYPE html>
<html lang="zh-CN">
    <head>
        <meta charset="utf-8">
        <title>iframe page</title>
    </head>
    <body>
        <p>iframe page: </p>
        count: <span id="container">0</span>
        <button id="reduce">reduce</button>
    </body>
    <script type="text/javascript">
        if (!!window.SharedWorker) {
            const container = document.getElementById('container');
            const reduce = document.getElementById('reduce');

            const myWorker = new SharedWorker('./sharedWorker.js');

            myWorker.port.start();
            
            myWorker.port.addEventListener('message', msg => {
                container.innerText = msg.data;
            })

            reduce.addEventListener('click', () => {
                myWorker.port.postMessage('reduce');
            });
        }
    </script>
</html>
```

3. sharedWorker 在接收到数据后，根据数据类型处理 num 计数，然后返回给每个已连接的主线程

```js
// sharedWorker.js

let num = 0;
const workerList = [];

self.addEventListener('connect', e => {
    const port = e.ports[0];
    port.addEventListener('message', e => {
        num += e.data === 'add' ? 1 : -1;
        workerList.forEach(port => { // 遍历所有已连接的part，发送消息
            port.postMessage(num);
        })
    });
    port.start();
    workerList.push(port); // 存储已连接的part
    port.postMessage(num); // 初始化
});
```

结果可以发现，index 页面和 iframe 页面的 count 始终保持一致，实现了多个页面数据同步。
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9b2916025a04d77a850f8196fd14015~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

`sharedWorker`调试
在 `sharedWorker` 线程里使用 console 打印信息，不会出现在主线程的的控制台中。如果你想调试 `sharedWorker`，需要在 Chrome 浏览器输入 [chrome://inspect/](chrome://inspect/) ，这里能看到所有正在运行的 `sharedWorker`，然后开启一个独立的 dev-tool 面板。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c95740b858ab4d43b329e582483037c7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)
