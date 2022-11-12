# 黑苹果一键开启 HiDPI


黑苹果和白苹果最大的区别其实在显示效果上。同样一个网页，白苹果的显示就会细腻很多，而黑苹果颗粒感非常严重，造成上述原因是因为大多数苹果设备的屏幕本身的分辨率很高，如果你的显示器分辨率达到视网膜级别的话，哪怕是黑苹果也是默认开启 HiDPI 的。So 本人的 2k 分辨率显示器就很尴尬了，下面记录一下开启 HiDPI 的过程。

## HiDPI 的概念

[有关 retina 和 HiDPI 那点事](https://zhuanlan.zhihu.com/p/20684620)

总之 HiDPI 是苹果一个牛逼的显示技术，通过牺牲一定的分辨率实现更细腻的显示效果，这就是为什么 2K 显示器开启 HiDPI 的效果要比 1080P 好的原因了。

## 黑苹果开启原生 HiDPI

终端中运行如下命令

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/xzhih/one-key-hidpi/master/hidpi.sh)"
```

如果出现 `curl: (7) Failed to connect to raw.githubusercontent.com port 443:xxx`，应该是被墙了，可以挂上梯子

```bash
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

终端运行后选择相应分辨率重启即可生效！

## HiDPI 开启效果

![](https://img.aladdinding.cn/hidpi.png)

可以看到 HiDPI 开启后，「显示器选项」里面的缩放显示如图所示。可随意选择缩放模式而且不会高糊，在显示上明显感觉图标颗粒感更小了，显示更加细腻了。
