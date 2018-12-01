---
layout: post
title: Android NestedScrolling解决滑动冲突问题(3) - 项目实战
date: 2018-11-30 20:53:36
categories: code
tags: [Android]
---

## 实际需求

在前面的两片文章中我们了解了 **NestedScroll** 的相关接口及一般处理逻辑。在本篇文章中就实现一个具体的联合滑动需求。

Android中经常在布局中嵌入 **WebView** 来展示网页内容，而且WebView内部还有交互逻辑（滚动之类的），如果外部布局也要处理滚动逻辑，就会有滑动冲突，这种场景在实际项目开发中很常见，例如在含有 `AppBarLayout` 的 `CoordinatorLayout` 中嵌入一个 **WebView** ， **WebView** 底部再放一个 **footer** 放置收藏按钮等，需要在向上滑动时首先保持 **WebView** 跟随 `AppBarLayout` 滑动，在 `AppBarLayout` 滑出屏幕之后， **WebView** 全屏展示，继续滑动 **WebView** ，**WebView** 划到底之后将 **WebView** 及 **footer** 一起向上继续滑动。实际效果如下图：

![实际滑动效果](https://github.com/liuwons/NestedScrollExample/raw/master/nested_scroll.gif)

## 需求解析

针对此需求，根据 `CoordinatorLayout` 及 `AppBarLayout` 的了解，我们可以将 **WebView** 放在 `CoordinatorLayout` 的一个子layout里，并将该layout的 `layout_behavior` 设为 `appbar_scrolling_view_behavior`，即可实现滑动时维持 **WebView** 在 `AppBarLayout` 底部并跟随滑动直至 `AppBarLayout` 滑出顶部 **WebView** 全屏展示。

但是如何在 **WebView** 全屏展示之后能够继续滑动 **WebView** 内容直至不能滑动，拖动出 **footer** 呢。

一种比较简单的做法是，将 **WebView** 及 **footer** 放在一个自定义的layout里，编程实现WebView的内容滚动及整个布局的滚动（ **WebView** 划到底之后滚动布局）。layout文件如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/activity_root"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:clipChildren="false"
    android:background="#ffffff"
    android:fitsSystemWindows="true">

    <android.support.design.widget.CoordinatorLayout
        android:id="@+id/preview_coordinator_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:clipChildren="false"
        android:fitsSystemWindows="true">

        <android.support.design.widget.AppBarLayout
            android:id="@+id/preview_app_bar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:clipChildren="false"
            app:elevation="0dp">

            <RelativeLayout
                android:id="@+id/title_bar"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:background="@color/colorPrimary"
                app:layout_scrollFlags="scroll">

                <TextView
                    android:id="@+id/text_title"
                    android:layout_width="wrap_content"
                    android:layout_height="50dp"
                    android:layout_alignParentTop="true"
                    android:gravity="center"
                    android:textColor="#ffffff"
                    android:textStyle="bold"
                    android:textSize="18sp"
                    android:text="随便写个标题"
                    android:layout_centerHorizontal="true"/>
            </RelativeLayout>
        </android.support.design.widget.AppBarLayout>

        <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior">

            <com.lwons.nestedscrollexample.ScrollingContent
                android:id="@+id/scrolling_content"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="#ffffff"
                android:orientation="vertical">
            </com.lwons.nestedscrollexample.ScrollingContent>

        </FrameLayout>

    </android.support.design.widget.CoordinatorLayout>
</LinearLayout>
```

这里 `com.lwons.nestedscrollexample.ScrollingContent` 是基于 `LinearLayout` 的自定义布局。里面放置了height为 `MATCH_PARENT` 的 **WebView** 及height为实际高度的 **footer** 。

而为了不影响 **WebView** 及 **footer** 的点击事件，我们需要尽量只拦截处理滑动相关的事件，这里需要在自定义布局的 `onInterceptTouchEvent` 中过滤 `MotionEvent` 。如下：

```java
private int mTouchSlop = ViewConfiguration.get(getContext()).getScaledTouchSlop();
private float mLastY;
private boolean mIsDraging;
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
    final int action = MotionEventCompat.getActionMasked(ev);

    if (action == MotionEvent.ACTION_CANCEL || action == MotionEvent.ACTION_UP) {
        mIsDraging = false;
        return false;
    }

    switch (action) {
        case MotionEvent.ACTION_MOVE: {
            if (mIsDraging) {
                return true;
            }
            final float yoff = Math.abs(mLastY - ev.getRawY());

            if (yoff > mTouchSlop) {
                // 只有手指滑动距离大于阈值时，才会开始拦截
                // Start scrolling!
                getParent().requestDisallowInterceptTouchEvent(true);
                return true;
            }
            break;
        }
        case MotionEvent.ACTION_DOWN:
            mLastY = ev.getRawY();
            break;
    }
    return false;
}
```

这样自定义布局就能够拦截到滑动事件，并能够得到每一步的滑动距离，但是如何处理这个滑动距离呢。

回顾 **NestedScroll** 接口的使用方式及特点，我们在自定义布局中拦截了滑动事件之后需要与外部布局联动，而发起联动及控制联动的一方是 `NestedScrollingChild` (后面简称NC)，因此我们需要在自定义布局里实现 `NestedScrollingChild` 相关的接口并控制滑动逻辑，而 `NestedScrollingParent` (后面简称NP)是哪个布局呢， 从`CoordinatorLayout`的代码中我们可以得知NP就是`CoordinatorLayout`，它会处理`AppBarLayout`的滑动。

联系需求的滑动交互详情，我们在向上滑动时，首先需要滑动`AppBarLayout`并使 **WebView** 跟随滑动，这一部分`CoordinatorLayout`会帮我们实现，我们只需要调用`dispatchNestedPreScroll`通知`CoordinatorLayout`就行了。然后`AppBarLayout`滑出顶部之后，需要继续滚动 **WebView** ，这一部分需要我们自己处理，只需要调用 **WebView** 的`scrollBy`接口即可。在 **WebView** 无法滑动时，我们需要滚动整个自定义布局，这里也简单，调用自定义布局的`scrollBy`接口即可，它会使得 **WebView** 和 **footer** 整体向上滚动。

分析到这里，整个向上滑动的操作过程就已经很清楚了。而向下的过程与向上基本相同。

处理逻辑的代码如下：

```java
@Override
public boolean onTouchEvent(MotionEvent ev) {
    boolean returnValue = false;

    MotionEvent event = MotionEvent.obtain(ev);
    final int action = MotionEventCompat.getActionMasked(event);
    float eventY = event.getRawY();
    switch (action) {
        case MotionEvent.ACTION_MOVE:
            if (getScrollState() == SCROLL_STAT_SCROLLING) {
                stopScroll();
            }
            if (mVelocityTracker == null) {
                mVelocityTracker = VelocityTracker.obtain();
            }
            if (!mIsDraging) {
                mIsDraging = true;
                startNestedScroll(ViewCompat.SCROLL_AXIS_VERTICAL, ViewCompat.TYPE_TOUCH);
            }
            // 滑动距离
            int deltaY = (int) (mLastY - eventY);
            mLastY = eventY;
            // 通知NP先进行滑动，这里CoordinatorLayout会滚动AppBarLayout及当前ScrollingContent布局
            if (dispatchNestedPreScroll(0, deltaY, mScrollConsumed, mScrollOffset, ViewCompat.TYPE_TOUCH)) {
                deltaY -= mScrollConsumed[1]; // mScrollConsumed[1]为CoordinatorLayout消耗掉的距离
                event.offsetLocation(0, -mScrollOffset[1]);
            }
            mVelocityTracker.addMovement(event);

            // 处理当前布局本身的滚动逻辑
            int scrollInternalY = 0;
            if (deltaY != 0) {
                scrollInternalY = scrollY(deltaY);
                deltaY -= scrollInternalY;
            }

            // 如果滑动距离还没有消耗完全，通知NP继续处理(NP可以选择处理或者不处理)
            if (deltaY != 0) {
                dispatchNestedScroll(0, mScrollConsumed[1]+scrollInternalY, 0, deltaY, mScrollOffset, ViewCompat.TYPE_TOUCH);
            }
            returnValue = true;
            break;
        case MotionEvent.ACTION_DOWN:
            break;
        case MotionEvent.ACTION_UP:
        case MotionEvent.ACTION_CANCEL:
            returnValue = true;
            mVelocityTracker.computeCurrentVelocity(1000);
            // fling逻辑
            onFlingY((int) -mVelocityTracker.getYVelocity());
            mVelocityTracker.clear();
            mIsDraging = false;
            // 停止手指拖拽的滑动
            stopNestedScroll(ViewCompat.TYPE_TOUCH);
            break;
    }
    return returnValue;
}

/**
 * 内部滚动逻辑
 * @param deltaY 当前未消耗的滑动距离
 * @return 内部滚动消耗掉的滑动距离
 */
private int scrollY(int deltaY) {
    int remainY = deltaY;
    int consumedY = 0;
    if (remainY > 0) {
        // 向上滑动

        if (mWebview != null && mWebview.canScrollUp() > 0) {
            // WebView还能继续向上滚动
            int readerScroll = Math.min(mWebview.canScrollUp(), remainY);
            mWebview.scrollBy(0, readerScroll);
            remainY -= readerScroll;
            consumedY += readerScroll;
        }

        if (remainY > 0 && getScrollY() < mFooter.getHeight()) {
            // 当前布局还能继续向上滚动
            int layoutScroll = Math.min(mFooter.getHeight() - getScrollY(), remainY);
            scrollBy(0, layoutScroll);
            consumedY += layoutScroll;
        }
    } else {
        // 向下滑动

        if (getScrollY() > 0) {
            // 当前布局还能继续向下滚动
            int layoutScroll = Math.max(-getScrollY(), remainY);
            scrollBy(0, layoutScroll);
            remainY -= layoutScroll;
            consumedY += layoutScroll;
        }

        if (mWebview != null && mWebview.canScrollDown() > 0) {
            // WebView还能继续向下滚动
            int readerScroll = Math.max(-mWebview.canScrollDown(), remainY);
            mWebview.scrollBy(0, readerScroll);
            consumedY += readerScroll;
        }
    }

    return consumedY;
}
```


## 完整实现

针对此需求，已创建了一个完整的Android工程放在GitHub上： [样例工程GitHub地址](https://github.com/liuwons/NestedScrollExample)

可以直接下载apk运行查看效果： [样例apk下载](https://github.com/liuwons/NestedScrollExample/releases/download/v1.0.0/nested_scroll_example_v1.0.0.apk)
