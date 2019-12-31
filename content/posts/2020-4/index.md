---
title: "Debian自用设置"
date: 2020-06-12T20:00:00+08:00
draft: false
authors: ["senzyo"]
tags: [Bash,Custom,Debian,Hardware,Linux]
series: [Use Debian]
series_weight: 1
featuredImagePreview: "/2020-4/featuredImagePreview.svg"
summary: 备忘, 用于系统重装后进行自定义设置。
---

> 用于系统重装后的快速设置。

## 用户

参考[Users and groups](https://wiki.archlinux.org/title/Users_and_groups)。

### 新建用户

1. `-d`: 指定用户文件夹, `-m`: 如果文件夹不存在就新建, `-s`: 指定用户登录时使用指定`shell`。

   ```bash
   sudo useradd -d /home/<UserName> -m -s /bin/bash <UserName>
   ```

   {{< admonition type=tip title="如有必要" open=true >}}
   递归更改用户文件夹的权限: `sudo chown -R <UserName>:<UserGroup> /home/<UserName>`
   {{< /admonition >}}

2. 修改用户密码: 

   ```bash
   sudo passwd <UserName>
   ```

3. 检查所有用户`cat /etc/passwd`, 检查所有用户组`cat /etc/group`。

### 用户加入sudoers

1. 更改`sudoers`文件权限: 

   ```bash
   sudo chmod 644 /etc/sudoers
   ```

2. 编辑`/etc/sudoers`, 在`root ALL=(ALL:ALL) ALL`下添加一行`你的用户名 ALL=(ALL:ALL) ALL`, 比如: 

   ```
   root ALL=(ALL:ALL) ALL
   <UserName> ALL=(ALL:ALL) ALL
   ```

### 设置root用户的密码

```bash
sudo passwd root
```

## 自定义终端

### alias

在 [环境变量配置文件](../2021-6/) 中添加以下内容: 

```bash
alias ls='ls -F --color=auto'
alias la='ls -aF --color=auto'
alias ll='ls -alF --color=auto'
alias grep='grep --color=auto'
alias diff='diff --color=auto'
alias ch='> ~/.bash_history;history -c;clear'
```

### bash提示符颜色

参考[自定义bash提示符颜色](../2020-11/)。效果如下: 

普通用户:

<div style="background-color:black;font-weight:bold;">
    <span style="color:white;">20:34:50</span> <span style="color:red;">UserName</span><span style="color:white;">@</span><span style="color:red;">Debian</span> <span style="color:cyan;">/home/UserName</span><div></div><span style="color:red;">$</span>
</div>

root用户:

<div style="background-color:black;font-weight:bold;">
    <span style="color:white;">20:34:50</span> <span style="color:red;">root</span><span style="color:white;">@</span><span style="color:red;">Debian</span> <span style="color:cyan;">/root</span><div></div><span style="color:red;">#</span>
</div>

### ls -l时间格式

在 [环境变量配置文件](../2021-6/) 中添加: 

```bash
export TIME_STYLE='+%Y-%m-%d %H:%M:%S'
```

输出示例: 

```bash
-rw------- 1 root root 4096 2022-05-28 20:00:00 .bash_history
```

## 修改SSH设置

{{< admonition type=info title="注意" open=true >}}
如果不使用SSH连接到设备, 可以不设置, `sudo systemctl status ssh.service`确定设备的SSH服务没有运行即可。
{{< /admonition >}}

1. 参考[设置 SSH 通过密钥登录](https://www.runoob.com/w3cnote/set-ssh-login-key.html), 将客户机的公钥内容追加到服务器`~/.ssh/authorized_keys`文件中, 再更改权限: 

   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

2. 更改`/etc/ssh/sshd_config`文件内容: 

   ```
   Include /etc/ssh/sshd_config.d/*.conf
   Port 12345                                   # SSH端口号
   LoginGraceTime 3s                            # 若用户未成功登录, 服务器将在此时间后断开连接
   MaxAuthTries 4                               # 每个连接最大允许的认证次数
   MaxSessions 4                                # 最大允许保持多少个未认证的连接
   PermitRootLogin yes                          # 允许root用户登录
   AuthenticationMethods publickey              # 指定登录方式为使用公钥验证
   PubkeyAuthentication yes                     # 允许使用公钥验证登录
   PasswordAuthentication no                    # 禁用密码登录
   PermitEmptyPasswords no                      # 禁用空密码登录
   ChallengeResponseAuthentication no           # 双重认证需要
   UsePAM no                                    # 插入式验证模块, 比如双重认证
   PrintLastLog yes                             # 显示上次登录的信息
   TCPKeepAlive yes                             # 确保联机正常, server和client中的任何一端离线, 马上断开
   X11Forwarding no                             # 禁用远程执行GUI程序支持
   PrintMotd no                                 # 禁用登录后提示信息
   Banner none                                  # 禁用远程登录后提示信息
   Compression no                               # 网速快, 没必要压缩
   ClientAliveInterval 60                       # 服务器没有从客户端收到消息后的超时时间
   ClientAliveCountMax 3                        # 客户端超时次数
   UseDNS no                                    # 禁用反向DNS解析
   AcceptEnv LANG LC_*                          # 允许客户端传递区域设置环境变量
   Subsystem sftp /usr/lib/openssh/sftp-server  # sftp
   AllowUsers root <UserName>                     # 允许登录的用户名
   ```

   - `ClientAliveInterval`乘以`ClientAliveCountMax`, 即超时60s*3=180s后ssh断开连接。
   - 如果仍然有登录后提示, 可以清空相应文件`> /etc/motd`。

3. 重启SSH服务: 

   ```bash
   sudo systemctl restart ssh.service
   或者
   /etc/init.d/ssh restart
   ```

4. 在防火墙关闭22端口, 放行自定义的SSH端口号。

## 软件镜像源

在`/etc/apt/source.list`中编辑镜像源。先使用http源 (将源中的https替换为http) 安装以下软件: 

```bash
sudo apt install apt-transport-https ca-certificates
```

之后再使用https源。

### Debian(bullseye)

参考[Debian 全球镜像站](https://www.debian.org/mirror/list), 选择相应镜像源, 比如: 

[清华大学开源软件镜像站使用帮助-Debian](https://mirrors.tuna.tsinghua.edu.cn/help/debian/)

{{< admonition type=quote title="点此展开镜像源" open=false >}}
#默认注释了源码镜像以提高 apt update 速度, 如有需要可自行取消注释

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free

#deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free

#deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free

#deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free

deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free

#deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
{{< /admonition >}}

不太推荐使用[官方源](https://wiki.debian.org/SourcesList): 

{{< admonition type=quote title="点此展开官方源" open=false >}}
#默认注释了源码镜像以提高 apt update 速度, 如有需要可自行取消注释

deb https://deb.debian.org/debian bullseye main contrib non-free

#deb-src https://deb.debian.org/debian bullseye main contrib non-free

deb https://deb.debian.org/debian-security/ bullseye-security main contrib non-free

#deb-src https://deb.debian.org/debian-security/ bullseye-security main contrib non-free

deb https://deb.debian.org/debian bullseye-updates main contrib non-free

#deb-src https://deb.debian.org/debian bullseye-updates main contrib non-free

deb https://deb.debian.org/debian bullseye-backports main contrib non-free

#deb-src https://deb.debian.org/debian bullseye-backports main contrib non-free
{{< /admonition >}}

### Ubuntu(20.04 LTS)

参考[Official Archive Mirrors for Ubuntu](https://launchpad.net/ubuntu/+archivemirrors), 选择相应镜像源, 比如: 

[清华大学开源软件镜像站使用帮助-Ubuntu](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

{{< admonition type=quote title="点此展开镜像源" open=false >}}
#默认注释了源码镜像以提高 apt update 速度, 如有需要可自行取消注释

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse

#deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse

#deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse

#deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

#deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
{{< /admonition >}}

## 网卡驱动

### Intel AX210

检测已安装的驱动: `lspci -v`。高版本的Linux内核已经自带了`iwlwifi-ty-a0-gf-a0-59.ucode`。

下载[驱动](https://wireless.wiki.kernel.org/_media/en/users/drivers/iwlwifi-ty-59.601f3a66.0.tgz), 更多网卡驱动[列表](https://www.intel.com/content/www/us/en/support/articles/000005511/wireless.html)。

安装: 

```bash
tar -zxvf iwlwifi-ty-59.601f3a66.0.tgz
sudo cp iwlwifi-ty-59.601f3a66.0/iwlwifi-ty-a0-gf-a0-59.ucode /lib/firmware
sudo rm /lib/firmware/*.pnvm
sudo reboot
```

### RTL8821CE

检测已安装的驱动: `lspci -v`。

由于Linux内核 (kernel >= 5.9) 自带的**rtw88**对RTL8821CE兼容性极差, 驱动很快就会掉, 基本无法使用, 所以需要手动安装RTL8821CE无线网卡驱动。

1. 电脑插入网线, 或者手机USB网络共享。

2. 编辑`/etc/modprobe.d/blacklist.conf`, 添加: 

   ```
   blacklist rtw88_8821ce
   ```

   这样就禁用了内核自带的**rtw88**驱动。

3. 从tomaspinho的[GitHub仓库](https://github.com/tomaspinho/rtl8821ce)下载驱动。

4. 执行命令: 

   ```bash
   sudo apt install bc module-assistant build-essential dkms
   sudo m-a prepare
   ```

5. `cd`到下载的驱动的目录, 执行以下命令: 

   ```bash
   sudo ./dkms-install.sh
   ```

6. 默认情况下, Linux可能会启用PCIe活动状态电源管理。这可能与该驱动程序冲突。编辑`/etc/default/grub`, 在`GRUB_CMDLINE_LINUX_DEFAULT`后添加`pci=noaer`, 比如: 

   ```
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pci=noaer"
   ```

   然后更新grub: 

   ```bash
   sudo update-grub
   ```

   > 开机自检遇到某些报错时, 也可以在`GRUB_CMDLINE_LINUX_DEFAULT`后添加`pci=noaer`来解决, 这是一个在Stack Overflow常见的解决方案。

7. 重启设备。

## 应用

### 常用软件

1. [7-zip](https://www.7-zip.org/)
2. [Android Platform Tools](https://developer.android.com/studio/releases/platform-tools)
3. [Aria2](https://github.com/aria2/aria2/releases/latest)
4. [FFmpeg](https://www.ffmpeg.org/download.html)
5. [Google Chrome](https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb)
6. [Hugo](https://github.com/gohugoio/hugo/releases/latest), 下载 `hugo_extended`
7. [Joplin](https://joplinapp.org/help/#desktop-applications)
8. [LocalSend](https://localsend.org/#/download)
9. [Lunacy](https://icons8.com/lunacy)
10. [Microsoft Visual Studio Code](https://code.visualstudio.com/#alt-downloads)
11. [OBS Studio](https://obsproject.com/download)
12. [pot-app](https://github.com/pot-app/pot-desktop/releases/latest)
13. [Rime](../2023-2/)
14. [Telegram](https://desktop.telegram.org/)
15. [VLC](https://www.videolan.org/vlc/)
16. [VirtualBox](https://www.virtualbox.org/wiki/Linux_Downloads)
17. [百度网盘](https://pan.baidu.com/download)
18. [小白羊网盘](https://github.com/gaozhangmin/aliyunpan/releases/latest)

### 应用列表

1. 首先确定待执行文件是否有可执行权限。
2. 在任意位置右键新建一个`链接到应用程序`, 然后在**应用程序**标签下的**命令**里填入软件所在路径。
3. 有的软件因为权限问题打不开软件, 可在路径后加上`--no-sandbox`。
4. 之后将这个配置好的`.desktop`文件移动到`~/.local/share/applications/`即可。

### 应用图标

在`/usr/share/applications/`中, 编辑要更改的软件的`.desktop`文件, 在`Icon=`后填入自定义的软件图标路径即可。

{{< admonition type=tip title="技巧" open=true >}}
为防止以后误删该图标, 也可将图标放入`/usr/share/pixmaps/`文件夹下 (第三方图标都默认存放在这里), 然后在`Icon=`后接图片名称即可, 不用加文件后缀, 比如现在有文件`/usr/share/pixmaps/PicGo.png`, 在软件的`.desktop`文件写入`Icon=PicGo`即可。
{{< /admonition >}}

##  字体

1. 提前在Windows中将需要的字体从`C:/Windows/Fonts`中复制出来, 放到`C:/Users/<UserName>/Fonts`中。
2. 使用文件管理器(比如Dolphin)挂载Windows系统盘, 这时Windows系统盘根路径一般为`/run/media/<UserName>/Windows/`。
3. 在Linux中为Windows字体新建一个目录: 

   ```bash
   sudo mkdir /usr/share/fonts/Windows
   ```

4. 然后将Windows的字体文件复制过来: 

   ```bash
   sudo cp /run/media/<UserName>/Windows/Users/<UserName>/Fonts/* /usr/share/fonts/Windows
   sudo cp /run/media/<UserName>/Windows/Users/<UserName>/Fonts/* /usr/share/fonts/Windows
   ```

5. 更多字体设置参考 [字体](../2023-4/#字体)。
