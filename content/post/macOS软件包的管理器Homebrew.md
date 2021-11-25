---
title: "MacOS 软件包的管理器 Homebrew"
date: 2020-01-05
tags: ["",""]
categories: ["macOS",""]
description: ""
summary: ""
draft: false
---

[introduce]: https://brew.sh/index_zh-cn

# Homebrew 简介

> [Homebrew][introduce] 是一款 MacOS 平台下的软件包管理工具，拥有安装、卸载、更新、查看、搜索等很多实用的功能。简单的一条指令，就可以实现包管理，而不用你关心各种依赖和文件路径的情况，十分方便快捷。

# Homebrew 的几个核心概念

在正式介绍 Homebrew 的使用之前，我先为你介绍一下 Homebrew 中的一些核心的概念，了解这些概念，就可以帮助你更好的去使用 Homebrew。

| 词汇        | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| formula (e) | 安装包的描述文件，formulae 为复数                            |
| cellar      | 安装好后所在的目录                                           |
| keg         | 具体某个包所在的目录，keg 是 cellar 的子目录                 |
| bottle      | 预先编译好的包，不需要现场下载编译源码，速度会快很多；官方库中的包大多都是通过 bottle 方式安装 |
| tap         | 下载源，可以类比于 Linux 下的包管理器 repository             |
| cask        | 安装 macOS native 应用的扩展，你也可以理解为有图形化界面的应用。 |
| bundle      | 描述 Homebrew 依赖的扩展                                     |

# Homebrew 安装

将以下命令粘贴至终端，回车运行

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

如果出现 `curl: (7) Failed to connect to raw.githubusercontent.com port 443:xxx`，应该是被墙了，挂上梯子即可

```bash
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

# Homebrew 更换国内源

## 替换默认源

``` bash
# 步骤一：替换 brew.git
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

# 步骤二：替换 homebrew-core.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

#步骤三
brew update
```

## 替换 Homebrew Bottles 源

>Homebrew 是 OS X 系统的一款开源的包管理器。出于节省时间的考虑，Homebrew 默认从 Homebrew Bottles 源中下载二进制代码包安装。Homebrew Bottles 是 Homebrew 提供的二进制代码包，目前镜像站收录了以下仓库：

- homebrew/homebrew-core
- homebrew/homebrew-dupes
- homebrew/homebrew-games
- homebrew/homebrew-gui
- homebrew/homebrew-python
- homebrew/homebrew-php
- homebrew/homebrew-science
- homebrew/homebrew-versions
- homebrew/homebrew-x11

``````bash
# 根据自己使用的 shell
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
``````

```bash
# 使配置生效
source ~/.bash_profile
```

# 复原默认源

```bash
# 步骤一
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git

# 步骤二
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core

# 步骤三
brew update
```

# Homebrew 常用命令

## 安装卸载软件

- `brew --version` 或者 `brew -v` 显示 brew 版本信息
- `brew install` 安装指定软件
- `brew unistall`  卸载指定软件
- `brew list`  显示所有的已安装的软件
- `brew search text` 搜索本地远程仓库的软件，已安装会显示绿色的勾
- `brew search /text/` 使用正则表达式搜软件

##  升级软件相关

- `brew update` 自动升级 homebrew（从 github 下载最新版本）
- `brew outdated` 检测已经过时的软件
- `brew upgrade`  升级所有已过时的软件，即列出的以过时软件
- `brew upgrade` 升级指定的软件
- `brew pin` 禁止指定软件升级
- `brew unpin` 解锁禁止升级
- `brew upgrade --all` 升级所有的软件包，包括未清理干净的旧版本的包

## 清理相关

- `brew cleanup -n` 列出需要清理的内容
- `brew cleanup` 清理指定的软件过时包
- `brew cleanup` 清理所有的过时软件
- `brew unistall` 卸载指定软件
- `brew unistall  --force` 彻底卸载指定软件，包括旧版本

## 服务相关

- `brew services start mysql` 启动 Mysql

- `brew services stop mysql` 停止 Mysql

- `brew services restart mysql` 重启 Mysql

- `brew services list` 查看启动列表
