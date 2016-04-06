---
layout: post
title: 用WindowsAPI截屏并转换为RGB格式
date: 2015-05-11 18:48:40
tags: Windows C++
categories: code
---

在Windows下捕获屏幕图像可以有多重方法，比较简单可以调用第三方库，如Qt的屏幕截屏API就很容易调用。
在这里介绍如何用Windows API实现截屏并转换成RGB格式存储。

```cpp
#include <windows.h>
//最终f的内存布局为BGRA格式，需要保证buf长度足够(>w*h*4)
void ScreenCap(void* buf, int* w, int* h)
{

    HWND hDesk = GetDesktopWindow();
    HDC hScreen = GetDC(hDesk);
    int width = GetDeviceCaps(hScreen, HORZRES);
    int height = GetDeviceCaps(hScreen, VERTRES);

    if (w != 0)
        *w = width;
    if (h != 0)
        *h = height;

    HDC hdcMem = CreateCompatibleDC(hScreen);
    HBITMAP hBitmap = CreateCompatibleBitmap(hScreen, width, height);

    BITMAPINFOHEADER bmi = { 0 };
    bmi.biSize = sizeof(BITMAPINFOHEADER);
    bmi.biPlanes = 1;
    bmi.biBitCount = 32;
    bmi.biWidth = width;
    bmi.biHeight = -height;
    bmi.biCompression = BI_RGB;
    bmi.biSizeImage = width*height;

    SelectObject(hdcMem, hBitmap);
    BitBlt(hdcMem, 0, 0, width, height, hScreen, 0, 0, SRCCOPY);

    GetDIBits(hdcMem, hBitmap, 0, height, buf, (BITMAPINFO*)&bmi, DIB_RGB_COLORS);

    DeleteDC(hdcMem);
    ReleaseDC(hDesk, hScreen);
    CloseWindow(hDesk);
    DeleteObject(hBitmap);
}
```
