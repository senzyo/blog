---
title: "Android刷机笔记"
date: 2022-12-01T10:00:00+08:00
draft: false
authors: ["senzyo"]
tags: [Android,Custom]
featuredImagePreview: ""
summary: 这是一篇许久以前的刷机笔记。
---

{{< admonition type=tip title="注意" open=true >}}
本文操作机型为 Xiaomi Redmi Note 5(whyred), 不支持 A/B 分区。
{{< /admonition >}}

## Official Tools

- 官方解锁工具: https://www.miui.com/unlock/download.html
- 官方线刷工具: http://bigota.d.miui.com/tools/MiFlash2018-5-28-0.zip

这两个工具的目录下一般有连接手机的相关驱动, 记得安装。

## Recovery

- TWRP: https://twrp.me/xiaomi/xiaomiredminote5pro.html, 不支持动态分区
- OrangeFox: https://sourceforge.net/projects/whyred-orangefox/files/TEST/, 只有使用 4.19 内核的版本支持动态分区
- LineageOS Recovery: https://t.me/noperoms/8125, 支持动态分区

对于 OrangeFox: 

- 选择与 ROM 内核对应的 Recovery
- 先刷入从 ZIP 包中解压得到的 recovery.img, 进入 Recovery 模式的 OrangeFox 后, 再刷入整个 ZIP 包

刷入命令: 

```shell
adb reboot bootloader
fastboot devices
fastboot flash recovery twrp.img
fastboot reboot
```

## File System

### Format Data and Cache to F2FS

一些 ROM 对于文件系统, 强制要求使用 `F2FS`, 如果使用了 `ext4`, 将会卡在开机界面, 或者循环开机。

可使用 TWRP 和 OrangeFox 等 Recovery 来格式化, 也可使用 fastboot 来完成。

通过 `fastboot --help` 发现可借助以下命令格式化指定分区: 

```shell
fastboot format[:FS_TYPE[:SIZE]] PARTITION
```

其中 `FS_TYPE` 是对应的文件系统, `PARTITION` 则是指定的分区。

将 `userdata` 分区格式化成 `f2fs`: 

```shell
fastboot format:f2fs userdata
```

将 `cache` 分区格式化成 `f2fs`: 

```shell
fastboot format:f2fs cache
```

#### Format Cache Error

如果你的设备在格式化 `cache` 分区时报错: 

```shell
# Fastboot
FAILED (remote: 'GetVar Variable Not found')
fastboot: error: Command failed

# TWRP
Cache 无效的分区
```

不要慌张, 有可能是你的设备没有划分 `cache` 分区。Google 对于 Android 各分区的 [解释](https://source.android.google.cn/docs/core/architecture/bootloader/partitions?hl=zh-cn) 中提到: 

{{< admonition type=quote title="解释" open=true >}}
`cache` 分区。此分区会存储临时数据, 如果设备使用 A/B (无缝) 更新, 则此分区是可选的。
{{< /admonition >}}

那么之后所有关于 `cache` 分区的操作直接跳过就好。

### Dynamic Partition

一些使用 4.19 内核的类原生 ROM, 比如 Evolution-X 和 LineageOS, 都使用了动态分区, 需要使用支持动态分区的 Recovery 来刷入 ROM, 比如上文 [Recovery](#recovery) 中的 [OrangeFox](https://sourceforge.net/projects/whyred-orangefox/files/TEST/) 和 [LineageOS Recovery](https://t.me/noperoms/8125)。

刷入 ROM 时, 如果出现有关分区的报错, 忽视即可, 因为此时分区尚未设为动态类型。

## ROM

### AOSP Similar

- Arrow: https://arrowos.net/download, 已停止维护
- crDroid: https://crdroid.net/whyred/7, 已停止维护
- DerpFest: https://sourceforge.net/projects/derpfest/files/whyred/
- DotOS: https://www.droidontime.com/devices/whyred, 已停止维护
- Evolution-X: https://evolution-x.org/device/whyred, https://sourceforge.net/projects/evolution-x/files/whyred/13/
- Havoc-OS: https://havoc-os.com/device#whyred, 已停止维护
- LineageOS: ~~[Official](https://download.lineageos.org/whyred)~~ 已撤包, [Unofficial](https://t.me/s/whyredupdates?q=LineageOS)
- PixelExperience: https://get.pixelexperience.org/whyred, 已停止维护
- PixelOS: https://pixelos.net/download/whyred, 已停止维护

更多系统参见 Telegram 频道: https://t.me/whyredupdates

### MIUI

- 中国: https://xiaomirom.com/rom/redmi-note-5-note-5-pro-whyred-global-fastboot-recovery-rom/
- 欧版: https://sourceforge.net/projects/xiaomi-eu-multilang-miui-roms/files/xiaomi.eu/MIUI-STABLE-RELEASES/

## Firmware

https://xiaomifirmwareupdater.com/firmware/whyred/

## KernelSU

相比 Magisk, 更推荐使用 [KernelSU](https://kernelsu.org/zh_CN/)。

这部手机的大部分 ROM 使用的内核都不支持 KernelSU, 不过可以刷入支持 KernelSU 的内核, 比如 [San-Kernel](https://github.com/Whyred-Dynamic-Development/san_kernel_sdm660_4/releases)。

对于 San-Kernel 内核文件名中的代号:

- `qti` 和 `qpnp` 的区别在于振动, 首选 `qti`
- `ksu` 表示内置了 KernelSU
- `oc` 表示超频
 
所以推荐使用 `San-Kernel-4.19-whyred-oldcam+qti-ksu-oc.zip`。

在刷入 ROM 成功后, 顺便刷入内核, 再顺便刷入 [GApps](#google-apps-1), 使用 `adb sideload` 刷入或在 Recovery 中滑动刷入均可。

### Zygisk

KernelSU 本体不支持 Zygisk, 但是可以安装 [ZygiskNext](https://github.com/Dr-TSNG/ZygiskNext/releases/latest) 模块来支持 Zygisk 模块, LSPosed 可以在 ZygiskNext 的支持下正常运行。

### Hide Root

1. 在 KernelSU 中安装 [Zygisk-LSPosed](https://github.com/LSPosed/LSPosed/releases/latest) 模块, 然后重启手机
2. 安装 [隐藏应用列表](https://github.com/Dr-TSNG/Hide-My-Applist/releases/latest), 在 LSPosed 管理器中将其启用, 然后重启手机
3. 配置隐藏应用列表, 设置黑名单。将 KernelSU 管理器和 LSPosed 模块以及一些使用 root 权限的敏感软件都列入隐藏名单, 应用于某些软件(银行之类)

## Magisk

{{< admonition type=question title="KernelSU 与 Magisk 兼容吗" open=true >}}
KernelSU 的模块系统与 Magisk 的 magic mount 有冲突, 如果 KernelSU 中启用了任何模块, 那么整个 Magisk 将无法工作。

但是如果你只使用 KernelSU 的 `su`, 那么它会和 Magisk 一起工作: KernelSU 修改 `kernel` 、 Magisk 修改 `ramdisk`, 它们可以一起工作。
{{< /admonition >}}

官方文档: https://topjohnwu.github.io/Magisk/install.html

Magisk 有三种: 

- Magisk 原版: https://github.com/topjohnwu/Magisk/releases/latest
- Magisk Alpha: https://t.me/s/magiskalpha?q=app-release.apk
- Magisk Delta: https://github.com/HuskyDG/magisk-files/releases/latest

### Hide Root

1. 使用 Magisk 的 `隐藏 Magisk` 功能(随机包名)。由于上游文件位于 GitHub, 所以使用此功能时最好启用网络代理

2. 在 Magisk 中 `启用 Zygisk`。如果是 Magisk 原版和 Magisk Alpha, 可在安装 Shamiko 模块后再重启

3. 在 Magisk 中安装 Shamiko 模块, 然后重启手机
   
   - Magisk 原版: 直接安装 Shamiko [最新版](https://github.com/LSPosed/LSPosed.github.io/releases/latest) 即可
   - Magisk Alpha: 安装最新版的 [Magisk Alpha](https://t.me/s/magiskalpha?q=app-release) 和 [Shamiko](https://t.me/s/magiskalpha?q=Shamiko)
   - Magisk Delta: 自带 Magisk Hide, 所以不用安装 Shamiko

4. Shamiko 默认使用黑名单模式, **不要**打开 Magisk 的 `遵守排除列表` 开关, 不然会冲突
5. 使用 Magisk 的 `配置排除列表` 功能。勾选需要对其隐藏 root 的应用(比如银行之类)的全部进程
6. 在 Magisk 中安装 [Zygisk-LSPosed](https://github.com/LSPosed/LSPosed/releases/latest) 模块, 然后重启手机
7. 安装 [隐藏应用列表](https://github.com/Dr-TSNG/Hide-My-Applist/releases/latest), 在 LSPosed 管理器中将其启用, 然后重启手机
8. 配置隐藏应用列表, 设置黑名单。将 Magisk 和 LSPosed 模块以及一些使用 root 权限的敏感软件都列入隐藏名单, 应用于某些软件(银行之类)

## Detect Root

https://github.com/apkunpacker/MagiskDetection

也就 [Momo](https://t.me/s/magiskalpha?q=momo) 和 [NativeTest](https://t.me/s/magiskalpha?q=NativeTest) 这两个软件的检测够强力, 别的不太行。

## LSPosed Modules

安装 LSPosed Modules 的前提是已经在 Magisk 或 KernelSU 中安装了 [Zygisk-LSPosed](https://github.com/LSPosed/LSPosed/releases/latest) 模块。

- [InstallerPlus](https://github.com/NextAlone/InstallerPlus/releases/latest), 在安装和卸载软件时提供更多信息
- [灭霸](https://github.com/Tornaco/Thanox/releases/latest), 调教软件
- [隐藏应用列表](https://github.com/Dr-TSNG/Hide-My-Applist/releases/latest), 使一些应用对另一些应用隐藏, 比如隐藏 root 应用和 LSPosed 模块
- [知了](https://github.com/shatyuka/Zhiliao/releases/latest), 精简知乎

## Check Info

使用 [设备信息](https://www.coolapk.com/apk/com.liuzh.deviceinfo) APP 查看比较快捷。

### Baseband Info

```shell
adb shell getprop gsm.version.baseband
```

### Fingerprint Info

```shell
adb shell getprop ro.build.fingerprint
```

### Kernel Info

```shell
adb shell cat /proc/version
```

### SELinux

```shell
adb shell getenforce
```

若状态是 `Permissive`, 可将状态变成 `Enforcing`: 

```shell
adb shell
su root
setenforce 1
```

## Safetynet

参考 [这篇文章](https://blog.csdn.net/a1240193326/article/details/113866931), 一般先使用 [Universal SafetyNet Fix](https://github.com/kdrag0n/safetynet-fix/releases/latest), 不行的话再上 [MagiskHidePropsConf](https://github.com/Magisk-Modules-Repo/MagiskHidePropsConf/releases/latest), 再不行就用 [XPrivacyLua](http://repo.xposed.info/module/eu.faircode.xlua)。

Play 商店设备未通过认证, 可 [手动注册](https://www.google.com/android/uncertified/), 参考少数派的这篇 [文章](https://sspai.com/post/55536), 偶尔有用。

## Customize MIUI

### Preinstalled Apps

将以下软件包名保存到文件中, 比如 `list.txt`: 

```
com.miui.contentextension 
com.miui.fm 
com.mi.liveassistant 
com.xiaomi.vipaccount 
com.xiaomi.gamecenter.sdk.service 
com.android.quicksearchbox 
com.xiaomi.payment 
com.android.browser 
com.miui.systemAdSolution 
com.baidu.input_mi 
com.miui.userguide 
com.xiaomi.ab 
com.xiaomi.jr 
com.miui.huanji 
com.miui.hybrid 
com.mi.health 
com.mfashiongallery.emag 
com.miui.miservice 
com.miui.virtualsim 
com.miui.personalassistant 
com.miui.bugreport 
com.miui.voicetrigger 
com.mipay.wallet 
com.xiaomi.pass 
com.xiaomi.shop 
com.iflytek.inputmethod.miui 
com.miui.newhome 
com.miui.video 
com.miui.hybrid.accessory 
com.sohu.inputmethod.sogou.xiaomi 
com.miui.touchassistant 
com.xiaomi.youpin 
com.miui.securityinputmethod 
com.xiaomi.gamecenter 
com.miui.analytics 
com.xiaomi.scanner 
com.duokan.reader 
com.caf.fmradio 
com.miui.voiceassist 
com.sina.weibolite 
com.miui.smarttravel 
com.miui.fmservice 
com.xiaomi.mimobile.noti 
com.xiaomi.smarthome 
com.xiaomi.drivemode 
com.miui.newmidrive 
com.xiaomi.aiasst.service 
com.android.stk 
com.android.email 
```

这些软件的名称是: 

| 软件名称         | 软件包名                          |
| ---------------- | --------------------------------- |
| 传送门           | com.miui.contentextension         |
| 收音机           | com.miui.fm                       |
| 小米直播助手     | com.mi.liveassistant              |
| 小米社区         | com.xiaomi.vipaccount             |
| 游戏服务         | com.xiaomi.gamecenter.sdk.service |
| 搜索             | com.android.quicksearchbox        |
| 米币支付         | com.xiaomi.payment                |
| 浏览器           | com.android.browser               |
| msa              | com.miui.systemAdSolution         |
| 百度输入法小米版 | com.baidu.input_mi                |
| 用户手册         | com.miui.userguide                |
| mab              | com.xiaomi.ab                     |
| 天星金融         | com.xiaomi.jr                     |
| 小米换机         | com.miui.huanji                   |
| 快应用服务框架   | com.miui.hybrid                   |
| 健康             | com.mi.health                     |
| 小米画报         | com.mfashiongallery.emag          |
| 服务与反馈       | com.miui.miservice                |
| 全球上网         | com.miui.virtualsim               |
| 智能助理         | com.miui.personalassistant        |
| 用户反馈         | com.miui.bugreport                |
| 语音唤醒         | com.miui.voicetrigger             |
| 小米钱包         | com.mipay.wallet                  |
| 小米卡包         | com.xiaomi.pass                   |
| 小米商城         | com.xiaomi.shop                   |
| 讯飞输入法小米版 | com.iflytek.inputmethod.miui      |
| 内容中心         | com.miui.newhome                  |
| 小米视频         | com.miui.video                    |
| HybridAccessory  | com.miui.hybrid.accessory         |
| 搜狗输入法小米版 | com.sohu.inputmethod.sogou.xiaomi |
| 悬浮球           | com.miui.touchassistant           |
| 小米有品         | com.xiaomi.youpin                 |
| 小米安全键盘     | com.miui.securityinputmethod      |
| 游戏中心         | com.xiaomi.gamecenter             |
| Analytics        | com.miui.analytics                |
| 扫一扫           | com.xiaomi.scanner                |
| 阅读             | com.duokan.reader                 |
| f_m_test         | com.caf.fmradio                   |
| 小爱同学         | com.miui.voiceassist              |
| 微博极速版       | com.sina.weibolite                |
| 智能出行         | com.miui.smarttravel              |
| 收音机           | com.miui.fmservice                |
| 全球上网插件     | com.xiaomi.mimobile.noti          |
| 米家             | com.xiaomi.smarthome              |
| 驾车场景         | com.xiaomi.drivemode              |
| 小米云盘         | com.miui.newmidrive               |
| AI虚拟助手       | com.xiaomi.aiasst.service         |
| USIM卡应用       | com.android.stk                   |
| 电子邮箱         | com.android.email                 |

#### Auto Script

批量卸载 [预装软件](#preinstalled-apps): 

```bash
cat list.txt | cut -d ':' -f 2 | tr '\r' ' ' | xargs -r -n1 -t adb shell pm uninstall --user 0 
```

或者批量禁用 [预装软件](#preinstalled-apps): 

```bash
cat list.txt | cut -d ':' -f 2 | tr '\r' ' ' | xargs -r -n1 -t adb shell pm disable-user 
```

命令解释: 

- `cut -d ':' -f 2`: 使用 : 分隔符来切割输出, 并只保留第二列, 也就是包名
- `tr '\r' ' '`: 将行末的 `\r` 字符替换为空格, 以便在 `xargs` 中使用换行符作为分隔符
- `xargs`: 将前面的输出作为参数传递给后面的命令
- `-r` 如果没有输入, 则不运行命令
- `-n1`: 将每个参数单独传递给 `adb xxx` 命令
- `-t`: 在运行每个命令之前先打印命令本身

### Google Apps

从 ApkMirror 下载安装 [Google Services Framework](https://www.apkmirror.com/apk/google-inc/google-services-framework/), [Google Play Services](https://www.apkmirror.com/apk/google-inc/google-play-services/), [Google Play Store](https://www.apkmirror.com/apk/google-inc/google-play-store/) 即可。

{{< admonition type=warning title="注意" open=true >}}
- MIUI 魔改了 Android 系统 API, 会导致 apks(apkm) 安装失败, 就算用 root 权限安装也容易出问题, 所以在 ApkMirror 中下载软件时, 不要下载带有 split 的 apkm, 要下载 apk。
- 从 Recovery 中刷入 GApps 可能出现 Play Store 闪退的情况。
{{< /admonition >}}

### Bluetooth AAC

MIUI 的蓝牙 AAC 黑白名单文件路径为 `/system/etc/bluetooth/interop_database.conf`, 比如将 OPPO Enco Air 添加至白名单: 

```
[INTEROP_ENABLE_AAC_CODEC]
OPPO Enco Air = Name_Based
# Address_Based除对应蓝牙耳机MAC地址前六位
48:D8:45 = Address_Based
```

## Customize LineageOS

### Google Apps

可在 Recovery 中通过 `adb sideload` 刷入 GApps, 推荐 [BiTGApps](https://bitgapps.io/), 也可选择 [Mind The Gapps](https://androidfilehost.com/?w=files&flid=322935) 或 [The Open GApps Project](https://opengapps.org/#downloadsection)。

{{< admonition type=warning title="注意" open=true >}}
成功刷入 ROM 后, 先不要开机, 要在这时刷入 GApps。若是不小心开了机, 只能恢复出厂设置后再刷入 GApps。因为 `SetupWizard` 需要在设备首次启动时运行。
{{< /admonition >}}

不推荐在 Magisk 或 KernelSU 中将 GApps 的 zip 作为模块刷入, 除非有过成功经验。

## ADB Shell Command

进入 Android Shell:

```shell
adb shell
```

### Package Manager  

常用命令: 

```shell
# 列出设备中已经安装的所有应用
pm list packages
# 列出包名中包含关键字的应用的完整包名
pm list packages "keyword"
# 列出所有已禁用的应用
pm list packages -d
# 列出所有已启用的应用
pm list packages -e
# 列出所有系统应用
pm list packages -s
# 列出所有第三方应用
pm list packages -3

# 列出系统上所有的用户
pm list users

# 列出该应用的所有数据位置
pm path package_name

# 清除指定应用的所有数据
pm clear package_name

# 卸载应用
# -k: 卸载应用且保留数据与缓存(不加-k则全部删除)
# --user 0: 为当前用户卸载
pm uninstall [-k] [--user 0] package_name

# 禁用应用
pm disable-user package_name
# 启用应用
pm enable package_name
```

更多 Package Manager 命令参见 https://developer.android.com/studio/command-line/adb?hl=zh-cn#pm

### NTP Server

```shell
settings put global ntp_server ntp.aliyun.com
# settings put global ntp_server cn.ntp.org.cn
# settings put global ntp_server cn.pool.ntp.org
# settings put global ntp_server ntp.ntsc.ac.cn
# settings put global ntp_server ntp.tencent.com
# settings put global ntp_server time.apple.com
# settings put global ntp_server time.windows.com
```

设置完成后, 即使重启仍然有效, 系统更新后是否仍有效暂未检查。

检查配置: 

```shell
settings get global ntp_server
# 或
settings list global | grep ntp
```

### Captive Portal

```shell
# 启用 Captive Portal
settings put global captive_portal_mode 1
# 使用 HTTPS 检测
settings put global captive_portal_use_https 1
# 设置检测地址
settings put global captive_portal_http_url http://connectivitycheck.platform.hicloud.com/generate_204
settings put global captive_portal_https_url https://connectivitycheck.platform.hicloud.com/generate_204
```

或使用其他检测地址: 

```shell
# Xiaomi
settings put global captive_portal_http_url http://connect.rom.miui.com/generate_204
settings put global captive_portal_https_url https://connect.rom.miui.com/generate_204
# Vivo
settings put global captive_portal_http_url http://wifi.vivo.com.cn/generate_204
settings put global captive_portal_https_url https://wifi.vivo.com.cn/generate_204
```

设置完成后, 即使重启仍然有效, 系统更新后是否仍有效暂未检查。

检查配置: 

```shell
settings list global | grep captive
```

## Apps

罗列一些常用软件。

1. [ADM](https://play.google.com/store/apps/details?id=com.dv.adm)
2. [All-In-One Calculator](https://play.google.com/store/apps/details?id=all.in.one.calculator)
3. [Bitwarden](https://play.google.com/store/apps/details?id=com.x8bit.bitwarden)
4. [Breezy-Weather](https://github.com/breezy-weather/breezy-weather/releases/latest), [几何天气](https://github.com/WangDaYeeeeee/GeometricWeather) 的二次开发
5. [Clash Meta for Android](https://github.com/MetaCubeX/ClashMetaForAndroid/releases)
6. [F-Droid](https://f-droid.org/zh_Hans/)
7. [Fake Location](https://github.com/Lerist/FakeLocation/releases/latest)
8.  [Fcitx5 for Android](https://f-droid.org/zh_Hans/packages/org.fcitx.fcitx5.android/)
9.  [Gboard](https://www.apkmirror.com/apk/google-inc/gboard/), 推荐使用 Gboard Lite
10. [GitHub](https://play.google.com/store/apps/details?id=com.github.android)
11. [Google Android System WebView](https://www.apkmirror.com/apk/google-inc/android-system-webview/)
12. [Google TTS](https://play.google.com/store/apps/details?id=com.google.android.tts)
13. [LibChecker](https://play.google.com/store/apps/details?id=com.absinthe.libchecker)
14. [LineageOS Camera](https://www.apkmirror.com/apk/lineageos/lineageos-camera-4/camera-60-13-release/)
15. [LineageOS Phone](https://www.apkmirror.com/apk/lineageos/lineage-os-phone/lineage-os-phone-23-0-release/)
16. [LineageOS Recorder](https://www.apkmirror.com/apk/lineageos/recorder/recorder-13-release/)
17. [LocalSend](https://play.google.com/store/apps/details?id=org.localsend.localsend_app)
18. [Microsoft Onedrive](https://play.google.com/store/apps/details?id=com.microsoft.skydrive)
19. [Microsoft Outlook Lite](https://play.google.com/store/apps/details?id=com.microsoft.outlooklite)
20. [Microsoft To Do](https://play.google.com/store/apps/details?id=com.microsoft.todos)
21. [MT管理器](https://www.coolapk.com/apk/bin.mt.plus)
22. [PicMarker](https://play.google.com/store/apps/details?id=com.github.kolacbb.picmarker)
23. [SAI](https://f-droid.org/zh_Hans/packages/com.aefyr.sai.fdroid/), 安装含有 splits 的 apkm 和 apks 格式的安装包
24. [Simple Contacts Pro](https://f-droid.org/zh_Hans/packages/com.simplemobiletools.contacts.pro)
25. [Simple Dialer](https://f-droid.org/zh_Hans/packages/com.simplemobiletools.dialer/)
26. [Simple Gallery Pro](https://f-droid.org/zh_Hans/packages/com.simplemobiletools.gallery.pro)
27. [Simple SMS Messenger](https://f-droid.org/zh_Hans/packages/com.simplemobiletools.smsmessenger)
28. [Telegram](https://play.google.com/store/apps/details?id=org.telegram.messenger)
29. [Twilio Authy](https://play.google.com/store/apps/details?id=com.authy.authy)
30. [WPS Office Lite](https://play.google.com/store/apps/details?id=cn.wps.moffice_i18n)
31. [X浏览器](https://play.google.com/store/apps/details?id=com.xbrowser.play)
32. [阿里云盘](https://www.coolapk.com/apk/com.alicloud.databox)
33. [百度网盘](https://www.coolapk.com/apk/com.baidu.netdisk)
34. [白马地图](https://bmaps.cn), 从 [蓝奏云](https://bmap.lanzouk.com/b01bujs3i) 下载
35. [简单水印](https://www.coolapk.com/apk/me.rosuh.easywatermark)
36. [椒盐音乐](https://www.coolapk.com/apk/com.salt.music), [GitHub](https://github.com/Moriafly/SaltPlayerSource)
37. [京东](https://play.google.com/store/apps/details?id=com.jingdong.app.mall)
38. [酷安](https://www.coolapk.com/)
39. [蜜蜂便签](https://www.coolapk.com/apk/xyz.hanks.note)
40. [幕布](https://www.coolapk.com/apk/com.mubu.app)
41. [淘宝](https://play.google.com/store/apps/details?id=com.taobao.taobao)
42. [铁路12306](https://www.coolapk.com/apk/com.MobileTicket)
43. [微信](https://play.google.com/store/apps/details?id=com.tencent.mm)
44. [小米计算器](https://play.google.com/store/apps/details?id=com.miui.calculator)
45. [小米万能遥控](https://play.google.com/store/apps/details?id=com.duokan.phone.remotecontroller)
46. [一叶日历](https://www.coolapk.com/apk/me.mapleaf.calendar)
47. [支付宝](https://play.google.com/store/apps/details?id=com.eg.android.AlipayGphone)