---
title: docker 搭建 shadowsocks 服务器
date: 2019-11-20 11:58:10
author: "Mayer Shi"
tags: ["tools"]
categories: ["Ops"]
---
搭建科学上网工具，想必通过Docker 部署的方式是最高效和简单的。本篇文章则是介绍如何通过docker 搭建 shadowsock 科学上网服务。
<!--more-->

1. 安装docker命令行工具
```bash
sudo apt-get remove docker docker-engine docker.io

sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update
sudo apt-get install docker-ce
```

2. 执行如下命令，同时执行开放相应的防火墙
```bash
docker run -d --restart unless-stopped -p 12345:12345 oddrationale/docker-shadowsocks -s 0.0.0.0 -p 12345 -k test12345  -m aes-256-cfb
```