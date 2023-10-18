---
title: vite-config配置
date: 2022-06-21 14:51:43
cover: "http://p1.qhimg.com/bdm/0_0_100/t013c3def46b8910030.jpg"
tags:
 - vite
 - JavaScript
---

# 前言

目前主要使用的是构建框架是vite,因此记录一下配置项。

# 一、使用 vite 创建的项目里默认的配置

使用 vite 创建项目完成后会自动生成 一个 vite.config.js 文件，当然你可以将其名重定义为 vite.config.ts 文件。其默认代码如下：

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
export default defineConfig({
 plugins: [vue()],
})
```

当以命令方式运行 vite 时，vite 会自动解析项目根目录下 vite.config.js 的文件。配置不全时，在开发环境下运行都是正常的，但是打包上线的时候就会出现各种问题。如：

- 假设不配置 base 时，打包之后，访问时出现白屏。
- alias 不配置的时候，每次引入文件需要找根目录，比较麻烦。

**以下是 vite.config.js 的更多常用参数配置以及意义：**

```js
//vite.config.js
import { defineConfig } from 'vite' // 帮手函数，这样不用 jsdoc 注解也可以获取类型提示
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'
import { viteMockServe } from "vite-plugin-mock"

const localEnabled = process.env.USE_MOCK || false;
const prodEnabled = process.env.USE_CHUNK_MOCK || false;
export default defineConfig({
  //项目根目录
  root: process.cwd(),
  //项目部署的基础路径
  base: "/",
  //环境配置 'development'|'production'
  mode: "development",
  //全局常量替换 Record<string, string>
  define: {
    "": "",
    user: "users",
  },
  //插件
  plugins: [vue(),
  viteMockServe({
   mockPath: "./src/server/mock",
   localEnabled: localEnabled, // 开发打包开关 true时打开mock  false关闭mock
   prodEnabled: prodEnabled,//prodEnabled, // 生产打包开关
   // 这样可以控制关闭mock的时候不让mock打包到最终代码内
   injectCode: `
    import { setupProdMockServer } from './mockProdServer';
    setupProdMockServer();
   `,
   logger: false, //是否在控制台显示请求日志
   supportTs:false //打开后，可以读取 ts 文件模块 打开后将无法监视 .js 文件
  })],
  //静态资源服务的文件夹
  publicDir: "public",
  //存储缓存文件的目录
  cacheDir: "node_modules/.vite",
  resolve: {
    //别名
    alias: {
      "@":resolve(__dirname, "/src"),
      comps: resolve(__dirname, "/src/components"),
    },
    dedupe: [],
    //解决程序包中package.json配置中的exports 字段
    conditions: [],
    //解析package.json中字段的优先级
    mainFields: ["module", "jsnext:main", "jsnext"],
    //导入时想要省略的扩展名列表
    extensions: [".mjs", ".js", ".ts", ".jsx", ".tsx", ".json"],
    //使Vite通过原始文件路径而不是真正的文件路径确定文件身份
    preserveSymlinks: false,
  },
  css: {
    //配置 CSS modules 的行为。选项将被传递给 postcss-modules。
    modules: {},
    // PostCSS 配置（格式同 postcss.config.js）
    // postcss-load-config 的插件配置
    postcss: {},
    //指定传递给 CSS 预处理器的选项
    preprocessorOptions: {
      scss: {
        additionalData: `$injectedColor: orange;`,
      },
    },
    //开发过程中是否启sourcemap
    devSourcemap: false,
  },
  json: {
    //是否支持从 .json 文件中进行按名导入
    namedExports: true,
    //若设置为 true，导入的 JSON 会被转换为 export default JSON.parse("...") 会比转译成对象字面量性能更好，
    stringify: false,
  },
  //继承自 esbuild 转换选项。最常见的用例是自定义 JSX
  esbuild: {
    jsxFactory: "h",
    jsxFragment: "Fragment",
    jsxInject: `import React from 'react'`,
  },
  //静态资源处理  字符串|正则表达式
  assetsInclude: ["**/*.gltf"],
  //调整控制台输出的级别 'info' | 'warn' | 'error' | 'silent'
  logLevel: "info",
  //设为 false 可以避免 Vite 清屏而错过在终端中打印某些关键信息
  clearScreen: true,
  //加载 .env 文件的目录
  envDir: "",
  //envPrefix开头的环境变量会通过import.meta.env暴露客户端源码
  envPrefix: "VITE_",
  //设置'spa' | 'mpa' | 'custom'应用操作
  appType: "spa",
  //服务
  server: {
    //服务器主机名
    host: "localhost",
    //端口号
    port: "5173",
    //设为 true 时若端口已被占用则会直接退出，而不是尝试下一个可用端口
    strictPort: true,
    //https.createServer()配置项
    https: "",
    //服务器启动时自动在浏览器中打开应用程序。
    open: "/docs/index.html",
    //自定义代理规则
    proxy: {
      // 字符串简写写法
      "/foo": "http://localhost:4567",
      // 选项写法
      "/api": {
        target: "http://jsonplaceholder.typicode.com",
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ""),
      },
      // 正则表达式写法
      "^/fallback/.*": {
        target: "http://jsonplaceholder.typicode.com",
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/fallback/, ""),
      },
      // 使用 proxy 实例
      "/api": {
        target: "http://jsonplaceholder.typicode.com",
        changeOrigin: true,
        configure: (proxy, options) => {
          // proxy 是 'http-proxy' 的实例
        },
      },
      // Proxying websockets or socket.io
      "/socket.io": {
        target: "ws://localhost:3000",
        ws: true,
      },
    },
    //开发服务器配置 CORS
    cors: {},
    //指定服务器响应的 header ,类型OutgoingHttpHeaders
    header: {},
    //禁用或配置 HMR 连接
    hmr: {},
    //传递给 chokidar 的文件系统监视器选项
    watch: {},
    //中间件模式创建 Vite 服务器,'ssr' | 'html'
    middlewareMode: "ssr",
    //HTTP请求中预留此文件夹，用于代理 Vite 作为子文件夹时使用。
    base: "",
    fs: {
      //限制为工作区 root 路径以外的文件的访问
      strict: true,
      //限制哪些文件可以通过 /@fs/ 路径提供服务
      allow: [
        // 搜索工作区的根目录
        searchForWorkspaceRoot(process.cwd()),
        // 自定义规则
        "/path/to/custom/allow",
      ],
      //限制Vite开发服务器提供敏感文件的黑名单
      deny: [".env", ".env.*", "*.{pem,crt}"],
    },
    //定义开发调试阶段生成资产的url
    origin: "http://127.0.0.1:8080",
  },
  //构建
  build: {
    //浏览器兼容性  "esnext"|"modules"
    target: "modules",
    //否自动注入 module preload 的 polyfill
    polyfillModulePreload: true,
    //输出路径
    outDir: "dist",
    //生成静态资源的存放路径
    assetsDir: "assets",
    //小于此阈值的导入或引用资源将内联为 base64 编码，以避免额外的 http 请求。设置为 0 可以完全禁用此项
    assetsInlineLimit: 4096,
    //启用/禁用 CSS 代码拆分
    cssCodeSplit: true,
    //不同的浏览器target设置CSS的压缩
    cssTarget: "",
    //构建后是否生成 source map 文件
    //boolean | 'inline' | 'hidden'
    sourcemap: false,
    //自定义底层的 Rollup 打包配置
    rollupOptions: {
      //要打包的文件路径
      input: "src/main.js",
      //文件输出位置
      output: {
        //打包生产文件路径
        file: "dist/index.js",
        //打包输出格式
        // "amd", "cjs", "system", "es", "iife" or "umd
        format: "cjs",
        //包的全部变量名称
        name: "bundleName",
        //声明全局变量
        globals: {
          jquery: "$",
        },
      },
      //插件
      plugins: [],
      //不需打包的文件
      external: ["lodash"],
    },
    //@rollup/plugin-commonjs 插件的选项
    commonjsOptions: {},
    //@rollup/plugin-dynamic-import-vars 选项
    dynamicImportVarsOptions: {},
    //构建的库
    lib: {
      entry:resolve(__dirname, "lib/main.js"),
      //暴露的全局变量
      name: "mylib",
      //'es' | 'cjs' | 'umd' | 'iife'
      formats: "es",
      //输出的包文件名
      fileName: "my-lib",
    },
    //当设置为 true，构建后将会生成 manifest.json 文件
    manifest: false,
    //当设置为 true，构建后将会生成SSR的manifest.json 文件
    ssrManifest: false,
    //生成面向 SSR 的构建
    ssr: "undefined",
    //设置为 false 可以禁用最小化混淆，
    //boolean | 'terser' | 'esbuild'
    minify: "esbuild",
    //传递给 Terser 的更多 minify 选项。
    terserOptions: {},
    //设置为 false 来禁用将构建后的文件写入磁盘
    write: true,
    //默认情况下，若 outDir 在 root 目录下，则 Vite 会在构建时清空该目录。
    emptyOutDir: true,
    //启用/禁用 gzip 压缩大小报告
    reportCompressedSize: true,
    //触发警告的 chunk 大小（以 kbs 为单位）
    chunkSizeWarningLimit: 500,
    //设置为 {} 则会启用 rollup 的监听器
    watch: null,
  },
  //开发服务器
  preview: {
    //开发服务器主机名
    host: "localhost",
    //开发服务器端口号
    port: "5173",
    //设为 true 时若端口已被占用则会直接退出，而不是尝试下一个可用端口
    strictPort: true,
    //https.createServer()配置项
    https: "",
    //服务器启动时自动在浏览器中打开应用程序。
    open: "/docs/index.html",
    //开发服务器，自定义代理规则
    proxy: {
      // 字符串简写写法
      "/foo": "http://localhost:4567",
      // 选项写法
      "/api": {
        target: "http://jsonplaceholder.typicode.com",
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ""),
      },
      // 正则表达式写法
      "^/fallback/.*": {
        target: "http://jsonplaceholder.typicode.com",
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/fallback/, ""),
      },
      // 使用 proxy 实例
      "/api": {
        target: "http://jsonplaceholder.typicode.com",
        changeOrigin: true,
        configure: (proxy, options) => {
          // proxy 是 'http-proxy' 的实例
        },
      },
      // Proxying websockets or socket.io
      "/socket.io": {
        target: "ws://localhost:3000",
        ws: true,
      },
    },
    //开发服务器配置 CORS
    cors: {},
  },
  //依赖优化选项
  optimizeDeps: {
    //检测需要预构建的依赖项
    entries: [],
    //预构建中强制排除的依赖项
    exclude: ["jquery"],
    //默认情况下，不在 node_modules 中的，链接的包不会被预构建。使用此选项可强制预构建链接的包。
    include: ['axios'],
    //部署扫描和优化过程中传递给EsBuild
    esbuildOptions: {},
    //设置为 true 可以强制依赖预构建，而忽略之前已经缓存过的、已经优化过的依赖
    force: true,
  },
  //SSR 选项
  ssr: {
    //列出的是要为 SSR 强制外部化的依赖
    external: [],
    //列出的是防止被 SSR 外部化依赖项。
    noExternal: [],
    //SSR 服务器的构建目标
    target: "node",
    //SSR 服务器的构建语法格式 'esm' | 'cjs'
    format: "esm",
  },
  worker: {
    //worker 打包时的输出类型 'es' | 'iife'
    format: "iife",
    // worker 打包的 Vite 插件
    plugins: [],
    //打包 worker 的 Rollup 配置项
    rollupOptions: {},
  },
});

```
