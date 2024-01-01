---
title: "Clash on Windows"
date: 2023-10-10T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Custom,Proxy,PowerShell,Windows]
series: [Use Clash Core]
series_weight: 1
featuredImagePreview: ""
summary: 本文不使用 Clash for Windows 等 GUI 客户端, 而是通过 Dashboard 网页来控制 Clash Core (包括 Premium 和 Meta)。
---

{{< admonition type=info title="提示" open=true >}}
鉴于原版 Clash Core (包括开源版本和 Premium 版本) 删库, 请使用 Clash Meta Core, Clash Premium Core 章节仅作参考。
{{< /admonition >}}

## Clash Meta Core

### 下载

- Core: https://github.com/MetaCubeX/mihomo/releases/tag/Prerelease-Alpha, 版本选择参考 [FAQ](https://wiki.metacubex.one/startup/faq/)。将 `mihomo-windows-amd64.exe` 重命名为 `ClashMetaCore.exe`。
- WinTUN.dll: Clash Meta Core 集成 `wintun.dll`, 不必单独下载。
- Dashboard: [Yacd-meta](https://ghproxy.net/https://github.com/MetaCubeX/Yacd-meta/archive/gh-pages.zip) 或者 [MetaCubeXD](https://ghproxy.net/https://github.com/MetaCubeX/metacubexd/archive/gh-pages.zip), 解压后将文件夹重命名为 `ui`。

将 `ClashMetaCore.exe` 和 `ui` 文件夹一起放入 Clash 工作目录中, 比如 `D:\Clash Meta`。
如果在配置文件中使用 `external-ui-url`, core 启动时会自动下载 Dashboard, 这样就不用手动下载放置了。

### Clash配置文件

在 Clash 工作目录中创建 `config.yaml` 文件, 参考以下配置、官方的 [参考配置](https://github.com/MetaCubeX/mihomo/blob/Alpha/docs/config.yaml) 和 [配置详解](https://wiki.metacubex.one/config/) 写入: 

```yaml
# 官方参考配置 https://github.com/MetaCubeX/mihomo/blob/Alpha/docs/config.yaml
# 官方配置详解 https://wiki.metacubex.one/config/

mixed-port: 7890
allow-lan: false
bind-address: "*"
find-process-mode: off
mode: rule
geodata-mode: true
geodata-loader: memconservative
geox-url:
  geoip: "https://cdn.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geoip.dat"
  geosite: "https://cdn.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geosite.dat"
  mmdb: "https://cdn.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/country.mmdb"
geo-auto-update: true
geo-update-interval: 24
global-ua: clashmeta
log-level: silent
ipv6: true
external-controller: 127.0.0.1:9090
secret: 1j8Dm52W2nS4aV69Fw15
external-ui: ui
# external-ui-url: "https://ghproxy.net/https://github.com/MetaCubeX/metacubexd/archive/refs/heads/gh-pages.zip"
# external-ui-url: "https://ghproxy.net/https://github.com/MetaCubeX/Yacd-meta/archive/gh-pages.zip"
unified-delay: false
tcp-concurrent: true
global-client-fingerprint: chrome
keep-alive-interval: 30

profile:
  store-selected: false
  store-fake-ip: false

tun:
  enable: false
  stack: mixed
  auto-route: true
  auto-detect-interface: true
  strict_route: false
  dns-hijack:
    - any:53

sniffer:
  enable: true
  force-dns-mapping: true
  parse-pure-ip: true
  override-destination: false
  sniff:
    HTTP:
      ports: [80]
    TLS:
      ports: [443]
    QUIC:
      ports: [443]

dns:
  enable: true
  prefer-h3: false
  listen: 0.0.0.0:53
  ipv6: true
  # redir-host 或者 fake-ip
  enhanced-mode: redir-host
  fake-ip-range: 198.18.0.1/16
  fake-ip-filter:
    - "+.stun.*.*"
    - "+.stun.*.*.*"
    - "+.stun.*.*.*.*"
    - "+.stun.*.*.*.*.*"
    - "*.n.n.srv.nintendo.net"
    - "+.stun.playstation.net"
    - "xbox.*.*.microsoft.com"
    - "+.xboxlive.com"
    - "+.msftncsi.com"
    - "+.msftconnecttest.com"
    - WORKGROUP
    - "connect.rom.miui.com"
    - "connectivitycheck.platform.hicloud.com"
    - "wifi.vivo.com.cn"
    - "www.qualcomm.cn"
    - "+.ntp.org.cn"
    - "+.pool.ntp.org"
    - "ntp.aliyun.com"
    - "ntp.ntsc.ac.cn"
    - "ntp.tencent.com"
    - "time.apple.com"
    - "time.windows.com"
    - "+.edu.cn"
  default-nameserver:
    - 223.5.5.5
    - 8.8.8.8
  # 更多 DNS 参考: https://senzyo.net/2022-22/
  nameserver-policy:
    "geosite:cn":
      - tls://223.5.5.5
      - tls://1.12.12.12
  nameserver:
    - tls://1.1.1.1
    - tls://8.8.8.8
    - tls://9.9.9.9

proxy-groups:
  - name: ✈️ 节点选择
    type: select
    disable-udp: false
    proxies:
      - 🌏 日韩台新
      - DIRECT
      - 📌 手动切换
      - 🇭🇰 香港节点
      - 🇯🇵 日本节点
      - 🇰🇷 韩国节点
      - 🇹🇼 台湾节点
      - 🇸🇬 新加坡节点
      - 🇺🇸 美国节点
  - name: 🌏 日韩台新
    type: url-test
    disable-udp: false
    # 测试节点延迟的时间间隔, 单位为 s, 此处仅影响 proxies 下的节点, 不影响 use 下的节点
    # interval: 300
    # lazy 默认为 true, 未选择到当前策略组时,则不会进行测试
    # lazy: false
    # 当 (某一节点的延迟) < (当前节点的延迟-容差) 时, 才会切换到新的节点。默认值为 0ms, 可选项
    # tolerance: 20
    url: "https://www.gstatic.com/generate_204"
    filter: "🇯🇵|日本|JP|Japan|🇰🇷|韩国|KR|South Korea|🇹🇼|台湾|TW|Taiwan|🇸🇬|新加坡|SG|Singapore"
    use:
      - Provider
  - name: 📌 手动切换
    type: select
    disable-udp: false
    exclude-filter: "剩余|流量|有效|时间|到期|expire|地址|网址|官网|自动|最优|最快"
    use:
      - Provider
  - name: 🤖 OpenAI
    type: select
    disable-udp: false
    proxies:
      - 🌏 日韩台新
      - 📌 手动切换
      - 🇯🇵 日本节点
      - 🇰🇷 韩国节点
      - 🇹🇼 台湾节点
      - 🇸🇬 新加坡节点
      - 🇺🇸 美国节点
  - name: ☁️ OneDrive
    type: select
    disable-udp: false
    proxies:
      - DIRECT
      - 🌏 日韩台新
      - 📌 手动切换
      - 🇭🇰 香港节点
      - 🇯🇵 日本节点
      - 🇰🇷 韩国节点
      - 🇹🇼 台湾节点
      - 🇸🇬 新加坡节点
      - 🇺🇸 美国节点
  - name: 🎮 游戏平台
    type: select
    disable-udp: false
    proxies:
      - 🌏 日韩台新
      - DIRECT
      - 📌 手动切换
      - 🇭🇰 香港节点
      - 🇯🇵 日本节点
      - 🇰🇷 韩国节点
      - 🇹🇼 台湾节点
      - 🇸🇬 新加坡节点
      - 🇺🇸 美国节点
  - name: 🐟 漏网之鱼
    type: select
    disable-udp: false
    proxies:
      - 🌏 日韩台新
      - DIRECT
      - 📌 手动切换
      - 🇭🇰 香港节点
      - 🇯🇵 日本节点
      - 🇰🇷 韩国节点
      - 🇹🇼 台湾节点
      - 🇸🇬 新加坡节点
      - 🇺🇸 美国节点
  - name: 🇭🇰 香港节点
    type: url-test
    disable-udp: false
    url: "https://www.gstatic.com/generate_204"
    filter: "🇭🇰|香港|HK|Hong Kong"
    use:
      - Provider
  - name: 🇯🇵 日本节点
    type: url-test
    disable-udp: false
    url: "https://www.gstatic.com/generate_204"
    filter: "🇯🇵|日本|JP|Japan"
    use:
      - Provider
  - name: 🇰🇷 韩国节点
    type: url-test
    disable-udp: false
    url: "https://www.gstatic.com/generate_204"
    filter: "🇰🇷|韩国|KR|South Korea"
    use:
      - Provider
  - name: 🇹🇼 台湾节点
    type: url-test
    disable-udp: false
    url: "https://www.gstatic.com/generate_204"
    filter: "🇹🇼|台湾|TW|Taiwan"
    use:
      - Provider
  - name: 🇸🇬 新加坡节点
    type: url-test
    disable-udp: false
    url: "https://www.gstatic.com/generate_204"
    filter: "🇸🇬|新加坡|SG|Singapore"
    use:
      - Provider
  - name: 🇺🇸 美国节点
    type: url-test
    disable-udp: false
    url: "https://www.gstatic.com/generate_204"
    filter: "🇺🇸|美国|US|USA|United States"
    use:
      - Provider

rules:
  - DOMAIN-SUFFIX,mangafuna.xyz,✈️ 节点选择
  # GeoSite, https://github.com/MetaCubeX/meta-rules-dat?tab=readme-ov-file#geositedatgeositedb-内容
  - GEOSITE,category-ads-all,REJECT
  - GEOIP,private,DIRECT,no-resolve
  - GEOSITE,private,DIRECT
  - RULE-SET,NTPService,DIRECT
  - GEOSITE,tracker,DIRECT
  - RULE-SET,OpenAI,🤖 OpenAI
  - RULE-SET,OneDrive,☁️ OneDrive
  - GEOSITE,category-games@cn,DIRECT
  - GEOSITE,category-games,🎮 游戏平台
  - GEOSITE,geolocation-!cn,✈️ 节点选择
  - RULE-SET,CloudCN,DIRECT
  - GEOIP,cn,DIRECT,no-resolve
  - GEOSITE,cn,DIRECT
  - MATCH,🐟 漏网之鱼

proxy-providers:
  Provider:
    type: http
    # 因为 clash premium core 更新 providers 时不带 User-Agent, 会造成机场返回的订阅信息可能不符合 clash 的格式。可以在订阅连接后添加 `&flag=clash` 来指定。
    # 而 clash meta core 因为有 global-ua 设置项, 一般不用添加 `&flag=clash` 来指定, 但是有的机场需要添加 `&flag=clashmeta` 来指定。
    url: "https://example.com/xxx&flag=clashmeta"
    # 更新订阅的时间间隔, 单位为 s
    interval: 43200
    path: ./Provider.yaml
    health-check:
      enable: true
      # 测试节点延迟的时间间隔, 单位为 s, 此处仅影响 proxy-group 中 use 下的节点
      interval: 600
      lazy: false
      url: https://www.gstatic.com/generate_204

rule-providers:
  NTPService:
    type: http
    behavior: classical
    url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/NTPService/NTPService.yaml
    path: ./rule-providers/NTPService.yaml
    # 更新规则的时间间隔, 单位为 s
    interval: 604800
  OpenAI:
    type: http
    behavior: classical
    url: https://ghproxy.net/https://raw.githubusercontent.com/senzyo/as-gist/master/Rule/OpenAI/OpenAI.yaml
    path: ./rule-providers/OpenAI.yaml
    interval: 604800
  CloudCN:
    type: http
    behavior: classical
    url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/Cloud/CloudCN/CloudCN.yaml
    path: ./rule-providers/CloudCN.yaml
    interval: 604800
```

### 开机自启动

#### 方案一: 任务计划程序

打开任务计划程序, 点击右侧操作列表中的“创建任务”:

1. 常规

   `更改用户或组`, 输入 `system`, 点击 `检查名称` 按钮, 变为 <u>SYSTEM</u>, 即使用 `SYSTEM` 账户。

2. 触发器

    新建, `开始任务` 选择 `登录时` 或 `启动时`, 按需选择 `延迟任务时间`, 确认勾选 `已启用`。

3. 操作

    新建, `操作` 选择 `启动程序`; `程序或脚本` 填写 `ClashMetaCore.exe`; `添加参数` 填写 `-d .\ -f config.yaml`; `起始于` 填写 Clash 工作目录。

4. 条件

    全部取消勾选, 包括灰显的。

5. 设置

   1. 仅勾选 `允许按需运行任务`。
   2. 最下方的 `如果此任务已经运行, 以下规则适用`, 确认选择 `请勿启动新实例`。

6. 运行任务, 检查是否正常。

#### 方案二: Windows服务

利用 [winsw](https://github.com/winsw/winsw/releases) 安装 Windows 服务。

1. 将以下内容保存为 `Clash Meta Core Service.xml`, 和 `winsw.exe` 一起放入 Clash 工作目录中。

    ```xml
    <service>
        <startmode>Automatic</startmode>
        <id>Clash Meta Core Service</id>
        <name>Clash Meta 核心服务</name>
        <description></description>
        <executable>ClashMetaCore.exe</executable>
        <arguments>-d .\ -f config.yaml</arguments>
        <log mode="none"></log>
    </service>
    ```

    注意, `<id>` 指 `Name`, `<name>` 指 `DisplayName`。更多 XML 设置参考 [文档](https://github.com/winsw/winsw/blob/v3.0.0-alpha.11/docs/xml-config-file.md)。

2. 在 Clash 工作目录中运行以下命令将 Clash Meta Core 安装为 Windows 服务: 

    ```shell
    winsw.exe install "Clash Meta Core Service.xml"
    ```

3. 运行服务, 检查是否正常。

### 防火墙

防火墙入站规则放行 Clash Core 程序。

### Dashboard

访问 [http://127.0.0.1:9090/ui/](http://127.0.0.1:9090/ui/), 在 `API Base URL` 栏填写 `http://127.0.0.1:9090`, `Secret` 栏填写 `config.yaml` 中自己设置的 `secret` 随机字符串, 支持大小写英文字母+数字+特殊符号。

{{< admonition type=tip title="提醒" open=true >}}
首次使用 MetaCubeXD 管理 Clash Meta Core 时, 最好先点击一下配置页面的 `更新 GEO 数据库` 按钮, 取决于网络状况, 数据库文件可能下载不完全, 比如只下载了 `GeoSite.dat` 而没有下载 `GeoIP.dat`。
{{< /admonition >}}

{{< admonition type=success title="基本完成了" open=true >}}
现在可以运行 `任务计划程序` 或 `Windows 服务` 测试一下整个流程, 应该一切正常。
{{< /admonition >}}

### 脚本

{{< admonition type=info title="推荐使用脚本" open=true >}}
这些脚本是为了方便日常使用的。设置好脚本的快捷方式后, 日常点一下就行。
{{< /admonition >}}

#### Clash Switch.ps1

**切换系统代理和TUN**: 

```shell
# Clash 配置文件设置的 mixed-port
$proxyPort = 7890
# Clash 配置文件设置的 external-controller
$controllerAPI = "http://127.0.0.1:9090"
# Clash 配置文件设置的 external-controller 的 secret
$apiSecret = "1j8Dm52W2nS4aV69Fw15"
# Clash 任务计划程序名称
$taskName = "Clash Meta Core"
# Clash 进程名称
$processName = "ClashMetaCore"

# 检测是否存在 Clash 进程
$clashStatus = Get-Process -Name $processName -ErrorAction SilentlyContinue
if (! $clashStatus) {
    Write-Host "Clash is not running."
    # 刷新 DNS 缓存
    Clear-DnsClientCache
    # 启动 Clash 任务计划程序
    Start-ScheduledTask -TaskName $taskName
    Write-Host "Clash started."
}

# 检查系统代理开关状态
$proxyEnabled = (Get-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings').ProxyEnable
if ($proxyEnabled -eq 0) {
    # 打开并设置系统代理, 关闭 TUN 模式
    Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 1
    Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyServer -Value "127.0.0.1:$proxyPort"
    Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyOverride -Value "localhost;127.*;10.*;172.16.*;172.17.*;172.18.*;172.19.*;172.20.*;172.21.*;172.22.*;172.23.*;172.24.*;172.25.*;172.26.*;172.27.*;172.28.*;172.29.*;172.30.*;172.31.*;192.168.*;<local>"
    Write-Host "Proxy enabled."
    Invoke-RestMethod -Headers @{ "Authorization" = "Bearer $apiSecret" } -Method PATCH -Body "{""tun"": {""enable"": false}}" -ContentType "application/json" -Uri "$controllerAPI/configs"
    Write-Host "TUN disabled."
} else {
    # 关闭系统代理, 打开 TUN 模式
    Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 0
    Write-Host "Proxy disabled."
    Invoke-RestMethod -Headers @{ "Authorization" = "Bearer $apiSecret" } -Method PATCH -Body "{""tun"": {""enable"": true}}" -ContentType "application/json" -Uri "$controllerAPI/configs"
    Write-Host "TUN enabled."
}

Start-Sleep -Seconds 1
```

#### Clash Off.ps1

**关闭系统代理和TUN, 停止核心**: 

```shell
# Clash 配置文件设置的 external-controller
$controllerAPI = "http://127.0.0.1:9090"
# Clash 配置文件设置的 external-controller 的 secret
$apiSecret = "1j8Dm52W2nS4aV69Fw15"
# Clash 进程名称
$processName = "ClashMetaCore"

# 关闭系统代理
Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 0
Write-Host "Proxy disabled."

# 检测是否存在 Clash 进程
$clashStatus = Get-Process -Name $processName -ErrorAction SilentlyContinue
if ($clashStatus) {
    # 关闭 TUN 模式
    Invoke-RestMethod -Headers @{ "Authorization" = "Bearer $apiSecret" } -Method PATCH -Body "{""tun"": {""enable"": false}}" -ContentType "application/json" -Uri "$controllerAPI/configs"
    Write-Host "TUN disabled."
    # 停止 Clash 进程
    Stop-Process -Name $processName -Force
    Write-Host "Clash stopped."
} else {
    Write-Host "Clash is not running."
}

# 刷新 DNS 缓存
Clear-DnsClientCache

Start-Sleep -Seconds 1
```

#### Clash Restart.ps1

**重启核心, 打开系统代理**: 

```shell
# Clash 配置文件设置的 mixed-port
$proxyPort = 7890
# Clash 任务计划程序名称
$taskName = "Clash Meta Core"
# Clash 进程名称
$processName = "ClashMetaCore"

# 检测是否存在 Clash 进程
$clashStatus = Get-Process -Name $processName -ErrorAction SilentlyContinue
if ($clashStatus) {
    # 停止 Clash 进程
    Stop-Process -Name $processName -Force
    Write-Host "Clash stopped."
} else {
    Write-Host "Clash is not running."
}

# 启动 Clash 任务计划程序
Start-ScheduledTask -TaskName $taskName
Write-Host "Clash started."

# 打开并设置系统代理
Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 1
Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyServer -Value "127.0.0.1:$proxyPort"
Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyOverride -Value "localhost;127.*;10.*;172.16.*;172.17.*;172.18.*;172.19.*;172.20.*;172.21.*;172.22.*;172.23.*;172.24.*;172.25.*;172.26.*;172.27.*;172.28.*;172.29.*;172.30.*;172.31.*;192.168.*;<local>"
Write-Host "Proxy enabled."

# 刷新 DNS 缓存
Clear-DnsClientCache

Start-Sleep -Seconds 1
```

#### Clash Backup Core.ps1

由于 [Clash.Meta Core Alpha](https://github.com/MetaCubeX/mihomo/releases/tag/Prerelease-Alpha) 更新很快, 手动从 Releases 的 Assets 中下载需要的 Core 并备份到网盘也太麻烦了, 干脆用 PowerShell 脚本自动下载核心到 OneDrive 的文件夹中: 

```shell
$Releaseurl = "https://api.github.com/repos/MetaCubeX/mihomo/releases/tags/Prerelease-Alpha"
$Assets = Invoke-RestMethod -Uri $Releaseurl -Method Get
$AssetUrls = $Assets.assets | Where-Object { $_.name -like "mihomo-linux-amd64*" -or $_.name -like "mihomo-windows-amd64*" } | Select-Object -ExpandProperty browser_download_url

$DefaultDownloadPath = "$env:USERPROFILE\Downloads"
$LinuxBackupPath = "$env:OneDriveCommercial\Software\Linux\Clash.Meta Core Alpha"
$WindowsBackupPath = "$env:OneDriveCommercial\Software\Windows\Clash.Meta Core Alpha"

Get-ChildItem -Path $LinuxBackupPath, $WindowsBackupPath -Recurse | Remove-Item

foreach ($Url in $AssetUrls) {
    $FileName = [System.IO.Path]::GetFileName($Url)

    if($FileName -like "*linux*"){
        $FilePath = "$LinuxBackupPath\$FileName"
    }elseif($FileName -like "*windows*"){
        $FilePath = "$WindowsBackupPath\$FileName"
    }else{
        $FilePath = "$DefaultDownloadPath\$FileName"
    }

    Start-BitsTransfer -Source $Url -Destination $FilePath -DisplayName $FileName -ProxyUsage SystemDefault
}

Write-Host "All done.You can close this window now."
Read-Host
```

{{< admonition type=warning title="注意" open=true >}}
1. 由于使用了 `Start-BitsTransfer`, 也就是 Background Intelligent Transfer Service (BITS) 服务, 你需要确保它没有被禁用。
2. 不要使用中文路径。
{{< /admonition >}}

### 关于脚本

Clash Meta Core 的外部控制 API 不支持开关系统代理, 支持开关 TUN 模式。

脚本的两个功能: 系统代理的开关是通过管理注册表实现的, TUN 模式的开关是通过 [PATCH](https://wiki.metacubex.one/api/#configs) 请求实现的。

#### 权限

为 `.ps1` 文件 (PowerShell 脚本文件) 创建快捷方式, 然后右击文件, “属性→快捷方式→目标”, 在文件路径前添加 `powershell.exe -ExecutionPolicy Bypass -File`, 比如: 

```
powershell.exe -ExecutionPolicy Bypass -File "Clash Off.ps1"
```

然后点击“应用”。

此外, 因为管理任务计划程序和 Windows 服务需要管理员权限, 所以需要让 `.ps1` 文件以管理员身份运行: 在“属性→快捷方式→高级”中勾选“用管理员身份运行”。

{{< admonition type=quote title="或者在脚本开头添加自动提权代码 (不推荐)" open=false >}}
```shell
# 检查是否以管理员权限运行脚本
if (! ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) {
    # 如果没有管理员权限, 则重新以管理员权限运行脚本
    Start-Process powershell -ArgumentList "-NoProfile -ExecutionPolicy Bypass -File `"$PSCommandPath`"" -Verb RunAs
    Exit
}
```
{{< /admonition >}}

不要忘记设置脚本的权限, 让普通用户仅限读取和执行。

这样一路设置下来, 以后只需点击快捷方式即可。然后可以为快捷方式更换图标, 参见 [附件](#附件)。

#### 对于使用 Windows 服务的方案

以上脚本适用于使用任务计划程序的 [方案一](#方案一-任务计划程序), 对于使用 Windows 服务的 [方案二](#方案二-windows服务), 可将脚本中的

```shell
# Clash 任务计划程序名称
$taskName = "Clash Meta Core"

# 启动 Clash 任务计划程序
Start-ScheduledTask -TaskName $taskName
```

替换为

```shell
# Clash 服务的 Name, 不是 DisplayName
$serviceName = "Clash Meta Core Service"

# 启动 Clash 服务
Start-Service -Name $serviceName
```

#### 不代理范围

对于系统代理的不代理 IP 范围: 

```
localhost;127.*;10.*;172.16.*;172.17.*;172.18.*;172.19.*;172.20.*;172.21.*;172.22.*;172.23.*;172.24.*;172.25.*;172.26.*;172.27.*;172.28.*;172.29.*;172.30.*;172.31.*;192.168.*
```

即本地地址 `localhost`, `127.*` 加上 IP 私有地址范围: 

- A类地址范围: 10.0.0.0 ~ 10.255.255.255
- B类地址范围: 172.16.0.0 ~ 172.31.255.555
- C类地址范围: 192.168.0.0 ~ 192.168.255.255

## Clash Premium Core

### 下载

- Core: https://github.com/Dreamacro/clash/releases/tag/premium, 版本选择参考 [FAQ](https://dreamacro.github.io/clash/zh_CN/introduction/faq.html)。将 `clash-windows-amd64-v3.exe` 重命名为 `ClashPremiumCore.exe`
- WinTUN.dll: https://www.wintun.net/
- Dashboard: https://github.com/haishanh/yacd/releases/latest, 解压后将文件夹重命名为 `ui`。

将 `ClashPremiumCore.exe`, `wintun.dll` 文件和 `ui` 文件夹一起放入 Clash 工作目录中, 比如 `D:\Clash Premium`。

### Clash配置文件

在 Clash 工作目录中创建 `config.yaml` 文件, 参考以下配置和官方的 [参考配置](https://dreamacro.github.io/clash/zh_CN/configuration/configuration-reference.html) 写入: 

```yaml
# 官方参考配置 https://dreamacro.github.io/clash/zh_CN/configuration/configuration-reference.html

mixed-port: 7890
allow-lan: false
bind-address: "*"
mode: rule
log-level: silent
ipv6: true
external-controller: 127.0.0.1:9090
secret: 1j8Dm52W2nS4aV69Fw15
# https://github.com/haishanh/yacd/releases/latest
external-ui: ui

profile:
  store-selected: false
  store-fake-ip: false

tun:
  enable: false
  stack: system
  auto-route: true
  auto-detect-interface: true
  dns-hijack:
    - any:53

dns:
  enable: true
  listen: 0.0.0.0:53
  ipv6: true
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  fake-ip-filter:
    - "+.stun.*.*"
    - "+.stun.*.*.*"
    - "+.stun.*.*.*.*"
    - "+.stun.*.*.*.*.*"
    - "*.n.n.srv.nintendo.net"
    - "+.stun.playstation.net"
    - "xbox.*.*.microsoft.com"
    - "+.xboxlive.com"
    - "+.msftncsi.com"
    - "+.msftconnecttest.com"
    - WORKGROUP
    - "connect.rom.miui.com"
    - "connectivitycheck.platform.hicloud.com"
    - "wifi.vivo.com.cn"
    - "www.qualcomm.cn"
    - "+.ntp.org.cn"
    - "+.pool.ntp.org"
    - "ntp.aliyun.com"
    - "ntp.ntsc.ac.cn"
    - "ntp.tencent.com"
    - "time.apple.com"
    - "time.windows.com"
    - "+.edu.cn"
  default-nameserver:
    - 223.5.5.5
    - 8.8.8.8
  # 更多 DNS 参考: https://senzyo.net/2022-22/
  nameserver:
    - tls://223.5.5.5
    - tls://1.12.12.12
  fallback:
    - tls://1.1.1.1
    - tls://8.8.8.8
    - tls://9.9.9.9
  fallback-filter:
    geoip: true
    geoip-code: CN

proxy-groups:
  - name: ✈️ 节点选择
    type: select
    disable-udp: false
    proxies:
      - 🌏 日韩台新
      - DIRECT
      - 📌 手动切换
      - 🇭🇰 香港节点
      - 🇯🇵 日本节点
      - 🇰🇷 韩国节点
      - 🇹🇼 台湾节点
      - 🇸🇬 新加坡节点
      - 🇺🇸 美国节点
  - name: 🌏 日韩台新
    type: url-test
    disable-udp: false
    # 测试节点延迟的时间间隔, 单位为 s, 此处仅影响 proxies 下的节点, 不影响 use 下的节点
    # interval: 300
    # lazy 默认为 true, 未选择到当前策略组时,则不会进行测试
    # lazy: false
    # 当 (某一节点的延迟) < (当前节点的延迟-容差) 时, 才会切换到新的节点。默认值为 0ms, 可选项
    # tolerance: 20
    url: "https://www.gstatic.com/generate_204"
    filter: "🇯🇵|日本|JP|Japan|🇰🇷|韩国|KR|South Korea|🇹🇼|台湾|TW|Taiwan|🇸🇬|新加坡|SG|Singapore"
    use:
      - Provider
  - name: 📌 手动切换
    type: select
    disable-udp: false
    use:
      - Provider
  - name: 🤖 OpenAI
    type: select
    disable-udp: false
    proxies:
      - 🌏 日韩台新
      - 📌 手动切换
      - 🇯🇵 日本节点
      - 🇰🇷 韩国节点
      - 🇹🇼 台湾节点
      - 🇸🇬 新加坡节点
      - 🇺🇸 美国节点
  - name: ☁️ OneDrive
    type: select
    disable-udp: false
    proxies:
      - DIRECT
      - 🌏 日韩台新
      - 📌 手动切换
      - 🇭🇰 香港节点
      - 🇯🇵 日本节点
      - 🇰🇷 韩国节点
      - 🇹🇼 台湾节点
      - 🇸🇬 新加坡节点
      - 🇺🇸 美国节点
  - name: 🎮 游戏平台
    type: select
    disable-udp: false
    proxies:
      - 🌏 日韩台新
      - DIRECT
      - 📌 手动切换
      - 🇭🇰 香港节点
      - 🇯🇵 日本节点
      - 🇰🇷 韩国节点
      - 🇹🇼 台湾节点
      - 🇸🇬 新加坡节点
      - 🇺🇸 美国节点
  - name: 🐟 漏网之鱼
    type: select
    disable-udp: false
    proxies:
      - 🌏 日韩台新
      - DIRECT
      - 📌 手动切换
      - 🇭🇰 香港节点
      - 🇯🇵 日本节点
      - 🇰🇷 韩国节点
      - 🇹🇼 台湾节点
      - 🇸🇬 新加坡节点
      - 🇺🇸 美国节点
  - name: 🇭🇰 香港节点
    type: url-test
    disable-udp: false
    url: "https://www.gstatic.com/generate_204"
    filter: "🇭🇰|香港|HK|Hong Kong"
    use:
      - Provider
  - name: 🇯🇵 日本节点
    type: url-test
    disable-udp: false
    url: "https://www.gstatic.com/generate_204"
    filter: "🇯🇵|日本|JP|Japan"
    use:
      - Provider
  - name: 🇰🇷 韩国节点
    type: url-test
    disable-udp: false
    url: "https://www.gstatic.com/generate_204"
    filter: "🇰🇷|韩国|KR|South Korea"
    use:
      - Provider
  - name: 🇹🇼 台湾节点
    type: url-test
    disable-udp: false
    url: "https://www.gstatic.com/generate_204"
    filter: "🇹🇼|台湾|TW|Taiwan"
    use:
      - Provider
  - name: 🇸🇬 新加坡节点
    type: url-test
    disable-udp: false
    url: "https://www.gstatic.com/generate_204"
    filter: "🇸🇬|新加坡|SG|Singapore"
    use:
      - Provider
  - name: 🇺🇸 美国节点
    type: url-test
    disable-udp: false
    url: "https://www.gstatic.com/generate_204"
    filter: "🇺🇸|美国|US|USA|United States"
    use:
      - Provider

rules:
  - DOMAIN-SUFFIX,mangafuna.xyz,✈️ 节点选择
  - RULE-SET,Advertising_Classical_No_Resolve,REJECT
  - RULE-SET,Lan_No_Resolve,DIRECT
  - RULE-SET,NTPService,DIRECT
  - RULE-SET,PrivateTracker,DIRECT
  - RULE-SET,OpenAI,🤖 OpenAI
  - RULE-SET,OneDrive,☁️ OneDrive
  - RULE-SET,Game,🎮 游戏平台
  - RULE-SET,Global_Classical,✈️ 节点选择
  - RULE-SET,CloudCN,DIRECT
  - RULE-SET,ChinaMax_Classical,DIRECT
  - MATCH,🐟 漏网之鱼

proxy-providers:
  Provider:
    type: http
    # 因为 clash premium core 更新 providers 时不带 User-Agent, 会造成机场返回的订阅信息可能不符合 clash 的格式。可以在订阅连接后添加 `&flag=clash` 来指定。
    url: "https://example.com/xxx&flag=clash"
    # 更新订阅的时间间隔, 单位为 s
    interval: 43200
    path: ./Provider.yaml
    health-check:
      enable: true
      # 测试节点延迟的时间间隔, 单位为 s, 此处仅影响 proxy-group 中 use 下的节点
      interval: 600
      lazy: false
      url: https://www.gstatic.com/generate_204

rule-providers:
  Advertising_Classical_No_Resolve:
    type: http
    behavior: classical
    url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/Advertising/Advertising_Classical_No_Resolve.yaml
    path: ./rule-providers/Advertising_Classical_No_Resolve.yaml
    # 更新规则的时间间隔, 单位为 s
    interval: 604800
  Lan_No_Resolve:
    type: http
    behavior: classical
    url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/Lan/Lan_No_Resolve.yaml
    path: ./rule-providers/Lan_No_Resolve.yaml
    interval: 604800
  NTPService:
    type: http
    behavior: classical
    url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/NTPService/NTPService.yaml
    path: ./rule-providers/NTPService.yaml
    interval: 604800
  PrivateTracker:
    type: http
    behavior: classical
    url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/PrivateTracker/PrivateTracker.yaml
    path: ./rule-providers/PrivateTracker.yaml
    interval: 604800
  OpenAI:
    type: http
    behavior: classical
    url: https://ghproxy.net/https://raw.githubusercontent.com/senzyo/as-gist/master/Rule/OpenAI/OpenAI.yaml
    path: ./rule-providers/OpenAI.yaml
    interval: 604800
  OneDrive:
    type: http
    behavior: classical
    url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/OneDrive/OneDrive.yaml
    path: ./rule-providers/OneDrive.yaml
    interval: 604800
  Game:
    type: http
    behavior: classical
    url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/Game/Game.yaml
    path: ./rule-providers/Game.yaml
    interval: 604800
  Global_Classical:
    type: http
    behavior: classical
    url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/Global/Global_Classical.yaml
    path: ./rule-providers/Global_Classical.yaml
    interval: 604800
  CloudCN:
    type: http
    behavior: classical
    url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/Cloud/CloudCN/CloudCN.yaml
    path: ./rule-providers/CloudCN.yaml
    interval: 604800
  ChinaMax_Classical:
    type: http
    behavior: classical
    url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/ChinaMax/ChinaMax_Classical.yaml
    path: ./rule-providers/ChinaMax_Classical.yaml
    interval: 86400
```

其余 Dashboard、开机自启动、防火墙的设置参考上文 Clash Meta Core 的设置即可。

### 脚本

Clash Premium Core 的外部控制 API 不支持开关系统代理和 TUN 模式。

脚本的两个功能: 系统代理的开关是通过管理注册表实现的, TUN 模式的开关是通过改写 Clash 配置文件中 `tun.enable` 的值, 然后重启 Clash 核心实现的。

#### Clash Switch.ps1

**切换系统代理和TUN, 重启核心**: 

```shell
# Clash 配置文件设置的 mixed-port
$proxyPort = 7890
# Clash 配置文件路径
$filePath = "D:\Clash Premium\config.yaml"
# Clash 配置文件中 tun.enable 所处行数
$lineNumberToModify = 19
# Clash 任务计划程序名称
$taskName = "Clash Premium Core"
# Clash 进程名称
$processName = "ClashPremiumCore"

# 检查系统代理开关状态
$proxyEnabled = (Get-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings').ProxyEnable
if ($proxyEnabled -eq 0) {
    # 打开并设置系统代理, 关闭 TUN 模式
    Write-Host "Enable Proxy. Disable TUN."
    Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 1
    Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyServer -Value "127.0.0.1:$proxyPort"
    Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyOverride -Value "localhost;127.*;10.*;172.16.*;172.17.*;172.18.*;172.19.*;172.20.*;172.21.*;172.22.*;172.23.*;172.24.*;172.25.*;172.26.*;172.27.*;172.28.*;172.29.*;172.30.*;172.31.*;192.168.*;<local>"
    $replacementLine = "  enable: false"
} else {
    # 关闭系统代理, 打开 TUN 模式
    Write-Host "Disable Proxy. Enable TUN."
    Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 0
    $replacementLine = "  enable: true"
}

# 将 TUN 的开关状态写入 Clash 配置文件
$fileContent = Get-Content -Path $filePath -Encoding UTF8
$fileContent[$lineNumberToModify - 1] = $replacementLine
$fileContent | Out-File -FilePath $filePath -Encoding UTF8

# 检测是否存在 Clash 进程
$clashStatus = Get-Process -Name $processName -ErrorAction SilentlyContinue
if ($clashStatus) {
    # 停止 Clash 进程
    Stop-Process -Name $processName -Force
    Write-Host "Clash stopped."
} else {
    Write-Host "Clash is not running."
}

# 启动 Clash 任务计划程序
Start-ScheduledTask -TaskName $taskName
Write-Host "Clash started."

# 刷新 DNS 缓存
Clear-DnsClientCache

Start-Sleep -Seconds 1
```

#### Clash Off.ps1

**关闭系统代理和TUN, 停止核心**: 

```shell
# Clash 配置文件路径
$filePath = "D:\Clash Premium\config.yaml"
# Clash 配置文件中 tun.enable 所处行数
$lineNumberToModify = 19
# Clash 进程名称
$processName = "ClashPremiumCore"

# 检查系统代理开关状态
$proxyEnabled = (Get-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings').ProxyEnable
if ($proxyEnabled -eq 0) {
    Write-Host "Proxy already disabled."
} else {
    # 关闭系统代理
    Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 0
    Write-Host "Proxy disabled."
}

# 关闭 TUN 模式
$fileContent = Get-Content -Path $filePath -Encoding UTF8
$fileContent[$lineNumberToModify - 1] = "  enable: false"
$fileContent | Out-File -FilePath $filePath -Encoding UTF8
Write-Host "TUN disabled."

# 检测是否存在 Clash 进程
$clashStatus = Get-Process -Name $processName -ErrorAction SilentlyContinue
if ($clashStatus) {
    # 停止 Clash 进程
    Stop-Process -Name $processName -Force
    Write-Host "Clash stopped."
} else {
    Write-Host "Clash is not running."
}

# 刷新 DNS 缓存
Clear-DnsClientCache

Start-Sleep -Seconds 1
```

#### 编码

[PowerShell 7](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.management/set-content?view=powershell-7.2#-encoding) 默认使用 `utf8NoBOM` 编码, 而 [PowerShell 5](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.management/set-content?view=powershell-5.1#-encoding) 默认使用 `Default` (通常是 `ANSI`), 即使指定了 `UTF-8`, 也是 `UTF-8 with BOM`。所以 `config.yaml` 可能会被保存为 `UTF-8 with BOM` 编码, 不过无影响。

#### 对于使用 Windows 服务的方案

以上脚本适用于使用任务计划程序的 [方案一](#方案一-任务计划程序), 对于使用 Windows 服务的 [方案二](#方案二-windows服务), 可将脚本中的

```shell
# Clash 任务计划程序名称
$taskName = "Clash Premium Core"

# 启动 Clash 任务计划程序
Start-ScheduledTask -TaskName $taskName
```

替换为

```shell
# Clash 服务的 Name, 不是 DisplayName
$serviceName = "Clash Premium Core Service"

# 启动 Clash 服务
Start-Service -Name $serviceName
```

## FAQ

### Core无法启动

如果不能启动, 请尝试更换版本, Clash Premium Core 版本选择参考 [FAQ](https://dreamacro.github.io/clash/zh_CN/introduction/faq.html), Clash Meta Core 版本选择参考 [FAQ](https://wiki.metacubex.one/startup/faq/)。

### 多个Core正在运行

在 MetaCubeXD 中点击了“重启核心”和“更新核心”按钮, 这时 Clash Core 会重启, Clash Core 的任务计划程序或者 Windows 服务转为就绪状态, 如果这时再手动启动 Clash Core 的任务计划程序或者 Windows 服务, 就会有两个 Clash Core 进程并存。

本文脚本中有重启 Clash Core 的任务计划程序或者 Windows 服务的操作, 所以 MetaCubeXD 搭配本文脚本使用不会出现此问题。

### PowerShell脚本无法运行

参考微软官方文档 [Set-ExecutionPolicy](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.security/set-executionpolicy), 以管理员身份运行 PowerShell, 执行以下命令: 

```shell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine
```

### UWP不走代理

部分应用没有启用本地网络回环功能, 特别是 UWP 应用, 默认是关闭网络回环的。使用 [AppContainer Loopback Exemption Utility](https://www.telerik.com/fiddler/add-ons) 来开启这些应用的本地网络回环功能, 使其流量经由系统代理。

## 附件

几个用于脚本快捷方式的自制图标:

<table>
  <tr>
    <td align="center"><img src="./Clash Backup Core.svg" width="100px" /></td>
    <td align="center"><img src="./Clash Off.svg" width="100px" /></td>
    <td align="center"><img src="./Clash Restart.svg" width="100px" /></td>
    <td align="center"><img src="./Clash Switch.svg" width="100px" /></td>
  </tr>
</table>
