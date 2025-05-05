---
title: "搭建《我的世界》基岩版服务器"
date: 2025-05-02T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Game,Linux]
featuredImagePreview: ""
summary: "Host a dedicated server for Minecraft Bedrock Edition on your Linux🥳"
---

## 流程

```bash
sudo useradd -r -M -s /usr/sbin/nologin minecraft
```

```bash
sudo mkdir /opt/minecraft
```

从 [Minecraft 官网](https://www.minecraft.net/en-us/download/server/bedrock) 获取下载链接, 当前最新版链接: `https://www.minecraft.net/bedrockdedicatedserver/bin-linux/bedrock-server-1.21.73.01.zip`

```bash
cd /opt/minecraft
```

```bash
curl -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36" -o bedrock-server.zip "https://www.minecraft.net/bedrockdedicatedserver/bin-linux/bedrock-server-1.21.73.01.zip"
```

```bash
unzip bedrock-server.zip
```

```bash
sudo chown -R minecraft:minecraft /opt/minecraft
```

```bash
sudo vim /etc/systemd/system/minecraft.service
```

写入以下内容: 

```点击展开
[Unit]
Description=The Minecraft Server
After=network-online.target

[Service]
Type=simple
User=minecraft
Group=minecraft
WorkingDirectory=/opt/minecraft
Environment="LD_LIBRARY_PATH=."
ExecStart=/opt/minecraft/bedrock_server
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl enable minecraft --now
```

```bash
sudo systemctl status minecraft
```

## 注意事项

### 重启服务器

操作相关文件夹或文件后, 比如 `worlds/`, `server.properties`, `allowlist.json`, `permissions.json` 等, 重启服务器才能使相关更改生效。

```bash
sudo systemctl restart minecraft
```

### 文件夹归属

确保 `/opt/minecraft` 属于 minecraft 用户: 

```bash
sudo chown -R minecraft:minecraft /opt/minecraft
```

### 防火墙端口

记得在服务器的防火墙, 以及服务器网页控制台的防火墙中开放 `/opt/minecraft/server.properties` 中 `server-port` 和 `server-portv6` 使用的端口号。

### 世界

世界的数据, 也就是存档, 位于 `/opt/minecraft/worlds/` 路径下。

`/opt/minecraft/server.properties` 中的 `level-name` 即 `/opt/minecraft/worlds/XXX/levelname.txt` 中的值, 也就是世界的名称。

### 白名单

编辑 `/opt/minecraft/allowlist.json`: 

```json
[
  {
    "name": "Xbox 用户名",
    "xuid": "XUID (十进制) ",
    "ignoresPlayerLimit": false
  },
  {
    "name": "Xbox 用户名",
    "xuid": "XUID (十进制) ",
    "ignoresPlayerLimit": false
  }
]
```

通过 https://www.cxkes.me/xbox/xuid 获取 `xuid`。

### 角色权限

编辑 `/opt/minecraft/permissions.json`: 

```json
[
  {
    "permission": "operator",
    "xuid": "XUID (十进制) "
  },
  {
    "permission": "member",
    "xuid": "XUID (十进制) "
  }
]
```
