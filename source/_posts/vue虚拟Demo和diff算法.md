---
title: vue虚拟Demo和diff算法
date: 2022-07-14 16:29:12
tags:
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

# 有key和无key的算法区别

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

其实以上可以看到，无key的时候，渲染很简单，就是直接用新Node渲染替换对应索引位置的旧Node，然后对比新旧数量，如果新的少了，
则删除多的，如果多了就添加DOM

## 有key

那么重中之重就是有key的计算逻辑了，在这个阶段，其实我们最中心的逻辑就是一个:**同一个组件，如果没有改变，那么就不再重新渲染计算**
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

### 第一步：`sync from start` 自前向后的对比
>
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

## 最长递增子序列

[原博客地址](https://blog.csdn.net/m0_58941767/article/details/126436955)

一下是个人总结和觉得重要的知识点
> `子序列`：不一定是连续的
   `字串`：一定是连续的

假设一个数组 `nums`，`dp[i]` 表示以 `nums[i]` 这个数结尾的最长递增子序列的长度。

```js
let nums=[1,4,3,4,2]
//db[3]=3;db[4]=2
```

这个 GIF 展示了算法演进的过程：
![渲染](https://img-blog.csdnimg.cn/9039b120c2b947968959e4514823cdf6.gif)

根据这个定义，我们的最终结果（子序列的最大长度）**应该是 dp 数组中的最大值。**

```js
int res = 0;
for (int i = 0; i < dp.length; i++) {
    res = Math.max(res, dp[i]);
}
return res;
```

那么如何归纳呢 假设我们已经知道了 dp[0..4] 的所有结果，我们如何通过这些已知结果推出 dp[5] 呢?

```js
let nums=[1,4,3,4,2,3]
//db=[1,2,2,3,2] 已知
//db[5]=3?
```

**nums[5] = 3，既然是递增子序列，我们只要找到前面那些结尾比 3 小的子序列，然后把 3 接到这些子序列末尾，就可以形成一个新的递增子序列，而且这个新的子序列长度加一。**
nums[5] 前面有哪些元素小于 nums[5]？这个好算，用 for 循环比较一波就能把这些元素找出来。

```js
// i 为当前我所需要算的db索引
for (int j = 0; j < i; j++) {
    //寻找 nums[0..j-1] 中比 nums[i] 小的元素
    if (nums[i] > nums[j]) {
        // 对比让db[i]一直取到最大的值且加1
        dp[i] = Math.max(dp[i], dp[j] + 1);
    }
}
```

如上所述，通过归纳法我们已近总结出了算法逻辑了

```js
function lengthOfLIS(nums:Number[]) {
    // 定义：dp[i] 表示以 nums[i] 这个数结尾的最长递增子序列的长度
     // base case：dp 数组全都初始化为 1
    let dp:Number[] = nums.map(()=>1)
    for (let i = 0; i < nums.length; i++) {
        for (let j = 0; j < i; j++) {
            if (nums[i] > nums[j]) 
                dp[i] = Math.max(dp[i], dp[j] + 1);
        }
    }
    
    let res = 0;
    for (let i = 0; i < dp.length; i++) {
        res = Math.max(res, dp[i]);
    }
    return res;
}
```
