---
title: "Hugo + Github Actions 实现自动化部署"
tags: ["hugo","自动"]
categories: ["转载"]
author: "MagicQ"
date: 2021-11-26T20:57:45+08:00
description: " "
---



### 上传 Hugo 源文件

把 Hugo 程序 push 到 GitHub 上，GitHub 上新建一个 repo，并只需保留以下文件上传到 master ，同时随手建个 gh-pages 分支：

### 添加 Github Actions 代码：

实现效果，直接网页上修改 data/links.toml 或任意文件，触发 Actions 自动化运行 Hugo 程序生成静态文件并推送到 gh-pages 分支上，等待几十秒可看到更新。

具体操作：

![](https://s3.bmp.ovh/imgs/2021/11/34e87f0d2036b19b.png)

![](https://s3.bmp.ovh/imgs/2021/11/b2ee262c3b9bd66f.png)

点 https://github.com/settings/tokens 新建一个，勾选 repo 和 workflow ,暂存；

![](https://s3.bmp.ovh/imgs/2021/11/b71d9d9afc40c41b.jpg)

进项目 settings/secrets 新建标题为 personal_token ，内容是刚创建的 tokens ;

回项目，点 Actions -- New wordflow -- Set up a workflow yourself ，添加如下代码：

```
name: Deploy Hugo # 任君喜欢

on:
  push:
    branches:
      - master   # master 更新触发

jobs:
  build-deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v1

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: latest

      - name: Build 
        run: hugo

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.personal_token }} # personal_token 这里新建一个 https://github.com/settings/tokens
          PUBLISH_BRANCH: gh-pages  # 推送到当前 gh-pages 分支
          PUBLISH_DIR: ./public  # hugo 生成到 public 作为跟目录
          commit_message: ${{ github.event.head_commit.message }}
```

等，3、2、1，看看 Actions 顺利不，再看看 gh-pages 静态文件更新了不，最终打开 Pages ，祝贺！
