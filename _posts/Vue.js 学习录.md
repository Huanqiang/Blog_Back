---
title: 从0开始搭建基于 Nodejs 的 Vue 工程
author: Huanqiang
tags: [vue]
categories: [前端]
keyword: [vue]
date: 2017-08-22 10:31:49
---

## 何为 Vue.js

Vue.js 是一套构建用户的渐进式框架，将界面和其状态相互映射，这样当界面数据发生变化时，试图能够自动更新。Vue.js 本身只做了这一块，其他的 router、vuex 都是额外的插件。

> 渐进式框架：为不同复杂度的场景提供不同复杂度的工具集；就是说它为使用者提供了很多的可选择性，用户一上来就能使用，需要的时候，还能使用官方或者生态圈内的其他插件；

<!-- more -->

## Virtual DOM

Virtual DOM 是实现 Vue.js 响应式更新试图的关键。这一点我们看 [官方文档](https://cn.vuejs.org/v2/guide/reactivity.html#%E5%A6%82%E4%BD%95%E8%BF%BD%E8%B8%AA%E5%8F%98%E5%8C%96) 就能发现。

那么 Virtual DOM 是一个什么东西呢？简单地说，Virtual DOM 就是用 JS 对象表示了整个 DOM 树。

### 算法实现

关于 Virtual DOM 的算法实现主要分为了三步：

1. 将 DOM 树用 JS 对象来表示；
2. 将改变后的 Virtual DOM 和原 Virtual DOM 进行对比（深度优先搜索），那么两颗树差异的地方就是改变了的地方；
3. 将差异的地方应用到真正的 DOM 树上；

> 详见资料2；

## 资料

1. [Vue作者尤雨溪：Vue 2.0，渐进式前端解决方案](https://mp.weixin.qq.com/s?__biz=MzIwNjQwMzUwMQ==&mid=2247484393&idx=1&sn=142b8e37dfc94de07be211607e468030&chksm=9723612ba054e83db6622a891287af119bb63708f1b7a09aed9149d846c9428ad5abbb822294&mpshare=1&scene=1&srcid=1026oUz3521V74ua0uwTcIWa&from=groupmessage&isappinstalled=0#wechat_redirect&utm_source=tuicool&utm_medium=referral)
2. [深度剖析：如何实现一个 Virtual DOM 算法](https://github.com/livoras/blog/issues/13)