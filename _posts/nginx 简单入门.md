---
title: nginx 简单入门
author: Huanqiang
tags: [nginx, nginx 简单配置, 负载均衡]
categories: [前端]
keyword: [nginx, nginx 简单配置, 负载均衡]
date: 2017-04-14 08:31:49
---

Nginx是一个网页服务器，它能反向代理HTTP, HTTPS, SMTP, POP3, IMAP的协议链接，以及一个负载均衡器和一个HTTP缓存。

<!-- more -->

## 反向代理

反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

对于反向代理的处理部分，主要集中在一下代码：

```nginx
server {  
       listen       80;
       server_name  localhost; 
       client_max_body_size 1024M;

       location / {
           proxy_pass http://localhost:8080;
           proxy_set_header Host $host:$server_port;
       }
   }
```

我们来看一下这里面各标签的含义：

1. `listen`：代表了当前代理服务器（最外层的server）监听的端口。如果有多个 `server`，那么这个 `listen` 就要各不相同。
2. `server_name`：代表了代理服务器的域名，只有请求的 `$hostname` 和 `server_name` 匹配的时候，才会进入这个 `server` 域。只有一个 `server` 域的时候除外（参见下面）。
3. `location`：表示匹配的路径，此处配置 `/` 表示将匹配所有的请求，但是正则和最长字符串会优先匹配。（`location`具体写法详见 资料3 ）。
4. `proxy_pass`：表示代理路径，进入了该 `location` 的请求会被转发到此路径。值得注意的是，**proxy_pass 后面的路径必须加http://**。其他注意点详见 资料4。

> 对 `server_name` 的补充：如果 nginx 中只配置一个 `server` 域的话，则 nginx 是不会去进行 `server_name` 的匹配的。因为只有一个 `server` 域，也就是这有一个虚拟主机，那么肯定是发送到该 nginx 的所有请求均是要转发到这一个域的，即便做一次匹配也是没有用的。还不如干脆直接就省了。如果一个 `http` 域的 `server` 域有多个，nginx 才会根据 `$hostname` 去匹配 `server_name` 进而把请求转发到匹配的 `server` 域中。此时的匹配会按照匹配的优先级进行，一旦匹配成功进不会再进行匹配，关于具体的匹配规则可以参见 nginx 官网提供的文档。实际上，nginx 会匹配请求头中的 `host` 和 `server_name`，如果没有匹配的上，nginx 会指定该请求到一个默认的 `server`域，如果比较说明，默认的 `server` 为第一个 `server`。

## 负载均衡

负载均衡其意思就是分摊到多个操作单元上进行执行，例如Web服务器、FTP服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务。简单而言就是当有2台或以上服务器时，根据规则随机的将请求分发到指定的服务器上处理，负载均衡配置一般都需要同时配置反向代理，通过反向代理跳转到负载均衡。

Nginx目前支持自带3种负载均衡策略：`轮询（默认）`、`weight（权重）` 和 `ip_hash`。还有2种常用的第三方策略：`fair` 和 `url_hash`。

以下是 负载均衡 的核心代码：

```nginx
upstream test {
       server localhost:8080;
       server localhost:8081;
   }
   server {
       listen       81;                                                         
       server_name  localhost;                                               
       client_max_body_size 1024M;

       location / {
           proxy_pass http://test;
           proxy_set_header Host $host:$server_port;
       }
   }
```

最核心代码就是 `upstream` 部分了。值得注意的是，**upstream 后的名字必须和 server -> location -> proxy_pass 中的路径主域名相同（即路径除了 http:// 以外的部分）**。

在此 `upstream` 里，我们配置了两个服务器。分别为 `localhost:8080` 和 `localhost:8081`。即服务器名为 `域名:端口` 的形式。

### 轮询（默认）

`upstream` 按照轮询（默认）方式进行负载，每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 `down` 掉了，能自动剔除。（比如我们上面这个写法就已经用了 轮询 策略了。）

虽然这种方式简便、成本低廉。但缺点是：可靠性低和负载分配不均衡。适用于图片服务器集群和纯静态页面服务器集群。

### weight（权重）

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。 `weight` 默认为1。`weight` 越大，负载的权重就越大。

在下面这个例子中：`10.0.0.88` 的访问比率要比 `10.0.0.77` 的访问比率高一倍。

```nginx
upstream test {
    server 10.0.0.77 weight=5;
      server 10.0.0.88 weight=10;
   }
```

### ip_hash（访问ip）

每个请求按访问ip的 hash 结果分配，**这样每个访客固定访问一个后端服务器，可以解决 session 的问题。**

```nginx
upstream test {
    ip_hash;
    server 10.0.0.10:8080;
    server 10.0.0.11:8080;
}
```

### fair（第三方）

按后端服务器的响应时间来分配请求，响应时间短的优先分配。与weight分配策略类似。

```nginx
upstream test {
    server 10.0.0.10:8080;
    server 10.0.0.11:8080;
    fair;
}
```

### url_hash（第三方）

按访问 url 的 `hash` 结果来分配请求，使每个 url 定向到同一个后端服务器，后端服务器为缓存时比较有效。

注意：**在 upstream 中加入 hash 语句，server 语句中不能写入 weight 等其他的参数，hash_method 是使用的 hash 算法。**

```nginx
upstream test {
    server 10.0.0.10:7777;
    server 10.0.0.11:8888;
    hash $request_uri;
    hash_method crc32;
}
```

### 其他状态值

1. `down`：表示当前的 `server` 暂时不参与负载.
2. `weight`：默认为1。`weight` 越大，负载的权重就越大。
3. `max_fails`：允许请求失败的次数默认为1。当超过最大次数时，返回 `proxy_next_upstream` 模块定义的错误。
4. `fail_timeout`：在 `max_fails` 次失败后，暂停的时间。
5. `backup`：其它所有的非 `backup` 机器 `down` 或者忙的时候，才会去请求 `backup` 机器。所以这台机器压力会最轻。

```nginx
  # 定义负载均衡设备的Ip及设备状态
upstream bakend{ 
  ip_hash;
	  server 10.0.0.11:9090 down;
	  server 10.0.0.11:8080 weight=2;
	  server 10.0.0.11:6060;
	  server 10.0.0.11:7070 backup;
}
```

## HTTP服务器（包含动静分离）

Nginx 本身也是一个静态资源的服务器，当只有静态资源的时候，就可以使用 Nginx 来做服务器，同时现在也很流行动静分离，就可以通过 Nginx 来实现。

所谓动静分离，就是让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作，这就是网站静态化处理的核心思路。

```nginx
upstream test{  
     server localhost:8080;  
     server localhost:8081;  
  }   

  server {  
      listen       80;  
      server_name  localhost;  

      location / {  
          root   e:wwwroot;  
          index  index.html;  
      }  

      # 所有静态请求都由nginx处理，存放目录为html  
      location ~ .(gif|jpg|jpeg|png|bmp|swf|css|js)$ {  
          root    e:wwwroot;  
      }  

      # 所有动态请求都转发给tomcat处理  
      location ~ .(jsp|do)$ {  
          proxy_pass  http://test;  
      }  

      error_page   500 502 503 504  /50x.html;  
      location = /50x.html {  
          root   e:wwwroot;  
      }  
  }
```

这样我们就可以吧 HTML 以及图片和 css 以及 js 放到 `wwwroot` 目录下，而 tomcat 只负责处理 jsp 和请求，例如当我们后缀为 gif 的时候，Nginx 默认会从 `wwwroot` 获取到当前请求的动态图文件返回，当然这里的静态文件跟 Nginx 是同一台服务器，我们也可以在另外一台服务器，然后通过反向代理和负载均衡配置过去就好了，只要搞清楚了最基本的流程，很多配置就很简单了，另外 `localtion` 后面其实是一个正则表达式，所以非常灵活。

## ubuntu 系统下的 nginx 配置文件

nginx 的主配置文件是 `/path/conf/nginx.conf`。

但是我们要注意 ubuntu 系统下的 nginx，虽然他的主配置也是 `/path/conf/nginx.conf`，但是我们在写`反向代理`、`负载均衡`和其他用途的配置的时候，我们可以不需要写在该主配置文件里面。我们可以在 `/etc/nginx/conf.d/` 目录下新建一个 `name.conf` 文件，里面上写所需的配置即可。

为什么 ubuntu 下的 nginx 可以这样写呢。因为其主配置文件里包含了这样一段代码：

```nginx
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```

即它会去读取 `/etc/nginx/conf.d/` 目录下的 `./conf` 文件，并加载到这个主配置文件里面去。

举个 `/etc/nginx/conf.d/` 目录下的 `./conf` 文件的栗子：

```nginx
upstream docker-tomcat-cluster {
      server web1:8080;
      server web2:8080;
  }
  server { 
      listen 80;
      server_name localhost; # must give the domain to match
      location / {
          proxy_pass http://docker-tomcat-cluster;
          proxy_redirect off; 
          proxy_set_header Host $host; 
          proxy_set_header X-Real-IP $remote_addr; 
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
      } 
  }
```

## 资料

1. [全面了解 Nginx 主要应用场景](http://blog.jobbole.com/110400/)
2. [Windows下Nginx+Tomcat整合的安装与配置（一）](http://zyjustin9.iteye.com/blog/2017394)
3. [nginx配置location总结及rewrite规则写法](http://seanlook.com/2015/05/17/nginx-location-rewrite/)
4. [Nginx配置proxy_pass](http://dmouse.iteye.com/blog/1880474)：proxy_pass转发的路径后是否带 “/” 的结果会大不相同。
5. [Nginx配置upstream实现负载均衡](http://favccxx.blog.51cto.com/2890523/1622091)：负载均衡各策略简要说明。