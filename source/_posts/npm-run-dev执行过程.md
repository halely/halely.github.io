---
title: npm run dev执行过程
date: 2023-03-10 16:04:23
cover: 'https://www.foodiesfeed.com/wp-content/uploads/2015/05/girl-holding-coffee-espresso.jpg'
tags:
 - npm
 - node
---


当执行 `npm run dev`命令的时候，会去项目的`package.json`文件对象的`scripts`对象中去寻找，然后执行对应的命令，比如我们使用`vite`去构建`vue`项目。配置文件

```json
"scripts": {
    "dev": "vite",
    "build": "vue-tsc && vite build",
    "preview": "vite preview"
  }
```
如上，找到`dev`后,会去执行`vite` 命令,但是如果我们直接执行 `vite`,那么就会报错，所以就可以得到执行`vite`的时候这边项目做了配置。

其实在执行的时候，这边会在本地的`node_modules`文件中去寻找`vite`去执行,如果没有就会去 `npm install -g`（全局包）中去寻找，如果这里面有，也可以去执行，如果这里也没有，那么就去直接去系统的环境变量中去寻找，但是一般你没配置就没有了，然后就报错，这就是这边执行的全部过程。

那么这边是vite是怎么配置的呢，在`node_modules`文件指定vite配置了一个软连接，指向的一个`bin/vite.js`的文件 在这个文件里面有对`vite`的配置，去构建一个临时的环境变量，然后在这边执行，其他的比如打包什么的都是同理
