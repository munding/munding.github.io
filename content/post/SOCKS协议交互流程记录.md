---
title: "SOCKS 协议交互流程记录"
date: 2020-07-19
tags: ["socks","网络协议"]
description: "一个二进制协议，不记录一下容易忘，看wiki太慢"
summary: ""
draft: false
---

>SOCKS 是一种网络传输协议，主要用于客户端与外网服务器之间通讯的中间传递。SOCKS 是 "Socket Secure" 的缩写。当防火墙后的客户端要访问外部的服务器时，就跟 SOCKS 代理服务器连接。这个代理服务器控制客户端访问外网的资格，允许的话，就将客户端的请求发往外部的服务器。这个协议最初由 David Koblas 开发，而后由 NEC 的 Ying-Da Lee 将其扩展到 SOCKS4。最新协议是 SOCKS5，与前一版本相比，增加支持 [UDP（用户数据报协议）](https://zh.wikipedia.org/wiki)、验证，以及 [IPv6](https://zh.wikipedia.org/wiki/IPv6)。根据 [OSI 模型](https://zh.wikipedia.org/wiki/OSI)，SOCKS 是 [会话层](https://zh.wikipedia.org/wiki) 的协议，位于 [表示层](https://zh.wikipedia.org/wiki) 与 [传输层](https://zh.wikipedia.org/wiki) 之间。SOCKS 协议不提供 [加密](https://zh.wikipedia.org/wiki)。

# SOCKS5

## SOCKS 协议版本认证

创建与 SOCKS5 服务器的 TCP 连接后客户端需要先发送请求来确认协议版本及认证方式（以字节为单位）

### client send

```
# +----+----------+----------+
# |VER | NMETHODS | METHODS  |
# +----+----------+----------+
# | 1  |    1     | 1 to 255 |
# +----+----------+----------+
```

- VER：SOCKS5 协议版本 0x05
- NMETHODS：METHODS 所占字节长度
- METHODS：客户端支持的认证方式列表，每个方法占 1 字节
  - 0x00 不需要认证
  - 0x01 GSSAPI
  - 0x02 用户名、密码认证
  - 0x03 to 0x7F 由 IANA 分配（保留）
  - 0x80 to 0xFE 私人方法保留
  - 0xFF 无可接受的方法

服务器从客户端提供的 `METHODS` 中选择一个并通过以下消息通知客户端

### server reply

```
# +----+--------+
# |VER | METHOD |
# +----+--------+
# | 1  |   1    |
# +----+--------+
```

- VER：协议版本 0x05

- METHOD：服务端选中的方法。如果返回 0xFF 表示没有一个认证方法被选中，客户端需要关闭连接。

## SOCKS 用户名密码认证

如果 SOCKS 协议版本认证中服务端返回的 `METHOD` 是 `0x02`，即需要用户名密码认证，则客户端会发送用户名密码认证信息。如果返回的 `METHOD` 是 `0x00`，即不需要认证，直接跳转下一步发送 SOCKS 请求信息。

### client send

```
# +----+------+----------+------+----------+
# |VER | ULEN |  UNAME   | PLEN |  PASSWD  |
# +----+------+----------+------+----------+
# | 1  |  1   | 1 to 255 |  1   | 1 to 255 |
# +----+------+----------+------+----------+
```

- VER：认证协议版本 0x01
- ULEN：用户名长度
- UNAME： 用户名
- PLEN：密码长度
- PASSWD：密码

### server reply

```
# +----+--------+
# |VER | STATUS |
# +----+--------+
# | 1  |   1    |
# +----+--------+
```

- VER：认证协议版本 0x01
- STATUS：认证状态
  - 0x00 成功
  - 0x01 失败

## 发送 SOCKS 请求信息

认证结束后客户端就可以发送请求信息。如果认证方法有特殊封装要求，请求必须按照方法所定义的方式进行封装。

### client send

```
# +----+-----+-------+------+----------+----------+
# |VER | CMD |  RSV  | ATYP | DST.ADDR | DST.PORT |
# +----+-----+-------+------+----------+----------+
# | 1  |  1  | X'00' |  1   | Variable |    2     |
# +----+-----+-------+------+----------+----------+
```

- VER：SOCKS5 协议版本 0x05
- CMD：SOCK 命令码
  - 0x01 CONNECT 请求
  - 0x02 BIND 请求
  - 0x03 UDP 转发
- RSV：0x00 保留
- ATYP：DST.ADDR 类型
  - 0x01 IPv4 地址，DST.ADDR 部分 4 字节长度
  - 0x03 域名，DST.ADDR 部分第一个字节为域名长度，DST.ADDR 剩余的内容为域名，没有 \ 0 结尾。
  - 0x04 IPv6 地址，16 个字节长度。
- DST.ADDR：目的地址
- DST.PORT：网络字节序表示的目的端口

### server reply

```
# +----+-----+-------+------+----------+----------+
# |VER | REP |  RSV  | ATYP | BND.ADDR | BND.PORT |
# +----+-----+-------+------+----------+----------+
# | 1  |  1  | X'00' |  1   | Variable |    2     |
# +----+-----+-------+------+----------+----------+
```

- VER：SOCKS5 协议版本 0x05
- REP：应答字段
  - 0x00 成功
  - 0x01 常规 SOCKS 服务器连接失败
  - 0x02 现有规则不允许连接
  - 0x03 网络不可达
  - 0x04 主机不可达
  - 0x05 连接被拒
  - 0x06 TTL 超时
  - 0x07 不支持的命令
  - 0x08 不支持的地址类型
  - 0x09 to 0xFF 未定义
- RSV：0x00 保留
- ATYP：BND.ADDR 类型
  - 0x01 IPv4 地址，DST.ADDR 部分 4 字节长度
  - 0x03 域名，DST.ADDR 部分第一个字节为域名长度，DST.ADDR 剩余的内容为域名，没有 \ 0 结尾。
  - 0x04 IPv6 地址，16 个字节长度。
- BND.ADDR：服务器绑定的地址
- BND.PORT：网络字节序表示的服务器绑定的端口

当服务端返回 `REP` 应答字段为 `0x00`，即成功时，客户端和服务端之间进行数据透传，完成 SOCKS5 代理。

# SOCKS4

SOCKS 4 只支持 TCP 转发

## 发送 SOCKS 请求信息

### client send

```
# +----+------+----------+--------+----------+----------+
# |VN  | CD   |  DSTPORT |  DSTIP |  USERID  |   NULL   |
# +----+------+----------+--------+----------+----------+
# | 1  |  1   |     2    |   4    | variable |    1     |
# +----+------+----------+--------+----------+----------+
```

- VN：SOCKS4 协议版本 0x04
- CD：SOCK 命令码
  - 0x01 CONNECT 请求
  - 0x02 BIND 请求
- DSTPORT：目的主机的端口
- DSTIP：目的主机的 IP 地址
- USERID：用户 USERID
- NULL：0x00

### server reply

```
# +-------+-------+----------+-----------+
# |  VN   |   CD  |  DSTPORT |   DSTIP   |
# +-------+-------+----------+-----------+
# |   1   |   1   |     2    |     4     |
# +-------+-------+----------+-----------+
```

- VN：回复代码的版本，应为 0x00（注意不是 0x04)
- CD：SOCK 命令码
  - 90(0x5a) 请求得到允许；
  - 91(0x5b) 请求被拒绝或失败；
  - 92(0x5c) 由于 SOCKS 服务器无法连接到客户端的 identd（一个验证身份的进程），请求被拒绝；
  - 93(0x5d) 由于客户端程序与 identd 报告的用户身份不同，连接被拒绝。
- DSTPORT：目的主机的端口（和请求包中相同）
- DSTIP：目的主机的 IP 地址（和请求包中相同）

当服务端返回 `CD` 字段为 `90(0x5a)`，即允许时，客户端和服务端之间进行数据透传，完成 SOCKS4 代理。

# SOCKS4a

SOCKS4a 协议是 SOCKS4 的一个补丁版，可以在 SOCKS4a 代理服务器上完成 DNS 解析

## 发送 SOCKS 请求信息

### client send

```
# +----+------+----------+--------+----------+----------+------------+----------+
# |VN  | CD   |  DSTPORT |  DSTIP |  USERID  |   NULL   |  HOSTNAME  |   NULL   |
# +----+------+----------+--------+----------+----------+------------+----------+
# | 1  |  1   |     2    |    4   | variable |    1     |  variable  |    1     |
# +----+------+----------+--------+----------+----------+------------+----------+
```

- DSTIP：0.0.0.x，其中 x 是非零，一般都为 1。（原文：such an address is inadmissible as a destination IP address and thus should never occur if the client can resolve the domain name）
- HOSTNAME：域名

其余字段和 SOCKS4 相同

### server reply

SOCKS4a 代理首先把 `HOSTNAME` 如：www.example.com 解析成对应的主机 IP 地址，并且和 IP 地址连接上，再向客户端发送和 SOCKS4 一样的响应。当服务端返回 `CD` 字段为 `90(0x5a)`，即允许时，客户端和服务端之间进行数据透传，完成 SOCKS4a 代理。

# SOCKS4，SOCKS4a 和 SOCKS5 的区别

比如浏览器使用 SOCKS4 代理访问 `www.baidu.com`，浏览器先用本地的 DNS 解析把 `www.baidu.com` 转换成对应的 IP 地址，然后向 SOCKS4 服务器发送报文。如果此时我们的电脑受限本地完成不了 DNS 解析，那怎么办呢？SOCKS4a 就是解决这样的问题的，客户端可以把域名发送到 SOCKS4a 服务器上完成 DNS 解析，发送的 `DSTIP` 则为 0.0.0.1 这样的假 IP，然后就是和 SOCKS4 一样进行数据转发。SOCKS5 代理和 SOCKS4 SOCKS4a 比，多了一个验证功能和 udp 代理的功能。

# SOCKS 协议 RFC

- [SOCKS4.protocol.txt](https://github.com/cfcs/ocaml-socks/blob/master/rfc/SOCKS4.protocol.txt) and [SOCKS4A.protocol.txt](https://github.com/cfcs/ocaml-socks/blob/master/rfc/SOCKS4A.protocol.txt) for `SOCKS 4` and the `SOCKS 4A` extension, respectively.
- [SOCKS5_rfc1928.txt](https://github.com/cfcs/ocaml-socks/blob/master/rfc/SOCKS5_rfc1928.txt) and [SOCKS5_rfc1929.txt](https://github.com/cfcs/ocaml-socks/blob/master/rfc/SOCKS5_rfc1929.txt) for `SOCKS 5`, and `SOCKS 5 Username/Password authentication`.
