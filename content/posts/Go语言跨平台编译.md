---
title: "Go语言跨平台编译"
date: 2021-10-29
tags: ["",""]
categories: ["Go",""]
description: ""
summary: "默认我们go build的可执行文件都是当前操作系统可执行的文件，如果我想在macOS下编译一个linux下可执行文件，那需要怎么做呢"
draft: false
---

### go build

使用：

```
go build [-o 输出名] [-i] [编译标记] [包名]
```

默认我们`go build`的可执行文件都是当前操作系统可执行的文件，如果我想在macOS下编译一个linux下可执行文件，那需要怎么做呢？

只需要指定目标操作系统的平台和处理器架构即可，例如macOS平台终端下按如下方式指定环境变量。

```
SET CGO_ENABLED=0  // 禁用CGO
SET GOOS=linux  // 目标平台是linux
SET GOARCH=amd64  // 目标处理器架构是amd64
```

### macOS

编译 Linux 和 Windows平台 64位 可执行程序:

```
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build
```

### Linux

编译 Mac 和 Windows 平台64位可执行程序：

```
CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build
```

### Windows

编译Mac平台64位可执行程序

```
SET CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build
```



