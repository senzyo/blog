---
title: "Debian安装NVIDIA驱动"
date: 2020-07-21T20:00:00+08:00
draft: false
authors: ["senzyo"]
tags: [Debian,Hardware,NVIDIA]
featuredImagePreview: "/2020-13/featuredImagePreview.svg"
summary: 在 Debian 系 Linux 中安装或卸载 NVIDIA 官方驱动。
---

{{< admonition type=warning title="" open=true >}}
本文可能并不准确, 仅供参考。
{{< /admonition >}}

系统一般自带开源驱动`nouveau`, 其性能不如NVIDIA官方驱动。然而安装NVIDIA官方驱动的过程是十分坎坷的。

## 识别显卡型号

```bash
lshw -numeric -C display
```

## 预安装操作

1. 安装当前运行内核的内核头文件和开发包: 

    ```bash
    sudo apt-get install linux-headers-$(uname -r)
    ```

2. 编译和安装NVIDIA驱动程序需要以下先决条件: 

    ```bash
    sudo apt install build-essential libglvnd-dev pkg-config
    ```

3. 如果想使用32位兼容性库应用, 比如Steam, 则需要: 

    ```bash
    sudo dpkg --add-architecture i386
    sudo apt update
    sudo apt install libc6:i386
    ```

## 禁用Nouveau

1. 创建文件`/etc/modprobe.d/blacklist-nouveau.conf`, 内容如下: 

    ```
    blacklist nouveau
    options nouveau modeset=0
    ```

    写在`/etc/modprobe.d/blacklist.conf`里也可以生效, 不过视系统而定。

2. 重新生成内核initramfs:

    ```bash
    sudo update-initramfs -u
    ```

## 检查NVIDIA是否卸载完全

1. 可以通过这些命令查看, 应该都提示找不到或者没有nvidia就算卸载好了: 

    ```bash
    cat /proc/driver/nvidia/version
    nvidia-smi
    dkms status
    lsmod | grep nvidia
    ```

2. 如果之前是编译run包安装的, 可以使用

    ```bash
    sudo ./NVIDIA_....run --uninstall
    ```
 
    卸载, 或者

    ```bash
    sudo /usr/bin/nvidia-uninstall
    ```

3. 如果`dkms status`有驱动, 则使用

    ```bash
    dkms remove nvidia/418.102.04 --all
    ```

    卸载, 注意这里版本号要与`dkms staus`输出的信息一致。

4. 不放心的话, 可以再执行`apt purge *nvidia*`。如果卸载了所有驱动, `proc/driver`目录下还是有nvidia的东西, 重启即可 (实在不行手动删除)。

## TTY下安装NVIDIA驱动

如果设备只有一个NVIDIA显卡, 禁用nouveau后, 再重启可能就没有图形界面了;如果还有一个Intel集成显卡, 则开机还有图形界面。

有几种方法进入tty: 

- 开机时按`Ctrl+Alt+F2`进入
- Ubuntu需要在grub进入recover模式, 选择root用户操作。
- 终端输入`sudo init 3`或`sudo telinit 3`进入。 (输入`sudo init 5`或`sudo telinit 5`恢复平常的图形界面) 

进入tty后, 使用以下命令排除影响: 

```bash
sudo systemctl isolate multi-user.target
sudo lsof /dev/nvidia* | grep -v PID | grep -v lsof | awk '{print $2}' | xargs sudo kill -9
modprobe -r nouveau
```

### 从源安装驱动

1. 首先安装NVIDIA官方驱动, 参考[Debian WiKi](https://wiki.debian.org/NvidiaGraphicsDrivers)

2. 对于Debian: 

    ```bash
    apt install nvidia-driver firmware-misc-nonfree
    ```
3. 对于Ubuntu, 使用`ubuntu-drivers devices`查看系统推荐的版本, 然后安装指定版本, 比如: 

    ```bash
    sudo apt install nvidia-driver-470
    ```

    如果源中没有, 需要手动添加PPA源: 
    
    ```bash
    sudo add-apt-repository ppa:graphics-drivers/ppa
    ```

### 编译安装NVIDIA官网run包

{{< admonition type=warning title="极不推荐" open=true >}}
成功概率不高, 按上面的步骤走下来, 虽然可以避免大部分问题, 但仍有可能卡在最后一步, 并且视设备与系统的不同, 最后一步的报错可能也不同, 甚至每次都不一样。🙄
{{< /admonition >}}

除了从源直接安装驱动, 也可以到NVIDIA[官网](https://www.nvidia.cn/Download/index.aspx?lang=cn)下载`run`包, 手动编译安装, [NVIDIA run包编译安装指南](https://us.download.nvidia.cn/XFree86/Linux-x86_64/450.57/README/installdriver.html)

切换到run包所在目录, 赋予run包可执行权限, 然后`sh NVIDIA_....run`。

## 可能遇到的问题

可能遇到的问题相当之多, 参考[neucrack](https://neucrack.com/p/252)

## 使用Prime切换显卡

```bash
sudo nvidia-prime
```

切换Intel显卡: 

```bash
sudo prime-select intel
```

切换NVIDIA显卡: 

```bash
sudo prime-select nvidia
```

查看正在运行的显卡: 

```bash
prime-select query
```

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
1. [Linux Nvidia 驱动安装(GPU for tensorflow) 及常见问题](https://neucrack.com/p/252)
2. [Linux安装NVIDIA显卡驱动的正确姿势](https://blog.csdn.net/wf19930209/article/details/81877822)
3. [NvidiaGraphicsDrivers](https://wiki.debian.org/NvidiaGraphicsDrivers)
4. [Installing the NVIDIA Driver](https://us.download.nvidia.cn/XFree86/Linux-x86_64/450.57/README/installdriver.html)
5. [Installation Guide Linux :: CUDA Toolkit Documentation](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)
{{< /admonition >}}