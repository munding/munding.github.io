---
title: "为什么 Proxy 认证要发两次请求"
date: 2022-11-18T10:53:13+08:00
draft: false
tags: ["http", "Proxy"]
categories: ["协议"]
---

## 现象

这段时间通过上服务器观察日志，发现部分语言的 HTTP 客户端在进行代理认证的时候会发送两次 HTTP 请求

1. 第一次请求不会携带任何认证信息
2. 第二次请求才会携带上 `Proxy-Authorization` 的 Header

涉及到的 HTTP 客户端还是很多的，例如：

- Java 的 httpclient、jsoup
- C# 的 HTTP 客户端
- Chrome 使用 SwitchyOmega 插件

## 原因

Go 的 net 包和 Python 的 Request 就没有这个问题，虽然在用户侧发送两次请求是无感知的，但是一个请求发送两次，相应耗时还是会增加的

由于对 Java 等不太熟悉，就不直接看源码了，google 找了下问题原因：

- [rfc2616：Proxy-Authenticate](https://www.rfc-editor.org/rfc/rfc2616#section-14.33)

- [HttpClient 4.2.2 and Proxy with username/password](https://stackoverflow.com/questions/13288038/httpclient-4-2-2-and-Proxy-with-username-password)

直接破案了，因为 HTTP 协议中的 `Proxy-Authenticate` Header，当请求中没有 `Proxy-Authorization` 时，它需要伴随着 `407 (Proxy Authentication Required)` 一并返回给客户端，告诉客户端使用那种认证方式

最常见的就是 Basic 认证（用户名: 密码计算 base64）：

```go
func BasicAuth(username, password string) string {
	auth := username + ":" + password
	return base64.StdEncoding.EncodeToString([]byte(auth))
}
```

除了 Basic，还有如下认证方案 ：

[HTTP/Authentication](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Authentication)

- **Basic** (查看 [RFC 7617](https://datatracker.ietf.org/doc/html/rfc7617)，base64 编码凭证。详情请参阅下文.),
- **Bearer** (查看 [RFC 6750](https://datatracker.ietf.org/doc/html/rfc6750)，bearer 令牌通过 OAuth 2.0 保护资源),
- **Digest** (查看 [RFC 7616](https://datatracker.ietf.org/doc/html/rfc7616)，只有 md5 散列 在 Firefox 中支持，查看 [bug 472823](https://bugzilla.mozilla.org/show_bug.cgi?id=472823) 用于 SHA 加密支持),
- **HOBA** (查看 [RFC 7486](https://datatracker.ietf.org/doc/html/rfc7486)（草案），**H**TTP **O**rigin-**B**ound 认证，基于数字签名),
- **Mutual** (查看 [draft-ietf-httpauth-mutual](https://tools.ietf.org/html/draft-ietf-httpauth-mutual-11)),
- **AWS4-HMAC-SHA256** (查看 [AWS docs](https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-auth-using-authorization-header.html))

后来进行测试，当 Proxy 返回 407 不带 `Proxy-Authenticate: Basic`，这种发送两次的 HTTP 客户端就不会在发送第二次请求了（当然用户收到的请求状态码也是 407）...

回到最初的问题，发送两次的原因就是

1. 第一次要拿到 `Proxy-Authenticate`，Proxy 是通过什么方式认证的，如 Basic
2. 通过返回的认证方式使用对应的算法对认证信息进行编码

## 优化

发送两次的 HTTP 客户端更多的考虑到通用性，用户可以在不知道 Proxy 使用什么认证方式的情况下正常使用

### 客户端

不过在实际项目使用中，考虑到性能因素，在知道认证方式的情况下：

1. 把 `Proxy-Authorization` 写死
2. 直接白名单 IP 认证

### HTTP 客户端优化

1. 能自主选择认证方式，像 Go、Python 这类的认证方式默认就是 Basic
2. 对于地址相同的 Proxy，第一次拿到认证方式后进行缓存，后续就不必在每次请求发送两次
