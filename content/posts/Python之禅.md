---
title: "Python 之禅"
date: 2020-01-16T17:00:13+08:00
draft: false
tags: ["Python"]
categories: ["编程语言"]
---

## 原文

> Beautiful is better than ugly.
> Explicit is better than implicit.
> Simple is better than complex.
> Complex is better than complicated.
> Flat is better than nested.
> Sparse is better than dense.
> Readability counts.
> Special cases aren't special enough to break the rules.
> Although practicality beats purity.
> Errors should never pass silently.
> Unless explicitly silenced.
> In the face of ambiguity, refuse the temptation to guess.
> There should be one-- and preferably only one --obvious way to do it.
> Although that way may not be obvious at first unless you're Dutch.
> Now is better than never.
> Although never is often better than *right* now.
> If the implementation is hard to explain, it's a bad idea.
> If the implementation is easy to explain, it may be a good idea.
> Namespaces are one honking great idea -- let's do more of those!

## 解读

The Zen of Python 是 Python 语言的指导原则，遵循这些基本原则，你就可以像个 Pythonista 一样编程。具体内容你可以在 Python 命令行输 `import this` 看到：

- Beautiful is better than ugly.

  \# 优美胜于丑陋（Python 以编写优美的代码为目标）

- Explicit is better than implicit.

  \# 明了胜于晦涩（优美的代码应当是明了的，命名规范，风格相似）

- Simple is better than complex.

  \# 简洁胜于复杂（优美的代码应当是简洁的，不要有复杂的内部实现）

- Complex is better than complicated.

  \# 复杂胜于凌乱（如果复杂不可避免，那代码间也不能有难懂的关系，要保持接口简洁）

- Flat is better than nested.

  \# 扁平胜于嵌套（优美的代码应当是扁平的，不能有太多的嵌套）

- Sparse is better than dense.

  \# 间隔胜于紧凑（优美的代码有适当的间隔，不要奢望一行代码解决问题）

- Readability counts.

  \# 可读性很重要（优美的代码是可读的）

- Special cases aren't special enough to break the rules.
  Although practicality beats purity.

  \# 即便假借特例的实用性之名，也不可违背这些规则（这些规则至高无上）

- Errors should never pass silently.
  Unless explicitly silenced.

  \# 不要包容所有错误，除非你确定需要这样做（精准地捕获异常，不写 except:pass 风格的代码）

- In the face of ambiguity, refuse the temptation to guess.

  \# 当存在多种可能，不要尝试去猜测

- There should be one-- and preferably only one --obvious way to do it.

  \# 而是尽量找一种，最好是唯一一种明显的解决方案（如果不确定，就用穷举法）

- Although that way may not be obvious at first unless you're Dutch.

  \# 虽然这并不容易，因为你不是 Python 之父（这里的 Dutch 是指 Guido）

- Now is better than never.
  Although never is often better than *right* now.

  \# 做也许好过不做，但不假思索就动手还不如不做（动手之前要细思量）

- If the implementation is hard to explain, it's a bad idea.

  \# 如果你无法向人描述你的方案，那肯定不是一个好方案

- If the implementation is easy to explain, it may be a good idea.

  \# 如果你能向人简洁描述你的方案，那也许是一个好方案（方案测评标准）

- Namespaces are one honking great idea -- let's do more of those!

  \# 命名空间是一种绝妙的理念，我们应当多加利用（倡导与号召）

这首特别的 “诗” 开始作为一个笑话，但它确实包含了很多关于 Python 背后的哲学真理。Python 之禅已经正式成文 PEP 20，具体内容见：[PEP 20](https://www.python.org/dev/peps/pep-0020/)