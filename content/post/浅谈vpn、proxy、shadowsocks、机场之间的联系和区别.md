---
title: "浅谈 vpn、proxy、shadowsocks、机场之间的联系和区别"
date: 2021-03-20
tags: ["科学上网",""]
categories: ["网络协议",""]
description: ""
summary: ""
draft: false
---

## 墙的原理

在讨论 vpn、proxy 这些之前，有必要先提一下目前主流防火墙的实现原理。GFW 实现网络封锁的手段主要有两种：dns 劫持和 ip 封锁（除此之外，还有 dns 污染和关键词过滤，这里我们不讨论）。

### Dns 劫持

ip 是网络上各主机的 “地址”，要想访问 “别人家”，当然得要有地址。但 ip 是一串数字，是给电脑看的，人记起来太麻烦，所以就有了域名（也就是我们常说的网址）和 [dns](https://zh.wikipedia.org/zh-hans/%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F)（网域名称系统，Domain Name System）。

域名是一串英文字符串，方便人记忆。dns 将域名和 ip 关联起来，形成映射。用户访问域名所在的目标网站前，将域名发给 dns 服务器询问这对映射关系，拿到对应的 ip 后就可以在茫茫网海中找到那个 “她” 了。而 GFW 所做的就是站在用户和 dns 服务器之间，破坏它们的正常通讯，并向用户回传一个假 ip。用户拿不到真正的 ip，自然也就访问不到本想访问的网站了。

Dns 劫持是 GFW 早期唯一的技术手段，所以那个时候的用户通过修改 [Hosts](https://zh.wikipedia.org/wiki/Hosts%E6%96%87%E4%BB%B6) 文件的方式就可以零成本突破封锁了。

### ip 封锁

dns 劫持之后，GFW 引入了 ip 封锁，直接锁住了访问目标网站的去路，用户发往被封锁 ip 的任何数据都会被墙截断。

这个时候，依靠类似于修改 Hosts 文件这种低成本方法突破封锁就显得有些天方夜谭了。那么，解决办法是什么呢？答案是：在第三方架设翻墙服务器，中转与目标服务器间的来往流量。目前为止，GFW 采用的是黑名单模式，像 Google、Facebook 这种在黑名单上的网站的 ip 无法访问，而不在黑名单上的第三方不记名 ip 可以。

于是，一切就很明朗了，我们目前几乎所有的翻墙手段都是基于上述原理实现的。vpn 是，shadowsocks 是，还有一些比较冷门的（比如 v2ray）同样如此，只不过它们的技术细节不同（这个我们不会深入）。

## VPN

[VPN](https://zh.wikipedia.org/wiki/%E8%99%9B%E6%93%AC%E7%A7%81%E4%BA%BA%E7%B6%B2%E8%B7%AF)，全称 “虚拟私人网络（Virtual Private Network）” 或者是“虚拟专用网络”，是一种加密通讯技术。vpn 是一个统称，它有很多的具体实现，比如 PPTP、L2TP/IPSec 等。

vpn 最常用的场景就是在外网的小伙伴（如酒店网络，家庭 WIFI 等等）访问公司内部的网络使用，如果不使用 vpn，直接连入公司内网，则数据明文在外网中传输会有被窃听的风险。vpn 是一种加密通讯技术，它被设计出来的目的是数据传输安全和网络匿名。

vpn 出现远早于 GFW，所以它不是为了翻墙而生的。而既然不是为翻墙而生，那从翻墙的角度上讲，vpn 协议就存在诸多问题。使用 VPN，不足之处在于数据分流不灵活，会将开启了 VPN 的设备的所有数据流量全部导向至 VPN 服务器上；另外如果 VPN 服务器上有流量监视软件运行，那么用户所传输的数据将有信息安全威胁；进一步来说，由于 VPN 设计的初衷并不是用于翻墙，因此数据流量的特征非常明显，容易引起审查机构注意，导致被封。

所以，VPN 这种翻墙方式基本已经没落了。但即便如此，vpn 作为过去很长一段时间最主流最热门最常用最为人所知的翻墙手段，已然成为翻墙的代名词。即便是 vpn 已不再常用的今天，当人们谈及翻墙的时候，说得最多的仍是：“你有什么好用的 vpn 吗？”。

## Proxy（代理）

Proxy（代理）又分为正向代理和反向代理。

## 正向代理

翻墙所用的代理都是正向代理。正向代理主要有 HTTP、HTTP over TLS(HTTPS)、Socks、Socks over TLS 几种。其中，HTTP 和 Socks 无法用于翻墙，HTTPS 和 Socks over TLS 可以用于翻墙。不过，Socks over TLS 几乎没人用，我们这里就不多说了。

Proxy 的历史同样早于 GFW，它最早被设计出来的目的当然也不是翻墙。正向代理最主要的目的和 vpn 差不多，都是用于匿名，但 HTTP 和 Socks 不能加密，只能匿名，HTTPS 既可以匿名，也可以用于加密通信。

从理论上讲，四种代理协议都可以通过 “用户先将数据发给代理服务器，再由代理服务器转发给目的服务器” 的方法达到翻墙目的。但由于 HTTP 和 Socks 都是明文协议，GFW 可以通过检查数据包内的内容得知用户的真实意图，进而拦截数据包。所以，HTTP 和 Socks 一般只用作本地代理。而 HTTPS 协议是加密通讯，GFW 无法得知数据包内的真实内容，类似于关键词过滤的手段无法施展。不仅如此，HTTPS 代理的流量特征和我们平时访问网站时所产生的 HTTPS 流量几乎一模一样，GFW 无法分辨，稳定性爆表。理论上讲，HTTPS 代理无论是安全性，还是在隐匿性，都要比目前最为流行的 shadowsocks 好。

事实上，在所有已知的翻墙协议中，无论是 vpn 协议，还是代理协议，它应该都是最好的。v2ray 的 vmess over tls 也许能和 HTTPS 代理媲美。但 v2ray 存在的时间较短、使用者较少、社区也没有 HTTPS 代理活跃（从全球范围上看），故而，相比于 HTTPS 代理，vmess 协议潜在的安全漏洞可能要多。

当然，HTTPS 代理也有它的缺点，其中最大的缺点就是配置复杂。即便能用默认参数就用默认参数，用户自己只作最低限度的配置，对新手而言，这也是一个无比痛苦的过程。更别说，想要正常使用 HTTPS 代理，你还要购买域名和证书这些，非常麻烦。所以，即便是在 shadowsocks 出现之前，HTTPS 代理也没在大陆流行起来。这也是造成 v2ray 的小众的主要原因之一（另一个是用户没有从 shadowsocks 迁移到 v2ray 的动力），它的配置同样相当复杂。除此之外，HTTPS 代理只能转发 tcp 流量，对 udp 无能为力。
这里推荐刘亚晨先生的一篇文章「[各种加密代理协议的简单对比](https://medium.com/@Blankwonder/%E5%90%84%E7%A7%8D%E5%8A%A0%E5%AF%86%E4%BB%A3%E7%90%86%E5%8D%8F%E8%AE%AE%E7%9A%84%E7%AE%80%E5%8D%95%E5%AF%B9%E6%AF%94-1ed52bf7a803)」。

## 反向代理

反向代理的作用主要是为服务器做缓存和负载均衡。这里不做过多讨论，感兴趣的朋友可以看 [这里](https://zh.wikipedia.org/wiki/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86)。顺带一提，shadowsocks 里也有负载均衡的概念，但 shadowsocks 的负载均衡和反向代理的负载均衡不是一个概念。

反向代理的负载均衡是指：在多个真正的服务器前架设一个代理服务器，用户所有的数据都发给代理服务器，然后代理服务器根据各个真实服务器的状态将数据转发给一个任务较少的服务器处理。这样，服务商既可以架设多个服务器分担任务、减轻压力，用户也只要记一个域名或 ip 就可以了。

而 shadowsocks 的负载均衡是指：每隔一段时间更改一次翻墙服务器，将用户的数据平均发给多个不同的翻墙服务器，以避免发往某一个翻墙服务器的流量过多。



## shadowsocks

最后，就是我们的 shadowsocks 闪亮登场了。介绍之前，我这里先附上 shadowsocks 的 [官网链接](http://www.shadowsocks.org/en/index.html)。英文比较好的同学建议看看官网上对 shadowsocks 的介绍。

在 shadowsocks 之前，墙内网民主要依靠寻找现成的技术实现翻墙。比如 vpn、HTTPS、tor 的中继网桥以及之后的 meek 插件等等，虽然也有自己的技术，比如一种依靠 Google 隐藏 ip 实现翻墙的技术（名字忘了）, 但毕竟难成大器，再加上 GFW 逐渐加大对 VPN 的干扰，人们迫切需要一种简单可靠的技术来抵御 GFW 的进攻。

于是，大概是在 2013 年吧（具体时间我也不太清楚），[@clowwindy](https://github.com/clowwindy) 带着他的 shadowsocks 横空出世。Shadowsocks 同样是一种代理协议，但是作为 clowwindy 为国人设计的专门用于翻墙的代理协议，相对于 vpn，shadowsocks 有着极强的隐匿性；相对于 HTTP 代理，shadowsocks 提供了较为完善的加密方案，虽然比不上 HTTPS 代理和 vpn，但使用的也是成熟的工业级的加密算法，普通个人用户完全不用顾虑；相对于 HTTPS 代理，shadowsocks 的安装配置更为简单，中文社区更为活跃，中文文档教程更完善，更符合中国国情。

Shdadowsocks 最初的版本是由 clowwindy 使用 Python（一种目前非常热门的脚本编程语言）实现的。所以 clowwindy 的版本被称为 Python 版。shadowsocks 有点名气之后，不同的开发者使用不同的编程语言为其写了很多分支版本。比如，[@cyfdecyf](https://github.com/cyfdecyf)开发维护的 Go 版本，[@madeye](https://github.com/madeye)开发维护的 libev 版本（由纯 C 语言编写，基于 libev 库开发），由 [@librehat](https://github.com/librehat) 开发维护的 c++ 版，由 [@zhou0](https://github.com/zhou0) 开发维护的 Perl 版。这些版本的安装使用指南都可以在 shadowsocks 的官网上查阅。

2015 年，clowwindy 因喝茶事件被迫停止了 shadowsocks 的维护，并删除了其开源在 GitHub 上的代码，Python 版就此停滞。但其它版本仍处于维护更新中。其中，更新最频繁，新技术跟进最快的是由 @madeye 维护的 libev 版本。这里有必要说明下，目前，shadowsocks 协议（请区分 “shadowsocks 协议” 和“shadowsocks 协议的具体实现”这两者的区别）是由 shadowsocks 社区内的成员共同维护，协议上任何新改进都是社区成员共同商讨的结果。但对这些变化，不同的版本的 shadowsocks 跟进速度不同。而跟进速度最快的就是我上面说的 libev 版。无论是 [SIP007](https://github.com/shadowsocks/shadowsocks-org/issues/42) 确认的 ADEA Ciphers（一种同时进行认证和加密的算法），还是 [SIP003](https://github.com/shadowsocks/shadowsocks-org/issues/28) 引进的 simple-obfs（tor 开发的一种混淆插件），shadowsocks-libev 都是最早引入自己软件的。

shadowsocks 是 c/s 架构，shadowsocks 的客户端则就是百花齐放了，有我们现在用的小飞机（Shadowsocks），ClashX，移动端等等。

## 机场

随着 GFW 的不断升级，其实 shadowsocks 流量也会被检测出来，导致部署 shadowsocks 的服务器 IP 被封禁。其实 shadowsocks 还只是众多科学上网协议中的一种，其实还有 ssr（ShadowsocksR），v2ray（改善了 shadowsocks 的一些缺点，更难被 GFW 检测到，不过配置复杂），Trojan（上文提到过，模仿 https 流量，隐蔽性更强）。

由于自己购买国外的 vps 部署的协议随着 GFW 的不断升级是有可能被识别到导致封 IP 的，如果 vps 提供商不支持更换 IP，那么你这台 vps 就浪费了。如果不想被封就需要不断学习新的隐蔽性更好的协议，所以对于个人用户来说，学习成本很高。

这时候，机场服务就应运而生，只需要少量的付费，通过一条订阅就可以拿到上百条支持各种协议的线路，即使是那个节点被封，那随便切换一条就行。如今对于普通用户来说，机场已经成为了最多的科学上网方式。

## 总结

- vpn 是是一种加密通讯技术，它的核心技术是在加密，防窃听上。由于 GFW 刚上的时候 vpn 这项技术成熟，vpn 被迫营业，充当起了第一代翻墙手段。
- shadowsocks 闪亮登场，有着很强的隐蔽性，配置简单
- GFW 不断升级，更多的隐蔽性强的协议 v2ray，Trojan
- 机场服务，通过订阅拿到上百条线路，并且协议齐全，如果你不太想折腾并且不追求极致，机场服务是个不错的选择。

## 参考

- [https://github.com/sxcool1024/freedom](https://github.com/sxcool1024/freedom)
- [https://github.com/shadowsocks](https://github.com/shadowsocks)