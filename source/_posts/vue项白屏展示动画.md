---
title: vue项白屏展示动画
date: 2022-02-21 17:33:20
cover: 'https://w.wallhaven.cc/full/y8/wallhaven-y8622k.jpg'
tags:
- Vue
categories: 
  - 前端学习
---

# 前言

在我开发vue项目的时候，在加载一些静态资源的时候总是有白屏出现，后来就发现在加载的时候做一个预加载的动画，记录一下。代码入下

```css
.first-loading-wrap {
          display: flex;
          width: 100%;
          height: 100vh;
          justify-content: center;
          align-items: center;
          flex-direction: column;
        }
    
        .first-loading-wrap > h1 {
          font-size: 128px
        }
    
        .first-loading-wrap .loading-wrap {
          padding: 98px;
          display: flex;
          justify-content: center;
          align-items: center
        }
    
        .dot {
          animation: antRotate 1.2s infinite linear;
          transform: rotate(45deg);
          position: relative;
          display: inline-block;
          font-size: 32px;
          width: 32px;
          height: 32px;
          box-sizing: border-box
        }
    
        .dot i {
          width: 14px;
          height: 14px;
          position: absolute;
          display: block;
          background-color: #1890ff;
          border-radius: 100%;
          transform: scale(.75);
          transform-origin: 50% 50%;
          opacity: .3;
          animation: antSpinMove 1s infinite linear alternate
        }
    
        .dot i:nth-child(1) {
          top: 0;
          left: 0
        }
    
        .dot i:nth-child(2) {
          top: 0;
          right: 0;
          -webkit-animation-delay: .4s;
          animation-delay: .4s
        }
    
        .dot i:nth-child(3) {
          right: 0;
          bottom: 0;
          -webkit-animation-delay: .8s;
          animation-delay: .8s
        }
    
        .dot i:nth-child(4) {
          bottom: 0;
          left: 0;
          -webkit-animation-delay: 1.2s;
          animation-delay: 1.2s
        }
    
        @keyframes antRotate {
          to {
            -webkit-transform: rotate(405deg);
            transform: rotate(405deg)
          }
        }
    
        @-webkit-keyframes antRotate {
          to {
            -webkit-transform: rotate(405deg);
            transform: rotate(405deg)
          }
        }
    
        @keyframes antSpinMove {
          to {
            opacity: 1
          }
        }
    
        @-webkit-keyframes antSpinMove {
          to {
            opacity: 1
          }
        }
```

```html
<div class="first-loading-wrap">
    <div class="loading-wrap">
        <span class="dot dot-spin"><i></i><i></i><i></i><i></i></span>
    </div>
</div>

```

这些代码需要在`<div id='app'></div>` 中加入，这样在vue资源生效之前就可以呈现一个加载动画，加载好了后这样就会被`vue`生成在`dom`覆盖
