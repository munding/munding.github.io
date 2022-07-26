---
title: "Protocol Buffers协议使用小结"
date: 2022-07-26
tags: ["proto","网络协议"]
description: "protocol buffers 是 google 开源的一套成熟的结构数据序列化机制，在 grpc 中默认就使用的是 protocol buffers 协议"
draft: false
---



>[官方文档 Language Guide (proto3) ](https://developers.google.com/protocol-buffers/docs/proto3 )
>
>[protocol-buffers 代码生成指南](https://developers.google.com/protocol-buffers/docs/reference/overview)

网络上关于 Protocol Buffers 的翻译已经很多了，这里就不再重新写一遍了，主要记录一下常用语法。其实就是定义一个 `.proto` 文件，然后根据不同语言的插件生成不同的代码，通常代码会分为两个文件，以 go 语言为例子

- xxx.pb.go 主要包含消息定义的 go 语言代码
- xxx_grpc.pb.go 主要包含 grpc，也就是服务端，客户端之间交互的代码

至于为什么要分成两个文件（记得之前版本 go 语言只生成一个文件），应该是不同语言 grpc 的通信实现有很多吧，像 python 就有 gevent 和 asyncio 两种，拆分也是让文件分工更加清晰

至于 `.proto` 文件也是主要以：**定义消息**、**定义服务**、**其他规范：注释、选项等等** 组成

# 其他规范

- 文件开头通常是 `syntax = "proto3";`，这个必须写在开头，表示使用的是 `protov3` 的语法，不写的话默认为 `protov2`

- 其次是类似 `option go_package = "example.com/proto";` 这样的选项，用于不同语言生成代码的参数

- 注释与 c/c++ 语法相同，使用：`//`（单行注释）和 `/* ... */`（多行注释）

- Message 命名采用驼峰命名方式，字段命名采用小写字母加下划线分隔方式

  ```protobuf
    message SongServerRequest {
        required string song_name = 1;
    }
  ```

- Enums 类型名采用驼峰命名方式，字段命名采用大写字母加下划线分隔方式

  ```protobuf
    enum Foo {
        FIRST_VALUE = 1;
        SECOND_VALUE = 2;
    }
  ```

- Service 与 rpc 方法名统一采用驼峰式命名

# 定义消息 (Message)

Message 基本格式

```protobuf
message <message name> {
	<filed rule>(规则) <filed type>(类型) <filed name>(名称) = <filed number>(编号)
}
```

- message name：在同一个 `.proto` 文件内必须唯一
- filed rule：可以没有，常用的有 `repeated`、`oneof`
- filed type：数据类型，protobuf 定义的数据类型, 生产代码的会映射成对应语言的数据类型
- filed name：字段名称，同一个 message 内必须唯一
- filed number：字段的编号， 序列化成二进制数据时的字段编号

## 指定字段类型

这里就截取我常用的语言 `Python` 和 `Go`，其他语言查看：[Scalar Value Types](https://developers.google.com/protocol-buffers/docs/proto3#scalar )

| .proto Type | Python Type[3]                  | Go Type | Notes                                                        |
| ----------- | ------------------------------- | ------- | ------------------------------------------------------------ |
| double      | float                           | float64 |                                                              |
| float       | float                           | float32 |                                                              |
| int32       | int                             | int32   | 使用变长编码，对于负值的效率很低，如果你的域有可能有负值，请使用 sint64 替代 |
| int64       | int/long                        | int64   |                                                              |
| uint32      | int/long                        | uint32  | 使用变长编码                                                 |
| uint64      | int/long                        | uint64  | 使用变长编码                                                 |
| sint32      | int                             | int32   | 使用变长编码，这些编码在负值时比 int32 高效的多                |
| sint64      | int/long                        | int64   | 使用变长编码，有符号的整型值。编码时比通常的 int64 高效。      |
| fixed32     | int/long                        | uint32  | 总是 4 个字节，如果数值总是比总是比 228 大的话，这个类型会比 uint32 高效。 |
| fixed64     | int/long                        | uint64  | 总是 8 个字节，如果数值总是比总是比 256 大的话，这个类型会比 uint64 高效。 |
| sfixed32    | int                             | int32   | 总是 4 个字节                                                  |
| sfixed64    | int/long                        | int64   | 总是 8 个字节                                                  |
| bool        | bool                            | bool    |                                                              |
| string      | str/unicode                     | string  | 一个字符串必须是 UTF-8 编码或者 7-bit ASCII 编码的文本。         |
| bytes       | str (Python 2) bytes (Python 3) | []byte  | 可能包含任意顺序的字节数据。                                 |

## 默认值

当某个消息被解析时，如果某个被解析的信息不包含字段的值话，会使用该字段类型的默认值

- 对于 string，默认是一个空 string
- 对于 bytes，默认是一个空的 bytes
- 对于 bool，默认是 false
- 对于数值类型，默认是 0
- 对于枚举，默认是第一个定义的枚举值，必须为 0
- 对于消息类型（message），默认值根据编程语言来确定（`Python` 中为 `None`）


## 分配字段编号

消息定义中每个字段都有一个唯一的编号（1、2、3、4...）。这些编号用来标识二进制格式的字段。

1-15 只需要一个字节进行编码，而 16-2047 则需要两个字节（经常使用的字段应当对应 1-15 编号）

- 编号范围是 1 到 2^29-1(536,870,911)
- 编号 19000 到 19999 不可用（`protocol buffer` 协议实现中保留）

## 保留字段编号

对消息的字段有更新或者删除操作时最好使用 `reserved` 来声明保留字段编号，避免由于消息的更新或者删除操作无法正常解析

```protobuf
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

## 枚举

当某个字段的值需要是某些特定值中的一个时，可以在消息定义中添加一个枚举 `enum` 并且为每个特定值定义一个常量。例如，假设要为每一个 SearchRequest 消息添加一个 corpus 字段，而 corpus 的值可能是 UNIVERSAL，WEB，IMAGES，LOCAL，NEWS，PRODUCTS 或 VIDEO 中的一个。 其实可以很容易地实现这一点：通过向消息定义中添加一个枚举（enum）并且为每个可能的值定义一个常量就可以了。

```protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
```

Corpus 枚举的第一个常量映射为 0：每个枚举类型必须将其第一个类型映射为 0，这是因为：

- 必须有有一个 0 值，我们可以用这个 0 值作为默认值
- 这个零值必须为第一个元素，为了兼容 proto2 语义，枚举的第一个值总是默认值

如果想在一个枚举中，不同的值可以指定相同的常量，需要设置 `option allow_alias = true;`，否则编译器会报错

```protobuf
message MyMessage1 {
  enum EnumAllowingAlias {
    option allow_alias = true;
    UNKNOWN = 0;
    STARTED = 1;
    RUNNING = 1;
  }
}
message MyMessage2 {
  enum EnumNotAllowingAlias {
    UNKNOWN = 0;
    STARTED = 1;
    // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
  }
}
```

枚举中设置的常量必须在 32 位整形范围内，因为枚举值是使用可变编码方式的，对负数不够高效，因此不推荐在枚举中使用负数

## 嵌套类型

你可以在其他消息类型中定义、使用消息类型，在下面的例子中，Result 消息就定义在 SearchResponse 消息内，如：

```protobuf
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

## 使用其他消息类型

### 导入定义

可以通过导入其他 `.proto` 文件中的定义来使用它们

```protobuf
import "myproject/other_protos.proto";
```

### 使用 proto2 消息类型

在你的 proto3 消息中导入 proto2 的消息类型也是可以的，反之亦然，然后 proto2 枚举不可以直接在 proto3 的标识符中使用（如果仅仅在 proto2 消息中使用是可以的）



------

看文档中还有 Map、Any、Oneof 这样的语法，没有实际接触过，后续接触了在进行补充 

# 定义服务 (Service)

定义服务就比较简单了，根据不同的响应模式添加 `stream`

```protobuf
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse); // 一元模式
  rpc Search (stream SearchRequest) returns (SearchResponse); // 客户端流模式
  rpc Search (SearchRequest) returns (stream SearchResponse); // 服务端流模式
  rpc Search (stream SearchRequest) returns (stream SearchResponse); // 双向流模式
}
```
