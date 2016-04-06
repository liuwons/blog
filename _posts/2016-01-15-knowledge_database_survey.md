---
layout: post
title: 开放知识库调研
date: 2016-01-15 15:17:28
tags: [Knowledge Graph]
categories: tech
---


目前调研到可用的开放知识库包括：**Knowledge Graph**, **Freebase**, **Wikidata**。下文描述能够获取的数据以及对应最方便的接口方式。

<!-- more -->

## 1 Knowledge Graph

### 1.1 关键字搜索接口

接口方式: HTTP GET

数据格式: json

数据内容:

 - mid: **Freebase** 实体id，能通过此id访问实体在 **Freebase** 中的信息。
 - name: 实体名称。
 - type: 实体类型。
 - description: 实体的一句话简短描述。
 - image: 描述实体的一幅图片，如人物的照片，机构的徽章等。
 - detailed description: 比较详细的介绍文章，包含摘要以及文章的url，文章大部分来自wikipedia。

## 2 Freebase

提供关键字搜素接口，并提供html格式的实体信息页面。

### 2.1 关键字搜索

接口方式: HTTP GET

数据格式: json

数据内容：

 - name: 实体名称
 - mid: **Freebase** 实体id

可以参考[Freebase搜索Beijing](https://www.googleapis.com/freebase/v1/search?query=beijing&format=entity)。

### 2.2 实体信息页面

接口方式: HTTP GET

数据格式: html

实体信息页面以html格式提供实体的详细信息，还包含很多的相关实体以及实体关系。但是由于信息结构化程度低，并且不同种类实体提供的信息也不一样，因此分析困难。

比较一般性的信息包含：

 - name: 实体名称。
 - description: 实体描述，一般来自wikipedia，附有资源的url。
 - alias: 实体的其他别名。
 - image: 描述实体的图片。
 - topic: 与实体相关的一些文章。

其他具体的内容依据相应实体的类别而异。例如机构类实体可能包含：

 - 官方网站
 - 地理位置
 - 电话号码
 - 员工信息

名人类实体可能包含：

 - 出生时间
 - 死亡时间
 - 国籍
 - 家庭关系

数据内容可以参考[Freebase Beijing 信息页](https://www.freebase.com/m/01914)。

## 3 Wikidata

提供关键字搜索接口，并能依据id进行实体详细信息查询。

### 3.1 关键字搜索

接口方式: HTTP GET

数据格式: html

数据内容: 只包含相应实体在 **Wikidata** 中的id。

### 3.2 实体详细信息查询

依据实体的 **Wikidata** id查询其详细信息。

接口方式: HTTP GET

数据格式: 可以指定html或者json

数据内容: 以实体以及关系描述的实体详细信息。html格式为人类可读的页面，json格式是类似于三元组描述的实体关系数据。


html格式的数据可以参考[Wikidata html: Beijing](https://www.wikidata.org/wiki/Q956)，json格式的数据可以参考[Wikidata json: Beijing](https://www.wikidata.org/wiki/Special:EntityData/Q956.json)。
