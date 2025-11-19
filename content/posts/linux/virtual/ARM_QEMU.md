+++
date = '2024-11-18T13:27:19+08:00'
draft = false
title = 'KVM-QEMU虚拟化调试环境'
+++

# 编译相关

## QEMU 编译

一定要在官网上下周一个 tar 包，三方库才是完整的包含的。

```shell
wget https://download.qemu.org/qemu-8.2.6.tar.xz
tar xvJf qemu-8.2.6
cd qemu-8.2.6
编译所有平台：
./configure
make
```

QEMU 编译选项：ARM 平台：

```shell
./configure --target-list=aarch64-softmmu --enable-debug
make -j$(nproc)
make install
```

## Linux 编译内核

设置交叉编译环境：

```shell
source toolchain.sh
export PATH=$PATH:~/Documents/toolchain/gcc-arm-11.2-2022.02-x86_64-aarch64-none-linux-gnu/bin
export CROSS_COMPILE=aarch64-none-linux-gnu-
export ARCH=arm64
```

编译 linux:

```shell
生产.config
make ARCH=arm64 defconfig
make ARCH=arm64 menuconfig
编译：
make ARCH=arm64 modules dtbs Image -j4
make ARCH=arm64 deb-pkg -j8
make ARCH=arm64 modules -j10
make ARCH=arm64 Image -j4
clean:
make ARCH=arm64 mrproper
```

## ubuntu 文件系统

- 下载镜像：要下 File system image 文件系统。
- 下载 EFI 的就是有 EFI 有自动引导分区的。

<https://cloud-images.ubuntu.com/jammy/20240810/>

```shell
jammy-server-cloudimg-arm64.tar.gz                                           2024-08-10 03:00  534M  File system image and Kernel packed

可能通过如下方法设置密码
1.安装guestfs
sudo apt install libguestfs-tools
2. 给镜像设置密码
virt-customize -a jammy-server-cloudimg-amd64.img --root-password password:123456
3、扩展分区大小：
qemu-img resize jammy-server-cloudimg-arm64.img 10G
4、启动虚拟机
lsblk
之后可以考到分区已经扩展了。
mtdblock0  31:0    0  128M  0 disk
vda       254:0    0    5G  0 disk /
5、重新分配分区大小
sudo parted /dev/vda
(parted) print               # 检查分区
(parted) resizepart 1 100%   # 调整分区大小到磁盘末尾
(parted) quit
6、扩展文件系统以利用新增空间：
sudo resize2fs /dev/vda1
```

这个只能用来调试内核，不时候用来调试驱动，驱动必须自己做。

启动命令：<mark>网络没有配置系统会报错！！！</mark>原因是 cloud-init

```shell
  1 qemu-system-aarch64 -machine virt -cpu cortex-a57\
  2     -smp 4\
  3     -m 4096M\
  4     -name ubuntu \
  5     -kernel ./kernel/Image\
  6     --append "noinitrd root=/dev/vda rw console=ttyAMA0 loglevel=8"  \
  7     -nographic \
  8     -device virtio-serial \
  9     -drive file=jammy-server-cloudimg-arm64.img,if=virtio \
 10     -netdev tap,id=net0,ifname=tap0,script=no,downscript=no \
 11     -device e1000,netdev=net0 \
 12     #-device virtio-net-pci,netdev=net0 \
 13 #-S -s
```

**启动错误：**

**修改自己编译的 QEMU 无法加载镜像：**

```
efi-virtio.rom 到标准路径
sudo cp /usr/local/share/qemu/efi-virtio.rom /usr/share/qemu/
这样，QEMU 就能够自动找到 efi-virtio.rom 文件，无需每次指定完整路径。
```

## QEMU 中提取设备树

```shell
方法 1:
ls /proc/device-tree
保存设备树为 DTS 文件： 使用 dtc 命令将设备树目录导出为 DTS 文件：
dtc -I fs -O dts -o extracted.dts /proc/device-tree
```

# 自己创建系统

virt-manager 安装最后进入的是命令行界面，无图形安装。

- 安装的时候要选择架构 aarch64 virt
- UEFI 选择:

```shell
UEFI aarch64: /usr/share/AAVMF/AAVMF_CODE.fd
/usr/bin/qemu-system-aarch64
virt-6.2
```

```shell
下载iso:
wget https://cdimage.ubuntu.com/releases/22.04.5/release/ubuntu-22.04.5-live-server-arm64.iso
qemu-img create -f raw ubuntu-arm64.img 15G
```

参考以下文章：

<https://blog.csdn.net/weixin_40837318/article/details/136737347>

# 启动调试

## UEFI 形式启动

```
sudo bash -c "
 ./qemu-system-aarch64 \
 -enable-kvm \
 -m 2048 \
 -cpu host \
 -machine virt,gic-version=2 \
 -smp 4,sockets=4,cores=1,threads=1 \
 -nographic \
 -drive if=pflash,format=raw,file=efi.img,readonly=on \
 -drive if=pflash,format=raw,file=varstore.img \
 -drive if=none,file=jammy-server-cloudimg-arm64.img,id=hd0 \
 -device virtio-blk-pci,drive=hd0  \
 -netdev tap,id=net0,ifname=tap0,script=no,downscript=no \
 -device virtio-net-pci,netdev=net0,romfile="" \
 -D qemu_trace.log
```

在 QEMU 启动的时候有一个写 ACPI 表的动作！！

## kernel 启动

```
./qemu-system-aarch64 \
        -machine virt,gic-version=2 \
        -enable-kvm \
        -cpu host \
        -m 2048 \
        -smp 4,sockets=4,cores=1,threads=1 \
        -kernel ./kernel/Image \
        --append "noinitrd root=/dev/vda rw console=ttyS1 nokaslr loglevel=7"  \
        -name ubuntu \
        -drive file=jammy-server-cloudimg-arm64.img,format=raw,if=virtio \
        -nographic \
        -netdev tap,id=net0,ifname=tap0,script=no,downscript=no \
         -device virtio-net-pci,netdev=net0,romfile="" \
        -L /usr/local/share/qemu \
        -S -s

#       -serial file:serial.log \
#       -serial pty \
#       -S -s 启动调试
#       -machine virt,gic-version=2,dumpdtb=sprd.dtb\
#       --append "noinitrd root=/dev/vda rw console=ttyAMA0 console=ttyS1 loglevel=8"  \
#       -d trace:pl011* -D /var/log/qemulogfile.log \
#       sudo -s gdb --args ./qemu-system-aarch64 \
```

## **QEMU 调试 linux 内核**

```shell
Thread 1 hit Breakpoint 1, 0xffff8000097b0c1c in start_kernel () at init/main.c:935
935     init/main.c: No such file or directory.
(gdb) info source
Current source file is init/main.c
Compilation directory is /home/ben7zy/Music/kernel/jammy
Source language is c.
Producer is GNU C89 11.2.1 20220111 -mlittle-endian -mgeneral-regs-only -mabi=lp64 -mbranch-protection=pac-ret+leaf+bti -mstack-protector-guard=sysreg -mstack-protector-guard-reg=sp_el0 -mstack-protector-guard-offset=1112 -g -g -O2 -O1 -std=gnu90 -fno-strict-aliasing -fno-common -fshort-wchar -fno-PIE -fno-asynchronous-unwind-tables -fno-unwind-tables -fno-delete-null-pointer-checks -fno-allow-store-data-races -fstack-protector-strong -fno-omit-frame-pointer -fno-optimize-sibling-calls -fno-stack-clash-protection -fno-var-tracking -femit-struct-debug-baseonly -fno-strict-overflow -fstack-check=no -fconserve-stack -fno-function-sections -fno-data-sections.
Compiled with DWARF 5 debugging format.
Does not include preprocessor macro info.
(gdb) set substitute-path /home/ben7zy/Music/kernel/jammy /home/raspberry/Music/jammy
(gdb) target remote :1234

```

**断点要使用 hb 才能断住。**

**使用--tui 能够跟踪到代码一行一行的运行！！！**

![gdb-tui-debug](D:\benqiang.yu\Note\虚拟化\QEMU\gdb-tui-debug.png)
