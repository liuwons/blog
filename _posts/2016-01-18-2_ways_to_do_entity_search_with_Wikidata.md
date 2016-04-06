---
layout: post
title: 用Wikidata做实体搜索的两种方案
date: 2016-01-18 23:33:25
tags: [Knowledge Graph]
categories: tech
---


**Wikidata** 是一个可协同编辑的知识库，是继2006年的维基学院之后，第一个新的维基媒体基金会项目。这一项目与维基共享资源的工作方式类似，将为其他维基计划及各语种维基百科中的信息框、列表及跨语言链接等提供统一存放的数据，该项目在2012年10月30日投入使用。

**Wikidata** 的所有数据都是对外公开的，官网对外提供了两类数据获取方式：在线API和数据库下载。在线API提供了方便的调用接口，数据库下载可以获取完整的数据库备份。

利用Wikidata做实体搜素时，针对这两类数据获取方式，相应的有两种方案：在线方法和离线方法。

<!-- more -->

## 1 在线方法
利用Wikidata提供的在线API可以很方便地实现在线实体搜索，过程可以分为三步：

 - 实体id确定
 - 实体信息获取
 - 实体信息解析
 - 相关实体信息获取

### 1.1 实体id确定
这一步利用用户输入的查询关键字确定对应的实体id。可以直接调用MediaWiki API，并且可以指定返回的数据格式(json等)。返回的数据里包含查询到的实体id。

例如搜索Fudan时，可以直接HTTP GET以下url：

[https://www.wikidata.org/w/api.php?action=wbsearchentities&search=Fudan&language=en&limit=20&format=json](https://www.wikidata.org/w/api.php?action=wbsearchentities&search=Fudan&language=en&limit=20&format=json)

### 1.2 实体信息获取
在得到实体的id之后，可以直接利用此id通过MediaWiki API获取实体的详细信息，并且可以指定返回格式。例如获取实体Q495015的详细信息可以HTTP GET以下url：

[https://www.wikidata.org/w/api.php?action=wbgetentities&ids=Q495015&format=json&languages=en](https://www.wikidata.org/w/api.php?action=wbgetentities&ids=Q495015&format=json&languages=en)

### 1.3 实体信息解析
得到指定格式的实体信息之后，需要对实体信息进行解析，具体方法可以参考第三章。

### 1.4 相关实体信息获取
解析实体信息之后，会得到与此实体相关的其他实体(实体id)以及关系属性(属性id)，通过相关实体的实体id和属性id可以进一步得到相关实体信息：相关实体信息可以直接用id查询，属性信息可以解析属性详情页。例如属性P580的详情页为[https://www.wikidata.org/wiki/Property:P580](https://www.wikidata.org/wiki/Property:P580)。

## 2 离线方法
在线方法虽然实现起来方便快捷，但是Wikidata并不能保证所有的请求都按时返回，甚至请求可能会被堵塞(参考[API:Etiquette](https://www.mediawiki.org/wiki/API:Etiquette))。因此在需要发起大量请求时在线方案不适用。

Wikidata提供了完整的数据库下载，因此可以下载完整的数据库，然后搭建自己的实体搜索服务。大致可以分为3步：

 - 数据下载
 - 数据导入
 - 搭建搜索服务

### 2.1 数据下载
Wikidata提供多种格式的数据下载，具体可以参考[Wikidata:Database download](https://www.wikidata.org/wiki/Wikidata:Database_download)。

### 2.2 数据导入
将数据导入本地数据库，如MySQL、MongoDB等。json格式的dump每行为一个实体，数据导入比较方便，但是数据量非常大（json格式的dump大小为57G，包含1800万行），数据导入将非常耗时。

### 2.3 搭建搜索服务
基于本地服务器搭建搜索服务。搜索时数据的解析可以参考第三章。用关键字搜索实体id的接口可以直接调用Mediawiki的在线API，或者自己实现。

## 3 数据解析
Wikidata存储的是实体以及实体之间的关系，具体的数据结构可以参考官方文档[Wikibase/DataModel](https://www.mediawiki.org/wiki/Wikibase/DataModel).

### 3.1 综述
典型的json格式的数据如下：

```json
Q5816: {
    "pageid": 6892,
    "ns": 0,
    "title": "Q5816",
    "lastrevid": 287405642,
    "modified": "2015-12-31T09:46:04Z",
    "type": "item",
    "id": "Q5816",
    "labels": {
        "en": {
            "language": "en",
            "value": "Mao Zedong"
        },
        "zh-hans": {
            "language": "zh-hans",
            "value": "毛泽东"
        }
    },
    "descriptions": {
        "en": {
            "language": "en",
            "value": "Chairman of the Communist Party of China"
        },
        "zh-hans": {
            "language": "zh-hans",
            "value": "中国共产党中央委员会主席"
        }
    },
    "aliases": {
        "en": [
            {
                "language": "en",
                "value": "Mao Tse-tung"
            },
            {
                "language": "en",
                "value": "Chairman Mao"
            }
        ]
    },
    "claims": {
        "P109": [
            {
                "mainsnak": {
                    "snaktype": "value",
                    "property": "P109",
                    "datavalue": {
                        "value": "Mao Zedong signature.svg",
                        "type": "string"
                    },
                    "datatype": "commonsMedia"
                },
                "type": "statement",
                "id": "Q5816$618e6d2e-43ba-5d72-16d2-fda07ffca933",
                "rank": "normal",
                "references": [
                    {
                        "hash": "167445151e65821ce4e9d2141afbb3dafb53b8e5",
                        "snaks": {
                            "P143": [
                                {
                                    "snaktype": "value",
                                    "property": "P143",
                                    "datavalue": {
                                        "value": {
                                            "entity-type": "item",
                                            "numeric-id": 30239
                                        },
                                        "type": "wikibase-entityid"
                                    },
                                    "datatype": "wikibase-item"
                                }
                            ]
                        },
                        "snaks-order": [
                            "P143"
                        ]
                    }
                ]
            }
        ],
    },
    "sitelinks": {
        "afwiki": {
            "site": "afwiki",
            "title": "Mao Zedong",
            "badges": [ ],
            "url": "https://af.wikipedia.org/wiki/Mao_Zedong"
        },
    }
}
```

### 3.2 顶级字段
json格式的entity顶级字段有：

 - id: 实体id。
 - type: 实体类型。
 - labels: 不同语言描述的实体标签。
 - descriptions: 不同语言的实体描述。
 - aliases: 不同语言描述的实体别名。
 - claims: 以属性分组的实体声明(claims)或者陈述(statements)。
 - sitelinks: 各种网站上关于此实体的描述。
 - lastrevid: 当前json文件的版本。
 - modified: 当前json文件的发布日期。


每个entity都有识别码（id）、标签（label）、描述（description）、别名（aliases），使不同的entity得以区分。而entity中的具体数据被称为claim，一个entity可以有许多 claim。

### 3.3 Claims 以及 Statements
claim 包含一条主体信息(main Snak)以及一些修饰信息(qualifier Snaks)。statement是含有参考资料（reference）的claim。每个claim总是与一个属性(property)关联(claim是关于此property的)。并且在一个实体中可以有多条claim与同一property关联。

claim含有以下字段:

 - id: 识别码，只能保证当前数据库中唯一，不包含其他信息。
 - type: claim的类型，目前只有statement和claim两种。
 - mainsnak: 如果claim含有type值，那么它具有mainsnak字段包含与property相关的主体信息。
 - rank: 表示claim是否应该显示在查询结果中，为preferred, normal 或者 deprecated.
 - qualifiers: 修饰信息，一般为主体信息的上下文信息，每一条都与一个属性(property)关联。
 - references: 如果claim是statement，那么会有一个参考资料的列表。

### 3.4 解析示例
3.1节中的json字段，除了claims顶级字段，其它信息都可以直接提取利用。

claims字段下为一个字典，字典的键为属性(property)，示例中只有一个key：P109。通过属性页[P109属性页](https://www.wikidata.org/wiki/Property:P109)可以知道此属性表示签名。

与此属性关联的claim只有一个，mainsnak为此claim的主体信息，datavalue中的value为"Mao Zedong signature.svg"，表示实体毛泽东的签名文件文件名为"Mao Zedong signature.svg"。另外此claim的type为statment，因此含有一个参考资料列表-references。mainsnak中的datavalue也可以为其它关联实体(提供实体id)。

### 3.5 解析难点

 - 属性都是以属性ID表示的，不能直接解析属性的含义
 - 关联实体是以实体ID表示的，需要多次的查询-解析。
 - 不同种类实体含有的属性不同
