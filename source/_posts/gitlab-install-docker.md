---
title: docker swarm 部署 gitlab-ee 并设置https
date: 2019-12-19 11:58:10
author: "Mayer Shi"
tags: ["gitlab"]
categories: ["Ops"]
---
通过docker swarm 安装部署gitlab服务可以更方便的升级版本，以及高效运维。本篇主要介绍gitlab的docker swarm 安装方式。
<!--more-->

1. 初始化docker swarm 集群

```bash
 $ sudo docker swarm init
```

2. 创建lvm逻辑磁盘卷，格式化并挂载/gitlab目录下

```bash
 $ sudo fdisk /dev/vdb
 $ sudo pvcreate /dev/vdb1
 $ sudo vgcreate gitlab /dev/vdb1
 $ sudo lvcreate -L 199G -n gitlab gitlab
 $ sudo mkfs.xfs /dev/gitlab/gitlab
```

3. 创建gitlab数据挂载目录

```bash
 $ sudo mkdir -pv /gitlab/{config,data,logs}
```

4. 开始部署gitlab-ee版本

```bash
 $ sudo docker service create \
   --name "git-inside-gitlab" \
   --hostname git.test.cn \
   --network pilipa-network \
   --replicas 1 \
   --publish "mode=host,published=2222,target=22" \
   --publish "mode=host,published=80,target=80" \
   --publish "mode=host,published=443,target=443" \
   --mount type=bind,src=/gitlab/config,dst=/etc/gitlab \
   --mount type=bind,src=/gitlab/logs,dst=/var/log/gitlab \
   --mount type=bind,src=/gitlab/data,dst=/var/opt/gitlab \
"gitlab/gitlab-ee:11.4.9-ee.0"
```

5. 配置gitlab.rb
```ruby
external_url "https://git.test.cn"
nginx['redirect_http_to_https'] =true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/git.test.cn.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/git.test.cn.key"
```

6. 重新加载配置文件使其生效
```bash
 $ sudo gitlab-ctl reconfigure
```

7. 通过阿里云的SLB代理到ECS上搭建的gitlab服务上，然后设置dns解析即可。

