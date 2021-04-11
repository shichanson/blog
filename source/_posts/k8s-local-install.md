---
title: 本地搭建kubernetes环境
date: 2019-12-20 11:27:45
author: "Mayer Shi"
tags: ["kubernetes"]
categories: ["Container Cloud"]
---

此篇文章主要是介绍如何通过minikube快速部署一个k8s环境用作学习和实验。我演示的机器是macOS，k8s版本是1.15.x 最新版本。无需翻墙哦。

<!--more-->

**条件准备**
- 安装虚拟化软件virtualbox驱动
```bash
brew cask install virtualbox 
```

- 安装kubectl 客户端

```bash
brew install kubernetes-cli
```

- 下载阿里云minikube
```bash
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.2.0/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

**开始搭建**

- 启动minikube安装k8s 1.15.0 版本

```bash
minikube start --registry-mirror=https://registry.docker-cn.com --kubernetes-version v1.15.0
```

- 成果验证

```bash
# 查看节点
$ kubectl get node
NAME STATUS ROLES AGE VERSION
minikube Ready master 10m v1.15.0

# 查看kube-system ns 下的pod状态
$ kubectl get pod -n kube-system
NAME READY STATUS RESTARTS AGE
coredns-6967fb4995-9bplg 1/1 Running 0 10m
coredns-6967fb4995-9xn9t 1/1 Running 0 10m
etcd-minikube 1/1 Running 0 9m25s
kube-addon-manager-minikube 1/1 Running 0 9m8s
kube-apiserver-minikube 1/1 Running 0 9m24s
kube-controller-manager-minikube 1/1 Running 0 9m21s
kube-proxy-2gw65 1/1 Running 0 10m
kube-scheduler-minikube 1/1 Running 0 9m39s
storage-provisioner 1/1 Running 0 10m

```

- 开启dashboard
```bash
$ minikube addons enable dashboard
✅ dashboard was successfully enabled

$ minikube dashboard
```