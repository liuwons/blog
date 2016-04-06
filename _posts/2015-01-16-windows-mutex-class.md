---
layout: post
title: Windows下简单的互斥锁封装
date: 2015-01-16 14:09:22
tags: Windows C++ Thread
categories: code
---

在多线程编程中线程的互斥与同步尤为重要。Windows操作系统提供了不同的线程互斥同步机制：临界区、互斥锁、事件和信号量。

以下为其中的互斥锁Mutex的封装类，为应用提供了简单的接口来操作互斥锁。
<!-- more -->

``` cpp
#include <windows.h>

class Mutex
{
private:
	HANDLE mutex;

public:
	Mutex()
	{
		mutex = CreateMutex(NULL, FALSE, NULL);
	}

	~Mutex()
	{
		CloseHandle(mutex);
	}

	void waitAndLock()
	{
		WaitForSingleObject(mutex, INFINITE);
	}

	void unlock()
	{
		ReleaseMutex(mutex);
	}
};
```