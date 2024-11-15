---
title: vue搭建和配置
date: 2023-01-30 17:44:26
cover: https://www.foodiesfeed.com/wp-content/uploads/2023/09/can-of-fresh-soda.jpg
tags:
 - Vue
 - npm
categories: 
  - 前端学习
---

# vue2

# vue3

Vue3内置的构建工具是Vite,
vite的优势：

- `冷服务`   默认的构建目标浏览器是能 在 script 标签上支持原生 ESM 和 原生 ESM 动态导入
- `HMR`  速度快到惊人的 模块热更新（HMR）
- `Rollup打包`  它使用 Rollup 打包你的代码，并且它是预配置的 并且支持大部分rollup插件

## vite构建

```sh
# npm
npm init vite@latest
# yarn
yarn create vite
```

## vue直接构建

这个配置比yarn配置会更详细一点

```sh
npm init vue@latest
```

### vite.config.js基本配置

```js
import { defineConfig } from "vite";// 帮手函数，这样不用 jsdoc 注解也可以获取类型提示
import { resolve } from "path";//主要用于alias文件路径别名
import vue from '@vitejs/plugin-vue'; //插件来支持Vue

//基于resolve写个小方法，方便以后新增别名使用(非必要)
function pathResolve(dir) {
  return resolve(__dirname, ".", dir);
}

export default defineConfig({
    base: "",//在生产中服务时的基本公共路径
    // 配置需要使用的插件列表，这里将vue添加进去
    plugins:[vue()],
    publicDir: 'public',  // 静态资源服务的文件夹, 默认"public"
    resolve: {
        alias: {
          "/@": pathResolve("src"),
        }
    },
    //引入第三方的配置,强制预构建插件包
    optimizeDeps: {
        include: ['axios'],
    },
     css: {
        preprocessorOptions: {
        scss: {
            charset: false, // 关闭编译时 字符编码 报错问题
            javascriptEnabled: true,
            additionalData: `@import "${resolve(
            __dirname,
            "src/assets/css/var.scss"
            )}";`,
        },
        },
    },
    json: { 
        //是否支持从 .json 文件中进行按名导入 
        namedExports: true,
        //若设置为 true 导入的json会被转为 export default JSON.parse("..") 会比转译成对象字面量性能更好 
        stringify:false, 
    },
    build: {
      target: 'modules',// 设置最终构建的浏览器兼容目标。modules:支持原生 ES 模块的浏览器
      outDir: 'dist',//指定输出路径
      assetsDir: 'assets',//指定生成静态资源的存放路径
      assetsInlineLimit: '4096', // 小于此阈值的导入或引用资源将内联为base64编码，设置为0可禁用此项。默认4096（4kb）
      cssCodeSplit: true, // 启用/禁用CSS代码拆分，如果禁用，整个项目的所有CSS将被提取到一个CSS文件中,默认true
      sourcemap: false, // 构建后是否生成 source map 文件
      minify: 'terser' // 混淆器，terser构建后文件体积更小
      write: true,   //设置为 false 来禁用将构建后的文件写入磁盘  
      emptyOutDir: true,  //默认情况下，若 outDir 在 root 目录下，则 Vite 会在构建时清空该目录。  
      brotliSize: true,  //启用/禁用 brotli 压缩大小报告 
      chunkSizeWarningLimit: 500,  //chunk 大小警告的限制 
      //去除 console debugger
      terserOptions: {   
            compress: { 
                drop_console: true,
                drop_debugger: true, 
            },
        }
    },
    //本地运行配置，及反向代理配置
    server: {
        host: 'localhost', // 指定服务器主机名
        port: 3000, // 指定服务器端口
        cors: true,// 默认启用并允许任何源
        open: true,//在服务器启动时自动在浏览器中打开应用程序
        strictPort: false, // 设为 false 时，若端口已被占用则会尝试下一个可用端口,而不是直接退出
        https: false, // 是否开启 https
        //反向代理配置，注意rewrite写法，防止踩坑
        proxy: {
          // 字符串简写写法 
          '/foo': 'http://192.168.xxx.xxx:xxxx', 
          // 选项写法
          '/api': {
              target: 'http://192.168.99.223:3000',   //代理接口
              changeOrigin: true,
              rewrite: (path) => path.replace(/^\/api/, '')
          }
        }
    }
});


```

**忽略.vue后缀**
在开发vue2的时候导入文件习惯性忽略.vue后缀的。但在Vite里会引起报错。

```javascript
import Home from '@/views/home' // error
import Home from '@/views/home.vue' // ok
```

```javascript
// vite.config.ts
import { defineConfig } from 'vite'
export default defineConfig({
  resolve: {
    extensions: ['.js', '.ts', '.jsx', '.tsx', '.json', '.vue']
  }
})
```

>这里要注意，手动配置extensions要记得把其他类型的文件后缀也加上，因为其他类型如js等文件**默认**是可以忽略后缀导入的，不写上的话其他类型文件的导入就变成需要加后缀了。虽然可以这么做，不过毕竟官方文档说了不建议忽略.vue后缀，所以建议大家在实际开发中还是老老实实写上.vue。

**按需导入`element-plus`**

```sh
#安装依赖
npm install -D unplugin-vue-components unplugin-auto-import
npm install -D vite-plugin-style-import
npm i -D unplugin-icons @iconify/json
```

`依赖简介`

- [unplugin-auto-import](https://github.com/antfu/unplugin-auto-import)：自动按需引入 vue\vue-router\pinia 等的 api
- [unplugin-vue-components](https://github.com/antfu/unplugin-vue-component)：自动按需引入 第三方的组件库组件 和 我们自定义的组件
- [unplugin-icons](https://github.com/antfu/unplugin-icons)：可以自动按需引入 所使用的图标，而不用手动 import
- [vite-plugin-style-import](https://github.com/vbenjs/vite-plugin-style-import)：自动按需引入 我们手动引入的组件的css样式

```javascript
///自动按需引入 vue\vue-router\pinia 等的 api
import AutoImport from 'unplugin-auto-import/vite'
//自动按需引入 第三方的组件库组件 和 我们自定义的组件
import Components from 'unplugin-vue-components/vite'
//自动按需引入 我们手动引入的组件的css样式
import styleImport from 'vite-plugin-style-import';
//可以自动按需引入我们所要使用的图标，而不用手动 import
import Icons from 'unplugin-icons/vite'
import IconsResolver from 'unplugin-icons/resolver'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
  plugins: [
    vue(),
    //按需导入组件
    AutoImport({
      resolvers: [ElementPlusResolver()],
    }),
    Components({
      resolvers: [ElementPlusResolver(),IconsResolver()],
    }), 
    Icons({
      compiler: 'vue3',
      autoInstall: true,
    }),
    //按需导入组件样式
    styleImport({
      libs: [{
        libraryName: 'element-plus',
        esModule: true,
        resolveStyle: (name) => {
          return `element-plus/theme-chalk/${name}.css`;
        },
      }]
    })
  ],
})
```

**启动优化**

>值得一提的是：如果你的 vite版本 > 2.9.0，那么就不需要安装启动优化插件

在使用这几个按需引入插件后，我们使用 Vite 开发项目时，第一次经常会看到以下的场景：
![](/img/postImg/Snipaste/Snipaste_2022-10-17_13-53-01.png)
这个耗时很久所以我们需要[vite-plugin-optimize-persist](!https://github.com/antfu/vite-plugin-optimize-persist)插件来解决这个问题

```sh
npm i -D vite-plugin-optimize-persist vite-plugin-package-config
```

```javascript
// vite.config.ts
import OptimizationPersist from 'vite-plugin-optimize-persist'
import PkgConfig from 'vite-plugin-package-config'

export default {
  plugins: [
    PkgConfig(),
    OptimizationPersist()
  ]
}
```

这样配置完成后，我们第一次使用按需引入后，它就会在 `package.json` 中生成内容（**生成的内容根据你项目所使用的组件而定**），下次进来就可以快速的跑起项目啦

**TypeScript 报红**

很多人第一次配置完之后，会发现 编辑器 给我们在按需引入的组件上报了红：
![](/img/postImg/Snipaste/Snipaste_2023-10-18_14-44-06.png)
这是因为我们生成的 `auto-imports.d.ts` 和 `components.d.ts`两个文件，默认是生成在项目根目录，正常我们配置的 TypeScript 解析的文件都放在 src 文件夹下，自然 TS 会报这个错啦；
`解决办法很简单：`

1. 在 `tsconfig.json` 中 `include` 这两个文件
2. 像下面这样配置，把两个 **.d.ts** 的生成目录放到 src 文件夹

```javascript
// vite.config.ts
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'

export default {
  plugins: [
    // ...
    AutoImport({
      dts: './src/auto-imports.d.ts',
    }),
    Components({
      dts: './src/components.d.ts'
    }),
  ],
}

```

**Eslint 报红**

如果我们的项目中使用了`Eslint`，且使用了 `unplugin-auto-import`，那么你就会发现下面这一幕
![](/img/postImg/Snipaste/Snipaste_2023-10-18_15-40-44.png)\
这是因为 `Eslint` 找不到我们按需引入的这些 `api`，我们不需要import是爽了，人家`Eslint`不知道呀，就咯的一下给我们报红，那怎么办呢？好办，既然 `Eslint` 不知道我们按需引入了 `api`，那我们让它知道不就好了？看代码：

```javascript
// vite.config.ts
import AutoImport from 'unplugin-auto-import/vite'

export default {
  plugins: [
    // ...
    AutoImport({
      // Generate corresponding .eslintrc-auto-import.json file.
      // eslint globals Docs - https://eslint.org/docs/user-guide/configuring/language-options#specifying-globals
      eslintrc: {
        enabled: true, // Default `false`
        filepath: './.eslintrc-auto-import.json', // Default `./.eslintrc-auto-import.json`
        globalsPropValue: true, // Default `true`, (true | false | 'readonly' | 'readable' | 'writable' | 'writeable')
      },
    }),
  ],
}

```

这段代码，会让我们在根目录下生成一个 `.eslintrc-auto-import.json`文件，里面都是我们所引入的所有 API 名称，接下来我们只需要在 .`eslintrc.js`中加载这个文件即可

```javascript
// .eslintrc.js

module.exports = { 
  /* ... */
  extends: [
    // ...
    './.eslintrc-auto-import.json',
  ],
}

```

>完整的使用实例可以看另一篇[文章](https://juejin.cn/post/7058201396113309703)
