---
title: "PVT"
tags:
  - 工艺角
  - PVT
publish: yes
---
芯片的延时一般受三个因素影响：工艺（Process）、电压（Voltage）、温度（Temperature）。合起来称为PVT参数。其中，工艺和温度因素较为复杂，是本文重点介绍的对象。

这个范围以“工艺角”(Process Corners)的形式提供给设计师。其思想是：把NMOS和PMOS晶体管的速度波动范围限制在由四个角所确定的矩形内。如下图所示，这四个角分别是：快NFET和快PFET,慢NFET和慢PFET，快NFET和慢PFET，慢NFET和快PFET。例如，具有较薄的栅氧、较低阈值电压的晶体管，就落在快角附近。
![工艺角](/attachment/%E5%B7%A5%E8%89%BA%E8%A7%92.png)


目前的Process分为不同的corner：
- TT：Typical N Typical P 
- FF：Fast N Fast P 
- SS：Slow N Slow P 
- FS：Fast N Slow P 
- SF：Slow N Fast P

主要是 VDD 和 threshold voltage 不同导致的

N代表NMOS，P代表PMOS。TT表示两者都是正常角，但两者都有快角和慢角，所以还会出现FF、SS、FS、SF四种组合。

> 正常情况下大部分是TT，而以上5种corner在±3σ可以覆盖约99.73%的范围，这种随机性的发生符合正态分布。

一般只需要考虑TT、FF、SS三种情况即可。它们的延时大小关系为：FF>TT>SS。

