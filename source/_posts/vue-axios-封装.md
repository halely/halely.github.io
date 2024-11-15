---
title: vue axios 封装
date: 2022-04-28 10:21:37
cover: https://www.foodiesfeed.com/wp-content/uploads/2023/04/delicious-steak-with-herbs-cut-on-slices.jpg  
tags:
 - Vue
 - TypeScript
categories: 
  - 前端学习
---

# 前言

为了方便开发，个人封装`axios`

1. 在utils目录中构架`api`文件夹
2. 设置 `axios.ts`、`status.ts`

```ts
//axios.ts
import axios from 'axios'
import { showMessage } from "./status";   // 引入状态码文件
import { saveAs } from 'file-saver';//file-saver导出文件
import { tansParams, blobValidate } from '../index'

// 创建axios实例
const service = axios.create({
    // axios中请求配置有baseURL选项，表示请求URL公共部分
    baseURL: '',
    // 超时
    timeout: 16000
})

//http request 拦截器
service.interceptors.request.use(
    config => {
        // 是否不需要设置 token
        const isNoToken: boolean = ((config.headers || {}) as any).isToken === false
        // 配置请求头
        if (!isNoToken) {
            // 让每个请求携带自定义token 
            //getToken()为你获取token的方法 请根据实际情况自行修改
            // config.headers['token'] = getToken() 
        }
        return config;
    },
    error => {
        return Promise.reject(error);
    }
);

//http response 拦截器
service.interceptors.response.use(
    response => {
        // 二进制数据则直接返回
        if (response.request.responseType === 'blob' || response.request.responseType === 'arraybuffer') {
            return response.data
        }
        //其他状态自行判断
        return response.data;
    },
    error => {
        const { response } = error;
        if (response) {
            // 请求已发出，但是不在2xx的范围
            showMessage(response.status);           // 传入响应码，匹配响应码对应信息
            return Promise.reject(response.data);
        } else {
            //弹出报错方式，根据使用的ui配置
            // ElMessage.warning('网络连接异常,请稍后再试!');
        }
    }
);
// 封装 GET POST 请求并导出
export function request(url:string = '', type:string = 'POST', params:any = {},headers: any={}) {
    //设置 url params type 的默认值
    return new Promise((resolve, reject) => {
        let promise
        if (type.toUpperCase() === 'GET') {
            promise = service({
                url,
                params,
                headers
            })
        } else if (type.toUpperCase() === 'POST') {
            promise = service({
                method: 'POST',
                url,
                data: params,
                headers
            })
        }
        //处理返回
        (promise as Promise<any>).then(res => {
            resolve(res)
        }).catch(err => {
            reject(err)
        })
    })
}

// 通用下载方法
export function download(url: string, params: any, filename: string | undefined, config: any) {
    //1.可以在这步设置loading提示
    return service.post(url, params, {
        transformRequest: [(params) => { return tansParams(params) }],
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        responseType: 'blob',
        ...config
    }).then(async (data: any) => {
        const isLogin = await blobValidate(data);
        if (isLogin) {
            const blob = new Blob([data])
            saveAs(blob, filename)
        } else {
            const resText = await data.text();
            const rspObj = JSON.parse(resText);
            console.error(rspObj)
        }
    }).catch((r) => {
        console.error(r)
        console.error('下载文件出现错误，请联系管理员！')
    })
}
```

```ts
//status.ts
/*
 * @description 管理接口返回状态码
*/

export const showMessage = (status: number | string): string => {
    let message: string = "";
    switch (status) {
        case 400:
            message = "请求错误(400)";
            break;
        case 401:
            message = "未授权，请重新登录(401)";
            break;
        case 403:
            message = "拒绝访问(403)";
            break;
        case 404:
            message = "请求出错(404)";
            break;
        case 408:
            message = "请求超时(408)";
            break;
        case 500:
            message = "服务器错误(500)";
            break;
        case 501:
            message = "服务未实现(501)";
            break;
        case 502:
            message = "网络错误(502)";
            break;
        case 503:
            message = "服务不可用(503)";
            break;
        case 504:
            message = "网络超时(504)";
            break;
        case 505:
            message = "HTTP版本不受支持(505)";
            break;
        default:
            message = `连接出错(${status})!`;
    }
    return `${message}，请检查网络或联系管理员！`;
};

```

补充 引用方法

```ts
/**
* 参数处理
* @param {*} params  参数
*/
interface Params {
    [key: string]: any
}
export function tansParams(params: Params) {
    let result = ''
    for (const propName of Object.keys(params)) {
        const value = params[propName];
        var part = encodeURIComponent(propName) + "=";
        if (value !== null && value !== "" && typeof (value) !== "undefined") {
            if (typeof value === 'object') {
                for (const key of Object.keys(value)) {
                    if (value[key] !== null && value[key] !== "" && typeof (value[key]) !== 'undefined') {
                        let params = propName + '[' + key + ']';
                        var subPart = encodeURIComponent(params) + "=";
                        result += subPart + encodeURIComponent(value[key]) + "&";
                    }
                }
            } else {
                result += part + encodeURIComponent(value) + "&";
            }
        }
    }
    return result
}

// 验证是否为blob格式
export async function blobValidate(data: Blob | any) {
    try {
        const text = await data.text();
        JSON.parse(text);
        return false;
    } catch (error) {
        return true;
    }
}
```
