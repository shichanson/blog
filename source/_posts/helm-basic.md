---
title: "helm基础概念"
author: "Mayer Shi"
tags: ["helm"]
categories: ["Container Cloud"]
date: 2018-12-11T10:28:13+08:00
draft: false
---
Kubernetes的巨大成功创造了一个工具生态系统，可以简化应用程序开发和部署的复杂性。而该系列文章主要分享helm在噼里啪技术团队的生产实践经验总结。
<!--more-->
**针对helm篇的实践落地方案分为如下几个部分：**

> * helm 基础理论篇
> * helm 使用技巧篇
> * 基础中间件服务运维篇
> * 微服务应用版本管理篇
> * 基于jenkins + helm的CICD方案
> * Helm 实践趟坑篇
> * 基于Helm Istio Jenkins灰度发布实践方案


### helm 是什么
**[helm](https://github.com/helm/helm.git)** 是一款可以帮你在k8s上很好运维管理复杂的应用包管理工具。如果把k8s比作CentOS操作系统的话，那么helm类似CentOS系统中的yum工具。

*这两个工具从某种程度来说的确很相似，yum可以解决rpm之间的依赖问题，而helm也可以解决应用与基础服务依赖关系。比如：WordPress应用启动之前需要启动MySQL，那就可以在WordPress的charts里定义需要依赖MySQL的charts。那么在部署WordPress的charts时，helm也会拉取并部署MySQL的Charts。*

### helm 名称概念

**Charts:** yum安装的rpm包则对应helm的charts。charts包含了整套复杂应用组件的k8s资源（Deployment、Service、Ingress、ConfigMap、Secret等）模板yaml文件以及模板对应value文件。Chart的目录结构如下：

```bash
mychart
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── ingress.yaml
│   └── service.yaml
└── values.yaml
```

* charts 目录存放依赖的 chart
* Chart.yaml 包含 Chart 的基本信息，包括 chart 版本，名称等
* templates 目录下存放应用一系列 k8s 资源的 yaml 模板
* _helpers.tpl 此文件中定义一些可重用的模板片断，此文件中的定义在任何资源定义模板中可用
* NOTES.txt 介绍 chart 部署后的帮助信息，如何使用 chart 等
* values.yaml 包含了必要的值定义（默认值）, 用于存储 templates 目录中模板文件中用到变量的值



**Release:** 扩展上述类比，要在基于CentOS的系统上安装NGNIX，您将运行yum install nginx。同样，要将NGINX安装到Kubernetes集群，您只需运行helm install nginx即可。每次向群集安装Charts都称为release。但是，与传统的操作系统软件包管理器不同，使用Helm可以轻松地将一个charts多次安装到单个集群中，每个release都有自己的特定配置。所以简单的来说一个release就是一个charts的实例化。


```bash
helm ls 
istio                             	1       	Thu Dec 13 11:50:08 2018	DEPLOYED	ack-istio-1.0.4             	1.0.4        	istio-system
```

**Repositories:** Helm Charts 还可以发布到存储库。这些charts可以发布到私有仓库，也可以是公共托管。像yum和apt一样，可以搜索它们以发现可用的charts。

```bash
helm search nginx
NAME                       	CHART VERSION	APP VERSION	DESCRIPTION
stable/nginx-ingress       	1.0.0        	0.21.0     	An nginx Ingress controller that uses ConfigMap to store ...
stable/nginx-ldapauth-proxy	0.1.2        	1.13.5     	nginx proxy with ldapauth
stable/nginx-lego          	0.3.1        	           	Chart for nginx-ingress-controller and kube-lego
stable/gcloud-endpoints    	0.1.2        	1          	DEPRECATED Develop, deploy, protect and monitor your APIs...
```

### HELM 核心组件

**helm** 是个客户端工具，它主要的作用如下：

* 本地chart开发
* 管理repositories
* 与tiller 服务端进行交互
   * 发送要安装的charts
   * 获取相关release的信息
   * 请求更新或者删除已存在的release
   
**tiller** 是一个部署在k8s集群内部的一个与helm客户端进行交互同时也与k8s api连接的服务。主要负责如下功能：

* 监听来自helm客户端传入的请求
* 将charts和配置组合渲染来构建一个release
* 将charts部署到k8s集群中并跟踪后续版本
* 通过与k8s进行交互来更新以及删除集群中存在的release。

<center>![](/media/posts/media/jh.png)</center>
简而言之，helm 客户端负责管理charts, tiller 服务端负责管理release生命周期。

### helm 内部实现
1. Helm客户端使用Go编程语言编写，并使用gRPC协议与Tiller服务器进行交互。
2. Tiller服务端也是用Go编写的。它提供了一个与客户端连接的gRPC服务器，它使用Kubernetes客户端库与Kubernetes进行通信。目前，该库使用REST + JSON。
3. Tiller服务器将信息存储在位于Kubernetes内的ConfigMaps中。它不需要自己的数据库。

### release 管理机制

![](/media/posts/media/release.png)

#### 创建 release

* helm 客户端从指定的目录或本地 tar 文件或远程 repo 仓库解析出 chart 的结构信息
* helm 客户端指定的 chart 结构和 values 信息通过 gRPC 传递给 Tiller
* Tiller 服务端根据 chart 和 values 生成一个 release
* Tiller 将 install release 请求直接传递给 kube-apiserver

#### 更新 release

* helm 客户端将需要更新的 chart 的 release 名称 chart 结构和 value 信息传给 Tiller
* Tiller 将收到的信息生成新的 release，并同时更新这个 release 的 history
* Tiller 将新的 release 传递给 kube-apiserver 进行更新

#### 删除 release

* helm 客户端从指定的目录或本地 tar 文件或远程 repo 仓库解析出 chart 的结构信息
* helm 客户端指定的 chart 结构和 values 信息通过 gRPC 传递给 Tiller
* Tiller 服务端根据 chart 和 values 生成一个 release
* Tiller 将 delete release 请求直接传递给 kube-apiserver