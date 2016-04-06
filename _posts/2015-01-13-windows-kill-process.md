---
layout: post
title: Windows下C++程序终止进程
date: 2015-01-13 13:22:53
tags: Windows C++
categories: code
---

Windows下C++程序有多种方式可以关闭进程：
1. 通过执行命令行程序，如taskkill等。
2. 向进程发送关闭信号
3. 通过Windows API来关闭进程，如TerminateProcess函数

<!--more-->
以下函数通过调用TerminateProcess来关闭指定名称的进程。

``` cpp
bool kill_process(const char* lpszProcessName)
{
    unsigned int   pid = -1;
    bool    retval = true;

    if (lpszProcessName == NULL)
        return false;

    DWORD dwRet = 0;
    HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    PROCESSENTRY32 processInfo;
    processInfo.dwSize = sizeof(PROCESSENTRY32);
    int flag = Process32First(hSnapshot, &processInfo);

    while (flag != 0)
    {
        if (strcmp(processInfo.szExeFile, lpszProcessName) == 0)
        {
            pid = processInfo.th32ProcessID;
            HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, TRUE, pid);

            if (TerminateProcess(hProcess, 0) != TRUE)
            {
                retval = false;
                break;
            }
        }

        flag = Process32Next(hSnapshot, &processInfo);
    }

    CloseHandle(hSnapshot);

    if (pid == -1)
        return false;

    return retval;
}
```