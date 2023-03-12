# 一 树莓派环境搭建

## 前言

本篇将会实现使用树莓派作为家庭服务器的运维工具，主要是在树莓派上搭建 influxDB 和 Grafana，对 Proxmox VE 的统计数据进行收集和可视化。本篇的第一章将会介绍树莓派环境的搭建，包括树莓派系统的安装以及 Docker 的安装；第二章将会介绍如何使用 Docker 运行 influxDB 和 Grafana，配置 PVE 将数据上报到 influxDB，以及在 Grafana 中建立一个面板来监控 PVE 的运行状态。

## 前期准备

- 一个树莓派（3B、4B均可），我使用的是 4B
- 一张 TF 卡，推荐大于 32G
- 读卡器

## 树莓派系统安装

树莓派官方已经推出了官方的64位系统，因此没有必要再使用第三方的系统了（例如之前挺好用的树莓派爱好者基地的64位系统 [openfans-community-official / Debian-Pi-Aarch64](https://github.com/openfans-community-offical/Debian-Pi-Aarch64) ，该项目已经不再维护）。官方的系统安装方式也非常简单，前往 [官方网站](https://www.raspberrypi.com/software/) 下载对应系统的烧录工具，安装后打开即可。

进入软件后界面如图所示。

![](http://img.ameow.xyz/20221023172648.png)

点击选择操作系统，选择 other，选择 Raspberry Pi OS Lite (64-bit) ，因为此处我们是想将树莓派完全作为一个没有显示输出的服务器使用，所以没必要带上图形界面。

![](http://img.ameow.xyz/20221023172749.png)

![](http://img.ameow.xyz/202210231802000.png)

选择完镜像后选择存储卡，如果插入了多张卡，请务必选择正确的卡，否则会被抹掉数据。

最后点击右下角设置，勾选设置主机名（方便稍后查找树莓派的 IP），开启 SSH 服务，并设置用户名和密码，最后点击保存。然后点击烧录即可。点击烧录之后会弹出来一个警告，确认现有数据会被删除，点击是。（ macOS 用户可能需要输入自己用户的密码，第一次使用时可能还需要授权，允许即可。）

![](http://img.ameow.xyz/202210231803977.png)

等待写入和验证文件完成，然后就可以拔出 TF 卡。

![](http://img.ameow.xyz/20221023173516.png)

此时，将 TF 卡拔出，插入树莓派的卡槽中，电源线和网线，通电开机。至此，树莓派系统安装完成。

## 树莓派配置

稍等 30s 左右，树莓派应该开机完成了。打开终端（在 Windows 上则打开 cmd），输入  `arp -a`  并回车，应该能看到我们之前设置的 `raspberrypi.lan` 主机名，记下对应的 IP 地址。使用你习惯的终端，输入 IP 地址、用户名密码，连接即可。

### （可选）安装 vim

```bash
sudo apt-get install vim
```

### 配置静态地址

由于后续需要通过固定的地址来访问树莓派上的服务，所以需要配置一个静态的 IP 地址。过程可以参考 [链接](https://raspberrypi-guide.github.io/networking/set-up-static-ip-address)。

```bash
sudo vim /etc/dhcpcd.conf
```

（如果你没有安装 vim，也可以使用 nano）根据实际情况填入以下内容。

```
interface INTERFACE
static ip_address=YOURSTATICIP/24
static routers=YOURGATEWAYIP
static domain_name_servers= YOURGATEWAYIP
```

然后重启树莓派，如果你修改了 IP 地址，同时也要修改 SSH 连接配置。

```bash
sudo reboot
```

### 安装 Docker 和 Git

这里使用 DaoCloud 的脚本来安装 Docker。

```
curl -sSL https://get.daocloud.io/docker | sh
```

安装 Git。

```bash
sudo apt-get install git
```

## 大功告成

至此，树莓派的环境已经搭建完成。