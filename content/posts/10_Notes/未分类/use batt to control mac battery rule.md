---
title: "use batt to control mac battery rule"
date: 2025-06-09_23:18
tags:
  - macbook
  - battery
  - plugin
publish: yes
---
和iphone一样，控制电池的最大充电容量，从而延长电池寿命

download and open it.
```shell
brew install batt
sudo brew services start batt
```

often used cmd
```shell
batt status
batt limit 80
batt lower-limit-delta 5
batt magsafe-led enable
```

{{< figure src="/attachment/use%20batt%20to%20control%20mac%20battery%20rule.png" alt="use batt to control mac battery rule" width="500" >}}

2025-04-16_16:34目前暂无gui界面，
作者正在考虑实现Al Dente那样的功能

2025-11-09_23:26
some error
```
➜  ~ batt status
Error: failed to get charging status: got 500: "key has no data, check if it is valid"
```
采用手动安装办法，brew 安装版本太老了 `0.3.7`
新版本为 `batt-v0.5.3`

{{< figure src="/attachment/PixPin_2025-11-09_23-39-47.png" alt="PixPin_2025-11-09_23-39-47" width="200" >}}

