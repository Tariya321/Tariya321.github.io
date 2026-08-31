---
title: "iVentoy"
date: 2025-09-15
tags:
  - docker
  - 系统
publish: yes
---
半夜刷到在局域网下用 nas docker 给任意主机刷机的视频[^1]
心动了：~


由大佬 szabis维护的docker 地址：[szabis/iventoy - Docker Image | Docker Hub](https://hub.docker.com/r/szabis/iventoy)

部署中报错信息
```
WARN[0000] /volume4/docker/iventoy/docker-compose.yaml: `version` is obsolete 
[+] Running 1/1
 ✘ iventoy Error Get "https://registry-1.docker.io/v2/":...               15.0s 
Error response from daemon: Get "https://registry-1.docker.io/v2/": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
```

参考[^2], 编辑 json 文件
```
sudo vi /etc/docker/daemon.json
```

添加以下镜像站
```
"registry-mirrors":["https://docker.m.daocloud.io/","https://huecker.io/","https://dockerhub.timeweb.cloud","https://noohub.ru/","https://dockerproxy.com","https://docker.mirrors.ustc.edu.cn","https://docker.nju.edu.cn","https://xx4bwyg2.mirror.aliyuncs.com","http://f1361db2.m.daocloud.io","https://registry.docker-cn.com","http://hub-mirror.c.163.com","https://docker.mirrors.ustc.edu.cn","https://pee6w651.mirror.aliyuncs.com","https://mirror.baidubce.com","https://docker.xuanyuan.me"]
```
> 注意遵循 json 语法，否则可能出现 docker 异常

之后重启服务
```
sudo systemctl daemon-reexec
sudo systemctl restart docker
```
2025-09-15_19:03没有一次性装成功，改了 dns 和镜像站之后没有解决 pulling 时间很长的问题



[^1]: https://www.bilibili.com/video/BV1kdabzJEpn/?buvid=XU5A8545D7C47B206A00974204B828274B91A&from_spmid=tm.recommend.0.0&is_story_h5=false&mid=UrkH7GiRN1fpDXMd0uilkw%3D%3D&p=1&plat_id=116&share_from=ugc&share_medium=android_hd&share_plat=android&share_session_id=ac467b1e-370c-4b49-a18a-6a7385b50f2f&share_source=WEIXIN&share_tag=s_i&spmid=united.player-video-detail.0.0&timestamp=1757868178&unique_k=AgHT5gs&up_id=1995424953

[^2]: [解决Error Get "https://registry-1.docker.io/v2/":环境报错问题 - 白码一号 - 博客园](https://www.cnblogs.com/OneSeting/p/18532166)
