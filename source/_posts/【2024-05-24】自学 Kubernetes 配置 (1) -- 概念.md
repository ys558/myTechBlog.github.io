---
title: 自学 Kubernetes (1) -- 概念和架构
date: 2024-05-24 08:05:00
tags:
  - Kubernetes
  - Docker
  - K8s
  - K3s
  - minikube

cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.61/articles/Kubernetes/cover.webp
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

<img src="https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/  articles/Kubernetes/kubernates_user.png" style="width: 80px"/>
<img src="https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.60/  articles/Kubernetes/kubernetes_crole.png" style="width: 80px"/>
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

## 架构

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
