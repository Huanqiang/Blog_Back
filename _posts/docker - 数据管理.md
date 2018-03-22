---
title: docker - 数据管理
author: Huanqiang
tags: [Docker, Docker 数据管理]
categories: [Docker]
keyword: [Docker, Docker 数据管理]
date: 2018-01-30 09:31:49
---

不建议将数据存储在容器的数据存储层中，因为容器可能被随时删除、重建。所以推荐的 Docker 的数据管理有两种方式：

1. Docker 建立的数据卷；
2. 挂载主机目录；

> 我们知道 Docker 在容器创建时会在其上创建一层存储层，但是容器存储层的生命周期是随着容器的，容器被销毁时，存储层也会销毁，此时任何存在容器存储层上的数据都会随着容器的删除而丢失。

<!-- more -->

### 1. 数据卷

数据卷是一个**独立于容器的特殊目录**，它能在**多个容器之间共享和复用**，对数据卷的修改会立刻生效，而且对数据卷的更新也不会影响容器，最重要的是，数据卷**默认会一直存在**，即使容器被删除了。

> 数据卷是由 Docker 命令建立的，存在于 Docker 里，而不是在宿主机中的目录。

#### 创建数据卷

```
docker volume create my-volume
```

#### 查看数据卷

```
docker volume ls
```

查看指定数据卷

```
docker volume inspect volume-name
```

如果我们将数据加载至容器后，如果我们使用 `docker inspect docker-name` 查看容器信息的话，还能在容器信息的 “Mounts” key值下看到对应的数据卷信息。举个例子：

```
"Mounts": [
	{
 		"Type": "volume",
      	"Name": "my-vol",
      	"Source": "/var/lib/docker/volumes/my-vol/_data",
      	"Destination": "/app",
      	"Driver": "local",
      	"Mode": "",
      	"RW": true,
      	"Propagation": ""
    }
],
```

#### 连接数据卷

有两个参数可以将容器连接至数据卷：`-v` 和 `--mount` 。

```
docker run -d -P --name web --mount source=my-vol,target=/webapp nginx
```

这里的 `--mount source=my-vol,target=/webapp` 就是将（`source`）数据卷 my-vol 加载到容器的 `/webapp` 目录（`target`）。

使用 -v 的话，将 `--mount source=my-vol,target=/webapp` 改成 `-v my-vol:/webapp` 即可，如下：

```
docker run -d -P --name web -v my-vol:/webapp nginx
```

#### 删除数据卷

```
docker volume rm volume-name
```

因为数据库默认是一直存在的，Docker 也没有提供自动回收机制来处理没有被容器使用的数据卷（因为不知道你以后会不会还是用这个数据卷，所以默认不会删除）。所以我们在删除了容器后，很可能忘记删除没用了的数据卷，那么这些数据卷会占用很大的空间。

一次性清理多个无主数据卷：

```
docker volume prune
```

### 2. 挂载主机目录

#### 挂载一个主机目录作为数据卷

和数据卷一样，有两种参数：`-v` 和 `--mount`。 还是推荐使用 `--mount`。

```
docker run -d -P --name web --mount type=bind,source=/src/web,target=/opt/webapp nginx
```

这个命令将主机目录 `/src/web` 挂载到容器的 `/opt/webapp` 目录下。

> 请注意：这里的两个目录路径都是**绝对路径**。而且主机目录一定要存在，如果不存在就会报错。

Docker 默认的挂载的主机目录是可读写的，如果你想将主机目录设置成只读，那么只需要加一个 readonly 即可：`--mount type=bind,source=/src/web,target=/opt/webapp,readonly`。

如果使用 `-v` 参数，将 `--mount type=bind,source=/src/web,target=/opt/webapp` 替换成 `-v /src/web:/opt/webapp` 就好了：

```
docker run -d -P —name web -v /src/web:/opt/webapp nginx
```

#### 挂载一个本地主机文件作为数据卷

```
docker run -d -P --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history nginx
```

如果使用 `-v`，则如下：

```
docker run -d -P -v $HOME/.bash_history:/root/.bash_history nginx
```

#### 查看容器中的数据卷信息

这个和数据卷的操作一样：先使用 `docker inspect container-name` 查看容器信息，再查看容器信息中 “Mounts” 关键字所对应的值即可。