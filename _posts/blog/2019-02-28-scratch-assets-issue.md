---
layout: post
title: Scratch 3.0 的资源加载问题
categories: [Blog]
description: some word here
keywords: scratch, 资源
---


## 问题

在启动 Scratch 3.0 的在线编辑器时，有时会出现无法正常加载编辑器的情况，如下图所示：

![](/images/blog/2019-02-28/scratch-3.0-crash-assets.jpg){: width="500px"}

![](/images/blog/2019-02-28/scratch-3.0-crash-assets-cn.jpg){: width="500px"}

这个现象在 Scratch 3.0 发布的那几天就出现过，当时还以为是因为发布新版本的原因，系统还不太稳定所造成的。但目前该问题似乎仍然没有得到有效的解决，从 Scratch 官网打开在线编辑器的时候就会偶尔就会出现。如果是在本地编译运行 Scratch GUI 的话，偶尔也会出现同样的问题。

## 原因排查

在对编辑器加载过程进行跟踪之后发现，原来是加载过程中所需要用到的两个资源文件偶尔会无法访问到：

    https://assets.scratch.mit.edu/internalapi/asset/b7853f557e4426412e64bb3da6531a99.svg/get/
    https://assets.scratch.mit.edu/internalapi/asset/b7853f557e4426412e64bb3da6531a99.svg/get/

它们其实就是在打开编辑器时，创建项目默认角色（那只黄色小猫）时使用的两张图片，当这两个资源文件无法正常加载时，Scratch 编辑器会出现奔溃的情况。

如此看来，原因就应该比较清楚了：当在线编辑器访问位于 assets.scratch.mit.edu 服务器上的相关资源时，有时会出现访问失败的情况。

## 解决方案

原因找到之后，下面的问题是如何解决呢？从域名我们可以猜测 assets.scratch.mit.edu 应该是 Scratch 用来存放相关资源的服务器，那本质上应该是国内访问该服务器时稳定性不够导致的。那样的话似乎意味着我们只能等着 Scratch 运营的团队来解决这一问题了。

进一步的跟踪发现，为了应对资源访问的问题，Scratch 官网对这些资源的访问使用了 CDN 加速，而且 Scratch 3.0 中有些资源访问对应的就是 cdn.assets.scratch.mit.edu 这样的域名。我们测试了一下上面的两个资源，如果通过相应的 CDN 加速域名访问的话也是可以的：

    https://cdn.assets.scratch.mit.edu/internalapi/asset/b7853f557e4426412e64bb3da6531a99.svg/get/
    https://cdn.assets.scratch.mit.edu/internalapi/asset/b7853f557e4426412e64bb3da6531a99.svg/get/

利用 [DNS 查找工具](https://tools.geekflare.com/tools/dns-lookup)， 我们可以看到无论是 assets.scratch.mit.edu 还是 cdn.assets.scratch.mit.edu 其对应的地址都有多条记录，并且理论上讲是有可能会变化的。

虽然其中的原因目前尚不清楚，但并不妨碍我们得到一个解决编辑器加载失败的临时方案：将所有对域名 assets.scratch.mit.edu 或 cdn.assets.scratch.mit.edu 的访问，解析到一个在国内可以访问到的 IP 地址上。例如，近期通过上面的地址查询到 assets.scratch.mit.edu 对应的 IP 地址为 151.101.2.133 ，如下图所示：

![](/images/blog/2019-02-28/scratch-assets-cdn-address.jpg){: width="500px"}

如果相应的 IP 地址可以 ping 通的话，我们就可以将域名指定到该 IP 地址上。

## 修改 AP

目前我们的做法是通过在树莓派上运行 hostapd 和 dnsmasq 来将树莓派当成一个无线 AP 来使用，当用于编程的计算机（电脑或者平板）通过无线 WiFi 连接到树莓派上之后，相应的地址解析就由树莓派来脱管了。因此，上面的问题可以通过将域名指定为能够访问的 IP 地址来解决：

    151.101.2.133 assets.scratch.mit.edu

这一做法潜在的危险在于如果哪天 151.101.2.133 服务器也无法访问了的话，可能需要再次查找一台可用服务器的 IP 地址，再次对地址解析记录进行修改。

## 后记

目前并不确定这一做法就是最佳方案，后续可能需要做进一步的观察，如果你有发现更多的信息或者想法也欢迎一起讨论。
