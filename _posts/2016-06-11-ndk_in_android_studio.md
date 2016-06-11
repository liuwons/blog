---
layout: post
title: 在Android Studio中进行NDK开发的一般流程
date: 2016-06-11 23:58:08
categories: code
tags: [Android]
---

1 在类中声明native方法

2 在 ```app/src/main``` 下创建 ```jni``` 目录

3 在 ```app/src/main/java``` 下运行命令 ```javah -jni -d ../jni com.path2class.ClassName```

4 在 ```app/src/main/jni``` 下生成了对应的头文件，创建cpp源文件，利用此头文件实现对应的native方法

5 在 ```app``` 下的 ```build.gradle``` 文件中，android->defaultConfig下添加代码：

```
ndk {    
   moduleName "jnitest"          // 生成的so动态库名称
   abiFilters "armeabi", "armeabi-v7a", "x86" // 输出指定三种abi体系结构下的so库，目前可有可无
}
```

6 在需要用到native方法的java类中添加如下代码来加载native库：

```
static {    
   System.loadLibrary("jnitest");    // 必须与之前在build.gradle中设置的so库名称一致
}
```

7 现在可以在加载了so库的java类中调用native方法了
