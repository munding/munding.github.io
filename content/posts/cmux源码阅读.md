---
title: "Cmux 源码阅读"
date: 2022-11-14T11:42:24+08:00
draft: true
tags: ["cmux"]
categories: ["源码阅读"]
---

## cmux

[soheilhy/cmux](https://github.com/soheilhy/cmux) 是一个通用的 Go 库，可以根据有效负载实现多路连接。使用 cmux，您可以在同一个 TCP 侦听器上为 gRPC、 SSH、 HTTPS、 HTTP、 Go RPC 和几乎任何其他协议提供服务，以下是简单使用：

```go
// Create the main listener.
l, err := net.Listen("tcp", ":23456")
if err != nil {
	log.Fatal(err)
}

// Create a cmux.
m := cmux.New(l)

// Match connections in order:
// First grpc, then HTTP, and otherwise Go RPC/TCP.
grpcL := m.Match(cmux.HTTP2HeaderField("content-type", "application/grpc"))
httpL := m.Match(cmux.HTTP1Fast())
trpcL := m.Match(cmux.Any()) // Any means anything that is not yet matched.

// Create your protocol servers.
grpcS := grpc.NewServer()
grpchello.RegisterGreeterServer(grpcS, &server{})

httpS := &http.Server{
	Handler: &helloHTTP1Handler{},
}

trpcS := rpc.NewServer()
trpcS.Register(&ExampleRPCRcvr{})

// Use the muxed listeners for your servers.
go grpcS.Serve(grpcL)
go httpS.Serve(httpL)
go trpcS.Accept(trpcL)

// Start serving!
m.Serve()
```

## 源码阅读

cmux 总体实现可以总结为以下步骤：

1. 主线程中启动 cmux 监听 tcp listener 传来的 payload
2. 通过之前传递的各种协议的 match 规则，对 payload 进行 “嗅探”
3. 如果匹配则通过一个 net.Conn 的 chan 传递给对应协议的 listener
4. 由于 MuxConn 实现了 net.Conn 的全部接口，之前 “嗅探” 的数据在 buffer 中，不影响后续的读取

