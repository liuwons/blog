---
layout: post
title: 三种知识图谱
date: 2016-01-13 23:21:03
tags: [Knowledge Graph]
categories: tech
---

知识图谱本质上是一种语义网络。其结点代表实体（entity）或者概念（concept），边代表实体/概念之间的各种语义关系。**Knowledge Graph** , **Freebase** , **Wikidata** 是目前最常见的三种知识图谱。

## [Knowledge Graph](https://developers.google.com/knowledge-graph/)

**Knowledge Graph** 是Google的一个知识库，其使用语义检索从多种来源收集信息，以提高Google搜索的质量。**Knowledge Graph**  2012年加入Google搜索，2012年5月16日正式发布，首先可在美国使用。**Knowledge Graph**  除了显示其他网站的链接列表，还提供结构化及详细的关于主题的信息。其目标是，用户将能够使用此功能提供的信息来解决他们查询的问题，而不必导航到其他网站并自己汇总信息。

### 1.搜索api

**Knowledge Graph** 提供了查询api，官方文档见[API Reference](https://developers.google.com/knowledge-graph/?hl=en)。
可以直接使用HTTP GET进行查询，如使用以下url查询与 **Fudan** 关的实体：
[https://kgsearch.googleapis.com/v1/entities:search?query=Fudan&key=](https://kgsearch.googleapis.com/v1/entities:search?query=Fudan&key=)
注意url中的key字段为开发者账号申请的api调用key。

### 2.搜索结果
用户可以指定返回的查询结果格式，json格式的内容如下：

```json
{
	@context:
	{
		@vocab: "http://schema.org/",
		goog: "http://schema.googleapis.com/",
		EntitySearchResult: "goog:EntitySearchResult",
		detailedDescription: "goog:detailedDescription",
		resultScore: "goog:resultScore",
		kg: "http://g.co/kg"
	},
	@type: "ItemList",
	itemListElement:
	[
		{
			@type: "EntitySearchResult",
			result:
			{
				@id: "kg:/m/0jktd",
				name: "Fudan University",
				@type:
				[
					"CollegeOrUniversity",
					"Organization",
					"EducationalOrganization",
					"Place",
					"Thing"
				],
				description: "University in Shanghai, China",
				image:
				{
					contentUrl: "http://t3.gstatic.com/images?q=tbn:ANd9GcRL6bWR-Z8BDYVYytbMaXJiTa8l690RY2pwpAbj7EvIlRgrDb97",
					url: "https://commons.wikimedia.org/wiki/File:Fudan-logo.jpg",
					license: "http://creativecommons.org/licenses/by-sa/3.0"
				},
				detailedDescription:
				{
					articleBody: "Fudan University, located in Shanghai, China, is one of the most prestigious and selective universities in China, and is a member in the C9 League and Universitas 21. ",
					url: "http://en.wikipedia.org/wiki/Fudan_University",
					license: "https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License"
				},
				url: "http://www.fudan.edu.cn/"
			},
			resultScore: 40.484966
		}
	]
}
```

其中 **@id** 字段为对应的entity在 **Freebase** 中的mid。

### 3.api特点

优点：

 - 查询简单便捷
 - 查询结果可以指定以json等格式返回
 - entity中的大部分信息是直接显示在Google相关搜索的右侧栏wiki中的，质量较高并且相对比较丰富
 - 每个账户的免费额度为每天100,000次查询，能够满足大部分使用场景

缺点：

 - 不能直接得到与其相关联的其他entity信息

## [Freebase](https://www.freebase.com)

**Freebase** 是一个由元数据组成的大型合作知识库，内容主要来自其社区成员的贡献。它整合了许多网上的资源，包括部分私人wiki站点中的内容。**Freebase** 致力于打造一个允许全球所有人（和机器）快捷访问的资源库。它由美国软件公司Metaweb开发并于2007年3月公开运营。2010年7月16日被谷歌收购。 2014年12月16日，Google宣布将在六个月后关闭 **Freebase** ，并将全部数据迁移至 **Wikidata** 。

### 1.搜索api

官方文档见[API Reference](https://developers.google.com/freebase/)。
可以直接使用HTTP GET进行查询，如使用以下url查询与 **Fudan** 相关的实体：
[https://www.googleapis.com/freebase/v1/search?query=fudan&format=entity](https://www.googleapis.com/freebase/v1/search?query=fudan&format=entity)

### 2.搜索结果
典型的搜索结果类似如下：
```json
{
	"status":"200 OK",
	"result":
	[
		{
			"mid":"/m/0jktd",
			"id":"/en/fudan_university",
			"name":"Fudan University",
			"under":"Shanghai",
			"lang":"en","score":39.771729
		},
	],
	"cursor":20,
	"cost":4,
	"hits":543
}
```

### 3.api特点

优点：

 - 查询简单便捷
 - 查询结果以json格式返回
 - 每个账户的免费额度为每天100,000次查询，能够满足大部分使用场景

缺点：

 - 不能直接得到与其相关联的其他实体信息
 - 实体中能直接利用的信息较少

### 4.查看实体详细信息

虽然 **Freebase** 的实体查询结果中能直接利用的信息较少，不过可以通过Freebase提供的网页查看相应实体的详细信息，只需要在首页地址[https://www.freebase.com](https://www.freebase.com)后添加上对应的mid就能直接反问html格式的详细信息页面，如 **Fudan University** 的mid为m/0jktd，则其详细信息页面的url为[https://www.freebase.com/m/0jktd](https://www.freebase.com/m/0jktd)。这使得利用网络爬虫获取实体的详细信息成为可能。

### 5.Data Dumps

另外 **Freebase** 还提供完整的数据库下载，详情参考[Data Dumps](https://developers.google.com/freebase/data)。dump的数据为元组的形式，实际利用起来具有一定的挑战性。

### 6.停止开放
需要注意的是，**Freebase** 不久将停止开放，详情参考[ShutDown](https://plus.google.com/109936836907132434202/posts/bu3z2wVqcQc)。

## [Wikidata](https://www.wikidata.org)

**Wikidata** 是一个可协同编辑的知识库，是继2006年的维基学院之后，第一个新的维基媒体基金会项目。这一项目与维基共享资源的工作方式类似，将为其他维基计划及各语种维基百科中的信息框、列表及跨语言链接等提供统一存放的数据，该项目在2012年10月30日投入使用。

### 1.搜索api
官方文档见[API Reference](https://www.wikidata.org/w/api.php)。
可以直接使用HTTP GET进行查询，如使用以下url查询与 **Fudan** 关的实体：
[https://www.wikidata.org/w/api.php?action=query&list=search&srsearch=Fudan&format=json](https://www.wikidata.org/w/api.php?action=query&list=search&srsearch=Fudan&format=json)

### 2.搜索结果

典型的搜索结果类似如下：

```json
{
	"batchcomplete":"",
	"continue":
	{
		"sroffset":10,
		"continue":"-||"
	},
	"query":
	{
		"searchinfo":
		{
			"totalhits":17
		},
		"search":
		[
			{
				"ns":0,
				"title":"Q495015",
				"snippet":"universit\u00e9 <span class=\sity Universit\u00e0 >Fudan</span>-universiteit",
				"size":17783,
				"wordcount":253,
				"timestamp":"2016-01-06T21:09:34Z"
			},
		]
	}
}
```

### 3.api特点

优点：

 - 查询简单便捷
 - 查询结果可以设定以json格式返回
 - 没有查询额度限制

缺点：

 - 不能直接得到与其相关联的其他实体信息
 - 实体中能直接利用的信息较少

### 4.获取entity详细信息

除了提供实体查询接口，**Wikidata** 还提供了专门的api用于通过实体的id获取实体的详细信息，这些信息包含与其相关联的其他实体信息。
此api可以将结果以多种格式返回，例如以HTTP GET的方式获取id为Q495015的实体的详细信息并指定以json格式返回的url为：
[https://www.wikidata.org/wiki/Special:EntityData/Q495015.json](https://www.wikidata.org/wiki/Special:EntityData/Q495015.json)
另外还可以直接通过html方式展示实体详细信息，例如：
[https://www.wikidata.org/wiki/Q495015](https://www.wikidata.org/wiki/Q495015)

### 5.Database Download

**Wikidata** 提供完整的数据库下载，详见[Database Download](https://www.wikidata.org/wiki/Wikidata:Database_download)

## 对比

| 项目         				| Knowledge Graph 									  | Wikidata                        | Freebase 												|
| --------------------------- | -------------------------------------------------  | -------------------------------- | ---------------------------------------------------- |
| 额度                        | 10万/天		  									    | 不限		                     |10万/天  					   							|
| 查询结果中能直接利用的信息    | 多，大部分信息都会放在Google相关搜索的右侧wiki栏		 | 很少（只有name和id)		        |很少         										 |
| 数据获取					   | 在线api											  | 在线api + data dump				| 在线api + data dump									 |
| 获取关联实体                 | 查询能得到实体在freebase中的mid,通过此mid获取相关实体 | 可以直接查询实体详细信息得到关联实体 |通过dump的数据离线分析(复杂)或者爬虫分析实体详情页面        |
| 维护						| Google											 | Wikipedia                         | 不久将被shut down									 |
