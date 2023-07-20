---
title: nmp包源管理
date: 2022-09-20 17:29:09
cover: https://w.wallhaven.cc/full/m3/wallhaven-m3zjx1.jpg
tags:
 - npm
---

# 包管理
在学习的过程中，如果发现npm很慢，可以使用`nrm` 管理包源

```cmd
# 选择较低的版本安装（指定版本安装依赖）因为有版本高会报错
$ npm install -g nrm@1.1.0
# 确认已选择的镜像
$ nrm ls
# 使用taobao镜像或者cnpm
$ nrm use cnpm
```

当然在学习[小满](https://xiaoman.blog.csdn.net/?type=blog)的ts的视频的时候发现了他写的包源管理，当然安利一波

```cmd
# 选择较低的版本安装（指定版本安装依赖）因为有版本高会报错
$ npm install xmzs -g
# 查看已选择的镜像
$ mmp ls
# 使用taobao镜
$ mmp use cnpm
```

# 了解 npm、cnpm、yarn、pnpm

## npm
[npm](https://github.com/npm/cli) 是 *Node.js* 自带的包管理器，平时通过 `npm install` 命令来安装各种 npm 包（比如：`npm install vue-router` ），就是通过这个包管理器来安装的。
关于 npm 包下载镜像源的设置：

```cmd
# 查看下载源
npm config get registry
# 绑定下载源
npm config set registry https://registry.npmmirror.com
# 删除下载源
npm config rm registry
```

npm 的包的版本锁定文件是 `package-lock.json` ，如果有管理多人协作仓库的需求，可以根据实际情况把它添加至 .gitignore 文件，便于统一团队的包管理。

## cnpm

cnpm 是阿里巴巴推出的包管理工具，安装之后默认会使用 `https://registry.npmmirror.com` 这个镜像源。 它的安装命令和npm非常一致，通过 `cnpm install` 命令来安装（比如 `cnpm install vue-router`）。 在使用它之前，需要通过 npm 命令进行全局安装：

```cmd
npm install -g cnpm
# 或者
npm install -g cnpm --registry=https://registry.npmmirror.com
```

cnpm 不生成 版本锁定 lock 文件，也不会识别项目下的 lock 文件，所以还是推荐使用 npm 或者其他包管理工具，通过绑定镜像源的方式来管理项目的包。

## yarn
yarn 也是一个常用的包管理工具，和npm 十分相似npmjs上的包，也会同步到 [yarnpkg](https://yarnpkg.com/) 。 也是需要全局安装才可以使用：

```cmd
npm install -g yarn
```

但是安装命令上会有点不同，yarn是用 `yarn add` 代替 `npm install` ，用 yarn remove 代替`npm uninstall` ，例如：

```cmd
# 安装单个包
yarn add vue-router

# 安装全局包
yarn global add typescript

# 卸载包
yarn remove vue-router
```

而且在运行脚本的时候，可以直接用 `yarn` 来代替 `npm run` ，例如 `yarn dev` 相当于 `npm run dev` 。 升级的时候用 `yarn upgrade` 代替 `npm update`命令。 yarn 默认绑定的是 <https://registry.yarnpkg.com> 的下载源，如果包的下载速度太慢，也可以配置镜像源，但是命令有所差异：

```cmd
# 查看镜像源
yarn config get registry
# 绑定镜像源
yarn config set registry https://registry.npmmirror.com

# 删除镜像源（注意这里是 delete ）
yarn config delete registry
```

yarn 的 版本锁定文件是 `yarn.lock` ，如果有管理多人协作仓库的需求，可以根据实际情况把它添加至 `.gitignore` 文件，便于统一团队的包管理。

## pnpm
pnpm 是包管理工具的一个后起之秀，主要优点在于快速的、节省磁盘空间，如果你的包在一个项目中已经下载了，其它项目再用到这个包就不需要再次下载，而是通过软链接的方式关联。用法跟其他包管理器很相似，没有太多的学习成本， npm 和 yarn 的命令它都支持。 也是必须先全局安装它才可以使用：

```cmd
npm install -g pnpm
```

目前 pnpm 在开源社区的使用率越来越高，包括接触最多的 `Vue / Vite` 团队也在逐步迁移到 pnpm 来管理依赖。 pnpm 的下载源使用的是 npm ，所以如果要绑定镜像源，按照 npm 的方式处理就可以了。
