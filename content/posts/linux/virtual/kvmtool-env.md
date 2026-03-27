+++
date = '2024-11-18T13:27:19+08:00'
draft = false
title = 'KVM-tool虚拟化调试环境'
categories = ['linux', 'virtual']
+++

# KVM-tool 虚拟化调试搭建

## 搭建步骤

Linux 内核下载的是 ubuntu 上的 jammy

```shell
ssh-copy-id user@hostname免密.
https://git.launchpad.net/~ubuntu-kernel/ubuntu/+source/linux/+git/jammy
linux编译bzImage
```
下载对应的：

```shell
下载KVMtool:
https://github.com/kvmtool/kvmtool?tab=readme-ov-file
cd kvmtool && make
```

下载根文件系统

```shell
下载根文件系统：·
https://cloud-images.ubuntu.com/jammy/20240227/
jammy-server-cloudimg-amd64.tar.gz              2024-02-27 02:44  564M  File system image and Kernel packed

ubuntu cloud image 是ububtu 针对云出的系统，可以安装到openstack 上，但是默认没有密码，只能ssh 登录。
可能通过如下方法设置密码
1.安装guestfs
sudo apt install libguestfs-tools
2. 给镜像设置密码
virt-customize -a jammy-server-cloudimg-amd64.my.img --root-password password:123456

```

## **如何启动 kvmtool**

```·
sudo ./lkvm run --disk  jammy-server-cloudimg-amd64.img --kernel bzImage  --network virtio
```

登陆：

```
用户:root
密码：123456
```

## 如何使用 GDB调试内核

```Shell
This document explains how to debug a guests' kernel using KGDB.

1. Run the guest:
lkvm run -k bzImage -p "kgdboc=ttyS1 kgdbwait" --tty 1

这里是提示后面要使用
And see which PTY got assigned to ttyS1 (you'll see:
'  Info: Assigned terminal 1 to pty /dev/pts/X').

2. Run GDB on the host:
gdb bzImage

3. Connect to the guest (from within GDB):
target remote /dev/pty/X

4. Start debugging! (enter 'continue' to continue boot).
如何加载符号信息，因为bzImage是一个无符号的信息。
```

**内核编译调试：**

```shell
确认KGDB打开
grep CONFIG_DEBUG_INFO .config
CONFIG_DEBUG_INFO=y
1	Kernel hacking  --->
2    [*] Kernel debugging
3    Compile-time checks and compiler options  --->
4        [*] Compile the kernel with debug info
5        [*]   Provide GDB scripts for kernel debuggin
6
7
8	Processor type and features ---->
9    [] Randomize the address of the kernel image (KASLR)

make vmlinux -j 80
make bzImage -j 80
```

​ 编译完成后，会在当前目录下生成 vmlinux，这个在 gdb 的时候需要加载，用于读取 symbol 符号信息，包含了所有调试信息，所以比较大。

压缩后的镜像文件为 bzImage， 在 arch/x86/boot/目录下。

> 几种 linux 内核文件的区别:
>
> vmlinux 编译出来的最原始的内核文件，未压缩。
>
> zImage 是 vmlinux 经过 gzip 压缩后的文件。
>
> bzImage bz 表示“big zImage”，不是用 bzip2 压缩的。两者的不同之处在于，zImage 解压缩内核到低端内存(第一个 640K)。
>
> bzImage 解压缩内核到高端内 存(1M 以上)。如果内核比较小，那么采用 zImage 或 bzImage 都行，如果比较大应该用 bzImage。
>
> uImage U-boot 专用的映像文件，它是在 zImage 之前加上一个长度为 0x40 的 tag。
>
> vmlinuz 是 bzImage/zImage 文件的拷贝或指向 bzImage/zImage 的链接。
>
> initrd 是“initial ramdisk”的简写。一般被用来临时的引导硬件到实际内核 vmlinuz 能够接管并继续引导的状态。

```shell
gdb file vmlinux //添加符号表.
source vmlinux-gdb.py
```
