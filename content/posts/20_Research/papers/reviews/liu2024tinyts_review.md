---
title: TinyTS：受限初评（未取得全文）
date: 2026-09-12
tags:
  - research/papers
  - paper-review
publish: true
---
## 状态与证据边界

**这不是完整论文 review。TinyTS PDF 尚未下载成功，全文方法、公式、图表和实验没有完成审阅。** 此文件用于单独保留可核实的摘要信息及后续评审要点，不能算作全文评审完成。

文献：Liu 等，*TinyTS: Memory-Efficient TinyML Model Compiler Framework on Microcontrollers*，HPCA 2024，848–860。[DOI](https://doi.org/10.1109/HPCA57654.2024.00070)。

证据来源为[作者项目仓库](https://github.com/nycu-caslab/TinyTS)与[阳明交通大学出版记录及摘要](https://scholar.nycu.edu.tw/en/publications/tinyts-memory-efficient-tinyml-model-compiler-framework-on-microc/)。检索过作者页面、公开仓库和开放获取索引；本环境访问 IEEE 全文端点失败，现有索引未提供可用开放 PDF。访问失败不能说明论文质量有问题，也不证明世界上没有其他合法副本。

按照 academic-paper 五维框架记录；`criteria_binding_unavailable`。没有进行 TinyTS 代码构建或板级实验，也没有假设仓库 README 覆盖正式论文全部内容。

## 可支持的工作概述

公开资料将 TinyTS 描述为面向 MCU 上 CNN 的模型编译框架，通过 tensor splitting 在计算图上以较小张量执行，减少峰值 SRAM 使用。作者项目说明强调保持模型准确率与较小延迟代价。

机构摘要报告：在 9 个 TinyML 模型上，峰值 SRAM 使用最多降低 5.92×；相对 patch-based inference，速度的几何平均提升为 8.83×。**前者为最大值，后者为几何平均，而且比较指标和基线不同，不能拼成同一模型同时取得的结果。** README 所述约 5% 延迟开销也不能直接与上述速度收益相减，因为基线可能不同。

## 五维判断

| 维度 | 状态 | 原因 |
|---|---|---|
| Originality | 暂不能确定 | 摘要能说明问题和方案方向，不能完成与最近邻编译/patch 推理方法的差异核对 |
| Methodological Rigor | NOT_ASSESSED | 未取得全文，无法检查切分、调度、边界处理及内存规划方法 |
| Evidence Sufficiency | NOT_ASSESSED | 已有摘要数字，但缺逐模型表格、硬件设置和测量口径 |
| Argument Coherence | NOT_ASSESSED | 不以摘要替代正文论证 |
| Writing Quality | NOT_ASSESSED | 未阅读全文，无法评价组织和表达质量 |

## 全文取得后应核对的重点

1. **正确性与算子覆盖。** 切分是否保持卷积 halo、padding、stride、残差连接及量化舍入语义；哪些算子和图结构不支持。需要算法、实现约束与正确性测试，不能现在认定其缺失。
2. **峰值内存口径。** SRAM 是否包括 tensor arena、scratch buffer、栈、运行时元数据与 DMA 缓冲；权重放在 Flash 还是 SRAM。比较应采用相同硬件与相同可用内存预算。
3. **延迟收益的基线。** 分别核对未切分推理、patch-based inference 和其他内存优化方法；说明不适配内存的基线是否通过额外处理才可执行。
4. **优化代价。** tensor splitting 是否引入重复计算、权重重读、额外 kernel 调用，何种张量形状开始得不偿失；搜索时间及编译成本有多大。
5. **实验泛化。** 逐模型 SRAM—延迟—准确率曲线、MCU/编译器/内核版本、运行重复次数和测量误差；区分“最多降低”与通常表现。

这些是待核查问题，不是已证实的论文缺陷，因此不分配 Major/Critical 严重度，也不给录用或修订判断。

## 当前结论

TinyTS 的公开摘要与仓库表明其与 MCU 内存受限推理问题直接相关，值得进一步阅读；现有材料不足以形成负责任的完整学术评审。**待补：合法可访问的论文 PDF，随后进行全文阅读并替换本受限初评。** 当前评审置信度低。