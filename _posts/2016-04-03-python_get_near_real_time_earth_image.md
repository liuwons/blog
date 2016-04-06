---
layout: post
title: Python工具:下载卫星Himawari8拍摄的实时地球图片
date: 2016-04-03 18:06:18
categories: code
tags: Python
---

# Himawari8 Image Downloader

**himawari8downloader** 是下载卫星 [**Himawari8**](https://en.wikipedia.org/wiki/Himawari_8) 拍摄到的地球近实时照片的 **Python** 脚本。

**GitHub** 地址： [**himawari8downloader**](https://github.com/liuwons/himawari8downloader) 。

## 依赖
**himawari8downloader** 依赖 `PIL` 和 `Requests`:

```bash
pip install Pillow
pip install requests
```

## 使用

直接运动 `himawari8downloader.py` ，参数： *fout*, *scale* 。

*fout* 是保存图片的路径。

*scale* 设置结果图片的大小， 图片的宽和高都为 *scale*×550 。
*scale* 可以是 1, 2, 4, 8, 16, 20.

例如运行以下命令

```python
python himawari8downloader.py earth.png 2
```

会在当前目录下生成大小为1100×1100的 *earth.png* 图片文件。

## 结果

![Result Image](blog/earth.png)
