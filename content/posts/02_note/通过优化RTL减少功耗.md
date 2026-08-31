---
title: "通过优化RTL减少功耗"
src: WX-数字芯片实验室
tags: 低功耗
link: https://mp.weixin.qq.com/s/fAIE1Yy-MvLze8nBRVxV6g
date: 2024-03-31
publish: yes
---
## 1. abstract
在VLSI行业的早期阶段，功耗分析被认为是一种后端活动。但随着芯片复杂性的增加，必须将功耗分析转移到前端阶段，以确保正确的估计和优化，仅在后端阶段进行优化就无法满足要求。
PA was assumed as an backend activity at the early stage of VLSI industry. However, with the increasing of chip complexity, it's a must to move the PA to frontend.

## 2. vocabulary
power hot spots
clk gating
toggle

## 3. mainmatter
 文章是从level来划分low power method的
 - 架构
 - 综合
 - RTL

RTL功耗分析工具
{{< figure src="/attachment/Pasted%20image%2020240331111434.png" alt="Pasted image 20240331111434" width="300" >}}