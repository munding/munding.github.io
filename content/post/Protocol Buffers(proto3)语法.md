---
title: Protocol Buffers(proto3)语法
date: 2022-04-14
tags: ["proto","网络协议"]
description: "protocol buffers 是一种灵活，高效，自动化机制的结构数据序列化方法－可类比 XML，但是比 XML 更小、更快、更为简单！谷歌出品，必是精品"
summary: ""
draft: true
---

>protocol buffers 是一种灵活，高效，自动化机制的结构数据序列化方法－可类比 XML，但是比 XML 更小、更快、更为简单。你可以定义数据的结构，然后使用特殊生成的源代码轻松的在各种数据流中使用各种语言进行编写和读取结构数据。你甚至可以更新数据结构，而不破坏根据旧数据结构编译而成并且已部署的程序。

>[官方文档 Language Guide (proto3) ](https://developers.google.com/protocol-buffers/docs/proto3 )

Protocol Buffer语法基本可以分为三部分：

1. Options：一些声明选项（使用的`proto`版本、导出语言对应的包名、类名等等）
2. Message：定义或者导入消息
3. Services：定义服务

# 定义消息(Message)

Message基本格式

```protobuf
message <message name> {
	<filed rule>(规则) <filed type>(类型) <filed name>(名称) = <filed number>(编号)
}
```

- message name：在同一个`.proto`文件内必须唯一
- filed rule：可以没有，常用的有`repeated`、`oneof`
- filed type：数据类型，protobuf定义的数据类型, 生产代码的会映射成对应语言的数据类型
- filed name：字段名称，同一个message 内必须唯一
- filed number：字段的编号， 序列化成二进制数据时的字段编号

在一个`.proto`文件中可以定义多个消息

```protobuf
syntax = "proto3"; //必须写在文件开头（空行或者注释行除外）

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */
message SearchResponse {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
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

- `syntax = "proto3";` 作用是声明使用的是`proto3`语法，如果不声明的话`protocol buffer`编译器会默认使用`proto2`语法

## 指定字段类型

这里就截取我常用的语言`Python`和`Go`，其他语言查看：[Scalar Value Types](https://developers.google.com/protocol-buffers/docs/proto3#scalar )

| .proto Type | Python Type[3]                  | Go Type | Notes                                                        |
| ----------- | ------------------------------- | ------- | ------------------------------------------------------------ |
| double      | float                           | float64 |                                                              |
| float       | float                           | float32 |                                                              |
| int32       | int                             | int32   | 使用变长编码，对于负值的效率很低，如果你的域有可能有负值，请使用sint64替代 |
| int64       | int/long                        | int64   |                                                              |
| uint32      | int/long                        | uint32  | 使用变长编码                                                 |
| uint64      | int/long                        | uint64  | 使用变长编码                                                 |
| sint32      | int                             | int32   | 使用变长编码，这些编码在负值时比int32高效的多                |
| sint64      | int/long                        | int64   | 使用变长编码，有符号的整型值。编码时比通常的int64高效。      |
| fixed32     | int/long                        | uint32  | 总是4个字节，如果数值总是比总是比228大的话，这个类型会比uint32高效。 |
| fixed64     | int/long                        | uint64  | 总是8个字节，如果数值总是比总是比256大的话，这个类型会比uint64高效。 |
| sfixed32    | int                             | int32   | 总是4个字节                                                  |
| sfixed64    | int/long                        | int64   | 总是8个字节                                                  |
| bool        | bool                            | bool    |                                                              |
| string      | str/unicode                     | string  | 一个字符串必须是UTF-8编码或者7-bit ASCII编码的文本。         |
| bytes       | str (Python 2) bytes (Python 3) | []byte  | 可能包含任意顺序的字节数据。                                 |

## 默认值

当某个消息被解析时，如果某个被解析的信息不包含字段的值话，会使用该字段类型的默认值

- 对于string，默认是一个空string

- 对于bytes，默认是一个空的bytes

- 对于bool，默认是false

- 对于数值类型，默认是0

- 对于枚举，默认是第一个定义的枚举值，必须为0

- 对于消息类型（message），默认值根据编程语言来确定（`Python`中为`None`）

  [generated code guide 生成代码指南](https://developers.google.com/protocol-buffers/docs/reference/overview)

## 枚举

当某个字段的值需要是某些特定值中的一个时，可以在消息定义中添加一个枚举`enum`并且为每个特定值定义一个常量

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

Corpus枚举的第一个常量映射为0：每个枚举类型必须将其第一个类型映射为0，这是因为：

- 必须有有一个0值，我们可以用这个0值作为默认值
- 这个零值必须为第一个元素，为了兼容proto2语义，枚举的第一个值总是默认值

如果想在一个枚举中，不同的值可以指定相同的常量，需要设置`option allow_alias = true;`，否则编译器会报错

```
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

枚举中设置的常量必须在32位整形范围内，因为枚举值是使用可变编码方式的，对负数不够高效，因此不推荐在枚举中使用负数

## 分配字段编号

消息定义中每个字段都有一个唯一的编号（1、2、3、4...）。这些编号用来标识二进制格式的字段。

1-15只需要一个字节进行编码，而16-2047则需要两个字节（经常使用的字段应当对应1-15编号）

- 编号范围是1到2^29-1(536,870,911)
- 编号19000到19999不可用（`protocol buffer`协议实现中保留）

## 指定字段规则

规则可以是以下之一

- singular（单数）：字段在消息中可以有0个或者1个（但不能超过一个）。proto3默认的字段规则，通常被省略
- repeated（重复）：表示该字段在在消息中可以重复任意个数（包括0个），且顺序会被保留

```protobuf
...

message SearchResponse {
  repeated Result results = 1; // 表示results由多个Result类型的值组成
}
```

## 添加注释

要向`. proto `文件添加注释，使用c/c++的注释语法`//`和`/* ... */`

```protobuf
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
```

## 保留字段编号

如果想保留字段编号或者是字段名称，可以使用`reserved`来声明

```protobuf
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

## 使用其他消息类型

可以将其他消息类型用作字段类型

```protobuf
...

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}

message SearchResponse {
  repeated Result results = 1;
}
```

## 导入定义

想要使用的消息类型在其他`.proto`文件中可以通过`import`声明导入

```protobuf
import "myproject/other_protos.proto";
```



# 定义服务(Service)

定义服务就比较简单了

```protobuf
```

