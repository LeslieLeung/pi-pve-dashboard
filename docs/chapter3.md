# 配置告警

## 前言

本篇将会实现使用树莓派作为家庭服务器的运维工具，主要是在树莓派上搭建 influxDB 和 Grafana，对 Proxmox VE 的统计数据进行收集和可视化。本篇的第一章将会介绍树莓派环境的搭建，包括树莓派系统的安装以及 Docker 的安装；第二章将会介绍如何使用 Docker 运行 influxDB 和 Grafana，配置 PVE 将数据上报到 influxDB，以及在 Grafana 中建立一个面板来监控 PVE 的运行状态。

第三章在第二章搭建好面板的基础上，使用 Grafana 的告警功能，监控 PVE 的某些指标，当出现异常时进行告警。

## 前期准备

- 已经搭建好第二章中的 InfluxDB 和 Grafana，配置好一个面板

## 构造监控指标 query

进入我们之前配置好的 dashboard，在感兴趣的标题处悬停，然后点击 Explore，进入下列界面。

![](http://img.ameow.xyz/202303121959129.png)

其实这就是面板上写好的 query，如果有需要，也可以自行写 query。可以参考 [链接](https://docs.influxdata.com/influxdb/v2.6/query-data/get-started/) 。复制上面输入框的内容即可。

![](http://img.ameow.xyz/202303122001652.png)

## 添加告警途径

点击左边 Alerting，进入 Contact points，点击 New contact point 新增一个通知渠道。自带支持钉钉、Email、Telegram、Wecom、webhook等。便于演示，我将会使用企业微信进行测试，如果有 webhook 或者同时通知到多个服务的需求，可以看一下我的另一个项目 [heimdallr](https://github.com/LeslieLeung/heimdallr)。

![](http://img.ameow.xyz/202303122003028.png)

填入企业微信机器人的 webhook 地址即可。点击保存。

![](http://img.ameow.xyz/202303122007949.png)

告警渠道配置完后，还需要将新增的告警渠道设置为默认渠道。进入 Notification policies，修改默认 Contact point 为刚才的 Wecom。

![](http://img.ameow.xyz/202303122014887.png)

## 配置告警规则

回到 Alert rules，点击 New alert rule。在 A 的输入框里粘贴刚才我们从 dashboard 复制的 query，马上下面就能预览。

![](http://img.ameow.xyz/202303122009270.png)

继续往下，找到 B (Expression)，这里配置的是具体告警的规则。根据需要选择，我这里选择平均值大于15。下面的 Alert evaluation behavior 配置多久计算一次，持续多久后告警。最后在 3 处给规则命名，选择文件夹，点击右上角 Save and exit 即可。

![](http://img.ameow.xyz/202303122011828.png)

## 大功告成

为了测试告警，我们让系统模拟一下 CPU 负载增加的情况。

```bash
cat /dev/zero > /dev/null
```

稍等片刻后，看到企业微信来告警了。

![](http://img.ameow.xyz/202303122017765.png)

停止模拟负载，告警恢复。

![](http://img.ameow.xyz/202303122017578.png)