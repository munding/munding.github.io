---
title: "HTTPS 协议相关学习记录"
date: 2021-11-23
tags: ["https",""]
categories: ["",""]
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

# TLS 两个阶段

1. 握手（Handshake）阶段，其目的是通信双方约定在数据传输阶段使用的加密密钥（非对称加密算法）
2. 数据传输（Application_Data）阶段，使用握手阶段协商出的加密密码，加密数据传输（为效率考虑，使用对称加密算法）。

下面就详细介绍这两个切换，其中最重要的就是握手阶段

## HANDSHAKE 阶段

### 客户端：

#### Client Hello

主要作用是告诉目标服务器，客户端所支持的 TLS 协议版本，以及所支持的加密算法等等。

![image-20211123173729470](https://img.aladdinding.cn/20211123173731.png)

**客户端随机数（Random）**：客户端生成的随机数，暂时称作 ClientHello random，用于后续的密钥协商。

**会话 ID（Session ID）**：当 Client 通过一次完整的握手，与 Server 建立了一次完整的 Session，Server 会记录这次 Session 的信息，以备恢复会话的时候使用。上图中该字段为空，说明这是第一次连接到服务器。

[加密算法套件（Cipher Suite）](https://blog.helong.info/blog/2015/01/23/ssl_tls_ciphersuite_intro/)：以客户端所倾向的顺序（最推荐的在最先）包含了客户端所支持的密码算法套件。TLS 中使用的密码套件有一种标准格式。上面的报文中，客户端发送了 46 套加密套件，服务端会从中选出一种用于本次加密连接使用。一个加密算法套件是 4 个算法的组合。

**压缩方法（Compression Methods）**：加密之前的压缩算法。这个字段在 TLS 1.2 中用的不多。在 TLS 1.3 中这个字段被删除。

[扩展（Extension）](https://halfrost.com/https-extensions/)：结构定义是一个枚举类型，用于携带一些扩展参数，加强 TLS 功能。（携带域名，是否支持 http/2.0 等等）

从 Server Hello 到 Server Hello Done，有些服务端的实现是每条单独发送，有服务端实现是合并到一起发送。Sever Hello 和 Server Hello Done 都是只有头没有内容的数据。

### 服务端

收到 Client Hello 之后服务器必须发送 Server Hello 信息，服务器会检查指定诸如 TLS 版本和算法的 Client Hello 的条件，如果服务器接受并支持所有条件，它将发送其证书以及其他详细信息，否则，服务器将发送握手失败消息。

#### Server Hello

![image-20211123175420329](https://img.aladdinding.cn/20211123175422.png)

**服务端随机数（Random）**：服务端生成的随机数，暂时称作 ServerHello random。注意，至此客户端和服务端都拥有了两个随机数（ClientHello random+ ServerHello random），这两个随机数会在后续生成对称加密密钥等情况用到。

**会话 ID（Session ID）**：服务器将约定的 Session 参数存储在 TLS 缓存中，并生成与其对应的 Session id。它与 Server Hello 一起发送到客户端。客户端可以写入约定的参数到此 Session id，并给定到期时间。客户端将在 Client Hello 中包含此 id。如果客户端在此到期时间之前再次连接到服务器，则服务器可以检查与 Session id 对应的缓存参数，并重用它们而无需完全握手。这非常有用，因为服务器和客户端都可以节省大量的计算成本。

**[加密算法套件（Cipher Suite）](https://blog.helong.info/blog/2015/01/23/ssl_tls_ciphersuite_intro/)**：抓包中服务端选择了 TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)，其中密钥协商算法是 ECDHE、身份认证算法是 RSA、加密模式是 AES_128_GCM、消息认证码算法（MAC）是 SHA256。

**压缩方法（Compression Methods）**：如果支持，服务器将同意客户端的首选压缩方法。

**[扩展（Extension）](https://halfrost.com/https-extensions/)**：同上，只有由 Client 给出的扩展才能出现在 Server 的列表中。例子中访问的百度首页，发现服务端的 application_layer_protocol_negotiation 仅支持 http/1.1。

#### Certificate

服务器将数字证书和到根 CA 整个链发给客户端，使客户端能用服务器证书中的服务器**公钥**认证。

![image-20211123175544807](https://img.aladdinding.cn/20211123175545.png)

#### Server Key Exchange

ServerKeyExchange 这个消息的目的就是传递了必要的密码信息，使客户端生成**预备主密钥**。

抓包中密钥协商算法是 ECDHE，所以需要 Server Key Exchange 消息传递动态/静态 DH 信息（DH 参数和 DH 公钥）。

如果密钥协商算法为 RSA ，客户端不需要额外参数就可以计算出**预备主密钥**，然后使用服务端 Certificate 中的公钥加密发送给服务端，所以不需要 Server Key Exchange 可以完成协商。

密钥交换算法这里就不继续深究，附上一片文章详细讲解此算法：[RSA 和 ECDHE 算法](https://cloud.tencent.com/developer/article/1780038)

![image-20211123175624007](https://img.aladdinding.cn/20211123175625.png)

#### Certificate Request（可选）

这一步是可选的，如果有则表示双向认证。如果在对安全性要求高的常见可能用到。服务器用来验证客户端。服务器端发出 Certificate Request 消息，要求客户端发他自己的证书过来进行验证。该消息中包含服务器端支持的证书类型（RSA、DSA、ECDSA 等）和服务器端所信任的所有证书发行机构的 CA 列表，客户端会用这些信息来筛选证书。常见的就是银行给你发的 USB 盾牌。

#### Server Hello Done

此消息完成握手协商的服务器部分。它不携带任何附加信息

![image-20211123175638530](https://img.aladdinding.cn/20211123175639.png)

### 客户端

#### Certificate（可选）

如果在第二阶段服务器端要求发送客户端证书，客户端便会在该阶段将自己的证书发送过去。服务器端在之前发送的 Certificate Request 消息中包含了服务器端所支持的证书类型和 CA 列表，因此客户端会在自己的证书中选择满足这两个条件的第一个证书发送过去。若客户端没有证书，则发送一个 no_certificate 警告。

#### Client Key Exchange

作用：交换或者协商出**预备主密钥**，用于**主密钥**的计算。

对于 RSA 握手协商算法来说，客户端会生成的一个 48 字节的**预备主密钥**，其中前 2 个字节是 ProtocolVersion，后 46 字节是随机数，用服务端的公钥加密之后通过 Client Key Exchange 子消息发给 服务端，服务端用私钥来解密。

对于 (EC)DHE 来说，**预备主密钥**是双方通过椭圆曲线算法生成的，双方各自生成临时公私钥对，保留私钥，将公钥发给对方，然后就可以用自己的私钥以及对方的公钥通过椭圆曲线算法来生成**预备主密钥**，预备主密钥长度取决于 DH/ECDH 算法公钥。**预备主密钥长度是 48 字节或者 X 字节**。此时传递的是 DH/ECDH 算法公钥（图中的 Pubkey），**预备主密钥**是双端计算出来的。

当服务端得到**预备主密钥**后，结合之前的 ClientHello random 和 ServerHello random，通过伪随机函数 PRF 生成并截取 48 字节为**主密钥**，用于后续对称加密传输数据的密钥，客户端同理。

![image-20211124152914343](https://img.aladdinding.cn/20211124152917.png)

### 客户端

#### Change Cipher Spec

作用：表明接下来传输数据过程中可以对应用数据协议进行加密了。（注：此时报文头中类型为 CHANGE_CIPHER_SPEC（20），不属于握手 Handshake（22）类型报文）

密码切换协议，表示随后的信息都将用双方商定的加密方法和密钥发送（Change Cipher Spec 是一个独立的协议，体现在数据包中就是一个字节的数据，用于告知服务端，客户端已经切换到之前协商好的加密套件（Cipher Suite）的状态，准备使用之前协商好的加密套件加密数据并传输了。其中第一条加密的消息就是 Encrypted Handshake Message

![image-20211124153940334](https://img.aladdinding.cn/20211124153942.png)

#### Encrypted Handshake Message（Finshed）

生成对称加密密钥之后，发送一条加密的数据，让服务端解密验证，确认密钥的正确性。

![image-20211124154000101](https://img.aladdinding.cn/20211124154002.png)

### 服务端

#### Change Cipher Spec

密码切换协议，服务端和客户端一样，告诉客户端可以开始加密通信。

![image-20211124161056299](https://img.aladdinding.cn/20211124161057.png)

#### Encrypted Handshake Message（Finshed）

生成对称加密密钥之后，发送一条加密的数据，让客户端解密验证；如果对方可以解密，则双方认证无误开始通信。

![image-20211124161120679](https://img.aladdinding.cn/20211124161121.png)

## APPLICATION_DATA 阶段

这个阶段就很简单了，数据开始加密传输，Record Type Values 为 Application Data（23）

![image-20211124182026481](https://img.aladdinding.cn/20211124182033.png)

# 总结

1. 交换 Hello 信息中，交换随机数，协商出后续使用的加密套件和对应的算法
2. 单向/双向发送证书允许客户端和服务端进行身份认证
3. Server/Client key Exchange 根据选择的算法交换相应参数，协商出**预备主密钥**
4. 双端通过**预备主密钥**、随机数计算出**主密钥**，用于后续的对称加密数据传输

# 参考

1. [Traffic analysis of a TLS session](https://megamorf.gitlab.io/2020/03/03/traffic-analysis-of-a-tls-session/#handshake-protocol-format)
2. [The Transport Layer Security (TLS) Protocol Version 1.2](https://datatracker.ietf.org/doc/html/rfc5246)
3. [HTTPS 温故知新系列——冰霜大佬](https://halfrost.com/tag/https/)
