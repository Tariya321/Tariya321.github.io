---
date: 2025-07-05_12:14
tags:
  - macbook
  - ssh
  - x11
publish: yes
---
# mac使用ssh+x11转发打开gui

please refer: this [note](https://www.cnblogs.com/Undefined443/p/18057261) ,
and https://blog.iks-ran.com/2025/01/06/x11-forwarding-for-docker-on-macos/

## 1. setting x11 forward tool

install using homebrew
```shell
brew install --cask xquartz
```

使用 SSH 连接到服务器，并使用 `-Y` 选项启用 X11 转发
```shell
ssh -Y <server_name>
```

本地 pc 端先设置 display 变量
```
export DISPLAY=:0
```

请先打开 xquartz 工具，再使用！
{{< figure src="/attachment/mac%E4%BD%BF%E7%94%A8ssh%2Bx11%E8%BD%AC%E5%8F%91%E6%89%93%E5%BC%80gui.png" alt="mac使用ssh+x11转发打开gui" width="500" >}}

使用体验：
- gui 端能明显感到 1 秒左右的延时。不过不清楚是不是因为我通过 vpn 连接的原因
- 调起 gui 比较慢
- ssh 连接测试平均延迟为 50ms

问题：
- 点了上面的东西没反应

2025-07-07_21:33
回到局域网后没有这个问题

{{< figure src="/attachment/mac%E4%BD%BF%E7%94%A8ssh%2Bx11%E8%BD%AC%E5%8F%91%E6%89%93%E5%BC%80gui-1.png" alt="mac使用ssh+x11转发打开gui-1" width="400" >}}

然后需要在 XQuarze 的选项中打开"允许从网络客户端访问"
{{< figure src="/attachment/mac%E4%BD%BF%E7%94%A8ssh%2Bx11%E8%BD%AC%E5%8F%91%E6%89%93%E5%BC%80gui-2.png" alt="mac使用ssh+x11转发打开gui-2" width="450" >}}
## 2. 允许原生 terminal 使用转发服务

2025-10-01_00:20最近又在折腾了

为了在 mac 自带的 terminal 中也能用，在 xq 的命令行中输入
```
xhost + localhost
xhost + <remote_server_ip>
```
然后就可以愉快的访问了

## 3. 曲线救国-跳板机

方法
- mac 使用 window app 连接开启了 rdp 协议的服务器 A
- 服务器 A 上使用 `ssh -X` 连接服务器 B

> 注意 rdp 链接需要打开 32bit 显示，否则可能不正常



