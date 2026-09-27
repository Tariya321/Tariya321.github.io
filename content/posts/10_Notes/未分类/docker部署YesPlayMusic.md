---
title: "docker部署YesPlayMusic"
date: 2025-04-24_13:49
tags:
  - docker
  - music
publish: yes
---

**Reference**
1. shell build
https://tyooe.com/docker-deploy-yesplaymusic-and-r3playx/
https://www.huwangyun.cn/blog/music


2. docker compose build
https://www.cnblogs.com/houhuilinblogs/p/18076854
[YesPlayMusic/docker-compose.yml at master · qier222/YesPlayMusic](https://github.com/qier222/YesPlayMusic/blob/master/docker-compose.yml)

```
services: 
  yesplaymusic: 
    container_name: yesplaymusic 
    image: fogforest/yesplaymusic:0.4.9
    ports: 
      - 7950:80 
    restart: always
```

界面有点帅气呢
![docker部署YesPlayMusic-2](/attachment/docker%E9%83%A8%E7%BD%B2YesPlayMusic-2.png)

支持登陆网易云，还不错
{{< figure src="/attachment/docker%E9%83%A8%E7%BD%B2YesPlayMusic-1.png" alt="docker部署YesPlayMusic-1" width="400" >}}


2025-04-26_10:22
however在使用两天之后发出了一个警告短信
{{< figure src="/attachment/IMG_3990.png" alt="IMG_3990" width="300" >}}
查了docker插件，确有此问题


2025-05-07_13:41
docker用的是80端口，使用nas的7950端口进行访问
将路由的7950转发给内部7950端口，对应nas端口
从而实现校园网ip+7950端口，访问局域网下nas的docker 80端口


2025-11-08_16:34
再次扫码登录，手机端提示本次登录环境异常，不让我登录了
遂升级到最新的 0.4.9 版本，但是依旧无法解决问题





