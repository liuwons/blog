---
layout: post
title: Android NestedScrolling解决滑动冲突问题(2) - fling问题与NestedScroll++
date: 2018-11-30 10:26:18
categories: code
tags: [Android]
---

## 滑动的处理

在前一篇文章中，我们分析了解决滑动冲突问题的 **NestedScroll** 接口，也给出了解决此类问题的一般性方案：

#### NestedScrollingChild侧
`NestedScrollingChild`(后面简称NC)处理`MotionEvent`(一般在`onTouchEvent`中，如果是`ViewGroup`还要注意`onInterceptTouchEvent`的处理，拦截滑动相关的`MotionEvent`事件)，分析用户滑动操作。

在滑动开始时，调用`startNestedScroll`找到联动此次滑动的`NestedScrollingParent`(后面简称NP)。

对于每次用户交互产生的滑动距离，先调用`dispatchNestedPreScroll`，询问联动NP是否预先处理此滑动，如果NP预先处理了，会给出消耗掉的滑动距离。

对于NP预处理剩下的滑动距离，NC决定自己是否处理部分或者全部距离(自己的滑动)。

如果NC自己滚动之后，还剩下部分滑动距离，则调用`dispatchNestedScroll`让NP自行选择是否处理最后剩下的这些滑动距离。

用户交互停止滑动，调用`stopNestedScroll`通知NC停止滑动联动。

#### NestedScrollingParent侧
在`onStartNestedScroll`中，决定是否与此次NC发起的滑动请求联动，如果决定联动，返回`true`，否则返回`false`。返回`true`之后，会收到`onNestedScrollAccepted`回调，表示NC同意与其联动，可以开始做初始化操作了；返回false之后，后面的NC联动操作不会通知此`NestedScrollingParent`(不会收到后续的`onNestedPreScroll`、`onNestedScroll`、`onStopNestedScroll`等)。

在`onNestedPreScroll`中，决定是否预处理滑动单步，并给出消耗掉的滑动距离(不处理则为0)。

在`onNestedScroll`中，决定是否消耗NC处理剩下的滑动距离。

在`onStopNestedScroll`做联动滑动收尾工作。

通过NC与NP的配合，可以做到很多复杂的滑动操作。只要分析了界面上外层视图与内层视图在滑动时的交互逻辑，就可以利用这两个接口实现。

## fling的处理

相对于滑动操作，还有一个fling操作，也叫猛划，指用户拖住UI元素快速滑动之后抬手，这时会有一个fling事件，一般的操作逻辑是UI元素在抬手之后按照初始速度做减速运动。

**NestedScroll** 接口也提供了API处理fling事件，在`NestedScrollingChild`中`dispatchNestedPreFling`通知NP预处理fling事件，`dispatchNestedFling`通知NP后处理fling事件。

```java
boolean dispatchNestedPreFling(float velocityX, float velocityY);
boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed);
```

`NestedScrollingParent`中对应的接口为`onNestedPreFling`、`onNestedFling`。

```java
boolean onNestedPreFling(@NonNull View target, float velocityX, float velocityY);
boolean onNestedFling(@NonNull View target, float velocityX, float velocityY, boolean consumed);
```

通过这几个接口，可以让NC和NP各自对fling事件做出反应，但是不能像滑动事件一样联动。即不能先让NP预处理 **部分** fling 速度，然后NC处理剩下的 **部分** fling速度，再将最后剩下的交给NP继续处理。这种情况下，上层UI元素与下层UI元素缺乏交互，很难做到像滑动操作一样的UI效果(例如fling时先收起上层视图部分内容，再滑动下层视图)。

## NestedScroll++

为了解决此问题，在support包 `26.0.0-beta2` 版本中引入了 **NestedScroll** 接口的升级版本(后面称为 **NestedScroll++** ): `NestedScrollingParent2`、`NestedScrollingChild2` 。

在 **NestedScroll++** 接口中，引入了touch **type** 的概念：对于用户手指触摸拖拽产生的滑动事件type为 `ViewCompat.TYPE_TOUCH`， fling产生的滑动事件 **type** 为 `ViewCompat.TYPE_NON_TOUCH`。滑动接口的相应API中加入了此 **type** 参数，如`onNestedScroll`接口改为：

```java
void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed, @NestedScrollType int type);
```

**NestedScroll++** 接口中，对于滑动事件的处理，与 **NestedScroll** 接口一样，只是API中加入了 **type** 参数。

而对于fling事件的处理，不再依赖于 `dispatchNestedPreFling`、`dispatchNestedFling`、`onNestedPreFling`、`onNestedFling`等接口，而是选择使用与滑动事件相同的处理方式，只是 **type** 不同（为`ViewCompat.TYPE_NON_TOUCH`）。

相应的交互逻辑改为：

#### NestedScrollingChild侧

在fling开始时，调用`startNestedScroll`找到联动此次滑动的`NestedScrollingParent`(后面简称NP)。

每次刷新视图时，计算当前时间片由fling产生的滑动距离，先调用`dispatchNestedPreScroll`，询问联动NP是否预先处理此滑动距离，如果NP预先处理了，会给出消耗掉的滑动距离。

对于NP预处理剩下的滑动距离，NC决定自己是否处理部分或者全部距离(自己的滑动)。

如果NC自己滚动之后，还剩下部分滑动距离，则调用`dispatchNestedScroll`让NP自行选择是否处理最后剩下的这些滑动距离。

用户交互停止滑动，调用stopNestedScroll通知NC停止滑动联动。

#### NestedScrollingParent侧

在`onStartNestedScroll`中，决定是否与此次NC发起的fling联动请求，如果决定联动，返回 `true` ，否则返回 `false` 。返回`true`之后，会收到`onNestedScrollAccepted`回掉，表示NC同意与其联动，可以开始做初始化操作了；返回false之后，后面的NC联动操作不会通知此`NestedScrollingParent`(不会收到后续的`onNestedPreScroll`、`onNestedScroll`、`onStopNestedScroll`等)。

在`onNestedPreScroll`中，决定是否预处理fling产生的滑动距离，并给出消耗掉的滑动距离(不处理则为0)。

在`onNestedScroll`中，决定是否消耗NC处理剩下的滑动距离。

在`onStopNestedScroll`做联动滑动收尾工作。
