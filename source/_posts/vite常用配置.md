---
title: vite常用配置
date: 2023-02-28 10:23:56
cover: '/img/background/backdrop1.png'
tags:
  - vite
---

# 前言

最近一直使用vite去开发vue,故需要记录一些基本配置

# resolve.alias

这是我最常用的的功能，因为写全相对路径太繁琐，就给常用的路径使用别名

```js
// vite.config.js
import { defineConfig } from 'vite'
import path from 'path'

export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src') // 路径别名
    }
  }
})
```

我们也可以使用插件，来自动给 src 和 src 下所有的文件夹定义路径别名：

```js
// vite.config.js
import { defineConfig } from 'vite'
import { ViteAliases } from './node_modules/vite-aliases' // 通过名称引入会报错，可能是插件问题

export default defineConfig({
  plugins: [
    ViteAliases()
  ]
})

/**
src -> @
  assets -> @assets
  components -> @components
  router -> @router
  stores -> @stores
  views -> @views
  ...
*/
```

# css.preprocessorOptions

`CSS` 预处理器的配置选项,列如配置`scss`全局变量

```js
// src/assets/styles/variables.scss
$injectedColor: orange;
$injectedFontSize: 16px;

// vite.config.js
import { defineConfig } from 'vite'
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        //additionalData: `$injectedColor: orange;` // 单个配置全局变量
        additionalData: `@import '/src/assets/styles/variables.scss';` // 引入全局变量文件
      }
    }
  }
})
```

这样在 `.scss` 文件或 `.vue` 文件中就可以使用这些变量了。

# css.postcss

PostCSS 也是用来处理 CSS 的，只不过它更像是一个工具箱，可以添加各种插件来处理 CSS。像浏览器样式兼容问题、浏览器适配等，都可以通过 PostCSS 来解决。
如移动端使用 `postcss-px-to-viewport` 对不同设备进行布局适配：

```js
npm install postcss-px-to-viewport -D

// vite.config.js
import { defineConfig } from 'vite'
import postcssPxToViewport from 'postcss-px-to-viewport'

export default defineConfig({
  css: {
    postcss: {
      plugins: [
        // viewport 布局适配
        postcssPxToViewport({
          viewportWidth: 375
        })
      ]
    }
  }
})

```

这样我们书写的 `px` 单位就会转为 `vw` 或 `vh` ，很轻松地解决了适配问题。
