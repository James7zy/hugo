+++
date = '2024-11-18T13:27:19+08:00'
draft = false
title = '树莓派4B-Openwrt安装OpenClash'
categories = ['linux', 'net']
+++

# 前言

   如果你的设备也是树莓派 4B+ 且不想折腾的话，我这里准备了一个开箱即用的 OpenWrt 镜像，可以用作二级路由器。如果你是通过 PPPoE 拨号上网，只需要编辑 wan 接口，将其协议改成 PPPoE，填写宽带用户名和密码就行了。镜像自带 OpenClash，分区扩容到了 1G，路由器管理密码和 WiFi 密码都是 12345678，WiFi 名为 OpenWrt，路由器地址为 192.168.110.1。烧录运行后，只需要改下密码和 OpenClash 节点信息就行了。 [天翼云盘（访问码：ba8p）](https://cloud.189.cn/t/3iqiYfYRnA3e) 或者 [OneDrive](https://1drv.ms/u/s!Avz7vfylhEW5kyy60lACnqFGODSC?e=lPcP9v)

# 安装 OpenWrt

## 准备工具

- 1G 以上 micro SD 卡，树莓派 4B+，网线
- 镜像烧录工具 [balenaEtcher](https://etcher.balena.io/)

## 下载镜像

首先是去 OpenWRT 官网找自己的设备对应的固件，去到官网 [下载页](https://openwrt.org/downloads) ：

![](_resources/Raspberry_PI_4B/e7a55cad7c8e0a476a4fff06e0e633bc_MD5.jpg)

点击 *Table of Hardware* ，进入到硬件列表页面：
![](_resources/Raspberry_PI_4B/04f518fabb63bf74fb756e1fa34791cf_MD5.jpg)

在输入框里输入关键字筛选，然后在列表中找到自己的型号。我的板子型号是 3B+，因此选择下载 3B+ 对应的固件。一共有四种固件可供选择，分别是 release 和 snapshot 版本的完整镜像及其升级包。由于我是全新安装，因此选择完整镜像就行了，这里我选择的是 release 版本的完整镜像 Factory image。

注意，如果你选择 snapshot 版本的镜像，可能会遇到一些坑，因为 snapshot 固件的软件仓库没有进行版本隔离，软件仓库中的软件永远是适配最新版固件的，因此后面你安装软件的时候，可能会出现兼容性问题。当然，你可以手动更换到固定版本的软件仓库来解决这个问题，如何更换后面会提到。

## 烧录镜像

下载好镜像后解压得到 openwrt-23.05.0-bcm27xx-bcm2710-rpi-3-ext4-factory.img，然后使用 balenaEtcher 镜像烧录工具，把固件烧录到我们的存储卡中，存储卡不用很大，1G 足够了：  
![](_resources/Raspberry_PI_4B/3b0d8069f52871b10b507784428f2c58_MD5.jpg)  
烧录完成之后，把 SD 卡插入树莓派，通上电，就可以开始配置了。

## 配置 OpenWrt

1. 登录路由器管理后台
	OpenWrt 无线网卡默认是未启用状态，因此没法通过 WiFi 连接并登录路由器管理界面。我们把树莓派通过网线连接到电脑，然后去电脑的设置里查看有线网络的网关 IP，一般都是 192.168.1.1，在浏览器中输入这个 IP 就可以看到 OpenWrt 的管理界面了：
	![](_resources/Raspberry_PI_4B/18777c980e61f4857105271cf0cd1746_MD5.jpg)
2. 修改密码，添加 SSH 公钥
	我们还没有设置过密码，直接点击 Login 就行。登录之后会提示你设置路由器后台密码，点击 Go to password configuration...。你还可以添加电脑的 SSH 公钥，方便后续 SSH 登录。
	![](_resources/Raspberry_PI_4B/08b682a19111bd4622574c446539cdd9_MD5.jpg)
3. 研究一下初始配置（不感兴趣的话可略过）
	去到 Network -> Interface 界面，可以看到只有一个 lan 接口：
	![](_resources/Raspberry_PI_4B/4b96ee6defdafcc3b1b32855e35d41f3_MD5.jpg)
	点击编辑先研究下配置：
	![](_resources/Raspberry_PI_4B/6f9e89e7b126465a8c4b4383c9241c4a_MD5.jpg)

	可以看到这个接口配置的是静态 IP，接口关联的设备是 br-lan 网桥，在这个接口上跑了一个 DHCP 服务器。点击 Dismiss ，切换到 Devices 选项卡，研究下 br-lan 网桥的配置。可以看到 br-lan上附加了一个 eth0，eth0 是树莓派的有线网卡。电脑通过网线连接到路由器，就成功地和路由器建立了链路层的连接，又因为路由器在这个接口上配置了 DHCP 服务器，因此同一链路上的电脑作为 DHCP 客户端自动获得了 IP，加入到了路由器所创建的局域网中。这就是为什么电脑连接树莓派后，不需要任何配置就能获取 IP 和网关，访问路由器后台。
	![](_resources/Raspberry_PI_4B/18a2ff0d2b427e935048ea096ad15af4_MD5.jpg)
	然而现在也只能访问路由器后台，没法联网。为了能联网，我们需要完成以下配置：
	- 创建 wan 接口，并让其作为 DCHP 客户端去连接上层路由器；
		- 修改现在的 lan 接口，让其关联的设备是无线网卡而非现在的以太网卡；
		- 配置并激活无线网卡，开启无线网络，让局域网的无线设备能够连接进来；
	以上是配置思路，接下来说下具体方法。
4. 创建 wan 接口。
	在 Network -> Interface 页面，点击 Add new interface... ：
	![](_resources/Raspberry_PI_4B/5fae4a04ab2a481eaf8c9df67294d9a1_MD5.jpg)
	填入下图所示配置，然后点击 Create interface ：
	Device 选择 br-lan 网桥也行，但网桥上只附加了一个 eth0，就没有使用的必要了，直接用 eth0 就行，br-lan 后面可以直接删了。
	![](_resources/Raspberry_PI_4B/f1591d985c5e82f3b73099d6fdbbe772_MD5.jpg)
	在新弹出的窗口中，点击 Save ：
	![](_resources/Raspberry_PI_4B/02adb6505a2f1d37e22c5fe418bbb19f_MD5.jpg)
5. 开启无线网络，创建无线局域网。
	点击 lan 接口的 Edit 按钮，修改其配置：
	![](_resources/Raspberry_PI_4B/c23ba4ef8ba1440d09185f7ca43b7660_MD5.jpg)
	如下图所示，将 Device 改成 Wireless Network: Master "OpenWrt" (lan) ，Device 展示的名称和选项列表中的名称不一样，这个不用管。IPv4 address 改成 192.168.110.1。然后点击 Save 保存。
	![](_resources/Raspberry_PI_4B/3f7e42dcdb45ad95c19a096d0d40330b_MD5.jpg)
	然后我们去到 Network -> Wireless ，点击 Edit 按钮配置一下无线网络：
	![](_resources/Raspberry_PI_4B/bd9723ad7fe25df7ac8cd671a2ae3c04_MD5.jpg)
	如下图所示，把 Channel 改成 44（最开始我用的是 auto，发现 switch 死活连不上，最后按照网上的方法改成 44 就好了），加密方式我选择的是 WPA-PSK/WPA2-PSK Mixed Mode ，兼容性好一点。 Key 自己看着填就行。最后点击 Save 。
	![](_resources/Raspberry_PI_4B/4546975a6a984fad993010e67075485b_MD5.jpg)
	退出配置后，回到上级界面，点击 Enable ：  
	![](_resources/Raspberry_PI_4B/4838e038f21ac99082193266cdef35e7_MD5.jpg)
	这一步会不仅会激活无线网卡并开启其无线 AP 功能，还有将之前所有的配置都应用。页面上会显示 Applying configuration changes 的 90s 倒计时，你需要在 90s 之内连接到 OpenWrt ，否则配置就会回滚。
	![](_resources/Raspberry_PI_4B/323ae1dca02ad1f75be4f43acfab1a35_MD5.jpg)
	找到名为 OpenWrt 的无线网络，连接成功以后页面会自动刷新，配置应用成功。现在可以断开树莓派和电脑的连接，把树莓派连到路由器的 lan 口上。不出意外的话，就能愉快地上网了。

# 安装 OpenClash

软路由配置好了，接下来可以装代理软件了，我选择的是 OpenClash，原因也是一样，可玩性还行，并且有一定的用户量。就是配置稍微麻烦点。

## 更换 OpenWrt 软件源

在搭 OpenClash 之前我们需要更换一下 OpenWrt 的软件源，否则软件包的下载可能会很慢。去到 System -> Software，点击 Configure opkg...，如下图所示：

![](_resources/Raspberry_PI_4B/eac5214e3672e866723449e1a3d7f5ab_MD5.jpg)  
![](_resources/Raspberry_PI_4B/db37034b74d8f90bffda102286deaa35_MD5.jpg)

将红框中的 [https://downloads.openwrt.org](https://downloads.openwrt.org/) 替换为 [https://mirrors.tuna.tsinghua.edu.cn/openwrt](https://mirrors.tuna.tsinghua.edu.cn/openwrt) ：

```bash
src/gz openwrt_core https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/23.05.0/targets/bcm27xx/bcm2710/packages

src/gz openwrt_base https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/23.05.0/packages/aarch64_cortex-a53/base

src/gz openwrt_luci https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/23.05.0/packages/aarch64_cortex-a53/luci

src/gz openwrt_packages https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/23.05.0/packages/aarch64_cortex-a53/packages

src/gz openwrt_routing https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/23.05.0/packages/aarch64_cortex-a53/routing

src/gz openwrt_telephony https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/23.05.0/packages/aarch64_cortex-a53/telephony
```

## 安装 OpenClash

因为 OpenWrt 软件仓库里并没有提供 OpenClash，我们需要去 [OpenClash Github Release](https://github.com/vernesong/OpenClash/releases) 页下载最新的 ipk 包：

![](_resources/Raspberry_PI_4B/ce78d97dc081b349aeab816cbaa62972_MD5.jpg)

然后按照 Release 页的说明，先通过 ssh 登录软路由（密码是你一开始设置的路由器管理密码）：

```bash
➜  ssh root@192.168.1.1
```

再执行 `opkg update` 命令以获取最新的软件包列表：

```bash
root@OpenWrt:~# opkg update
```

接着安装 OpenClash 依赖，根据所使用的防火墙，选择不同的依赖，我用的是 nftables：

```bash
root@OpenWrt:~# opkg install coreutils-nohup bash dnsmasq-full curl ca-certificates ipset ip-full libcap libcap-bin ruby ruby-yaml kmod-tun kmod-inet-diag unzip kmod-nft-tproxy luci-compat luci luci-base
```

不出意外的话，两三分钟的时间就装. 

依赖安装好后，就可以安装 OpenClash 了。去到 System -> Software，点击 Upload Package...：

![](_resources/Raspberry_PI_4B/c7471b94c3f616426fa28ab3ff8dba7e_MD5.jpg)

点击 Browse... 选择之前下载的 ipk 包，点击 Upload ，然后点击 Install ：

![](_resources/Raspberry_PI_4B/1d36da564ca770a97a120cb670314cf0_MD5.jpg)

![](_resources/Raspberry_PI_4B/b8e59ec5ef95e549d092d7960e896d51_MD5.jpg)

这个时候可能会失败：

![](_resources/Raspberry_PI_4B/388473b826dd35929429e4e76f2c70cd_MD5.jpg)

==看错误信息貌似是 dnsmasq-full 和已经安装的 dnsmasq 有冲突，看名字 dnsmasq-full 应该是比 dnsmasq 功能更全面的包，所以我们可以把 dnsmasq 卸载了==。我们在 Installed 标签页筛出这个包来，点击 Remove 。（直接在 ssh 里执行 `opkg remove dnsmasq` 也行）

![](_resources/Raspberry_PI_4B/c75c9ed4cf6d8f05fd51cbf6f0afcae2_MD5.jpg)

再次安装 OpenClash ipk 包，不出意外的话就装好了。可能会弹出 XHR 错误，忽略它。重启路由器，会发现顶部菜单栏多出了一个 Service ，OpenClash 入口就在这个菜单下。点击进到 OpenClash 页，这时候 OpenClash 会提示你安装内核，点击 Install 进行安装。然后会自动跳转到日志页展示内核的安装日志，==我这里的日志显示是安装成功了，如果你安装失败了，就得手动下载内核放到指定目录，可以去网上搜一下，很简单。==
[[OpenWrt安装OpenClash]]

![](_resources/Raspberry_PI_4B/d0bdb8df3e550fb950ff28521f990cca_MD5.jpg)

## 编辑 OpenClash 配置

回到 OpenClash，切换到 Config Manage 选项卡，滑到下面的 **Config File Edit** 编辑你的配置。左边是编辑区，右边的是运行时配置预览区。如果你手里已经有配置文件，只需要把它粘贴到左边就行了：

![](_resources/Raspberry_PI_4B/58c8455696ab97dc2b744d6cd3e21931_MD5.jpg)

编辑完成后，点击下面的 Apply Settings ，这时候 OpenClash 会重启，重启完之后，不出意外的话，这时候再让 switch 连上 OpenWrt，下载速度应该快到飞起。当然，前提是你的线路本身足够快: ) 。

OpenWrt 还有很多可以折腾的地方，可以换语言，换主题，装插件。这个我就不再细说了，大家可以去网上搜索教程。我反正是折腾了一圈，最后还用回了官方的主题。还是官方主题用起来顺手，特别是顶部菜单的设计，鼠标悬浮就能展开二级菜单，有些主题菜单在侧边栏，需要点击一级菜单才能展开二级菜单，有点麻烦。


# 扩容和备份

## 扩容

后来我才知道下面这种方法只对 rootfs 是 ext4 的固件有效，如果固件的 rootfs 是 squashfs + overlay ，请参考这篇文章进行扩容： [OpenWRT overlay 空间扩容](https://www.techkoala.net/openwrt_resize/) ，备用连接： [OpenWRT overlay 空间扩容 - WebArchive](https://web.archive.org/web/20250724024336/https://www.techkoala.net/openwrt_resize/)

不管你的 SD 卡有多大，烧完 OpenWrt 后，rootfs 分区也只剩下几十 M 的空间，折腾的时候极有可能会遇到磁盘空间不足的情况。因此我们先对 OpenWrt 做一下扩容，先把它扩到 1G。

当然，你也可以扩容到更大，但因为后面要对分区进行备份，分区太大备份要花较长时间，建议折腾完后再扩容到最大。

我用的分区工具是 Linux 平台的 Gparted，Windows 的傲梅分区助手应该也行。打开 Gparted，右上角选择 SD 卡，然后右键 rootfs 分区调出上下文菜单，点击 调整大小/移动 。

![](_resources/Raspberry_PI_4B/10b45be1dcbaa5cd77b9ccdbca28de51_MD5.jpg)

输入分区的新大小（也可以用上面的滑块来调整），然后点击 调整大小/移动 ，最后点击 **✔** 提交。

![](_resources/Raspberry_PI_4B/44dfd9758eba27e3833f86936d9cca5e_MD5.jpg)

OK，分区现在有 1G 了，你可以尽情去折腾了。

## 备份

折腾的过程中软路由可能会变砖，因此我们要养成一个好习惯，经常做下备份，留个检查点，不至于前功尽弃。其实 OpenWrt 自己就有备份的功能，但它备份的是 /etc 目录，假如某个软件的配置文件存在其他地方，恢复的时候就可能出现一些奇奇怪怪的问题。更稳妥的备份方法当然是把分区给备份下来，扩容后分区的总大小也只有 1G 左右，完整备份也不费事。我这里用来备份的工具是 Windows 平台的 Win32DiskImager，Mac 平台和 Linux 平台应该也有类似的工具，大家可以去找一下。

我们先创建一个名为 openwrt\_backup.img 的空文件，然后打开 Win32DiskImager，映像文件选择我们刚创建的 openwrt\_backup.img，设备选择 SD 卡对应的盘符，勾选 仅读取已分配分区 ，然后点击 读取 ，接着 SD 卡中的内容就会读入到 openwrt\_back.img 中了。备份过程中可能会出现 “剩余空间错误” 的弹窗，直接忽略。

![](_resources/Raspberry_PI_4B/1181d0652ad0c041d66db7e2ef3d73a8_MD5.jpg)

恢复的方法和前面烧录镜像的方法一样。
其实 Win32DiskImager 也能实现烧录，步骤和上面一样，只是把 读取 改成 写入 。
