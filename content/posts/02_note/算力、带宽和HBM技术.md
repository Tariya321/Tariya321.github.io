---
title: "算力、带宽和HBM技术"
date: 2026-09-16
tags:
  - 后摩尔
publish: true
---

## 1. 算力和 TOPS 单位
---
FLOPS (Floating-Point Operations Per Second)，每秒浮点数运算次数

TOPS (Tera Operations Per Second)，通常是指 INT8 运算的次数

常用 TOPS/W 来评估能效 energy efficiency



## 2. HBM 技术和存储带宽
---
读到了关于 HBM 的一篇推文，想起来好像是中科大的夏令营面试英文翻译题目
大概是一种 DRAM 多层互联的技术（也可以说是一种 chiplet 技术），为内存和逻辑组件提供了高带宽，可服务于 AI GPU。目前的问题是使用的硅中介层费用较高，导致相同容量的存储而价格却是同时期 GDDR5 的三倍左右，所以无法大规模推向市场。
>在考虑其为 AI GPU 带来的带宽优势前，要将价格开销代入计算

**bandwidth = data rate $\times$ bus width**

bandwidth 带宽的单位通常是 bytes per second

data rate 指的是数据 in/out 存储器的速率，业界以 MT/s（MegaTransfers per second）来表示，例如 DDR4-3200 即 3200MT/s

bus width 是指（微型）计算机的数据总线宽度，一般以 bit 表示，通用计算机一般是64bits

例如计算一个 64bit 机器的 DDR4-3200 的带宽
$$\rm{bandwidth = 3200\times 64/8 = 25600\quad MB/s}$$



