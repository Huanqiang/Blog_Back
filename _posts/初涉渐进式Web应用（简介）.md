---
title: 初涉渐进式Web应用（简介）
author: Huanqiang
tags: [前端, 渐进式Web应用]
categories: [前端]
keyword: [前端, PWA, 渐进式Web应用]
date: 2018-02-27 10:31:49
---

## What is Progressive Web Apps？

在移动端，我们习惯于浏览和使用原生 `App`，而很少使用手机自带的 `Web` 浏览器，比如 `iOS` 的 `Safari`。因为使用浏览器的话，我们还要输入网站地址，或者搜索到该网站，因为 `Web App` 不会像原生 `App` 一样出现在桌面上，这就是 **Web App 的入口问题**，而这也会让大部分用户望而却步。不仅如此，还存在一个更为重要的问题：**网络延迟所造成数据加载问题**，在原生 App 中，我们一打开应用就能看到操作界面，而 Web App 则不行，它必须要经过网络加载获取到 `HTML`、`JS`、`CSS` 后，才能显示操作页面，这期间用户就要忍受网络加载时的白屏界面。

<!-- more -->

而随着 `Progressive Web Apps` 及其相关技术的出现，`Web App` 就能很好的解决这两个问题。首先，它能让 `Web App` 的入口像原生 `App`一样出现在主界面。其次就是它能在离线环境下使用的 `Service Worker` 与 `Cache Storage`，将之前加载的数据保存在本地，从而让用户下一次打开或离线环境时无需忍受白屏加载，能直接看见和使用之前的数据，它也能提高应用的加载速度，从而提高用户体验。然后， `Progressive Web Apps` 能进一步提高操作系统的集成能力，让 `Web App` 能在未激活时发送推送通知的 `Push API` 和 `Notification API` 等。

也就是说`Progressive Web Apps` 能让 `Web App` 获得与原生 App 一样的用户体验，成为移动端的一等公民。

## 核心概念

- **响应式**: UI 自适应设备屏幕大小
- **类 APP 体验**: 它不应该感觉像网站，而应该（尽可能地）更像 APP
- **离线支持**: 它将使用设备存储以提供离线体验
- **可安装的**: 设备浏览器可提示用户安装你的 APP
- **重新参与**: 推送通知以帮助用户在安装之后重新探索你 APP 中功能
- **可发现的**: 搜索引擎及 SEO 优化比应用商店可提供更多用户
- **新颖**: 一旦 APP 是在线的，它会更新自身及其内容
- **安全**: 它使用 HTTPS
- **渐进式**: 它可以在任何设备上工作，包括老旧设备，即使功能受限（例如，只是和网站类似，不可安装）
- **可链接的**: 使用 URL 时非常易于指向

## 关键技术

### Service Workers

简单地说，Service Worker 是一个特殊的 `Web Worker`，它是一个与运行在 `worker context` 中的网页相关的 `JS` 文件。它是**网页与网络之间的可编程代理，可拦截和缓存网络请求，从而有效地为应用创建离线优先体验**。配合随之引入 `Cache Storage API`，你可以自由管理 `HTTP` 请求文件粒度的缓存，这使得 `Service Worker` 可以从缓存中向 `web` 应用提供资源。

> 简单地说， Service Worker 就是为渐进式 Web 应用提供了离线用户体验。

### App Mainfest

App Mainfest 是一个 JSON 文件，里面包含了需要暴露给浏览器的元数据，比如名称，icon 的 URL，描述，应用旋转方向等等信息。

只需要在网站的每个页面头部添加一个 mainfest 连接即可：

```
<link rel="mainfest" href="/mainfest.json">
```

> [Web App Mainfest](https://www.w3.org/TR/appmanifest/)

## 其他替代方案

这里聊一聊其他的构建移动端应用的方案。

### 原生 App

这是最直接的一个方案，`iOS` 你可以使用 `Swift` 或是 `Objective-c` 去构建，安卓你可以使用 `Java` 或者 `Kotlin` 构建。毫无疑问，构建出来的原生 `App` 是这些方案中用户体验最好的。

但是鱼和熊掌不可兼得，如果你的 `App` 需要横跨 `iOS` 和 安卓两个平台，那么很不幸，你必须要学习至少两门编程语言，掌握不同的开发流程，同时还要熟悉各个平台的 `UI` 风格。

### Hybrid（混合） App

混合 App 使用 Web技术编写应用，然后通过中间技术在构建的时候生成不同平台的代码，并打包成各个平台的安装包。所以混合 App 只需要编写一次就能随处可用。

而混合App 的不足之处则在于我们不能通过它构建各个平台特定的用户体验。

### React Native

参见 《 [[置顶\] React Native运行原理解析](http://blog.csdn.net/xiangzhihong8/article/details/52623852) 》

### 参考资料

1. [几分钟快速读懂渐进式 Web 应用 PWA](https://www.oschina.net/translate/an-introduction-to-progressive-web-apps?lang=chs&page=2#)
2. [下一代 Web 应用模型 — Progressive Web App - 黄玄](https://zhuanlan.zhihu.com/p/25167289)