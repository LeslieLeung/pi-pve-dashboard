# pi-pve-dashboard

树莓派作为 Home Server 的运维工具。

## 前言

本篇将会实现使用树莓派作为家庭服务器的运维工具，主要是在树莓派上搭建 influxDB 和 Grafana，对 Proxmox VE 的统计数据进行收集和可视化。本篇的第一章将会介绍树莓派环境的搭建，包括树莓派系统的安装以及 Docker 的安装；第二章将会介绍如何使用 Docker 运行 influxDB 和 Grafana，配置 PVE 将数据上报到 influxDB，以及在 Grafana 中建立一个面板来监控 PVE 的运行状态。本章为系列的第二章。

如果你对树莓派不熟悉，可以参考本系列第一章 [树莓派作为 Home Server 运维工具 （一） 树莓派环境搭建](docs/chapter1.md)。

第一章实际上是针对树莓派的配置过程，第二章不针对树莓派，任意可以运行 Docker 的设备均适用。因此，你也可以使用其他设备（甚至就在 Home Server 本身）进行。

## 前期准备

- 一台树莓派（或其他独立的设备）
- 已经安装好 Docker, Git

## 使用 Docker 运行 InfluxDB 和 Grafana

为了简化部署，将使用 docker compose 来运行 InfluxDB 和 Grafana。

```bash
git clone https://github.com/LeslieLeung/pi-pve-dashboard.git
cd pi-pve-dashboard
sudo docker compose up -d
```

## 配置 InfluxDB

浏览器输入 `ip:8086` （ `ip` 为树莓派的 IP ）访问 InfluxDB。跟随向导创建账户。

![](http://img.ameow.xyz/202210231835152.png)

然后选择 configure later 即可。进入面板后，点击 Load Data，选择 API Tokens，添加一个 token。

![](http://img.ameow.xyz/202210231837703.png)

填入说明，确保 RW 权限选中刚才新建的 database `proxmox` 。点击 Save。

![](http://img.ameow.xyz/202210231838038.png)

重新点击刚才新建的 token， 复制刚才生成的 token。

![](http://img.ameow.xyz/202210231840046.png)

## 配置 PVE 上报

登录 PVE，依次点击 Datacenter - Metric Server - Add - InfluxDB。

![](http://img.ameow.xyz/202210231842307.png)

依次填入刚才的信息，点击 Create。

![](http://img.ameow.xyz/202210231843828.png)

回到 InfluxDB，点击 Data Explorer，选择下面的 filter，点击 submit，能看到数据即为成功。

![](http://img.ameow.xyz/202210231845479.png)

## 配置 Grafana

浏览器输入 `ip:3000` （ `ip` 为树莓派的 IP ）访问 Grafana，首次登录默认账号密码为 admin, admin，会提示修改密码，修改后进入 Grafana。

点击左下角设置，Data sources。点击 Add data source，选择 InfluxDB。

![](http://img.ameow.xyz/202210231848904.png)

![](http://img.ameow.xyz/202210231849536.png)

然后填入对应的信息。注意 Query Language 需要选择 Flux（因为使用了 InfluxDB 2.0+）。

![](http://img.ameow.xyz/202210231850127.png)

最后点击 Save & test，无报错即成功。

## 创建 Grafana 面板

点击 Dashboards，Import，然后输入 `16537`，点击 Load。这是目前我发现支持 InfluxDB 2.0+ 且 不需要任何其他插件的面板，如果需要更多现成的面板，可以看看这个 [链接](https://grafana.com/grafana/dashboards/?search=proxmox) 。

![](http://img.ameow.xyz/202210231852481.png)

![](http://img.ameow.xyz/202210231853463.png)

选择数据源为刚才添加的 InfluxDB，然后点击 Import 即可。

![](http://img.ameow.xyz/202210231856882.png)

## 大功告成

![](http://img.ameow.xyz/202210231857879.png)

## Reference

[Monitoring Proxmox with InfluxDB and Grafana in 4 Easy steps](https://www.linuxsysadmins.com/monitoring-proxmox-with-grafana/)
