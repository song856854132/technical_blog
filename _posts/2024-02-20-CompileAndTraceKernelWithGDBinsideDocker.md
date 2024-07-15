---
title: Build & Trace Linux Kernel for Qemu in Docker
date: 2024-02-20
categories: [Linux, Kernel]
tags: [KVM]
img_path: /assets/img/kernel/
---

## Why Need to Trace Kernel

When it comes to learning Linux or system programming in C, one of the most effective methods is tracing kernel code. Despite the complexity of the kernel and its constant evolution, debugging can still yield valuable insights. For me, a student possess computer science degree, know how linux operates, like file system architecture, process and signal handling, or even virtual memory mechanism. However, do I? However, do I truly understand the core of this system? Is my understanding based solely on theoretical concepts taught by my professors?  What does it actually look like by face to face inspection.

After long search for the approaching, I want to thank mgalgs for [his post](https://mgalgs.io/2021/03/23/how-to-build-a-custom-linux-kernel-for-qemu-using-docker.html), he wrapped up whole complex process and leave us such a short cut. Great article.

## Why use Docker

Docker is widely recognized for its lightweight deployment capabilities. With a Dockerfile, it becomes straightforward to create a ready-to-go environment suitable for compiling kernel source code. 

```
$ cat Dockerfile
FROM debian:10.8-slim
RUN apt update
RUN apt install -y bc bison build-essential cpio flex libelf-dev libncurses-dev libssl-dev vim-tiny

$ docker run -it -v ./build-linux:/teeny teeny-linux-builder
```


## Prerequisites
- busybox source code:  https://busybox.net/downloads/busybox-1.32.1.tar.bz2
- kernel source code: https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.11.7.tar.xz
- docker installed

```shell
tree -L 2 build-linux
build-linux/
├── busybox-1.32.1
│   └── TL;DR
├── docker
│   └── Dockerfile
└── linux-5.11.7
    └── TL;DR
```

## Prepare the User Layer File System

BusyBox, as a userspace file system, mostly be seen in resource-constrained environments such as embedded systems, make it an ideal root filesystem for us if we only just want to trace kernel.

A initramfs is a temporary filesystem used by the kernel to initialize essential drivers or modules to mount the root filesystem and then complete the boot sequence.
It usually contains a gzipped "cpio" format archive. After extracting, the kernel checks to see if rootfs contains a file "init", and if so it executes it as PID 1. If rootfs doesn't contain an init program, the kernel will fall through to the older code to locate and mount a root partition.

```shell
// inside container
# cd /teeny/busybox-1.32.1
# mkdir -pv ../obj/busybox-x86
# make O=../obj/busybox-x86 defconfig
# make O=../obj/busybox-x86 menuconfig -> tick [*] Build static binary (no shared libs)
# cd ../obj/busybox-x86
# make -j$(nproc)
# make install
# mkdir -pv /teeny/initramfs/x86-busybox && cd /teeny/initramfs/x86-busybox
# mkdir -pv {bin,sbin,etc,proc,sys,usr/{bin,sbin}}
# cp -a /teeny/obj/busybox-x86/_install/* .
# vi /teeny/initramfs/x86-busybox/init
# chmod +x /teeny/initramfs/x86-busybox/init
# cd /teeny/initramfs/x86-busybox
# find . -print0 | cpio --null -ov --format=newc | gzip -9 > /teeny/obj/initramfs-busybox-x86.cpio.gz
```


## Prepare the Kernel Image

```shell
# cd /teeny/linux-5.11.7
# make O=../obj/linux-x86-basic x86_64_defconfig
# make O=../obj/linux-x86-basic menudefconfig -> tick [*] Provide GDB scripts for kernel debugging & [*] Compile the kernel with debug info
# make O=../obj/linux-x86-basic -j$(nproc)
# tree -L 2 /teeny/
/teeny/
├── busybox-1.32.1
│   └── TL;DR
├── docker
│   └── Dockerfile
├── initramfs
│   └── x86-busybox
├── linux-5.11.7
│   └── TL;DR
└── obj
    ├── busybox-x86
    ├── initramfs-busybox-x86.cpio.gz
    └── linux-x86-basic
```
Select `Provide GDB scripts for kernel debugging` to enable kgdb features, which make  hypervisors like QEMU allow to debug the Linux kernel and its modules during runtime using gdb.

![Img](kernel_menuconfig.png)

## Launch VM by QEMU and Start Tracing by GDB

In this case, we use qemu to launch VM. You can also launch with VMware or Virtualbox if you want. However, here are few things worth explaining. First, normally we use bootloader inside disk image to start booting process, but this time we use QEMU itself to load the kernel with `-kernel` option. Since KASLR was default on
after kernel v4.1.2, which will prevent us from hitting breakpoint as we cannot detect the base address of main function, or PIE. `nokaslr` must be appended to turn default kaslr feature off. `-S` option stands for freeze CPU at startup.

```
// on host machine
$ qemu-system-x86_64 -kernel obj/linux-x86-basic/arch/x86_64/boot/bzImage -initrd obj/initramfs-busybox-x86.cpio.gz -nographic -s -S -append "console=ttyS0 nokaslr"
```

```shell
// inside container
# gdb /teeny/obj-5.11.7/linux-x86-basic/vmlinux
GNU gdb (Debian 8.2.1-2+b3) 8.2.1
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from /teeny/obj-5.11.7/linux-x86-basic/vmlinux...done.
warning: File "/teeny/linux-5.11.7/scripts/gdb/vmlinux-gdb.py" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load".

To enable execution of this file add
        add-auto-load-safe-path /teeny/linux-5.11.7/scripts/gdb/vmlinux-gdb.py
line to your configuration file "/root/.gdbinit".
To completely disable this security protection add
        set auto-load safe-path /
line to your configuration file "/root/.gdbinit".
For more information about this security protection see the
"Auto-loading safe path" section in the GDB manual.  E.g., run from the shell:
        info "(gdb)Auto-loading safe path"
(gdb) target remote 172.17.0.1:1234
Remote debugging using 172.17.0.1:1234
0x000000000000fff0 in exception_stacks ()
(gdb) hb start_kernel
Hardware assisted breakpoint 1 at 0xffffffff82b2aad5: file /teeny/linux-5.11.7/init/main.c, line 850.
(gdb) c
Continuing.

Breakpoint 1, start_kernel () at /teeny/linux-5.11.7/init/main.c:850
850     {
(gdb) tui enable -> this can also be turned on as a gdb option
```

Because we are inside Docker container, our remote target will be 172.17.0.1, or host machine of container. 

![Img](gdb_tui.png)