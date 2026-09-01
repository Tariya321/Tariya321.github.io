---
title: "部署Perlite构建ob阅读web平台"
date: 2026-02-03
tags:
  - obsidian
  - docker
publish: yes
---
[secure-77/Perlite: A web-based markdown viewer optimized for Obsidian](https://github.com/secure-77/Perlite)

我提出的 issue：[can someone help me to figure out the no-content problem? · Issue #166 · secure-77/Perlite](https://github.com/secure-77/Perlite/issues/166)

原因：nas 文件和 docker 的权限问题
解决方案：根据 gpt 建议，添加了 rsync 机制，将 ob 原始目录单向同步到 docker 下面的目录
![PixPin_2026-02-03_16-25-26](/attachment/PixPin_2026-02-03_16-25-26.png)

配置 nginx 的访问权限
![PixPin_2026-02-03_16-44-11](/attachment/PixPin_2026-02-03_16-44-11.png)
只允许 tailscale ip 访问

```
sudo apt update
sudo apt --fix-broken install -y
sudo apt install apache2-utils -y
```
增加密码文件
```
sudo htpasswd -c /volume4/docker/perlite/web/config/.htpasswd yourusername
```
注意修改 docker compose，把密码文件映射一下
以及修改 nginx conf，使用密码认证
