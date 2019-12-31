---
title: "Windows自用设置"
date: 2022-01-03T20:00:00+08:00
draft: false
authors: ["senzyo"]
tags: [Custom,Windows]
featuredImagePreview: ""
summary: 备忘, 用于系统重装后进行自定义设置。
---

> 用于系统重装后的快速设置。

## WhyNotWin11

检测是否支持升级 Windows11: [https://github.com/rcmaehl/WhyNotWin11/releases/latest](https://github.com/rcmaehl/WhyNotWin11/releases/latest)

## 下载ISO

### 从MSDN下载

更推荐从 MSDN 下载 ISO。因为 MSDN 的 ISO 是集成了最近的重要更新的, 所以版本比较新, 相比之下, 微软官网似乎只发布大版本的 ISO。

MSDN 官网: https://next.itellyou.cn/

### 从微软官网下载

最好直接下载 ISO, 而不是通过 `创建工具` 下载。

Windows11: [https://www.microsoft.com/zh-cn/software-download/windows11](https://www.microsoft.com/zh-cn/software-download/windows11)

Windows10: [https://www.microsoft.com/zh-cn/software-download/windows10](https://www.microsoft.com/zh-cn/software-download/windows10)

对于 Windows10 的下载页面, 如果浏览器 UA 是 PC, 则只有下载 `创建工具` 一种方式, 切换为手机 UA, 页面就会像 Windows11 的下载页面一样, 有三种下载方式了。

## 激活系统

{{< admonition type=warning title="数字权利证书激活工具已失效" open=true >}}
据 [Reddit](https://www.reddit.com/r/Piracy/comments/16to3w3/it_has_finally_happened_hwid_activation_for/?rdt=44392) 消息, 自世界标准时间 2023-09-26 16:22 以后, 微软阻止新的 HWID 激活请求, 现有的 HWID 许可证不受影响。
{{< /admonition >}}

{{< admonition type=success title="好消息" open=true >}}
Microsoft Activation Scripts (MAS) 采用新方法重新支持 HWID 激活, 更多信息参见 [HWID Activation](https://massgrave.dev/hwid.html)。
{{< /admonition >}}

[Microsoft Activation Scripts](https://github.com/massgravel/Microsoft-Activation-Scripts/releases/latest) 适用于 Windows8.1 至 Windows11 以及 Office。

## 禁止系统更新自动更新驱动

有时候自己安装的更新比 Windows 更新的时候安装的驱动更好、更新, 为了防止 Windows 更新将驱动换掉, 需要禁用自动驱动更新: 

1. 按下 Win + S/Q, 搜索“查看高级系统设置”。
2. 在“高级系统设置”的“硬件”选项卡下, 点击“设备安装设置”。
3. 选择“否, 每次只为我安装最兼容的驱动程序”并保存更改。

如果使用 Windows 10 专业版, 可以通过本地组策略编辑器修改 Windows Update 的行为: 

1. 按下 Win + S/Q, 搜索“编辑组策略”。
2. 转到“计算机配置”→“管理模板”→“Windows 组件”→“Windows 更新”。
3. 找到“Windows 更新不包括驱动程序”, 启用它。

## 下载软件

- [7-zip](https://www.7-zip.org/)
- [AIDA64](https://www.aida64.com/downloads)
- [AM-DeadLink](https://www.aignes.com/deadlink.htm)
- [Android Platform Tools](https://developer.android.com/studio/releases/platform-tools)
- [AppContainer Loopback Exemption Utility](https://www.telerik.com/fiddler/add-ons)
- [Aria2](https://github.com/aria2/aria2/releases/latest)
- [Autoruns](https://learn.microsoft.com/en-us/sysinternals/downloads/autoruns)
- [Bulk Crap Uninstaller](https://github.com/Klocman/Bulk-Crap-Uninstaller/releases/latest)
- [Context Menu Manager](https://gitee.com/BluePointLilac/ContextMenuManager)
- [CrystalDiskInfo & CrystalDiskMark](https://crystalmark.info/en/download/)
- [dupeGuru](https://github.com/arsenetar/dupeguru/releases/latest)
- [EasyUEFI](https://www.easyuefi.com/index-us.html)
- [Everything](https://www.voidtools.com/zh-cn/)
- [FFmpeg](https://www.ffmpeg.org/download.html)
- [FileZilla](https://filezilla-project.org/)
- [Geek Uninstaller](https://geekuninstaller.com/)
- [Git](https://git-scm.com/downloads)
- [Google Chrome](https://www.google.cn/intl/zh-CN/chrome/?standalone=1) 离线安装版
- [HEVC扩展​](https://store.rg-adguard.net/), 来自设备制造商的 HEVC 扩展, ​ProductId: `9n4wgh0z6vhq`
- [HiBit Uninstaller](https://www.hibitsoft.ir/)
- [Hugo](https://github.com/gohugoio/hugo/releases/latest), 下载 `hugo_extended`
- [InControl](https://www.grc.com/incontrol.htm), 用于控制 Windows 更新, 可以只更新补丁, 不更新版本
- [Joplin](https://joplinapp.org/help/#desktop-applications)
- [Kaspersky](https://my.kaspersky.com/)
- [Kodi](https://kodi.tv/)
- [LocalSend](https://localsend.org/#/download)
- [LockHunter](https://lockhunter.com/)
- [Lunacy](https://icons8.com/lunacy)
- [Microsoft 365 E5 Renew Plus](https://e5renew.com/)
- [Microsoft .NET SDK](https://dotnet.microsoft.com/zh-cn/download/dotnet)
- [Microsoft Visual C++ Redistributable](https://learn.microsoft.com/zh-CN/cpp/windows/latest-supported-vc-redist)
- [Microsoft Visual Studio Code](https://code.visualstudio.com/#alt-downloads)
- [MusicPlayer2](https://github.com/zhongyang219/MusicPlayer2/releases/latest)
- [Node.js](https://nodejs.org/en/download/)
- [OBS Studio](https://obsproject.com/download)
- [Office Tool Plus](https://otp.landian.vip/zh-cn/)+[GVLKs](https://learn.microsoft.com/en-us/deployoffice/vlactivation/gvlks)+[KMS](https://www.coolhub.top/tech-articles/kms_list.html)
- [OpenJDK Archived](https://jdk.java.net/archive/)
- [Oracle JDK Archive](https://www.oracle.com/java/technologies/downloads/archive/)
- [Pandoc](https://github.com/jgm/pandoc/releases/latest)
- [PixPin](https://pixpinapp.com/), 相比侧重于贴图的 Snipaste 有更多功能
- [pot-app](https://github.com/pot-app/pot-desktop/releases/latest)
- [Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer)
- [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon)
- [QtScrcpy](https://github.com/barry-ran/QtScrcpy/releases/latest)
- [QQ](https://im.qq.com/pcqq)
- [Resource Hacker](http://angusj.com/resourcehacker/)
- [Scrcpy](https://github.com/Genymobile/scrcpy/releases/latest)
- [Snipaste](https://zh.snipaste.com/)
- [Telegram](https://telegram.org/)
- [TopMost](https://www.sordum.org/9182/window-topmost-control-v1-2/)
- [TrafficMonitor](https://gitee.com/zhongyang219/TrafficMonitor)
- [ValiDrive](https://www.grc.com/validrive.htm), 检测优盘、存储卡、移动硬盘的真实容量, 避免虚标
- [Ventoy](https://github.com/ventoy/Ventoy/releases/latest)
- [Victoria HDD](https://hdd.by/victoria/)
- [Vim](https://www.vim.org/download.php)
- [VLC](https://www.videolan.org/vlc/)
- [Windows 11 Classic Context Menu](https://www.sordum.org/14479/windows-11-classic-context-menu-v1-1/)
- [Windows Defender Control](https://www.sordum.org/9480/defender-control-v2-1/)
- [Windows Update Blocker](https://www.sordum.org/9470/windows-update-blocker-v1-8/), 启用和禁用 Windows 更新
- [WizTree](https://diskanalyzer.com/download), 磁盘文件大小分析
- [阿里云盘](https://www.aliyundrive.com/)
- [百度网盘](https://pan.baidu.com/download)
- [哔哩下载姬](https://github.com/leiurayer/downkyi/releases/latest)
- [微信](https://weixin.qq.com/)

## 托盘时间显示秒数

{{< admonition type=success title="好消息" open=true >}}
较新版本的 Windows11 在设置中直接添加了这个功能的开关, 用户不必手动编辑注册表了:

设置→个性化→任务栏→任务栏行为→在系统栏托盘时钟中显示秒数 (耗电更多)
{{< /admonition >}}

定位到注册表 `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced`, 新建 `DWORD (32位) 值`, 命名为 `ShowSecondsInSystemClock`, 修改数值数据为 **1** (十六进制), 最后重启 **explorer.exe**。

此项功能 Windows10 和 Windows11 可用。值得注意的是, Windows11 一开始删除了这项功能, 后来又把它加回来了。如果你设置后没有生效, 请将 Windows11 更新至最新。

## 右键菜单检测文件哈希值

为什么要分成 Windows10 和 Windows11 两个部分呢, 因为我没能成功地自定义 Windows Terminal 打开后的窗口大小 (虽然翻了官方文档, 也看了 GitHub Issues)。

### Windows10

将以下内容写入一个 `txt` 中, 修改文件后缀名保存为 `.reg` 文件, 双击导入到注册表: 

```
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash]
"MUIVerb"="Get-FileHash"
"SubCommands"=""
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

; MD5
[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\MD5]
"MUIVerb"="MD5"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\MD5\command]
@="PowerShell -NoExit -Command \"mode con cols=90 lines=15\";\"Get-FileHash '%1' -Algorithm MD5 | Format-List\""

;SHA1
[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA1]
"MUIVerb"="SHA1"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA1\command]
@="PowerShell -NoExit -Command \"mode con cols=90 lines=15\";\"Get-FileHash '%1' -Algorithm SHA1 | Format-List\""

;SHA256
[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA256]
"MUIVerb"="SHA256"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA256\command]
@="PowerShell -NoExit -Command \"mode con cols=90 lines=15\";\"Get-FileHash '%1' -Algorithm SHA256 | Format-List\""
```

### Windows 11

将以下内容写入一个 `txt` 中, 修改文件后缀名保存为 `.reg` 文件, 双击导入到注册表: 

```
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash]
"MUIVerb"="Get-FileHash"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"
"SubCommands"=""

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell]

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\MD5]
"MUIVerb"="MD5"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\MD5\command]
@="PowerShell -NoExit Get-FileHash '%1' -Algorithm MD5 | Format-List"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA1]
"MUIVerb"="SHA1"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA1\command]
@="PowerShell -NoExit Get-FileHash '%1' -Algorithm SHA1 | Format-List"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA256]
"MUIVerb"="SHA256"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA256\command]
@="PowerShell -NoExit Get-FileHash '%1' -Algorithm SHA256 | Format-List"
```

### 移除

将以下内容写入一个 `txt` 中, 修改文件后缀名保存为 `.reg` 文件, 双击导入到注册表: 

```
Windows Registry Editor Version 5.00

[-HKEY_CLASSES_ROOT\*\shell\Get-FileHash]
```
