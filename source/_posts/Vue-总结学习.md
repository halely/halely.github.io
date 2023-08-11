---
title: Vue 总结学习
date: 2023-04-03 16:58:07
cover: https://www.foodiesfeed.com/wp-content/uploads/2023/06/vegan-curry-with-fresh-herbs.jpg
tags:
 - Vue
---

# 前言

为了方便自己开发做的总结

# 总结

## 插槽slot

**匿名插槽**

```html
<!-- 子组件 -->
<template>
    <div><slot></slot></div>
</template>
<!-- 父使用组件 -->
 <Dialog>
    <template v-slot>
        <div>2132</div>
    </template>
</Dialog
```

**具名插槽**
具名插槽其实就是给插槽取个名字。一个子组件可以放多个插槽，而且可以放在不同的地方，而父组件填充内容时，可以根据这个名字把内容填充到对应插槽中

```html
<!-- 子组件 -->
<div>
    <slot name="header"></slot>
    <slot></slot>
    <slot name="footer"></slot>
</div>
<!-- 父使用组件 -->
<Dialog>
    <template v-slot:header>
        <div>1</div>
    </template>
    <template v-slot>
        <div>2</div>
    </template>
    <template #footer>
        <div>3</div>
    </template>
</Dialog>

```

**作用域插槽**

```html
<!-- 子组件 -->
<div>
    <slot name="header"></slot>
    <div>
        <div v-for="item in 100">
            <slot :data="item"></slot>
        </div>
    </div>
    <slot name="footer"></slot>
</div>
<!-- 父使用组件 -->
<Dialog>
    <template #header>
        <div>1</div>
    </template>
    <template #default="{ data }">
        <div>{{ data }}</div>
    </template>
    <template #footer>
        <div>3</div>
    </template>
</Dialog>
```

**动态插槽**
插槽可以是一个变量名

```html
 <Dialog>
    <template #[name]>
        <div>
            23
        </div>
    </template>
</Dialog>
```

```js
const name = ref('header')
```

## 异步组件

在大型应用中，我们可能需要将应用分割成小一些的代码块 并且减少主包的体积
这时候就可以使用异步组件

**顶层 await**
在`setup`语法糖里面 使用方法
`<script setup>` 中可以使用顶层`await`。结果代码会被编译成 `async setup()`

```html
<script setup>
const post = await fetch(`/api/post/1`).then(r => r.json())
</script>
```

父组件引用子组件 通过`defineAsyncComponent`加载异步配合`import`函数模式便可以**分包**

```html
<script setup lang="ts">
import { reactive, ref, markRaw, toRaw, defineAsyncComponent } from 'vue'
const Dialog = defineAsyncComponent(() => import('../../components/Dialog/index.vue'))
//完整写法
const AsyncComp = defineAsyncComponent({
  // 加载函数
  loader: () => import('./Foo.vue'),
  // 加载异步组件时使用的组件
  loadingComponent: LoadingComponent,
  // 展示加载组件前的延迟时间，默认为 200ms
  delay: 200,
  // 加载失败后展示的组件
  errorComponent: ErrorComponent,
  // 如果提供了一个 timeout 时间限制，并超时了
  // 也会显示这里配置的报错组件，默认值是：Infinity
  timeout: 3000
})
```

**suspense**

`<suspense>` 组件有两个插槽。它们都只接收一个直接子节点。default 插槽里的节点会尽可能展示出来。如果不能，则展示 fallback 插槽里的节点。

```html
<Suspense>
    <template #default>
        <Dialog>
            <template #default>
                <div>我在哪儿</div>
            </template>
        </Dialog>
    </template>
    <template #fallback>
        <div>loading...</div>
    </template>
</Suspense>
```

## Teleport传送组件

`Teleport` Vue 3.0新特性之一。
`Teleport` 是一种能够将我们的模板渲染至指定DOM节点，不受父级style、v-show等属性影响，但data、prop数据依旧能够共用的技术；类似于 React 的 Portal。
**使用方法**
通过`to`属性 插入指定元素位置 to="body" 便可以将Teleport 内容传送到指定位置

```html
<Teleport to="body" :disabled='true'>
    <Loading></Loading>
</Teleport>
```

> `disabled` 属性默认是`false`,如果设置为`true`，则不再传送,也可以自定义传送位置 支持 class id等 选择器

感觉很好用，假如自己写弹窗或者其他什么的

## transition 动画

对单个组件使用`transition`
对多个list使用`transition-group`

通过自定义 class 结合css动画库[animate css](https://animate.style/)会实现一下非常好的效果

```html
<transition
        leave-active-class="animate__animated animate__bounceInLeft"
        enter-active-class="animate__animated animate__bounceInRight"
    >
        <div v-if="flag" class="box"></div>
</transition>
```

当你想使用动画生命周期的时候结合`gsap`动画库使用`GreenSock`

```js
const beforeEnter = (el: Element) => {
    console.log('进入之前from', el);
}
const Enter = (el: Element,done:Function) => {
    console.log('过度曲线');
    setTimeout(()=>{
       done()
    },3000)
}
const AfterEnter = (el: Element) => {
    console.log('to');
```

实现数字过渡小状态代码

```html
<template>
  <div>
    <input step="20" v-model="num.current" type="number" />
    <div>{{ num.tweenedNumber.toFixed(0) }}</div>
  </div>
</template>
<script setup lang='ts'>
import { reactive, watch } from 'vue'
import gsap from 'gsap'
const num = reactive({
  tweenedNumber: 0,
  current: 0
})
watch(() => num.current, (newVal) => {
  gsap.to(num, {
    duration: 1,
    tweenedNumber: newVal
  })
})
</script>
```

## 依赖注入Provide / Inject

在vue3中可以在祖先组件中设置`provide`

```js
import { provide, ref,readonly } from 'vue'
let flag = ref<number>(1)
provide('flag',ref(flag))
//上面的值可以被子孙更改的
//如果不想被更改的话可以设置readonly
```

在子孙组件中可以通过inject获取,且可以更改，也是响应式的

```js
import { inject, Ref, ref } from 'vue'
//后面的ref(1)为设置默认值
const flag = inject<Ref<number>>('flag', ref(1))
const change = () => {
    flag.value = 2
}
```

在v3中可以设置v-bind响应式绑定注册的值

```html
<script setup lang='ts'>
   import {ref } from 'vue'
   const color=ref('red')
</script>
<style scope>
    .box{
        background-color:v-bing(color);
    }
</style>
```

## 自动引入插件

在开发vue3的过程中，我们每次开发页面都需要引入ref reactive
有一个插件可以帮我们自动引入这些
[unplugin-auto-import/vite](https://github.com/antfu/unplugin-auto-import)

如果访问慢可以使用[加速](https://gitcode.net/mirrors/antfu/unplugin-auto-import?utm_source=csdn_github_accelerator)

安装

```
npm i -D unplugin-auto-import
```

```js
// vite.config.ts，因为我们使用的是vite版本
import AutoImport from 'unplugin-auto-import/vite'

export default defineConfig({
  plugins: [
    AutoImport({
        imports:['vue'],
        dts: 'src/auto-imports.d.ts',//公共引入地址
    }),
  ],
})
```

在后面的开发中就可以每次都引入vue的一些api

## 自定义指令vue2 和vue3 总结

**directive-自定义指令（属于破坏性更新）**

1. Vue3指令的钩子函数
   - created 元素初始化的时候
   - beforeMount 指令绑定到元素后调用 只调用一次
   - mounted 元素插入父级dom调用
   - beforeUpdate 元素被更新之前调用
   - update 这个周期方法被移除 改用updated
   - beforeUnmount 在元素被移除前调用
   - unmounted 指令被移除后调用 只调用一次
2. Vue2 指令
   - 指令绑定到元素时（bind）
   - 元素插入时（inserted）
   - 组件更新时（update）
   - 组件更新后（componentUpdated）
   - 指令与元素解绑时（unbind）

指令参数

1. `el：`指令所绑定的元素，可以用来直接操作 DOM。
2. `binding`一个包含指令信息的对象
   - `name`：指令名，不包括 v- 前缀。
   - `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 2。
   - `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
   - `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"` 中，表达式为 "1 + 1"。
   - `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 "foo"。
   - `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 { foo: true, bar: true }。
