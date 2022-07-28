---
title: "GO 中的 string、rune 和 byte"
date: 2022-07-19
tags: ["go",""]
description: "在 Go 语言中，一个string类型的值既可以被拆分为一个包含多个字符的序列，也可以被拆分为一个包含多个字节的序列"
draft: false
---

Go 语言中的 byte 和 rune 实际上是 uin8 和 int32 类型

- byte 一般来表示一些原始数据（如网络中的数据传输）
- rune 则用来表示 Unciode 字符

在 go 中 string 的底层用的就是 byte 字节数组存储的，它的遍历有两种情况

```Go
package main
import (
    "fmt"
)
func main() {
    s := "abc汉字"

    for i := 0; i <len(s); i++ {
    fmt.Printf("%c,", s[i])
    }

    fmt.Println()

    for _, r := range s {
        fmt.Printf("%c,", r)
    }
}

// 结果
// a,b,c,æ,±,,å,­,,
// a,b,c,汉,字,


```

想获取字符串中的字符个数需要转换 `[]rune` 数组，获取中文字符下标同理

```Go
package main

import (
  "fmt"
)

func main() {
  s := "你好世界"
  fmt.Println(len(s))
  fmt.Println(len([]rune(s)))
}

// 结果
// 12
// 4
```