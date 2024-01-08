---
title: "Clash Core with Web Dashboard on Linux"
date: 2023-12-17T09:00:00+08:00
draft: false
authors: ["senzyo"]
tags: [Custom,Proxy,Linux]
series: [Use Clash Core]
series_weight: 2
featuredImagePreview: ""
summary: 本文不使用 Clash Verge 等 GUI 客户端, 而是通过 Dashboard 网页来控制 Clash Meta Core。
---

{{< admonition type=info title="提示" open=true >}}
鉴于原版 Clash Core (包括开源版本和 Premium 版本) 删库, 请使用 Clash Meta Core。
{{< /admonition >}}

## Clash核心

下载 [Clash Meta Core](https://github.com/MetaCubeX/mihomo/releases/tag/Prerelease-Alpha), 版本选择参考 [FAQ](https://wiki.metacubex.one/startup/faq/)。将 `mihomo-linux-amd64` 重命名为 `clash-meta-core`。

```bash
chmod +x clash-meta-core
sudo mv clash-meta-core /usr/local/bin/
```

## 配置文件

参考 [Clash配置文件](../2023-7/#Clash配置文件)。

下载 [Dashboard](https://github.com/MetaCubeX/metacubexd/releases/latest), 将 Dashboard 的文件夹重命名为 `MetaCubeXD`, 然后和其他 Clash 配置文件一起放到 `/etc/clash-meta/` 中。

## 设置守护进程

参考 [创建运行服务](https://wiki.metacubex.one/startup/service/)。

```bash
sudo vim /etc/systemd/system/clash-meta.service
```

写入以下内容:

```
[Unit]
Description=Clash Meta
After=network.target NetworkManager.service systemd-networkd.service

[Service]
Type=simple
LimitNPROC=500
LimitNOFILE=1000000
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE CAP_SYS_TIME
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE CAP_SYS_TIME
Restart=always
ExecStartPre=/usr/bin/sleep 1s
ExecStart=/usr/local/bin/clash-meta-core -d /etc/clash-meta
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

重载 systemd: 

```bash
sudo systemctl daemon-reload
```

设置开机自启并立刻启动: 

```bash
sudo systemctl enable --now clash-meta.service
```

查看状态: 

```bash
sudo systemctl status clash-meta.service
```

### Dashboard

访问 [http://127.0.0.1:9090/ui/](http://127.0.0.1:9090/ui/), 在 `API Base URL` 栏填写 `http://127.0.0.1:9090`, `Secret` 栏填写 `config.yaml` 中自己设置的 `secret` 随机字符串, 支持大小写英文字母+数字+特殊符号。

{{< admonition type=tip title="提醒" open=true >}}
完成之后的配置以后, 首次使用 MetaCubeXD 管理 Clash Meta Core 时, 最好先点击一下配置页面的 `更新 GEO 数据库` 按钮, 取决于网络状况, 数据库文件可能下载不完全, 比如只下载了 `GeoSite.dat` 而没有下载 `GeoIP.dat`。
{{< /admonition >}}
