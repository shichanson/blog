---
title: zoom 国际版视频会议软件屏蔽后解决办法
date: 2019-12-26 11:11:46
author: "Mayer Shi"
tags: ["zoom"]
categories: ["Tools"]
---

前段时间zoom视频会议软件由于视频数据没法留存在中国，导致zoom被墙掉了。但是可以通过一种方式解决。

<!--more-->

### 修改本地hosts文件

```bash
sudo vim /etc/hosts

221.122.88.132   zoom.us
52.202.62.203   zoom.us
52.202.62.241   zoom.us
52.202.62.241   share.zoom.us
52.202.62.203   share.zoom.us

104.16.51.111   support.zoom.us
104.16.52.111   support.zoom.us
104.16.53.111   support.zoom.us
104.16.54.111   support.zoom.us

221.122.88.132   www.zoom.us
221.122.88.132   www3.zoom.us
221.122.88.132   google.zoom.us
221.122.88.132   facebook.zoom.us
221.122.88.132   log.zoom.us
221.122.88.132   api.zoom.us
221.122.88.132   launcher.zoom.us
221.122.88.132   imauth.zoom.us
221.122.88.132   static.zoom.us
221.122.89.232   file-ia.zoom.us
221.122.89.180   xmpp001.zoom.us
221.122.89.180   xmpp002.zoom.us
221.122.89.180   xmpp003.zoom.us
221.122.89.180   xmpp004.zoom.us
221.122.89.180   xmpp005.zoom.us
221.122.89.180   xmpp006.zoom.us
221.122.89.180   xmpp007.zoom.us
221.122.89.180   xmpp008.zoom.us
221.122.89.180   xmpp009.zoom.us
221.122.89.180   xmpp010.zoom.us
221.122.89.180   xmpp011.zoom.us
221.122.89.180   xmpp012.zoom.us
221.122.89.180   xmpp013.zoom.us
221.122.89.180   xmpp014.zoom.us
221.122.89.180   xmpp015.zoom.us
221.122.89.231   xmppapi.zoom.us
```

### 验证是否可以访问

浏览器输入： https://zoom.us