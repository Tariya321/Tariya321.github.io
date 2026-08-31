---
title: "docker部署koodo书库"
date: 2025-09-15
tags:
  - docker
publish: yes
---
书库软件 Koodo，开源且有 docker 版本，部署在 nas 上很香！
[Koodo Reader](https://koodoreader.com/en/deploy-docker)

对应的 compose file：[koodo-reader/docker-compose-secret.yml at master · koodo-reader/koodo-reader](https://github.com/koodo-reader/koodo-reader/blob/master/docker-compose-secret.yml)

[【Docker项目实战】使用Docker部署Koodo Reader电子书阅读器-云社区-华为云](https://bbs.huaweicloud.com/blogs/459234)

```
# This docker-compose file uses secrets to manage sensitive information like passwords.
services:
  koodo-reader:
    image: ghcr.io/koodo-reader/koodo-reader:master
    container_name: koodo-reader
    restart: unless-stopped
    ports:
      - "80:80"
      - "8080:8080"
    environment:
      - SERVER_USERNAME=${SERVER_USERNAME:-admin}
      - SERVER_PASSWORD_FILE=${SERVER_PASSWORD_FILE:-my_secret}
      - ENABLE_HTTP_SERVER=false
    volumes:
      # 使用主机目录（推荐）
      - <path_you_want_to_store>:/app/uploads
    secrets:
      - my_secret
secrets:
  my_secret:
    file: ./my_secret.txt
```

其中
默认情况下，**数据源使用 8080**，**Web 前端使用 80**



报错
```
failfull start project 'koodo' err: Container koodo-reader  StartingError response from daemon: driver failed programming external connectivity on endpoint koodo-reader (f3ca65fd0bde5e824b7c62ca1f15f1a6f5a017c1f80c7564173ab3e9aca70dc7): Error starting userland proxy: listen tcp4 0.0.0.0:80: bind: address already in use
```

解决方法：将 80 端口映射到宿主机的其他端口，例如 8081
```
ports:
  - "8081:80"
  - "8080:8080"
```

成功部署
{{< figure src="/attachment/docker%E9%83%A8%E7%BD%B2koodo%E4%B9%A6%E5%BA%93.png" alt="docker部署koodo书库" width="500" >}}

问题是：我希望把原有书籍直接绑定进来，而不是再一次拖入，然后被复制到原有书籍库

打开 8080 链接时，显示 `not found`

而且，貌似上传的文件也并没有自动保存到我定义的目录下
不好用！

