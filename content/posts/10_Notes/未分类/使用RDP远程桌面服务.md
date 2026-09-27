---
title: "使用 RDP 远程桌面服务"
date: 2025-04-14_16:10
tags:
  - linux
  - server
  - macbook
  - gui
publish: true
---
mac以gui方式连接远程服务器

mac
- windows app


## 1. linux setting

 Centos 7

```shell
sudo yum install epel-release -y
# install xrdp service
sudo yum install xrdp -y
sudo systemctl enable xrdp --now
sudo systemctl status xrdp
# open firewall permission (optional)
sudo firewall-cmd --permanent --add-port=3389/tcp
sudo firewall-cmd --reload
# check port is listening
sudo netstat -tlnp | grep 3389
```

## 2. windows setting
开启远程桌面
```
Set-ItemProperty `
  -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' `
  -Name fDenyTSConnections `
  -Value 0
```

```
Set-Service TermService -StartupType Automatic
Start-Service TermService
```
check status
```
Get-Service TermService
```
check port listening
```
netstat -ano | findstr :3389
```

![启动远程桌面连接服务](/attachment/%E5%90%AF%E5%8A%A8%E8%BF%9C%E7%A8%8B%E6%A1%8C%E9%9D%A2%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1.png)

在本机上查看是否能否访问该端口
```
nc -vz <ts_ip> 3389
```

![启动远程桌面连接服务-1](/attachment/%E5%90%AF%E5%8A%A8%E8%BF%9C%E7%A8%8B%E6%A1%8C%E9%9D%A2%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1-1.png)


