---
title: k8s 监控部署
author: Huanqiang
tags: [kubernetes, kubernetes 监控]
categories: [kubernetes]
keywords: [kubernetes, kubernetes 监控, cAdvisor, Heapster, influxdb, grafana]
date: 2018-04-03 13:52:43
---

在 k8s 上，我们通过结合 cAdvisor、Heapster、influxdb 和 grafana 来完成对集群监控信息的收集、存储和展示。

1. cAdvisor： 能收集正在运行的容器的信息：CPU、内存、文件系统、网络等；
2. Heapster：通过节点（node）的 kubelet 来查询/聚合 cAdvisor 收集的数据；
3. influxdb：存储 heapster 聚合的信息；
4. grafana：将存储的信息展示出来；

![mage-20180403133652](/img/k8s-monitor/image-201804031336520.png)

<!-- more -->

## cAdvisor

cAdvisor 是一个已经被集成到节点的 kubelet 中的开源代理，它能收集正在运行的容器的信息：CPU、内存、文件系统、网络等。

### kubeadm 开启 cAdvisor

新版本的 `k8s` 自带了 `cAdvisor`，如果你是用的 `kubeadm` 安装的 `k8s`，那么恭喜你，默认 `cAdvisor` 是被禁止的，我们可以看一下 `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` 的内容，就会发现 `--cadvisor-port` 的值被设置为了 0（如下输出）。而根据 `kubelet -h`，我们可以查看 `--cadvisor-port` 的值等于 0 时是禁用了 `cAdvisor`，所以我们需要注释掉改行，再重启 `kubelet` 就好了。

```bash
dbsi@k8s-master-demo:~$ cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true"
Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
Environment="KUBELET_DNS_ARGS=--cluster-dns=10.96.0.10 --cluster-domain=cluster.local"
Environment="KUBELET_AUTHZ_ARGS=--authorization-mode=Webhook --client-ca-file=/etc/kubernetes/pki/ca.crt"
# Environment="KUBELET_CADVISOR_ARGS=--cadvisor-port=0"
Environment="KUBELET_CERTIFICATE_ARGS=--rotate-certificates=true --cert-dir=/var/lib/kubelet/pki"
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_AUTHZ_ARGS $KUBELET_CADVISOR_ARGS $KUBELET_CERTIFICATE_ARGS $KUBELET_EXTRA_ARGS
```

重启 `kubelet` ：

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```



## Heapster

`Heapster` 像其他的应用程序一样在 `k8s` 上运行，它的 `pod` 能发现同一集群下的所有节点，然后从每一个节点的 `kubelet` 中提取度量标准（`metrics`），并通过 `pod` 和 `label` 将其聚合起来，最后将其存储到存储系统或是监控服务中。

![mage-20180403133037](/img/k8s-monitor/image-201804031330371.png)

> 1. Heapster 的 API 文档： [Heapster Metric Model](https://github.com/kubernetes/heapster/blob/master/docs/model.md)；
> 2. Heapster 的监控数据：[Metrics](https://github.com/kubernetes/heapster/blob/master/docs/storage-schema.md)；

## 部署

我们直接将这些部署在 k8s 中。

### 1. 部署 influxdb

这里需要将 `docker` 镜像修改一下，我这里是自己下载了，并上传至私有的 `docker hub`；

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: monitoring-influxdb
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: influxdb
    spec:
      containers:
      - name: influxdb
        image: 10.1.18.83:5000/heapster-influxdb-amd64:v1.3.3
        volumeMounts:
        - mountPath: /data
          name: influxdb-storage
      volumes:
      - name: influxdb-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: monitoring-influxdb
  name: monitoring-influxdb
  namespace: kube-system
spec:
  ports:
  - port: 8086
    targetPort: 8086
  selector:
    k8s-app: influxdb
```



### 2. 部署 grafana

1. 这里需要将 `docker` 镜像修改一下；
2. 将 `Service` 的 `type` 设置成 `NodePort`，以便外部访问；

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: monitoring-grafana
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: grafana
    spec:
      containers:
      - name: grafana
        image: 10.1.18.83:5000/heapster-grafana-amd64:v4.4.3
        ports:
        - containerPort: 3000
          protocol: TCP
        volumeMounts:
        - mountPath: /etc/ssl/certs
          name: ca-certificates
          readOnly: true
        - mountPath: /var
          name: grafana-storage
        env:
        - name: INFLUXDB_HOST
          value: monitoring-influxdb
        - name: GF_SERVER_HTTP_PORT
          value: "3000"
          # The following env variables are required to make Grafana accessible via
          # the kubernetes api-server proxy. On production clusters, we recommend
          # removing these env variables, setup auth for grafana, and expose the grafana
          # service using a LoadBalancer or a public IP.
        - name: GF_AUTH_BASIC_ENABLED
          value: "false"
        - name: GF_AUTH_ANONYMOUS_ENABLED
          value: "true"
        - name: GF_AUTH_ANONYMOUS_ORG_ROLE
          value: Admin
        - name: GF_SERVER_ROOT_URL
          # If you're only using the API Server proxy, set this value instead:
          # value: /api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
          value: /
      volumes:
      - name: ca-certificates
        hostPath:
          path: /etc/ssl/certs
      - name: grafana-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: monitoring-grafana
  name: monitoring-grafana
  namespace: kube-system
spec:
  # In a production setup, we recommend accessing Grafana through an external Loadbalancer
  # or through a public IP.
  # type: LoadBalancer
  # You could also use NodePort to expose the service at a randomly-generated port
  type: NodePort
  ports:
  - port: 80
    targetPort: 3000
  selector:
    k8s-app: grafana
```

### 3. 部署 heapster

1. 这里需要将 `docker` 镜像修改一下；
2. `Deployment` 中的 `--source` 参数需要设置一下，因为这里是将 `heapster` 放在了 `k8s` 中且只监控本身所在的集群，所以参数只需要设置成， `--sink` 要改成存储系统的源，这里使用的是 `influxdb`，再加上其 `influxdb Sevice` 的地址。

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: heapster
  namespace: kube-system
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: heapster
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: heapster
    spec:
      serviceAccountName: heapster
      containers:
      - name: heapster
        image: 10.1.18.83:5000/heapster-amd64:v1.5.2
        imagePullPolicy: IfNotPresent
        command:
        - /heapster
        - --source=kubernetes
        - --sink=influxdb:http://10.99.238.140:8086
---
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: Heapster
  name: heapster
  namespace: kube-system
spec:
  ports:
  - port: 80
    targetPort: 8082
  selector:
    k8s-app: heapster
```

### 4. 部署 heapster rbac

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: heapster
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:heapster
subjects:
- kind: ServiceAccount
  name: heapster
  namespace: kube-system
```

没有报错的话，这样就成功部署完成了，然后打开浏览器输入网址：`http://k8s-master-ip:grafana-service-nodeport`  即可看到收集的监控信息；