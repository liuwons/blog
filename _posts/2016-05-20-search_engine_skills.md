---
layout: post
title: 常用搜索引擎技巧
date: 2016-05-20 02:08:07
categories: tech
tags: [Search Engine]
---

### 指定站内搜索

使用site指定在某网站内搜索

如只在知乎中搜索 *liuwons* : ```liuwons site:zhihu.com```

### 精确匹配

使用双引号来指定精确匹配单词或短语

如精确搜索 *liuwons* : ``` "liuwons" ```

### 模糊搜索

使用星号代替一个单词进行模糊搜索
例如```"a * saved is a * earned"```会搜到如下结果：

*A penny saved is a penny earned*

### 指定索搜结果不包含某些内容

使用减号指定搜索结果中不包含某些内容

如 ```liuwons -site:github.com``` 将不包含 github上的内容

`liuwons -上海` 将不包含内容中含有 *上海* 的网页

### 指定文件类型

使用 **filetype** 指定文件类型搜索

如 ```effective java filetype:pdf``` 搜索 *effective java* 的电子书

### 标题搜索

使用 **intitle** 限定筛选出标题中含有特定关键字的网页

例如 `liuwons intitle:wechat`  搜索 *liuwons* 并筛选出标题中含有 *wechat* 的网页


### 参考资料

[百度用户服务中心: 百度搜索技巧](http://help.baidu.com/question?prod_id=1&class=553)

[Google Help Center: Google Search Help](https://support.google.com/websearch/answer/2466433?hl=en&ref_topic=3081620)

[知乎: 如何高效地使用搜索引擎](https://www.zhihu.com/question/28013848)

[知乎: 如何用好 Google 等搜索引擎](https://www.zhihu.com/question/20161362)
