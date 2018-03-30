---
title: Vue-嵌套路由
author: Huanqiang
tags: [Vue, Vue-router]
categories: [前端]
keyword: [Vue, Vue-router]
date: 2017-02-07 08:31:49
---

其实关于 [嵌套路由](https://router.vuejs.org/zh-cn/essentials/nested-routes.html) 官网上已经说得很清楚了，自己再来一下总结好了。

首先，举个栗子来说一个什么是嵌套路由，看下面这张图，很显然，这个网页是由两层导航构成的，那么我们如何想看一个内容，就必须经过两层导航栏的选择。

<!-- more -->

![图标的两种状态](/img/Nested-Route/Nested-Route.png)

[图标的两种状态](http://huanqiang.wang/img/Nested-Route/Nested-Route.png)

那么我们如何去构造这样的两层甚至多层导航呢？很简单，感谢 [vue-router](https://router.vuejs.org/zh-cn/)，它为我们提供了解决方案： **嵌套路由**。

关于示例请详见官方。