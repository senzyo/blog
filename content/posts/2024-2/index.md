---
title: "sing-box on Linux"
date: 2024-01-08T09:00:00+08:00
draft: false
authors: ["senzyo"]
tags: [Custom,Proxy,Linux]
series: [Use sing-box]
series_weight: 2
featuredImagePreview: ""
summary: 开机自动更新 `config.json` 并启动 sing-box。通过 Dashboard 网页来控制 sing-box。
---

由于 sing-box 不像 clash 那样支持 `proxy-providers`, 且核心程序不支持定时更新配置文件 (好在 Android 和 Apple 图形客户端支持), 所以只能通过脚本定时下载远端配置文件来更新。

思路是通过 [Systemd](https://wiki.archlinux.org/title/Systemd) 的 [Timer](https://wiki.archlinux.org/title/systemd/Timers) 来开机自启动对应的 Service, 这个 Service 将执行一个脚本, 脚本循环检测网络可用性, 当网络可用时, 从远端下载配置文件替换本地的配置文件, 然后启动 `sing-box.service`。

## sing-box

### Install

使用 sing-box [官方脚本](https://sing-box.sagernet.org/zh/installation/package-manager/) 安装的是正式版, 要想使用预发行版, 需要手动下载安装。

https://github.com/SagerNet/sing-box/releases/

```
tar -zxf sing-box-<version>-linux-amd64.tar.gz
mv sing-box /usr/local/bin/sing-box
```

### Service

```bash
sudo vim /etc/systemd/system/sing-box.service
```

```
[Unit]
Description=sing-box service
Documentation=https://sing-box.sagernet.org
After=network.target nss-lookup.target

[Service]
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE CAP_SYS_PTRACE CAP_DAC_READ_SEARCH
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE CAP_SYS_PTRACE CAP_DAC_READ_SEARCH
ExecStart=/usr/local/bin/sing-box -D /etc/sing-box -c /home/<UserName>/.config/sing-box/config.json run
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=10s
LimitNOFILE=infinity

[Install]
WantedBy=multi-user.target
```

## Dashboard

### 自动

如果 `config.json` 中设置了 `external_ui_download_url` 相关:

```json
{
...
  "experimental": {
    "clash_api": {
      "external_controller": "127.0.0.1:9090",
      "external_ui": "ui",
      "external_ui_download_url": "https://ghproxy.net/https://github.com/MetaCubeX/Yacd-meta/archive/gh-pages.zip",
      "external_ui_download_detour": "🐢 直连",
      "secret": "AjIuQAZf795UQ16V3si6",
      "default_mode": "rule"
    },
...
  },
...
}
```

则不需要手动下载, 只需等待 sing-box 自动处理即可。

### 手动

```bash
mkdir /home/<UserName>/.config/sing-box
```

下载 [Yacd-meta](https://ghproxy.net/https://github.com/MetaCubeX/Yacd-meta/archive/gh-pages.zip) 或者 [MetaCubeXD](https://ghproxy.net/https://github.com/MetaCubeX/metacubexd/archive/gh-pages.zip), 解压后将文件夹重命名为 `ui`, 移动到 sing-box 工作目录下, 比如 `/etc/sing-box/ui`。

## Timer

```bash
sudo vim /etc/systemd/system/sing-box-trigger.timer
```

```
[Unit]
Description=sing-box trigger timer

[Timer]
OnBootSec=10s
Unit=sing-box-trigger.service

[Install]
WantedBy=timers.target
```

```bash
sudo vim /etc/systemd/system/sing-box-trigger.service
```

```
[Unit]
Description=sing-box trigger service

[Service]
Type=simple
ExecStart=/home/<UserName>/.config/sing-box/update.sh
Restart=on-watchdog
WatchdogSec=60
```

```bash
sudo systemctl enable sing-box-trigger.timer
```

```bash
sudo systemctl status sing-box-trigger.timer
```

## 日常使用

### 更新配置文件

按需修改 `update.sh` 的内容:

更改 `$url_sub` 的值为自己的订阅链接, 至于模板 `$url_tpl`, 可以参考 [Generate config of sing-box](../2024-1/) 选择更多模板。

脚本通过 KDE 的 `kwriteconfig5` 来控制系统代理的开关。更改 `$kioslaverc` 的值为自己的 `kioslaverc` 文件路径, 如不需要可将其删除并注释 `kwriteconfig5` 部分。

{{< admonition type=question title="无法启动?" open=false >}}
对于 sing-box 的运行参数, 由 `sing-box help` 得知: 

```
Flags:
  -c, --config stringArray             set configuration file path
  -C, --config-directory stringArray   set configuration directory path
  -D, --directory string               set working directory
      --disable-color                  disable color output
  -h, --help                           help for sing-box
```

`-C` 指定了配置文件所在文件夹的路径, 但没有指定使用哪一个配置文件; `-c` 则指明了使用的配置文件的路径。

如果在 [Service](#service) 中 `ExecStart` 部分使用的 sing-box 运行参数是 `-D /etc/sing-box -C /home/<UserName>/.config/sing-box run`。
则要求配置文件所在的文件夹中不能有多个 `json` 文件, 否则 sing-box 无法判定使用哪一个作为配置, 继而无法启动。
{{< /admonition >}}

{{< admonition type=tip title="提示" open=true >}}
- 从下面两种方案中选择一个即可。
- 手动运行这个更新配置文件的脚本时需要 `sudo`。
{{< /admonition >}}

#### 一种配置

下载使用 `mixed` 或 `tun` 入站的配置文件。

```bash
vim /home/<UserName>/.config/sing-box/update.sh
```

```bash
#!/bin/bash

service="sing-box.service"
ping_site="ntp.aliyun.com"
dir_config="/home/<UserName>/.config/sing-box"
url_gene="https://sing-box.senzyo.net"
url_sub="https://example.com/xxx"
inbound="mixed" # mixed or tun
url_tpl="https://raw.githubusercontent.com/senzyo/sing-box-template/normal/$inbound/doh/8.8.8.8/ghproxy.net/config.json"
url_dl="$url_gene/config/url=$url_sub/&emoji=1&UA=clashmeta&file=$url_tpl"
kioslaverc="/home/<UserName>/.config/kioslaverc"
proxy_port="7890"

systemctl stop $service
kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 0
until ping -c 1 $ping_site &>/dev/null; do
    sleep 2
done
cd $dir_config
count=0
while [ $count -lt 3 ]; do
    curl -fsSLo temp.json "$url_dl"
    sing-box check -c temp.json &>/dev/null
    if [ $? -eq 0 ]; then
        echo -e "Downloaded \e[1;33mconfig($inbound)\e[0m is \e[1;32mvalid\e[0m."
        mv temp.json config_$inbound.json
        echo -e "Use \e[1;33mconfig($inbound)\e[0m."
        cp config_$inbound.json config.json
        break
    fi
    rm -f temp.json
    ((count++))
    echo -e "Downloaded \e[1;33mconfig($inbound)\e[0m is \e[1;31minvalid\e[0m. Tried $count times."
done
if [ $count -eq 3 ]; then
    echo "No changes, keep the existing files."
fi
systemctl start $service
sleep 3
systemctl status $service --no-pager
if [ $? -eq 0 ]; then
    if [[ $inbound == "mixed" ]]; then
        echo -e "Switch to using \e[1;34msystem proxy\e[0m."
        kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "ftpProxy" "http://127.0.0.1 $proxy_port"
        kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "httpProxy" "http://127.0.0.1 $proxy_port"
        kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "httpsProxy" "http://127.0.0.1 $proxy_port"
        kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "socksProxy" "socks://127.0.0.1 $proxy_port"
        kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 1
    elif [[ $inbound == "tun" ]]; then
        echo -e "Switch to using \e[1;34mTUN\e[0m."
        kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 0
    fi
fi
```

```bash
chmod +x /home/<UserName>/.config/sing-box/update.sh
```

{{< admonition type=success title="完成了" open=true >}}
现在可以运行 `sudo systemctl start sing-box-trigger.timer` 测试一下整个流程, 应该一切正常。
{{< /admonition >}}

#### 两种配置

下载使用 `mixed` 和 `tun` 入站的配置文件。优先使用 `mixed` 入站, 即只使用系统代理。搭配 [切换模式重启](#切换模式重启) 函数来切换模式。

```bash
vim /home/<UserName>/.config/sing-box/update.sh
```

```bash
#!/bin/bash

service="sing-box.service"
ping_site="ntp.aliyun.com"
dir_config="/home/<UserName>/.config/sing-box"
url_gene="https://sing-box.senzyo.net"
url_sub="https://example.com/xxx"
inbound_prefer="mixed" # mixed or tun
kioslaverc="/home/<UserName>/.config/kioslaverc"
proxy_port="7890"

systemctl stop $service
kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 0
until ping -c 1 $ping_site &>/dev/null; do
    sleep 2
done
cd $dir_config
inbound_list=(mixed tun)
for inbound in "${inbound_list[@]}"; do
    url_tpl="https://raw.githubusercontent.com/senzyo/sing-box-template/normal/$inbound/doh/8.8.8.8/ghproxy.net/config.json"
    url_dl="$url_gene/config/url=$url_sub/&emoji=1&UA=clashmeta&file=$url_tpl"
    count=0
    while [ $count -lt 3 ]; do
        curl -fsSLo temp.json "$url_dl"
        sing-box check -c temp.json &>/dev/null
        if [ $? -eq 0 ]; then
            echo -e "Downloaded \e[1;33mconfig($inbound)\e[0m is \e[1;32mvalid\e[0m."
            mv temp.json config_$inbound.json
            if [[ $inbound == $inbound_prefer ]]; then
                echo -e "Use \e[1;33mconfig($inbound)\e[0m."
                cp config_$inbound.json config.json
            fi
            break
        fi
        rm -f temp.json
        ((count++))
        echo -e "Downloaded \e[1;33mconfig($inbound)\e[0m is \e[1;31minvalid\e[0m. Tried $count times."
    done
    if [ $count -eq 3 ]; then
        echo "No changes, keep the existing files."
    fi
done
systemctl start $service
sleep 3
systemctl status $service --no-pager
if [ $? -eq 0 ]; then
    if [[ $inbound_prefer == "mixed" ]]; then
        echo -e "Switch to using \e[1;34msystem proxy\e[0m."
        kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "ftpProxy" "http://127.0.0.1 $proxy_port"
        kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "httpProxy" "http://127.0.0.1 $proxy_port"
        kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "httpsProxy" "http://127.0.0.1 $proxy_port"
        kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "socksProxy" "socks://127.0.0.1 $proxy_port"
        kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 1
    elif [[ $inbound_prefer == "tun" ]]; then
        echo -e "Switch to using \e[1;34mTUN\e[0m."
        kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 0
    fi
fi
```

```bash
chmod +x /home/<UserName>/.config/sing-box/update.sh
```

{{< admonition type=success title="完成了" open=true >}}
现在可以运行 `sudo systemctl start sing-box-trigger.timer` 测试一下整个流程, 应该一切正常。
{{< /admonition >}}

### 开关系统和浏览器代理

Chrome 会跟随 KDE 系统代理, Firefox 则不然。所以对于 KDE 系统代理和 [Firefox](../2024-4) 代理, 将操作命令封装成函数写入 [环境变量配置文件](../2021-6/) 中, 比如修改以下内容写入 `/etc/proxy-custom`: 

```bash
proxy_port="7890"
kioslaverc="/home/<UserName>/.config/kioslaverc"
firefox_profile="/home/<UserName>/.mozilla/firefox/xxx/user.js"

function proxy-set() {
    export http_proxy=http://127.0.0.1:$proxy_port
    export https_proxy=http://127.0.0.1:$proxy_port
    echo -e "Proxy environment variable has been \e[1;32mset\e[0m."
}

function proxy-unset() {
    unset http_proxy https_proxy
    echo -e "Proxy environment variable has been \e[1;31munset\e[0m."
}

function kde-proxy-on() {
    kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "ftpProxy" "http://127.0.0.1 $proxy_port"
    kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "httpProxy" "http://127.0.0.1 $proxy_port"
    kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "httpsProxy" "http://127.0.0.1 $proxy_port"
    kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "socksProxy" "socks://127.0.0.1 $proxy_port"
    kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 1
    echo -e "Proxy for KDE \e[1;32menabled\e[0m."
}

function kde-proxy-off() {
    kwriteconfig5 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 0
    echo -e "Proxy for KDE \e[1;31mdisabled\e[0m."
}

function ff-proxy-on() {
    cat <<EOF >"$firefox_profile"
user_pref("network.proxy.backup.ssl", "127.0.0.1");
user_pref("network.proxy.backup.ssl_port", $proxy_port);
user_pref("network.proxy.http", "127.0.0.1");
user_pref("network.proxy.http_port", $proxy_port);
user_pref("network.proxy.share_proxy_settings", true);
user_pref("network.proxy.socks", "127.0.0.1");
user_pref("network.proxy.socks_port", $proxy_port);
user_pref("network.proxy.socks_remote_dns", true);
user_pref("network.proxy.ssl", "127.0.0.1");
user_pref("network.proxy.ssl_port", $proxy_port);
user_pref("network.proxy.type", 1);
EOF
    echo -e "Proxy for Firefox \e[1;32menabled\e[0m. Restart Firefox to apply."
}

function ff-proxy-off() {
    echo 'user_pref("network.proxy.type", 0);' >"$firefox_profile"
    echo -e "Proxy for Firefox \e[1;31mdisabled\e[0m. Restart Firefox to apply."
}
```

然后在 `/etc/bash.bashrc` 中写入: 

```bash
if [ -f /etc/proxy-custom ]; then
  source /etc/proxy-custom
fi
```

### 全部关闭

将以下代码追加到 `/etc/proxy-custom` 中: 

```bash
function proxy-off() {
    service="sing-box.service"
    kde-proxy-off
    ff-proxy-off
    sudo systemctl stop $service
    sleep 1
    sudo systemctl status $service --no-pager
}
```

### 当前模式重启

将以下代码追加到 `/etc/proxy-custom` 中: 

```bash
function proxy-restart() {
    dir_config="/home/<UserName>/.config/sing-box"
    service="sing-box.service"
    cd $dir_config
    sing-box check -c config.json &>/dev/null
    if [ $? -ne 0 ]; then
        echo -e "File config.json is invalid."
        exit 1
    fi
    sudo systemctl restart $service
    sleep 3
    sudo systemctl status $service --no-pager
    if [ $? -eq 0 ]; then
        if grep -q '"type": "tun"' config.json; then
            echo -e "Keep using \e[1;34mTUN\e[0m."
            kde-proxy-off
            ff-proxy-off
        else
            echo -e "Keep using \e[1;34msystem proxy\e[0m."
            kde-proxy-on
            ff-proxy-on
        fi
    fi
    cd -
}
```

相比直接 `sudo systemctl restart sing-box.service`, 这个函数看起来有些多此一举, 其实目的是方便调试 `config.json`, 同时覆盖代理开关状态。

### 切换模式重启

sing-box 的入站方式是固定在配置文件中的, 而且也不支持像 Clash Meta Core 那样简单地通过外部控制 API 来切换, 所以只能通过切换配置文件的方式在 `mixed` 和 `tun` 之间切换。

将以下代码追加到 `/etc/proxy-custom` 中: 

```bash
function proxy-switch() {
    dir_config="/home/<UserName>/.config/sing-box"
    service="sing-box.service"
    cd $dir_config
    if grep -q '"type": "tun"' config.json; then
        echo -e "Switch to using \e[1;34msystem proxy\e[0m."
        inbound="mixed"
    else
        echo -e "Switch to using \e[1;34mTUN\e[0m."
        inbound="tun"
    fi
    sing-box check -c config_$inbound.json &>/dev/null
    if [ $? -ne 0 ]; then
        echo -e "File \e[1;33mconfig_$inbound.json\e[0m is invalid."
        exit 1
    fi
    sudo cp config_$inbound.json config.json
    sudo systemctl restart $service
    sleep 3
    sudo systemctl status $service --no-pager
    if [ $? -eq 0 ]; then
        if [[ $inbound == "mixed" ]]; then
            kde-proxy-on
            ff-proxy-on
        elif [[ $inbound == "tun" ]]; then
            kde-proxy-off
            ff-proxy-off
        fi
    fi
    cd -
}
```