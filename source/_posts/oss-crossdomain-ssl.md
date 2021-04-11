---
title: "基于阿里云OSS解决前端图片跨域的问题"
author: "Mayer Shi"
tags: ["前端跨域"]
categories: ["frontend"]
date: 2020-08-09 11:25:23
draft: false
---

开发前端应用通常将图片等一些资源存放在一些对象存储中比如，阿里云OSS、腾讯OSS等，在oss的基础加上CDN进行资源分发。这种场景下前端开发一般都会遇到跨域问题，该篇博客记录下解决过程。
<!--more-->


1. 登录阿里云oss账号，创建bucket。

2. 对bucket进行跨域设置，设置规则如下：


![ouJYe.p](https://wx2.sbimg.cn/2020/08/09/ouJYe.png)

[![ouLFD.png](https://wx1.sbimg.cn/2020/08/09/ouLFD.png)]

3. 针对OSS设置CDN，然后在dns中添加CNAME记录。

3. 设置完毕以后，我们在开发中总是遇到报错。

```bash
Mixed Content: The page at 'https://www.mayershi.me/#/front/user/center' was loaded over HTTPS, but requested an insecure XMLHttpRequest endpoint 'http://as.test.com/file/20200802115932-deqw.png'. This request has been blocked; the content must be served over HTTPS.
```

同时浏览器还报“与此网站构建的连接不完全安全。”

4. 解决方案就是，将CDN所有的http请求强制转化成https.

