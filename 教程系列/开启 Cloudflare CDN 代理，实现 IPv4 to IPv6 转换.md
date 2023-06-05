---
title: 开启 Cloudflare CDN 代理，实现 IPv4 to IPv6 转换
date: 2023-06-01 16:29:06
tags: 
- 原创
- cloudflare
- CDN
- IP
categories:
- 折腾之路
---

# 开启 Cloudflare CDN 代理，实现 IPv4 to IPv6 转换

通过公网IPv6地址实现远程访问专栏系列文章：
1. [《使用公网IPv6远程访问内网设备》](https://ccccx159.github.io/2023/03/21/%E4%BD%BF%E7%94%A8%E5%85%AC%E7%BD%91IPv6%E8%BF%9C%E7%A8%8B%E8%AE%BF%E9%97%AE%E5%86%85%E7%BD%91%E8%AE%BE%E5%A4%87/)
2. [《DDNS动态域名解析IPv6地址》](https://ccccx159.github.io/2023/03/21/DDNS%E5%8A%A8%E6%80%81%E5%9F%9F%E5%90%8D%E8%A7%A3%E6%9E%90IPv6%E5%9C%B0%E5%9D%80/)
3. [《开启 Cloudflare CDN 代理，实现 IPv4 to IPv6 转换》](https://ccccx159.github.io/2023/06/01/%E5%BC%80%E5%90%AF%20Cloudflare%20CDN%20%E4%BB%A3%E7%90%86%EF%BC%8C%E5%AE%9E%E7%8E%B0%20IPv4%20to%20IPv6%20%E8%BD%AC%E6%8D%A2/)

> <font color="blue">温馨提示：</font>
> 本文存在一部分付费内容，但是付费仅限于域名的购买，如果已经有域名的朋友，请放心大胆食用本文，因为剩余内容均为免费使用。没有域名的朋友，可以移步上一篇文章[《DDNS动态域名解析IPv6地址》](https://ccccx159.github.io/2023/03/21/DDNS%E5%8A%A8%E6%80%81%E5%9F%9F%E5%90%8D%E8%A7%A3%E6%9E%90IPv6%E5%9C%B0%E5%9D%80/)，里面详细介绍了如何在腾讯云购买便宜好用的域名。

## 一、前言

在前两篇文章中，我们详细介绍了如何开启 IPv6 来实现远程访问内网设备，以及如何使用域名和搭建 DDNS 服务实现通过域名来进行远程访问。我们先简单回顾一下，首先需要开启本地网络运营商分发 IPv6 地址的功能，并且开启内网设备的 IPv6 网络权限，因为 IPv6 地址是公网地址，所以此时我们可以直接使用内网设备具体的 IPv6 地址进行直接访问。但是由于IPv6 地址实在是太长了，难以记忆，因此我们通过域名（domain）来绑定 IP 地址，方便记忆。运营商提供的 IP 地址是动态地址，在一定时间后、或者重新拨号联网后会发生变化，针对这一情况，我们在本地搭建了 DDNS 服务，用于监测当前的 IP 地址是否发生变化，如果发生变化，则将新 IP 发送给 DNS 解析服务商，更新域名的 DNS 解析记录。

但是我们仍然遗留了一个问题，部分网络环境没有 IPv6 解析能力，比如我们公司的网络，在这种情况下，我们无法仅仅使用前两篇文章的内容来进行远程访问了。那么是否有方法能在 IPv4 Only 的环境来访问 IPv6 的站点呢？答案是有的，即<mark>套用 CDN 进行流量回源，简单来说就是在源站和客户端之间建立一个中间服务节点，用来 IPv6 和 IPv4 流量的双向转换</mark>。

当然答案并不是唯一的，有能力的大神，完全可以自建用于中继转换的服务，但是有免费，简单的轮子，我们当然首选直接拿来用啦。

## 二、什么是 CDN

全称：Content Delivery Network 或 Content Ddistribute Network，即内容分发网络，顾名思义，它是一个分布式节点网络（也称为边缘位置服务器），它有助于根据用户的位置，内容源服务器和边缘服务器向最终用户的地点传送内容（网页、视频、图像等）。CDN节点具有缓存内容的缓存功能，并且可以从地理上靠近最终用户的位置向用户提供内容。CDN节点由CDN提供商部署在多个地理位置，并且可以跨越多个ISP（因特网服务提供商）网络。

简单来说，它是一个边缘位置服务器，再简单点，它就是一个服务器，用来干什么，用来传递（中转）内容。也就是说我们在访问源站的过程中，实质上是先访问了 CDN 中的边缘服务器，然后由它向源站请求内容后再由它向我们传送了响应内容。

如果无法理解，那也没关系，看完本文内容，会用即可。

## 三、为什么选择 Cloudflare（简称“CF”）

我们先说一下 Cloudflare 开启 CDN 后最大的缺点：慢！如果使用 Cloudflare 提供的默认边缘节点，有可能会让你的访问速度变得奇慢无比，因为 Cloudflare 的服务器大部分在境外，所以国内访问这些境外的边缘节点的速度你自然懂的。

但是为什么我们还是选择 Cloudflare 呢？有这几个让人无法拒绝的理由：
1. 提供了免费的 DNS 解析和 CDN 代理，DNS 支持泛解析；
2. CDN 支持 IPv4 和 IPv6 双栈流量的互相转换；
3. 可以使用第三方开源的 Cloudflare 边缘节点 IP 优选脚本，通过 host 劫持来提高访问速度；
4. 启用 CDN 后，我们可以隐藏真实 IP 地址，提高个人网络安全；
5. 国内的 CDN 收费，并且需要绑定实名备案的云服务器，部分 CDN 不支持 IPv6 回源（腾讯云默认 CDN 不支持，需要购买额外的 ECDN 支持 IPv6 的回源）；

其实仅凭第四点就薄纱国内的 CDN 服务了。CF 速度慢可以通过付费和优选 IP 解决，既然都要付费，那为什么不付给更良心的 CF 呢？

## 四、将域名托管至 CF 

前文中，我们在腾讯云购买了域名，并使用 dnspod 进行域名解析。那我们在使用 CF 前，首先要做的就是将域名托管到 CF。CF 使用需要注册账号，这一步就不做过多赘述了，网站支持简体中文，我相信按照说明注册账号应该都能顺利完成。

在注册完成后，我们点击主页中的“**添加站点**”按钮，导入我们购买的域名：

![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/483cec301c56b56f697ccb21605e50ce/1685951169889.jpg)

![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/37a856c85d1a6fe44f380261327cbb6f/1685953812251.jpg)

这里我们选择免费计划即可，如果有额外需求的，可以按需选择付费计划：

![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/e64117b1d2645418d6777759462d8143/1685954149112.png)


![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/b2ca18a9f53b7c773ed56b598e340aed/1685954472920.png)

完成到这一步时，我们已经完成了托管过程中在 CF 的界面所有操作，接下来我们去腾讯云的控制台，修改域名的名称服务器：

进入我的域名界面：

![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/1ffede4f3fc54987e9195931b3ddbca4/1685955178290.png)

在“修改DNS服务器”界面中，完成名称服务器的修改：

![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/795c0d5460d21f80df505c4a368f94c7/1685954836479.png)


## 四、开启 CDN

在域名托管的过程中，CF 会自动将原有的域名解析记录导入，我们进入 CF 的域名详情页面，选择左侧的 DNS 选项，打开当前域名的 DNS 解析记录界面：

![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/d39ea2b19a4fb646e050a8100fec5d31/image.png)

可以看到我这边已经添加了几条解析记录，下面我们从零开始介绍，如何添加解析记录并开启代理。

1. 手动添加一条 DNS 泛解析记录，并关闭代理：

   ![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/7c7b5244b0822b67697fc111b1385e26/image.png)

2. 本地尝试 Ping 域名，确认 DNS 解析生效：

   ![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/237a1e5abb6c40d63531a4418c43906b/image.png)

   可以看到 CF 的 DNS 解析已经生效了，域名被正确解析到了我们填写的 IP 上。

3. 修改 DNS 解析记录，开启 CDN 代理：

   ![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/88df5e9aab35e19c32c3ff9ecb895f63/image.png)

4. 再次尝试 Ping 域名，观察其返回的 IP 是否已经更新为代理的边缘节点 IP：

   ![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/731154f5637920912d564a89d9f572cd/image.png)

5. 关闭本地电脑的 IPv6 网络，重新 Ping 域名，观察是否能正常 Ping 通，且返回的 IP 为 IPv4 地址：

   ![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/c0f48015f6d7c6824e1af70cfcf1ac56/image.png)

6. 修改 OpenWrt 中的 DDNS 信息：

   需要先在 CF 的个人资料中获取一个 API Key，用于更新 DNS 解析记录：

   ![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/a53419986eba8d4b3b27de92b501c857/image.png)

   然后再去 OpenWrt 的“动态 DNS”插件中添加/修改 DDNS 服务配置信息：

   ![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/1a4bb932f2720abbc81a808122e1c664/image.png)

   <font color=red>注意：当开启 CDN 代理时，这个插件可能会有 “warn” 级别日志，因为它默认使用了 nslookup 获取域名指向的 IP，在开启代理后 nslookup 获取到的是 CF 边缘节点的 IP 地址，和我们真实的 IP 地址并不相同，并且会获取到多个 IP 导致脚本执行过程中有一步骤 expand_ipv6 会报错。但是这两个问题是没有什么关系的，唯一的影响就是每次检查 IP 的时候，都会强制更新一次 DNS 解析记录，即使真实的 IP 没有发生变化。</font>

经过以上6步，我们已经成功给域名套上了 CDN，所有对域名的请求将通过 CF 的边缘节点进行分发和返回，并且我们可以看到，当本地的 IPv6 网络被关闭时，CF 自动给我们分配了 IPv4 的边缘节点，实现了无 IPv6 网络环境下对 IPv6 源站的访问。

## 五、Cloudflare IP 优选

在上面开启 CDN 代理的操作步骤中，第2步未开启代理时，单次 ping 的响应时间是15 ms，而第4、5步中 ping 的响应时间则直接上涨到了 200 ms，可见 CF 开启 CDN 代理，对我们访问的速度影响还是比较大的。因此我们需要对 CDN 边缘节点的接入 IP 进行优选。

此处推荐一个第三方开源的 IP 优选脚本：[XIU2/CloudflareSpeedTest](https://github.com/XIU2/CloudflareSpeedTest)。详细的使用方法和文档在其 github 主页中都有详细介绍，本文就不再赘述了。

## 六、Cloudflare 开启 CDN 后的局限性

是否只要套用的 CDN，就万事大吉了呢？实则不然，Cloudflare CDN 仅仅能代理 HTTP 和 HTTPS 流量，而我们实际使用过程中，往往还有不同的协议流量，例如 SSH 访问服务器后台（不建议将 SSH 服务暴露到公网）、微软的RDP（mstsc）等，无法通过被代理的域名进行访问。

解决办法倒也不算麻烦，只要单独为特殊的流量设置独立子域名，并关闭代理即可。比如添加一个用于微软 RDP（mstsc）的子域名解析记录：rdp.yourDomain.com，并指定对应的 IP 地址。同时在添加一个 `rdp.yourDomain.com` 的 DDNS 服务即可。但是这种方式由于没有 CDN 的代理，也就意味着将直接访问 IPv6 地址，当处于无 IPv6 能力的环境下时，将不可访问。

当然 Cloudflare 提供了更安全的付费服务 [Cloudflare Spectrum](https://developers.cloudflare.com/spectrum/) 来解决这个问题

![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/e489644ba5f788f159e905142f4fca6d/image.png)

对于 HTTP 和 HTTPS 流量的代理，也存在一定的局限性。由于国内无法使用标准的 80 和 443 端口，因此我们不得不使用非标准端口来进行 HTTP(s) 通信。而 Cloudflare 支持转发的端口存在限制，仅支持以下端口的转发：

~~~shell
HTTP  端口：80、8080、8880、2052、2082、2086、2095
HTTPS 端口：443、2053、2083、2087、2096、8443
~~~

因此势必需要设置端口转发，将 CF 端口的流量转发到部署的服务指定的端口。

![](https://gitlab.ccccxyz.top:8443/xu4nch3n/notebooks/uploads/242211644d7144a59450b84bdc80a411/image.png)

## 七、总结

到这里，IPv6 远程访问的系列专题基本告一段落了。我们通过三篇文章，详细地介绍了如何开启 IPv6 网络，如何通过域名进行远程访问，以及如何在无 IPv6 网络环境下通过 CDN 代理访问 IPv6 源站。

虽然这个专题主要介绍的都是 IPv6，但是 IPv4 网络也同样使用，无非就是域名的 DNS 解析类型从 “AAAA” 转变为 “A” 记录。

尽管 Cloudflare 免费计划无法做到尽善尽美，但是我们可以略微绕个弯进行规避后，一般的个人家用场景基本足够使用了，更别说还有 Frp，ZeroTier 这些优秀的穿透工具可以辅助使用。对 Frp 和 ZeroTier 感兴趣的朋友，推荐观看司波图的这期视频：[独享带宽，教你搭建只属于自己的内网穿透服务器（基于frp与zerotier moon服务器）](https://www.bilibili.com/video/BV1dr4y147aq/?spm_id_from=333.999.0.0&vd_source=d8c59e2abebb7aaa6127921565c34c80)

希望这个专题系列能给有需要的人带去帮助~


## 参考资料

1. [Cloudflare DNS 官方文档](https://developers.cloudflare.com/dns/)
2. [Cloudflare spectrum 官方文档](https://developers.cloudflare.com/spectrum/)
3. [Cloudflare API 官方文档](https://developers.cloudflare.com/api/)
4. [《家里只有 IPv6 公网地址，怎么操作才能使其他 IPv4 网络也访问到？》](https://www.v2ex.com/t/870627)
5. [XIU2/CloudflareSpeedTest](https://github.com/XIU2/CloudflareSpeedTest)