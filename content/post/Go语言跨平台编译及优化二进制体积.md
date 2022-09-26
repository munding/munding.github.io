---
title: "go 语言跨平台编译及优化二进制体积"
date: 2021-10-29
tags: ["go",""]
description: "跨平台编译并减小编译后的二进制的体积，能够加快程序的发布和安装过程"
draft: false
---

# 编译

```
go build [-o 输出名] [-i] [编译标记] [包名]
```

默认我们 `go build` 的可执行文件都是当前操作系统可执行的文件，如果我想在 macOS 下编译一个 linux 下可执行文件，那需要怎么做呢？

只需要指定目标操作系统的平台和处理器架构即可，例如 Window 平台终端下按如下方式指定环境变量。

```
SET CGO_ENABLED=0  // 禁用 CGO
SET GOOS=linux  // 目标平台是 linux
SET GOARCH=amd64  // 目标处理器架构是 amd64
```

## macOS

编译 Linux 和 Windows 平台 64 位 可执行程序:

```
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build
```

## Linux

编译 Mac 和 Windows 平台 64 位可执行程序：

```
CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build
```

## Windows

编译 Mac 平台 64 位可执行程序

```
SET CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build
```

# 优化

## 新增args

Go 编译器默认编译出来的程序会带有符号表和调试信息，一般来说 release 版本可以去除调试信息以减小二进制体积

```go
go build -ldflags="-s -w" -o server main.go
```

- -s：忽略符号表和调试信息
- -w：忽略DWARFv3调试信息，使用该选项后将无法使用gdb进行调试

## 使用 upx 减小体积

[upx](https://github.com/upx/upx) 是一个常用的压缩动态库和可执行文件的工具，通常可减少 50-70% 的体积。

upx 的安装方式非常简单，我们可以直接从 [github](https://github.com/upx/upx/releases/) 下载最新的 release 版本，支持 Windows 和 Linux，在 Ubuntu 或 Mac 可以直接使用包管理工具安装。

upx 有很多参数，最重要的则是压缩率，`1-9`，`1` 代表最低压缩率，`9` 代表最高压缩率。

```
go build -ldflags="-s -w" -o server main.go && upx -9 server
```

upx 压缩后的程序和压缩前的程序一样，无需解压仍然能够正常地运行，这种压缩方法称之为带壳压缩，压缩包含两个部分：

- 在程序开头或其他合适的地方插入解压代码；
- 将程序的其他部分压缩。

执行时，也包含两个部分：

- 首先执行的是程序开头的插入的解压代码，将原来的程序在内存中解压出来；
- 再执行解压后的程序。

也就是说，upx 在程序执行时，会有额外的解压动作，不过这个耗时几乎可以忽略。

如果对编译后的体积没什么要求的情况下，可以不使用 upx 来压缩。一般在服务器端独立运行的后台服务，无需压缩体积。

# 参考

> [How to reduce compiled file size? StackOverflow](https://stackoverflow.com/questions/3861634/how-to-reduce-compiled-file-size)
