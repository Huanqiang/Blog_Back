---
title: RESTful 风格的API
author: Huanqiang
tags: [RESTful API]
categories: [前端]
keyword: [RESTful, RESTful API, API 设计]
date: 2017-01-06 10:31:49
---

## RESTful 风格的API

规划一个好的API的样式风格，要先于开发它实际的功能。首先你要知道数据该如何设计，核心服务/应用程序会如何工作。如果你纯粹新开发一个API，这样会比较容易一些。但如果你是往已有的项目中增加API，你可能需要提供更多的抽象。

有时候一个资源集合可以表达一个数据库表，而一个资源可以表达成里面的一行记录，但是这并不是常态。事实上，你的API应该尽可能通过抽象来分离数据与业务逻辑。这点非常重要，只有这样做你才不会打击到那些拥有复杂业务的第三方开发者，否则他们是不会使用你的API的。

<!-- more -->

当然你的服务可能很多部分是不应该通过API暴露出去的。比较常见的例子就是很多API是不允许第三方来创建用户的。

## HTTP 动词

关于 http 动词，就是指 `GET`、`POST` 等操作。也是通过这些http动词，我们才能对服务器中的资源进行操作。常用的HTTP动词有下面五个（括号里是对应的SQL命令）：

```
GET（SELECT）：从服务器获取资源，没有副作用，是幂等的；
POST（CREATE）：在服务器新建一个资源，非幂等；
PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源），幂等；
PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）；
DELETE（DELETE）：从服务器删除资源，有副作用，但满足幂等性；
```

> 值得注意的是，一个好的 API， 那么它**只能**用这几个动词进行数据交互，而不能在其URL段中使用其他动词，只能使用名词。
>
> 幂等：无边际效应，多次操作得到相同的结果。 详细参见 `参考4`

## 路径（Endpoint）

路径又称”终点”（endpoint），表示API的具体网址。

在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的”集合”（collection），所以API中的名词也应该使用复数。

举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样。

```
* https://api.example.com/v1/zoos
* https://api.example.com/v1/animals
* https://api.example.com/v1/employees
```

## 过滤（Filtering）

过滤操作是基于 [http 动作](http://huanqiang.wang/2017/01/06/RESTful-%E9%A3%8E%E6%A0%BC%E7%9A%84API/#HTTP) 的，http操作是对资源的一系列操作，那么 `filtering` 就是对获取到的资源进行过滤，返回需要的结果。

举几个栗子吧：

```
* ?limit=10：指定返回记录的数量；
* ?offset=10：指定返回记录的开始位置；
* ?page=2&per_page=100：指定第几页，以及每页的记录数；
* ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序；
* ?animal_type_id=1：指定筛选条件；
```

> 其实，这个过滤和上面的 GET 操作在设计上有些冗余，比如 `GET /zoo/ID/animals` 与 `GET /animals?zoo_id=ID`，这两个操作返回的结果是一样的。

## 版本（Versioning）

如果你的项目是一个持续更新的，那么随着时间的推移，你总是要更新你的项目功能，那么当你有大功能更新的话，就应该做一个版本的更新。

在 `RESTful` 风格的API中，我们应该将API的版本号放入URL中。

```
https://api.example.com/v1/
```

当然还有另外一种做法，就是将版本号放在HTTP头信息中，但不如放入URL方便和直观。

## 状态码

- `200` OK - [GET]：从服务器请求数据，服务器成功查询到这些数据，并返回；（幂等）
- `201` CREATED - [POST/PUT/PATCH]：用户向服务器提供数据，服务器创建资源；
- `204` NO CONTENT – [DELETE]：用户要求服务器删除一个资源，服务器成功删除了它；
- 400 INVALID REQUEST – [POST/PUT/PATCH]：消费者给了服务器坏数据让服务器去查询它，而服务器没有；（幂等）
- `404` NOT FOUND – [*]：消费者向服务器发出资源请求（查找或者其他操作），而服务器什么都没有做（可能是这个资源根本就是不存在的）；（幂等）
- `500` INTERNAL SERVER ERROR – [*]：服务器遇到错误，即使请求是成功发出了，但用户没有收到结果。

## 参考

1. [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)：本文的参考文档之一
2. [RESTful API 设计参考文献列表，可帮助你更加彻底的了解REST风格的接口设计。](https://github.com/aisuhua/restful-api-design-references)：包含了很多关于 REST 架构的文章
3. [好RESTful API的设计原则](http://www.cnblogs.com/moonz-wu/p/4211626.html)：本文的参考文档之一，写得非常好
4. [理解HTTP幂等性](http://www.cnblogs.com/weidagang2046/archive/2011/06/04/2063696.html)：重要
5. [Using HTTP Methods for RESTful Services](http://www.restapitutorial.com/lessons/httpmethods.html)：关于Http动词的含义，及其对应状态码