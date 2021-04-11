---
title: "搭建免费科学上网服务器v2ray"
author: "Mayer Shi"
tags: ["tools"]
categories: ["Ops"]
date: 2020-08-06 20:25:23
draft: false
---
利用gcp每个绑定信用卡会送300美元可以免费搭建一个v2ray的梯子，而且这个羊毛可以重复薅无数次。当该账号用完以后，重新注册一个新的谷歌账号，绑定原来的信用卡继续赠送300美元。
<!--more-->



## 安装步骤

利用gcp每个绑定信用卡会送300美元可以免费搭建一个梯子，而且这个羊毛可以重复薅无数次。当该账号用完以后，重新注册一个新的谷歌账号，绑定原来的信用卡继续赠送300美元。

Google Cloud Platform： https://cloud.google.com/gcp
创建compute engine—虚拟机实例—创建实例—服务器选址（ 除非有特殊需求，区域一般选亚太-，香港、台湾速度比较快，但不是太稳定，我一般选日本。）
然后开始搭建梯子，软件比较多，SS，SSR，Trojan；但是现在主流还是V2Ray，用233boy大的一键脚本安装会比较简单，只有2行命令

```bash
sudo -i bash <(curl -s -L https://git.io/v2ray.sh)
```

然后如果想选择WebSocket + TLS传输协议，还需要有一个域名。
