# Archlinux-基础安装-1-前期准备


Archlinux 的安装网上是有详细的教程，但不算特别全面，因而本合集在此进行汇总。
本文只列举讲解了安装 Archlinux 的前期准备。
&lt;!--more--&gt;

## 前言

安装 Linux 尤其是 Archlinux 特别需要能够查找资料的能力，但能搜到这篇博客的想必资料查找能力已然过关。本文也只是简要说明，若有详细需求可参阅参考资料。

## 安装介质种类

### 镜像

从 ArchIso 中安装

适用于U盘插入已完整安装的主板上进行安装。

#### 来源

- [Archlinux Downloads](https://archlinux.org/download)
- [阿里云镜像站](https://mirrors.aliyun.com/archlinux/iso/latest/)
- [中国科技技术大学开源镜像站](https://mirrors.ustc.edu.cn/archlinux/iso/latest/)
- [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/)
- 由 [archiso](https://wiki.archlinux.org/title/Archiso) 手动构建镜像，可自定义程度高
- [arCNiso](https://github.com/clsty/arCNiso/releases/latest)，对简中用户安装进行加强，对设备需求低，可仅触屏下完成安装(能但困难)。

#### 方式

在下载地址中有许多种文件，在此简述文件类别及作用。由文件后缀列举：

- .iso，镜像文件
- .iso.sig 镜像文件签名，可用GPG验证，在此不做展开
- .torrent 镜像文件磁力链接，可用专用BT工具下载，在此不做展开
- .txt 校验文件

### 脚本

从 Bootstrap 安装 Archlinux

适用于已经有硬盘盒，直接将系统安装到硬盘中。

#### 地址

- [Archlinux Downloads](https://archlinux.org/download)
- [阿里云镜像站](https://mirrors.aliyun.com/archlinux/iso/latest/)
- [中国科技技术大学开源镜像站](https://mirrors.ustc.edu.cn/archlinux/iso/latest/)
- [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/)
- arch-install-script(pacman 中)

#### 方式

在下载地址中有许多种文件，在此简述文件类别及作用。由文件后缀列举：

- .tar.zst zst压缩格式的打包/归档文件夹，如何解压在此不做展开
- .tar.zst.sig zst压缩格式的打包/归档文件夹的签名，可用GPG验证，在此不做展开

### 网络

从服务器中安装 Archlinux

适用于统一对多台电脑进行安装（不过适用范围极窄，有这种需求的根本不会去装 Archlinux 这种滚动发行版，因而在本文中仅是列举，不做讲解）。

#### 地址

- [Arch Linux Netboot](https://archlinux.org/releng/netboot)

#### 方式

- iPXE

## 准备安装介质

### 镜像

将 `.iso` 的镜像文件刻录到U盘中

&gt; [!WARNING]
&gt; U盘的刻录必定会将其上数据全部删除，一定一定要做好数据备份

&gt; [!NOTE]
&gt; 可使用 Ventoy 镜像实现一次刻录，即无须因 .iso 文件不同而多次刻录 U 盘，详情见主页 [Ventoy](https://www.ventoy.net/cn/)

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

### 脚本

使用 `arch-install-scripts` 软件包进行安装

&gt; [!NOTE]
&gt; `arch-install-scripts` 基于 `pacman`，因而需要 Archlinux 环境，若要在其他 Linux 发行版上使用，可用 [`arch-bootstrap`](https://github.com/tokland/arch-bootstrap)，具体操作见项目仓库。

&gt; [!NOTE]
&gt; 其实 Archlinux 的镜像安装就是在U盘上安装一个带有 `arch-install-script` 的 Archlinux 系统。

## 参考资料

[1] [Nakano Miku，ice-kylin，wuhang2003，等.archlinux 简明指南[EB/OL].GitHub.2024-10-09.https://arch.icekylin.online](https://arch.icekylin.online)  
[2] [Installation guide[DB/OL].ArchlinuxWiki.2024-09-30.https://wiki.archlinux.org/title/Installation_guide](https://wiki.archlinux.org/title/Installation_guide)  
[3] [vconlln.Archlinux安装(超详细)[EB/OL].博客园.2023-01-23.https://www.cnblogs.com/vconlln/p/17065420.html](https://www.cnblogs.com/vconlln/p/17065420.html)  
[4] [Arch Linux安装教程 （以Legacy BIOS为例）[EB/OL].极客学习笔记.2022-07-03.https://geekdaxue.co/read/microsoftxiaoice@archlinux/ahdvik#54nlwi](https://geekdaxue.co/read/microsoftxiaoice@archlinux/ahdvik#54nlwi)  
[5] [Install Arch Linux from existing Linux[DB/OL].ArchlinuxWiki.2024-07-29.https://wiki.archlinux.org/title/Install_Arch_Linux_from_existing_Linux](https://wiki.archlinux.org/title/Install_Arch_Linux_from_existing_Linux)  
[6] [王帅.WSL2 安装 ArchLinux —— In The Arch Way[EB/OL].2023-03-13.https://www.devws.cn/posts/wsl2-archlinux](https://www.devws.cn/posts/wsl2-archlinux)  
[7] [Zero___________.iPXE 学习 部署Linux/Windows系统 支持IPv4/IPv6[EB/OL].CSDN.2024-02-01.https://blog.csdn.net/Zero___________/article/details/135866160](https://blog.csdn.net/Zero___________/article/details/135866160)


---

> 作者: Bendancom  
> URL: https://bendancom.github.io/posts/archlinux-%E5%9F%BA%E7%A1%80%E5%AE%89%E8%A3%85/  

