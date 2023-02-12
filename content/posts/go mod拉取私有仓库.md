---
title: "Go Mod 拉取私有仓库"
date: 2023-01-13T10:07:31+08:00
draft: false
tags: ["go"]
categories: ["编程语言"]
---

## 配置环境变量

`GOPRIVATE="github.com/xxx/xxx"`

可以精确到用户名或者是仓库名

尤其是配置了 `GOPROXY` 的，代理网站肯定是访问不到你的私有仓库的，所以需要配置 `GOPRIVATE`

## SSH 方式

如果是 SSH 方式拉取的代码，需要将 Git 配置为使用 SSH 代替 HTTPS ，添加下面几行到 `~/.gitconfig`

```
[url "ssh://git@github.com/"]
	insteadOf = https://github.com/
```

## HTTPS 方式

如果是 HTTPS 方式拉取的代码，需要告诉 Git 你的账号 access token，添加一行到 `$HOME/.netrc`

```
machine github.com login USERNAME password APIKEY
```

## 参考

https://go.dev/doc/faq#git_https
