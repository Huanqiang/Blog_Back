---
title: Docker 各部件概念及其关系总结
author: Huanqiang
tags: [Docker, Docker Engine, Docker Compose, Docker Machine, Docker Swarm]
categories: [Docker]
keyword: [Docker, Docker Engine, Docker Compose, Docker Machine, Docker Swarm]
date: 2018-02-02 10:31:49
---

本文力求以最简单的描述来表述清楚 Docker 中各个组件（`Docker Engine`、`Docker Compose`，Docker Machine, `Docker Swarm`）是什么及其关系。

> 未待完续…

<!-- more -->

### Docker Engine

我们平时所说的 `Docker` 一般指的是 `Docker Engine`，比如在 `Mac` 上安装的 `Docker for Mac`。它是由 `Docker 守护进程`、REST API 和 `命令行接口客户端` 这三个部分组成。一个物理机或是虚拟机安装了 `Docker Engine` 之后就成为了一个 `Docker 主机`（`Docker host`）。

> Docker Engine 还内置了 `Docker Compose`、`Docker Machine` 和 `Docker Swarm Mode`。（这句话不一定准确，Docker for Mac 确实是内置了，但是 Linux 还真不一定。

### Docker Compose

`Docker Compose` 是能帮助用户快速部署一个基于 `Docker` 的分布式项目。`Docker Compose` 是面向项目的。

一个 `Docker Compse` 模板文件包含了该项目所需的所有服务。然后 Docker Compose 帮助你将这个项目部署到一个 `Docker` 主机中，并且为每一个服务创建一个容器，同一个服务可以创建多次，按照 `项目名-服务名-序号`格式命名 。

### Docker Machine

`Docker Machine` 是用来快速搭建 `Docker` 主机的。比如说你先要为 20 台云服务器安装 `Docker Engine`，那么一台一台安装就会很耗时间。利用 `Docker Machine` 你就能在你本地通过 `docker-machine create` 命令快速地将这些服务器搭建成 `Docker Host`。

### Docker Swarm Mode

Docker Swarm Mode 是基于 Swarmkit 实现的，能帮助用户快读搭建一个基于 Docker 的集群。它提供了对集群节点的编排和管理。

一个集群中可以有多个管理节点和工作节点，但是**负责集群节点编排的管理节点只能有一个**，通常称之为 `Leader`。用户可以通过管理节点分发任务至工作节点，工作节点能通过内部的一个代理返回任务的运行情况。

> 一个管理节点默认就是一个工作节点，会运行任务，当然你也可以将其设置成仅运行管理功能的节点。

每一个节点都是一个 `Docker Host`，而每一个任务则是一个容器。一个服务由多个任务组成，每个任务可以运行于不同节点的容器之中。

### Docker Compose 和 Docker Swarm Mode 的组合使用

> 未待完续…

在 `Swarm` 集群中，我们使用 docker service create 一次只能配置一个服务。然而我们如果将 Docker Compose 和 Docker Swarm Mode 一起使用的话，可以通过 `docker-compose.yml` 文件来配置、启动多个关联服务。

> 在 `docker-compose.yml` 文件中，我们可以通过 `deploy` 指令来完成这项工作。