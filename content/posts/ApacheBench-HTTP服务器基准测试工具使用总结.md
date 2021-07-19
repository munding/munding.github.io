---
title: "ApacheBench HTTP服务器基准测试工具使用总结"
date: 2020-05-26
tags: ["HTTP"]
description: ""
summary: ""
draft: true
---

> [ApacheBench](https://httpd.apache.org/docs/2.4/programs/ab.html)（ab）是用于对Apache超文本传输协议（HTTP）服务器进行基准测试的工具。ab命令会创建很多的并发访问线程，模拟多个访问者同时对某一URL地址进行访问。它的测试目标是基于URL的，因此，既可以用来测试Apache的负载压力，也可以测试nginx、lighthttp、tomcat、IIS等其它Web服务器的压力。ab命令对发出负载的计算机要求很低，既不会占用很高CPU，也不会占用很多内存，但却会给目标服务器造成巨大的负载，其原理类似CC攻击。自己测试使用也须注意，否则一次上太多的负载，可能造成目标服务器因资源耗完，严重时甚至导致死机。

# ApacheBench安装

## macOS

Mac下自带apache，查看版本：

```bash
apachectl -v
```

查看ab版本:

```bash
ab -V
```

## Window

Windows系统Apache：[下载链接](https://www.apachehaus.com/cgi-bin/download.plx)

## Linux

Ubuntu

```bash
apt-get install apache2-utils
```

CentOS

```bash
yum -y install httpd-tools
```

# ApacheBench使用

## Options

ab压力测试工具的用法，查看：

```bash
ab -h
# 或者
man ab
```

```bash
Usage: ab [options] [http[s]://]hostname[:port]/path
Options are:
    -n requests     Number of requests to perform
    # 在测试会话中所执行的请求个数（本次测试总共要访问页面的次数）。默认时，仅执行一个请求。
    -c concurrency  Number of multiple requests to make at a time
    # 一次产生的请求个数（并发数）。默认是一次一个。
    -t timelimit    Seconds to max. to spend on benchmarking
                    This implies -n 50000
		# 测试所进行的最大秒数。其内部隐含值是-n 50000。它可以使对服务器的测试限制在一个固定的总时间以内。默认时，没有时间限制。
    -s timeout      Seconds to max. wait for each response
                    Default is 30 seconds
    # 套接字超时之前要等待的最大秒数。默认值为30秒。在2.4.4及更高版本中可用。               
    -b windowsize   Size of TCP send/receive buffer, in bytes
    # TCP发送/接收缓冲区的大小，以字节为单位。
    -B address      Address to bind to when making outgoing connections
    # 建立传出连接时要绑定的地址。
    -p postfile     File containing data to POST. Remember also to set -T
    # 包含要发布的数据的文件。记住也要设置-T。
    -u putfile      File containing data to PUT. Remember also to set -T
    # 包含数据到PUT的文件。记住也要设置-T。
    -T content-type Content-type header to use for POST/PUT data, eg.
                    'application/x-www-form-urlencoded'
                    Default is 'text/plain'
    # 用于POST / PUT数据的内容类型标头，例如 application/x-www-form-urlencoded。默认值为text/plain。
    -v verbosity    How much troubleshooting info to print
    # 设置详细级别 4上方将在标题上显示信息，3上方将显示响应代码（404、200等），2上方将显示警告和信息。
    -w              Print out results in HTML tables
    # 在HTML表格中打印出结果。默认表是两列宽，带有白色背景。
    -i              Use HEAD instead of GET
    # 做HEAD请求，而不是GET。
    -x attributes   String to insert as table attributes
    # 用作的属性的字符串<table>。插入属性。<table here >
    -y attributes   String to insert as tr attributes
    # 用作的属性的字符串<tr>。
    -z attributes   String to insert as td or th attributes
    # 用作的属性的字符串<td>。
    -C attribute    Add cookie, eg. 'Apache=1234'. (repeatable)
    # cookie-name=value 对请求附加一个Cookie:行。 其典型形式是name=value的一个参数对。此参数可以重复，用逗号分割。
    -H attribute    Add Arbitrary header line, eg. 'Accept-Encoding: gzip'
                    Inserted after all normal header lines. (repeatable)
    # 将额外的标头添加到请求。该参数是典型地在一个有效报头线的形式，含有一个冒号分隔的字段值对（即，"Accept-Encoding: zip/zop;8bit"）。
    -A attribute    Add Basic WWW Authentication, the attributes
                    are a colon separated username and password.
    # 向服务器提供BASIC身份验证凭据。用户名和密码用单个:分隔，并通过编码为base64的网络发送。无论服务器是否需要该字符串，都将发送该字符串（即，已发送所需的401身份验证）。
    -P attribute    Add Basic Proxy Authentication, the attributes
                    are a colon separated username and password.
    # 在代理途中提供BASIC身份验证凭据。用户名和密码用单个:分隔，并通过编码为base64的网络发送。不管代理是否需要它都将发送该字符串（即，已发送所需的407代理身份验证）。
    -X proxy:port   Proxyserver and port number to use
    # 使用代理服务器处理请求。
    -V              Print version number and exit
    # 显示版本号并退出。
    -k              Use HTTP KeepAlive feature
    # 启用HTTP KeepAlive功能，即在一个HTTP会话中执行多个请求。默认为no KeepAlive。
    -d              Do not show percentiles served table.
    # 不要显示“ XX [ms]表中的投放百分比”。（旧版支持）。
    -S              Do not show confidence estimators and warnings.
    # 当平均值和中位数相距标准偏差的一倍或两倍以上时，请勿显示中位数和标准偏差值，也不会显示警告/错误消息。并默认为最小值/平均值/最大值。（旧版支持）。
    -q              Do not show progress when doing more than 150 requests
    # 当处理150个以上的请求时，每10％或100个左右的请求ab输出进度计数stderr。该 -q标志将禁止显示这些消息。
    -l              Accept variable document length (use this for dynamic pages)
    # 如果响应的长度不是恒定的，请不要报告错误。这对于动态页面很有用。在2.4.7及更高版本中可用。
    -g filename     Output collected data to gnuplot format file.
    # 将所有测量值写为“ gnuplot”或TSV（制表符单独值）文件。此文件可以轻松导入到Gnuplot，IDL，Mathematica，Igor甚至Excel等软件包中。标签位于文件的第一行。
    -e filename     Output CSV file with percentages served
    # 编写一个逗号分隔值（CSV）文件，其中包含为每个百分比（从1％到100％）提供该百分比请求所花费的时间（以毫秒为单位）。通常，它比“ gnuplot”文件有用。因为结果已经“装箱”了。
    -r              Don not exit on socket receive errors.
    # 套接字接收错误时不退出
    -m method       Method name
    # 请求的自定义HTTP方法。在2.4.10及更高版本中可用。
    -h              Display usage information (this message)
    # 显示使用情况信息。
    -I              Disable TLS Server Name Indication (SNI) extension
    # 禁用TLS服务器名称指示(SNI)扩展
    -Z ciphersuite  Specify SSL/TLS cipher suite (See openssl ciphers)
    # 指定SSL / TLS密码套件（请参阅openssl密码）
    -f protocol     Specify SSL/TLS protocol
                    (SSL2, TLS1, TLS1.1, TLS1.2 or ALL)
    # 指定SSL / TLS协议（SSL2，SSL3，TLS1，TLS1.1，TLS1.2或ALL）。TLS1.1和TLS1.2支持在2.4.4及更高版本中提供。
    -E certfile     Specify optional client certificate chain and private key
    # 指定可选的客户端证书链和私钥
```

## Output

执行命令

```bash
ab -n 1000 -c 200 http://pts.aliyun.com/lite/index.htm/
```

获取结果分析

```bash
# apache版本信息
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking pts.aliyun.com (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests

# 请求返回header类型
Server Software:        Tengine/Aserver
# 请求ip或者域名
Server Hostname:        pts.aliyun.com
# 请求端口，当前请求为https所以端口为443，请求https端口80
Server Port:            80

# 从命令行字符串解析请求URI。
Document Path:          /lite/index.htm/
# 这是第一个成功返回的文档的大小（以字节为单位）。如果在测试过程中文档长度发生变化，则将响应视为错误。
Document Length:        357 bytes

# 测试期间使用的并发客户端数
Concurrency Level:      200
从建立连接到最后接受完成总时间
Time taken for tests:   1.280 seconds
# 完成请求数
Complete requests:      1000
# 失败请求数
Failed requests:        0
# 不在200系列响应代码中的响应数。如果所有响应均为200，则不会打印此字段。
Non-2xx responses:      1000
# 从服务器接收的字节总数
Total transferred:      642000 bytes
# HTML接收字节数，减去了Total transferred中HTTP响应数据中的头信息的长度
HTML transferred:       357000 bytes
# 吞吐率：每秒请求数（总请求数/总时间，相当于LR中的每秒事务数TPS）
Requests per second:    781.47 [#/sec] (mean)
# 用户平均请求等待时间
Time per request:       255.928 [ms] (mean)
# 服务器处理每个请求平均响应时间，mean表示为平均值
Time per request:       1.280 [ms] (mean, across all concurrent requests)
# 由公式计算得出的传输速率 totalread / 1024 / timetaken
Transfer rate:          489.95 [Kbytes/sec] received

# 连接消耗时间分解
Connection Times (ms)
              min  mean[+/-sd] median   max
						最小值 平均值 标准差  中间值   最大值
Connect:       22   30   4.0     30      38
Processing:    22   95 164.5     34    1200
Waiting:       22   83 154.3     33    1200
Total:         46  125 163.4     66    1227

# 按完成请求的百分比，得出完成请求中花费时间最长的那一个请求的时间，也就是这些请求完成时间的最大值（毫秒）
Percentage of the requests served within a certain time (ms)
	# 50%请求完成时间的最大值是66毫秒
  50%     66
  66%     69
  75%     71
  80%     80
  90%    317
  # 90%请求完成时间的最大值是148毫秒
  95%    536
  98%    744
  99%    825
  # // 100%请求完成时间的最大值是1227毫秒（最长请求）
 100%   1227 (longest request)
```

# 性能指标

## 1、吞吐率（Requests per second）

服务器并发处理能力的量化描述，单位是reqs/s，指的是在某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。
记住：吞吐率是基于并发用户数的。这句话代表了两个含义：
a、吞吐率和并发用户数相关；
b、不同的并发用户数下，吞吐率一般是不同的。
计算公式：总请求数/处理完成这些请求数所花费的时间，即：
Request per second=Complete requests/Time taken for tests
必须要说明的是，这个数值表示当前机器的整体性能，值越大越好。

## 2、并发连接数（The number of concurrent connections）

并发连接数指的是某个时刻服务器所接受的请求数目，简单的讲，就是一个会话。

## 3、并发用户数（Concurrency Level）

要注意区分这个概念和并发连接数之间的区别，一个用户可能同时会产生多个会话，也即连接数。在HTTP/1.1下，IE7支持两个并发连接，IE8支持6个并发连接，FireFox3支持4个并发连接，所以相应的，我们的并发用户数就得除以这个基数。

## 4、用户平均请求等待时间（Time per request）

计算公式：处理完成所有请求数所花费的时间/（总请求数/并发用户数），即：Time per request=Time taken for tests/（Complete requests/Concurrency Level）

## 5、服务器平均请求等待时间（Time per request:across all concurrent requests）

计算公式：处理完成所有请求数所花费的时间/总请求数，即：
Time taken for/testsComplete requests
可以看到，它是吞吐率的倒数。同时，它也等于用户平均请求等待时间/并发用户数，即：
Time per request/Concurrency Level
