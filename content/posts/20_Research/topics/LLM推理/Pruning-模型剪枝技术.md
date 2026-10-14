---
title: "Pruning-模型剪枝技术"
date: 2026-07-18
tags:
  - AI
publish: true
---

什么是剪枝？
---
模型剪枝：将模型中不重要的权重和分支裁剪掉。将权重矩阵中一部分元素变为零元素


剪枝方案
---
阈值剪枝：接近 0 的权重去除

梯度剪枝：梯度小的权重去除


![pruning-剪枝](/attachment/pruning-%E5%89%AA%E6%9E%9D.png)


refs：
https://www.cnblogs.com/passion2021/p/18548165

