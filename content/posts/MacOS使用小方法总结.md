---
title: "MacOS 使用小方法总结"
date: 2021-07-30T10:51:21+08:00
draft: false
tags: ["macOS"]
categories: ["操作系统"]
---

# 忽略 macOS Catalina 10.15 更新通知

隐藏 / 关闭更新通知依次打开 启动台 - 其他 - 终端，输入以下神秘代码，Enter 后输入用户密码即可。

```bash
sudo softwareupdate --ignore "macOS Catalina"
```

将来要是想开了，又想更新了，再次输入以下神秘代码即可。

```bash
sudo softwareupdate --reset-ignored
```

Tip：此方法在 macOS 10.16 Big Sur 已经不适用，暂时没有找到合适的方法

# 屏蔽升级 Catalina 软件更新小红点

如果在此之前已经点击了软件更新出现了小红点，对于强迫症用户十分不友好。

打开 启动台 - 其他 - 终端 - 输入

```bash
defaults write com.apple.systempreferences AttentionPrefBundleIDs 0
```

然后继续输入

```bash
killall Dock
```

烦人的小红点就消失了！！！

# 外接显示器 Dock 栏会乱跑

鼠标在上方屏幕红框处，也就是鼠标移不下去的地方放置一两秒，Dock 栏就会跑到上方屏幕

同理，鼠标放下下方屏幕，鼠标移不下去的地方放置，Dock 栏就会跑到下方屏幕

个人感觉这个设计很烂，而且也没有开关能永久固定 Dock 栏，希望 Appple 设计师能改改！！！

![image-20210730112937596](https://img.aladdinding.cn/image-20210730112937596.png)

也可以在设置 -> 调度中心取消勾选「显示器具有单独的空间」

![](https://img.aladdinding.cn/202203291420777.png)

# 按住 Command 连续选中文件

Command 键是 macOS 上最基础的文件选择辅助按键之一。按住 Command 后再选文件，只要保持 Command 键一直按着，已经选中的文件就会一直保持被选中的状态。
相关的快捷键有：Command+A 全选所有文件

# 按住 Command 连续选中文件

Command 键是 macOS 上最基础的文件选择辅助按键之一。按住 Command 后再选文件，只要保持 Command 键一直按着，已经选中的文件就会一直保持被选中的状态。
相关的快捷键有：Command+A 全选所有文件

# 按住 Command + 鼠标移动，可以快速移动和删除图标

这个组合键，适用于 Mac 菜单栏上的图标和一些包括 Finder 在内的系统自带软件的菜单栏。

# 通过 Command + 点击在新窗口中打开 Finder 侧边栏上的项目

点住 Command 键后，再点击 Finder 侧边栏上的任意项目，这样就可以在新 Finder 窗口中打开。这个操作可以应用在任何侧边栏项目，包括收藏、共享和设备。当我们想要在两个窗口之间复制或移动文件时，这个功能非常实用。

# Command + 鼠标拖拽可以移动后方的窗口，同时不影响前端窗口

想要查看背景中窗口，但是不想失去对最前方窗口的控制？通过 Command + 拖拽即可实现。

# Command + 回车 Spotlight 中的搜索结果可以直接在 Finder 中查看

与 Dock 操作一样，当使用 Spotlight 搜索时，只需通过 Command + 回车即可打开搜索结果中文件的位置，直接回车会打开文件。

# 使用 Command 键选择不相邻的文件

![image-20210802115401303](https://img.aladdinding.cn/20210802115401.png)

# Command 的其他组合键

- Command + W：关闭最前面的窗口
- Command + M：将最前面的窗口最小化至 “程序坞”

- Command + F：查找
- Command + X：剪切所选项并拷贝到剪贴板

- Command + C：将所选项拷贝到剪贴板
- Command + V：将剪贴板的内容粘贴到当前文稿或应用中

- Command + Z：撤销上一个命令
- Command + A：全选各项

- Command + tab：切换应用
- Command + shift + 3: 全屏截图

- Command + shift + 4: 部分截图