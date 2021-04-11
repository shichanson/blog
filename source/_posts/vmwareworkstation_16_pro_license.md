---
title: "VMware Workstation Pro 16 内置容器工具vctl彻底替换docker（内含批量激活密钥）"
author: "Mayer Shi"
tags: ["tools"]
categories: ["Ops"]
date: 2020-09-16 20:25:23
draft: false
---

本篇文章为您介绍VMware本月刚发布的新版本PC桌面虚拟化软件VMware Workstation 16 Pro的容器新特性。

<!--more-->


## 新特性有哪些？

* 支持容器和kubernetes
  * 通过vctl命令行工具build、run、pull、push 管理容器镜像
  * 支持通过KIND工具部署的kubernetes集群运行在workstation上。

   **注意：** 要求Windows10 1809版本或者更高 

* 虚拟机支持新的操作系统版本
  * RHEL 8.2
  * Debian 10.5
  * Fedora 32
  * CentOS 8.2
  * SLE 15 SP2 GA
  * FreeBSD 11.4
  * ESXi 7.0

等等新特性。



## VMware 新出的容器工具 vctl

### 使用说明

```bash
vctl - A CLI tool for the container engine powered by VMware Workstation
vctl Highlights:
• Build and run OCI containers.
• Push and pull container images between remote registries & local storage.
• Use a lightweight virtual machine (CRX VM) based on VMware Photon OS to host a container. Use 'vctl system config -h' to learn more.
• Easy shell access into virtual machine that hosts container. See 'vctl execvm’.

USAGE:
  vctl COMMAND [OPTIONS]

COMMANDS:
  build                        Build a container image from a Dockerfile.
  create                       Create a new container from a container image.
  describe                     Show details of a container.
  exec                         Execute a command within a running container.
  execvm                       Execute a command within a running virtual machine that hosts container.
  help                         Help about any command.
  images                       List container images.
  inspect                      Return low-level information on objects.
  kind                         Get system environment ready for vctl-based KIND.
  login                        Log in to a registry.
  logout                       Log out from a registry.
  ps                           List containers.
  pull                         Pull a container image from a registry.
  push                         Push a container image to a registry.
  rm                           Remove one or more containers.
  rmi                          Remove one or more container images.
  run                          Run a new container from a container image.
  start                        Start an existing container.
  stop                         Stop a container.
  system                       Manage the container engine.
  tag                          Tag container images.
  version                      Print the version of vctl.
  volume                       Manage volumes.

Run 'vctl COMMAND --help' for more information on a command.

OPTIONS:
  -h, --help   Help for vctl
```

### 小试牛刀

1. 启动vctl命令行工具

```bash
vctl system start
```
2. 基本用法

```bash
vctl.exe run -d  -p 80:80 --name nginx --restart unless-stopped daocloud.io/nginx #创建容器

vctl.exe ps # 查看运行中的容器。
────    ─────                      ───────                   ──               ─────       ──────    ─────────────
NAME    IMAGE                      COMMAND                   IP               PORTS       STATUS    CREATION TIME
────    ─────                      ───────                   ──               ─────       ──────    ─────────────
nginx   daocloud.io/nginx:latest   /docker-entrypoint.s...   192.168.197.10   80:80/tcp   running   2020-09-17T22:28:44+08:00

```

### 容器与虚拟机网络融合

通过上面vctl ps 命令查看到容器的IP其实是属于VMware Workstation 安装时自动创建的虚拟网络VMnet8子网下。是不是发现了什么？

对，就是容器和虚拟机之间的网络处于同一平面，虚拟机与容器在网络中地位同等。

### 总结

笔者认为稳态和敏态两种类型应用将在未来企业架构中长期并存的，所以VMware这种将容器和虚拟化高度融合的方案是非常巧妙的，容器满足互联网化敏态应用的高速迭代场景，虚拟化兼顾需要稳态的数据库等中间件以及容器化成本过高的老旧系统场景。

### VMware Workstation Pro 16 许可证密钥，批量永久激活

```bash
ZF3R0-FHED2-M80TY-8QYGC-NPKYF
YF390-0HF8P-M81RQ-2DXQE-M2UT6
ZF71R-DMX85-08DQY-8YMNC-PPHV8
```

下载地址：

https://n802.com/file/21152422-461565567