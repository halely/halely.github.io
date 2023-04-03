---
title: css提升长列表性能渲染
date: 2021-09-14 15:37:36
tags:
    - CSS
    - HTML
cover: "https://www.foodiesfeed.com/wp-content/uploads/2023/03/jelly-beans-all-over-the-table.jpg"
---

# 前言

在很多网页开发的时候，都会涉及到长列表，当数据量特别多的时候，渲染会非常慢，有时候甚至会卡死，这严重影响了用户的体验。

因此我们会经常采用`虚拟滚动`、`分页`、`上拉加载更多`等不同的方式来进行优化，这些方式的思想都是一样的，都是`只渲染可见区域，等用户需要时再加载更多的内容`。而以上的方式无论哪种，都需要写大量的js或者css逻辑去实现。
而现在，我们多了一种方式—— `content-visibility`。只需要一行CSS代码，就可以实现可见网页只加载可见区域内容，使网页的渲染性能得到数倍的提升！

# 介绍

`content-visibility` 是一个css属性，它控制一个元素是否呈现其内容，能让用户潜在地控制元素的呈现。用户可以使用它跳过元素的呈现(包括布局和绘制)，让页面的初始直到用户需要为止，渲染得到极大的提升。

__Value__

`content-visibility`属性有三个可选值:

- `visible`: 默认值。对布局和呈现不会产生什么影响。

- `hidden`: 元素跳过其内容的呈现。用户代理功能（例如，在页面中查找，按Tab键顺序导航等）不可访问已跳过的内容，也不能选择或聚焦。类似于对其内容设置了display: none属性。
- `auto`: 对于用户可见区域的元素，浏览器会正常渲染其内容；对于不可见区域的元素，浏览器会暂时跳过其内容的呈现，等到其处于用户可见区域

**效果对比**

__使用前__

代码
如下代码，在浏览器中简单的使用100个卡片，并对其设置扫光效果动画:

```html
<html>
  <head>
    <title>content-visibility</title>
    <style type="text/css">
      .card {
        position: relative;
        overflow: hidden;
        transition-duration: 0.3s;
        margin-bottom: 10px;
        width: 200px;
        height: 100px;
        background-color: #ffaa00;
      }
      .card:before {
        content: '';
        position: absolute;
        left: -665px;
        top: -460px;
        width: 300px;
        height: 15px;
        background-color: rgba(255, 255, 255, 0.5);
        transform: rotate(-45deg);
        animation: searchLights 2s ease-in 0s infinite;
      }
      @keyframes searchLights {
        0% {
        }
        75% {
          left: -100px;
          top: 0;
        }
        100% {
          left: 120px;
          top: 100px;
        }
      }
    </style>
  </head>
  <body>
    <div class="card"></div>
    <div class="card"></div>
    <!-- ... -->
    <!-- 此处省略97个<div class="card"></div> -->
    <!-- ... -->
    <div class="card"></div>
  </body>
</html>
```

**渲染效果**
从chrome可以看出，渲染时间花费了1454ms：

![渲染](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b082024f98064bbda674a9377a770362~tplv-k3u1fbpfcp-watermark.awebp)

**使用后**
__代码__

在class为card中添加 `content-visibility: auto;`：


```css

   .card {
        position: relative;
        overflow: hidden;
        transition-duration: 0.3s;
        margin-bottom: 10px;
        width: 200px;
        height: 100px;
        background-color: #ffaa00;
        content-visibility: auto;
      }
      .card:before {
        content: '';
        // ...
      }
```

**渲染效果**
可以明显的看到，使用`content-visibility: auto;`后渲染时间只需要381ms，性能提升了近4倍！而且随着元素内容变复杂，提升的性能会有更明显的上升。

![渲染](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2466446195ad42ca9dacb3dd48dd0fee~tplv-k3u1fbpfcp-watermark.awebp)

再从下图的`dom`结构变化中也可以看到，当card未出现在屏幕可见区域内是，其内容(::before等动画)在元素出现在可见效果时才出现：

![渲染](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09d6813f59124dab916762ee3084cb1b~tplv-k3u1fbpfcp-watermark.awebp)

# 缺陷

**兼容性**
`content-visibility`是`chrome85`今年新增的特性，所以目前兼容性还不高，但是相信兼容性的问题在不久的将来会得到解决。目前兼容性如下：

![兼容](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39d62c940c5540b292b78e5d3843276c~tplv-k3u1fbpfcp-watermark.awebp)

>目前的最新的版本是的chrome93

分元素导致浏览器渲染出问题

问题
当元素的部分内容如`<img />`标签这种，元素的高度是有图片内容决定的，因此在这种情况下，如果使用`content-visibility`，则可见视图外的`img`初始未渲染，高度为0，随着滚动条向下滑动，页面高度增加，会导致滚动条的滚动有问题。
如下代码：

```html

<html>
  <head>
    <title>content-visibility</title>
    <style type="text/css">
      .card {
        margin-bottom: 10px;
        content-visibility: auto;
      }
    </style>
  </head>
  <body>
    <div class="card">
      <img
        src="https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=1057266467,784420394&fm=26&gp=0.jpg"
        alt="小狗"
      />
      <!-- ... -->
      <!-- 此处省略n个card内容 -->
      <!-- ... -->
      <img
        src="https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=1057266467,784420394&fm=26&gp=0.jpg"
        alt="小狗"
      />
    </div>
  </body>
</html>
```

效果如下，可以看出滚动条随着图片的呈现，会出现问题：

![問題](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fdfa3b283d1c40e4a83115b278621eae~tplv-k3u1fbpfcp-watermark.awebp)

解决思路
解决此问题，如果在已知元素高度的情况下，可以使用`contains-intrinsic-size`属性，为上面的card添加：`contains-intrinsic-size：312px;`，这会给内容附一个初始高度值。（如果高度不固定也可以附一个大致的初始高度值，会使滚动条问题相对减少）。
修改代码如下:

```css
  .card {
    margin-bottom: 10px;
    content-visibility: auto;
    contain-intrinsic-size: 312px; // 添加此行
    }
```

再次看滚动条就没有问题了：

![問題](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/545c31be87fb48f0805cd16c28a8935e~tplv-k3u1fbpfcp-watermark.awebp)


# 总结

`content-visibility` 是一个非常实用的CSS属性，通过一行CSS可以代替虚拟滚动、上拉加载更多等多种长列表渲染优化方式。
虽然其兼容性现在不是很好，但是相信不久的将来这并不是问题。现在来看是部分场景下它对浏览器的滚动条影响问题，如果你的列表项高度相同，那么可以通过`contain-intrinsic-size`来设置一个初始高度解决。如果列表项高度不固定而又非常重视用户的滚动条体验，那么不建议使用此属性。当然了，这一css属性出来的时间并不是太长，虽然它的完善，这一问题或许在将来也能够得到解决。
