---
title: "Clash on Linux"
date: 2023-12-17T09:00:00+08:00
draft: false
authors: ["SenZyo"]
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

## 设置守护进程

参考 [创建运行服务](https://wiki.metacubex.one/startup/service/)。

```bash
sudo vim /etc/systemd/system/clash-meta.service
```

写入以下内容:

```
[Unit]
Description=Clash Meta
Documentation=https://wiki.metacubex.one
After=network.target NetworkManager.service systemd-networkd.service

[Service]
Type=simple
LimitNPROC=500
LimitNOFILE=1000000
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE CAP_SYS_TIME CAP_SYS_PTRACE CAP_DAC_READ_SEARCH
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE CAP_SYS_TIME CAP_SYS_PTRACE CAP_DAC_READ_SEARCH
Restart=on-failure
RestartSec=10s
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

## Dashboard

下载 [Yacd-meta](https://ghproxy.net/https://github.com/MetaCubeX/Yacd-meta/archive/gh-pages.zip) 或者 [MetaCubeXD](https://ghproxy.net/https://github.com/MetaCubeX/metacubexd/archive/gh-pages.zip), 解压后将文件夹重命名为 `ui`。然后和其他 Clash 配置文件一起放到 `/etc/clash-meta/` 中。

如果在配置文件中使用 `external-ui-url`, core 启动时会自动下载 Dashboard, 这样就不用手动下载放置了。

访问 [http://127.0.0.1:9090/ui/](http://127.0.0.1:9090/ui/), 在 `API Base URL` 栏填写 `http://127.0.0.1:9090`, `Secret` 栏填写 `config.yaml` 中自己设置的 `secret` 随机字符串, 支持大小写英文字母+数字+特殊符号。

{{< admonition type=tip title="提醒" open=true >}}
首次使用 MetaCubeXD 管理 Clash Meta Core 时, 最好先点击一下配置页面的 `更新 GEO 数据库` 按钮, 取决于网络状况, 数据库文件可能下载不完全, 比如只下载了 `GeoSite.dat` 而没有下载 `GeoIP.dat`。
{{< /admonition >}}

## 日常使用

`clash-meta.service` 可以用 `systemctl` 命令方便地开关。

Chrome 会跟随 KDE 系统代理, Firefox 则不然。对于 KDE 系统代理和 [Firefox](../2024-4) 代理, 参考 [开关系统和浏览器代理](../2024-2/#开关系统和浏览器代理)。

至于开关 TUN, 通过 Dashboard 手动点一下得了, 懒得写脚本, 反正我已经转为使用 [sing-box](../2024-2/) 了。