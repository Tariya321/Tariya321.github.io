---
title: "PagedAttention"
date: 2026-07-12
tags:
  - vLLM
  - attention
  - Memory-Optimization
  - LLM-Inference
  - KV-Cache
publish: yes
---
https://vllm.ai/blog/2023-06-20-vllm
https://zhuanlan.zhihu.com/p/691038809


## 1. 背景问题

在自回归生成过程中，模型需要缓存每个历史 token 对应的 Key 和 Value，以避免在后续解码步骤中重复计算。随着生成长度增加，KV Cache 会持续增长。

传统 KV Cache 管理通常为每个请求分配一块连续显存，并按照预估的最大序列长度提前预留空间。然而，请求的实际生成长度在运行前通常无法确定，因此容易造成显存预留过多、内存碎片，以及不同请求之间难以高效共享和复用显存的问题。


## 2. 核心方法
PagedAttention 借鉴操作系统的分页机制，将 KV Cache 划分为固定大小的 **blocks（或 pages）**。

对于每个请求，KV Cache 在逻辑上仍对应一段连续的 token 序列，但其实际存储的物理显存块不需要连续。系统根据请求的实际生成长度按需分配物理块，并通过 **block table** 维护逻辑块到物理块的映射关系。

这样可以避免为每个请求提前预留大块连续显存，提高 KV Cache 的显存利用率，并减少内存碎片

![Pasted image 20260929175135](/attachment/Pasted%20image%2020260929175135.png)
https://vllm.ai/blog-assets/figures/annimation1.gif



- PagedAttention 应与 [FlashAttention](/posts/20_Research/topics/LLM%E6%8E%A8%E7%90%86/FlashAttention/) 区分：前者更偏 KV 管理/访问组织，后者更偏 Attention 计算中的 tiling、融合和中间结果读写。




