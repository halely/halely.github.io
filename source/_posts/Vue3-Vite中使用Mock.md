---
title: Vue3+Vite中使用Mock
date: 2023-02-28 17:01:41
cover: https://www.foodiesfeed.com/wp-content/uploads/2021/05/meal-wheel.jpg
tags:
  - vite
  - Vue
  - TypeScript
categories: 
  - 前端学习
---

# 前言

前后端分离前端需要数据可以使用`Mock`,更加方便，故学习
[vite-plugin-mock](https://github.com/vbenjs/vite-plugin-mock/blob/main/README.zh_CN.md)

# 开始

>**注意**:当前是3.0.0版本，目前发现修改mock文件会生成很多个xxx.mjs文件,控制台不停报错，[issue](https://github.com/vbenjs/vite-plugin-mock/issues/98),关注是否已经解决

## 安装 (yarn or npm)

```cmd
yarn add mockjs
# or
npm i  mockjs -S
# or
pnpm add mockjs
```

and

```cmd
yarn add vite-plugin-mock -D
# or
npm i vite-plugin-mock -D
# or
pnpm add vite-plugin-mock -D
```

## 开发环境使用

开发环境是使用 `Connect` 中间件实现的。
与生产环境不同，您可以在 `Google Chrome` 控制台中查看网络请求记录

```ts
//vite.config.ts 配置
import { UserConfigExport, ConfigEnv } from 'vite'
import vue from '@vitejs/plugin-vue'
import { viteMockServe } from 'vite-plugin-mock'
export default ({ command }: ConfigEnv): UserConfigExport => {
  return {
    plugins: [
      vue(),
      viteMockServe({
        mockPath: './src/mock',
        enable: true,
      }),
    ],
  }
}
```

```ts
//viteMockServe 配置
{
    mockPath?: string;
    ignore?: RegExp | ((fileName: string) => boolean);
    watchFiles?: boolean;
    enable?: boolean;
    ignoreFiles?: string[];
    configPath?: string;
    logger?:boolean;
}
```

- `mockPath`
    type: string
    default: mock
    设置模拟.ts 文件的存储文件夹
    如果watchFiles：true，将监视文件夹中的文件更改。 并实时同步到请求结果
    如果 configPath 具有值，则无效
- `ignore`
    type: RegExp | ((fileName: string) => boolean);
    default: undefined
    自动读取模拟.ts 文件时，请忽略指定格式的文件
- `watchFiles`
    type: boolean
    default: true
    设置是否监视mockPath对应的文件夹内文件中的更改
- `enable`
    type: boolean
    default: true
    是否启用 mock 功能
- `configPath`
    type: string
    default: vite.mock.config.ts
    设置模拟读取的数据条目。 当文件存在并且位于项目根目录中时，将首先读取并使用该文件。 配置文件返回一个数组
- `logger`
    type: boolean
    default: true
    是否在控制台显示请求日志

### Mock实例

```ts
 // src/mock/index.ts
import { MockMethod, MockConfig } from 'vite-plugin-mock'

// 第一种方式
export default [
  {
    url: '/api/get',
    method: 'get',
    response: ({ query }) => {
      return {
        code: 0,
        data: {
          name: 'vben',
        },
      }
    },
  },
  {
    url: '/api/post',
    method: 'post',
    timeout: 2000,
    response: {
      code: 0,
      data: {
        name: 'vben',
      },
    },
  },
  {
    url: '/api/text',
    method: 'post',
    rawResponse: async (req, res) => {
      let reqbody = ''
      await new Promise((resolve) => {
        req.on('data', (chunk) => {
          reqbody += chunk
        })
        req.on('end', () => resolve(undefined))
      })
      res.setHeader('Content-Type', 'text/plain')
      res.statusCode = 200
      res.end(`hello, ${reqbody}`)
    },
  },
] as MockMethod[]
// 第二种方式
export default function (config: MockConfig) {
  return [
    {
      url: '/api/text',
      method: 'post',
      rawResponse: async (req, res) => {
        let reqbody = ''
        await new Promise((resolve) => {
          req.on('data', (chunk) => {
            reqbody += chunk
          })
          req.on('end', () => resolve(undefined))
        })
        res.setHeader('Content-Type', 'text/plain')
        res.statusCode = 200
        res.end(`hello, ${reqbody}`)
      },
    },
  ]
}

 ```

Mock配置

```ts
{
  // 请求地址
  url: string;
  // 请求方式
  method?: MethodType;
  // 设置超时时间
  timeout?: number;
  // 状态吗
  statusCode?:number;
  // 响应数据（JSON）
  response?: ((opt: { [key: string]: string; body: Record<string,any>; query:  Record<string,any>, headers: Record<string, any>; }) => any) | any;
  // 响应（非JSON）
  rawResponse?: (req: IncomingMessage, res: ServerResponse) => void;
}

```

## 生产环境中的使用

创建`mockProdServer.ts`文件

```ts
//  mockProdServer.ts
import { createProdMockServer } from 'vite-plugin-mock/client'

// 逐一导入您的mock.ts文件
// 如果使用vite.mock.config.ts，只需直接导入文件
// 可以使用 import.meta.glob功能来进行全部导入
import testModule from '../mock/test'

export function setupProdMockServer() {
  createProdMockServer([...testModule])
}
```

```ts
//配置 vite-plugin-mock
import { viteMockServe } from 'vite-plugin-mock'
import { UserConfigExport, ConfigEnv } from 'vite'
export default ({ command }: ConfigEnv): UserConfigExport => {
  return {
    plugins: [
      viteMockServe({
        mockPath: 'mock',
        // 根据项目配置。可以配置在.env文件
        enable: true,
      }),
    ],
  }
}
```

加餐

如何运行打包好的dist文件

```cmd
npm install -g http-server
cd dist
# 启动http-server。在命令行中执行以下命令：
http-server
```

此时，在命令行中会输出生成的服务网站

>注意。默认网址端口号是8080，如果已经被占用可使用 `​​​​​​​http-server -p 8081` 来修改端口


