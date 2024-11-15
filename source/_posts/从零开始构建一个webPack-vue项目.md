---
title: 从零开始构建一个webPack-vue项目
date: 2023-11-15 13:49:35
cover: 'https://cdn2.zzzmh.cn/wallpaper/origin/c9d3deb2880411ebb6edd017c2d2eca2.jpg/fhd?auth_key=1733328000-2e5684945cd96f15e41c2dbe594efe516e22fdf3-0-8835f1a51bc732711627ad46f06856be'
tags: 
- webpack
- vue
---
# 前言
为了加深我们对`webpack`的了解方便以后灵活运用`webpack`的技术,故以我们从零开始构建一个简单的`webpack-vue`项目.
# 一、项目结构
```
├─dist
├─node_modules
│  └─...
├─src
│  ├─components
│  ├─assets
│  ├─views
│  ├─App.vue
│  └─main.ts
├─public
│  └─index.html
├─.babelrc
├─.gitignore
├─package-lock.json
├─package.json
├─tsconfig.json
├─webpack.config.js
└─README.md
```
# 二、命令配置
```
# 配置项目文件
npm init -y
tsc --init
#如果tsc没有则可以安装
pnpm install typescript -D
# 安装依赖 
pnpm add webpack webpack-cli webpack-dev-server html-webpack-plugin typescript @vue/compiler-sfc friendly-errors-webpack-plugin -D
# 配置安装loader解析器
pnpm add ts-loader vue-loader style-loader css-loader sass-loader -D
pnpm add vue -S
```
**介绍依赖功能**
- `webpack` 打包工具
- `webpack-cli` 命令行工具
- `webpack-dev-server` 开发服务器
- `html-webpack-plugin` 生成html文件
- `clean-webpack-plugin` 清除之前的打包文件
- `ts-loader` 解析ts文件
- `vue-loader` 解析vue文件
- `style-loader` 解析css
- `css-loader` 解析css
- `sass-loader` 解析sass
- `@vue/compiler-sfc` 解析vue
- `friendly-errors-webpack-plugin` 美化错误信息

# 三、配置文件
## `webpack.config.js`
```js
const { Configuration } = require("webpack");
const { resolve } = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin"); //生成html文件
const { CleanWebpackPlugin } = require("clean-webpack-plugin"); //清除之前的打包文件
const { VueLoaderPlugin } = require("vue-loader/dist/index"); //处理vue文件
const FriendlyErrorsWebpackPlugin = require("friendly-errors-webpack-plugin");
/**
 * @type {Configuration} //配置智能提示
 */
const config = {
  mode: "development",
  entry: "./src/main.ts",
  output: {
    path: resolve(__dirname, "dist"),
    filename: "[hash].js",
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: "vue-loader",
      },
      {
        test: /\.ts$/, //解析ts
        loader: "ts-loader",
        options: {
          configFile: resolve(process.cwd(), "tsconfig.json"),
          appendTsSuffixTo: [/\.vue$/],
        },
      },
      {
        test: /\.css$/, //解析css
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.sass$/, //解析sass
        use: ["style-loader", "css-loader","sass-loader"],
      },
    ],
  },
  resolve: {
    alias: {
      "@": resolve(__dirname, "src"),
    },
    //extensions: [".ts", ".js", ".vue"], //自动解析扩展名
  },
  devServer: {
    // proxy: {},
    port: 8008,
    hot: true,
    open: true,
  },
  stats:"errors-only",
  plugins: [
    new HtmlWebpackPlugin({
      template: "./public/index.html", //以public/index.html为模板
    }),
    new CleanWebpackPlugin(),
    new VueLoaderPlugin(),
    new FriendlyErrorsWebpackPlugin(
        {
            compilationSuccessInfo:{ //美化样式
                messages:['You application is running here http://localhost:8008']
            }
        }
    )
  ],
  // 打包库文件时使用,压缩包体积，需要配合cdn使用
  //externals: {
  // vue: "Vue",
  //},
};
module.exports = config;
```
