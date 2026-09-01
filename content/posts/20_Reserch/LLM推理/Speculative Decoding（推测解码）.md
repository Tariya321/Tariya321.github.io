---
title: Speculative Decoding（推测解码）
date: 2026-08-30
tags:
  - llm/inference
  - llm/decoding
  - accelerator
  - systems
aliases:
  - Speculative Decoding
  - 推测解码
publish: yes
---

> [!abstract] 一句话
> 让便宜的 **Draft Model** 先连续猜测一小段 token，再让昂贵的 **Target Model** 一次性验证；正确的 draft token 直接接受，首次拒绝后丢弃其后的 draft 后缀并从正确上下文继续生成。

> [!tip] 核心目标
> Speculative decoding 主要减少的是 Target Model 的**串行 decode 调用次数**，而不是让 Target Model 完全不运行。它用额外的 draft 计算，换取 Target Model 更高的并行度、权重读取复用率和硬件利用率。

## 1. 背景

### 1.1. 自回归生成

大语言模型（LLM）的生成过程具有严格的自回归依赖：

$$
x_t \sim p_T(x_t \mid x_{<t})
$$

其中：

- $x_t$：第 $t$ 个 token；
- $x_{<t}$：此前已经生成的 token；
- $p_T$：Target Model 的条件概率分布。

生成第 $t$ 个 token 之前，必须先知道前面所有 token。因此标准 decode 通常是：

~~~text
Target Model → token 1
Target Model → token 2
Target Model → token 3
Target Model → token 4
...
~~~

每个 token 都依赖上一个 token，无法直接把未来 token 全部并行生成。

### 1.2. Prefill 与 Decode

LLM 推理通常分成两个阶段：

| 阶段 | 输入形态 | 主要特征 | 常见瓶颈 |
|---|---|---|---|
| Prefill | 一次输入较长 prompt | 序列内并行计算 | 计算量、显存容量 |
| Decode | 每次生成一个或少量 token | 跨时间步串行 | 权重读取、KV cache、内存带宽 |

Speculative decoding 主要优化的是 **decode 阶段**。

在低 batch 或单用户场景中，decode 常具有以下特点：

- batch size 较小；
- 每次只处理一个新 token；
- 每一步都需要访问大模型参数；
- 矩阵运算的有效矩阵维度较小；
- arithmetic intensity 较低；
- 经常接近 **memory-bandwidth-bound**；
- GPU/NPU 的大规模并行计算能力难以充分利用。

因此，LLM decode 的一个核心瓶颈是：

> **单 token 串行执行，导致昂贵的模型权重读取和调度开销被重复支付。**

## 2. 核心思想

Speculative Decoding 可以概括为：

> **让一个更便宜的小模型提前猜测多个 token，再让大模型一次性验证这些 token。**

其中：

- 小模型称为 **Draft Model**，记作 $D$ 或 $q$；
- 大模型称为 **Target Model**，记作 $T$ 或 $p$；
- 每轮 draft 的 token 数量称为 **speculation length** 或 **draft length**，记作 $K$。

### 2.1. 基本流程

~~~text
已生成上下文 c
        │
        ▼
┌────────────────┐
│  Draft Model D │  便宜、快速、逐 token 猜测
└───────┬────────┘
        │
        ▼
   y1  y2  y3  y4   （K 个 draft token）
        │
        ▼
┌────────────────────┐
│ Target Model T     │  一次 forward 验证整段候选
│ Verification       │
└────────┬───────────┘
         │
    ┌────┴────┐
    ▼         ▼
 接受前缀    首次拒绝
    │         │
    ▼         ▼
 一次推进    修正该 token，丢弃后缀
~~~

例如 Draft Model 生成：

~~~text
Draft:
A  B  C  D
~~~

Target Model 验证后：

~~~text
Target:
✓  ✓  ✓  ×
~~~

那么前三个 token 可以一次性接受：

~~~text
A B C
~~~

第 4 个 token 不能直接使用，需要由 Target Model 生成或根据精确采样规则修正。然后从修正后的上下文继续下一轮。

### 2.2. 它与“只调用大模型处理困难 token”的区别

标准 speculative decoding：

~~~text
小模型先猜一段
        ↓
大模型验证整段
        ↓
接受正确前缀，修正首次错误
~~~

它的特点是 Target Model 仍然参与每一轮验证，因此可以保持 Target Model 的输出分布。

而 token-level cascade 更接近：

~~~text
小模型计算一个 token
        │
        ├─ 置信度高 → 直接输出
        └─ 置信度低 → 调用大模型
~~~

后者可能进一步减少 Target Model 的调用，但不天然保持 Target Model 的输出分布，而且会遇到 Target Model KV cache 不连续的问题。两者可以组合，但概念上应分开。

## 3. 为什么这样可以更快？

### 3.1. 减少 Target Model 的串行次数

普通 autoregressive decoding：

~~~text
Target → T1
Target → T2
Target → T3
Target → T4
~~~

需要执行 4 次 Target Model 的逐 token decode。

Speculative decoding：

~~~text
Draft:
T1 T2 T3 T4

Target:
一次验证 T1 T2 T3 T4
~~~

如果 4 个 token 全部接受，大模型一次验证就能推进多个 token。

### 3.2. 不是简单地减少 Target FLOPs

Target Model 验证 $K$ 个 token 仍然要进行与这 $K$ 个位置相关的计算。真正的收益来自：

1. **跨位置并行**：多个位置可以在一次 forward 中同时计算；
2. **权重读取复用**：同一层权重可以服务多个 token 位置；
3. **矩阵形态改善**：单 token 的小矩阵运算更接近 GEMV，多 token 验证更接近 GEMM；
4. **调度次数减少**：减少 kernel launch、同步和模型唤醒次数；
5. **串行关键路径缩短**：Target Model 不必为每个 token 单独往返一次。

因此更准确的表述是：

> **用额外的 Draft Model 计算，换取 Target Model 更好的批处理形态与更少的串行调用。**

### 3.3. 一个简单的时间线

~~~text
普通 decode
时间 →
┌─────┐┌─────┐┌─────┐┌─────┐
│ T1  ││ T2  ││ T3  ││ T4  │   4 次 Target 调用
└─────┘└─────┘└─────┘└─────┘

Speculative decode
时间 →
┌─D1─┬─D2─┬─D3─┬─D4─┐ ┌────────────── Target verify ──────────────┐
└────┴────┴────┴────┘ └───────────────────────────────────────────┘
                              一次验证，接受若干 token
~~~

注意：Draft Model 的 4 个 token 通常仍然是串行生成的，只是 Draft Model 便宜很多；Target Model 的验证阶段则可以并行处理这些位置。

## 4. 为什么 Target Model 可以“一次验证多个 token”？

这是理解 speculative decoding 最关键的一点。

### 4.1. Transformer 在训练时本来就会并行计算多个位置

假设已有上下文 $c$，Draft Model 产生：

$$
y_1,y_2,\dots,y_K
$$

Target Model 需要评估：

$$
p_T(y_i \mid c,y_{<i}), \quad i=1,\dots,K
$$

其中：

$$
y_{<i}=(y_1,\dots,y_{i-1})
$$

把上下文和候选序列一起送入 Target Model 后，它可以同时得到多个位置的 logits。对 $K$ 个 draft token，Target 需要的分布是：

~~~text
输入位置：    c   y1  y2  y3  y4
预测分布：   q1  q2  q3  q4  q5
目标 token： y1  y2  y3  y4  next
~~~

这里：

$$
q_i(\cdot)=p_T(\cdot\mid c,y_{<i})
$$

例如：

$$
q_1(\cdot)=p_T(\cdot\mid c)
$$

$$
q_2(\cdot)=p_T(\cdot\mid c,y_1)
$$

$$
q_3(\cdot)=p_T(\cdot\mid c,y_1,y_2)
$$

其中 $q_1,\dots,q_K$ 分别用于验证 $y_1,\dots,y_K$；若整个 draft block 都被接受，最后一个位置的 $q_{K+1}$ 还可以直接用于产生下一个 Target token。

### 4.2. Causal Mask 保证因果性

Transformer 使用 causal mask，使位置 $i$ 只能看到自己及其之前的位置：

~~~text
可见性矩阵：

        c   y1  y2  y3
c       ✓   ×   ×   ×
y1      ✓   ✓   ×   ×
y2      ✓   ✓   ✓   ×
y3      ✓   ✓   ✓   ✓
~~~

因此 Target Model 虽然在一次 forward 中处理多个位置，但不会偷看未来 token。它计算的仍然是合法的自回归条件概率。

### 4.3. 一次 forward 不等于一次 token 计算

“一次验证多个 token”容易被误解为 Target Model 只做了一个 token 的工作。更准确地说：

- 计算图中有多个位置；
- 这些位置在序列维度上并行执行；
- 线性层可以复用权重读取；
- 目标是把多个小的 decode 工作合并成一个更高效的 block forward。

所以它优化的是**执行形态和串行深度**，而不是把 $K$ 个 token 的全部理论计算量变成常数。

## 5. Draft Model

### 5.1. Draft Model 的要求

Draft Model 通常需要满足：

1. 比 Target Model 小很多；
2. 单 token 推理速度明显更快；
3. 与 Target Model 使用兼容的 tokenizer 和词表；
4. 对 Target Model 的高概率路径有较好的拟合；
5. 能够低延迟地产生连续的 $K$ 个候选 token。

常见形式包括：

- 独立的小型语言模型；
- Target Model 的蒸馏模型；
- 共享部分结构的轻量模型；
- n-gram 或 lookup-based draft；
- 额外的 decoding head 或树状候选生成模块。

例如：

~~~text
Target Model:
70B

Draft Model:
1B / 3B / 7B
~~~

### 5.2. Draft Model 太大

Draft Model 太大时，其生成成本上升：

$$
C_D\uparrow
$$

即使 acceptance rate 较高，Draft Model 自身的时间也可能抵消 Target Model 的收益。

### 5.3. Draft Model 太小

Draft Model 太小时，预测质量下降：

$$
\text{agreement}(D,T)\downarrow
$$

从而导致：

- acceptance rate 下降；
- 每轮真正推进的 token 数减少；
- draft token 的计算更容易被丢弃；
- Target Model 的验证开销仍然存在。

因此存在核心 trade-off：

$$
\boxed{
\text{Draft Cost}
\; \leftrightarrow \;
\text{Acceptance Rate}
\; \leftrightarrow \;
\text{Target Verification Cost}
}
$$

重要的是：Draft Model 预测得不好通常不会破坏精确 speculative sampling 的正确性，只会让加速效果变差。

## 6. Acceptance Rate

### 6.1. 定义

设第 $r$ 轮生成 $K_r$ 个 draft token，其中被 Target Model 接受的连续前缀长度为 $A_r$，则长期 acceptance rate 可以定义为：

$$
\alpha=
\frac{\sum_r A_r}{\sum_r K_r}
$$

简单例子：

~~~text
Draft 生成：10 tokens
Target 接受：8 tokens
~~~

则：

$$
\alpha=0.8
$$

### 6.2. 接受的是“前缀”，不是任意集合

若验证结果为：

~~~text
✓  ✓  ×  ✓  ✓
~~~

实际只能接受：

~~~text
token 1, token 2
~~~

第 4、5 个 token 即使单独看起来合理，也不能直接提交。接受长度是首次拒绝之前的连续前缀：

$$
A=
\begin{cases}
j-1, & \text{第 }j\text{ 个 token 首次被拒绝}\\
K, & \text{整个窗口都被接受}
\end{cases}
$$

也就是说，$A$ 是从窗口开头开始连续通过验证的 token 数量。

### 6.3. $\alpha$ 的解释

当：

$$
\alpha\rightarrow1
$$

说明 Draft Model 与 Target Model 在当前 workload 上高度一致，speculative decoding 通常更有效。

当：

$$
\alpha\rightarrow0
$$

则可能出现：

~~~text
Draft:  A  B  C  D
Target: ×
~~~

此时 Draft Model 的计算大部分被浪费，而且还增加了调度、缓存和数据搬运开销。

### 6.4. Acceptance rate 不是固定模型属性

$\alpha$ 会受到以下因素影响：

- prompt 和上下文内容；
- 代码、数学、对话等任务类型；
- temperature；
- top-k / top-p；
- tokenizer；
- Draft/Target 的训练关系；
- batch size 与调度状态；
- speculation length $K$。

因此不要只报告一个全局 acceptance rate。更有意义的做法是按 workload、上下文长度、采样参数和 $K$ 分组测量。

## 7. Speculation Length

每轮 Draft Model 提前预测的 token 数量记作：

$$
K
$$

也称为：

- speculation length；
- draft length；
- speculative window；
- verification block size。

例如 $K=4$：

~~~text
Draft → token 1
      → token 2
      → token 3
      → token 4
      ↓
Target 一次验证
~~~

### 7.1. $K$ 太小

例如：

$$
K=1
$$

流程近似为：

~~~text
Draft → 1 token
Target → verify 1 token
~~~

Target Model 没有获得足够的序列维度并行度，收益可能很小。

### 7.2. $K$ 太大

例如：

$$
K=32
$$

若 acceptance rate 不高：

~~~text
✓  ✓  ✓  ×  ...
~~~

第一个拒绝之后的 29 个 token 都会失效，但 Draft Model 已经为它们支付了计算成本。

### 7.3. 接受长度的期望

设第 $i$ 个位置在此前缀已被接受的条件下，其接受概率为 $\alpha_i$。由于只能接受连续前缀：

$$
\Pr(A\ge i)=\prod_{j=1}^{i}\alpha_j
$$

于是：

$$
\mathbb{E}[A]
=\sum_{i=1}^{K}\Pr(A\ge i)
=\sum_{i=1}^{K}\prod_{j=1}^{i}\alpha_j
$$

若近似认为每个位置的条件接受概率都相同，为 $\alpha$，则：

$$
\mathbb{E}[A]
=\sum_{i=1}^{K}\alpha^i
=\frac{\alpha(1-\alpha^K)}{1-\alpha},\quad \alpha\ne1
$$

当 $\alpha=1$ 时：

$$
\mathbb{E}[A]=K
$$

每轮实际推进的 token 数通常是：

$$
\text{progress}=A+1
$$

其中额外的 $1$ 是首次拒绝位置的修正 token；如果 $A=K$，则这个 $1$ 是 Target Model 在整段候选之后产生的下一个 token。

### 7.4. 动态选择 $K$

实际系统通常不必固定 $K$。可以根据以下信号调整：

- 最近若干轮的 acceptance rate；
- 最近若干轮的 accepted prefix length；
- Target Model 当前队列长度；
- Draft/Target 的相对吞吐；
- GPU/NPU 利用率；
- KV cache 剩余容量；
- 延迟目标，例如 p95/p99。

简单策略：

~~~text
最近接受率高 + Target 空闲  → 增大 K
最近接受率低 + Target 繁忙  → 减小 K
KV 压力高                   → 限制 K
~~~

## 8. 为什么第一个错误之后的 token 都不能直接使用？

这是 speculative decoding 中最容易被忽略、但最重要的因果性问题。

### 8.1. Draft 后缀依赖被拒绝 token

设 Draft Model 按照以下分布生成：

$$
y_i\sim q(\cdot\mid c,y_{<i})
$$

假设第 $j$ 个 token 首次被 Target Model 拒绝。Draft 后面的 token 是在这个 draft 前缀上生成的：

$$
y_{j+1}
\sim q(\cdot\mid c,y_{<j},y_j)
$$

但是，Target Model 需要把第 $j$ 个位置修正为某个 token $z_j$，所以正确的下一个条件分布应为：

$$
p_T(\cdot\mid c,y_{<j},z_j)
$$

通常：

$$
y_j\ne z_j
$$

因此：

$$
q(\cdot\mid c,y_{<j},y_j)
\ne
p_T(\cdot\mid c,y_{<j},z_j)
$$

后缀 token 已经属于另一条因果路径。

### 8.2. 直观图示

~~~text
正确上下文 c
      │
      ▼
    y1 → y2 → yj  ──×──► yj+1 → yj+2
                    │
                    │ Target 修正为 zj
                    ▼
                  zj → 重新生成后续
~~~

后缀 token 并不是一定“不合理”，而是它在被拒绝 token 存在的条件下生成。修正为 $z_j$ 后，后续 token 的条件上下文已经改变。

### 8.3. Target 的 logits 也对应错误前缀

Target Model 对整个 draft block 的一次 forward 得到的后缀 logits，实际上对应：

$$
p_T(\cdot\mid c,y_1,\dots,y_j,\dots)
$$

而不是修正后的：

$$
p_T(\cdot\mid c,y_1,\dots,y_{j-1},z_j,\dots)
$$

要得到修正路径上的后续 logits，必须让 Target Model 重新处理修正后的 token。因此不能因为后缀 token 在原始 verification 输出中“看起来也通过”，就直接提交它们。

### 8.4. 标准处理方式

在首次拒绝位置 $j$：

1. 接受 $y_1,\dots,y_{j-1}$；
2. 由 Target Model 生成或采样修正 token $z_j$；
3. 丢弃 $y_j$ 以及整个后缀 $y_{j+1},\dots,y_K$；
4. 以 $c,y_{<j},z_j$ 作为新上下文继续生成。

这正是“接受前缀、修正分叉点、丢弃后缀”的原因。

## 9. 标准 Speculative Sampling 算法

下面描述保持 Target Model 输出分布的经典形式。记：

- $q_i(\cdot)=q(\cdot\mid c,y_{<i})$：Draft 分布；
- $p_i(\cdot)=p_T(\cdot\mid c,y_{<i})$：Target 分布；
- $y_i$：Draft 采样出的第 $i$ 个 token。

### 9.1. 接受概率

对第 $i$ 个 draft token，计算：

$$
a_i=\min\left(1,\frac{p_i(y_i)}{q_i(y_i)}\right)
$$

采样 $u\sim U(0,1)$：

$$
u\le a_i
\Rightarrow \text{接受 }y_i
$$

如果 $u>a_i$，则在第 $i$ 个位置首次拒绝。

### 9.2. 拒绝时的残差分布

为了保持 Target Model 的原始分布，首次拒绝位置可以从残差分布采样：

$$
r_i(x)=
\frac{[p_i(x)-q_i(x)]_+}
{\sum_v[p_i(v)-q_i(v)]_+}
$$

其中：

$$
[z]_+=\max(z,0)
$$

直觉上，Draft Model 已经覆盖的概率质量可以被接受；Target 比 Draft 多出来的概率质量由残差分布补回。

### 9.3. 概念性伪代码

~~~text
context = initial_context

while not finished:
    # 1. Draft Model 连续生成 K 个候选
    y[1:K], q[1:K] = draft_generate(context, K)

    # 2. Target Model 一次 forward，得到每个位置的分布
    p[1:K+1] = target_verify(context, y[1:K])

    accepted = 0

    for i in 1..K:
        if exact_accept(y[i], p[i], q[i]):
            output(y[i])
            accepted += 1
        else:
            # 在首次拒绝处生成修正 token
            z = sample_residual(p[i], q[i])
            output(z)

            # y[i] 以及 y[i+1:K] 全部丢弃
            context = context + accepted_tokens + z
            break
    else:
        # 整个 draft block 全部接受
        z = sample_from_target(p[K+1])
        output(z)
        context = context + y[1:K] + z
~~~

实际实现会额外处理 EOS、停止词、top-k/top-p、KV cache 回滚和 batch 中不同样本的接受长度。

### 9.4. Greedy decoding 的简化形式

如果目标是确定性的 greedy decoding，可以直接比较：

$$
y_i=\arg\max_x q_i(x)
$$

与：

$$
\hat y_i=\arg\max_x p_i(x)
$$

若两者相同则接受；首次不同则输出 Target Model 的 greedy 结果并丢弃后缀。

这种做法可以保持 greedy 序列，但它不等同于随机采样场景下的精确分布保持。

## 10. Lossless 与 Approximate Speculation

### 10.1. Lossless acceleration

在满足以下条件时，经典接受/拒绝采样可以保持 Target Model 的输出分布：

- Draft 与 Target 的 tokenization 和词表兼容；
- 使用正确的 Target 概率；
- 使用正确的接受概率；
- 首次拒绝后使用残差分布修正；
- 不直接提交拒绝位置之后的 draft 后缀。

这里的 **lossless** 指输出分布等价于直接使用 Target Model，并不表示：

- 速度一定更快；
- 能耗一定更低；
- 内存占用一定更小；
- 任何启发式实现都不会引入误差。

### 10.2. Approximate speculation

以下方案可能改变 Target Model 的输出分布，但有时更简单或更快：

- 只要 Draft 置信度高就直接输出；
- 用固定阈值跳过 Target verification；
- 只比较 top-1 或 top-k 是否一致；
- 用启发式规则接受一段后缀；
- 在不相同 tokenizer 之间做近似映射；
- 只在周期性位置同步 Target Model。

这类方法应明确标为 approximate、quality-aware 或 cascade，而不要笼统称为 lossless speculative decoding。

## 11. 性能模型

### 11.1. 基本符号

定义：

- $C_D$：Draft Model 生成一个 token 的平均成本；
- $C_T^{\text{decode}}$：Target Model 标准逐 token decode 成本；
- $C_T^{\text{verify}}(K)$：Target Model 验证 $K$ 个候选的成本；
- $C_{\text{sync}}$：KV、DMA、设备同步等成本；
- $C_{\text{sched}}$：调度、kernel launch、模型切换等成本；
- $A$：本轮接受的 draft 前缀长度。

不使用 speculation 时，生成 $N$ 个 token 的时间近似为：

$$
T_{\text{base}}\approx N\cdot C_T^{\text{decode}}
$$

使用 speculation 时，一轮的时间近似为：

$$
C_{\text{round}}
\approx
K C_D
+C_T^{\text{verify}}(K)
+C_{\text{sync}}
+C_{\text{sched}}
$$

一轮平均推进 $1+\mathbb{E}[A]$ 个 token，因此吞吐近似为：

$$
\text{Throughput}_{\text{spec}}
\approx
\frac{1+\mathbb{E}[A]}
{K C_D+C_T^{\text{verify}}(K)+C_{\text{sync}}+C_{\text{sched}}}
$$

相对于标准 decode 的理想 speedup 可写为：

$$
S\approx
\frac{(1+\mathbb{E}[A])C_T^{\text{decode}}}
{K C_D+C_T^{\text{verify}}(K)+C_{\text{sync}}+C_{\text{sched}}}
$$

获得加速的必要条件是：

$$
(1+\mathbb{E}[A])C_T^{\text{decode}}
>
K C_D+C_T^{\text{verify}}(K)+C_{\text{sync}}+C_{\text{sched}}
$$

### 11.2. 为什么 acceptance rate 不是唯一指标

两个系统可能有相同的 $\alpha$，但速度不同：

- 系统 A 的 Draft Model 更快；
- 系统 B 的 Target verification kernel 更高效；
- 系统 A 的 KV 搬运更少；
- 系统 B 的模型切换和唤醒开销更低；
- 系统 A 的 acceptance 更集中在长前缀；
- 系统 B 的接受 token 分布更分散。

因此评估时至少要同时报告：

~~~text
acceptance rate
accepted prefix length
tokens per speculative round
draft latency
target verification latency
end-to-end TPOT / ITL
~~~

### 11.3. 关键直觉

当 Target Model 主要受内存带宽限制时，验证多个位置可能让一次权重读取服务多个 token：

$$
\text{weight read per token}\downarrow
$$

当 Target Model 已经处于高 batch、高利用率状态时，额外的 speculative verification 可能不再划算，甚至造成拥塞。因此 speculative decoding 的收益依赖 workload，而不是只依赖模型大小。

## 12. KV Cache 与 Target State

### 12.1. Draft KV 与 Target KV 不可直接互换

Draft Model 和 Target Model 通常拥有不同的参数、层数、hidden size、attention head 配置，因此：

$$
KV_D\ne KV_T
$$

如果前面的 token 只由 Draft Model 处理，而 Target Model 从未执行过这些 token，那么 Target Model 的历史 KV 并不存在：

$$
KV_T(y_1),KV_T(y_2),\dots
$$

不能因为 Draft Model 已经有 KV，就直接当作 Target Model 的 KV 使用。

### 12.2. 标准 speculative decoding 如何处理

一种概念上的实现方式是：

~~~text
Target KV for common context
            │
            ▼
   temporary Target KV for draft block
            │
            ▼
        verify y1...yK
            │
      ┌─────┴─────┐
      ▼           ▼
  接受前缀      首次拒绝
      │           │
  提交公共部分   丢弃拒绝位置后的临时 KV
~~~

首次拒绝后，需要让 Target Model 继续处理修正 token，形成正确的新状态；不能保留错误路径的后缀状态。

### 12.3. KV Cache 内存估算

对一个常见 decoder-only Transformer，KV cache 的数量级可以写成：

$$
M_{KV}
\approx
2\cdot L\cdot H_{KV}\cdot d_h\cdot B\cdot T
$$

其中：

- $2$：分别对应 K 和 V；
- $L$：层数；
- $H_{KV}$：KV heads 数量；
- $d_h$：每个 head 的维度；
- $B$：每个元素的字节数；
- $T$：缓存 token 数量。

Speculative window 会额外产生近似与 $K$ 成正比的临时 KV：

$$
\Delta M_{KV}\propto K
$$

因此扩大 $K$ 不只是增加计算，也会增加临时缓存、回滚和内存带宽压力。

## 13. Token-Level Cascade 与多个小模型

前面提到的另一类方案是：简单 token 交给小模型，困难 token 才交给大模型。

### 13.1. 单个小模型的置信度路由

令小模型分布为 $p_S$，定义置信度：

$$
C_S(t)=\max_x p_S(x\mid x_{<t})
$$

一种直接规则是：

$$
x_t=
\begin{cases}
\arg\max p_S(x_t), & C_S(t)\ge\tau\\
\arg\max p_T(x_t), & C_S(t)<\tau
\end{cases}
$$

其中 $\tau$ 是阈值。

也可以使用：

- entropy：
  $$
  H(p_S)=-\sum_xp_S(x)\log p_S(x)
  $$
- top-1 与 top-2 的概率 margin；
- 多个小模型的一致性；
- draft 与 target 的历史 agreement；
- token 类型或语法状态。

但是，小模型的高置信度不等价于小模型与大模型一定一致。置信度需要校准，并应在目标 workload 上验证。

### 13.2. 成本模型

设共有 $N$ 个 token，$\rho(\tau)$ 是低置信度、需要 defer 到 Target Model 的比例，则粗略成本为：

$$
C_{\text{cascade}}
\approx
N C_S+\rho(\tau)N C_T+C_{\text{sync}}
$$

阈值越高，通常越多 token 会进入大模型，但质量风险可能下降；阈值越低，成本下降，但小模型直接输出的风险上升。

### 13.3. 多个小模型

多个小模型可以组成级联：

~~~text
Context
   │
   ▼
Small Model 1
   │ confidence
   ├──────── 高 ───────► 直接输出
   │
   ▼ 低
Small Model 2
   │ confidence
   ├──────── 高 ───────► 输出
   │
   ▼ 低
Target Model
~~~

也可以让多个小模型并行产生候选，再由 Target Model 统一验证：

~~~text
             ┌─ Draft 1 ─┐
Context ─────┼─ Draft 2 ─┼──► Candidate block / tree
             └─ Draft 3 ┘              │
                                        ▼
                                Target verification
~~~

如果所有候选最终都经过正确的 Target verification，仍有机会保持精确性；如果高置信度 token 直接绕过 Target，则属于 approximate cascade。

### 13.4. Token-Level Cascade 的 KV 难点

假设：

~~~text
token:  1  2  3  4  5  6  7  8
small:  ✓  ✓  ✓  ?  ✓  ✓  ✓  ?
large:              ↑              ↑
~~~

表面上似乎只需要让 Target Model 计算 token 4 和 8。但如果 token 1–3 只经过小模型，Target Model 缺少：

$$
KV_T(1),KV_T(2),KV_T(3)
$$

Target Model 不能只凭一个“困难 token”独立完成标准 decode。实际方案通常需要：

1. **周期性 replay**：Target Model 重放一段小模型已生成的 token，补齐自己的 KV；
2. **块级 verification**：把多个 token 累积成 block，再一次验证；
3. **异步 lagging Target**：Target Model 在后台追赶小模型状态；
4. **共享或投影 KV**：减少状态转换，但会引入兼容性、精度和硬件成本；
5. **目标模型常驻**：牺牲部分能耗，降低重新唤醒和状态重建成本。

这也是为什么标准 speculative decoding 通常选择“先生成一段，再统一验证”：它让 Target Model 的状态推进和验证过程天然对齐。

## 14. Accelerator / System 视角

### 14.1. 从 GEMV 形态转向 GEMM 形态

标准单 token decode 的线性层近似处理一个很小的序列维度：

~~~text
1 token × hidden dimension
        ↓
小矩阵运算 / GEMV-like
~~~

Speculative verification 处理 $K$ 个位置：

~~~text
K tokens × hidden dimension
        ↓
更大的矩阵运算 / GEMM-like
~~~

这样更容易利用：

- Tensor Core / Matrix Engine；
- NPU systolic array；
- SRAM/缓存中的权重复用；
- 向量化和流水线并行；
- 高带宽内存的突发访问。

相关背景可参见 GEMM 和 GEMV 与 LLM inference。

### 14.2. 异构设备协同

一个典型的 CPU + 小 NPU + 大 NPU 结构可以抽象为：

~~~text
                 ┌────────────────┐
                 │ CPU / small NPU│
                 │ Draft Model    │
                 └───────┬────────┘
                         │ token block
                         │ DMA / shared buffer
                         ▼
                 ┌────────────────┐
                 │ large NPU/GPU  │
                 │ Target Verify  │
                 └───────┬────────┘
                         │ accepted prefix
                         ▼
                    output stream
~~~

系统设计需要关注：

- Draft 与 Target 的设备放置；
- block 传输与格式转换；
- Target Engine 的唤醒延迟；
- 共享内存、DMA 和同步栅栏；
- 临时 KV 的存储位置；
- 失败后 KV 回滚；
- 多请求之间的 batch 合并。

### 14.3. Target Model 的 burst execution

大模型不一定要每次收到一个 token 就启动完整执行流程。更适合的硬件接口是：

~~~text
submit(context_id, draft_tokens[1:K])
        ↓
one verification job
        ↓
accepted_length, correction_token, cache_commit_info
~~~

硬件或 runtime 可以围绕 accepted length 做：

- prefix commit；
- suffix rollback；
- cache pointer 更新；
- 下一轮任务调度；
- 不同请求的动态 batch 重组。

### 14.4. KV Cache 流量

扩大 $K$ 会增加：

- Target 临时 KV 写入；
- 验证阶段 KV 读取；
- rejection 后的 rollback；
- cache page 管理；
- 长上下文下的 HBM traffic。

因此在硬件优化中不能只看 MAC 数量，还要测量：

~~~text
HBM bytes / generated token
KV read / write volume
cache hit rate
rollback bytes
device-to-device transfer
~~~

### 14.5. Batch size 的影响

当 batch 很小且 Target Model 利用率低时，speculative decoding 往往更有价值。

当 batch 已经很大时，Target Model 原本就可以通过 batch 获得较好的并行度，此时 speculative verification 的边际收益可能下降。还要额外处理：

- 不同请求的首次拒绝位置不同；
- 不同请求需要不同长度的 cache commit；
- batch 内请求可能在不同时间结束；
- continuous batching 的调度复杂度增加。

### 14.6. 能耗视角

Speculative decoding 的能耗收益不是自动成立的：

$$
E_{\text{spec}}
=
E_D+E_T^{\text{verify}}+E_{\text{memory}}+E_{\text{sync}}
$$

如果 Draft Model 始终运行、Target Model 频繁唤醒，额外的设备切换和数据搬运可能抵消减少的 Target decode 能耗。评估时应报告：

$$
\text{Joules / generated token}
$$

而不是只报告 Target Model 调用次数。

## 15. 动态调度与策略设计

### 15.1. 基于历史 acceptance 的自适应窗口

维护最近窗口的接受率估计：

$$
\hat\alpha_t
=
\lambda\hat\alpha_{t-1}
+(1-\lambda)\alpha_t
$$

然后按照规则调整 $K$：

~~~text
if α̂ 高且 Target 利用率低:
    K = min(K + 1, Kmax)

if α̂ 低或 Target 队列变长:
    K = max(K - 1, Kmin)
~~~

### 15.2. 把系统状态加入路由

仅根据模型置信度选择 $K$ 不够，还可以把系统状态纳入决策：

$$
K_t=f(\hat\alpha_t,
      \text{Target load},
      \text{KV pressure},
      \text{latency budget})
$$

例如：

- Target 空闲：使用较大的 $K$，提高单轮推进量；
- Target 拥塞：减小 $K$，避免大 verification block 占用队列；
- KV 接近上限：减少临时窗口；
- p99 延迟敏感：限制最坏情况下的 rollback 和同步成本。

### 15.3. Draft 自身的早停

如果 Draft Model 在第 $i$ 个位置已经非常不确定，可以提前结束本轮 draft，而不是机械地产生固定 $K$ 个 token：

~~~text
Draft confidence:
高  高  高  低
                ↑
             提前停止
~~~

这样可以减少低质量后缀和无效 draft 计算，但会降低 Target verification 的 block 利用率，需要结合硬件粒度选择阈值。

## 16. 一个完整的拒绝示例

假设当前上下文为：

~~~text
c = "The cat"
~~~

Draft Model 生成：

~~~text
y1 = " is"
y2 = " sitting"
y3 = " on"
y4 = " the"
~~~

Target Model 一次验证，结果为：

~~~text
y1: ✓
y2: ✓
y3: ×
y4: 不可直接使用
~~~

如果 Target 在第 3 个位置修正为：

~~~text
z3 = " quietly"
~~~

则有效路径是：

~~~text
The cat → is → sitting → quietly
~~~

而 y4 = " the" 是在：

~~~text
The cat → is → sitting → on
~~~

这个不同的前缀下生成的，所以必须丢弃。下一轮应从：

~~~text
The cat is sitting quietly
~~~

继续由 Draft Model 生成新的候选。

## 17. 实验与评估指标

| 指标 | 含义 | 说明 |
|---|---|---|
| Acceptance rate | draft token 被接受的比例 | 需要按 workload 和 $K$ 分组 |
| Accepted prefix length | 每轮连续接受的 token 数 | 比单独的 token-level acceptance 更直接 |
| Tokens per round | 每轮实际推进的 token 数 | 通常接近 $1+\mathbb{E}[A]$ |
| Draft latency | 产生 $K$ 个候选的时间 | 包含 Draft kernel 和同步 |
| Target verify latency | 验证 block 的时间 | 与 $K$、上下文长度、batch 有关 |
| Target call count | Target 调用次数 | 不能单独代表端到端收益 |
| TPOT / ITL | 每个输出 token 的端到端延迟 | 重点指标 |
| Throughput | tokens/s | 注意 batch 和请求数 |
| TTFT | 首 token 延迟 | 与 prefill、首轮 draft 有关 |
| KV memory | Target/Draft/临时 KV 占用 | 长上下文尤其重要 |
| HBM traffic | 每 token 的内存流量 | 反映 memory-bound 程度 |
| Energy/token | 每个输出 token 能耗 | 评估 edge/端侧部署必需 |
| Quality / KL | 与 Target 分布或输出的差异 | 区分 lossless 与 approximate |
| p95 / p99 latency | 尾延迟 | 动态 batch 和 rollback 会影响 |

一个合理的对比至少应包含：

~~~text
baseline Target decode
Target + fixed K speculative decoding
Target + adaptive K speculative decoding
不同 Draft Model
不同 batch size / context length
~~~

## 18. 常见误区

### 18.1. 误区 1：接受率高就一定加速

不一定。若 Draft Model 太慢、Target verification 太重或同步成本太高，端到端速度仍可能下降。

### 18.2. 误区 2：Target 一次 forward 验证 K 个 token，所以计算量变成原来的 1/K

不准确。理论工作量仍然与多个位置有关，收益主要来自并行、权重复用和减少串行调用。

### 18.3. 误区 3：第一次拒绝之后，只要后缀 token 的概率也高就可以接受

不可以。后缀是在被拒绝 token 的条件下生成的；修正分叉点后，因果上下文不同。

### 18.4. 误区 4：小模型置信度高就等于大模型一定同意

不等于。置信度是小模型自身分布的属性，需要校准和实测 agreement。

### 18.5. 误区 5：只计算困难 token 时，大模型可以直接跳到该位置

通常不行。Target Model 需要自己的历史 KV；被跳过的简单 token 仍可能需要 replay、同步或 block verification。

### 18.6. 误区 6：只看 Target Model 调用次数

调用次数减少不代表端到端延迟或能耗一定下降。必须把 Draft、KV、DMA、同步、设备唤醒和调度都计入。

### 18.7. 固定 K 对所有场景都最优

不同上下文、采样温度、batch 和硬件状态下，最优 $K$ 可能不同。应考虑动态窗口。

## 19. 与相关方法的关系

~~~text
                是否由 Target 验证？
                 │
          ┌──────┴──────┐
          │             │
         是             否 / 部分
          │             │
  Speculative decoding  Token-level cascade
  Draft-and-verify       Confidence routing
  Multi-drafter verify   Approximate early exit
~~~

| 方法 | Draft 是否连续生成 | Target 是否验证 | 是否可保持 Target 分布 |
|---|---:|---:|---:|
| Standard speculative decoding | 是 | 验证整个候选 block | 可以，使用精确接受/拒绝规则 |
| Greedy speculative decoding | 是 | 比较 Target greedy 结果 | 可以保持确定性 greedy 序列 |
| Token-level cascade | 可选 | 只在 defer 时调用 | 通常不天然保持 |
| Multi-drafter verification | 是，可并行/树状 | Target 统一验证 | 取决于验证与采样规则 |
| Confidence-only routing | 不一定 | 可能跳过 | 通常是 approximate |

## 20. 总结

Speculative decoding 的核心链路是：

~~~text
便宜模型猜测一段
        ↓
昂贵模型一次验证
        ↓
接受连续前缀
        ↓
首次拒绝处修正
        ↓
丢弃错误路径后缀
        ↓
从正确上下文继续
~~~

最关键的技术要点：

1. 自回归 decode 的瓶颈是 Target Model 的串行单 token 执行；
2. Causal mask 允许 Target Model 在一次 forward 中验证多个位置；
3. Draft Model 的成本、质量和 acceptance rate 共同决定收益；
4. 首次拒绝之后的 token 依赖错误的前缀，不能直接接受；
5. 精确 speculative sampling 需要正确的接受概率和残差修正；
6. KV cache 一致性是 token-level cascade 和硬件实现的核心难点；
7. 真正的端到端收益取决于并行度、HBM 流量、同步、调度、batch 和能耗；
8. 在 accelerator 设计中，最值得优化的是 Draft/Target 协同、block verification、临时 KV 管理和动态窗口调度。

从 LLM inference / accelerator 的角度看，最有价值的抽象不是简单的：

$$
p<\tau\Rightarrow\text{调用大模型}
$$

而是：

$$
\boxed{
\text{confidence routing}
+\text{KV-cache synchronization}
+\text{heterogeneous scheduling}
+\text{block verification}
}
$$

这四部分共同决定了 speculative decoding 能否从算法层面的“少调用几次模型”，转化为真实硬件上的低延迟、低能耗 LLM 推理系统。

## 21. 相关笔记与进一步阅读

- LLM inference
- GEMM 和 GEMV
- PagedAttention
- [NPU 的软硬件设计层次](/posts/20_Reserch/LLM%E6%8E%A8%E7%90%86/NPU%E7%9A%84%E8%BD%AF%E7%A1%AC%E4%BB%B6%E8%AE%BE%E8%AE%A1%E5%B1%82%E6%AC%A1/)
- Leviathan, Kalman, Matias：*Fast Inference from Transformers via Speculative Decoding*。
- Chen et al.：*Accelerating Large Language Model Decoding with Speculative Sampling*。
- 关键词：draft-and-verify、speculative sampling、multi-drafter、adaptive speculation、token-level cascade、KV-cache synchronization。
