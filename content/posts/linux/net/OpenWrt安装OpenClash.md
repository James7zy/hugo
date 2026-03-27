+++
date = '2024-11-18T13:27:19+08:00'
draft = false
title = 'Openwrt  安装OpenClash'
categories = ['linux', 'net']
+++

# 简介

[OpenClash](https://github.com/vernesong/OpenClash) 在Github上的简介：

> 本插件是一个可运行在 OpenWrt 上的 Clash 客户端
> 兼容 Shadowsocks、ShadowsocksR、Vmess、Trojan、Snell 等协议，根据灵活的规则配置实现策略代理

说简单点，就是提供一个翻墙的工具。
使用手册： [https://github.com/vernesong/OpenClash/wiki](https://github.com/vernesong/OpenClash/wiki)
下载地址： [https://github.com/vernesong/OpenClash/releases](https://github.com/vernesong/OpenClash/releases)
主要目的是实现全屋科学上网配合安卓TV使用！！
## 安装
- 客户端IPK下载 [前往下载](https://github.com/vernesong/OpenClash/releases)
- 内核下载，同步开发分支，下载解压后请上传至/etc/openclash/core/clash并给权限
### 卸载dnsmasq
```bash
opkg remove dnsmasq
mv /etc/config/dhcp /etc/config/dhcp.bak
```

![](_resources/OpenWrt%E5%AE%89%E8%A3%85OpenClash/0057a5bb266450df8ade91026a100790_MD5.jpg)

> 这是因为接下来需要安装 `dnsmasq-full` ，它会与之冲突

```bash
opkg install dnsmasq-full
```

### 安装依赖

根据官方文件，目前有两种方案：

#### 旧版本基于iptables的Firewall3防火墙

```bash
#iptables
opkg update
opkg install coreutils-nohup bash iptables dnsmasq-full curl ca-certificates ipset ip-full iptables-mod-tproxy iptables-mod-extra libcap libcap-bin ruby ruby-yaml kmod-tun kmod-inet-diag unzip luci-compat luci luci-base
```

### 新版本基于nftables的Firewall4防火墙

```bash
#nftables
opkg update
opkg install coreutils-nohup bash curl ca-certificates ipset ip-full libcap libcap-bin ruby ruby-yaml kmod-tun kmod-inet-diag unzip kmod-nft-tproxy luci-compat luci luci-base
```

> Firewall4 现已替代 firewall3 成为 OpenWrt 镜像中的默认防火墙配置软件. Firewall4 使用了 nftables 代替 iptables 来配置 Linux 的网络过滤规则。
> 
> Firewall4 的 UCI 配置界面与之前的防火墙配置界面一致。旧的防火墙配置会无缝迁移到基于 nftables 的 firewall4。
> 
> `/etc/firewall.user` 文件中的自定义防火墙规则需要手动将其标记为兼容，方能够正常工作。同时，Firewall4 还支持引入 nftables 片段的功能。防火墙相关文档详细描述了如何在 firewall4 下自定义防火墙规则。 部分社区维护的软件包添加的自定义规则可能在此版本无法使用，这些规则将在 22.03 后续更新中逐步迁移到 firewall4。
> 
> iptables工具集不再默认在固件中安装。若有需要，你可以通过 opkg 或者 [ImageBuilder](https://openwrt.org/docs/guide-user/additional-software/imagebuilder) 来安装。 `iptables-nft`, `arptables-nft`, `ebtables-nft` 和 `xtables-nft` 软件包可以在使用 `nftables` 的情况下，提供与之前的工具相同的命令接口。

> 我们安装的为官方22.03版本，是基于nftables的Firewall4防火墙

通过SSH连接至OpenWrt，依次输入命令即可安装。

![](_resources/OpenWrt%E5%AE%89%E8%A3%85OpenClash/22a7ffdd95a617a035f8d422e876d22d_MD5.jpg)

### 安装程序文件

在 [Github Release](https://github.com/vernesong/OpenClash/releases) 下载最新的程序文件，上传软件包进行安装。

![](_resources/OpenWrt%E5%AE%89%E8%A3%85OpenClash/eef66d0a1e73ac05f972bb7a95c8f274_MD5.jpg)

> 刷新页面无法看到程序的话，重新登录一下OpenWrt
> 
> ![](_resources/OpenWrt%E5%AE%89%E8%A3%85OpenClash/e86116f680a5c7563afb3635cc9645b7_MD5.jpg)

## 配置

### 更新内核

首先打开【插件设置】-【版本更新】，这边文件不存在的都要进行更新，如果本地网络无法更新的话，点击右侧的【下载到本地】上传到相应目录即可。

![](_resources/OpenWrt%E5%AE%89%E8%A3%85OpenClash/03ffe22afda2ab6f30e47c8c7236951c_MD5.jpg)

### 订阅设置

增加订阅文件，也就是机场，这个网上找找应该有免费的。

> 如果你不清楚你的机场是什么类型的，建议将【在线订阅转换】打开，【添加Emoji】和【UDP支持】也可以启用。

### 插件设置

1. 选择【模式设置】，拉到底下点击【切换页面到Fake-IP模式】，勾选【使用Meta内核】，保存配置；
	关于不同模式间的区别，可以参考： [浅谈在代理环境中的 DNS 解析行为](https://blog.skk.moe/post/what-happend-to-dns-in-proxy/)
2. 选择【DNS设置】，在【本地DNS劫持】选项默认【使用Dnsmasq转发】，我们选择【禁用】，保存配置；
	> 如果【使用Dnsmasq转发】，那么流量就不经过SmartDns，直接到OpenClash
3. 选择【GEO数据库订阅】，勾选【自动更新 GeoIP MMDB 数据库】【自动更新 GeoIP Dat 数据库】【自动更新 GeoSite 数据库】，保存配置；

### 覆写设置

来到【DNS设置】，勾选【自定义上游 DNS 服务器】【追加上游 DNS】【追加默认 DNS】【Fake-IP 持久化】。

在下方【自定义上游 DNS 服务器】中添加：

> 这边填写的是SmartDNS的地址，添加 UDP/TCP/TLS/HTTPS 四种协议，如果没有安装SmartDNS，就直接添加正常的公共DNS。

接着来到【Meta设置】，勾选【启用 TCP 并发】：

> 剩余的是一些比较基础的设置，可以自行设置
> 
> 如果有不清楚的地方，可以参阅 [OpenClash使用手册](https://github.com/vernesong/OpenClash/wiki) ，上面写的非常详细。

## 启动与测试

回到主界面，点击【启动OpenClash】。

> 如果无法成功启动，一定要检查【运行日志】，根据日志的内容进行排查！

在访问 SpeedTest 网页时，显示的IP为国内宽带，也就说明它的分流规则已经生效，经过测速，也是自身宽带速率。

## 结束

在使用OpenClash之前，OpenVPN占据了比较重要的位置，但是鉴于它是全局代理，所以要频繁连接，现在使用OpenClash后，全部交给它，我们只要上网就可以了！

## 最后

在我实际使用过程中，发现OpenClash的分流做的不是特别理想，比如访问一个香港域名，明明本地线路访问就很快，它会默认为国外线路，另外在访问Nvidia这类国际化网站时，基本也都是走的国外线路，之前下载显卡驱动的时候就是国外线路的网速，而不是国内网络的速度。

后来在设置自定义规则中，发现失灵时不灵，而且类似google这类网站，都划分到漏网之鱼里面去了，有点没搞懂。。。

但是它目前是唯一一款基于nftables的软件，其他这类软件都还只支持iptables，因此这个取舍非常难。

---

