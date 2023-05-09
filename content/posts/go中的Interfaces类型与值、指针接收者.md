---
title: "Go 中的 Interfaces 类型与值、指针接收者"
date: 2021-07-25T11:01:34+08:00
draft: false
tags: ["Golang"]
categories: ["编程语言"]
---

以下代码有几点说明：

- Cat 和 Dog 都实现了 Animal 接口的 `SetName` 方法
- Cat 实现的接口为值接收者（value receiver）
- Dog 实现的接口为指针接收者（pointer receiver）

```go
package main

import (
	"fmt"
)

type Animal interface {
	SetName()
}
type Cat struct {
	Name string
}

func (c Cat) SetName() {
	c.Name = "喵星人"
}

type Dog struct {
	Name string
}

func (d *Dog) SetName() {
	d.Name = "汪星人"
}

func main() {

	var a1 Animal = Cat{}
	a1.SetName()
	fmt.Println("cat name:", a1.(Cat).Name)

	var a2 Animal = &Cat{}
	a2.SetName()
	fmt.Println("cat name:", a2.(Cat).Name)

	var a3 Animal = &Dog{}
	a1.SetName()
	fmt.Println("dog name:", a3.(*Dog).Name)

	// cannot use Dog{} (value of type Dog) as type Animal in variable declaration:
	// Dog does not implement Animal (SetName method has pointer receiver)
	var a4 Animal = Dog{}
	a1.SetName()
	fmt.Println("dog name:", a4.(*Dog).Name)

	var a5 Dog = Dog{}
	a5.SetName()
	fmt.Println("dog name:", a5.Name)

	var a6 *Dog = &Dog{}
	a6.SetName()
	fmt.Println("dog name:", a6.Name)
}
```

运行这段代码可以发现两点问题：

1. a5、a6 赋值 Dog 结构体的值和指针都可以顺利运行
2. a1、a2、a3、a4 都是 Animal 接口类型，为什么 a4 为 Dog 结构体值的时候编译报错，a1 为 Cat 结构体值却不会

首先，在 golang 中，所有的参数传递都是值传递，并且有等价写法如下：

```go
func (c Cat) SetName() {
	c.Name = "喵星人"
}
```

等价于

```go
func SetName(c Cat) {
	c.Name = "喵星人"
}
```

故 value receiver 和 pointer receiver 区别为：

- value receiver：方法拿到的是结构体的拷贝
- pointer receiver：方法拿到的是结构体指针的拷贝，和原结构体指针指向的是同一个结构体（这也是为什么在方法内访问或者修改结构体需要定义为 pointer receiver）

在 [go 语言官方文档：pointers vs values](https://go.dev/doc/effective_go#pointers_vs_values) 有一段这样的描述：

> When the value is addressable, the language takes care of the common case of invoking a pointer method on a value by inserting the address operator automatically. In our example, the variable b is addressable, so we can call its Write method with just b.Write. The compiler will rewrite that to (&b).Write for us.
>
> 当值是可寻址的时，go 会通过自动插入 address 操作符来处理对值调用指针方法的常见情况。在我们的例子中，变量 b 是可寻址的，所以我们可以用 b.Write 调用它的 Write 方法。编译器将把它重写为（&b）

故以上代码可以理解为：Dog 定义的 `SetName` 接口方法为 pointer receiver，但是 `var a5 Dog = Dog{}` 中的 a5 虽然是 value（可寻址的），我们可以通过 `&a5` 隐式获取 a5 结构体的地址，从而实现 pointer receiver 的调用。这就是为什么 a5、a6 都能调用 `SetName` 并且编译不会报错

那为什么类型为接口类型 Animal 的 a4 就不好使了呢？直接无法编译？

可以根据以上原因推断：接口类型 Animal 的 a4 是不可寻址的，not addressable 的，编译器编译阶段就禁止获取接口类型的地址

还是在 [go 语言官方文档：Interfaces](https://github.com/golang/go/wiki/MethodSets#interfaces) 有一段这样的描述的描述，也可以看看文档中的举例：

> The concrete value stored in an interface is not addressable, in the same way that a map element is not addressable. Therefore, when you call a method on an interface, it must either have an identical receiver type or it must be directly discernible from the concrete type: pointer- and value-receiver methods can be called with pointers and values respectively, as you would expect. Value-receiver methods can be called with pointer values because they can be dereferenced first. Pointer-receiver methods cannot be called with values, however, because the value stored inside an interface has no address. When assigning a value to an interface, the compiler ensures that all possible interface methods can actually be called on that value, and thus trying to make an improper assignment will fail on compilation.

接口类型和 map 一样是无法寻址的，接口类型是一种抽象，实现接口类型的结构体可能会有很多。（感觉还是跟语言设计有关，Python 这样的脚本语言就是在运行过程中才知道对象有哪些方法，哪些属性）

**Value-receiver methods can be called with pointer values because they can be dereferenced first.**

**Pointer-receiver methods cannot be called with values, however, because the value stored inside an interface has no address.**

Cat 实现的 `SetName` 为 Value receiver，所以调用者为 value 或者是 pointer 都是 ok 的（pointer 会先取值，dereferenced）；

Dog 实现的 `SetName` 为 pointer receiver，当调用者为 value 时，由于调用者的类型为 Animal 接口类型（不可寻址的），所以无法通过 value 获取到 pointer，所以编译期就失败。

小小总结一下，对于接口类型变量调用实现的方法注意下面的情况：

|                      | value 调用                 | pointer 调用           |
| -------------------- | ------------------------- | --------------------- |
| value receiver 方法   | 编译✅                     | 编译✅（dereferenced） |
| pointer receiver 方法 | 编译❌ （not addressable） | 编译✅                 |

最后关于「变量自动取地址」和「指针自动解引用」也算是 golang 提供的为数不多的语法糖吧，不研究一下确实不明白～