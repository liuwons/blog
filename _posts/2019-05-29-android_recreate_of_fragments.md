---
layout: post
title: Android 中Fragment的自动重建
date: 2018-05-29 21:08:12
categories: code
tags: [Android]
---


Fragment常见的两种重建方式，一种是通过调用`setRetainInstance`来通知系统在重建Activity(例如屏幕配置改变)时保留此Fragment；另一种方式是系统在重建Activity时自动重建Fragment，典型例子是FragmentActivity对管理的Fragment的重建。


### setRetainInstance

直接参考[官方文档](https://developer.android.com/reference/android/app/Fragment#setRetainInstance(boolean))的解释：

> Control whether a fragment instance is retained across Activity re-creation (such as from a configuration change). This can only be used with fragments not in the back stack. If set, the fragment lifecycle will be slightly different when an activity is recreated:
> - onDestroy() will not be called (but onDetach() still will be, because the fragment is being detached from its current activity).- onCreate(android.os.Bundle) will not be called since the fragment is not being re-created.- onAttach(android.app.Activity) and onActivityCreated(android.os.Bundle) will still be called.

调用了此方法后，Activity被销毁时，此Fragment会被保留(进程不消亡的前提下)，Activity重建时，Fragment直接挂载。

原理：当配置发生变化时，Activity进入销毁过程，FragmentManager先销毁队列中Fragment的视图，然后检查每个Fragment的retainInstance属性。如果retainInstance为false，FragmentManager会销毁该Fragment实例；如果retainInstance为true，则不会销毁该Fragment实例，Activity重建后，新的FragmentManager会找到保留的Fragment并为其创建视图。


### FragmentActivity对Fragment的重建

FragmentActivity在配置改变被销毁时，FragmentManager中的Fragment状态会被保存下来，等之后Activity重建时，被销毁的Fragment也会被重建。所以在这种情况下，Activity重建时，不需要再在onCreate中创建新的Fragment，而是使用被自动重建的Fragment(可以通过tag找到)。Fragment的状态保存及重建过程可以在FragmentActivity的代码中查看。


当配置改变，Activity即将销毁时，FragmentActivity会先保存所有队列中的Fragment状态。

```
    @Override    protected void onSaveInstanceState(Bundle outState) {        super.onSaveInstanceState(outState);        markFragmentsCreated();        Parcelable p = mFragments.saveAllState();        if (p != null) {            outState.putParcelable(FRAGMENTS_TAG, p);        }
```

mFragments为FragmentManager的引用，`FragmentManager.saveAllState`返回一个包含需要保存的Fragment的Parcelable，此数据之后会用于重建这些Fragment。

查看`FragmentManager.saveAllState`方法的实现，可以看出返回的Parcelable实际为一个FragmentManagerState实例，该类的数据成员定义为：

```
final class FragmentManagerState implements Parcelable {    FragmentState[] mActive;    int[] mAdded;    BackStackState[] mBackStack;    int mPrimaryNavActiveIndex = -1;    int mNextFragmentIndex;
}
``` 

FragmentState的数据成员：

```
final class FragmentState implements Parcelable {    final String mClassName;    final int mIndex;    final boolean mFromLayout;    final int mFragmentId;    final int mContainerId;    final String mTag;    final boolean mRetainInstance;    final boolean mDetached;    final Bundle mArguments;    final boolean mHidden;    Bundle mSavedFragmentState;    Fragment mInstance;
}
```

FragmentState包含描述Fragment的各个数据变量，足够从零重新创建一个被销毁的Fragment。

在保存了Fragment数据之后，Activity实例以及没有调用`setRetainInstance(true)`的Fragment实例都被销毁。

而当Activity重建时，被销毁的Fragment也会被重建：

```
    @Override    protected void onCreate(@Nullable Bundle savedInstanceState) {        mFragments.attachHost(null /*parent*/);        super.onCreate(savedInstanceState);        NonConfigurationInstances nc =                (NonConfigurationInstances) getLastNonConfigurationInstance();        if (nc != null) {            mViewModelStore = nc.viewModelStore;        }        if (savedInstanceState != null) {            Parcelable p = savedInstanceState.getParcelable(FRAGMENTS_TAG);            mFragments.restoreAllState(p, nc != null ? nc.fragments : null);
```

FragmentActivity通过调用FragmentManager的restoreAllState方法，重建之前保存下来并被销毁的Fragment。