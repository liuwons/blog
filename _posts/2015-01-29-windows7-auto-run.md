---
layout: post
title: Windows7开机启动项管理
date: 2015-01-29 16:28:29
tags: Windows
categories: tech
---

Windows7有很多种方式可以管理开机启动程序，这里介绍三种方式：管理“启动”目录，修改注册表，msconfig系统配置。

### 1.“启动目录”

Windows7下的系统“启动”目录中可以很方便地添加开机启动项，要添加时，只要向此目录里添加程序（或批处理文件）的快捷方式即可。如下：
![打开“启动”目录](img/start.jpg)
![“启动”目录](img/start_folder.jpg)

### 2.修改注册表

可以在注册表路径HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows/CurrentVersion/Run里添加或者删除启动项，如下：
![打开注册表编辑器](img/open_regedit.jpg)
![注册表编辑器启动项目录](img/regedit_run.jpg)

### 3.msconfig系统配置

可以在msconfig系统配置程序里管理（启动/禁用）启动项。
![打开系统配置程序](img/open_msconfig.jpg)
![系统配置程序启动项管理](img/msconfig.jpg)
