---
layout: post
title: Windows下从dll中导出lib文件
date: 2015-01-16 15:04:01
tags: Windows
categories: code
---

在Windows平台上使用MSYS编译一些开源代码时经常只会生成DLL文件，有事获取的SDK也可能丢失lib文件，而如果打算在VS中使用DLL则需要有对应的LIB文件，以下方法介绍如何从DLL文件中导出LIB文件。

此方法有两个步骤：由dll文件生成def文件; 由def文件导出lib文件。

## 1. 通过pexports或微软编译环境自带的dumpbin.exe导出DLL对应的def文件:

``` shell
pexports ***.dll > ***.def
```

## 2. 通过微软编译环境自带的lib.exe程序根据.def生成我们需要的lib文件:

``` shell
lib /def:***.def /machine:i386 /out:***.lib
```
