---
title: "Tmux的QuickStart"
date: 2019-08-09
tags: ["tmux",""]
categories: ["",""]
description: ""
summary: ""
draft: false
---

### Tmux简介
> Tmux 的全称是 Terminal MUtipleXer，及终端复用软件。顾名思义，它的主要功能就是用于在一个终端窗口中运行多个终端会话并且在你关闭终端窗口之后保持进程的运行。

### Tmux安装
```bash
# Ubuntu 或 Debian
$ sudo apt-get install tmux

# CentOS 或 Fedora
$ sudo yum install tmux

# Mac
$ brew install tmux
```

### Tmux概念
Tmux 中有几个重要概念：
- 会话（session）: 建立一个 tmux 工作区会话，会话可以长期驻留，重新连接服务器不会丢失，我们只需重新 tmux attach 到之前的工作区就可以恢复会话
- 窗口（window）: 容纳多个窗格
- 窗格（pane）: 可以在窗口中分成多个窗格
![Tmux概念图](https://img.aladdinding.cn/tmux.png)


### Tmux基本操作
#### 常用命令
- tmux new　　创建默认名称的会话
- tmux new -s mysession　　创建名为mysession的会话
- tmux ls　　显示会话列表
- tmux a　　连接上一个会话
- tmux a -t mysession　　连接指定会话
- tmux rename -t s1 s2　　重命名会话s1为s2
- tmux kill-session　　关闭上次打开的会话
- tmux kill-session -t s1　　关闭会话s1
- tmux kill-session -a -t s1　　关闭除s1外的所有会话
- tmux kill-server　　关闭所有会话

**Tmux 默认的快捷键前缀是 ctrl+b，当然你也可以修改它（后文会提到）
以下所有的操作都是激活控制台之后，即键入Ctrl+b前提下才可以使用的命令**

#### 会话操作（session）
- ?　　列出所有快捷键；按q返回
- d　　脱离当前会话,可暂时返回Shell界面，输入tmux attach能够重新进入之前会话
- s　　选择并切换会话；在同时开启了多个会话时使用
- D　　选择要脱离的会话；在同时开启了多个会话时使用
- :　　进入命令行模式；此时可输入支持的命令，例如kill-server所有tmux会话
- [　　复制模式，光标移动到复制内容位置，空格键开始，方向键选择复制，回车确认，q/Esc退出
- ]　　进入粘贴模式，粘贴之前复制的内容，按q/Esc退出
- ~　　列出提示信息缓存；其中包含了之前tmux返回的各种提示信息
- t　　显示当前的时间
- Ctrl+z　　挂起当前会话

#### 窗口操作（window）
- c　　创建新窗口
- &　　关闭当前窗口
- 数字键　　切换到指定窗口
- p　　切换至上一窗口
- n　　切换至下一窗口
- l　　前后窗口间互相切换
- w　　通过窗口列表切换窗口
- ,　　重命名当前窗口，便于识别
- .　　修改当前窗口编号，相当于重新排序
- f　　在所有窗口中查找关键词，便于窗口多了切换

#### 面板操作（pane）
- “　　将当前面板上下分屏
- %　　将当前面板左右分屏
- x　　关闭当前分屏
- !　　将当前面板置于新窗口,即新建一个窗口,其中仅包含当前面板
- Ctrl+方向键　　以1个单元格为单位移动边缘以调整当前面板大小
- Alt+方向键　　以5个单元格为单位移动边缘以调整当前面板大小
- 空格键　　可以在默认面板布局中切换，试试就知道了
- q　　显示面板编号
- o　　选择当前窗口中下一个面板
- 方向键　　移动光标选择对应面板
- {　　向前置换当前面板
- }　　向后置换当前面板
- Alt+o　　逆时针旋转当前窗口的面板
- Ctrl+o　　顺时针旋转当前窗口的面板
- z　　tmux 1.8新特性，最大化当前所在面板

### Tmux便捷配置
#### 新增Tmux的配置文件
```bash
#新建Tmux配置文件
vi $HOME/.tmux.conf
```
**修改Tmux 快捷键前缀为 ctrl+s，便于操作**

```bash
#设置前缀
set -g prefix ^s
 
#解除Ctrl+b 与前缀的对应关系
unbind ^b
 
# split window
unbind '"'
# vertical split (prefix -)
bind - splitw -v
unbind %
bind | splitw -h # horizontal split (prefix |)
 
#将r 设置为加载配置文件，并显示"reloaded!"信息
bind r source-file ~/.tmux.conf \; display "Reloaded!"
 
#up
bind k select-pane -U
#down
bind j select-pane -D
#left
bind h select-pane -L
#right
bind l select-pane -R
 
#kill pane
bind q killp
 
setw -g mode-keys vi
```

#### 设置alias快捷键
```bash
alias ta='tmux a -t '
alias tf='tail -f'
alias tls='tmux ls'
alias tnew='tmux new -s '
```
### Oh my tmux

🇫🇷 Oh my tmux! My self-contained, pretty & versatile tmux configuration made with ❤️

https://github.com/gpakosz/.tmux

更好看、强悍的tmux配置，有时间可以研究

### 使用Tips

#### 跳转Tmux窗口号为两位数的窗口

通常使用`Prefix + 数字键`可以跳转到指定窗口，但是窗口号如果是`10`，当你按下`1`的时候就已经跳转到`1`号窗口了，可以先使用`Prefix + ,`，然后输入index