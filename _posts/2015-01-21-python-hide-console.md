---
layout: post
title: Python程序隐藏控制台
date: 2015-01-21 10:02:08
tags: Python
categories: code
---

Python程序有时需要隐藏控制台，让进程在后台继续运行而不显示控制台。这时可以用ctypes库实现：
<!-- more -->
``` python
import ctypes

def hide_console():
	whnd = ctypes.windll.kernel32.GetConsoleWindow()
	if whnd != 0:
		ctypes.windll.user32.ShowWindow(whnd, 0)
		ctypes.windll.kernel32.CloseHandle(whnd)
```