---
title: GitHub访问加速
date: 2024-10-16 14:32:57
copyright_author: 码海漫游者8
cover: 'https://picx.zhimg.com/v2-b54ec7e8730d9b69ad2ae52ff354ef87_r.jpg?source=12a79843'
copyright_author_href: https://blog.csdn.net/mahaimanyouzhe?type=blog
copyright_url: https://blog.csdn.net/mahaimanyouzhe/article/details/148039518
copyright_info: 本文章来自CSDN
tags:
 - GitHub
categories: 
  - 前端学习
---

# 前言


最近在技术交流群里看到不少小伙伴吐槽：“GitHub又双叒叕打不开了！”、“clone个仓库比蜗牛还慢…”（懂的都懂😭）。作为每天要和GitHub打交道的开发者，今天我就把自己多年积累的加速秘籍全盘托出，手把手教你突破网络限制！

# 一、为什么GitHub这么慢？（先搞懂原理）

## 1.1 网络延迟的罪魁祸首

GitHub服务器主要部署在北美地区，国内访问需要经过多个国际网络节点。根据我的实际测试（使用tracert命令），北京到GitHub的请求竟然要经过18个路由节点！！！

## 1.2 DNS污染问题

某些地区的[DNS解析](https://so.csdn.net/so/search?q=DNS%E8%A7%A3%E6%9E%90&spm=1001.2101.3001.7020)会被劫持，导致无法正确解析github.com的IP地址。试试这个命令：

```bash
nslookup github.com
```

如果返回的IP不是`20.205.243.166`这类官方地址，说明你的DNS被污染了！

# 二、5大加速方案实测对比（附详细步骤）

## 2.1 镜像站大法（新手首选）

**推荐指数：⭐⭐⭐⭐⭐**

国内维护的镜像站实测速度可达10MB/s+！常用镜像地址：

*   https://hub.yzuu.cf
*   https://gitclone.com
*   https://github.com.cnpmjs.org

**使用技巧**：直接把github.com替换成镜像域名即可。比如原地址：

```
git clone https://github.com/vuejs/vue.git
```

替换后：

```
git clone https://hub.yzuu.cf/vuejs/vue.git
```

## 2.2 修改Hosts文件（永久生效）

**推荐指数：⭐⭐⭐⭐**

1.  打开[IP查询网站](https://www.ipaddress.com)
2.  查询以下域名的IP：
    *   github.com
    *   assets-cdn.github.com
    *   github.global.ssl.fastly.net
3.  编辑hosts文件（路径：C:\\Windows\\System32\\drivers\\etc\\hosts）
4.  添加记录（示例）：

```
20.205.243.166 github.com
185.199.108.153 assets-cdn.github.com
199.232.69.194 github.global.ssl.fastly.net
```

## 2.3 Git配置代理（程序员必备）

**推荐指数：⭐⭐⭐⭐⭐**

如果你有科学上网工具，可以设置git代理：

```bash
# Socks5代理
git config --global http.proxy socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080

# HTTP代理
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
```

## 2.4 使用Gitee中转（适合大项目）

**推荐指数：⭐⭐⭐**

1.  在Gitee导入GitHub仓库
2.  从Gitee克隆仓库
3.  修改remote地址指向原始GitHub仓库：

```bash
git remote set-url origin https://github.com/原仓库地址.git
```

## 2.5 终极方案：GitHub加速器（黑科技）

**推荐指数：⭐⭐⭐⭐**

推荐几个开源加速工具：

*   [FastGit](https://doc.fastgit.org)（基于镜像的加速服务）
*   [dev-sidecar](https://github.com/docmirror/dev-sidecar)（开发者边车工具）
*   [Steam++](https://steampp.net)（多平台加速工具）

以dev-sidecar为例：

1.  下载对应系统的客户端
2.  开启GitHub加速模式
3.  访问速度立竿见影！

# 三、避坑指南（血泪经验）

## 3.1 不要用盗版加速器！

最近发现有些"加速器"会注入恶意代码（亲身中招过😱），建议使用开源方案或知名工具。

## 3.2 SSH连接比HTTPS更快

把仓库地址从https改为ssh协议，速度能提升30%以上：

```bash
git remote set-url origin git@github.com:user/repo.git
```

## 3.3 大文件用Git LFS

如果仓库包含大文件，一定要配置Git LFS：

```bash
git lfs install
git lfs track "*.psd"
```

#四、速度测试对比（单位：MB/s）

| 方法 | 白天速度 | 晚上速度 |
| --- | --- | --- |
| 直连 | 0.12 | 0.05 |
| 镜像站 | 8.76 | 6.32 |
| Hosts修改 | 2.45 | 1.89 |
| 代理 | 12.34 | 10.21 |
| 加速器 | 9.87 | 8.65 |

# 五、总结与推荐

*   **个人用户**：镜像站+SSH协议是最佳组合
*   **团队开发**：自建GitLab+GitHub镜像同步
*   **科研机构**：建议使用学术加速通道

