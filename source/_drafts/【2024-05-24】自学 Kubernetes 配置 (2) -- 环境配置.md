---
title: 自学 Kubernetes (2) -- 环境配置
date: 2024-05-24 08:05:00
tags:
  - Kubernetes
  - Docker
  - K8s
  - K3s
  - minikube
---

最近在自学 Kubernetes，容器编排框架，需要较为复杂的架构，因为当个电脑是无法模拟出较为复杂的应用场景的。所以这里把自学的环境搭记录下来。

<!-- more -->

## minikube

minikube 是一个轻量级的 Kubernetes 实现，可在本地计算机上创建虚拟机，并部署仅包含一个节点的简单集群。

一般生产环境的 Kubernetes 集群，一般都是使用云厂商提供的 Kubernetes 服务，如阿里云的 ACK、腾讯云的 TKE、华为云的 CCE 、AWS 的 EKS 等 。他们需要多个节点的集群才能实现高可用。每个节点都是一个服务器或者虚拟机。所以本地开发环境可以使用 minikube 来验证 Kubernetes 的功能。

## multipass 虚拟机

安装虚拟机的目的是模拟多台电脑的场景，方便调试。

### 0. 安装

超级简单，这里这里 [multipass](https://multipass.run/install) 点击相应的版本安装即可

### 1. 创建一个 ubuntu1 虚拟机

```bash
(base) zyzy:~ $ multipass launch --name ubuntu1 --cpus 4 --memory 8G
Launched: ubuntu1
```

### 2. 查看本机所运行的所有虚拟机

```bash
(base) zyzy:~ $ multipass ls
Name                    State             IPv4             Image
ubuntu1                 Running           192.168.64.2     Ubuntu 24.04 LTS
```

### 3. 进入 Ubuntu1 虚拟机系统

```bash
(base) zyzy:~ $ multipass shell ubuntu1
。。。
ubuntu@ubuntu1:~$
```

### 4. 启用虚拟机

```bash
(base) zyzy:~ $ multipass start ubuntu1
```

### 5. 停止虚拟机

```bash
(base) zyzy:~ $ multipass stop ubuntu1
```
