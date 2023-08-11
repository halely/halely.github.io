---
title: Rollup构建TS项目 & webpack构建TS项目 & esbuild + swc
date: 2023-06-06 15:32:12
tags:
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
    "dev": "cross-env NODE_ENV=development  rollup -c -w",
    "build":"cross-env NODE_ENV=produaction  rollup -c"
  },
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
import ts from 'rollup-plugin-typescript2'//识别ts文件
import { terser } from 'rollup-plugin-terser'//代码压缩
import server from 'rollup-plugin-serve'//启动服务
import livereload from 'rollup-plugin-livereload'//热更新
import replace from 'rollup-plugin-replace'//注册浏览器参数
console.log(process.env);//可以获取配置的环境变量
const isDev = () => {
    return process.env.NODE_ENV === 'development'
}
export default {
    input: './src/index.ts',//入口
    //出口
    output: {
        file: path.resolve(__dirname, './lib/index.js'),
        format: 'umd',//格式
        sourcemap: true,//生成sourcemap文件，定位问题，否则定位的代码是打包后的文件
    },
    plugins: [
        ts(),//需要开启ts配置"module"为"ES2015",  如果CommonJS会兼容报错  
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
