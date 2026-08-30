---
title: "docker部署网页版qq"
date: 2026-08-16
tags:
  - docker
  - qq
publish: yes
---
需求：只有手机上有qq，但是不希望在其他pc上安装QQ
但是qq又有一些信息需要阅读，故需要一个支持一键部署且无法多次登录的环境

方案：采用[wechat-selkies](https://github.com/nickrunning/wechat-selkies?tab=readme-ov-file)

docker compose 文件
```yaml
services:
  wechat-selkies:
    image: nickrunning/wechat-selkies:latest    # or ghcr.io/nickrunning/wechat-selkies:latest
    container_name: wechat-selkies
    ports:
      - "${HTTP_PORT:-3000}:3000"
      - "${HTTPS_PORT:-3001}:3001"
    restart: unless-stopped
    volumes:
      - ./config:/config
    devices:
      - /dev/dri:/dev/dri
    environment:
      - PUID=${PUID:-1000}
      - PGID=${PGID:-100}
      - TZ=Asia/Shanghai
      - LC_ALL=zh_CN.UTF-8
      - AUTO_START_WECHAT=true
      - AUTO_START_QQ=false
      - CUSTOM_USER=${CUSTOM_USER:-}
      - PASSWORD=${PASSWORD:-}
    shm_size: "${SHM_SIZE:-1gb}"
```

为了可以用上dxp4800硬件配置中的N100自带的集显，可以在environment加入
```
- PIXELFLUX_WAYLAND=true
```

访问网页：
```
https://<服务器IP>:3001
```

success
{{< figure src="/attachment/PixPin_2026-08-16_15-39-10.png" alt="PixPin_2026-08-16_15-39-10" width="611" >}}



refs:
https://www.youtube.com/watch?v=mYTuL9TRx-c
https://github.com/linuxserver/docker-baseimage-selkies


