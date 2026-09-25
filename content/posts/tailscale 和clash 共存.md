---
title: "tailscale 和clash 共存"
date: 2026-09-25
tags:
  - clash
  - tailscale
  - 运维
publish: true
---
背景
tailscale 用于内网穿透
clash 用于访问外网

遇到的问题：
clash 开着访问外网时，开启 ts 并使用 ts-ip ssh 访问机器
外网访问断连

解决办法
- ts 关闭 local dns
- clash 添加规则

![2026-09-25](/attachment/2026-09-25.png)

新增 clash 规则
```
"DOMAIN-SUFFIX,ts.net,DIRECT",
"DOMAIN-SUFFIX,tailscale.com,DIRECT",
"IP-CIDR,100.64.0.0/10,DIRECT,no-resolve"
```


refs：
https://jiz4oh.com/2024/09/tailscale-with-clash/
https://blog.ichr.me/post/tailscale-mihomo-quantumult-x/
https://linux.do/t/topic/2177060/13

