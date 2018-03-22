---
title: 搭建私有npm源--Sinopia
author: Huanqiang
tags: [npm, sinopia]
categories: [前端]
keyword: [npm, sinopia, nodejs]
date: 2017-03-01 08:31:49
---

## 服务器端

### 安装

首先，我们配置一下 `node.js` 和 `npm` 的环境。

再安装 [`sinopia`](https://github.com/rlidwka/sinopia) 或者 [`verdaccio`](https://github.com/verdaccio/verdaccio)。

```bash
npm install -g sinopia
```

> verdaccio：因为 `sinopia` 不再更新了，所以它 fork 了一份，并持续更新。其配置与 `sinopia` 一致。

<!-- more -->

### 运行

可以通过以下语句直接运行：

```bash
sinopia
```

或者 可以使用 `pm2` 或者其他守护进程进行管理。

`pm2` 的启动方式如下：

```bash
pm2 start which sinopia
```

> 无论是 `pm2` 还是 `forever` 都很坑，可能会启动出错。
> 如果上面那句启动失败，试试这个： `pm2 start which sinopia`；

### 配置

参见：<https://github.com/rlidwka/sinopia/blob/master/conf/full.yaml>

## 客户端

推荐使用 `nrm` 来管理 npm 的源。

### npm 源的管理：nrm

#### 安装

```bash
npm install -g nrm
```

#### 命令集合

```bash
ls                           List all the registries
current                      Show current registry name
use <registry>               Change registry to registry
add <registry> <url> [home]  Add one custom registry
del <registry>               Delete one custom registry
home <registry> [browser]    Open the homepage of registry with optional browser
test [registry]              Show response time for specific or all registries
help                         Print this help
```

> 在使用 `nrm add` 的时候要注意加上 `http://`.

### 使用

1. 使用 `nrm` 切换至我们的私有源上；
2. 使用 `npm login` 登录私有源；
3. 使用 `npm publish` 命令发布新包；