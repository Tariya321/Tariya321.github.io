---
date: 2026-06-20
tags:
  - AI
  - attention
  - transformer
publish: yes
---
# FlashAttention


## 1. 核心思想
FlashAttention = 用 tiling + online softmax + kernel fusion，把 Attention 的中间矩阵从显存中消除，降低 memory traffic，因此加速 Transformer Attention，尤其是 Prefill 阶段


## 2. 设计背景
要解决的问题：transformer计算速度慢和存储占用高
优化重点：降低访存开销

## 3. 实现技术
FlashAttention 是一种优化 Transformer Attention 计算的算法，核心目标是：在保持数学结果不变的情况下，减少 GPU/AI accelerator 中的内存访问，从而加速 Attention 并降低显存占用

- kernel fusion
	- 将多个独立的GPU操作（Kernel）合并为一个定制的Kernel
	- 融合后的Kernel可以直接在SRAM中完成多个步骤的计算，避免了将中间结果频繁地写回HBM再读出的操作，从而大幅降低了内存带宽的需求
- Backward Recomputation
- softmax tiling
	- 将Softmax的计算分块（Tile）进行，以适应SRAM的容量限制



---
refs:
https://zhuanlan.zhihu.com/p/642962397
https://github.com/Dao-AILab/flash-attention
https://zhuanlan.zhihu.com/p/638468472
https://zifeng-mai.github.io/2026/03/13/FlashAttention/
https://www.cnblogs.com/menkeyi/p/18800377
