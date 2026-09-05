---
title: "Window Subsystem for Linux"
date: 2025-09-11
tags:
  - wsl
  - cmd
publish: yes
---
## 1. command
please refer:
[Basic commands for WSL | Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/basic-commands)

查看当前所有虚拟机器的状态
```shell
wsl -l -v
```

启动机器 (单一机器情况)
```shell
wsl
```

启动特定机器
```
wsl -d Ubuntu-22.04
```

关闭所有正在运行虚拟机器的指令
```
wsl --shutdown
```

关闭特定的虚拟机器
```shell
wsl --terminate <Distribution_Name>
```
关不掉？仔细看看是不是有什么进程没有关掉，例如在 wsl 中打开的 vscode

to remove your WSL virtual machine
```
wsl --unregister Ubuntu-22.04
```

### 1.1. 切换到普通用户登录

在 root 账户下修改 `/etc/wsl.conf` 文件以将登录账户切换为我的账户
```
[boot]
systemd=ture

[user]
default=hongzhilian
```
然后 terminate 这个系统，重启即可生效

## 2. usage

### 2.1. usb透传


window安装`usbipd`[透传工具](https://github.com/dorssel/usbipd-win)
```
winget install usbipd
```
check status
```
usbipd list
```
bind (share) USB device（永久，只要固定usb接口）
```
usbipd bind --busid=<BUSID>
```
attach USB device to WSL2 (非永久)
```
usbipd attach --wsl --busid=<BUSID>
```
attach后在WSL2端可以用`lsusb`检查附着情况

> [!NOTE] 永久和非永久
> 重启wsl系统、断开USB设备，均需要重新attach
> bind则是永久share了PC的某个USB口


