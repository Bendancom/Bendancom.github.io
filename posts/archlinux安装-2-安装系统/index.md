# Archlinux安装-2-安装系统


Archlinux 的安装网上是有详细的教程，但不算特别全面，因而本合集在此进行汇总。

本文只讲解了 Archlinux 系统的安装。
&lt;!--more--&gt;

## 前言

Archlinux安装对于镜像或引导程序而言，其主要步骤并无区别。

## 存储介质分区

通常而言就是硬盘，但还有一些其他分类的（TF卡、SD卡等），所以在文中使用存储介质一词。

### 创建分区

在这一步时应去查阅主板的引导程序（Bootloader）类型及方式，以确定所需创建在存储介质上的启动分区的文件系统类型与格式。

不同设备的引导程序：
 - Legecy BIOS —— MBR分区格式存储介质
 - UEFI —— GPT 分区格式存储介质
 - U-Boot —— 一些嵌入式开发板下
 - LLB —— Apple Sillon 下的苹果设备

&gt; [!NOTE]
&gt; MacOS 仍保有 UEFI 启动方式

#### 必须

 - ROOT 分区
 - 启动分区，至少需要一个

&gt; [!NOTE]
&gt; 多系统已有一个启动分区，所以可以不创建

#### 可选

 - BOOT 分区，用于存放 Linux启动文件，通常是否创建由以下几个因素决定：
    - 根目录需要加密或使用复杂的文件系统（如Btrfs、zfs）
    - 系统管理与维护性（与 EFI/BIOS 隔离）
    - 硬件或固件限制
 - SWAP 分区，用于虚拟内存，通常由以下几个因素决定：
    - 物理内存不够大
    - 是否设置休眠
 - HOME 分区，用于用户存储，通常由以下几个因素决定：
    - 系统管理与维护性（与根目录隔离）

&gt; [!NOTE]
&gt; 可选的分区不只有这些，自行根据自身需求去创建

### 格式化分区

 - SWAP 分区 —— 以 swap 的格式 格式化
 - 启动分区 —— FAT32

其余分区无特殊文件系统要求。

&gt; [!NOTE]
&gt; 但得要内核支持该文件系统

### 挂载分区

`/mnt` 为临时挂载目录

 - ROOT 分区 —— `/mnt`
 - EFI 分区 —— `/mnt/boot` 或 `/mnt/efi` == `/mnt/boot/efi`
 - HOME 分区 —— `/mnt/home`
 - SWAP 分区 —— swap 挂载

## 安装 linux 及软件包

进入 Archlinux 环境中。
使用 `pacstrap` 向挂载的分区安装(位于 `arch-install-scripts` 软件包内)

或将相应架构的 Archlinux 系统压缩包解压写入相对应存储介质分区中

### 必须

 - `base`
 - `base-devel`
 - `linux`
 - `linux-firmware`

### 可选

 - `linux-header`，linux系统头文件，用于 dkms 的编译
 - `sof-firmware`，声卡固件
 - `btrfs-progs`，BtrFS 文件系统工具
 - `apfs-progs`，APFS 文件系统工具
 - `linux-apfs-rw` APFS 文件系统支持（实验性写入）
 - `zfs-dkms`，ZFS 文件系统支持
 - `network-manager`，网络管理工具
 - `vim`/`neovim` 等系列终端编辑器
 - `zsh`/`bash` 等系列 shell
 - `tldr`/`man` 帮助手册

&gt; [!NOTE]
&gt; 可选软件包只是列举一些常用软件，并不一定符合安装者的习惯，自行根据需求进行增减。

### 引导程序

 - `efibootmgr`，管理一级引导程序（UEFI）启动序列，生成 .efi 文件用于 UEFI 启动
 - `grub`/`rEFInd`/`OpneCore`/`Clover` [等](https://cn.linux-terminal.com/?p=1040)，二级引导程序，管理多系统的选择

&gt; [!NOTE]
&gt; 在前面笔者标明 BOOT 分区为可选，是因为有统一内核映像(UKI)，可以将内核打包为一个 .efi 文件用于启动，所以只要内核支持的文件系统都能启动

### 微码

根据 CPU 芯片制造商安装

## 生成 fstab

```shell
genfstab -U /mnt &gt; /mnt/etc/fstab
```
具体参数用途查阅手册
推荐用存储介质 UUID 生成（因为其唯一对应）

## 杂项

 - 同步时间
 - 设置语言
 - 设置主机名与时区

## 参考资料

[1] [Nakano Miku，ice-kylin，wuhang2003，等.archlinux 简明指南[EB/OL].GitHub.2024-10-09.https://arch.icekylin.online](https://arch.icekylin.online)  
[2] [Installation guide[DB/OL].ArchlinuxWiki.2024-09-30.https://wiki.archlinux.org/title/Installation_guide](https://wiki.archlinux.org/title/Installation_guide)  
[3] [vconlln.Archlinux安装(超详细)[EB/OL].博客园.2023-01-23.https://www.cnblogs.com/vconlln/p/17065420.html](https://www.cnblogs.com/vconlln/p/17065420.html)  
[4] [Arch Linux安装教程 （以Legacy BIOS为例）[EB/OL].极客学习笔记.2022-07-03.https://geekdaxue.co/read/microsoftxiaoice@archlinux/ahdvik#54nlwi](https://geekdaxue.co/read/microsoftxiaoice@archlinux/ahdvik#54nlwi)  
[5] [Install Arch Linux from existing Linux[DB/OL].ArchlinuxWiki.2024-07-29.https://wiki.archlinux.org/title/Install_Arch_Linux_from_existing_Linux](https://wiki.archlinux.org/title/Install_Arch_Linux_from_existing_Linux)  
[6] [王帅.WSL2 安装 ArchLinux —— In The Arch Way[EB/OL].2023-03-13.https://www.devws.cn/posts/wsl2-archlinux](https://www.devws.cn/posts/wsl2-archlinux)  
[7] [Zero___________.iPXE 学习 部署Linux/Windows系统 支持IPv4/IPv6[EB/OL].CSDN.2024-02-01.https://blog.csdn.net/Zero___________/article/details/135866160](https://blog.csdn.net/Zero___________/article/details/135866160)
[8] [DreamOneX.利用Bootstrap从已有的Linux发行版上安装ArchLinux[EB/OL].2022-07-21.https://blog.dreamonex.eu.org/p/2022/07/install-arch-from-linux/#%E5%87%86%E5%A4%87resolvconf](https://blog.dreamonex.eu.org/p/2022/07/install-arch-from-linux/#%E5%87%86%E5%A4%87resolvconf)
[9] [Dontla.操作系统分区的时候/boot和/boot/efi有什么区别？为什么有时需要/boot分区有时不需要？[EB/OL].2025-01-08.https://blog.csdn.net/Dontla/article/details/139780558](https://blog.csdn.net/Dontla/article/details/139780558)
[10] [佚名.Ubuntu系统深度解析：swap空间与休眠模式的优化与配置[EB/OL].2024-12-29.https://www.oryoy.com/news/ubuntu-xi-tong-shen-du-jie-xi-swap-kong-jian-yu-xiu-mian-mo-shi-de-you-hua-yu-pei-zhi.html](https://www.oryoy.com/news/ubuntu-xi-tong-shen-du-jie-xi-swap-kong-jian-yu-xiu-mian-mo-shi-de-you-hua-yu-pei-zhi.html)
[11] [UEFI Specification[EB/OL].2024-11-21.https://uefi.org/sites/default/files/resources/UEFI_Spec_Final_2.11.pdf](https://uefi.org/sites/default/files/resources/UEFI_Spec_Final_2.11.pdf)
[12] [适用于家庭和嵌入式系统的 15 个最佳 Linux 引导加载程序[EB/OL].https://cn.linux-terminal.com/?p=1040](https://cn.linux-terminal.com/?p=1040)


---

> 作者: Bendancom  
> URL: https://bendancom.github.io/posts/archlinux%E5%AE%89%E8%A3%85-2-%E5%AE%89%E8%A3%85%E7%B3%BB%E7%BB%9F/  

