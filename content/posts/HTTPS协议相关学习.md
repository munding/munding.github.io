---
title: "HTTPS协议相关学习"
date: 2021-11-23
tags: ["",""]
categories: ["",""]
description: ""
summary: ""
draft: true
---

> Hypertext Transfer Protocol Secure (HTTPS) is an extension of the Hypertext Transfer Protocol (HTTP). It is used for secure communication over a computer network, and is widely used on the Internet. In HTTPS, the communication protocol is encrypted using Transport Layer Security (TLS) or, formerly, its predecessor, Secure Sockets Layer (SSL). The protocol is therefore also often referred to as HTTP over TLS, or HTTP over SSL.

HTTPS (Hypertext Transfer Protocol Secure) 是基于 HTTP 的扩展，用于计算机网络的安全通信，已经在互联网得到广泛应用。在 HTTPS 中，原有的 HTTP 协议会得到 TLS (安全传输层协议) 或其前辈 SSL (安全套接层) 的加密。因此 HTTPS 也常指 HTTP over TLS 或 HTTP over SSL。

## TLS两个阶段

1. 握手（Handshake）阶段，其目的是通信双方约定在数据传输阶段使用的加解密算法及密钥（非堆成加密算法）
2. 数据传输阶段，即发送到网络前加密数据，从网络收到数据后解密数据（为效率考虑，使用对称加密算法）。

## 客户端发出请求

#### Client Hello

![image-20211123173729470](https://img.aladdinding.cn/20211123173731.png)

## 服务器回应

从Server Hello到Server Hello Done，有些服务端的实现是每条单独发送，有服务端实现是合并到一起发送。Sever Hello和Server Done都是只有头没有内容的数据。

#### Server Hello

![image-20211123175420329](https://img.aladdinding.cn/20211123175422.png)

#### Certificate

![image-20211123175544807](https://img.aladdinding.cn/20211123175545.png)

#### Server Key Exchange

![image-20211123175624007](https://img.aladdinding.cn/20211123175625.png)

#### Server Hello Done

![image-20211123175638530](https://img.aladdinding.cn/20211123175639.png)

## 客户端回应

#### Client Key Exchange

#### Change Cipher Spec

#### Encrypted Handshake Message

基于协商生成的密钥，用AES加密验证信息让服务端进行认证；如果对方可以解密，则双方认证无误开始通信

## 服务器的最后回应

#### Change Cipher Spec

#### Encrypted Handshake Message

基于协商生成的密钥，用AES加密验证信息让客户端进行认证；如果对方可以解密，则双方认证无误开始通信
