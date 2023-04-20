---
title: nmp包源管理
date: 2022-09-20 17:29:09
cover: https://w.wallhaven.cc/full/m3/wallhaven-m3zjx1.jpg
tags:
 - npm
---

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
