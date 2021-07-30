---
title: "MacOS使用Tips总结
date: 2020-04-06
tags: ["",""]
categories: ["macOS",""]
description: ""
summary: ""
draft: false
---

> 北京时间 2019年10 月 8 日凌晨，苹果推送了 [macOS Catalina 正式版][introduce]更新。新版 macOS 将 iTunes 拆分为三个独立应用、推出了支持 iPad 作为第二屏幕的 「随航」，macOS 版「屏幕使用时间」等多项新功能。

[introduce]: https://zhuanlan.zhihu.com/p/85456461

### 忽略macOS Catalina 10.15更新通知

隐藏/关闭更新通知依次打开 启动台-其他-终端，输入以下神秘代码，Enter后输入用户密码即可。

```bash
sudo softwareupdate --ignore "macOS Catalina"
```

将来要是想开了，又想更新了，再次输入以下神秘代码即可。

```bash
sudo softwareupdate --reset-ignored
```

Tip：此方法在macOS 10.16 Big Sur已经不适用，暂时没有找到合适的方法

### 屏蔽升级Catalina软件更新小红点

如果在此之前已经点击了软件更新出现了小红点，对于强迫症用户十分不友好。

打开 启动台-其他-终端-输入

```bash
defaults write com.apple.systempreferences AttentionPrefBundleIDs 0
```

然后继续输入

```bash
killall Dock
```

烦人的小红点就消失了！！！

### 外接显示器Dock栏会乱跑

鼠标在上方屏幕红框处，也就是鼠标移不下去的地方放置一两秒，Dock栏就会跑到上方屏幕

同理，鼠标放下下方屏幕，鼠标移不下去的地方放置，Dock栏就会跑到下方屏幕

个人感觉这个设计很烂，而且也没有开关能永久固定Dock栏，希望Appple设计师能改改！！！

![image-20210730112937596](https://img.aladdinding.cn/image-20210730112937596.png)

