---
title: "NPU的软硬件设计层次"
date: 2026-06-15
tags:
  - npu
publish: yes
---

编译器将由“算子”组合而成的高层 Task 图结构，转化、降维并调度成了由底层硬件“ISA 指令”构成的执行程序

|层次|例子|描述|
|---|---|---|
|Graph|ResNet、Transformer 计算图|描述算子之间的数据流与依赖关系|
|Operator|Conv2D、MatMul、ReLU|模型中的功能级计算操作|
|Kernel|conv_kernel、gemm_kernel|Operator 在特定后端上的具体软件实现|
|ISA|load、store、vfmacc、matrix_mul、dma_start|定义机器可执行的指令及其语义|
|Microarchitecture|MAC/PE 阵列、Vector Unit、Register File、SRAM Buffer、DMA、Pipeline|定义 ISA/计算功能在硬件内部如何组织和执行|
|RTL|Verilog/SystemVerilog 中的 MAC array、DMA controller、SRAM interface|微架构的周期精确硬件实现|
|Circuit / Physical|乘法器、加法器、触发器、SRAM bitcell、标准单元、版图|具体晶体管/逻辑门级实现以及布局布线|


- 算子只定义了**数学功能**，不关心怎么实现
- Kernel 本质上是某个算子的具体执行实现（Implementation）
	- 根据平台、实现方式，一个算子可以有很多个 kernel

![NPU的软硬件设计层次-1](/attachment/NPU%E7%9A%84%E8%BD%AF%E7%A1%AC%E4%BB%B6%E8%AE%BE%E8%AE%A1%E5%B1%82%E6%AC%A1-1.png)


在将高层算子编译为底层指令之前，将会在逻辑和数学层面上减少计算量（算子融合）




## 1. 关联阅读

- [从 PyTorch 算子到 NPU 硬件：中间发生了什么？](/posts/20_Reserch/LLM%E6%8E%A8%E7%90%86/%E4%BB%8E%20PyTorch%20%E7%AE%97%E5%AD%90%E5%88%B0%20NPU%20%E7%A1%AC%E4%BB%B6%EF%BC%9A%E4%B8%AD%E9%97%B4%E5%8F%91%E7%94%9F%E4%BA%86%E4%BB%80%E4%B9%88%EF%BC%9F/)：解释 Operator、Schedule、Kernel、ISA 与硬件之间的连接。