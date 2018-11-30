---
layout: post
title: Android NestedScrolling解决滑动冲突问题(1) - 相关接口
date: 2018-11-29 22:14:36
categories: code
tags: [Android]
---


当父View及子View都可以滑动，并且滑动方向一致时(例如CoordinatorLayout内嵌RecyclerView或者Webview)，滑动冲突的解决就需要依赖于Android为我们提供的NestedScrolling接口。

NestedScrolling 接口分为两个部分：`NestedScrollingParent` 及 `NestedScrollingChild`。

为方便描述，以下简称`NestedScrollingParent`为`NP`, `NestedScrollingChild`为`NC`。

## NestedScrollingChild

包含的接口：
```java
public interface NestedScrollingChild {
    /**
     * 设置当前View是否启用nested scroll特性
     * @param enabled 是否启用
     */
    void setNestedScrollingEnabled(boolean enabled);

    /**
     * 当前View是否启用了nested scroll特性
     * @return
     */
    boolean isNestedScrollingEnabled();

    /**
     * 在axes轴上发起nested scroll开始操作
     * @param axes 滑动方向
     * @return 是否有NestedScrollingParent接受此次滑动请求
     */
    boolean startNestedScroll(@ViewCompat.ScrollAxis int axes);

    /**
     * 终止nested scroll
     */
    void stopNestedScroll();

    /**
     * 当前是否有NestedScrollingParent接受了此次滑动请求
     * @return 返回值
     */
    boolean hasNestedScrollingParent();

    /**
     * nested scroll的一步滑动操作中，在自己开始滑动处理之前，分配预处理操作（一般为询问NestedScrollingParent是否消耗部分滑动距离）
     * @param dx 当前这一步滑动的x轴总距离
     * @param dy 当前这一步滑动的y轴总距离
     * @param consumed 预处理操作消耗掉的距离（此为输出参数，consumed[0]为预处理操作消耗掉的x轴距离，consumed[1]为预处理操作消耗掉的y轴距离）
     * @param offsetInWindow 可选参数，可以为null。为输出参数，获取预处理操作使当前view的位置偏移(offsetInWindow[0]和offsetInWindow[1]分别为x轴和y轴偏移)
     * @return 预处理操作是否消耗了部分或者全部滑动距离
     */
    boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
                                    @Nullable int[] offsetInWindow);

    /**
     * 在当前View处理了滑动之后继续分配滑动操作 (一般在自己处理滑动之后，给NestedScrollingParent机会处理剩余的滑动距离)
     * @param dxConsumed 已经消耗了的x轴滑动距离
     * @param dyConsumed 已经消耗了的y轴滑动距离
     * @param dxUnconsumed 未消耗的x轴滑动距离
     * @param dyUnconsumed 未消耗的y轴滑动距离
     * @param offsetInWindow 可选参数，可以为null。为输出参数，获取预处理操作使当前view的位置偏移(offsetInWindow[0]和offsetInWindow[1]分别为x轴和y轴偏移)
     * @return
     */
    boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
                                 int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow);

    /**
     * 在当前NestedScrollingChild处理fling事件之前进行预处理(一般询问NestedScrollingParent是否处理消耗此次fling)
     * @param velocityX x轴速度
     * @param velocityY y轴速度
     * @return 预处理是否处理消耗了此次fling
     */
    boolean dispatchNestedPreFling(float velocityX, float velocityY);

    /**
     * 分配fling操作
     * @param velocityX x轴方向速度
     * @param velocityY y轴方向速度
     * @param consumed 当前NestedScrollingChild是否处理了此次fling
     * @return NestedScrollingParent是否处理了此次fling
     */
    boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed);
}

```


`NC`是`NP`的子孙(并非一定是直接子View)，也是联合滑动的请求方，滑动产生的一系列`MotionEvent`是在此View中跟踪处理的，一般此View是在 `onTouchEvent` 中依据对 `MotionEvent` 的跟踪分析来发起滑动请求。例如以下 `RecyclerView` 中 `onTouchEvent` 的简化版本：

```java
@Override
public boolean onTouchEvent(MotionEvent e) {
    final boolean canScrollHorizontally = mLayout.canScrollHorizontally();
    final boolean canScrollVertically = mLayout.canScrollVertically();

    if (mVelocityTracker == null) {
        mVelocityTracker = VelocityTracker.obtain();
    }
    boolean eventAddedToVelocityTracker = false;

    final MotionEvent vtev = MotionEvent.obtain(e);
    final int action = e.getActionMasked();
    final int actionIndex = e.getActionIndex();

    if (action == MotionEvent.ACTION_DOWN) {
        mNestedOffsets[0] = mNestedOffsets[1] = 0;
    }
    vtev.offsetLocation(mNestedOffsets[0], mNestedOffsets[1]);

    switch (action) {
        case MotionEvent.ACTION_DOWN: {
            mScrollPointerId = e.getPointerId(0);
            mInitialTouchX = mLastTouchX = (int) (e.getX() + 0.5f);
            mInitialTouchY = mLastTouchY = (int) (e.getY() + 0.5f);

            int nestedScrollAxis = ViewCompat.SCROLL_AXIS_NONE;
            if (canScrollHorizontally) {
                nestedScrollAxis |= ViewCompat.SCROLL_AXIS_HORIZONTAL;
            }
            if (canScrollVertically) {
                nestedScrollAxis |= ViewCompat.SCROLL_AXIS_VERTICAL;
            }
            // 发起滚动请求
            startNestedScroll(nestedScrollAxis, TYPE_TOUCH);
        } break;

        case MotionEvent.ACTION_MOVE: {
            final int x = (int) (e.getX(index) + 0.5f);
            final int y = (int) (e.getY(index) + 0.5f);
            int dx = mLastTouchX - x;
            int dy = mLastTouchY - y;

            // 先询问 NP 是否需要提前消耗滑动距离(部分或者全部)
            if (dispatchNestedPreScroll(dx, dy, mScrollConsumed, mScrollOffset, TYPE_TOUCH)) {
                // NP消耗了部分滑动距离
                dx -= mScrollConsumed[0]; // NP 消耗的X轴滑动距离
                dy -= mScrollConsumed[1]; // NP消耗的Y轴滑动距离
                vtev.offsetLocation(mScrollOffset[0], mScrollOffset[1]);
                // Updated the nested offsets
                mNestedOffsets[0] += mScrollOffset[0];
                mNestedOffsets[1] += mScrollOffset[1];
            }

            //分析是否本身需要滑动及本身滑动所消耗的滑动距离
            if (mScrollState != SCROLL_STATE_DRAGGING) {
                boolean startScroll = false;
                if (canScrollHorizontally && Math.abs(dx) > mTouchSlop) {
                    if (dx > 0) {
                        dx -= mTouchSlop;
                    } else {
                        dx += mTouchSlop;
                    }
                    startScroll = true;
                }
                if (canScrollVertically && Math.abs(dy) > mTouchSlop) {
                    if (dy > 0) {
                        dy -= mTouchSlop;
                    } else {
                        dy += mTouchSlop;
                    }
                    startScroll = true;
                }
                if (startScroll) {
                    setScrollState(SCROLL_STATE_DRAGGING);
                }
            }

            if (mScrollState == SCROLL_STATE_DRAGGING) {
                mLastTouchX = x - mScrollOffset[0];
                mLastTouchY = y - mScrollOffset[1];

                // 自己内部滑动
                if (scrollByInternal(
                        canScrollHorizontally ? dx : 0,
                        canScrollVertically ? dy : 0,
                        vtev)) {
                    getParent().requestDisallowInterceptTouchEvent(true);
                }
                if (mGapWorker != null && (dx != 0 || dy != 0)) {
                    mGapWorker.postFromTraversal(this, dx, dy);
                }
            }
        } break;

        case MotionEvent.ACTION_POINTER_UP: {
            onPointerUp(e);
        } break;

        case MotionEvent.ACTION_UP: {
            mVelocityTracker.addMovement(vtev);
            eventAddedToVelocityTracker = true;
            mVelocityTracker.computeCurrentVelocity(1000, mMaxFlingVelocity);
            final float xvel = canScrollHorizontally
                    ? -mVelocityTracker.getXVelocity(mScrollPointerId) : 0;
            final float yvel = canScrollVertically
                    ? -mVelocityTracker.getYVelocity(mScrollPointerId) : 0;
            // 分析是否产生fling事件(手机快速滑动之后抬起，视图继续滑动)
            if (!((xvel != 0 || yvel != 0) && fling((int) xvel, (int) yvel))) {
                setScrollState(SCROLL_STATE_IDLE);
            }
            resetTouch();
        } break;

        case MotionEvent.ACTION_CANCEL: {
            cancelTouch();
        } break;
    }

    if (!eventAddedToVelocityTracker) {
        mVelocityTracker.addMovement(vtev);
    }
    vtev.recycle();

    return true;
}
```

及 `scrollByInternal`简化版：

```java
boolean scrollByInternal(int x, int y, MotionEvent ev) {
    int unconsumedX = 0, unconsumedY = 0;
    int consumedX = 0, consumedY = 0;

    if (mAdapter != null) {
        if (x != 0) {
            // 自己滑动消耗的X轴滑动距离
            consumedX = mLayout.scrollHorizontallyBy(x, mRecycler, mState);
            //尚未消耗的X轴滑动距离
            unconsumedX = x - consumedX;
        }
        if (y != 0) {
            // 自己滑动消耗的Y轴滑动距离
            consumedY = mLayout.scrollVerticallyBy(y, mRecycler, mState);
            //尚未消耗的Y轴滑动距离
            unconsumedY = y - consumedY;
        }
    }

    // 通知 NP 继续消耗剩余的滑动距离
    if (dispatchNestedScroll(consumedX, consumedY, unconsumedX, unconsumedY, mScrollOffset,
            TYPE_TOUCH)) {
        // Update the last touch co-ords, taking any scroll offset into account
        mLastTouchX -= mScrollOffset[0];
        mLastTouchY -= mScrollOffset[1];
        if (ev != null) {
            ev.offsetLocation(mScrollOffset[0], mScrollOffset[1]);
        }
        mNestedOffsets[0] += mScrollOffset[0];
        mNestedOffsets[1] += mScrollOffset[1];
    }

    // 滑动距离是否已经完全消耗
    return consumedX != 0 || consumedY != 0;
}

```

所以针对一次滑动操作，`NC`的接口调用顺序为：

`startNestedScroll`  ->  `dispatchNestedPreScroll`  ->  `dispatchNestedScroll`  ->  `stopNestedScroll`

一般性的处理逻辑可以用以下伪代码总结：

```java
    private int mLastX;
    private int mLastY;
    private int[] mConsumed = new int[2];
    private int[] mOffsetInWindow = new int[2];
    @Override
    void onTouchEvent(MotionEvent event) {
        int eventX = (int) event.getRawX();
        int eventY = (int) event.getRawY();
        int action = event.getAction();
        int deltaX = eventX - mLastX;
        int deltaY = eventY - mLastY;
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                int nestedScrollAxis = ViewCompat.SCROLL_AXIS_NONE;
                if (canScrollHorizontally()) {
                    nestedScrollAxis |= ViewCompat.SCROLL_AXIS_HORIZONTAL;
                }
                if (canScrollVertically()) {
                    nestedScrollAxis |= ViewCompat.SCROLL_AXIS_VERTICAL;
                }
                startNestedScroll(nestedScrollAxis);
                break;
            case MotionEvent.ACTION_MOVE:
                if (dispatchNestedPreScroll(deltaX, deltaY, mConsumed, mOffsetInWindow)) {
                    deltaX -= mConsumed[0];
                    deltaY -= mConsumed[1];
                }
                int internalScrolledX = internalScrollByX(deltaX);
                int internalScrolledY = internalScrollByY(deltaY);
                deltaX -= internalScrolledX;
                deltaY -= internalScrolledY;
                if (deltaX != 0 || deltaY != 0) {
                    dispatchNestedScroll(mConsumed[0] + internalScrolledX, mConsumed[1] + internalScrolledY, deltaX, deltaY, mOffsetInWindow);
                }
                break;
            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_UP:
                stopNestedScroll();
                break;
        }
        mLastX = eventX;
        mLastY = eventY;
    }

    /**
     * X轴方向滑动
     * @param deltaX 滑动距离
     * @return 消耗的滑动距离
     */
    abstract int internalScrollByX(int deltaX);

    /**
     * Y轴方向滑动
     * @param deltaY 滑动距离
     * @return 消耗的滑动距离
     */
    abstract int internalScrollByY(int deltaY);

    /**
     * 是否支持横向滑动
     * @return 是否支持
     */
    abstract boolean canScrollHorizontally();

    /**
     * 是否支持竖向滑动
     * @return 是否支持
     */
    abstract boolean canScrollVertically();
```


## NestedScrollingParent

包含接口：

```java

public interface NestedScrollingParent {

    /**
     * 对NP子孙开始滑动请求的回应(NestedScrollingChild.startNestedScroll)
     * @param child 包含发起请求的NP子孙view的直接子view
     * @param target 发起请求的NP子孙view
     * @param axes 滑动方向
     * @return 是否响应此滑动事件
     */
    boolean onStartNestedScroll(@NonNull View child, @NonNull View target, @ViewCompat.ScrollAxis int axes);

    /**
     * 对开始滑动响应的回调(onStartNestedScroll返回true之后会有此回调产生),使NestedScrollingParent有做滑动初始化工作的时机
     * @param child 包含发起请求的NP子孙view的直接子view
     * @param target 发起请求的NP子孙view
     * @param axes 滑动方向
     */
    void onNestedScrollAccepted(@NonNull View child, @NonNull View target, @ViewCompat.ScrollAxis int axes);

    /**
     * 终止nested scroll的回调(NestedScrollingChild调用stopNestedScroll)
     * @param target 发起请求的NP子孙view
     */
    void onStopNestedScroll(@NonNull View target);

    /**
     * 在NestedScrollingChild处理滑动之前，预处理此滑动
     * @param target 发起请求的NP子孙view
     * @param dx x轴滑动距离
     * @param dy y轴滑动距离
     * @param consumed 回填参数，填入此次预处理消耗掉的滑动距离
     */
    void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed);

    /**
     * 处理NestedScrollingChild未消耗完的滑动距离
     * @param target 发起请求的NP子孙view
     * @param dxConsumed 已消耗的x轴滑动距离
     * @param dyConsumed 已消耗的y轴滑动距离
     * @param dxUnconsumed 未消耗的x轴滑动距离
     * @param dyUnconsumed 未消耗的y轴滑动距离
     */
    void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed,
                                 int dxUnconsumed, int dyUnconsumed);

    /**
     * 在NestedScrollingChild之前预处理fling事件
     * @param target 发起请求的NP子孙view
     * @param velocityX x轴fling速度
     * @param velocityY y轴fling速度
     * @return 是否处理此fling
     */
    boolean onNestedPreFling(@NonNull View target, float velocityX, float velocityY);

    /**
     * 处理fling事件
     * @param target 发起请求的NP子孙view
     * @param velocityX x轴fling速度
     * @param velocityY y轴fling速度
     * @param consumed NestedScrollingChild是否已处理此fling
     * @return 是否处理此fling
     */
    boolean onNestedFling(@NonNull View target, float velocityX, float velocityY, boolean consumed);


    /**
     * 获取滑动方向
     * @return 滑动方向
     */
    @ViewCompat.ScrollAxis
    int getNestedScrollAxes();
}

```

## 接口调用顺序

1. NC在处理MotionEvent时，决定发起滑动请求，调用startNestedScroll
2. 调用startNestedScroll会向上逐层遍历父view，调用其onStartNestedScroll接口，如果返回true，则此view为与此次nested scroll联动的NP并中断遍历；返回false则继续向上层遍历直到根view。如果遍历到根view还没找到联动NP，则后续滑动不可用联动。如果找到了，则进入第3步。
3. NC调用NP的onNestedScrollAccepted接口。
4. NP的onNestedScrollAccepted接口被调用，做一些滑动初始工作。
5. NC探测到用户交互产生了滑动距离，调用NP的onNestedPreScroll接口。
6. NP的onNestedPreScroll接口被调用，预处理此次滑动，消耗部分滑动距离(或者不消耗)。
7. NC处理剩余的滑动距离。
8. 如果NC没有处理完剩下的滑动距离，则调用dispatchNestedScroll。
9. NP的onNestedScroll被调用，自行决定是否继续处理剩下的滑动距离。
10. 交互上的滑动终止，NC调用stopNestedScroll。
11. NP的onStopNestedScroll被调用。
