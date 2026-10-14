---
title: "算力、带宽和High-Bandwidth技术"
date: 2026-09-16
tags:
  - 后摩尔
publish: true
---
[AI 的五层蛋糕](/posts/10_Notes/AI%20%E7%9A%84%E4%BA%94%E5%B1%82%E8%9B%8B%E7%B3%95/)

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

## 3. HBF

高带宽闪存（High-Bandwidth Flash，HBF）。它把 NAND 闪存堆叠起来，直接挂到 GPU 的存储通道上，读延迟比 SSD 低约一个数量级，带宽也高出好几倍。


当 GPU 需要 SSD 中的模型权重时，通常需要：
1. 发起存储 I/O 请求。
2. SSD Controller 从 NAND 中读取数据。
3. 数据经过 PCIe 传输。
4. 将数据放入 GPU 可访问的内存（通常是 HBM）。
5. GPU 再从 HBM 读取权重进行计算。

而 HBF 技术则无需再走 PCIe 通道，数据不再需要绕道

![算力、带宽和High-Bandwidth技术](/attachment/%E7%AE%97%E5%8A%9B%E3%80%81%E5%B8%A6%E5%AE%BD%E5%92%8CHigh-Bandwidth%E6%8A%80%E6%9C%AF.png)


