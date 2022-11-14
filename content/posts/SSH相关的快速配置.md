---
title: "SSH 相关的快速配置"
date: 2019-10-09T10:30:17+08:00
draft: false
tags: ["ssh"]
categories: ["终端工具"]
---

## SSH 简介
>SSH 为 Secure Shell 的缩写，由 IETF 的网络小组（Network Working Group）所制定；SSH 为建立在应用层基础上的安全协议。SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。SSH 最初是 UNIX 系统上的一个程序，后来又迅速扩展到其他操作平台。SSH 在正确使用时可弥补网络中的漏洞。SSH 客户端适用于多种平台。几乎所有 UNIX 平台—包括 HP-UX、Linux、AIX、Solaris、Digital UNIX、Irix，以及其他平台，都可运行 SSH。SSH 登录提供两种认证方式：口令 (密码) 认证方式和密钥认证方式。其中口令 (密码) 认证方式是我们最常用的一种，这里介绍密钥认证方式登录到 linux/unix 的方法。

## SSH 服务器之间免密登陆配置
### 生成密钥（公钥和私钥）
```bash
cd $HOME/.ssh
ssh-keygen -t rsa
# 全部回车默认
```
参数 -t rsa 表示使用 rsa 算法进行加密，执行后，会在 / home / 当前用户 /.ssh 目录下找到 id_rsa（私钥）和 id_rsa.pub（公钥）
### 放置公钥到目标服务器中
```bash
cat id_rsa.pub
```
复制 id_rsa.pub 内的公钥，登陆到目标服务器
```bash
cd $HOME/.ssh
vi authorized_keys
```
将复制的公钥粘贴到 authorized_keys 中，authorized_keys 存放远程免密登录的公钥，主要通过这个文件记录多台机器的公钥

------

以上的步骤就已经完成了 SSH 服务器之间的免密登陆。不过有的场景是一台跳板机和多台服务器完成了 SSH 免密传输，此时想换一台电脑管理这些服务器，可以将之前的跳板机私钥拷贝到新电脑中。

```bash
ssh-add ~/.ssh/id_rsa # id_rsa 为之前跳板机的私钥
```

如果出现提示 `Could not open a connection to your authentication agent.`, 运行如下命令

```bash
ssh-agent bash
```

## GitHub 的免密传输
### SSH 的免密传输
[connecting-to-github-with-ssh](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
1. 登陆到 Github 中，进入个人设置
2. 提交服务器 SSH 公钥
3. 选择 SSH and GPG keys，粘贴你的服务器公钥

#### 切换项目的传输方式
```bash
# （以 HTTPS 切换成 SSH 为例）
git remote remove origin
git remote add origin git@github.com:Username/Your_Repo_Name.git
# 重新设置 track branch
git branch --set-upstream-to=origin/master master
```
### ~~HTTPS 的免密传输~~

目前都是配置token了

#### 新建文件
```bash
vi $HOME/.git-credentials
```
#### 添加以下内容
```bash
# （GitHub 为 github.com，码云为 gitee.com）
https://{username}:{password}@github.com
```
#### 添加 git 配置
```bash
git config --global credential.helper store
```
#### 查看是否添加成功
```bash
# 查看 $HOME/.gitconfig 文件，会发现出现一下内容
[credential]
helper = store
```
这样设置存在一定风险，因为密码是明文存放在这个文件里的，比较容易泄露