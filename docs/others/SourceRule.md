---
title: 览阅APP书源规则
date: 2019-05-30 19:58:39
tags: 书源
---

# 

# 书源格式

- Xpath: 适用于第三方文学网站
- Json: 适用于第三方API接口



# 书源规则

> 目前Json格式规则已适配追书神器API，但由于第三方接口限制访问，建议使用Xpath语法。



## 书源地址

- 字段：rootLink
- 注释：作为书源的主键，利用了URL的唯一性
- 例子：https://www.zwdu.com
- 注意：末尾不加斜杠

## 书源名称

- 字段：sourceName
- 注释：作为书源的名称
- 例子：八一文学网
- 注意：书源名称唯一

## 书源类型

- 字段：sourceType
- 注释：xpath和json二选一
- 例子：xpath
- 注意：小写

## 编码格式

- 字段：encodeType
- 注释：指书源返回内容编码格式，该字段可不填，默认utf-8
- 例子：utf-8
- 注意：有些书源搜索利用了第三方工具，返回结果编码予整体书源编码不一致，则使用&符号分别连接整体网站编码和搜索页编码。如 gbk&utf-8

## 搜索地址

- 字段：searchLink
- 注释：搜索页面地址，%s为关键字占位符
- 例子：/search.php?keyword=%s
- 注意：以斜杠开始，用于和书源地址拼接成一个完整的搜索地址



---



## 搜索结果列表

- 字段：ruleSearchBooks
- 注释：主要用于Json格式中搜索结果定位。若为多层结构，则使用$符号拼接Json的键名
- 例子：books
- 注意：xpath格式中选填，若填写则将会与后面搜索结果的其他规则拼接成一个完整的xpath路径

## 搜索结果名称

- 字段：ruleSearchTitle
- 注释：搜索书籍的书名
- 例子：//div[@class='result-game-item-detail']//h3//a/@title
- 注意：

## 搜索结果作者

- 字段：ruleSearchTitle
- 注释：搜索书籍的作者
- 例子：//div[@class='result-game-item-info']//p[1]/span[2]/text()
- 注意：

## 搜索结果简介

- 字段：ruleSearchAuthor
- 注释：搜索书籍的简介；选填，因为结果页面可能没有该内容
- 例子：//p[@class='result-game-item-desc']/text()
- 注意：

## 搜索结果封面

- 字段：ruleSearchCover
- 注释：搜索书籍的封面，选填
- 例子：//div[@class='result-game-item-pic']//a//img/@src
- 注意：

## 搜索结果地址

- 字段：ruleSearchLink
- 注释：搜索书籍的详情地址
- 例子：//div[@class='result-game-item-detail']//h3//a/@href
- 注意：



---



## 书籍详情地址

- 字段：ruleDetailBook
- 注释：主要用于Json格式中的定位
- 例子：books
- 注意：xpath格式中选填，若填写则将会与后面搜索结果的其他规则拼接成一个完整的xpath路径

## 书籍详情名称

- 字段：ruleDetailTitle
- 注释：定位书籍详情页的名称
- 例子：//div[@id='info']/p[1]/text()
- 注意：选填，不填则使用搜索结果中对应的内容

## 书籍详情作者

- 字段：ruleDetailAuthor
- 注释：定位书籍详情页的名称
- 例子：//div[@id='info']/p[1]/text()
- 注意：选填，不填则使用搜索结果中对应的内容

## 书籍详情简介

- 字段：ruleDetailAuthor
- 注释：定位书籍详情页的简介，用于补充搜索结果没有的内容
- 例子：//div[@id='info']/p[1]/text()
- 注意：选填，不填则使用搜索结果中对应的内容

## 书籍详情封面

- 字段：ruleDetailCover
- 注释：定位书籍详情页的封面地址，用于补充搜索结果没有的内容
- 例子：//div[@id='info']/p[1]/text()
- 注意：选填，不填则使用搜索结果中对应的内容

## 书籍发现地址

- 字段：ruleFindLink
- 注释：定位书籍详情页的推荐书籍地址
- 例子：//div[@id='info']/p[1]/text()
- 注意：选填，不填则使用搜索结果中对应的内容

## 书籍目录地址

- 字段：ruleCatalogLink
- 注释：定位书籍详情页的目录地址
- 例子：//div[@id='info']/p[1]/text()
- 注意：选填，不填代表目录地址即为书籍详情地址



---



## 目录章节列表

- 字段：ruleChapters
- 注释：主要用于Json格式中目录定位
- 例子：//div[@id='list']
- 注意：同搜索结果列表

## 目录章节名称

- 字段：ruleChapterTitle
- 注释：定位目录页面每个章节名称
- 例子：//dl//dd//a/text()
- 注意：

## 目录章节地址

- 字段：ruleChapterLink
- 注释：定位目录页面每个章节地址
- 例子：//dl//dd//a/@href
- 注意：



---



## 章节内容

- 字段：ruleChapterContent
- 注释：用于定位书籍章节页面中的内容部分
- 例子：//div[@id='content']
- 注意：



---



## 发现书籍列表

- 字段：ruleFindBooks
- 注释：解析推荐书籍的列表，同搜索结果列表
- 例子：
- 注意：选填

## 发现书籍名称

- 字段：ruleFindBookTitle
- 注释：定位推荐书籍的名称
- 例子：//div[@id='listtj']//a/text()
- 注意：选填

## 发现书籍作者

- 字段：ruleFindBookAuthor
- 注释：定位推荐书籍的作者
- 例子：//div[@id='listtj']//a/text()
- 注意：选填

## 发现书籍封面

- 字段：ruleFindBookCover
- 注释：定位推荐书籍的封面
- 例子：//div[@id='listtj']//a/@href
- 注意：选填

## 发现书籍地址

- 字段：ruleFindBookCover
- 注释：定位推荐书籍的详情地址
- 例子：//div[@id='listtj']//a/@href
- 注意：选填



# 书源示例

## Json

```json
 {
    "rootLink": "http://api.zhuishushenqi.com",
    "sourceName": "追书神器",
    "sourceType": "json",
    "encodeType": "utf-8",
    "searchLink": "/book/fuzzy-search?query=%s",
    "ruleSearchBooks": "books",
    "ruleSearchTitle": "title",
    "ruleSearchAuthor": "author",
    "ruleSearchDesc": "shortIntro",
    "ruleSearchCover": "cover@http://statics.zhuishushenqi.com%s",
    "ruleSearchLink": "_id@http://api.zhuishushenqi.com/book/%s",
    "ruleCatalogLink": "_id@http://api.zhuishushenqi.com/mix-atoc/%s?view=chapters",
    "ruleFindLink": "_id@http://api.zhuishushenqi.com/book/%s/recommend",
    "ruleDetailDesc": "longIntro",
    "ruleChapters": "mixToc$chapters",
    "ruleChapterLink": "link@UrlEncode#http://chapterup.zhuishushenqi.com/chapter/%s",
    "ruleChapterTitle": "title",
    "ruleChapterContent": "chapter$body"
  }
```



## Xpath

```json
{
    "rootLink": "https://www.zwdu.com",
    "sourceName": "八一中文",
    "sourceType": "xpath",
    "encodeType": "gbk&utf-8",
    "searchLink": "/search.php?keyword=%s",
    "ruleSearchTitle": "//div[@class='result-game-item-detail']//h3//a/@title",
    "ruleSearchAuthor": "//div[@class='result-game-item-info']//p[1]/span[2]/text()",
    "ruleSearchDesc": "//p[@class='result-game-item-desc']/text()",
    "ruleSearchCover": "//div[@class='result-game-item-pic']//a//img/@src",
    "ruleSearchLink": "//div[@class='result-game-item-detail']//h3//a/@href",
    "ruleDetailAuthor": "//div[@id='info']/p[1]/text()",
    "ruleDetailDesc": "//div[@id='intro']/p[1]/text()",
    "ruleDetailCover": "//div[@id='fmimg']/img/@src",
    "ruleChapters": "//div[@id='list']",
    "ruleChapterLink": "//dl//dd//a/@href",
    "ruleChapterTitle": "//dl//dd//a/text()",
    "ruleChapterContent": "//div[@id='content']",
    "ruleFindBookLink": "//div[@id='listtj']//a/@href",
    "ruleFindBookTitle": "//div[@id='listtj']//a/text()"
  }
```

