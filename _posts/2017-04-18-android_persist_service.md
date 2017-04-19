---
layout: post
title: Android防止Service被杀死
date: 2017-04-18 23:42:47
categories: code
tags: [Android]
---

## 1. Service被杀死的两种场景



### 1.2 系统回收



在系统内存空间不足时可能会被系统杀死以回收内存，内存不足时Android会依据Service的优先级来清除Service。



### 1.2 用户清除



用户可以在"最近打开"(多任务窗口、任务管理窗口)中清除最近打开的任务，当用户清除了Service所在的任务时，Service可能被杀死(不同ROM有不同表现，在小米、魅族等第三方产商定制ROM上一般会被立即杀死，在Android N上没有被立即杀死)。



## 2. 解决方案



对于第一种场景(系统回收)，如果不用黑科技(双进程互开等)，我们只能够尽量提高Service的优先级，一种比较好的方式是使用**前台Service**。



对于第二种场景(用户手动清除)，可以将启动Service的任务排除出系统的最近任务列表。



### 2.1 前台Service



前台Service是一种有前台界面(Notification)、优先级非常高的Service，只有在系统内存空间严重不足时，才会清除前台Service。



只需要在Service的onCreate() 或者 onStartCommand() 内调用startForeground，就能将Service转为前台Service。示例如下：



```

private Notification buildForegroundNotification() {
   Notification.Builder builder = new Notification.Builder(this);

   builder.setOngoing(true);

   builder.setContentTitle(getString(R.string.notification_title))
           .setContentText(getString(R.string.notification_content))
           .setSmallIcon(R.mipmap.ic_launcher)
           .setTicker(getString(R.string.notification_ticker));
   builder.setPriority(Notification.PRIORITY_MAX);
   return builder.build();
}

@Override
public int onStartCommand(Intent intent, int flags, int startId) {
   Log.e(TAG, "onStartCommand");

   WindowManager windowManager = (WindowManager) getSystemService(WINDOW_SERVICE);
   mRootView = new FloatRootView(this);
   mRootView.attach(windowManager);
   mRootView.showBubble();
   startForeground(1, buildForegroundNotification());

   return START_STICKY;

}

```



### 2.2 移出最近任务



为了使Service不会被用户从"最近打开"中清除，我们可以将启动Service的任务从系统的最近应用列表中删除。做法是将任务Activity的excludeFromRecents设为true，如下：



```

<activity android:name=".MainActivity"
         android:excludeFromRecents="true">
   <intent-filter>
       <action android:name="android.intent.action.MAIN"/>

       <category android:name="android.intent.category.LAUNCHER"/>
   </intent-filter>
</activity>

```


