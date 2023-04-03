---
title: vue虚拟Demo和diff算法
date: 2022-12-14 16:29:12
cover: 'https://www.foodiesfeed.com/wp-content/uploads/2017/03/choosing-from-a-menu.jpg'
tags:
  - Vue
---

# 虚拟DOM

虚拟DOM就是通过JS去生成一个AST抽象语法节点树。AST是使用非常广泛，比如TS转测成JS的时候他也会进行AST转换，还有我们常用的babel插件，用于es6转换es5，也是通过AST进行转换，甚至我们的js通过v8引擎去转换字解码的时候也用到了AST

## 为什么要使用虚拟DOM

在和原生DOM选择中，这在之前的学习尤大的知乎回答的时候就已经有过回答，这是一个`性能` VS `可维护性`的取舍。
首先没有任何框架可以比纯手动的优化 DOM 操作更快,因为框架的 DOM 操作层需要应对任何上层 API 可能产生的操作，它的实现必须是普适的。首先，看下面的例子

```js

let div = document.createElement('div')
let str = ''
for (const key in div) {
  str += key + ''
}
console.log(str)
//aligntitlelangtranslatedirhiddenaccessKeydraggablespellcheckautocapitalizecontentEditableisContentEditableinputModeoffsetParentoffsetTopoffsetLeftoffsetWidthoffsetHeightstyleinnerTextouterTextonbeforexrselectonabortonbluroncanceloncanplayoncanplaythroughonchangeonclickoncloseoncontextmenuoncuechangeondblclickondragondragendondragenterondragleaveondragoverondragstartondropondurationchangeonemptiedonendedonerroronfocusonformdataoninputoninvalidonkeydownonkeypressonkeyuponloadonloadeddataonloadedmetadataonloadstartonmousedownonmouseenteronmouseleaveonmousemoveonmouseoutonmouseoveronmouseuponmousewheelonpauseonplayonplayingonprogressonratechangeonresetonresizeonscrollonsecuritypolicyviolationonseekedonseekingonselectonslotchangeonstalledonsubmitonsuspendontimeupdateontoggleonvolumechangeonwaitingonwebkitanimationendonwebkitanimationiterationonwebkitanimationstartonwebkittransitionendonwheelonauxclickongotpointercaptureonlostpointercaptureonpointerdownonpointermoveonpointeruponpointercancelonpointeroveronpointeroutonpointerenteronpointerleaveonselectstartonselectionchangeonanimationendonanimationiterationonanimationstartontransitionrunontransitionstartontransitionendontransitioncanceloncopyoncutonpastedatasetnonceautofocustabIndexattachInternalsblurclickfocusenterKeyHintvirtualKeyboardPolicyonpointerrawupdatenamespaceURIprefixlocalNametagNameidclassNameclassListslotattributesshadowRootpartassignedSlotinnerHTMLouterHTMLscrollTopscrollLeftscrollWidthscrollHeightclientTopclientLeftclientWidthclientHeightattributeStyleMaponbeforecopyonbeforecutonbeforepasteonsearchelementTimingonfullscreenchangeonfullscreenerroronwebkitfullscreenchangeonwebkitfullscreenerrorchildrenfirstElementChildlastElementChildchildElementCountpreviousElementSiblingnextElementSiblingafteranimateappendattachShadowbeforeclosestcomputedStyleMapgetAttributegetAttributeNSgetAttributeNamesgetAttributeNodegetAttributeNodeNSgetBoundingClientRectgetClientRectsgetElementsByClassNamegetElementsByTagNamegetElementsByTagNameNSgetInnerHTMLhasAttributehasAttributeNShasAttributeshasPointerCaptureinsertAdjacentElementinsertAdjacentHTMLinsertAdjacentTextmatchesprependquerySelectorquerySelectorAllreleasePointerCaptureremoveremoveAttributeremoveAttributeNSremoveAttributeNodereplaceChildrenreplaceWithrequestFullscreenrequestPointerLockscrollscrollByscrollIntoViewscrollIntoViewIfNeededscrollTosetAttributesetAttributeNSsetAttributeNodesetAttributeNodeNSsetPointerCapturetoggleAttributewebkitMatchesSelectorwebkitRequestFullScreenwebkitRequestFullscreenariaAtomicariaAutoCompleteariaBusyariaCheckedariaColCountariaColIndexariaColSpanariaCurrentariaDescriptionariaDisabledariaExpandedariaHasPopupariaHiddenariaKeyShortcutsariaLabelariaLevelariaLiveariaModalariaMultiLineariaMultiSelectableariaOrientationariaPlaceholderariaPosInSetariaPressedariaReadOnlyariaRelevantariaRequiredariaRoleDescriptionariaRowCountariaRowIndexariaRowSpanariaSelectedariaSetSizeariaSortariaValueMaxariaValueMinariaValueNowariaValueTextgetAnimationsnodeTypenodeNamebaseURIisConnectedownerDocumentparentNodeparentElementchildNodesfirstChildlastChildpreviousSiblingnextSiblingnodeValuetextContentELEMENT_NODEATTRIBUTE_NODETEXT_NODECDATA_SECTION_NODEENTITY_REFERENCE_NODEENTITY_NODEPROCESSING_INSTRUCTION_NODECOMMENT_NODEDOCUMENT_NODEDOCUMENT_TYPE_NODEDOCUMENT_FRAGMENT_NODENOTATION_NODEDOCUMENT_POSITION_DISCONNECTEDDOCUMENT_POSITION_PRECEDINGDOCUMENT_POSITION_FOLLOWINGDOCUMENT_POSITION_CONTAINSDOCUMENT_POSITION_CONTAINED_BYDOCUMENT_POSITION_IMPLEMENTATION_SPECIFICappendChildcloneNodecompareDocumentPositioncontainsgetRootNodehasChildNodesinsertBeforeisDefaultNamespaceisEqualNodeisSameNodelookupNamespaceURIlookupPrefixnormalizeremoveChildreplaceChildaddEventListenerdispatchEventremoveEventListener

```

你会发现，原生的DOM属性非常多，如果我们直接维护，那么是非常浪费性能的，但是虚拟DOM可以直接通过JS计算去操作，这是非常快的，重绘的性能也是可以接收的。所以选择的虚拟DOM

# 介绍Diff算法

> 注意：当前版本是Vue3.2的diff算法

Diff算法其实是仪征`对比方法`,核心就是`旧 DOM 组`更新为`新 DOM 组`时，如何更新才能效率更高。
>很多人以为更新了DOM就一定使用diff,但是想触发diff那么一定有DOM发生了变化，这两者是不同的。

**有key和无key的算法区别**

我们在遍历渲染DOM的时候，一般来说大家都知道写key，那么为什么写key呢，有什么区别呢
在vue源码`renderer`源码中，vue分为种，一种是没有key一种是有key(也有可能是个别有key和无key混杂在一起)的渲染

## 无key的渲染

在`patchUnkeyedChildren`方法中我们可以看到

```ts
  const patchUnkeyedChildren = (
    c1: VNode[],
    c2: VNodeArrayChildren,
    container: RendererElement,
    anchor: RendererNode | null,
    parentComponent: ComponentInternalInstance | null,
    parentSuspense: SuspenseBoundary | null,
    isSVG: boolean,
    slotScopeIds: string[] | null,
    optimized: boolean
  ) => {
    c1 = c1 || EMPTY_ARR
    c2 = c2 || EMPTY_ARR
    const oldLength = c1.length
    const newLength = c2.length
    const commonLength = Math.min(oldLength, newLength)
    let i
    //新的替换旧的
    for (i = 0; i < commonLength; i++) {
      const nextChild = (c2[i] = optimized? cloneIfMounted(c2[i] as VNode): normalizeVNode(c2[i]))
      patch(c1[i],nextChild,container,null,parentComponent,parentSuspense,isSVG,slotScopeIds,optimized)
    }
    if (oldLength > newLength) {
      // 删除旧的
      unmountChildren(c1,parentComponent,parentSuspense,true,false,commonLength)
    } else {
      // 添加新的
      mountChildren(c2,container,anchor,parentComponent,parentSuspense,isSVG,slotScopeIds,optimized,commonLength)
    }
  }
```

其实以上可以看到，无key的时候，渲染很简单，就是直接用新Node渲染替换对应索引位置的旧Node，然后对比新旧数量，多了就执行删除，少了就执行添加。

## 有key

其实在这个阶段，我们最中心的逻辑就是一个:**同一个组件，如果没有改变，那么就不再重新渲染计算**
一切都是为了优化来设计逻辑。

那么首先程序是如果能判断出一个DOM有没有变化呢？

这就是源码中 `isSameVNodeType` 简化后代码

```js
/**
 * 根据 key || type 判断是否为相同类型节点
 */
export function isSameVNodeType(n1: VNode, n2: VNode): boolean {
    return n1.type === n2.type && n1.key === n2.key
}
```

上面的判断方式是如果两个`vnode` 的`type`、`key`相等,则两个 `vnode` 为相同的 `vnode`

- `type`:`VNode` 的节点类型,列如:`li`、`div`等等
- `key` : 就是你绑定在DOM上的key值，其实也就是这个DOM的标识

当然也有其他逻辑，但是主要就是判断这两个，如果返回`true`，那么就不需要更新

那么后面的diff算法是执行的呢

![img](/img/postImg/patchKeyedChildren.png)

从上面的截图中，可以很清楚的看到整个`diff`的分步共分为 5 步，分别为：

1. `sync from start`：自前向后的对比
2. `sync from end`：自后向前的对比
3. `common sequence + mount`：新节点多于旧节点，需要挂载
4. `common sequence + unmount`：旧节点多于新节点，需要卸载
5. `unknown sequence`：乱序

### 第一步：自前向后的对比

>谨记： `diff` 场景一定是两组 `dom` 的对比

`diff` 的第一步主要是进行：两组 `dom` 的自前向后对比。 其核心的目的是：把两组 `dom` 自前开始，相同的 `dom` 节点（vnode）完成对比处理

下面我们把源码添加了详细备注（进行了适当简化）：

```js
  const patchKeyedChildren = (
    oldChildren,
    newChildren,
    container,
    parentAnchor
  ) => {
    /**
     * 索引
     */
    let i = 0
    /**
     * 新的子节点的长度
     */
    const newChildrenLength = newChildren.length
    /**
     * 旧的子节点最大（最后一个）下标
     */
    let oldChildrenEnd = oldChildren.length - 1
    /**
     * 新的子节点最大（最后一个）下标
     */
    let newChildrenEnd = newChildrenLength - 1

    // 1. 自前向后的 diff 对比。经过该循环之后，从前开始的相同 vnode 将被处理
    while (i <= oldChildrenEnd && i <= newChildrenEnd) {
      const oldVNode = oldChildren[i]
      const newVNode = normalizeVNode(newChildren[i])
      // 如果 oldVNode 和 newVNode 被认为是同一个 vnode，则直接 patch 即可
      if (isSameVNodeType(oldVNode, newVNode)) {
        patch(oldVNode, newVNode, container, null)
      }
      // 如果不被认为是同一个 vnode，则直接跳出循环
      else {
        break
      }
      // 下标自增
      i++
    }
```

在上面的代码中，主要进行了两大步的处理逻辑

1. 自前向后的 `diff` 比对中，会 依次获取相同下标的 `oldChild` 和 `newChild`
   - 如果 oldChild 和 newChild 为 相同的 VNode，则直接通过 patch 进行打补丁即可
   - 如果 oldChild 和 newChild 为 不相同的 VNode，则会跳出循环
2. 每次处理成功，则会自增 i 标记，表示：自前向后已处理过的节点数量

>总结：从前开始比较，相同的vnode 直接渲染，直到遇到不同的 vnode 为止

### 第二步：sync from end：自后向前的对比

和第一步很类似，但是是反过来的。即：两组 dom 的自后向前对比。 其核心的目的是：把两组 dom 自后开始，相同的 dom 节点（vnode）完成对比处理

```js
// 2. 自后向前的 diff 对比。经过该循环之后，从后开始的相同 vnode 将被处理
while (i <= oldChildrenEnd && i <= newChildrenEnd) {
  const oldVNode = oldChildren[oldChildrenEnd]
  const newVNode = normalizeVNode(newChildren[newChildrenEnd])
  if (isSameVNodeType(oldVNode, newVNode)) {
    patch(oldVNode, newVNode, container, null)
  } else {
    break
  }
  // 最后的下标递减
  oldChildrenEnd--
  newChildrenEnd--
}
```

>总结：从后开始比较，相同的vnode 直接渲染，直到遇到不同的 vnode 为止

### 新节点多于旧节点，需要挂载

经过第一步和第二步的处理，就要对剩下的数据进行一些数量对比了，
比如剩下的新旧节点数量不一致。

1. 新节点的数量多于旧节点的数量如：`arr.push(item)`
2. 旧节点的数量多于新节点的数量如：`arr.pop(item)`

那么这里咱们先来看**新节点多于旧节点**这种情况。

- 执行`arr.push()`：这样可以把新数据添加到尾部。即：多出的新节点位于 尾部
- 执行`arr.unshift()`：这样可以把新数据添加到头部。即：多出的新节点位于 头部

为了方便理解，源码也给出了实例

```js
//注意，这是经过之前两步计算的 
// oldChildrenEnd 旧的子节点最大（最后一个）下标
// newChildrenEnd 新的子节点最大（最后一个）下标
// (a b)
// (a b) c
// i = 2, oldChildrenEnd = 1, newChildrenEnd = 2
// (a b)
// c (a b)
// i = 0, oldChildrenEnd = -1, newChildrenEnd = 0

// 3. 新节点多余旧节点时的 diff 比对。
// i > oldChildrenEnd 就是旧的比较渲染完了
if (i > oldChildrenEnd) {
    // i <= newChildrenEnd 就是新的还有
    if (i <= newChildrenEnd) {
        const nextPos = newChildrenEnd + 1//剩余新节点的长度
        // 重点：找到锚点
        const anchor =
                nextPos < newChildrenLength ? newChildren[nextPos].el : parentAnchor
        while (i <= newChildrenEnd) {
                patch(null, normalizeVNode(newChildren[i]), container, anchor)
                i++
        }
    }
}
```

>总结：1. 首先要确定旧的已经比较完了，假如新的大于旧的;2.确定锚点，把剩下的插入头部或者尾部

### 第四步：旧节点多于新节点，需要卸载

第四步就是**旧节点多于新节点时**
这和第三步也是非常非常相似的，其实就是确定锚点，然后在头部或者尾部执行删除就好

```js
// (a b) c
// (a b)
// i = 2, oldChildrenEnd = 2, newChildrenEnd = 1
// a (b c)
// (b c)
// i = 0, oldChildrenEnd = 0, newChildrenEnd = -1

// 4. 旧节点多与新节点时的 diff 比对。
// 意思就是把新的已经比较完了，剩下的旧的不就都是多余的，执行删除就好了
else if (i > newChildrenEnd) {
    while (i <= oldChildrenEnd) {
        // 卸载 dom
        unmount(oldChildren[i])
        i++
    }
}
```

### 第五步：乱序

总算到最后一个阶段了，其实之前的步骤都是处理一些简单的情况，其实可以看成一个部分，理解起来也相对简单。

在这个章节我们需要理解一个重要的概念：**最长递增子序列**
>在 [维基百科](https://zh.m.wikipedia.org/zh-hans/%E6%9C%80%E9%95%BF%E9%80%92%E5%A2%9E%E5%AD%90%E5%BA%8F%E5%88%97) 定义的最长递增子序列
在一个给定的数值序列中，找到一个子序列，使得这个子序列元素的数值依次递增，并且这个子序列的长度尽可能地大。
切记序列**不一定是连续的**

那么最长递增子序列在`diff`中的作用到底是什么呢？

其实通过之前的四种场景可知，`diff`就是对一组节点进行`添加、删除、补丁`的操作，那么其实还有这一步就是**移动**,

那么怎么通过算法来优化移动呢
现在我们假设有两组数组

```

旧节点：1,2,3,4,5,6
新节点：1,3,2,4,6,5
```

那么根据这个新节点我们可以 生成`递增子序列（非最长）（注意：并不是惟一的）` ，其结果为：

1. 1、3、6
2. 1、2、4、6
3. ...

那么接下来，我们来分析一下移动的策略，整个移动根据递增子序列的不同，将拥有两种移动策略：

- 在第一种情况下，`1、3、6` 递增已确认，所以它们三个是不需要移动的，那么我们所需要移动的节点无非就是 三 个`2、4、5` 。
- 在第一种情况下，`1、2、4、6` 递增已确认，所以它们四个是不需要移动的，那么我们所需要移动的节点无非就是 三 个`3、5` 。
- 等等

由此我们可以得出，**最长递增子序列的确定，可以帮助我们减少移动的次数**

下面就是Vue3使用的 [维基百科(贪心+二分查找法)](https://zh.m.wikipedia.org/zh-hans/%E6%9C%80%E9%95%BF%E9%80%92%E5%A2%9E%E5%AD%90%E5%BA%8F%E5%88%97) `getSequence`

```js
/**
 * 获取最长递增子序列下标
 * 维基百科：https://en.wikipedia.org/wiki/Longest_increasing_subsequence
 * 百度百科：https://baike.baidu.com/item/%E6%9C%80%E9%95%BF%E9%80%92%E5%A2%9E%E5%AD%90%E5%BA%8F%E5%88%97/22828111
 */
 function getSequence(arr) {
  // 获取一个数组浅拷贝。注意 p 的元素改变并不会影响 arr
  // p 是一个最终的回溯数组，它会在最终的 result 回溯中被使用
  // 它会在每次 result 发生变化时，记录 result 更新前最后一个索引的值
  const p = arr.slice()
  // 定义返回值（最长递增子序列下标），因为下标从 0 开始，所以它的初始值为 0
  const result = [0]
  let i, j, u, v, c
  // 当前数组的长度
  const len = arr.length
  // 对数组中所有的元素进行 for 循环处理，i = 下标
  for (i = 0; i < len; i++) {
    // 根据下标获取当前对应元素
    const arrI = arr[i]
    // 
    if (arrI !== 0) {
      // 获取 result 中的最后一个元素，即：当前 result 中保存的最大值的下标
      j = result[result.length - 1]
      // arr[j] = 当前 result 中所保存的最大值
      // arrI = 当前值
      // 如果 arr[j] < arrI 。那么就证明，当前存在更大的序列，那么该下标就需要被放入到 result 的最后位置
      if (arr[j] < arrI) {
        p[i] = j
        // 把当前的下标 i 放入到 result 的最后位置
        result.push(i)
        continue
      }
      // 不满足 arr[j] < arrI 的条件，就证明目前 result 中的最后位置保存着更大的数值的下标。
      // 但是这个下标并不一定是一个递增的序列，比如： [1, 3] 和 [1, 2] 
      // 所以我们还需要确定当前的序列是递增的。
      // 计算方式就是通过：二分查找来进行的

      // 初始下标
      u = 0
      // 最终下标
      v = result.length - 1
      // 只有初始下标 < 最终下标时才需要计算
      while (u < v) {
        // (u + v) 转化为 32 位 2 进制，右移 1 位 === 取中间位置（向下取整）例如：8 >> 1 = 4;  9 >> 1 = 4; 5 >> 1 = 2
        // https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Right_shift
        // c 表示中间位。即：初始下标 + 最终下标 / 2 （向下取整）
        c = (u + v) >> 1
        // 从 result 中根据 c（中间位），取出中间位的下标。
        // 然后利用中间位的下标，从 arr 中取出对应的值。
        // 即：arr[result[c]] = result 中间位的值
        // 如果：result 中间位的值 < arrI，则 u（初始下标）= 中间位 + 1。即：从中间向右移动一位，作为初始下标。 （下次直接从中间开始，往后计算即可）
        if (arr[result[c]] < arrI) {
          u = c + 1
        } else {
          // 否则，则 v（最终下标） = 中间位。即：下次直接从 0 开始，计算到中间位置 即可。
          v = c
        }
      }
      // 最终，经过 while 的二分运算可以计算出：目标下标位 u
      // 利用 u 从 result 中获取下标，然后拿到 arr 中对应的值：arr[result[u]]
      // 如果：arr[result[u]] > arrI 的，则证明当前  result 中存在的下标 《不是》 递增序列，则需要进行替换
      if (arrI < arr[result[u]]) {
        if (u > 0) {
          p[i] = result[u - 1]
        }
        // 进行替换，替换为递增序列
        result[u] = i
      }
    }
  }
  // 重新定义 u。此时：u = result 的长度
  u = result.length
  // 重新定义 v。此时 v = result 的最后一个元素
  v = result[u - 1]
  // 自后向前处理 result，利用 p 中所保存的索引值，进行最后的一次回溯
  while (u-- > 0) {
    result[u] = v
    v = p[v]
  }
  return result
}
```

>在这里咱们主要了解了最长递增子序列的概念，大家需要明确的是：**最长递增子序列可以帮助咱们减少移动的次数，从而提升性能**。

那么到目前为止，我们已经明确了：

1. `diff` 指的就是：`添加、删除、打补丁、移动` 这四个行为
2. `最长递增子序列` 是什么，如何计算的，以及在 `diff` 中的作用
3. 场景五的乱序，是最复杂的场景，将会涉及到 `添加、删除、打补丁、移动` 这些所有场景。

那么明确好了以上内容之后，接下来咱们就来看第五步的详细逻辑，同样添加了详细备注：

```js
// 5. unknown sequence
// [i ... e1 + 1]: a b [c d e] f g
// [i ... e2 + 1]: a b [e d c h] f g
// i = 2, e1 = 4, e2 = 5
else {
  // 旧子节点的开始索引：oldChildrenStart
  const s1 = i 
  // 新子节点的开始索引：newChildrenStart
  const s2 = i 

  // 5.1 创建一个 <key（新节点的 key）:index（新节点的位置）> 的 Map 对象 keyToNewIndexMap。通过该对象可知：新的 child（根据 key 判断指定 child） 更新后的位置（根据对应的 index 判断）在哪里
  const keyToNewIndexMap: Map<string | number | symbol, number> = new Map()
  // 通过循环为 keyToNewIndexMap 填充值（s2 = newChildrenStart; e2 = newChildrenEnd）
  for (i = s2; i <= e2; i++) {
    // 从 newChildren 中根据开始索引获取每一个 child（c2 = newChildren）
    const nextChild = (c2[i] = optimized
      ? cloneIfMounted(c2[i] as VNode)
      : normalizeVNode(c2[i]))
    // child 必须存在 key（这也是为什么 v-for 必须要有 key 的原因）
    if (nextChild.key != null) {
      // key 不可以重复，否则你将会得到一个错误
      if (__DEV__ && keyToNewIndexMap.has(nextChild.key)) {
        warn(
          `Duplicate keys found during update:`,
          JSON.stringify(nextChild.key),
          `Make sure keys are unique.`
        )
      }
      // 把 key 和 对应的索引，放到 keyToNewIndexMap 对象中
      keyToNewIndexMap.set(nextChild.key, i)
    }
  }

  // 5.2 循环 oldChildren ，并尝试进行 patch（打补丁）或 unmount（删除）旧节点
  let j
  // 记录已经修复的新节点数量
  let patched = 0
  // 新节点待修补的数量 = newChildrenEnd - newChildrenStart + 1
  const toBePatched = e2 - s2 + 1
  // 标记位：节点是否需要移动
  let moved = false
  // 配合 moved 进行使用，它始终保存当前最大的 index 值
  let maxNewIndexSoFar = 0
  // 创建一个 Array 的对象，用来确定最长递增子序列。它的下标表示：《新节点的下标（newIndex），不计算已处理的节点。即：n-c 被认为是 0》，元素表示：《对应旧节点的下标（oldIndex），永远 +1》
  // 但是，需要特别注意的是：oldIndex 的值应该永远 +1 （ 因为 0 代表了特殊含义，他表示《新节点没有找到对应的旧节点，此时需要新增新节点》）。即：旧节点下标为 0， 但是记录时会被记录为 1
  const newIndexToOldIndexMap = new Array(toBePatched)
  // 遍历 toBePatched ，为 newIndexToOldIndexMap 进行初始化，初始化时，所有的元素为 0
  for (i = 0; i < toBePatched; i++) newIndexToOldIndexMap[i] = 0
 // 遍历 oldChildren（s1 = oldChildrenStart; e1 = oldChildrenEnd），获取旧节点（c1 = oldChildren），如果当前 已经处理的节点数量 > 待处理的节点数量，那么就证明：《所有的节点都已经更新完成，剩余的旧节点全部删除即可》
  for (i = s1; i <= e1; i++) {
    // 获取旧节点（c1 = oldChildren）
    const prevChild = c1[i]
    // 如果当前 已经处理的节点数量 > 待处理的节点数量，那么就证明：《所有的节点都已经更新完成，剩余的旧节点全部删除即可》
    if (patched >= toBePatched) {
      // 所有的节点都已经更新完成，剩余的旧节点全部删除即可
      unmount(prevChild, parentComponent, parentSuspense, true)
      continue
    }
    // 新节点需要存在的位置，需要根据旧节点来进行寻找（包含已处理的节点。即：n-c 被认为是 1）
    let newIndex
    // 旧节点的 key 存在时
    if (prevChild.key != null) {
      // 根据旧节点的 key，从 keyToNewIndexMap 中可以获取到新节点对应的位置
      newIndex = keyToNewIndexMap.get(prevChild.key)
    } else {
      // 旧节点的 key 不存在（无 key 节点）
      // 那么我们就遍历所有的新节点（s2 = newChildrenStart; e2 = newChildrenEnd），找到《没有找到对应旧节点的新节点，并且该新节点可以和旧节点匹配》（s2 = newChildrenStart; c2 = newChildren），如果能找到，那么 newIndex = 该新节点索引
      for (j = s2; j <= e2; j++) {
       // 找到《没有找到对应旧节点的新节点，并且该新节点可以和旧节点匹配》（s2 = newChildrenStart; c2 = newChildren）
        if (
          newIndexToOldIndexMap[j - s2] === 0 &&
          isSameVNodeType(prevChild, c2[j] as VNode)
        ) {
          // 如果能找到，那么 newIndex = 该新节点索引
          newIndex = j
          break
        }
      }
    }
    // 最终没有找到新节点的索引，则证明：当前旧节点没有对应的新节点
    if (newIndex === undefined) {
      // 此时，直接删除即可
      unmount(prevChild, parentComponent, parentSuspense, true)
    } 
    // 没有进入 if，则表示：当前旧节点找到了对应的新节点，那么接下来就是要判断对于该新节点而言，是要 patch（打补丁）还是 move（移动）
    else {
      // 为 newIndexToOldIndexMap 填充值：下标表示：《新节点的下标（newIndex），不计算已处理的节点。即：n-c 被认为是 0》，元素表示：《对应旧节点的下标（oldIndex），永远 +1》
      // 因为 newIndex 包含已处理的节点，所以需要减去 s2（s2 = newChildrenStart）表示：不计算已处理的节点
      newIndexToOldIndexMap[newIndex - s2] = i + 1
      // maxNewIndexSoFar 会存储当前最大的 newIndex，它应该是一个递增的，如果没有递增，则证明有节点需要移动
      if (newIndex >= maxNewIndexSoFar) {
        // 持续递增
        maxNewIndexSoFar = newIndex
      } else {
        // 没有递增，则需要移动，moved = true
        moved = true
      }
      // 打补丁
      patch(
        prevChild,
        c2[newIndex] as VNode,
        container,
        null,
        parentComponent,
        parentSuspense,
        isSVG,
        slotScopeIds,
        optimized
      )
      // 自增已处理的节点数量
      patched++
    }
  }

  // 5.3 针对移动和挂载的处理
  // 仅当节点需要移动的时候，我们才需要生成最长递增子序列，否则只需要有一个空数组即可
  const increasingNewIndexSequence = moved
    ? getSequence(newIndexToOldIndexMap)
    : EMPTY_ARR
  // j >= 0 表示：初始值为 最长递增子序列的最后下标
  // j < 0 表示：《不存在》最长递增子序列。
  j = increasingNewIndexSequence.length - 1
  // 倒序循环，以便我们可以使用最后修补的节点作为锚点
  for (i = toBePatched - 1; i >= 0; i--) {
    // nextIndex（需要更新的新节点下标） = newChildrenStart + i
    const nextIndex = s2 + i
    // 根据 nextIndex 拿到要处理的 新节点
    const nextChild = c2[nextIndex] as VNode
    // 获取锚点（是否超过了最长长度）
    const anchor =
      nextIndex + 1 < l2 ? (c2[nextIndex + 1] as VNode).el : parentAnchor
    // 如果 newIndexToOldIndexMap 中保存的 value = 0，则表示：新节点没有用对应的旧节点，此时需要挂载新节点
    if (newIndexToOldIndexMap[i] === 0) {
      // 挂载新节点
      patch(
        null,
        nextChild,
        container,
        anchor,
        parentComponent,
        parentSuspense,
        isSVG,
        slotScopeIds,
        optimized
      )
    } 
    // moved 为 true，表示需要移动
    else if (moved) {
      // j < 0 表示：不存在 最长递增子序列
      // i !== increasingNewIndexSequence[j] 表示：当前节点不在最后位置
      // 那么此时就需要 move （移动）
      if (j < 0 || i !== increasingNewIndexSequence[j]) {
        move(nextChild, container, anchor, MoveType.REORDER)
      } else {
        // j 随着循环递减
        j--
      }
    }
  }
}
```

由以上代码可知：

乱序下的 diff 是 最复杂 的一块场景
它的主要逻辑分为三大步：

1. 创建一个 `<key（新节点的 key）:index（新节点的位置）>`的 `Map`对象 `keyToNewIndexMap`。通过该对象可知：新的 `child（根据 key 判断指定 child）` 更新后的位置（根据对应的 index 判断）在哪里
2. 循环 `oldChildren`，并尝试进行 `patch（打补丁）`或 `unmount（删除`）旧节点
3. 处理 移动和挂载


