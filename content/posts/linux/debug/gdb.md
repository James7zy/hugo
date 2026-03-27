+++
date = '2024-11-18T13:27:19+08:00'
draft = false
title = 'gdb内核调试'
categories = ['linux', 'debug']
+++

# 使用GDB调试内核

```shell
gdb -tui 
```

重点参考：

- https://www.skywind.me/blog/archives/2036
- https://blog.csdn.net/gatieme/article/details/63254211

**设置源码目录调试：**

```shell
(gdb) set architecture i386:
intel         x64-32        x64-32:intel  x86-64        x86-64:intel
(gdb) set architecture i386:x86-64
The target architecture is set to "i386:x86-64".
(gdb) set substitute-path /data/jenkins/workspace/HY11CN/hy11_android13_oktb_releasebuild/android13/android-13.0.0_r24/aosp/celadon_kernel/ /data3/luofengxin/android/an
droid-13.0.0_r24/aosp/celadon_kernel
(gdb) b start_kernel
Breakpoint 1 at 0xffffffff8496fada: file /data/jenkins/workspace/HY11CN/hy11_android13_oktb_releasebuild/android13/android-13.0.0_r24/aosp/celadon_kernel/init/main.c, l
ine 936.
(gdb) list *start_xmit+0x7f
0xffffffff82108d7f is in start_xmit (/data/jenkins/workspace/HY11CN/hy11_android13_oktb_releasebuild/android13/android-13.0.0_r24/aosp/celadon_kernel/drivers/net/virtio
_net.c:1828).
```



**QEMU调试linux内核**

```
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

```



```shell
调试工具：
strace
addr2line
objdump
gdb
crash工具
```

