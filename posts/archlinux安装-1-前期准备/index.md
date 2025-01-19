# Archlinux安装-1-前期准备


Archlinux 的安装网上是有详细的教程，但不算特别全面，因而本合集在此进行汇总。

本文只讲解了安装 Archlinux 的前期准备。
&lt;!--more--&gt;

## 前言

安装 Linux 尤其是 Archlinux 特别需要能够查找资料的能力，但能搜到这篇博客的想必资料查找能力已然过关。本文也只是简要说明，若有详细需求可参阅参考资料。

&gt; [!NOTE]
&gt; 笔者没有多余的 ARM 架构的手机/电脑，无法实践，因而 ARM 方面会有不全、错漏

&gt; [!NOTE]
&gt; 若有问题可在 discussion 处留言或在博客本体（非挂载在github上的静态网站）处提 issues/PR

## 安装介质种类

### 镜像

从 ArchIso 中安装

适用于U盘插入已完整安装的主板上进行安装。

#### 来源

&gt; [!NOTE]
&gt; `archiso` 需在 Archlinux 环境下使用

##### X86-64

 - [Archlinux Downloads](https://archlinux.org/download)
 - [阿里云镜像站](https://mirrors.aliyun.com/archlinux/iso/latest/)
 - [中国科技技术大学开源镜像站](https://mirrors.ustc.edu.cn/archlinux/iso/latest/)
 - [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/)
 - 由 [archiso](https://wiki.archlinux.org/title/Archiso) 手动构建镜像，可自定义程度高
 - [arCNiso](https://github.com/clsty/arCNiso/releases/latest)，对简中用户安装进行加强，对设备需求低，可仅触屏下完成安装(能但困难)。
 - [Archboot](https://release.archboot.com/x86_64/latest/iso/)

##### ARM
 
 - [Archboot](https://release.archboot.com/aarch64/latest/iso/)
 - 由 [archiso](https://wiki.archlinux.org/title/Archiso) 手动构建镜像，可自定义程度高

#### 方式

在下载地址中有许多种文件，在此简述文件类别及作用。由文件后缀列举：

- .iso，镜像文件
- .iso.sig 镜像文件签名，可用GPG验证，在此不做展开
- .torrent 镜像文件磁力链接，可用专用BT工具下载，在此不做展开
- .txt 校验文件
- .tar.gz 归档压缩包，里面含有 Archlinux 系统，将之分门别类拷贝到存储中即可完成安装

### 引导程序

从 Bootstrap 安装 Archlinux

适用于直接将系统安装到存储中。

#### 来源

##### X86-64

 - [Archlinux Downloads](https://archlinux.org/download)
 - [阿里云镜像站](https://mirrors.aliyun.com/archlinux/iso/latest/)
 - [中国科技技术大学开源镜像站](https://mirrors.ustc.edu.cn/archlinux/iso/latest/)
 - [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/)

##### ARM

 - [archlinuxarm-bootstrap](https://github.com/biggestT/archlinuxarm-bootstrap)
 - [Archlinux ARM](https://archlinuxarm.org/about/downloads)

#### 方式

在下载地址中有许多种文件，在此简述文件类别及作用。由文件后缀列举：

 - .tar.zst zst压缩格式的打包/归档文件夹，如何解压在此不做展开
 - .tar.zst.sig zst压缩格式的打包/归档文件夹的签名，可用GPG验证，在此不做展开
 - .tar.gz 归档压缩包，如何解压在此不做展开

### 网络

从服务器中安装 Archlinux

适用于统一对多台电脑进行安装（不过适用范围极窄，有这种需求的根本不会去装 Archlinux 这种滚动发行版，因而在本文中仅是列举，不做讲解）。

#### 来源

- [Arch Linux Netboot](https://archlinux.org/releng/netboot)

#### 方式

- iPXE

## 准备安装介质

### 镜像

将 `.iso` 的镜像文件刻录到U盘中

&gt; [!WARNING]
&gt; U盘刻录必定会将其上数据全部删除，一定一定要做好数据备份

&gt; [!NOTE]
&gt; 可使用 Ventoy 镜像实现一次刻录，即无须因 .iso 文件不同而多次刻录 U 盘，详情见主页 [Ventoy](https://www.ventoy.net/cn/)

&gt; [!NOTE]
&gt; Ventoy 适用于 x86 Legacy BIOS、IA32 UEFI、x86_64 UEFI、ARM64 UEFI 和 MIPS64EL UEFI 启动方式。

#### Linux

##### 命令行

```shell
sudo dd bs=4M if=/path/to/file.iso of=/dev/sdX status=progress oflag=sync
```
其中：
 - `/path/to/file.iso` 为所下载的镜像文件
 - `/dev/sdX` 为你要刻录的U盘
 - `4M` 为读写块大小，一般不用更改

##### 图形界面

使用软件如下，不再叙述：

 - `gnome-multi-writer`

#### Windows

操作方法见微软官方文档 [从闪存驱动器安装 Windows](https://learn.microsoft.com/zh-cn/windows-hardware/manufacture/desktop/install-windows-from-a-usb-flash-drive?view=windows-11)

#### MacOS

操作方法见苹果官方文档 [在 Mac 上使用“磁盘工具”创建磁盘映像](https://support.apple.com/zh-cn/guide/disk-utility/dskutl11888/mac)

#### 其它系统

参见官方文档。

### 引导程序

#### Archlinux

使用 `arch-install-scripts` 软件包进行安装

&gt; [!NOTE]
&gt; 其实 Archlinux 的镜像安装就是在U盘上安装一个带有 `arch-install-script` 的 Archlinux 系统。

&gt; [!NOTE]
&gt; X86-64 与 ARM 的区别就在于其最后安装的软件包架构的不同，一个例子：交叉编译

#### Linux

使用引导文件压缩包，在某一目录（推荐`/tmp`）解压后使用 `chroot` 命令进入 `Archlinux` 环境。然后操作就与 `Archlinux` 相同了（通常这类引导包自带了`arch-install-scripts`）

#### MacOS

大概与Linux的操作相同？也可能因为兼容问题而不行

- [ ] 用虚拟机去测试

#### Windows

使用 WSL2 运行 Linux 虚拟机进行操作，之后同 Linux

## 离线安装

几乎所有镜像都需要在网络环境下进行安装，除了`Archboot` 可以离线。
为了使其余镜像也可离线安装，需提前准备要安装的软件包。

### 软件包

 - `base` 基础软件包，包含基础的命令
 - `base-devel` 基础构建工具
 - `linux` 系统本体
 - `linux-firmware` 固件，大部分设备都有
 - `sof-firmware` 声卡固件（正常是无须安装的，但有些固件比较偏门）

## 额外内核模块

### ZFS

由于法律问题，ZFS文件系统并未内置于内核中。

#### 镜像

要在镜像中添加，可使用 [archiso-zfs](https://github.com/eoli3n/archiso-zfs) 脚本。

或使用 `archiso` 构建带有该内核模块的镜像。

或直接下载该镜像 [archiso-zfs](https://github.com/x0rzavi/archiso-zfs/releases/latest)。

#### 引导程序

需在进行操作的 Linux 系统上安装该模块，安装方法自行查阅。

### APFS

#### 镜像

使用 `archiso` 构建带有该内核模块的镜像。

或直接下载该镜像 [archiso-apfs](https://github.com/Integral-Tech/archiso-apfs/releases/latest)。

#### 引导程序

同 ZFS 引导程序

## 参考资料

[1] [Nakano Miku，ice-kylin，wuhang2003，等.archlinux 简明指南[EB/OL].GitHub.2024-10-09.https://arch.icekylin.online](https://arch.icekylin.online)  
[2] [Installation guide[DB/OL].ArchlinuxWiki.2024-09-30.https://wiki.archlinux.org/title/Installation_guide](https://wiki.archlinux.org/title/Installation_guide)  
[3] [vconlln.Archlinux安装(超详细)[EB/OL].博客园.2023-01-23.https://www.cnblogs.com/vconlln/p/17065420.html](https://www.cnblogs.com/vconlln/p/17065420.html)  
[4] [Arch Linux安装教程 （以Legacy BIOS为例）[EB/OL].极客学习笔记.2022-07-03.https://geekdaxue.co/read/microsoftxiaoice@archlinux/ahdvik#54nlwi](https://geekdaxue.co/read/microsoftxiaoice@archlinux/ahdvik#54nlwi)  
[5] [Install Arch Linux from existing Linux[DB/OL].ArchlinuxWiki.2024-07-29.https://wiki.archlinux.org/title/Install_Arch_Linux_from_existing_Linux](https://wiki.archlinux.org/title/Install_Arch_Linux_from_existing_Linux)  
[6] [王帅.WSL2 安装 ArchLinux —— In The Arch Way[EB/OL].2023-03-13.https://www.devws.cn/posts/wsl2-archlinux](https://www.devws.cn/posts/wsl2-archlinux)  
[7] [Zero___________.iPXE 学习 部署Linux/Windows系统 支持IPv4/IPv6[EB/OL].CSDN.2024-02-01.https://blog.csdn.net/Zero___________/article/details/135866160](https://blog.csdn.net/Zero___________/article/details/135866160)  
[8] [DreamOneX.利用Bootstrap从已有的Linux发行版上安装ArchLinux[EB/OL].2022-07-21.https://blog.dreamonex.eu.org/p/2022/07/install-arch-from-linux/](https://blog.dreamonex.eu.org/p/2022/07/install-arch-from-linux/)


---

> 作者: Bendancom  
> URL: https://bendancom.github.io/posts/archlinux%E5%AE%89%E8%A3%85-1-%E5%89%8D%E6%9C%9F%E5%87%86%E5%A4%87/  

