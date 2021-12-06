---
title: "HTTPS 协议是如何握手的"
date: 2021-11-23
tags: ["https",""]
categories: ["协议",""]
description: ""
summary: "HTTPS (Hypertext Transfer Protocol Secure) 是基于 HTTP 的扩展，用于计算机网络的安全通信，已经在互联网得到广泛应用。在 HTTPS 中，原有的 HTTP 协议会得到 TLS （安全传输层协议） 或其前辈 SSL （安全套接层） 的加密。因此 HTTPS 也常指 HTTP over TLS 或 HTTP over SSL。"
draft: false
---

> Hypertext Transfer Protocol Secure (HTTPS) is an extension of the Hypertext Transfer Protocol (HTTP). It is used for secure communication over a computer network, and is widely used on the Internet. In HTTPS, the communication protocol is encrypted using Transport Layer Security (TLS) or, formerly, its predecessor, Secure Sockets Layer (SSL). The protocol is therefore also often referred to as HTTP over TLS, or HTTP over SSL.

HTTPS (Hypertext Transfer Protocol Secure) 是基于 HTTP 的扩展，用于计算机网络的安全通信，已经在互联网得到广泛应用。在 HTTPS 中，原有的 HTTP 协议会得到 TLS （安全传输层协议） 或其前辈 SSL （安全套接层） 的加密。因此 HTTPS 也常指 HTTP over TLS 或 HTTP over SSL。

# TLS 报文格式

TLS 协议各种通常分为两部分：

- 靠近应用层的握手协议 TLS Handshaking Protocols
- 靠近 TCP 的记录层协议 TLS Record Protocol

## 记录层协议（TLS Record Protocol）

记录层协议也可以理解为报文头（TLS Record header），占用 5 个字节

1. 第 1 个字节是类型（Record Type Values），目前有 4 种类型
2. 第 2-3 字节是版本（Version Values），目前有 4 种版本
3. 第 4-5 字节是长度（不包含 TLS 报文头本身长度）

```
           record type (1 byte)
          /
         /    version (1 byte major, 1 byte minor)
        /    /
       /    /         length (2 bytes)
      /    /         /
   +----+----+----+----+----+
   |    |    |    |    |    |
   |    |    |    |    |    | TLS Record header
   +----+----+----+----+----+

   Record Type Values       dec      hex
   -------------------------------------
   CHANGE_CIPHER_SPEC        20     0x14
   ALERT                     21     0x15
   HANDSHAKE                 22     0x16
   APPLICATION_DATA          23     0x17

   Version Values            dec     hex
   -------------------------------------
   SSL 3.0                   3,0  0x0300
   TLS 1.0                   3,1  0x0301
   TLS 1.1                   3,2  0x0302
   TLS 1.2                   3,3  0x0303
```

## 握手协议（TLS Handshaking Protocols）

TLS 握手协议还能细分为 5 个子协议：

- change_cipher_spec （密码切换协议，在 TLS 1.3 中这个协议已经删除，为了兼容 TLS 老版本，可能还会存在）
- alert（警告协议，用来表示关闭信息和错误）
- handshake（握手协议，TLS 协议簇中最最核心的协议）
- application_data（应用数据协议）
- heartbeat （心跳协议，这个是 TLS 1.3 新加的，TLS 1.3 之前的版本没有这个协议）

以记录类型为 handshake 举例：

其中 Handshake 协议中有 10 种握手消息类型（不计算扩展），格式如下：

```
                           |
         Record Layer      |  Handshake Layer
                           |                                  |
                           |                                  |  ...more messages
  +----+----+----+----+----+----+----+----+----+------ - - - -+--
  | 22 |    |    |    |    |    |    |    |    |              |
  |0x16|    |    |    |    |    |    |    |    |message       |
  +----+----+----+----+----+----+----+----+----+------ - - - -+--
    /               /      | \    \----\-----\                |
   /               /       |  \\
  type: 22        /        |   \         handshake message length
                 /              type
                /
           length: arbitrary (up to 16k)

   Handshake Type Values    dec      hex
   -------------------------------------
   HELLO_REQUEST              0     0x00
   CLIENT_HELLO               1     0x01
   SERVER_HELLO               2     0x02
   CERTIFICATE               11     0x0b
   SERVER_KEY_EXCHANGE       12     0x0c
   CERTIFICATE_REQUEST       13     0x0d
   SERVER_DONE               14     0x0e
   CERTIFICATE_VERIFY        15     0x0f
   CLIENT_KEY_EXCHANGE       16     0x10
   FINISHED                  20     0x14
```

# TLS/SSL 握手流程

1. 交换 Hello 信息中，交换随机数，协商出后续使用的加密套件和对应的算法
2. 单向/双向发送证书允许客户端和服务端进行身份认证
3. Server/Client key Exchange 根据选择的算法交换相应参数，协商出**预备主密钥**
4. 双端通过**预备主密钥**、随机数计算出**主密钥**，用于后续的对称加密数据传输

完整的握手流程：

```
     Client                                               Server

      ClientHello                  -------->
                                                      ServerHello
                                                     Certificate*
                                               ServerKeyExchange*
                                              CertificateRequest*
                                   <--------      ServerHelloDone
      Certificate*
      ClientKeyExchange
      CertificateVerify*
      [ChangeCipherSpec]
      Finished                     -------->
                                               [ChangeCipherSpec]
                                   <--------             Finished
      Application Data             <------->     Application Data
```

`*`表示着此阶段在某些条件下不存在，`[]`表示不属于握手报文

## Client Hello(Client -> Server)

主要作用是告诉 Server，Client 所支持的 TLS 协议版本、支持的加密算法等等。

![image-20211123173729470](https://img.aladdinding.cn/20211123173731.png)

**Client 随机数（Random）**：Client 生成的随机数，暂时称作 ClientHello random，用于后续的主密钥生成。

**会话 ID（Session ID）**：当 Client 通过一次完整的握手，与 Server 建立了一次完整的 Session，Server 会记录这次 Session 的信息，以备恢复会话的时候使用。上图中该字段为空，说明这是第一次连接到服务器。

**加密算法套件（Cipher Suite）**：包含了 Client 所支持的密码算法套件。TLS 中使用的密码套件有一种标准格式。上面的抓包中，Client 发送了 46 套加密套件，Server 会从中选出一种用于本次加密连接使用。一个加密算法套件是 4 个算法的组合。

**压缩方法（Compression Methods）**：加密之前的压缩算法。这个字段在 TLS 1.2 中用的不多。在 TLS 1.3 中这个字段被删除。

**扩展（Extension）**：结构定义是一个枚举类型，用于携带一些扩展参数，加强 TLS 功能。（携带域名，是否支持 http/2.0 等等）

## Server Hello(Server -> Client)

收到 Client Hello 之后服务器必须发送 Server Hello 信息，Server 会检查 TLS 版本和算法的 Client Hello 的条件，如果服务器接受并支持所有条件，它将发送其证书以及其他详细信息。否则，服务器将发送握手失败消息。

![image-20211123175420329](https://img.aladdinding.cn/20211123175422.png)

**Server 随机数（Random）**：Server 生成的随机数，暂时称作 ServerHello random。注意，至此 Client 和 Server 都拥有了两个随机数。

**会话 ID（Session ID）**：服务器将约定的 Session 参数存储在 TLS 缓存中，并生成与其对应的 Session id。它与 Server Hello 一起发送到 Client。Client 可以写入约定的参数到此 Session id，并给定到期时间。Client 将在 Client Hello 中包含此 id。如果 Client 在此到期时间之前再次连接到服务器，则服务器可以检查与 Session id 对应的缓存参数，并重用它们而无需完全握手。这非常有用，因为服务器和 Client 都可以节省大量的计算成本。

**加密算法套件（Cipher Suite）**：抓包中 Server 选择了 TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)，其中密钥协商算法是 ECDHE、身份认证算法是 RSA、加密模式是 AES_128_GCM、消息认证码算法（MAC）是 SHA256。

**压缩方法（Compression Methods）**：如果支持，服务器将同意 Client 的首选压缩方法。

**扩展（Extension）**：同上，只有由 Client 给出的扩展才能出现在 Server 的列表中。

## Certificate(Server -> Client)

Server 将数字证书和到根 CA 整个链发给 Client，使得 Client 能用证书中公钥进行认证。

![image-20211123175544807](https://img.aladdinding.cn/20211123175545.png)

## Server Key Exchange(Server -> Client)

主要作用是根据不同的密钥协商算法交换必要的密码信息。

如果密钥协商算法是 (EC)DHE，所以需要此阶段传递 DH 参数和 DH 公钥。

如果密钥协商算法为 RSA ，Client 不需要额外参数就可以计算出**预备主密钥**，然后使用 Server Certificate 中的公钥加密发送给 Server，所以不需要此阶段。

![image-20211123175624007](https://img.aladdinding.cn/20211123175625.png)

## Certificate Request(Server -> Client, 可选）

这一步是可选的，如果有则表示双向认证。如果在对安全性要求高的常见可能用到。服务器用来验证 Client。服务器端发出 Certificate Request 消息，要求 Client 发他自己的证书过来进行验证。例如银行给你发的 USB 盾牌。

## Server Hello Done(Server -> Client)

此消息完成握手协商的服务器部分，它不携带任何附加信息。

从 Server Hello 到 Server Hello Done，有些 Server 的实现是每条单独发送，有 Server 实现是合并到一起发送。Sever Hello 和 Server Hello Done 都是只有头没有内容的数据。

![image-20211123175638530](https://img.aladdinding.cn/20211123175639.png)

## Certificate(Client -> Server, 可选）

如果 Server 要求发送 Client 证书，Client 便会在该阶段将自己的证书发送过去。Server 在之前发送的 Certificate Request 消息中包含了服务器端所支持的证书类型和 CA 列表，因此 Client 会在自己的证书中选择满足这两个条件的第一个证书发送过去。若 Client 没有证书，则发送一个 no_certificate 警告。

## Client Key Exchange(Client -> Server)

主要作用是交换或者协商出**预备主密钥**，用于**主密钥**的计算。

对于 RSA 握手协商算法来说，Client 会生成的一个 48 字节的**预备主密钥**，其中前 2 个字节是 ProtocolVersion，后 46 字节是随机数，用 Server 的公钥加密之后通过此阶段发给 Server，Server 用私钥来解密。

对于 (EC)DHE 密钥协商算法来说，**预备主密钥**是双方通过椭圆曲线算法生成的，双方各自生成临时公私钥对，保留私钥，将公钥发给对方，然后就可以用自己的私钥以及对方的公钥通过椭圆曲线算法来生成**预备主密钥**。此时传递的是 (EC)DHE 算法公钥（图中的 Pubkey），**预备主密钥**是密钥在双方不直接传递密钥的情况下得到的。

当 Server 获得了可以计算**预备主密钥**的所有条件后，结合之前的 ClientHello random 和 ServerHello random，通过伪随机函数 PRF 生成并截取 48 字节为**主密钥**，用于后续对称加密传输数据的密钥，Client 同理。

![image-20211124152914343](https://img.aladdinding.cn/20211124152917.png)

## Change Cipher Spec(Client -> Server)

主要作用是表明接下来传输数据过程中可以对应用数据协议进行加密了。

密码切换协议，表示随后的信息都将用双方商定的加密方法和密钥发送（Change Cipher Spec 是一个独立的协议，体现在数据包中就是一个字节的数据，用于告知 Server，Client 已经切换到之前协商好的加密套件（Cipher Suite）的状态，准备使用之前协商好的加密套件加密数据并传输了。其中第一条加密的消息就是接下来的 Encrypted Handshake Message

![image-20211124153940334](https://img.aladdinding.cn/20211124153942.png)

## Encrypted Handshake Message(Client -> Server, 也称 Finshed)

生成对称加密密钥之后，发送一条加密的数据，让 Server 解密验证，确认密钥的正确性。

![image-20211124154000101](https://img.aladdinding.cn/20211124154002.png)

## Change Cipher Spec(Server -> Client)

密码切换协议，服务端和客户端一样，告诉客户端可以开始加密通信。

![image-20211124161056299](https://img.aladdinding.cn/20211124161057.png)

## Encrypted Handshake Message(Server -> Client, 也称 Finshed)

生成对称加密密钥之后，发送一条加密的数据，让客户端解密验证；如果对方可以解密，则双方认证无误开始通信。

![image-20211124161120679](https://img.aladdinding.cn/20211124161121.png)

### APPLICATION_DATA(CLient <-> Server)

这个阶段就很简单了，数据开始加密传输，其中`Record Type Values `为 `Application Data（23）`

![image-20211124182026481](https://img.aladdinding.cn/20211124182033.png)

# 参考

1. [Traffic analysis of a TLS session](https://megamorf.gitlab.io/2020/03/03/traffic-analysis-of-a-tls-session/#handshake-protocol-format)
2. [The Transport Layer Security (TLS) Protocol Version 1.2](https://datatracker.ietf.org/doc/html/rfc5246)
3. [HTTPS 温故知新系列——冰霜大佬](https://halfrost.com/tag/https/)
