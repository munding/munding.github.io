---
title: "静态博客部署的最佳实践（适用Hugo、Hexo）"
date: 2021-11-04
tags: ["Hugo","Hexo"]
categories: ["部署",""]
description: ""
summary: ""
draft: false
---

有关静态博客部署的文章、视频教程已经很多了。

大致总结可以分为三类：

1. Github Pages + Github Action（网站托管，如：Vercel）
2. 本地编译 + rsync远程服务器
3. 本地编译 + 推送对象存储（七牛云）



之前博客一直部署在Github+Vercel，奈何美国服务器延迟太高

国内的阿里云、腾讯云的网站托管、云开发体验太差

刚好双十一购入三年的2核4G 8M带宽的轻量服务器（腾讯云YYDS）

趁着周末研究研究如何部署到云服务器上

总体体验不错，满足了我既要远程编译，Github托管代码、访问速度要快的需求

这套流程中你只需要写好文章（不需要Go环境），推送至Github即可

#### 最佳实践：

1. 本地更新完文章提交推送
2. 触发Github Action
3. Action的workflow中完成静态资源的编译、推送至云服务器
4. 云服务器Nginx访问静态资源



#### 部分配置文件:

使用`PEM`格式生成公钥私钥

```
ssh-keygen -m PEM -t rsa -b 4096
```

生成的公钥追加到authorized_keys中

```
cd .ssh/;cat id_rsa.pub >> authorized_keys
```

.github/workflows/main.yml
```yaml
# This is a basic workflow to help you get started with Actions

name: github pages

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ main ]
    paths-ignore:
      - '.gitignore'
      - 'README.md'
  pull_request:
    branches: [ main ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  deploy:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      # Runs a single command using the runners shell
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.85.0'
          extended: true
          
      - name: Build
        run: hugo --minify

      # Runs a set of commands using the runners shell
      - name: Github Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.ACCESS_TOKEN }}
          publish_dir: ./public

      # Deploy to Server
      - name: Server
        uses: easingthemes/ssh-deploy@main
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SERVER_SSH_KEY }}
          SOURCE: "public/"
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
          REMOTE_USER: ${{ secrets.REMOTE_USER }}
          TARGET: ${{ secrets.REMOTE_TARGET }}
```

Nginx 配置文件

- 开启了HTTP/2.0
- HTTP访问301跳转到HTTPS

```
server {
    listen         80;
    listen       [::]:80;
    server_name  <your.domain>;
    return         301 https://$host$request_uri;
}

server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;
    server_name  <your.domain>;

    ssl_certificate "<your.pem>";
    ssl_certificate_key "<your.key>";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    access_log  /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    location / {
        root   <your.dir>/public;
        index  index.html;
   }
        
}
```







