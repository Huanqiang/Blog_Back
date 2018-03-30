---
title: Jenkins + Gitlab + Kubernetes 的自动化持续集成与部署
author: Huanqiang
tags: [Kubernetes, Jenkins, 自动部署，持续集成]
categories: [Kubernetes]
keywords: [Kubernetes, Jenkins, 自动部署，持续集成]
date: 2018-03-30 15:10:45
---

本文是以 `Maven` 项目为例来搭建  `Jenkins + Gitlab + Kubernetes` 的自动化持续集成与部署的流程，重点在于 `jenkins + gitlab` 的 `push` 的钩子触发和 `jenkins + Kubernetes` 的自动部署，至于项目编译，则可以依照各自需求进行编译。

本文的自动化持续集成与部署思路：

1. 在将代码上传至 `gitlab` 时，触发 `push` 的钩子，在 `jenkins` 中进行项目的编译；
2. 自动将编译后的文件打包成 `docker` 镜像，并上传至（私有）`docker hub`；
3. 修改 `Kubernetes` 的服务配置文件（`Service.yaml` 和 `Deployment.yaml`）中的镜像变量；（因为在我将每一个 `jenkins` 构建的版本号作为了 `docker` 镜像的版本号，所以每次都要自动改变）
4. 将 `Kubernetes` 的服务配置文件上传至 `Kubernetes` 的 `master` 服务器；
5. 在 `Jenkins` 中执行 `kubectl apply -f file.yaml`，在 `Kubernetes` 部署服务；

> 这里假装是一张流程图（脑补...）

<!-- more -->

## 环境搭建

### 1. Jenkins

#### Jenkins 搭建

在这里，我们将使用 Docker 版的 Jenkins。因为 Jenkins 本身自带了 Java，而不带 Maven，而我在测试的时候发现 Jenkins 里的自动安装 Maven 总是不起作用（不知道为什么），所以重新做一个自带 Maven 的 Jenkins 镜像：

```dockerfile
FROM jenkins/jen

# 这里需要你自己下载好 maven，然后放在 dockerfile 所在目录；
COPY ./apache-maven-3.5.0 /var/local/maven

# 这里是为了让 Jenkins 有权限使用主机的 docker (大雾...)
USER root
RUN apt-get update \
      && apt-get install -y sudo \
      && rm -rf /var/lib/apt/lists/*
RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
 
USER jenkins
```

在制作好镜像之后，我们将镜像 push 到私有的 Docker Hub 里。

在 Jenkins 的服务器里，首先，我们建立一个 Jenkins 的文件目录，以便将 Jenkins 的产生的文件保存在服务器本地，然后我们使用以下命令来构建 Jenkins 容器：

```bash
docker run -d -p 9191:8080 -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker -v /usr/lib/x86_64-linux-gnu/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7 --mount type=bind,source='主机的 Jenkins 目录',target='/var/jenkins_home' --name jenkins 127.0.0.1:5000/jenkins-maven
```

1. `/var/run/docker.sock` 的作用就是让 `Jenkins` 能通过主机的 `Docker` 守护进程（也就是 `Docker Engine`）来操作 docker 容器；< 详情请参考：[关于/var/run/docker.sock - Fundebug](https://www.jianshu.com/p/6c3fdb0e9cb5) >
2. `-v $(which docker):/usr/bin/docker` ：这个是将外部的docker 挂载到 jenkins 容器内部，以便其能使用 docker 命令；
3. `-v /usr/lib/x86_64-linux-gnu/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7`：不添加这个执行 docker 命令可能会报找不到 `libltdl.so.7` 的错误；

接下来只需要打开链接 `http://localhost:9191`  然后按照要求去找出密码即可（密码可以在你上一步设置的本机目录下的相应位置找到）。获取密码的指令：

```bash
sudo docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```



#### 插件安装及配置

这里需要很多插件需要安装，比如 

1. `gitlab`
2. `gitlab hook`：`gitlab` 上与 `Jenkins` 项目相连接的钩子；
3. `Publish Over SSH`：通过 `SSH` 将文件传到远程服务器；
4. `Maven Integration`：`Maven` 的编译操作流程；
5. `Kubernetes Continuous Deploy`：能够很方便的将项目部署在 K8s 上，但是我测试的时候有问题，本文也会介绍下其用法；

然后安装完成之后，有些需要在系统里面配置一下。如下：

##### 1. `Maven`

我们之前将下载的 `maven` 放入了镜像之中，然后现在我们配置一下，在 `系统管理 → 全局工具配置` 中：

![maven](/img/Jenkins-K8s/maven.png)

1处是我们之前在构建镜像时的目录；

##### 2. `Publish over SSH Plugin`

配置 `Publish over SSH Plugin` 插件，在 `系统管理 → 系统设置 → Publish over SSH`  中：

![publish-over-SSh](/img/Jenkins-K8s/Publish-over-SSH.png)

> Remote Derictory：服务器上的目录，插件将把文件传送到此默认目录下（可在 Job 中修改）。

在填写好之后，点击 `Test COnfiguration` 验证一下是否正确。



### 2. Kubernetes

这里使用的是 Kubeadm 搭建的 Kubernetes 集群：[基于 Kubeadm 的 K8s 安装](https://huanqiang.wang/2018/03/29/基于-Kubeadm-的-K8s-安装/)。

> 二进制方法搭建集群请参照：[手动搭建高可用的kubernetes 集群](https://blog.qikqiak.com/post/manual-install-high-available-kubernetes-cluster/)

### 3. Docker hub

可以参照 《docker_practice》的搭建私有仓库这一章节：[搭建私有仓库](https://yeasy.gitbooks.io/docker_practice/content/repository/registry.html)；

## 流程详述

### 1. 新建任务

这里，因为我们安装了 `Maven` 插件，所以会多出一个构建 Maven 项目的选项，选择它：

![create-task](/img/Jenkins-K8s/create-task.png)

### 2. Gitlab 设置

![ode-manager-settin](/img/Jenkins-K8s/code-manager-setting.png)

### 3. 构建触发器设置

![build-trigger-setting](/img/Jenkins-K8s/build-trigger-setting.png)

### 4. `Maven` 构建设置

此处需要设置 `pom.xml` 文件的位置和构建的执行命令，一般默认即可：![maven-config-file](/img/Jenkins-K8s/Maven-config-file.png)

### 5.  构建服务镜像

在 `Post Steps` 中，我们新增一个 `Execute shell` 操作，用于构建一个 Docker 镜像

![uild-docker-image](/img/Jenkins-K8s/build-docker-images.png)

命令如下：

```bash
HOST="127.0.0.1"
API_PORT=9191
IMAGE_NAME="$HOST:5000/bi-saiku:v$BUILD_NUMBER"
CONATINER_NAME=bi-saiku

cd $WORKSPACE/target
cp ../dockerfile .
cp ../.dockerignore .

sudo docker build -t $IMAGE_NAME .

sudo docker push $IMAGE_NAME
```

### 6. 修改 `k8s` 配置文件参数

从上一步我们构建的 `docker` 镜像的版本号可以看出，我们用的是 Jenkins 中每次构建所生成的版本号：`$BUILD_NUMBER`，所以对应的我们的 `K8s` `Deployment` 配置文件中需要的镜像也要用这个版本号，那么我们就需要在 Jenkins 中自动将版本号变量替换掉。

> 用 `Jenkins` 中每次构建所生成的版本号作为 docker 镜像的版本号的思路来源于 `《轻量级微服务架构》- 6.4.2节：自动发布 Docker 容器`；

首先，我们先看一下我们的  `K8s` `Deployment` 配置文件是如何写的：

![k8s-deployment-file](/img/Jenkins-K8s/k8s-deployment-file.png)

> 值得注意：我们这里的 `BUILD_NUMBER` 不能加 `$` 符号，加了在替换的时候会出问题；当然你也可以用其他标识符来替换这个 `BUILD_NUMBER`

好了，现在我们在 `Post Steps` 中，再新增一个 `Execute shell` 操作，来执行替换操作：

![change-k8s-deployment-file](/img/Jenkins-K8s/change-k8s-deployment-file.png)

### 7. 将 `k8s` 配置文件上传并执行

还是在 `Post Steps` 中，我们新增一个 `Send files or execute over SSH` 操作。

![upload-action-k8s-file](/img/Jenkins-K8s/upload-action-k8s-file.png)

1. 这里是选择你要部署的服务器，在 “系统管理” → “系统设置” → `Publish over SSH` 中设置。详细设置请参照下文。

2. 这里是你要上传的文件，支持表达式，如 `/dist/**` 表示复制 `dist` 文件夹下的所有东西 。

   > 文件对应的位置是一个**相对目录**，起始位置是：`之前设置的主机目录/workspace/项目名/` (这是主机下目录，如果不是使用 docker，那么目录就是 `/var/jenkinks/workspace/项目名/` )。

3. 文件复制时要过滤的目录。

4. 这里是文件上传的远程服务器的目录，也是一个**相对目录**，其起始位置是你在系统设置中设置的路径。

   > 其实如果你实在不知道位置，你可以在 `Exec command` 中使用 `pwd` 命令来打印地址的

5. 这里是将文件上之后可以进行的操作，可以是命令也可以是远程的一个脚本。

> 关于  Source files、Remove prefix、Remote directory 的使用，请参照 Jenkins 的 [Push Over 文档](https://wiki.jenkins.io/display/JENKINS/Publish+Over#PublishOver-Remotedirectory)。这里有详细说明和例子，非常好。

### 8. Gitlab webhook 触发配置

![gitlab-webhook](/img/Jenkins-K8s/gitlab-webhook.png)

图中的 URL 设置格式为：`http://your-jenkins-server/gitlab/build_now/project_name`。然后 Trigger 部分只需要什么（操作）的时候会触发该 `Hook`。

> 图中的 URL 设置方式表示 Gitlab 接收到 Push 操作后会通知 Jenkins，Jenkins 会立即进行构建等一系列的操作。
>
> 当然如果 URL 设置为 `http://your-jenkins-server/git/notifyCommit?url=<URL of the Git repository for the GitLab project>`，则表示  Gitlab 接收到 Push 操作后会通知 Jenkins，Jenkins 会等到下一个轮训时间后才进行构建等一系列操作。
>
> Gitlab Hook 文档地址：https://github.com/Huanqiang/gitlab-hook-plugin（这是我 fork 下来的备份地址）

### 附加：插件 `Kubernetes Continuous Deploy` 的使用

![ubernetes Continuous Deplo](/img/Jenkins-K8s/Kubernetes%20Continuous%20Deploy.png)

有个这个插件，我们就**不需要 `6-8` 这三步了**，它会帮我们自动处理了，非常之方便。

这里有一个地方要注意，我们第 `6` 步中的 `docker` 镜像现在是这样写的 `image: 10.1.18.83:5000/deployment:vBUILD_NUMBER`，最后是 `BUILD_NUMBER` 而不是 `$BUILD_NUMBER`，用了这个插件后，后面必须改成  `$BUILD_NUMBER`，让插件自动帮我们替换掉这个参数；



## 遇到的一些问题

### 1. `jenkins` 构建时不能创建 `Java` 虚拟机

Jenkins 时自带一个 Java 版本的，如果你在构建的时候遇到这个问题，我也不知道怎么解决，我一般在创建好一个 Jenkins 容器的时候，首先进行创建一个项目进行以下测试，通过了才进行下一步操作。

```bash
java -verison
mvn -v
```

### 2. `Publish Over SSH` 插件版本

`Publish Over SSH` 插件版本需最新，1.8.x 版本存在不能保存设置信息的 BUG。

### 3. `Docker` 的使用

`Jenkins` 中想使用 `docker` 打包有两种方案：`docker in docker` 和 `docker outside of docker` 两种，我们这里选择的是 `docker outside of docker`，这就是说 `docker` 中的 `jenkins` 容器使用其本机的 `docker engine` 来进行打包镜像等操作。

`dood` 方案的时候必须要绑定主机的 docker.sock 和 docker 目录，即：

```bash
-v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker
```

同时如果出现如下缺少 `libltdl.so.7` 的错误，还需要将该文件引入进来。

```bash
docker: error while loading shared libraries: libltdl.so.7: cannot open shared object file: No such file or directory
```

引入方式：

```bash
-v /usr/lib/x86_64-linux-gnu/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7
```

