---
title: jenkins 初始化无法拉取插件的解决办法
date: 2019-11-21 10:58:10
author: "Mayer Shi"
tags: ["tools"]
categories: ["DevOps"]
---

jenkins被很多互联网用于devops工具，同时jenkins拥有庞大的插件生态使其拥有强大的功能，但是拉取插件的时候遇到了下载不了的问题，通过配置国内插件镜像中心来解决。
<!--more-->

环境描述: 生产环境的jenkins 出现故障，删除jenkins应用的pod，使其重建，但是拉取的插件中心是updates.jenkins.io站点，这个站点突然无法访问。我们jenkins始终无法启动。jenkins的deploy会启动一个initcontainer来初始化配置以及拉取插件hpi文件。但是始终无法拉取成功。

看到github中个解决方案，在jenkins的init容器中加了一个环境变量就可以从jenkins的镜像节点拉取完毕。

环境变量是：

```bash
JENKINS_UC_DOWNLOAD: https://mirrors.tuna.tsinghua.edu.cn/jenkins/
```
同时改下配置文件：

jenkins/hudson.model.UpdateCenter.xml

```bash
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/current/update-center.json</url>
  </site>
</sites>
```