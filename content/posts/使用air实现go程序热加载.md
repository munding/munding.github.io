---
title: "使用 air 实现 go 程序热加载"
date: 2022-04-21T16:21:53+08:00
draft: false
tags: ["Golang"]
categories: ["编程语言"]
---

在使用 Python Web 框架 Django 本地 `runserver` 启动后修改了代码，程序够自动重新加载并执行（live-reload），在开发调试阶段非常实用，可以提高开发效率。

在使用 Go 开发自己的项目或者使用 gin 框架进行本地调试的时候，也需要在文件修改后自动编译运行查看效果，那么则可以使用 [air](https://github.com/cosmtrek/air) 这个工具。

## AIR

air 使用 go 语言开发，可以实现 go 语言应用程序的热加载，它支持以下特性：

1. 彩色日志输出
2. 自定义构建或二进制命令
3. 支持忽略子目录
4. 启动后支持监听新目录
5. 更好的构建流程

{{< image src="https://img.aladdinding.cn/202204221512710.png" caption="air 使用">}}

## 安装

由于是 go 语言开发，对于我来说就直接下载二进制文件放到系统 `PATH` 目录下了，当然也可以使用 `go get` 、`Docker` 等方式安装，具体可以查看 [Readme](https://github.com/cosmtrek/air#readme) 中其他的安装方法，这里就不再赘述了

## 使用

写好 `.air.conf` 文件放在项目目录下然后直接执行 `air` 命令就行，非常简单

完整的示例以及注释如下，需要新增环境变量或者是命令行参数的可以在 `full_bin` 前后添加

```toml
# [Air](https://github.com/cosmtrek/air) TOML 格式的配置文件

# 工作目录
# 使用 . 或绝对路径，请注意 `tmp_dir` 目录必须在 `root` 目录下
root = "."
tmp_dir = "tmp"

[build]
# 只需要写你平常编译使用的 shell 命令。你也可以使用 `make`
cmd = "go build -o ./tmp/main ."
# 由 `cmd` 命令得到的二进制文件名
bin = "tmp/main"
# 自定义的二进制，可以在前方添加环境变量或者是后方添加命令行参数启动 eg：APP_ENV=dev
full_bin = "./tmp/main"
# 监听以下文件扩展名的文件。
include_ext = ["go", "tpl", "tmpl", "html"]
# 忽略这些文件扩展名或目录
exclude_dir = ["assets", "tmp", "vendor", "frontend/node_modules"]
# 监听以下指定目录的文件
include_dir = []
# 排除以下文件
exclude_file = []
# 如果文件更改过于频繁，则没有必要在每次更改时都触发构建。可以设置触发构建的延迟时间
delay = 1000 # ms
# 发生构建错误时，停止运行旧的二进制文件。
stop_on_error = true
# air 的日志文件名，该日志文件放置在你的 `tmp_dir` 中
log = "air_errors.log"

[log]
# 显示日志时间
time = true

[color]
# 自定义每个部分显示的颜色。如果找不到颜色，使用原始的应用程序日志。
main = "magenta"
watcher = "cyan"
build = "yellow"
runner = "green"

[misc]
# 退出时删除 tmp 目录
clean_on_exit = true
```

生成并运行的二进制文件会放在当前目录下 `tmp` 目录，程序结束后会自动删除，非常贴心
