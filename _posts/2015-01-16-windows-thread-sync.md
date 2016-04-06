---
layout: post
title: Windows下多线程/进程互斥与同步机制
date: 2015-01-16 14:39:53
tags: Windows Thread
categories: code
---

Windows操作系统为多线程/进程编程提供了多种互斥与同步机制：
### 1.临界区(CriticalSection)
API: InitializeCriticalSection, DeleteCriticalSection, EnterCriticalSection, LeaveCriticalSection
此机制具有线程所有权特性，持有临界区的线程可以重复进入临界区
### 2.互斥锁(Mutex)
API: CreateMutex, OpenMutex, ReleaseMutex
此机制也具有线程所有权
### 3.事件(Event)
API: CreateEvent, OpenEvent, SetEvent, ResetEvent, WaitForSingleObject
### 4.信号量(Semaphore)
API: CreateSemaphore, OpenSemaphore, ReleaseSemaphore
