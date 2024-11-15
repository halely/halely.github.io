---
title: Rollup构建TS项目 & webpack构建TS项目 & esbuild + swc
date: 2023-06-06 15:32:12
cover: https://www.foodiesfeed.com/wp-content/uploads/2023/08/salmon-appetizers.jpg
tags:
  - JavaScript
  - TypeScript
  - webPack
  - Rollup
categories: 
  - 前端学习
---

# Rollup构建TS项目

[Rollup](https://so.csdn.net/so/search?q=Rollup) 打包很快,体积很小，学习一下

**配置环境**

1. `npm init -y` 生成 `package.json`
2. `tsc --init` 生成 `tsconfig.json`
3. 创建 `rollup.config.js`
4. 创建入口问题`/src/index.ts`文件和`public/index.html`文件

**安装依赖**

1. 全局安装rollup `npm install rollup-g`
2. 安装TypeScript   `npm install typescript -D`
3. 安装TypeScript 转换器 `npm install rollup-plugin-typescript2 -D`
4. 安装代码压缩插件`npm install rollup-plugin-terser -D`
5. 安装rollupWeb服务 `npm install rollup-plugin-serve -D`
6. 安装热更新 `npm install rollup-plugin-livereload -D`
7. 引入外部依赖 `npm install rollup-plugin-node-resolve -D`
8. 安装配置环境变量用来区分本地和生产  `npm install cross-env -D`
9. 替换环境变量给浏览器使用 `npm install rollup-plugin-replace -D`

**配置文件**

```json
// package.json
{
  "name": "rollupTs",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "cross-env NODE_ENV=development rollup -c -w",
    "build": "cross-env NODE_ENV=produaction rollup -c"
  },
  "type": "module",
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "cross-env": "^7.0.3",
    "rollup-plugin-livereload": "^2.0.5",
    "rollup-plugin-node-resolve": "^5.2.0",
    "rollup-plugin-replace": "^2.2.0",
    "rollup-plugin-serve": "^1.1.0",
    "rollup-plugin-terser": "^7.0.2",
    "rollup-plugin-typescript2": "^0.31.1",
    "typescript": "^4.5.5"
  }
}

```

`rollup -c`:打包命令
`rollup -c -w`:启动本地服务命令
`cross-env NODE_ENV=development`:配置**node** `process.env`环境变量用来区分本地和生产，注意，这在浏览器中不能直接获取的，需要插件`replace`配置

```js
//rollup.config.js
import path from 'path'
import ts from 'rollup-plugin-typescript2'//识别入口ts文件
import { terser } from 'rollup-plugin-terser'//代码压缩
import server from 'rollup-plugin-serve'//启动服务
import livereload from 'rollup-plugin-livereload'//热更新
import replace from 'rollup-plugin-replace'//注册浏览器参数
import { fileURLToPath } from 'url'
const __filenameNew = fileURLToPath(import.meta.url)
const __dirnameNew = path.dirname(__filenameNew)
const isDev = () => {
    return process.env.NODE_ENV === 'development'
}
export default {
    input: './src/index.ts',//入口
    //出口
    output: {
        //直接使用__dirname 会报错__dirname is not defined in ES module scope，因为rollup需要配置 "type": "module",，但是module不支持__dirname，故
        //需要创建__dirnameNew替代
        file: path.resolve(__dirnameNew, './lib/index.js'),
        format: 'umd',//输出格式
        sourcemap: true,//生成sourcemap文件，定位问题，否则定位的代码是打包后的文件
    },
    plugins: [
        ts(),
        isDev() && livereload(),//热更新组件
        terser({
            compress: {
                drop_console: !isDev(),//去除console
            }
        }),//压缩文件
        replace({
            'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV)//因为process是在node环境的下的参数，在浏览器上是访问不了的，所以需要注册到浏览器上
        }),
        isDev() && server({
            open: true,//是否启动打开页面
            port: 1988,//配置端口号
            openPage: '/public/index.html',//打开的页面
        }),

    ]

}
```

**js代码使用环境变量**

```js
let label: string = 'hale最喜欢八味';
console.log(label)
if (process.env.NODE_ENV === 'development') {
    alert('开发')
} else {
    alert('生产')
}
console.log(process.env.NODE_ENV)
```

**TS配置需要注意的**

```json
//tsconfig.json
{
"module": "ES2015", //需要设置ts配置"module"为"ES2015",CommonJS无法解析识别ts文件插件rollup-plugin-typescript2
"sourceMap": true, //生成sourcemap文件，定位问题，否则定位的代码是打包后的文件        
}
```

# Webpack构建TS项目

**配置环境**

1. `npm init -y` 生成 `package.json`
2. 创建入口问题`/src/index.ts`文件和`public/index.html`文件
3. `tsc --init` 生成 `tsconfig.json`
4. 创建 `webpack.config.js`

**安装依赖**

1. 安装webpack   `npm install webpack -D`
2. webpack4以上需要 `npm install  webpack-cli -D`
3. 编译TS  `npm install ts-loader -D`
4. TS环境 `npm install typescript -D`
5. 热更新服务 `npm install  webpack-dev-server -D`
6. HTML模板 `npm install html-webpack-plugin -D`

**配置文件**

```json
{
  "name": "webpackTs",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server",
    "build": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "html-webpack-plugin": "^5.5.3",
    "ts-loader": "^9.4.4",
    "typescript": "^5.1.6",
    "webpack": "^5.88.2",
    "webpack-cil": "0.0.1-security",
    "webpack-cli": "^5.1.4",
    "webpack-dev-server": "^4.15.1"
  },
  "dependencies": {}
}

```

```js
//webpack.config.js
const path = require('path')
const htmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
    entry: "./src/index.ts",//入口
    //出口
    output: {
        path: path.resolve(__dirname, './dist'),
        filename: "index.js"
    },
    mode: "development",
    stats: "none",
    //配置文件引入
    resolve: {
        extensions: ['.ts', '.js'],
        alias: {
            '@': path.resolve(__dirname, './src')
        }
    },
    //配置文件使用loader
    module: {
        rules: [
            {
                test: /\.ts$/,
                use: "ts-loader"
            }
        ]
    },
    devServer: {
        port: 1995,
        proxy: {}
    },
    //插件
    plugins: [
        new htmlWebpackPlugin({
            template: "./public/index.html"
        })
    ]
}
```

# esbuild + swc 构建ts项目

前端工具层出不穷，之前有常用的打包工具`webpack`，现在有了速度更快的`vite`。 vite的开发模式是**基于esBuild编译的,打包又是基于rollup**,启动项目是很快的。

`esbuild`为什么这么快
在esbuild的官方介绍中打包`threejs` 只需要0.37秒 无限接近于亚索的Q技能冷却时间可以说是很快了。

`esbuild`是`go`语言编写的并且是多线程执行,性能是js的好几十倍，所以很快。

- 无需缓存即可实现基础打包
- 支持 ES6 跟 CommonJS 模块
- 支持ES 6 Tree Shaking
- 体积小
- 插件化
- 其他
- 内置支持编译 jsx

**SWC**
`SWC`则宣称其比`Babel`快**20**倍(四核情况下可以快**70**倍)

wc是用`rust`写的，所实现的功能跟`babel`一样，es6语法转es5，但是速度比`babel`更快，前端基建工具使用rust的是越来越多了，未来可能还会有一个替代postCss的😂。
那如果把`esbuild + swc`结合起来构建那岂不是接近光速 我们来`try try`

**配置环境**

1. `npm init -y` 生成 `package.json`
2. `tsc --init` 生成 `tsconfig.json`
3. 创建 `config.js`
4. 创建入口问题`/src/index.ts`文件和`public/index.html`文件

**安装依赖**

1. 全局安装 `npm install @swc/core esbuild @swc/helpers`
2. 安装nodeTs声明 `npm install --save-dev @types/node`

其中，`@swc/core` 是 swc 的核心包，用于编译 JavaScript 和 TypeScript 代码；esbuild 是一个快速的 JavaScript 和 TypeScript 构建工具；`@swc/helpers` 是 swc 的辅助包，用于转换 JSX 代码。

```ts
//config.ts
import esbuild from 'esbuild'//打包工具
import swc from '@swc/core'//类似于babel es6 转 es5
import fs from 'node:fs'
await esbuild.build({
    entryPoints: ['./index.ts'], //入口文件
    bundle: true, //模块单独打包
    loader: {
        '.js': 'js',
        '.ts': 'ts',
        '.jsx': 'jsx',
        '.tsx': 'tsx',
    },
    treeShaking:true,
    define: {
       'process.env.NODE_ENV': '"production"',
    },
    plugins: [
        {
            //实现自定义loader
            name: "swc-loader",
            setup(build) {
                build.onLoad({ filter: /\.(js|ts|tsx|jsx)$/ }, (args) => {
                   // console.log(args);
                    const content = fs.readFileSync(args.path, "utf-8")
                    const { code } = swc.transformSync(content, {
                        filename: args.path
                    })
                    return {
                        contents: code
                    }
                })
            },
        }
    ],
    outdir: "dist"
})
```

**测试demo**

```ts
export const a:number = 1
export const b:string = 'ikun'
let x = 1
let fn = () => 123
console.log(x,fn);
```

**转移之后的**

```js
export var a = 1;
export var b = "ikun";
var x = 1;
var fn = function() {
  return 123;
};
console.log(x, fn);
```

除了上述基本用法之外，swc 和 esbuild 还提供了许多高级用法，可以更好地满足我们的构建需求。

**插件系统**

swc 和 esbuild 都提供了插件系统，可以通过插件来扩展其功能。例如，swc 的插件可以用于优化代码，提高性能。esbuild 的插件则可以用于处理特定类型的文件，或自定义转换规则。

**缓存系统**

swc 和 esbuild 都提供了缓存系统，可以减少重复编译时间。当文件内容没有发生变化时，swc 和 esbuild 会从缓存中读取已编译的代码，以提高构建速度。

**Watch 模式**

swc 和 esbuild 都支持 Watch 模式，可以在文件发生变化时自动重新编译代码。Watch 模式可以减少手动运行构建命令的频率，提高开发效率。

**自定义插件**

最后，我们可以通过编写自定义插件来扩展 swc 和 esbuild 的功能。例如，可以编写一个插件来自动引入 CSS 文件，或者优化 JavaScript 代码。自定义插件可以根据实际需求进行编写，以更好地满足项目的构建需求。

**结论**

本文介绍了如何使用 swc 和 esbuild 来构建一个简单的 TypeScript 应用程序，并讨论了一些高级用法。swc 和 esbuild 都是现代前端构建工具中的代表，它们都提供了快速编译、代码压缩等功能，可以有效提高应用程序的性能。通过学习 swc 和 esbuild 的使用方法，我们可以更好地进行前端工程化开发，提高开发效率和代码质量。
