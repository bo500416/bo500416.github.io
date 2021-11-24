---
title: "怪不得域名一直掉"
tags: ["域名","hugo"]
categories: ["经验"]
author: "MagicQ"
date: 2021-11-19T22:34:38+08:00
description: " "
---

昨天刚把hugo部署到github上，绑定了域名，弄了自动部署。

今天更新文章就发现一个问题。

只要actions一触发，网站就打不开了。

一直以为是不是`main.yml`写的不对，后来才发现，触发一次，`pages`里面的域名就不见了，空了。<!--more-->

百度了一下，才发现，是`theme`文件的`static`里面没有`cname`文件。

后来加上了`cname`文件才正常。
