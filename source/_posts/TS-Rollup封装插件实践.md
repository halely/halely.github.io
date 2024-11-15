---
title: TS-Rollup封装插件实践
date: 2023-08-22 17:23:07
cover: https://www.foodiesfeed.com/wp-content/uploads/2023/08/puffed-pastry-with-vegetables.jpg
tags:
  - JavaScript
  - TypeScript
  - Rollup
categories: 
  - 前端学习
---

# 前言

使用`cookie`是可以设置有效期的，但`localStorage`本身是没有该机制的，只能手动删除，否则会一直存放在浏览器当中，我们可以把localStorage跟cookie一样设置一个有效期进行二次封装实现该方案。

在存储的时候设置一个过期时间，并且存储的数据进行格式化方便统一校验，在读取的时候获取当前时间进行判断是否过期，如果过期进行删除即可。

# 实现

```ts
// /enum ts 定义枚举
//字典 Dictionaries    expire过期时间key    permanent永久不过期
export enum Dictionaries {
    expire = '__expire__',
    permanent = 'permanent'
}
```

```ts
//type ts 定义类型
//expire  过期时间key  //permanent 永久不过期
import { Dictionaries } from "../enum";
export type Str=string;
export type Expire = Dictionaries.permanent | number //有效期类型
export interface Date<T>{
   value:T,
   [Dictionaries.expire]:Expire
}

export interface Result<T> { //返回值类型
    message: string,
    value: T | null
}

export interface StorageCls {
    get: <T>(key:Str) => Result<T | null>;
    set: <T>(key:Str,value:T,expire:Expire) => void;
    remove: (key: Str) => void;
    clear: () => void;
}
```

```ts
//index.ts
//expire  过期时间key permanent 永久不过期
import { StorageCls, Str, Expire, Date, Result } from "./type";
import { Dictionaries } from "./enum";
export class Storage implements StorageCls {
    //获取localStorage
    public  get<T = any>(key: Str): Result<T | null> {
        const value = localStorage.getItem(key)
        if (value) {
            const data: Date<T> = JSON.parse(value);
            const now = new Date().getTime();
            //有效并且是数组类型 并且过期了 进行删除和提示
            if (typeof data[Dictionaries.expire] == 'number' && data[Dictionaries.expire] < now) {
                this.remove(key)
                return {
                    message: `您的${key}已过期`,
                    value: null
                }
            } else {
                //否则成功返回
                return {
                    message: "成功读取",
                    value: data.value
                }
            }
        }
        return {
            message: '值无效',
            value: null
        }
    }
    //设置localStorage和过期时间
    set<T = any>(key: Str, value: T, expire: Expire = Dictionaries.permanent) {
        //格式化数据
        const data: Date<T> = {
            value,
            [Dictionaries.expire]: expire
        }
        localStorage.setItem(key, JSON.stringify(data));

    }
    //删除
    remove(key: Str) {
        localStorage.removeItem(key);
    }
    //清除
    clear() {
        localStorage.clear()
    }
}
```

```ts
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
        file: path.resolve(__dirnameNew, './lib/index.js'),
        // format: 'umd',//输出格式
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
