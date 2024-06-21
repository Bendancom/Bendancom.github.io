# 如何建立一个Java版Minecraft服务器


在长久以来的单人Minecraft后，即使其游玩内容丰富，但仍想找到志同道合的朋友一起游玩去创造美好回忆。而中国版Minecraft可以说是限制颇大了，因而国际版Java服务器可以说是联机的首选了。

&lt;!--more--&gt;

## 前期准备

### 设备准备

选其中一个即可：

- 一台联网电脑，性能满足游戏需求，系统优先级为`Linux &gt; Windows = Mac`。
- 云服务器

### 环境准备

- `Java`

### 知识准备

- 基本电脑知识
- 基础`Linux`知识(`Linux`系统)
- 一定英语水平，便于解决疑难问题

## 服务器根目录

推荐创建在带快照的文件系统内。

### 个人电脑

#### `Linux`系统

`Btrfs`或`ZFS`文件系统

- 分区快照
  
`Ext4`或`XFS`

- 无快照

#### `Windows`系统

`NTFS`文件系统

- 全区快照

#### `Mac OS`系统

`APFS`文件系统

- 全区快照

### 云服务器

- 云快照

## 环境准备

自行查找Java的安装。

## 服务器安装

### 原版

[版本](https://launchermeta.mojang.com/mc/game/version_manifest.json)
在其中挑选所需的服务器版本，如有模组需注意二者`Minecraft`版本的匹配。

```json
{
    &#34;id&#34;: &#34;1.20.1&#34;,
    &#34;type&#34;: &#34;release&#34;,
    &#34;url&#34;: &#34;https://piston-meta.mojang.com/v1/packages/30e78c499d4df02aab34a811e290c1805f925105/1.20.1.json&#34;,
    &#34;time&#34;: &#34;2024-04-03T06:24:18&#43;00:00&#34;,
    &#34;releaseTime&#34;: &#34;2023-06-12T13:25:51&#43;00:00&#34;
},
```

从中获取版本号对应的`URL`并打开此`json`文件，在其中找到`downloads`元素(第165行)

```json
&#34;downloads&#34;: {
        &#34;client&#34;: {
            &#34;sha1&#34;: &#34;4e969a3079409732a39aa722ea61d03876c41367&#34;,
            &#34;size&#34;: 25230298,
            &#34;url&#34;: &#34;https://piston-data.mojang.com/v1/objects/4e969a3079409732a39aa722ea61d03876c41367/client.jar&#34;
        },
        &#34;client_mappings&#34;: {
            &#34;sha1&#34;: &#34;85283b9708072cada19de2a29955957939af2127&#34;,
            &#34;size&#34;: 9308979,
            &#34;url&#34;: &#34;https://piston-data.mojang.com/v1/objects/85283b9708072cada19de2a29955957939af2127/client.txt&#34;
        },
        &#34;server&#34;: {
            &#34;sha1&#34;: &#34;00cab0438130dc3e6ae91f53387bb96ae7986d31&#34;,
            &#34;size&#34;: 50546942,
            &#34;url&#34;: &#34;https://piston-data.mojang.com/v1/objects/00cab0438130dc3e6ae91f53387bb96ae7986d31/server.jar&#34;
        },
        &#34;server_mappings&#34;: {
            &#34;sha1&#34;: &#34;a5e08ee736fb987f2920b98d25961245aac087bc&#34;,
            &#34;size&#34;: 7172652,
            &#34;url&#34;: &#34;https://piston-data.mojang.com/v1/objects/a5e08ee736fb987f2920b98d25961245aac087bc/server.txt&#34;
        }
    },
```

下载第三个元素`server`中的`URL`文件`server.jar`至根目录中。

在服务器根目录中执行如下命令：
该命令为初始化服务器组件与启动服务器。

```sh
java -Xmx4G -jar server.jar nogui
```

参数含义：

- `-Xmx4G` 最大内存4G
- `-jar server.jar` 指定`server.jar`为执行目标
- `nogui` 无图形界面执行

执行完后文件目录

```tree
├── eula.txt
├── libraries
├── logs
├── server.jar
├── server.properties
└── versions
```

将`eula.txt`文件中的`eula`变量由`false`改为`true`

服务器配置文件为`server.properties`，可参照 [Minecraft中文Wiki——配置服务器设置](https://minecraft.fandom.com/zh/wiki/Server.properties) 配置


### `Forge`

[下载地址](https://files.minecraftforge.net/net/minecraftforge/forge)[^1]
选择自己想要的版本，点击`Install`后跳转至下载页面
等待5秒后点击右上角的Skip即可下载

将下载的文件移至根目录中，而后执行如下命令：

```sh
java -jar forge-1.20.1-48.0.39.jar --installServer
```

参数含义：

- `-jar forge-1.20.1-48.0.39.jar` 指定执行文件，根据自身下载的文件更改该参数
- `--installServer` `Forge`自带参数

执行完后根目录如下：

```tree
├── libraries
├── run.bat
├── run.sh
└── user_jvm_args.txt
```

执行对应系统的启动脚本：

- `Windows`: `run.bat`
- `Linux`与`Mac OS`: `run.sh`

执行完后根目录如下：

```tree
├── config
├── defaultconfigs
├── eula.txt
├── libraries
├── logs
├── mods
├── run.bat
├── run.sh
└── user_jvm_args.txt
```

将`eula.txt`文件中的`eula`变量由`false`改为`true`。
模组存放于`mods`文件夹中。
`Java`启动参数在`user_jvm_args.txt`中设置。
服务器配置文件为`server.properties`，可参照 [Minecraft中文Wiki——配置服务器设置](https://minecraft.fandom.com/zh/wiki/Server.properties) 配置。

### `Fabric`

[下载地址](https://fabricmc.net/use/server)
下载所需版本至根目录中。

执行如下命令：
该命令为初始化服务器组件与启动服务器。

```sh
java -Xmx4G -jar fabric-server-mc.1.19.2-loader.0.14.21-launcher.0.11.2.jar nogui
```

参数含义：

- `-jar fabric-server-mc.1.19.2-loader.0.14.21-launcher.0.11.2.jar` 执行目标为`Fabric`服务器文件
- `-Xmx4G` 最大内存为`4G`
- `nogui` 无图形界面

执行完毕后根目录如下：

```tree
├── eula.txt
├── fabric-server-mc.1.19.2-loader.0.14.21-launcher.0.11.2.jar
├── libraries
├── logs
├── mods
├── server.jar
├── server.properties
└── versions
```

将`eula.txt`文件中的`eula`变量由`false`改为`true`。
模组存放于`mods`文件夹中。
服务器配置文件为`server.properties`，可参照 [Minecraft中文Wiki——配置服务器设置](https://minecraft.fandom.com/zh/wiki/Server.properties) 配置。

## 防火墙

`Minecraft`默认端口为25565。
可在`server.properties`中的`server-port`变量配置

服务器需要公网`IP`供他人访问

### `IPv4`

在防火墙中开启对应端口的`IPv4`出入。

`IPv4`的公网`IP`难以获得，若没有只能内网穿透
目前`IPv4`内网穿透有两种方式

- `Frp`
- `P2P`

`Frp`可能需要付费(有些有限流限额的免费)，但仅需服务端进行配置，客户端无需繁琐操作
`P2P`完全免费，但服务端与客户端都要进行专门的配置

### IPv6

在防火墙中开启对应端口的`IPv6`出入

`IPv6`因其庞大的数量，目前所有`IPv6`设备都有自己的属于`IPv6`的公网`IP`。
`IPv6`在国内虽说普及率达到90%以上，但由于设备的报废期问题，可能受到路由器、光猫、运营商等的拦截。

[^1]:注：该官网可能需要`VPN`才能流畅连接

---

> 作者: Bendancom  
> URL: http://localhost:1313/posts/%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AAjava%E7%89%88minecraft%E6%9C%8D%E5%8A%A1%E5%99%A8/  

