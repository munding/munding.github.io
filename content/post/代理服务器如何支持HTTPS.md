---
title: "代理服务器如何支持 HTTPS"
date: 2021-05-10
tags: ["https","proxy"]
categories: ["网络协议",""]
description: ""
summary: ""
draft: false
---

说到 HTTPS 协议，很多人想到的是配置各种复杂的证书。其实不然，可以说大多数代理能够支持 https 都是通过 http 协议中的 Web 隧道（也有叫做 HTTP 隧道）功能来实现的。

# Web 隧道

Web 隧道允许用户通过 HTTP 连接发送非 HTTP 流量（例如 FTP，Telnet，SMTP），这样就可以在 HTTP 上携带其他协议数据了。使用 Web 隧道最常见的原因就是要在 HTTP 链接中嵌入非 HTTP 流量。我们知道很多软件都是实现了自己的应用层协议，但是这些软件都支持设置代理，如 QQ，微信。

Web 隧道是用 HTTP 的 CONNECT 方法建立起来的。CONNECT 方法并不是 HTTP/1.1 核心规范的一部分，但却是一种得到广泛应用的扩展。CONNECT 方法请求隧道网关创建一条到达任意目的服务器和端口的 TCP 连接，并对客户端和服务器之间的后继数据进行盲转发。下面截取《HTTP 权威指南》配图，讲讲 CONNECT 方法如何建立一条 Web 隧道。

![](https://img.aladdinding.cn/http_connect.png)

1. 客户端首先发送了一条 CONNECT 请求给代理服务器。
2. 代理服务器收到了 CONNECT 请求，解析出报文中客户端希望访问的域名及端口号，然后向目标服务器进行 TCP 连接。（图中是到打开到主机 orders.joes-hardware.com 的标准 SSL 端口 443 的连接）
3. 代理服务器一旦和目标网站建立了 TCP 连接，就发送一条 HTTP 200 Connection Established 的响应来通知客户端 Web 隧道建立成功，可以发送数据了。
4. 此时客户端通过 Web 隧道发送的所有数据都会被代理服务器直接转发给目标网站。（如果是 HTTPS 协议则是各种 SSL 握手信息，加密后的 HTTP 报文）

客户端只有收到 `200 Connection Established` 才会继续发送数据。如果代理服务器和目标网站连接不成功怎么办呢？代理服务器可以自己灵活自定义：连接目标网站失败 `502 Bad Gateway`、代理认证未通过 `407 Proxy Authentication Required` 等等。

## CONNECT 请求

除了起始行之外，CONNECT 的语法与其他 HTTP 方法类似，只不过是主机名和端口号取代了 URI。其中主机和端口都必须指定，不然代理服务器就不清楚与谁建立连接了。

```
CONNECT home.netscape.com:443 HTTP/1.0
User-Agent: Mozilla/4.0
```

CONNECT 请求的 header 通常只会携带建立 Web 隧道所需要的信息，而不包含需要传输的请求信息。

常见的 CONNECT 请求 header：

- User-Agent：用户设备
- Proxy-Authorization：认证信息
- Proxy-Connection：是否支持长连接

![](https://img.aladdinding.cn/wk_connect.png)

## CONNECT 响应

发送了请求之后，客户端会等待来自网关的响应。和普通 HTTP 报文一样，响应码 200 表示成功。按照惯例，响应中的原因短语通常被设置为 “Connection Established”。

```
HTTP/1.0 200 Connection Established
Proxy-Agent: Netscape-Proxy/1.1
```

与普通 HTTP 响应不同，这个响应并不需要包含 Content-Type 首部。此时连接只是对原始字节进行转接，不再是报文的承载者，所以不需要使用内容类型了。

![](https://img.aladdinding.cn/wk_connect_res.png)

# 不止 HTTPS

正因为有了 Web 隧道，代理服务器不需要其他应用层协议进行额外的编码解析，只要 Web 隧道建立成功之后即可发送任何非 HTTP 流量。

## websocket

在 Python 的 websocket-client 框架中，如果使用 http 代理，首先会对代理发送 CONNECT 连接建立 Web 隧道，然后在传输 ws、wss 协议数据

```python
def _tunnel(sock, host, port, auth):
    debug("Connecting proxy...")
    connect_header = "CONNECT %s:%d HTTP/1.1\r\n" % (host, port)
    connect_header += "Host: %s:%d\r\n" % (host, port)

    # TODO: support digest auth.
    if auth and auth[0]:
        auth_str = auth[0]
        if auth[1]:
            auth_str += ":" + auth[1]
        encoded_str = base64encode(auth_str.encode()).strip().decode().replace('\n', '')
        connect_header += "Proxy-Authorization: Basic %s\r\n" % encoded_str
    connect_header += "\r\n"
    dump("request header", connect_header)

    send(sock, connect_header)

    try:
        status, resp_headers, status_message = read_headers(sock)
    except Exception as e:
        raise WebSocketProxyException(str(e))

    if status != 200:
        raise WebSocketProxyException(
            "failed CONNECT via proxy status: %r" % status)

    return sock
```

# 非 Web 隧道

当然，也有的代理服务器能够在不建立 Web 隧道的情况下，实现了对其他应用层协议的解析，从而实现代理转发的目的。

例如 HTTPS 协议，客户端首先和代理服务器进行代理服务器完成 SSL 握手，代理服务器获取到客户端发送的完整请求（明文），然后在和目标主机进行 SSL 握手，成功后转发用户的请求。最后还要确定使用的 HTTP 客户端是否支持连接 HTTPS 代理，因为绝大多数应用层协议客户端都是通过 Web 隧道使用代理。

在 Python 的 HTTP 客户端框架 [urllib3 的 1.26.0 版本](https://github.com/urllib3/urllib3/releases/tag/1.26.0) 中才添加了对 HTPPS 代理连接的支持。

- Added support for HTTPS proxies contacting HTTPS servers (Pull [#1923](https://github.com/urllib3/urllib3/pull/1923), Pull [#1806](https://github.com/urllib3/urllib3/pull/1806))

```python
proxies = {
'http': 'http://proxy_ip:port', # http 请求
'https': 'http://proxy_ip:port', # CONNECT 请求
'https': 'https://proxy_ip:port'  # 与 proxy 进行 ssl
}
```

