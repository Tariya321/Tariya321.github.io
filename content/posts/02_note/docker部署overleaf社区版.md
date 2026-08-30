---
title: "docker部署overleaf社区版"
date: 2025-09-28
tags:
  - docker
publish: yes
---
## 1. main
official链接[^1]
民间链接[^2]

注意修改其中的 healthcare 部分为[^4]
```
healthcheck:
            test: "mongosh --quiet --eval 'rs.hello().setName ? rs.hello().setName : rs.initiate({_id: \"overleaf\",members:[{_id: 0, host:\"mongo:27017\"}]})'"
```
> 也可以采用另一种方法[^5]


端口为 `8088:80`

pull 某个包出问题
```
✘ sharelatex Error Get "... 90.1s  
Error response from daemon: Get "https://registry-1.docker.io/v2/": context deadline exceeded
```

更新 docker 加速，加入镜像站点[^3]
```
/etc/docker/daemon.json
```

重启 docker 服务
```
sudo systemctl daemon-reexec
sudo systemctl restart docker
```

首先进入 launch 界面
```
http://localhost/launchpad
```

account
```
hongzhilian@outlook.com
tariya@321
```


再进入 login 界面
```
http://localhost/login
```


注册登录后，无法编译文档
且其他用户注册不方便，重设的网址不存在


更新包和新下载包: 进入docker的shell
```
tlmgr option repository https://mirrors.cloud.tencent.com/CTAN/systems/texlive/tlnet/
tlmgr install scheme-full
tlmgr path add
```

## 2. 参考飞牛nas设置

https://club.fnnas.com/forum.php?mod=viewthread&tid=47589

## 3. 基于toolkit

```
git clone https://github.com/overleaf/toolkit.git ./overleaf-toolkit
```




## 4. reference
[^1]: https://github.com/overleaf/overleaf/blob/main/docker-compose.yml

[^2]: https://flxdu.cn/Notes/07-Docker/Overleaf%20in%20Docker.html

[^3]: https://github.com/dongyubin/DockerHub

[^4]: https://github.com/overleaf/overleaf/issues/1120#issuecomment-1714229816

[^5]: https://github.com/overleaf/overleaf/issues/1120#issuecomment-1623295314
