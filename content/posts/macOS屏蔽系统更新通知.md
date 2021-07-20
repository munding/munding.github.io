---
title: "MacOS屏蔽系统更新通知"
date: 2020-04-06
tags: ["",""]
categories: ["macOS",""]
description: ""
summary: ""
draft: false
---

> 北京时间 2019年10 月 8 日凌晨，苹果推送了 [macOS Catalina 正式版][introduce]更新。新版 macOS 将 iTunes 拆分为三个独立应用、推出了支持 iPad 作为第二屏幕的 「随航」，macOS 版「屏幕使用时间」等多项新功能。

[introduce]: https://zhuanlan.zhihu.com/p/85456461



### 说在前面

作为一个Hackintosh用户，Hackintosh的稳定性依然是最重要的。macOS Mojave 10.14.6作为macOS 10.14的最后一个大版本，凭借其稳定性、硬件兼容性等依旧是目前Hackintosh主推的版本。

### 忽略macOS Catalina 10.15更新通知

隐藏/关闭更新通知依次打开 启动台-其他-终端，输入以下神秘代码，Enter后输入用户密码即可。

```bash
sudo softwareupdate --ignore "macOS Catalina"
```

将来要是想开了，又想更新了，再次输入以下神秘代码即可。

```bash
sudo softwareupdate --reset-ignored
```

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

### 关于Big Sur

此方法在macOS 10.16 Big Sur已经不适用，暂时没有找到合适的方法