---
title: "Python 中的线程安全和原子操作"
date: 2020-04-19T10:45:32+08:00
draft: false
tags: ["Python"]
categories: ["编程语言"]
---

经常看到一些 Python 第三方库的 features 中都写到了 Thread safety（线程安全），那么究竟什么是线程安全呢？

# 线程不安全

首先看看线程不安全的情况，下面一段代码开启的了两个线程，对全局变量 number 自增 100 万次

```python
from threading import Thread

number = 0

def target():
    global number
    for _ in range(1000000):
        number += 1

thread_01 = Thread(target=target)
thread_02 = Thread(target=target)
thread_01.start()
thread_02.start()

thread_01.join()
thread_02.join()

print(number)
```

```
1476577
1134416
1437371
```

连续输出多次发现结果并不是我们想要的 200 万，这就是线程不安全。究其原因就是 `number+=1` 这段代码不是原子操作

# 原子操作

原子操作（**atomic operation**），指不会被线程调度机制打断的操作，这种操作一旦开始，就一直运行到结束，中间不会切换到其他线程，有点类似数据库中的事务。

在 Python 的 [官方文档](https://docs.python.org/3/faq/library.html#what-kinds-of-global-value-mutation-are-thread-safe) 中就列出了哪些操作是原子操作（L、L1、L2 是列表，D、D1、D2 是字典，x、y 是对象，i、j 是 int）

```
L.append(x)
L1.extend(L2)
x = L[i]
x = L.pop()
L1 [i:j] = L2
L.sort()
x = y
x.field = y
D[x] = y
D1.update(D2)
D.keys()
```

这些操作不是

```
i = i + 1
L.append(L[-1])
L [i] = L[j]
D [x] = D[x] + 1
```

两个线程同时读取到了同一个 number 值完成自增操作后然后赋值，本来已经加两次的操作却只增加了一次。

# dis 模块

当我们还是无法确定我们的代码是否具有原子性的时候，可以尝试通过 `dis`（Python 字节码反汇编器） 模块里的 dis 函数来查看

```
>>> from dis import dis
>>> number = 0
>>>
>>> def target():
...     global number
...     number += 1
...
>>> dis(target)
  3           0 LOAD_GLOBAL              0 (number)
              2 LOAD_CONST               1 (1)
              4 INPLACE_ADD
              6 STORE_GLOBAL             0 (number)
              8 LOAD_CONST               0 (None)
             10 RETURN_VALUE
```

可以发现 `numver += 1` 这一行代码是由 4 条字节码实现的，其他字节码可以查看 [Python 字节码说明](https://docs.python.org/zh-cn/3/library/dis.html#python-bytecode-instructions)

- LOAD_GLOBAL：加载全局变量 number
- LOAD_CONST：加载被加数 1
- INPLACE_ADD：将两个值相加
- STORE_GLOBAL：相加后的结果重新赋值给 number

当一行代码被分成多条字节码指令的时候，就代表在线程线程切换时，有可能只执行了一条字节码指令，此时若这行代码里有被多个线程共享的变量或资源时，并且拆分的多条指令里有对于这个共享变量的写操作，就会发生数据的冲突，导致数据的不准确。

其实一个操作是不是原子的有两种评判标准（个人理解）：

1. 对于纯 Python 代码，是不是只有一条 bytecode
1. 对于 C 实现的函数，内部有没有释放 GIL（如内置数据类型 ints, lists, dicts, etc 的一些操作）

# 如何线程安全

可以使用 Python 的 threading 模块提供的三种消息通信机制

- Event
- Condition
- Queue

如 `urllib3` 中实现的连接池就使用了 `Queue` 中的 `LifoQueue` 来实现线程安全

# 疑问

## Python 中有 GIL 了为什么还会出现线程不安全呢？

GIL 的作用是：对于一个解释器，只能有一个 thread 在执行 bytecode。所以每时每刻只有一条 bytecode 在被执行一个 thread。GIL 保证了 bytecode 这层面上是 thread safe 的。

但是如果你有个操作比如 `x += 1`，这个操作需要多个 bytecodes 操作，在执行这个操作的多条 bytecodes 期间的时候可能中途就换 thread 了，这样就出现了 data races 的情况了。

## Python 中 list 操作是线程安全为什么还要使用 Queue 呢？

列表操作确实是线程安全的，可以用作多线程中存储对象。但是一般不用列表，而是使用 `Queue`，因为后者内部实现了 `Condition` 锁的通信机制，能保证顺序等等。

# 参考

https://stackoverflow.com/questions/6319207/are-lists-thread-safe

https://www.zoulei.net/2016/07/31/list_dict_threading_safe/

https://www.cnblogs.com/wongbingming/p/9035579.html

https://zhuanlan.zhihu.com/p/34150765
