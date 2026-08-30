---
title: "window11上WSL2+Ubuntu+vitis的安装"
date: 2025-09-08
tags:
  - wsl
  - linux
  - xilinx
publish: yes
---
关键词：window11+WSL2+ubuntu+vitis

## 1. install WSL

勾选打开“**启用或关闭Windows功能**”下功能：virtual machine platform、window 虚拟机监控程序平台、适用于linux的window子系统；window11还可以打开hyper-V功能

**重启电脑**

命令设置默认为WSL-2版本
`wsl --set-default-version 2`

报信息：“适用于 Linux 的 Windows 子系统必须更新到最新版本才能继续。可通过运行 “wsl.exe --update” 进行更新。”

使用“wsl.exe --update“后报信息：已禁止(403)。
网页提示：https://www.bytezonex.com/archives/51juM4wc.html

到**github网站下载最新的wsl.msi文件**，双击安装后再次设置
```
wsl --set-default-version 2
```


For more info about WSL command, please read WSL.

## 2. install Ubuntu
自定义安装位置的ubuntu安装方式：https://loopguy.com/post/how-to-install-ubuntu-wsl-in-a-custom-location-windows-subsystem-for-linux

ubuntu for wls位置：https://cloud-images.ubuntu.com/wsl/jammy/current/

安装命令
```
wsl --import Ubuntu-22.04 E:\WSL\Ubuntu-22.04-LTS E:\ubuntu-jammy-wsl-amd64-ubuntu22.04lts.rootfs.tar.gz  --version 2
```

查看当前所有虚拟机器的状态
```shell
wsl -l -v
```

启动机器
```shell
wsl
```

创建普通用户
```shell
adduser hongzhilian
```

加入sudo权限组
```
usermod -aG sudo <user_name>
```
> group `sudo` for debian based OS, group `wheel` for rhel based OS


see example config file at [Advanced settings configuration in WSL | Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/wsl-config#wslconf)

更换源，参考 [ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)


更新包
```
sudo apt update
sudo apt upgrade
```

安装桌面环境
```
sudo apt-get install ubuntu-gnome-desktop
```

很多报错：“Failed to reload daemon: Transport endpoint is not connected”
解决办法
```
apt purge -y acpid acpi-support modemmanager
apt-mark hold acpid acpi-support modemmanager
```
> 网页解答：https://blog.csdn.net/qq_30448087/article/details/134897586

## 3. mobaXterm+WSL

moba 可自动读取到启用中的 wsl 虚拟机，点开即可启用
并且带有 x server，于是所有 gui 界面都可以使用了

WSL 需要安装 xrdp
```
sudo apt install x11-apps
sudo apt install xrdp -y && sudo systemctl enable xrdp
```
其实这一套装好也可以正常打开 gui 了

## 4. install Vitis

下面这一套**务必先配置好！**
```
sudo apt install libncurses5
sudo apt-get install libtinfo5
sudo apt install build-essential libc6-dev
sudo apt-get install gcc-multilib g++-multilib
```
环境设置，在 bashrc 里面加入
```
export LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LIBRARY_PATH
```


否则在安装过程中或者安装后使用时，你将遇到以下问题（amd：问就是在改进了）
```
/tools/Xilinx/Vitis_HLS/2022.2/tps/lnx64/gcc-8.3.0/include/c++/8.3.0/x86_64-pc-linux-gnu/bits/os_defines.h:39:10: fatal error: features.h: No such file or directory

/usr/include/features-time64.h:20:10: fatal error: bits/wordsize.h: No such file or directory #include <bits/wordsize.h>

/tools/Xilinx/Vivado/2022.2/tps/lnx64/binutils-2.37/bin/ld: cannot find crt1.o: No such file or directory
```


AMD 官网下载 vitis linux 的安装器 （installer），提权
```shell
sudo chmod a+x XXXX.bin
```

无 sudo 不可安装，有 sudo 必须加-E 否则无法打开 gui
```
sudo -E ./FPGAs_AdaptiveSoCs_Unified_SDI_2025.1_0530_0145_Lin64.bin
```

选择 vitis 版本即可，基本上后面可以继续选择要装的东西

选择好选项后，开始安装
{{< figure src="/attachment/window11%E4%B8%8AWSL2%2BUbuntu%2Bvitis%E7%9A%84%E5%AE%89%E8%A3%85.png" alt="window11上WSL2+Ubuntu+vitis的安装" width="500" >}}

结束处报错出现警告信息
```
######## Execution of Pre/Post Installation Tasks Failed ########
Warning: AMD software was installed successfully, but an unexpected status was returned from the following post installation task(s) /tools/Xilinx/2025.1/Vivado/bin/rdiArgs.sh: line 37: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8): No such file or directory /bin/bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8) terminate called after throwing an instance of 'std::runtime_error' what(): locale::facet::_S_create_c_locale name not valid /tools/Xilinx/2025.1/Vivado/bin/rdiArgs.sh: line 454: 200895 Aborted (core dumped) "$RDI_PROG" "$@" /tools/Xilinx/2025.1/Vivado/bin/rdiArgs.sh: line 37: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8): No such file or directory /bin/bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8) terminate called after throwing an instance of 'std::runtime_error' what(): locale::facet::_S_create_c_locale name not valid /tools/Xilinx/2025.1/Vivado/bin/rdiArgs.sh: line 454: 201350 Aborted (core dumped) "$RDI_PROG" "$@"
```

install 2022.2 ver
{{< figure src="/attachment/window11%E4%B8%8AWSL2%2BUbuntu%2Bvitis%E7%9A%84%E5%AE%89%E8%A3%85-1.png" alt="window11上WSL2+Ubuntu+vitis的安装-1" width="500" >}}

更新相关命令脚本
```
source /tools/Xilinx/Vitis_HLS/2022.2/settings64.sh
```

可以 invoke 了，大功告成
{{< figure src="/attachment/window11%E4%B8%8AWSL2%2BUbuntu%2Bvitis%E7%9A%84%E5%AE%89%E8%A3%85-2.png" alt="window11上WSL2+Ubuntu+vitis的安装-2" width="500" >}}

## 5. WSL with VScode

- 本地（window 端）的 vscode **安装插件：`Remote Development`**
> 2025-11-23_14:51，此插件不再支持 1.97 版本的 vscode
但是高版本的 vscode 无法连接组里的服务器（glic 版本较老）


好了，接下来咱们可以快乐的使用工作流啦。WSL+Code 就是坠叼的

在 wsl 中直接输入 `code .` 命令即可打开 vscode 进行编辑



problem
```
Unable to initialize Git; AggregateError(2)
	Error: Unable to find git
	Error: Unable to find git
```
原因是"需要在每个要与之一起使用的文件系统上安装 Git"，请见[在 WSL 上使用 Git 入门 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-git)


方案: install vscode in WSL
```shell
wget https://go.microsoft.com/fwlink/?LinkID=760868
sudo apt install ./code_1.103.2-1755709794_amd64.deb
```

不过安装完会报信息
```
To use Visual Studio Code with the Windows Subsystem for Linux, please install Visual Studio Code in Windows and uninstall the Linux version in WSL. You can then use the `code` command in a WSL terminal just as you would in a normal command prompt
```

卸载
```shell
sudo apt remove code
sudo apt autoremove
```

write git conifg file at `~/.gitconfig` for WSL
```
[user]
    name = <your_simple_name>
    email = xxx@example.com

[core]
    editor = nvim
    autocrlf = input
    safecrlf = true
    excludesfile = ~/.gitignore_global

[color]
    ui = auto

[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    lg = log --oneline --graph --decorate --all

[pull]
    rebase = false

[push]
    default = simple
```

install git for window, [Git - Downloading Package](https://git-scm.com/downloads/win)

## 6. Reference


**网友帖子**
https://blog.csdn.net/Natsuago/article/details/145594631
安装wsl2+linux平台

https://www.bilibili.com/opus/690388376404099104
安装vitis

[手把手教你在Windows下用WSL运行Vitis/Vivado/Petalinux](https://mp.weixin.qq.com/s/aBQcrIpEFl2jCXdzk3ruzA)
wsl+ubuntu+vitis

**官方网页指示**
https://learn.microsoft.com/zh-cn/windows/wsl/install-manual#step-2---check-requirements-for-running-wsl-2
安装wsl2

https://documentation.ubuntu.com/wsl/latest/howto/install-ubuntu-wsl2/
基于wsl安装ubuntu

https://leezp.top/?p=111
WSL+MobaXterm启用图形化界面

[Vivado HLS C simulation and C/RTL cosimulation running Debian/testing](https://adaptivesupport.amd.com/s/question/0D52E00006hpJpSSAU/vivado-hls-c-simulation-and-crtl-cosimulation-running-debiantesting?language=en_US)
为什么你的 hls 跑不起来，请看官方解答


[Visual Studio Code on Linux](https://code.visualstudio.com/docs/setup/linux)
为 wsl 安装 vscode

