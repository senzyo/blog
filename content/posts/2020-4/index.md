---
title: "Debian自用设置"
date: 2020-06-12T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Bash,Custom,Debian,Hardware,Linux]
series: [Use Debian]
series_weight: 1
featuredImagePreview: "/2020-4/featuredImagePreview.svg"
summary: 备忘, 用于系统重装后进行自定义设置。
---

> 用于系统重装后的快速设置。

{{< admonition type=example title="官方文档" open=true >}}
https://www.debian.org/doc/manuals/debian-reference/ch01.zh-cn.html
{{< /admonition >}}

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

### 用户与用户组

查看用户详细信息:

```bash
cat /etc/passwd
```

只看“普通用户” (UID >= 1000 的用户) , 通常自己创建的用户 UID 是从 1000 开始的:

```bash
awk -F: '$3 >= 1000 && $3 != 65534 {print $1}' /etc/passwd
```

同理只看“系统用户” (UID < 1000 的用户) :

```bash
awk -F: '$3 < 1000 && $3 != 65534 {print $1}' /etc/passwd
```

查看用户组详细信息:

```bash
cat /etc/group
```

把用户添加到用户组:

```bash
gpasswd -a 用户名 组名
```

把用户从用户组中删除:

```bash
gpasswd -d 用户名 组名
```

查看某个用户所属的所有组:

```bash
groups xray
```

查看某个用户组包含的用户:

```bash
grep "^nginx" /etc/group
```

## 自定义终端

### alias

在 [环境变量配置文件](../2021-6/) 中添加以下内容:

```bash
alias ls="ls -F --color=auto"
alias la="ls -aF --color=auto"
alias ll="ls -alF --color=auto"
alias grep="grep --color=auto"
alias diff="diff --color=auto"
alias ch="> $HOME/.bash_history;history -c;clear"
```

### bash提示符颜色

参考[自定义bash提示符颜色](../2020-11/)。

### ls -l时间格式

在 [环境变量配置文件](../2021-6/) 中添加:

```bash
export TIME_STYLE="+%Y-%m-%d %H:%M:%S"
```

输出示例:

```bash
-rw------- 1 root root 4096 2022-05-28 20:00:00 .bash_history
```

## 修改SSH设置

{{< admonition type=tip title="提示" open=true >}}
- `/etc/ssh/ssh_config`是本机作为SSH客户端的配置, `/etc/ssh/sshd_config`是本机作为SSH服务器的配置。
- 更多信息参考 https://man.openbsd.org/sshd_config
{{< /admonition >}}

1. 在`~/.ssh/authorized_keys`文件中追加客户端的公钥内容, 然后更改权限:

   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

2. 更改`/etc/ssh/sshd_config`文件内容:

   ```
   # ==========================================
   # 1. 基础连接设置
   # ==========================================

   # 默认禁用子配置目录引入, 防止不可控的外部子配置文件覆盖本文件的安全设定
   # Include /etc/ssh/sshd_config.d/*.conf

   # 指定 SSH 服务监听的非默认高位端口
   Port 50022


   # ==========================================
   # 2. 身份验证与访问控制
   # ==========================================

   # 仅允许通过指定的普通用户名进行 SSH 登录
   AllowUsers admin

   # 禁用 root 用户直接登录, 强制通过普通用户登录后再进行提权
   PermitRootLogin no

   # 启用公钥身份验证机制
   PubkeyAuthentication yes

   # 强制限定用户必须且仅能通过公钥进行身份验证, 不接受其他方式
   AuthenticationMethods publickey

   # 禁用传统的密码身份验证
   PasswordAuthentication no

   # 禁用空密码账户登录, 规避因本地账户无密码配置带来的漏洞
   PermitEmptyPasswords no

   # 禁用键盘交互式身份验证
   KbdInteractiveAuthentication no

   # 启用插入式验证模块（PAM）, 常用于支持多因素认证（MFA）或环境初始化设置
   UsePAM yes

   # 每个连接允许的最大身份验证尝试次数。一旦失败次数达到此值的一半, 系统将记录额外的失败日志
   MaxAuthTries 4

   # 限制用户连接后完成身份验证的宽限时间, 超时将自动断开连接
   LoginGraceTime 10s

   # 显式关闭基于主机的验证
   HostbasedAuthentication no

   # 禁用 GSSAPI 认证, 减少暴露面
   GSSAPIAuthentication no

   # 禁用 Kerberos 认证
   KerberosAuthentication no

   # ==========================================
   # 3. 密码学与主机密钥加固
   # ==========================================

   # 限制服务器主机发送指纹时所用密钥类型
   HostKey /etc/ssh/ssh_host_ed25519_key

   # 限制客户端公钥类型为 ed25519 密钥
   PubkeyAcceptedAlgorithms ssh-ed25519,ssh-ed25519-cert-v01@openssh.com,sk-ssh-ed25519@openssh.com,sk-ssh-ed25519-cert-v01@openssh.com


   # ==========================================
   # 4. 会话控制、超时与防 DoS
   # ==========================================

   # 指定每个网络连接允许的最大并发活动会话数
   MaxSessions 10

   # 限制最大并发未授权连接数（格式为“起始数:丢弃概率:最大值”）, 用于防范连接洪泛攻击
   MaxStartups 5:20:10

   # 服务器向客户端发送心跳活动消息的间隔时间（单位：秒）, 用于检测客户端是否在线
   ClientAliveInterval 60

   # 允许客户端无响应的最大重试次数。达到此上限后, 服务器将主动关闭连接
   ClientAliveCountMax 3

   # 启用系统的 TCP 保持连接机制, 避免因网络波动导致会话无限期挂起
   TCPKeepAlive yes

   # 原生防爆破惩罚机制
   PerSourcePenalties yes

   # ==========================================
   # 5. 安全转发限制
   # ==========================================

   # 禁用 X11 转发, 防止图形窗口通道被恶意利用
   X11Forwarding no

   # 禁用 SSH 代理转发, 防止跳板机上的代理连接被其他特权用户滥用
   AllowAgentForwarding no

   # 禁用 TCP 端口转发, 防止用户利用该连接进行内网穿透或规避防火墙限制
   AllowTcpForwarding no


   # ==========================================
   # 6. 系统环境与子系统设置
   # ==========================================

   # 禁用登录时展示的 MOTD 提示信息
   PrintMotd no

   # 登录时显示上次登录成功的信息
   PrintLastLog yes

   # 禁用传输过程中的数据压缩, 防止由于压缩算法引起的侧信道信息泄露
   Compression no

   # 配置并指定 SFTP 文件传输子系统的可执行程序路径
   Subsystem sftp /usr/lib/openssh/sftp-server
   ```

   - 如果仍然有登录后提示, 可以清空相应文件`> /etc/motd`。

3. 重启SSH服务:

   ```bash
   sudo systemctl restart ssh.service
   ```

4. 在防火墙关闭22端口, 放行自定义的SSH端口号。

## 软件镜像源

在`/etc/apt/source.list`中编辑镜像源。先使用http源 (将源中的https替换为http) 安装以下软件:

```bash
sudo apt install apt-transport-https ca-certificates
```

之后再使用https源。

### Debian

参考 [Debian 全球镜像站](https://www.debian.org/mirror/list) 和 [清华大学开源软件镜像站使用帮助-Debian](https://mirrors.tuna.tsinghua.edu.cn/help/debian/)

### Ubuntu

参考 [Official Archive Mirrors for Ubuntu](https://launchpad.net/ubuntu/+archivemirrors) 和 [清华大学开源软件镜像站使用帮助-Ubuntu](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

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

## 字体

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

## TCP调优

### BBR

查看系统现在的拥塞控制算法:

```bash
sysctl net.ipv4.tcp_congestion_control
```

查看系统可用的拥塞控制算法:

```bash
sysctl net.ipv4.tcp_available_congestion_control
```

如果没有 `bbr`, 检查 BBR 的内核模块是否已加载:

```bash
lsmod | grep bbr
```

手动加载 BBR 的内核模块:

```bash
sudo modprobe tcp_bbr
```

编辑 `/etc/sysctl.conf` 或者 `/etc/sysctl.d/99-sysctl.conf`, 添加:

```
net.ipv4.tcp_congestion_control = bbr
net.core.default_qdisc = fq
```

运行 `sysctl --system` 应用更改。

检查系统现在的拥塞控制算法和队列规则算法:

```bash
sysctl net.ipv4.tcp_congestion_control
sysctl net.core.default_qdisc
```

### TCP缓冲区

参考文章: [『败类教程』美西CN2跨网也能单线程500M且0重传！手把手教你TCP调优](https://www.nodeseek.com/post-197087-1)

但受运营商限制, 未能发挥理论作用, 效果不大。

查看当前 TCP 缓冲区大小:

```bash
sysctl net.ipv4.tcp_wmem
sysctl net.ipv4.tcp_rmem
```

根据 [TCP缓冲区计算器](https://tcp-cal.mereith.com) 临时设置 TCP 缓冲区大小:

```bash
sysctl -w net.ipv4.tcp_wmem="4096 16384 4194304"
sysctl -w net.ipv4.tcp_rmem="4096 131072 6291456"
```

或将参数写入 `/etc/sysctl.conf` 以永久保存, 这是 Debian 的默认参数:

```
net.ipv4.tcp_wmem = 4096 16384 4194304
net.ipv4.tcp_rmem = 4096 131072 6291456
```