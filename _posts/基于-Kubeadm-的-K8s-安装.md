---
title: 基于 Kubeadm 的 K8s 安装
author: Huanqiang
tags: [kubernetes, Kubeadm]
categories: [kubernetes]
keywords: [kubernetes, kubeadm]
date: 2018-03-29 16:50:25
---

> 本文以安装1.9.3 版本的 K8s 集群为例。需要的文件在 [Github](https://github.com/Huanqiang/K8s-1.9.3-setup) 上可下载。

<!-- more -->

## 一、 准备工作

### 1. 服务器

至少两台可以相互访问的服务器，而且服务器的主机名称一定不能一样，一样的话就有问题的。

> 测试相互访问：互相 `ping` 一下就好了；

#### 修改 host 的方式

```bash
sudo vi /etc/hostname	# 修改主机名称，比如 k8s-master
sudo vi /etc/hosts		# 将 127.0.0.1 后面的 hostname 修改成你自己的主机名称(k8s-master)
sudo reboot				# 重启主机后才会生效
```

### 2. kubernetes deb 安装包准备

我们需要四个安装包：`kubeadm`、`kubectl`、`kubelet`、`kubernetes-cni` 。

下载方式：可以在 [这里](https://packages.cloud.google.com/apt/dists/kubernetes-xenial/main/binary-amd64/Packages) 找到 deb 各个版本对应的filename，然后加上前缀 `https://packages.cloud.google.com/apt/` 即可下载，需要翻墙。

### 3. 关闭 swap

这个非常之重要，不关闭的话会导致 `kubelet` 不能正常工作。

```bash
sudo swapoff -a # 注意这里的修改只是临时的，重启之后会失效
```

#### 检查是否关闭

```bash
free -m   # 可用free -m查看swap是否关闭
              total        used        free      shared  buff/cache   available
Mem:            992         564          72           1         355         255
Swap:             0           0           0
```

#### 永久关闭

##### 方法一

> 网上都是这个方法，但是我测试不行啊，不知道为什么...

首先将 `vm.swappiness=0` 加入 `/etc/sysctl.d/k8s.conf` 文件中，在执行 `sysctl -p /etc/sysctl.d/k8s.conf` 使之生效。

```bash
cat <<EOF >  /etc/sysctl.d/k8s.conf
vm.swappiness=0
EOF
sysctl -p /etc/sysctl.d/k8s.conf
```

然后为了重启之后自动完成，将 sysctl 命令加入启动脚本：

```bash
sudo vi /etc/profile # 在文件的最后一行加上 sysctl -p /etc/sysctl.d/k8s.conf
```

##### 方法二

打开 `/etc/fstab` 这个文件，然后将带有 swap 字样的那一行注释掉。

```bash
sudo vi /etc/fstab
```

### 4. 关闭防火墙

似乎暂时不需要关闭

## 二、Master 节点配置

### 1. 安装 Docker

具体的方法可以参照 [docker 官网的安装步骤](https://docs.docker.com/engine/installation/)。

这里提供简要的安装步骤：（有两个方法）

```bash
# 方法一：从 Ubuntu 的仓库安装
apt-get update
apt-get install -y docker.io

# 方法二： 从 Docker 官网仓库安装 (推荐)
apt-get update
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
   "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')
```

> 安装 Docker 不需要翻墙；

#### 出现的问题

如果出现一下错误，请去 daocloud 配置一下 [Docker 加速器](https://www.daocloud.io/mirror#) 或者是 [中科大的加速器](https://lug.ustc.edu.cn/wiki/mirrors/help/docker)<优先推荐>。

```bash
error pulling image configuration: Get https://dseasb33srnrn.cloudfront.net/registry-v2/docker/registry/v2/blobs/sha256/73/73acd1f0cfadf6f56d30351ac633056a4fb50d455fd95b229f564ff0a7adecda/data?Expires=1521485533&Signature=DLri37AfNHayM0WDAKZSDvk1ppw7Pxvp0ak5LXuOClM1PzsFya0D~Vct1VA1bxI59kg8njgqhPPfFVRjMIWWOT7TIBt871dC~20th3fmLBkQ-bfmwnArvT12CiweIhN8sHJ0k4Idf8TKoQiZ6Dsv-~PfXrnd3FRo~q5t4m-0Rkk_&Key-Pair-Id=APKAJECH5M7VWIS5YZ6Q: net/http: TLS handshake timeout
```

配置好加速器之后，重启 Docker 命令：`sudo service docker restart`。

### 2. 安装 K8s 各组件

首先将我们之前准备好的 `kubeadm`、`kubectl`、`kubelet`、`kubernetes-cni` 这四个安装包拷入服务器之中，然后进入相应的目录安装它。

```bash
#!/bin/bash
apt-get install -y -q socat ebtables ethtool	# 这是 k8s 安装过程中所需要的包
dpkg -i kubernetes-cni_0.6.0-00_amd64.deb
dpkg -i kubelet_1.9.3-00_amd64.deb
dpkg -i kubectl_1.9.3-00_amd64.deb
dpkg -i kubeadm_1.9.3-00_amd64.deb
systemctl enable kubelet
systemctl start kubelet
```

### 3. 配置 K8s 依赖的 docker 镜像

因为 `K8s` 所需要的 · 镜像都是被墙了，你不能直接 `docker pull` 下来，所以我们只能自己去找到对应的版本，然后 docker pull ，再将其打上 `google` 的标签（`docker tag`）。

```bash
images=(kube-scheduler-amd64:v1.9.3 \
kube-apiserver-amd64:v1.9.3 \
kube-controller-manager-amd64:v1.9.3 \
kube-proxy-amd64:v1.9.3 \
etcd-amd64:3.1.11 \
pause-amd64:3.0 \
k8s-dns-sidecar-amd64:1.14.7 \
k8s-dns-kube-dns-amd64:1.14.7 \
k8s-dns-dnsmasq-nanny-amd64:1.14.7)
for imageName in ${images[@]} ; do
  sudo docker pull mirrorgooglecontainers/$imageName
  sudo docker tag mirrorgooglecontainers/$imageName gcr.io/google_containers/$imageName
  sudo docker rmi mirrorgooglecontainers/$imageName
done
```

```bash
sudo docker pull jmgao1983/flannel:v0.10.0-amd64
sudo docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.8.3

sudo docker tag jmgao1983/flannel:v0.10.0-amd64 quay.io/coreos/flannel:v0.10.0-amd64
sudo docker tag mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.8.3 k8s.gcr.io/kubernetes-dashboard-amd64:v1.8.3

sudo docker rmi jmgao1983/flannel:v0.10.0-amd64
sudo docker rmi mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.8.3
```

> 这里的 `docker image` 来源于 `docker hub` 的 `mirrorgooglecontainers` 这个大神的贡献。
>
> 后面拉取的这个 `flannel 的 image`  是下一步中的 `flannel` 所需要的。

### 4. 启动集群

```bash
kubeadm init --kubernetes-version=v1.9.3 --pod-network-cidr 10.244.0.0/16 
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
# 安装cni插件flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

> 注意：
>
> 1. 这里的 `--kubernetes-version` 必须要指定，不然会去访问 k8s 仓库一个 `release` 文件，而这个文件好像被墙了，所以会失败。
> 2. 其实这里只要 `kubeadm init` 就好了，后面的步骤在成功之后会有提示的。

在启动成功之后，会出现一个加入集群的命令提示：

```bash
sudo kubeadm join --token 2a34a6.cc41522033e15125 10.1.18.83:6443 --discovery-token-ca-cert-hash sha256:4e21f4db6616e66fd65eb709a95ffe0bc9570a8b0e505b47372bc9b7872b5d73
```

只要在安装好 k8s 的工作节点使用这个命令，就会自动加入集群中。

加入后可以使用 `sudo kubectl get nodes` 来查看：

```bash
NAME         STATUS    ROLES     AGE       VERSION
k8s-master   Ready     master    2h        v1.8.7
ubuntu       Ready     <none>    1h        v1.8.7
```

## 三、工作节点配置

### 1. 配置 docker

与上文相同。

### 2. 安装 K8s 各组件

与上文相同。

### 3. 配置 K8s 依赖的 docker 镜像

这里的 docker 镜像就不像前面的 Master 节点一样这么多了，只需要4个就够了。

```bash
sudo docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.9.3
sudo docker pull mirrorgooglecontainers/pause-amd64:3.0
sudo docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.8.3
sudo docker pull jmgao1983/flannel:v0.10.0-amd64

sudo docker tag mirrorgooglecontainers/kube-proxy-amd64:v1.9.3 gcr.io/google_containers/kube-proxy-amd64:v1.9.3
sudo docker tag mirrorgooglecontainers/pause-amd64:3.0 gcr.io/google_containers/pause-amd64:3.0
sudo docker tag mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.8.3 k8s.gcr.io/kubernetes-dashboard-amd64:v1.8.3
sudo docker tag jmgao1983/flannel:v0.10.0-amd64 quay.io/coreos/flannel:v0.10.0-amd64

sudo docker rmi mirrorgooglecontainers/kube-proxy-amd64:v1.9.3 
sudo docker rmi mirrorgooglecontainers/pause-amd64:3.0
sudo docker rmi mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.8.3
sudo docker rmi jmgao1983/flannel:v0.10.0-amd64
```

### 4. 加入集群

使用上文生成的 `kubeadm join` 来加入集群。

```bash
sudo kubeadm join --token 2a34a6.cc41522033e15125 10.1.18.83:6443 --discovery-token-ca-cert-hash sha256:4e21f4db6616e66fd65eb709a95ffe0bc9570a8b0e505b47372bc9b7872b5d73
```

## 四、遇到的错误

### 1. 节点状态 `NotReady`

1. 使用 `journalctl -u kubelet` 去查日志，如果看不出明显错误，再使用 `systemctl status kubelet` 去看 kubelet 的状态。如果出现以下错误

```bash
Mar 19 16:11:29 k8s-master-demo kubelet[4310]: W0319 16:11:29.378830    4310 cni.go:171] Unable to update cni config: No networks found in /etc/cni/net.d
Mar 19 16:11:29 k8s-master-demo kubelet[4310]: E0319 16:11:29.379674    4310 kubelet.go:2104] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: networ...
```

则使用命令 `sudo kubectl get pods --all-namespaces` 看一下 pod 有没有出错。再依次解决问题。

> 如果是子节点 `NotReady`，请用命令 `sudo kubectl describe node 子节点名`；

### 2. 解决 `flannel` 错误

> 关于 flannel 要注意的是，每一个节点都需要 flannel 这个 Docker 镜像，所以每一个节点都是要拉取的。

如果使用  `sudo kubectl get pods --all-namespaces` 出现这样的错误：

```bash
kube-system   kube-flannel-ds-7nt8p                     0/1       Init:ErrImagePull       0          49m
kube-system   kube-flannel-ds-l5ndv                     0/1       Init:ImagePullBackOff   0          49m
```

则是 flannel 的镜像拉取错误，去 Docker hub 上搜索镜像代替一下：

```bash
sudo docker pull jmgao1983/flannel:v0.10.0-amd64
sudo docker tag jmgao1983/flannel:v0.10.0-amd64 quay.io/coreos/flannel:v0.10.0-amd64
```

然后等会就会 pod 应用你下载的这个替代镜像重启。

### 3. 查看 `iptables`

查看 `iptables`：`sudo iptables -S -t nat`；

## 五、参考资料

1. [Docker快速安装以及换镜像源](https://www.jianshu.com/p/34d3b4568059)：Docker 加速器配置；
2. [kubernetes 1.8.7 国内安装(kubeadm)](https://my.oschina.net/andylo25/blog/1618342)：本文的主要参考文献之一；
3. [Ubuntu | 使用kubeadm方式安装kubernetes完整过程](http://www.sohu.com/a/205878489_468741)：本文的主要参考文献之一；
4. [使用kubeadm部署k8s1.8](https://segmentfault.com/a/1190000012415387)：提到了  `kubeadm`、`kubectl`、`kubelet`、`kubernetes-cni` 这四个安装包的下载方式；