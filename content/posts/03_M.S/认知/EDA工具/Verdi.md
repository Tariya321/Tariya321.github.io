---
title: "Verdi"
date: 2025-06-04_12:55
tags:
  - 仿真
  - 波形
  - synopsys
publish: yes
---

功能列表
1. 与VCS联调[VCS使用指南](/posts/03_M.S/%E8%AE%A4%E7%9F%A5/EDA%E5%B7%A5%E5%85%B7/VCS%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/)
2. 查看电路图




## 1. 加载波形
---
VCS这边要生成一个`.fsdb` 波形文件，Verdi是用来显示和debug的

拉出来waveform

加载示波窗口
{{< figure src="/attachment/v2-2ea667ea918764d5b0367dda4387d6ae_720w.webp" alt="v2-2ea667ea918764d5b0367dda4387d6ae_720w" width="800" >}}

加波形数据
{{< figure src="/attachment/v2-3bf1024e651fa3c8b7367ecb309c6ae9_720w.webp" alt="v2-3bf1024e651fa3c8b7367ecb309c6ae9_720w" width="800" >}}

加信号
{{< figure src="/attachment/v2-ffcd925821425b75b4eb19f62969790e_720w.webp" alt="v2-ffcd925821425b75b4eb19f62969790e_720w" width="800" >}}

## 2. schematic
---
由于DC的电路图已经匹配了标准单元，导致可读性较差

Verdi的可读性较好
>但是对于默认库中不存在的组件，将显示f表示是一个function集合

例如
{{< figure src="/attachment/Verdi.png" alt="Verdi" width="519" >}}

