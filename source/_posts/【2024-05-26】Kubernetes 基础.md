---
title: Kubernetes 基础
date: 2024-05-26 08:05:00
tags:
  - Kubernetes
  - Docker
  - K8s
  - K3s
  - minikube
  - multipass
  - kubectl

cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.62/articles/Kubernetes/cover3.png
---

Kubernetes 是一个开源的容器编排平台，可以轻松部署、扩展和管理容器化应用程序。这篇文章是我自学 Kubernetes 的过程，我将过程记录下来。

<!-- more -->

## 基础概念

弄清楚 Kubernetes 服务，要先弄清以下几个 Kubernates 的基础概念

### Node

<img src="https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/Kubernetes_node.png" style="width: 80px"/>

一个节点（Node）是一个虚拟机或者物理机

### Pod

<img src="https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/kubernetes_pod.png" style="width: 80px"/>

Pod 是 Kubernetes 最小的调度单元，一个 Pod 可以是一个或者多个容器（container）的组合，他创建了一个容器的运行环境，在这个容器中，容器可以共享一些资源，比如网络，存储以及一些运行时的配置等等。

比如我们有一个应用程序或一个数据库，我们可以将其分别放在两个不同的 pod 中。一个 Pod 运行一个容器，这是一种最佳实践。

当然，一些特殊情况，一个 Pod 也可运行多个容器，这种情况也仅限于这些容器是高度耦合的情况，即我们所说的边车模式 （Sidecar），如下图：

<img src="https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/kubernates_sidecar.png" />

### Service

<img src="https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/Kubernetes_svc.png" style="width: 80px"/>

每个 pod 都会有一个集群内部的 IP 地址，外部不可以访问到。pod 并不是一个稳定的实例，十分容易被创建或销毁，当 pod 发生故障时，会被销毁，IP 地址就会变更。这时候就需要 Kubernetes 的 Service 来管理和创建 IP 地址的变更，如下图：

![](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/kubernetes_svc1.png)

service 上图的 service 连接数据库的 pod 和 App 的 pod，当 App 访问 Service 时，Service 会分配对应的数据库 IP 给 App 进行访问，如下图：

![](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/kubernetes_svc2.png)

### Ingress

<img src="https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/kubernetes_ing.png" style="width: 80px"/>

当我们服务在本地开发时，可以用 IP 和端口号访问节点，而当部署到生产环境时，则必须配置域名，通过域名来访问

而 `Ingress` 就是用来管理从集群外部访问集群内部服务的入口和方式的。可以根据 Ingress 规定不同的转发规则，访问集群内部不同的 Service

除此之外，`Ingress` 还可以配置 SSL 证书、负载均衡等等

### ConfigMap

<img src="https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/kubernestes_confogmap.png" style="width: 80px"/>

配置完上述描述的配置后，我们就可以提供对外服务了

但还忽略一个问题，例如应用程序的 pod 访问数据库 pod 时，开发中我们会把数据库的地址、端口等连接信息写到配置文件或者环境变量中，然后在应用程序中读取这些信息。这样做，配置信息与应用程序就耦合在一起了。

一旦数据库的地址或者端口发生变化，那我们就要重新编译应用程序，再重新部署到集群中。

为了解决以上问题，Kubernetes 提供了 `ConfigMap` 的功能，将一些配置信息封装起来，就可以在应用程序中读取和使用了。

### Secret

<img src="https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/kubernestes_secret.png" style="width: 80px"/>

就是 `ConfigMap`的延伸，对其里面的内容进行 base64 进行编码。`ConfigMap`是明文的，比如我们会配置数据库访问的用户名，密码等，Secret 对这些敏感信息做编码，注意，还不是加密。base64 编码可以通过编程语言进行解析。

### User, C.role, Sa

<div style="display: flex; justify-content: flex-start; align-items: center;">

<img src="https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/kubernates_user.png" style="width: 80px"/>
<img src="https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/kubernetes_crole.png" style="width: 80px"/>
<img src="https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/kubernetes_sa.png" style="width: 80px"/>
</div>

这 3 个分别是身份认证、网络安全、访问控制

### Volume

我们知道 pod 是很容易被销毁的，数据库的 pod 一旦被销毁，其后果可能是灾难性的。K8s 提供了 `Volume` 组件，他可以将数据库 pod 的数据，同时同步挂载到本地的磁盘上或者集群外部的远程存储上。

这样即使 pod 被销毁，数据也能持久化存储。

### Deployment

现在为止，我们还须考虑到应用程序的 pod 高可用姓，比如某个应该程序的节点发生故障，或者节点需要升级或更新维护时，应用程序就会停止服务。这显然是不能接受的。

解决方法是复制多个 Node，并用 Service 进行几种管理，此时 k8s 会自动将请求分散到其他正常工作的节点上。

而 k8s 提供了一个 `Deployment` 组件，用于管理所有应用程序 pod，他可以定义和管理应用程序的副本数量，以及应用程序的更新策略，简化应用程序的部署和更新操作。如下图：

![deployment](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/kubernates_deployment.png)

`Deployment` 还具备

- 副本控制 ( 就是定义应用的副本数量，如上图，定义了 3 个，如果其中一个副本有故障，那么他会自动创建一个新的副本来替代原有副本，维持副本的数量在 3 个 )
- 滚动更新 ( 可以定义应用程序的升级策略，用于应用程序的版本升级 )
- 自动扩容

等高级功能。

### StatefulSet

相较于上面 `Deployment` 管理应用的副本，数据库其实也需要对应的副本管理工具，这个工具是 `StatefulSet`。

和应用不同的是，数据库的每个副本，都是有独立的状态的，而应用是无状态的。简单来说，10 秒前存入的数据的状态，和现在的数据的状态往往是不一样的。所以数据库的 pod，需要把数据同步到其他的副本中，或者把数据写入到一个共享的存储中。

和 `Deployment` 的功能类似，但 `StatefulSet` 还保证每个副本都有自己稳定的网络标识符和持久化存储。所以，像数据库，缓存，消息队列等等，以及一些保留了会话状态的应用，都使用 `StatefulSet` 来部署。

![](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/articles/Kubernetes/kubernates_statefulset.png)

**当然，一种更好的实践是将有状态的 pod 从各个节点中剥离出来，进行单独的部署**

## 基础架构

k8s 是一个典型的 `master-worker`架构。所以，比如以上所提到的部署了应用和数据库的节点，也被称为工作节点 `worker node`。而工作节点通常有多个，`master`只有一个，用于管理所有 `worker node`

### 工作节点 worker node

![](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.61/articles/Kubernetes/kuberneates_1_node_structure.png)

为了能提供对外服务，每个 Node 节点上都会包含 3 个对应的服务组件，分别是:

- `kubelet`，负责管理和维护每个节点上的 pod，也会定期从 `api-server` 组件接受新的 pod 规范，监视工作节点的运行情况，将信息汇报给 `api-server`
- `kube-proxy`，负责为 pod 提供网络代理和负载均衡
- `container runtime` 容器运行时

`container runtime` 就是我们常说的 Docker-Engine，每个节点都必须有容器运行时，当然，容器运行时也不止 docker，还有如：`Containerd`，`CRI-O`，`Mirantis`

通常情况下，一个集群包含多个节点，节点间的通信和负载均衡器就是通过 `k-proxy`

### 管理节点 master node

![](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.61/articles/Kubernetes/k8s_master_worker.png)

从上图可以看出，master 节点的结构和 worker 节点完全不同，他分别包括 `API server`，`scheduler`，`control manager`， `etcd`和 `cloud control manager`，通过 `api server` 来管理所有工作节点。

#### `API server`

![](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.61/articles/Kubernetes/k8s_api_server.png)

就像一个集群的网关，是整个系统的入口。所有请求都会经过他，再由他分发给不同的组件进行处理。而且所有的组件间也会通过 API server 进行通信。

例如，当我们部署一个新应用 pod 时，那么可以用客户端，例如 `kubectl` 命令行，`dashboard` 或者其他的 UI 界面工具。

当使用`kubectl` 命令行创建新的 pod 时，这个请求会先到达 api server 并验证这个请求的合法性，验证通过后再转发给相应的 `Scheduler` 调度器组件进行处理

#### `Scheduler`

负责监控集群中所有节点资源的使用情况，根据一些调度策略，将 pod 调度到合适的节点上运行。

例如下图：

![](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.61/articles/Kubernetes/k8s_scheduler.png)

调度器会将 pod 部署时较为空闲的节点上运行

#### `controller manager`

![](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.61/articles/Kubernetes/k8s_cloud_control_manager.png)

控制者管理器，复制管理集群中各种资源对象的状态。比如任何一个节点上的 pod 发生故障时，必须有一种机制监测到这个故障，尽快对其进行处理。例如重启该 pod 或者新启用一个 pod 进行替换。

controller manager 是如何知道哪个节点发生故障的呢？这就需要下面介绍的 `etcd` 组件

#### `etcd`

这个是一个高可用的键值存储系统，类似 `Redis`，存储集群中所有资源对象的状态信息。比如哪个 `pod` 挂掉，哪个 `pod` 又被新创建了。可以理解为集群的大脑。是整个集群的数据存储中心。

#### `cloud controller manager`

云控制器管理器，他是用来和各种云上的 api 进行交互的。而且他能提供一致的管理接口，使得用户可以在不同云平台中管理他们的应用程序。

## minicube 搭建单节点环境

本文为了实践 k8s，在本地搭建了个模拟线上的简易版环境来学习 k8s。

[minikube](https://minikube.sigs.k8s.io/docs/) 是一个轻量级的 Kubernetes 实现，可在本地计算机上创建虚拟机，并部署仅包含一个节点的简单集群。

一般生产环境的 Kubernetes 集群，一般都是使用云厂商提供的 Kubernetes 服务，如阿里云的 ACK、腾讯云的 TKE、华为云的 CCE 、AWS 的 EKS 等 。他们需要多个节点的集群才能实现高可用。每个节点都是一个服务器或者虚拟机。所以本地开发环境可以使用 minikube 来验证 Kubernetes 的功能。

### 安装

```bash
# macOS
brew install minikube

# Windows
choco install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### minikube 常用命令

```bash
# 帮助菜单
minikube

# 启动
# 后面的参数是国内网络问题下载镜像失败时加，如果下载成功，不加也可以：
minikube start --image-mirror-country=cn

# 检查状态
minikube status
```

### kubectl 命令

安装完成 minikube 之后，附带 `kubectl` 命令，这个命令就是 Kubernetes 自带的和集群交互的命令

```bash
(base) zyzy:~ $ kubectl get nodes
NAME       STATUS   ROLES           AGE    VERSION
minikube   Ready    control-plane   8m9s   v1.30.0
```

## multipass 虚拟机 + K3s 的多节点集群搭建

minikube 是一个单节点的集群环境，但是稍微复杂一点的环境就不适用了。这里我们用虚拟机和 k3s 技术，模拟一个多节点环境的搭建。

### `multipass` 安装

超级简单，点击这里 [multipass](https://multipass.run/install) 点击相应的版本安装即可

### `multipass` 常用命令

- 创建一个 ubuntu1 虚拟机

```bash
(base) zyzy:~ $ multipass launch --name k3s --cpus 2 --memory 4G --disk 10G
Launched: k3s
```

- 查看本机所运行的所有虚拟机

```bash
(base) zyzy:~ $ multipass ls
Name                    State             IPv4             Image
k3s                     Running           192.168.64.3     Ubuntu 24.04 LTS
```

- 进入 k3s 虚拟机系统

```bash
(base) zyzy:~ $ multipass shell k3s
。。。
ubuntu@k3s:~$
```

- 启用虚拟机

```bash
(base) zyzy:~ $ multipass start k3s
```

- 停止虚拟机

```bash
(base) zyzy:~ $ multipass stop k3s
```

- 删除虚拟机

```bash
(base) zyzy:~ $ multipass delete k3s
# 永久删除已经delete的虚拟机
(base) zyzy:~ $ multipass purge

```

### `k3s` 安装

登录进虚拟机后，安装 k3s 的 master 节点：

```bash
curl -sfL https://get.k3s.io | sh -

# 国内用户使用：
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -
```

`kubectl`命令验证是否安装成功：

```bash
ubuntu@k3s:~$ sudo kubectl get nodes
NAME   STATUS   ROLES                  AGE   VERSION
k3s    Ready    control-plane,master   23s   v1.29.4+k3s1
```

### 创建 worker node

上面我们已经安装好了 master node，下面我们来安装 worker node

先在 master 节点上获取 token，并复制下来

```zsh
ubuntu@k3s:~$ sudo cat /var/lib/rancher/k3s/server/node-token
```

重新开一个 mac 的终端，输入

```bash
(base) zyzy:~ $ TOKEN=$(multipass exec k3s sudo cat /var/lib/rancher/k3s/server/node-token)
(base) zyzy:~ $ echo $TOKEN
K10dc6a...
```

可以看到能 echo 打印出相应的 token，证明 token 配置成功，再将 master 的 IP 地址配置到本地并验证

```bash
(base) zyzy:~ $ MASTER_IP=$(multipass info k3s | grep IPv4 | awk '{print $2}')
(base) zyzy:~ $ echo $MASTER_IP
192.168.64.3
```

我们可以继续创建两个 worker 节点并将其配置到 master 节点中

```bash
multipass launch --name workder1 --cups 2 --memory 8G --disk 10G
multipass launch --name workder2 --cups 2 --memory 8G --disk 10G

# 在worker节点虚拟机上安装k3s，得用国内的镜像：
 for f in 1 2; do
     multipass exec worker$f -- bash -c "curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn K3S_URL=\"https://$MASTER_IP:6443\" K3S_TOKEN=\"$TOKEN\" sh -"
 done
```

验证，让我们进入 k3s 的 master 节点中，可以看到 `worker1` `worker2`节点已经加入到集群中：

```bash
(base) zyzy:~ $ multipass shell k3s
...
ubuntu@k3s:~$ sudo kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
k3s       Ready    control-plane,master   55m   v1.29.4+k3s1
worker1   Ready    <none>                 15m   v1.29.4+k3s1
worker2   Ready    <none>                 15m   v1.29.4+k3s1
```

## `kubectl` 管理工具

```bash
# 进入k3smaster节点：
multipass shell k3s
```

### 基础命令

```bash
# 查看帮助文档
kubectl --help

# 查看API版本
kubectl api-versions

# 查看集群信息
kubectl cluster-info

```

### 创建 `kubectl create` 及运行`kubectl (run | apply)`

```bash
# 创建并运行一个指定的镜像
kubectl run NAME --image=image [params...]
# e.g. 创建并运行一个名字为nginx的Pod
kubectl run nginx --image=nginx

# 根据nginx.yaml配置文件创建资源
kubectl create -f nginx.yaml
# 根据URL创建资源
kubectl create -f https://k8s.io/examples/application/deployment.yaml
# 根据目录下的所有配置文件创建资源
kubectl create -f ./dir

# 通过文件名或标准输入配置资源
kubectl apply -f (-k DIRECTORY | -f FILENAME | stdin)
# e.g.
# 根据nginx.yaml配置文件创建资源
kubectl apply -f nginx.yaml
```

### 查看 `kubectl (get | describe)`

```bash
# 其中，RESOURCE可以是以下类型：
kubectl get pods / po         # 查看Pod
kubectl get svc               # 查看Service
kubectl get deploy            # 查看Deployment
kubectl get rs                # 查看ReplicaSet
kubectl get cm                # 查看ConfigMap
kubectl get secret            # 查看Secret
kubectl get ing               # 查看Ingress
kubectl get pv                # 查看PersistentVolume
kubectl get pvc               # 查看PersistentVolumeClaim
kubectl get ns                # 查看Namespace
kubectl get node              # 查看Node
kubectl get all               # 查看所有资源

# 后面还可以加上 -o wide 参数来查看更多信息
kubectl get pods -o wide

# 查看某一类型资源的详细信息
kubectl describe RESOURCE NAME
# e.g. 查看名字为nginx的Pod的详细信息
kubectl describe pod nginx
```

### 调试 `kubectl logs` 交互 `kubectl exec`

比如上面用 replilcaset 创建的 pod，查看日志

```bash
sudo kubectl logs nginx-deployment-6d6565499c-bn7gd
```

也可以进入 pod 内部查看：

```bash
ubuntu@k3s:~$ sudo kubectl exec -it nginx-deployment-6d6565499c-bn7gd -- /bin/bash
root@nginx-deployment-6d6565499c-bn7gd:/#
```

### 修改，删除和清理资源

```bash
# 更新某个资源的标签
kubectl label RESOURCE NAME KEY_1=VALUE_1 ... KEY_N=VALUE_N
# e.g. 更新名字为nginx的Pod的标签
kubectl label pod nginx

# 删除某个资源
kubectl delete RESOURCE NAME
# e.g. 删除名字为nginx的Pod
kubectl delete pod nginx

# 删除某个资源的所有实例
kubectl delete RESOURCE --all
# e.g. 删除所有Pod
kubectl delete pod --all

# 根据YAML配置文件删除资源
kubectl delete -f FILENAME
# e.g. 根据nginx.yaml配置文件删除资源
kubectl delete -f nginx.yaml

# 设置某个资源的副本数
kubectl scale --replicas=COUNT RESOURCE NAME
# e.g. 设置名字为nginx的Deployment的副本数为3
kubectl scale --replicas=3 deployment/nginx
```

例如：

```bash
# 删除名为nginx的pod
ubuntu@k3s:~$ sudo kubectl delete pod nginx
pod "nginx" deleted

# 删除名为 nginx-deployment 的 deployment
ubuntu@k3s:~$ sudo kubectl delete deployment nginx-deployment
deployment.apps "nginx-deployment" deleted
ubuntu@k3s:~$ sudo kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   10h
```

## Deployment

### 利用 `kubectl` 创建

以上是创建的的基本命令，但实际操作中，创建一个 pod， 更提倡使用 Deployment 这样的上层资源来创建，具体如下：

```bash
# nginx-deployment 为别名
kubectl create deployment nginx-deployment --image=nginx

# 查看已创建的deployment
sudo kubectl get deployment
```

查看已创建的 Deployment 资源：

```bash
# 查看所有pod，包括非deployment创建的
ubuntu@k3s:~$ sudo kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx                               1/1     Running   0          6h44m
nginx-deployment-6d6565499c-bn7gd   1/1     Running   0          2m35s

# 通过 deployment 查看 pod
ubuntu@k3s:~$ sudo kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           85s

# 通过 replicaset 查看 pod
ubuntu@k3s:~$ sudo kubectl get replicaset
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6d6565499c   1         1         1       6m52s
```

修改 `replicaset` 副本数量：

上面的 replicaset 是通过 deployment 创建时自动生成的，实际上他是一个管理副本数量的组件

可以通过 `edit` 来编辑 replicaset 的 ymal 文本

```bash
ubuntu@k3s:~$ sudo kubectl edit deployment nginx-deployment
deployment.apps/nginx-deployment edited
```

当输入以上命令后，会自动进入 nginx-deployment 的 yaml 文件进行编辑，我们把`replicas`值改为 3 后保存退出

```bash
ubuntu@k3s:~$ sudo kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx                               1/1     Running   0          7h11m
nginx-deployment-6d6565499c-bn7gd   1/1     Running   0          29m
nginx-deployment-6d6565499c-m26m2   1/1     Running   0          4m28s
nginx-deployment-6d6565499c-sjvql   1/1     Running   0          4m27s

ubuntu@k3s:~$ sudo kubectl get replicaset
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6d6565499c   3         3         3       29m
```

查看后发现 pod 多了 3 个同属于一个 replicaset 的 pod

### 利用 yaml 文件创建 (重点)

`kubectl` 命令行方式创建 pod 可以加入很多参数，例如我们上面举例过的 `--image=nginx` ，这个参数如果一多或者嵌套的话，那么命令行的方式是非常麻烦的，这时，更推荐利用 `yaml` 文件的方式创建。

`yaml`类似 Docerfile，创建后再用 `kubectl` 进行调用，pod 就会自动更新了。

具体步骤：

1. 创建并编辑 yaml 文件

```bash
ubuntu@k3s:~$ vim nginx-deployment.yaml
```

编辑如下，下面是一个最基本的 yml 文件

```yaml
apiVersion: apps/v1
metadata:
  name: nginx-deployment
spec: # 顶层 spec 定义 deployment 的配置信息
  selector:
    matchLabels:
      app: nginx
  replicas: 3 # replicas 定义副本数量
  template:
    metadata:
      labels:
        app: nginx
    spec: # 底层 spec 定义 pod 的详细信息
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80 #向外暴露的端口
```

- `apiVersion` : 指定 apiServer 版本，格式是：组别/版本号，group/version，组别经常有 apps(应用), batch(批处理), autoscaling(自动扩缩容) 等等
- `kind`: 用来指定资源对象的类型
- `metadata`: 定义资源对象的元数据，比如资源的名称，标签，命名空间等。
- `spec`: 是 specification 的缩写，定义资源的各种配置信息，包括有多少个副本 (`replicas`), 像上面的 spec 有两层嵌套，第一层是定义 deployment 的信息，第二层是定义 Pod 的配置信息

2.  经 yml 文件创建创建资源 `kubectl create -f`

```bash
ubuntu@k3s:~$ sudo kubectl create -f nginx-deployment.yaml
deployment.apps/nginx-deployment created

# 检验
ubuntu@k3s:~$ sudo kubectl get all
NAME                                   READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-cd5968d5b-h4nng   1/1     Running   0          3h42m
pod/nginx-deployment-cd5968d5b-bg649   1/1     Running   0          3h42m
pod/nginx-deployment-cd5968d5b-qcs7t   1/1     Running   0          3h42m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   16h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           3h42m

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-cd5968d5b   3         3         3       3h42m
```

3. 修改(`kubectl apply`) 删除 (`kubectl delete`)

```bash
# 通过文件名或标准输入配置资源
kubectl apply -f (-k DIRECTORY | -f FILENAME | stdin)
# e.g.
# 根据nginx.yaml配置文件创建资源
kubectl apply -f nginx.yaml

# 删除
kubectl delete -f nginx-deployment.yaml
```

## `Service`

### 利用 `kubectl` 创建

上面我们用 Deployment 创建了 5 个 pod，我们可以用以下命令查询到其 IP 地址：

```bash
ubuntu@k3s:~$ sudo kubectl get pod -o wide
NAME                               READY   STATUS    RESTARTS   AGE     IP           NODE      NOMINATED NODE   READINESS GATES
nginx-deployment-cd5968d5b-h4nng   1/1     Running   0          4h3m    10.42.0.10   k3s       <none>           <none>
nginx-deployment-cd5968d5b-bg649   1/1     Running   0          4h3m    10.42.3.4    worker2   <none>           <none>
nginx-deployment-cd5968d5b-qcs7t   1/1     Running   0          4h3m    10.42.1.5    worker1   <none>           <none>
nginx-deployment-cd5968d5b-rml8k   1/1     Running   0          2m32s   10.42.3.5    worker2   <none>           <none>
nginx-deployment-cd5968d5b-z8k52   1/1     Running   0          2m32s   10.42.1.6    worker1   <none>           <none>
```

**这些 pod 的 IP 地址都是节点内部地址**，无法提供对外服务，所以我们必须创建 `Service` 并配置其外部服务

创建步骤：

```bash
# 创建服务命令
sudo kubectl create service nginx-service
# 对外暴露已有 deployment
sudo kubectl expose deployment nginx-deployment
```

查看已经创建服务：

```bash
ubuntu@k3s:~$ sudo kubectl get svc
NAME               TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
kubernetes         ClusterIP   10.43.0.1     <none>        443/TCP   17h
nginx-deployment   ClusterIP   10.43.54.95   <none>        80/TCP    22s
```

删除服务：

```bash
sudo kubectl delete service nginx-deployment
```

### 利用 yml 文件创建（重点）

```bash
vim nginx-service.yaml
```

内容如下：

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort # 指定对外暴露类型
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80 # 对外端口
    targetPort: 80 # pod内部端口
    nodePort: 30080
```

- `type`: 该参数指定了对外端口类型，如果不定义该字段，那么默认是 `Cluster IP` 类型服务，只能在节点内部间访问，`NodePort`为对外服务，具体类型参加下表
- `nodePort`： 该参数指定了对外暴露的端口号，范围必须在 `30000~32767`之间

|   服务类型   |                    描述                    |
| :----------: | :----------------------------------------: |
|  ClusterIP   |          默认类型，集群内部的服务          |
|   NodePort   |    节点端口类型，将服务公开到集群节点上    |
| LoadBalancer | 负载均衡类型，将服务公开到外部负载均衡器上 |
| ExternalName |  外部名称类型，将服务映射到一个外部域名上  |
|   Headless   |   无头类型，主要用于 DNS 解析和服务发现    |

启用：

```bash
sudo kubectl apply -f nginx-service.yaml
```

查看 pod 的 ip 地址：

```bash
ubuntu@k3s:~$ sudo kubectl get nodes -o wide
NAME      STATUS   ROLES                  AGE   VERSION        INTERNAL-IP    EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION     CONTAINER-RUNTIME
k3s       Ready    control-plane,master   17h   v1.29.4+k3s1   192.168.64.3   <none>        Ubuntu 24.04 LTS   6.8.0-31-generic   containerd://1.7.15-k3s1
worker2   Ready    <none>                 17h   v1.29.4+k3s1   192.168.64.6   <none>        Ubuntu 24.04 LTS   6.8.0-31-generic   containerd://1.7.15-k3s1
worker1   Ready    <none>                 17h   v1.29.4+k3s1   192.168.64.5   <none>        Ubuntu 24.04 LTS   6.8.0-31-generic   containerd://1.7.15-k3s1
```

复制一个任意一个 worker 节点的 ip，并在浏览器打开，发现打开成功，端口采用我们配置的 `nodePort: 30080`

![deployment](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.62/articles/Kubernetes/k8s_service_fr_outside.png)

## [portainer](https://www.portainer.io/) 图形界面管理工具

### 安装及访问

有两种方法：

1. 获取 `portainer` 的 yaml 文件配置安装，安装在 master 节点

   - 直接在 master 节点，即我们创建的 k3s 上执行`kubectl apply -n portainer -f https://downloads.portainer.io/ce2-19/portainer.yaml`
   - `-n` 参数指定 pod 的命名空间，看起来更直观，有隔离不同项目和环境的作用
   - 本地访问 master 节点的 ip+端口号 30777，例如浏览器访问：`192.168.64.6:30777`

2. 利用 `helm` 安装，安装在本地

   - [helm](https://helm.sh/) 是 k8s 的包管理工具，利用以下命令安装

   - Mac 电脑：

   ```bash
   brew install helm
   helm repo add portainer https://portainer.github.io/k8s/
   helm update portainer
   helm upgrade --install --create-namespace -n portainer portainer portainer/portainer --set tls.force=true
   ```

### 删除

删除 `portainer` 时，由于我们加了命名空间，所以也要指定命名空间，如下：

```bash
sudo kubectl delete namespace portainer
```

### 命名空间 `kubectl get ns | namespace`

上面提到的命名空间，可以用以下命令查看所有集群里的命名空间

```bash
ubuntu@k3s:~$ sudo kubectl get ns
NAME              STATUS   AGE
kube-system       Active   18h
kube-public       Active   18h
kube-node-lease   Active   18h
default           Active   18h
portainer         Active   10m
```

上面 `portainer`是我们指定的命名空间，其余是 k8s 默认创建的

平时创建任何资源如果不指定命名空间，则全部默认放到 default 的命名空间里

![namespace](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.62/articles/Kubernetes/k8s_namespace.png)

从上图可以发现，如创建了命名空间的资源，查看时必须加上 `-n` 参数

### 访问

这里我们要输入 master 节点 IP，而不是 portainer 的 IP
