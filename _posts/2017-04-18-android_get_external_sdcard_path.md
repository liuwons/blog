---
layout: post
title: Android获取外置SD卡读写路径
date: 2017-04-19 22:14:24
categories: code
tags: [Android]
---

## 1. 外置SD卡的一些问题

### 1.1 关于外置SD卡上的读写路径

`Android 4.4`及以上版本，应用的外置SD卡读写路径被限定在固定路径上(`外置SD卡根路径/Android/data/包名/files`)。

`Android4.4`以下版本，申请了外置SD卡读写权限的应用在整个外置SD卡上都有读写权限。



### 1.2 关于外置SD卡路径

另外**Android**没有提供获取外置SD卡路径的API(`getExternalStorageDirectory()`获取的实际是内置SD卡路径)。



## 2. 获取应用在外置SD卡读写根路径

在`Android 4.4`以下版本，获取的应该是外置SD卡的根目录(类似`/storage/sdcard1`)。在`Android 4.4`及以上版本，获取的是应用在SD卡上的限定目录(`外置SD卡根路径/Android/data/包名/files/file`)



代码如下：



``` Java

    public static String getExternalSDPath(Context aContext) {
        String root = null;
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.KITKAT) {
            root = getExternalSDPathKITKAT(aContext);
            File f = new File(root);
            if (!f.exists()) {
                try {
                    f.mkdirs();
                } catch (Exception e) {
                    e.printStackTrace();
                }
                if (!f.exists()) {
                    root = null;
                }
            }
        } else {
            root = getExternalSDCardPath(aContext);
        }
        return root;
    }


    // Android 4.4及以上版本,获取软件在外置SD卡上的保存路径
    public static String getExternalSDPathKITKAT(Context aContext) {
        String rootPath = getStoragePath(aContext, true);
        if (TextUtils.isEmpty(rootPath)) {
            return null;
        }
        File f = new File(rootPath, "Android/data/" + aContext.getPackageName() + "/files/file");
        String fpath = f.getAbsolutePath();
        return fpath;
    }

    // Android 4.4 以下版本获取外置SD卡根目录
    public static String getExternalSDCardPath(Context aContext) {
        HashSet<String> paths = getExternalMounts();
        File defaultPathFile = aContext.getExternalFilesDir(null);
        String defaultPath;
        if (defaultPathFile == null) {
            return null;
        } else {
            defaultPath = defaultPathFile.getAbsolutePath();
        }
        String prefered = null;
        for (Iterator it = paths.iterator(); it.hasNext();) {
            String path = (String) (it.next());
            if (prefered == null && !defaultPath.startsWith(path)) {
                prefered = path;
            }
        }
        return prefered;
    }
```
