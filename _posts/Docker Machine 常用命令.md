---
title: Docker Machine 常用命令
author: Huanqiang
tags: [Docker, Docker Machine, Docker Command]
categories: [Docker]
keyword: [Docker, Docker Machine, Docker Command]
date: 2018-02-02 07:31:49
---

上文我们介绍了[什么是 Docker Machine](http://huanqiang.wang/2018/01/31/Docker-Machine-%E7%AE%80%E4%BB%8B%EF%BC%88%E5%85%A5%E9%97%A8%E5%BF%85%E7%9C%8B%EF%BC%89/)，这里我们看一下 Docker Machine 的基本命令。

> 未待完续…

<!-- more -->

#### 1. 创建一个本地主机实例

```
➜  ~ docker-machine create -d virtualbox test
Running pre-create checks...
(test) Default Boot2Docker ISO is out-of-date, downloading the latest release...
(test) Latest release for github.com/boot2docker/boot2docker is v18.01.0-ce
(test) Downloading /Users/huangqiang/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.01.0-ce/boot2docker.iso...
(test) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(test) Copying /Users/huangqiang/.docker/machine/cache/boot2docker.iso to /Users/huangqiang/.docker/machine/machines/test/boot2docker.iso...
(test) Creating VirtualBox VM...
(test) Creating SSH key...
(test) Starting the VM...
(test) Check network to re-create if needed...
(test) Found a new host-only adapter: "vboxnet0"
(test) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env test
```

> 这里需要注意，`Mac` 上我们通常用的是 Docker for Mac，现在的这个 `Docker Engine` 是不依靠 `VirtualBox` 了的，它基于 `Mac` 的 `xhyve` 虚拟引擎。所以你必须要自己安装一个 [VirtualBox（官网）](https://www.virtualbox.org/wiki/Downloads) 。（`windows 10` 的话用 `hyperv` 驱动来代替 `VirtualBox`）

这样就创建了一个名为 `test` 的本地主机实例。其中 `-d` 表示选择的驱动类型。

> `Docker Machine` 支持以下虚拟引擎驱动：[Amazon Web Services](https://docs.docker.com/machine/drivers/aws/)、[Microsoft Azure](https://docs.docker.com/machine/drivers/azure/)、[Digital Ocean](https://docs.docker.com/machine/drivers/digital-ocean/)、[Exoscale](https://docs.docker.com/machine/drivers/exoscale/)、[Google Compute Engine](https://docs.docker.com/machine/drivers/gce/)、[Generic](https://docs.docker.com/machine/drivers/generic/)、[Microsoft Hyper-V](https://docs.docker.com/machine/drivers/hyper-v/)、[OpenStack](https://docs.docker.com/machine/drivers/openstack/)、[Rackspace](https://docs.docker.com/machine/drivers/rackspace/)、[IBM Softlayer](https://docs.docker.com/machine/drivers/soft-layer/)、[Oracle VirtualBox](https://docs.docker.com/machine/drivers/virtualbox/)、[VMware vCloud Air](https://docs.docker.com/machine/drivers/vm-cloud/)、[VMware Fusion](https://docs.docker.com/machine/drivers/vm-fusion/)、[VMware vSphere](https://docs.docker.com/machine/drivers/vsphere/)、[VMware Workstation](https://github.com/pecigonzalo/docker-machine-vmwareworkstation) (unofficial plugin, not supported by Docker)、[Grid 5000](https://github.com/Spirals-Team/docker-machine-driver-g5k) (unofficial plugin, not supported by Docker)

如果你还可以加上以下配置参数：

- `--engine-opt dns=114.114.114.114` 配置 Docker 的默认 DNS；

- `--engine-registry-mirror https://registry.docker-cn.com` 配置主机内存 配置主机 CPU；

- `--virtualbox-memory 2048` 配置主机内存；

- `--virtualbox-cpu-count 2` 配置 Docker 的仓库镜像；

  更多参数请使用 `docker-machine create --driver virtualbox --help` 命令查看。

#### 2. 连接到一个主机

首先，创建主机成功后，我们得通过 env 命令连接到该主机，以便让后续操作对象都是目标主机。

```
➜  ~ docker-machine env test # test 是上文设置的 machine 的名称
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/huangqiang/.docker/machine/machines/test"
export DOCKER_MACHINE_NAME="test"
# Run this command to configure your shell:
# eval $(docker-machine env test)
```

然后我们输入这个命令返回结果的最后一句提示后就操纵 `test` 主机了。

当然，你也可以直接通过 `Docker Machine` 的 `ssh` 命令来登录到你的主机。

```
➜  ~ docker-machine ssh test

docker@test:~$ docker --version
Docker version 17.10.0-ce, build f4ffd25
```

连接到主机之后你就可以在其上使用 `Docker` 了。

#### 3. 停止主机实例

```
docker-machine stop test
```

#### 4. 启动一个主机

```
docker-machine start test
```

#### 5. 查看所有的主机实例

```
docker-machine ls
```

#### 6. 查看单个主机详细信息

```
docker-machine inspect test
```

#### 7. 其他指令

- `active`：查看活跃的 Docker 主机；
- `config` 输出连接的配置信息；
- `ip`：获取主机地址；
- `kill`：停止某个主机；
- `provision`：重新设置一个已存在的主机；
- `regenerate-certs`：为某个主机重新生成 TLS 认证信息；
- `restart`：重启主机；
- `rm`：删除某台主机：
- `scp`：在主机之间复制文件；
- `mount`：挂载主机目录到本地；
- `status`：查看主机状态；