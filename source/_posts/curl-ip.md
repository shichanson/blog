---
title: "命令行下如何获取外网地址"
author: "Mayer Shi"
tags: ["cmd"]
categories: ["Ops"]
date: 2019-12-19T11:09:58+08:00
draft: true
---

公司业务涉及到数据安全层面的时候，往往会通过IP白名单的方式来控制数据或者应用的访问。如何快速获取当前主机所在外网IP是个刚需。

<!--more-->

**第一种方式：**

```bash
curl cip.cc
curl ipinfo.io
curl myip.ipip.net
curl http://members.3322.org/dyndns/getip
curl https://ip.cn  # 推荐使用这个
curl httpbin.org/ip
curl ip.sb 
curl whatismyip.akamai.com
curl ipecho.net/plain
curl icanhazip.com
```

**第二种方式**
```bash
curl -s  http://ip.taobao.com/service/getIpInfo2.php?ip=myip | awk -F"ip" '{print $2}' | awk -F'"' '{print $3}'

curl -s ifcfg.cn/echo | python -m json.tool
```