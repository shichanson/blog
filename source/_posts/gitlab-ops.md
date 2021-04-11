---
title: "gitlab 服务器参数优化"
author: "Mayer Shi"
tags: ["gitlab"]
categories: ["Ops"]
date: 2019-05-19T11:09:58+08:00
draft: true
---

很多企业在生产实践中采用 spring cloud 框架做开发以及 k8s 做部署的方案，但是在实践中也遇到测试环境容器的私有 IP 无法被调用本地应用调用的问题。

<!--more-->

### 问题背景

我们公司的研发分为天津和北京两个开发团队共同开发一个整体应用，负责不同功能组件的开发。同时开发环境也是部署在 k8s 上的，所以服务暴露给开发调用的方式并不是很多。比如： 采用 loadbalancer，ingress 等方案，但是这种不够灵活，同时也无法解决服务注册和发现的问题。

### 出现问题

1. 生产环境某个应用系统部分服务组件容器出现的不断重启现象。
2. gitlab web 服务正常，但是 git shell 推送时常卡慢甚至无法推送代码。
3. jenkins 测试环境也是大面积的构建任务失败。
4. gitlab 容器内存占用居高不下。

### 故障原因

1. 生产环境出问题的应用系统进过排查后，得出的原因在于应用采用的是 spring cloud 框架，而应用的配置下发拉取采用的是 spring cloud 框架的 config server 组件。应用最终的配置文件是存储在 gitlab 中。由于应用在配置的时候没有关闭应用运行后继续拉取配置的选项。应用定期会自动从 config server 拉取配置，但是 config server 从 gitlab 的 repo 同步配置的时候失败了，运行的应用也就无法拉取到相应的配置，最终导致应用无法正常启动应用且 pod 不断重启的原因。
   题外话： 用 gitlab 做 config server 配置文件的管理的管理在社区还是饱受争议的。另外我们也在调研通过 k8s configmap 作为配置文件管理的方案。

2. 我们的日常测试环境的构建频率相当频发，所以拉取代码也会很频繁，但是构建任务大规模失败的原因在于，每次发布新版本的时候，我们会通过 helm 进行版本管理，charts 的变更会被推送到 gitlab 中。基本上是推送 charts 变更的时候出的问题。

3. 代码无法推送等以上各种现象其实本质上都是 gitlab 出了问题。同时我们也发现 gitlab 容器消耗内存也是特别的高。

### 调优方式

按照官方文档说明上的介绍，当前 gitlab 能承载的能力最起码也得有一两千人的规模，事实是一百多人的规模就出现 gitlab 各种卡顿的问题。因此我们对 gitlab 的配置参数进行优化。
![](/media/posts/media/gitlab.png)

1.增加进程数和超时时间

```bash
# 超时时间默认值60s
unicorn['worker_timeout'] = 60
# 不能低于2，否则卡死，官方推荐 worker=CPU核数+1，CPU是4C。
unicorn['worker_processes'] = 5
```

2.GitLab 默认使用了 PostgreSQL，优化 PostgreSQL

```bash
# 减少数据库缓存大小 默认256，可适当改小
postgresql['shared_buffers'] = "256MB"

# 减少数据库并发数
postgresql['max_worker_processes'] = 8

# 减少sidekiq并发数
sidekiq['concurrency'] = 10
```

3.减少 unicorn 内存使用

```bash
# 减少内存
unicorn['worker_memory_limit_min'] = "200 * 1 << 20"
unicorn['worker_memory_limit_max'] = "300 * 1 << 20"
```

4.重新加载配置以及重启服务生效配置

```bash
# 重新加载配置
gitlab-ctl reconfigure

# 重启所有组件生效配置
gitlab-ctl restart
```

总结： 经过以上的参数配置的调优以及重启生效后，的确解决了生产环境以及 gitlab 的各种问题。
