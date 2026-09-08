---
title: LLM 中的知识蒸馏
aliases:
  - Knowledge Distillation in LLMs
  - 大语言模型知识蒸馏
tags:
  - AI/LLM
  - AI/knowledge-distillation
created: 2026-09-08
type: concept-note
publish: yes
---
> [!abstract] 核心概念
> **Knowledge distillation trains a student model using supervision supplied by a teacher model.**
>
> 知识蒸馏让学生模型（student）从教师模型（teacher）提供的监督信号中学习。常见目标是让较小模型以更低推理成本完成特定任务，但学生不一定更小，也不保证复现教师的全部能力。

本文面向理解基本语言模型训练的读者，介绍核心概念、数学目标与实践判断，不作为最新方法排行榜。

## 1. “知识”究竟是什么？

这里的“知识”不是从教师参数中提取出来的一份事实清单，而是可以用于训练的信号，例如教师给出的答案、下一 token 的概率分布，或可访问的中间表示。

假设任务是识别一段评论的情感。只给出“正面”标签，学生只能学习这个离散结果；给出“正面 0.7、中性 0.25、负面 0.05”，还提供了候选答案之间的相对关系。传统蒸馏把这种更丰富的分布信息作为监督来源。该思想的经典介绍见 [Hinton 等，2015](https://arxiv.org/abs/1503.02531)。

在生成式 LLM 中，同样可以对每一步的下一 token 分布进行匹配，也可以只保留教师生成的一整段文字。

> [!important] Behavioral transfer ≠ complete knowledge transfer
> **Matching a teacher on a training distribution does not imply recovering all of its knowledge or capabilities.**
>
> 蒸馏覆盖了哪些问题、学生有多少容量、监督是否可靠，都会影响最终能迁移哪些能力。

## 2. 两条主要路线

| 维度 | Response / sequence-level distillation | Token-level distribution distillation |
| --- | --- | --- |
| 教师提供什么 | 完整答案或带解释的答案 | 给定上下文时的下一 token 概率 |
| 学生学什么 | 提高教师文本的生成概率 | 让自身概率分布接近教师分布 |
| 常见训练形式 | 对教师文本做 SFT | 最小化 KL 等分布差异，可混合 SFT |
| 访问要求 | 只获得文本也能开展 | 通常需要 logits 或足够完整的概率信息 |
| 主要局限 | 一条答案丢失了其他候选的分布信息 | 词表对齐、教师计算和数据存储更复杂 |

序列级蒸馏在神经机器翻译中已有系统研究：使用教师生成的序列作为学生监督目标，是其重要形式。参见 [Kim 与 Rush，2016](https://aclanthology.org/D16-1139/)。

“黑盒蒸馏”通常指只能访问教师输出，“白盒蒸馏”通常指能访问 logits 或内部信息。但这些名称的用法并不完全一致，读论文时应直接检查实际可访问的信息。

### 2.1. 仅学习教师答案

设输入为 $x$，教师生成的回答为 $\hat y=(\hat y_1,\ldots,\hat y_m)$。学生使用负对数似然训练：

$$
\mathcal L_{\text{response}}
=-\mathbb E_{(x,\hat y)\sim\mathcal D_T}
\left[\sum_{t=1}^{m}\log p_S(\hat y_t\mid x,\hat y_{&lt;t})\right].
$$

其中 $\mathcal D_T$ 是教师生成并可能经过筛选的数据集。这里写的是序列内求和，实际实现也可能按有效 token 数归一化。

训练时，学生看到的是目标答案的前缀；不是让学生先自由回答，再按整段文字的相似程度打分。

> [!note] Distillation and SFT describe different aspects
> **SFT describes a training procedure; distillation describes the source and purpose of supervision.**
>
> 用教师答案做 SFT，可以同时属于微调与蒸馏。使用人工标签做 SFT，则不自动构成教师模型蒸馏。

教师采样、解码策略和筛选会改变学生实际学习的数据分布。只保留正确、简短的答案，与原样模仿教师全部输出，并不是同一个学习目标。

### 2.2. 学习教师的概率分布

令 $c=(x,y_{&lt;t})$ 表示某一步的上下文，$z_T,z_S$ 为教师与学生的 logits，$\tau>0$ 为温度：

$$
p_T^{(\tau)}(v\mid c)
=\operatorname{softmax}\left(z_T(c)/\tau\right)_v.
$$

学生分布同理定义。常见的 forward KL 目标为：

$$
\mathcal L_{\text{KD}}
=\tau^2\,\mathbb E_{c\sim\mathcal D_c}
\left[\sum_{v\in V}p_T^{(\tau)}(v\mid c)
\log\frac{p_T^{(\tau)}(v\mid c)}{p_S^{(\tau)}(v\mid c)}\right].
$$

这里 $\mathcal D_c$ 是训练上下文的分布；它可以来自已有数据、教师生成序列或学生生成序列。温度升高通常让分布更平滑，$\tau^2$ 是经典设置中用于补偿梯度尺度的系数，并非所有方法都必须采用。温度蒸馏的基础见 [Hinton 等，2015](https://arxiv.org/abs/1503.02531)。

也可以混合真实标签监督：

$$
\mathcal L=(1-\alpha)\mathcal L_{\text{gold}}+\alpha\mathcal L_{\text{KD}},\qquad 0\le\alpha\le1.
$$

> [!warning] Vocabulary alignment is an assumption
> **Direct token-level KL requires distributions over aligned events.**
>
> 上面的公式假设教师和学生的 token 事件可对齐。不同 tokenizer 的“第 100 个 token”不一定代表同一文本，不能直接逐项计算 KL。文本级蒸馏可以让学生重新分词；跨 tokenizer 的分布蒸馏则需要额外方法。仅有 top-k 概率，也不等于拥有完整分布。

## 3. Forward KL 与 Reverse KL

先固定同一个上下文，省略温度符号：

$$
D_{\mathrm{KL}}(p_T\Vert p_S)
=\sum_v p_T(v)\log\frac{p_T(v)}{p_S(v)},
$$

$$
D_{\mathrm{KL}}(p_S\Vert p_T)
=\sum_v p_S(v)\log\frac{p_S(v)}{p_T(v)}.
$$

第一种按教师概率加权；第二种按学生概率加权。两者的优化行为不同。

| 目标                                         | 直观倾向              | 需要注意                 |
| ------------------------------------------ | ----------------- | -------------------- |
| Forward KL：$D_{\mathrm{KL}}(p_T\Vert p_S)$ | 尽量覆盖教师认为可能的输出     | 学生容量不足时，覆盖多个模式可能带来折中 |
| Reverse KL：$D_{\mathrm{KL}}(p_S\Vert p_T)$ | 减少学生在教师低概率区域分配的概率 | 可能减少多样性，忽略部分有效模式     |

**“Mode-covering” and “mode-seeking” are useful intuitions, not universal guarantees.** 实际结果还取决于模型容量、采样、上下文分布和优化方法。

[MiniLLM](https://arxiv.org/abs/2306.08543) 研究了用 reverse KL 蒸馏生成式语言模型，并设计相应的优化方法。不能把它简化为“把 KL 两个参数交换一下就一定更好”：特别是在序列目标中，采样分布本身也依赖学生。

## 4. 为什么学生自己生成的前缀很重要？

学生在训练时可能总是接触教师写好的前缀，部署后却必须接着自己生成的文字继续写。一旦前面偏离教师轨迹，后面的上下文就可能属于训练中很少见的区域。

**On-policy distillation uses student-generated trajectories to obtain teacher supervision on states the student actually visits.**

例如，学生先生成一个解题过程，教师再对这些前缀提供下一步分布监督。目标是让训练覆盖学生实际会遇到的状态。[GKD](https://arxiv.org/abs/2306.13649) 系统研究了这一思路及可选的分布匹配目标。

> [!important] 两个独立的设计维度
> **Who generates the training prefixes?** 和 **Which divergence is minimized?** 是不同问题。
>
> On-policy 不等于 reverse KL；使用 forward KL 也可以在学生生成的前缀上训练。教师是否实时运行，与前缀是否由当前学生生成，也不是同一个判断标准。

## 5. 推理过程蒸馏能迁移什么？

只提供最终答案，学生获得的是结果监督；提供解释或推理步骤，则增加了中间文本监督。

例如，对“单价 12 元，买 3 件需要多少钱”，训练目标可以是“36 元”，也可以是“总价等于单价乘数量，12 × 3 = 36，因此是 36 元”。这个简单例子只用于说明数据形式，不代表实验效果。

[Distilling Step-by-Step](https://arxiv.org/abs/2305.02301) 使用教师提取的 rationales 作为额外监督，通过多任务训练学习解释与标签预测，并在论文评估的任务中取得了数据效率收益。它并不只是要求学生把每条长解释照抄到最终回答里。

> [!warning] Rationale imitation is not proof of faithful reasoning
> **A generated rationale is an observable training artifact, not a verified trace of the teacher's internal computation.**
>
> 学生能生成像样的解释，并不能单独证明其具有正确、稳健的推理能力。应使用未见题目、条件变化和可验证答案来评估。

实践中需要分别检查最终答案是否正确、步骤是否成立、解释长度是否必要。保留冗长但错误的过程，可能把错误和高输出成本一起传给学生。

## 6. 与相邻概念的区别

| 概念 | 主要改变什么 | 与蒸馏的关系 |
| --- | --- | --- |
| Fine-tuning / SFT | 用特定数据继续更新参数 | 可以承载蒸馏，也可以完全不使用教师 |
| Quantization | 数值表示精度 | 可与蒸馏结合，低精度本身不等于知识迁移 |
| Pruning | 移除参数或计算结构 | 蒸馏可用于帮助压缩后的模型恢复效果 |
| RAG | 推理时引入检索信息 | 检索本身不把知识训练进参数；检索增强的教师也可生成蒸馏数据 |
| Preference optimization | 利用偏好信号改变输出倾向 | 教师可提供偏好，但不能把所有偏好训练都称为传统 KD |
| Synthetic data training | 使用生成的数据训练 | 若明确用教师输出迁移其行为，可以属于广义蒸馏；“合成”本身只描述数据来源 |

**Compression is a common goal of distillation, not its definition.** 同规模模型、自蒸馏或多教师设置也可以使用蒸馏思想。

## 7. 一个可执行的实验框架

以下是方法设计建议，不是某篇论文的复现实验配置。

1. **限定目标任务。** 例如中文客服分类、结构化抽取或数学短题；先确定部署成本和可接受的质量损失。
2. **先划分评测集。** 按来源、主题或时间隔离，并检查近重复，避免教师生成和筛选流程污染测试集。
3. **评估教师。** 检查它在目标任务中的正确率；对于代码、计算等任务，优先用执行结果或明确规则验证。
4. **构造覆盖性数据。** 包含典型问题、边界条件、歧义输入与合理拒答，避免只收集教师最擅长的简单样本。
5. **选择监督信号。** 只有文本时先建立 response SFT 基线；能访问并对齐概率时，再比较分布蒸馏。
6. **设置对照实验。** 比较原始学生、仅真实数据微调的学生、蒸馏学生和教师；尽量控制训练预算和数据规模。
7. **检查部署收益。** 同时测任务质量、首 token 延迟、端到端延迟、输出长度和峰值内存。

最有解释力的消融通常是：只学答案与加入解释、筛选前与筛选后、真实数据与真实数据加蒸馏数据。一次改变一个主要因素，更容易判断收益来自哪里。

> [!tip] Match the evaluation to the deployment
> **A smaller parameter count does not guarantee lower end-to-end latency.**
>
> 输出变长、批处理方式、硬件和推理实现都会影响实际速度。比较推理效率时应固定这些条件，并报告输出长度。

## 8. 常见误区

**“学生不可能超过教师。”** 学生可能在特定评测上超过教师，例如受益于任务专门化、真实标签或筛选数据；这不意味着全面能力超过教师。

**“生成的数据越多越好。”** 重复、错误和覆盖偏差可能扩大。数据数量应该与正确率、难度和任务覆盖一起看。

**“只要模型更小，节约就已经实现。”** 蒸馏还需要教师生成、训练与评测成本；部署调用量足够大时，推理成本下降才可能抵消这些前期开销。

**“蒸馏会保留所有通用能力。”** 特定任务训练可能使其他能力退化。除了目标任务，还应保留基本语言能力和业务边界的回归评测。

## 9. 阅读路径

| 顺序 | 原始文献 | 阅读重点 |
| --- | --- | --- |
| 1 | [Hinton et al., Distilling the Knowledge in a Neural Network, 2015](https://arxiv.org/abs/1503.02531) | Soft targets、温度和经典蒸馏动机 |
| 2 | [Kim & Rush, Sequence-Level Knowledge Distillation, 2016](https://aclanthology.org/D16-1139/) | 从 token 分布到序列监督 |
| 3 | [Hsieh et al., Distilling Step-by-Step, 2023](https://arxiv.org/abs/2305.02301) | Rationales 如何成为额外监督 |
| 4 | [Gu et al., MiniLLM, 2023 预印本](https://arxiv.org/abs/2306.08543) | Reverse KL 与生成式模型优化 |
| 5 | [Agarwal et al., On-Policy Distillation of Language Models, 2023 预印本](https://arxiv.org/abs/2306.13649) | GKD、学生轨迹与训练推理分布差异 |

阅读方法论文时，可以回到四个问题：**What is transferred? Which prefixes are used? What objective is optimized? What is actually evaluated?**

### 仓库内的相关笔记

以下链接指向仓库中已有且已核对内容的笔记，可结合本文阅读：

- [LoRA低秩适应](/posts/20_Reserch/LLM%E6%8E%A8%E7%90%86/LoRA%E4%BD%8E%E7%A7%A9%E9%80%82%E5%BA%94/)：结合第 2、6 节理解参数高效微调。**LoRA specifies how parameters are adapted; distillation specifies the teacher-derived supervision.** 两者可以结合，但 LoRA 本身不等于蒸馏。
- [量化方案](/posts/20_Reserch/LLM%E6%8E%A8%E7%90%86/%E9%87%8F%E5%8C%96%E6%96%B9%E6%A1%88/)：结合第 6 节区分低精度表示与教师监督；该笔记介绍数据格式、位宽和量化表示，有助于理解蒸馏后进一步量化的部署选择。
- pruning-模型剪枝技术：结合第 6 节比较剪枝与蒸馏；该笔记介绍权重裁剪及阈值、梯度剪枝，可作为另一条模型压缩路线的入门。
- LLM inference：结合第 7 节理解部署评测中的 prefill、decode、TTFT、ITL 和 KV Cache，检查蒸馏后的收益体现在哪个推理阶段。
