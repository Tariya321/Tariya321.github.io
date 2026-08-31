---
title: "Digitial IC Design-IIT"
tags:
  - 数字电路设计
  - IIT
author: IIT(印度理工)
link: https://www.bilibili.com/video/BV1Tf4y127sE
date: 2023-10-15
publish: yes
---
## 1. L2
---
对于一个function，给定performance、area minimum作为约束，这是设计中最基本的要求

>不明白这是他们什么层次的课程，但是我们却不讲这样的内容……

接下来，通过使用某一电路实现逻辑功能作为示例，说明了逻辑最简的设计并不一定是（某一指标）最优的设计。

overall cost = design cost + fabricate cost + ……
cost是厂商最关心的指标

Synthesis
>Design consists of severl synthesis steps.

behavior representation -> structural representation
![Pasted image 20231015210215](/attachment/Pasted%20image%2020231015210215.png)

下一步转为门级结构

最后进行其物理实现 (layout -> tape-out)

step-1 logic synthesis
(1)minimization: get the minimal area, however, when the input increased, the complexity of simpifying boomed and people make faults!
So, we need a CAD tool!

(2)technology mapping: map the minimaized expression onto available gates
>1. FPGA don't have gates and their mapping is based on LUT
>2. for CMOS technology, the elementary gates are NAND, NOR, NOT etc

讲到了处理delay constraint，方法是找到critical path，手工分析对于简单电路很有效
然而，当电路规模达到百个千个之后，手工分析几乎不可能准确
此时就需要CAD tool：static timing analysis（静态时序分析）

digital design 就是非常需要工具的事业

step-2 logic verification
（1）vertical: 具象化、综合、网表。通常是做一个logic simulation，此时仍然需要一个CAD tool来做，穷尽所有输入可能
（2）horizon: 检查门电路的扇入扇出以及其它的特定rules

step-3 circuit design
choose a circuit style
针对CMOS style：full complement、pseudo NMOS，Domino
注意，即使到了此时，对于功耗、面积等指标我们也只能做estimation
接下来要对门的尺寸做sizing从而进一步尝试满足定义的specification（spec.），在只有少数门时，手工调整是完全可行的，但随着门数目的指数增加，这愈发于不可能
此时仍然需要CAD tool来验证performance和functionality

step-4 physical or layout design
DRC
ERC
LVS

整个的design flow
{{< figure src="/attachment/Design%20Flow.png" alt="Design Flow" width="500" >}}

## 2. L3 sequential logic design-1
---
先通过一个反馈电路清晰了问题，后面引出状态机来解决问题
Mealy型和Moore型，参加[Verilog](/posts/02_note/Verilog/#mealy%E5%92%8Cmoore%E5%9E%8B%E7%8A%B6%E6%80%81%E6%9C%BA)
state transition graph状态转移图
总体介绍了时序电路的synthesis过程

state minimization（状态精简）：相同输入输出和状态切换
常采用implicant chart进行检查

state encoding（状态码编码方式）：
选择多少位？至少是$\rm{ log_2(\# of \quad states)}$。但采用最少位是最好的嘛？
*remember that the goal is not to minimize the bits, but the overall circuit complexity*
最少位，但是逻辑复杂度增佳；onehot编码，位数较多，但得到了简单的逻辑
[Verilog](/posts/02_note/Verilog/#%E7%BC%96%E7%A0%81%E9%80%89%E6%8B%A9)

choose a flip-flop type to implement states：
选择D还是JK要根据实际情况

## 3. L4 sequential logic design-2
---


## 4. L5 register transfer level design
---


{{< figure src="/attachment/Pasted%20image%2020231219211308.png" alt="Pasted image 20231219211308" width="500" >}}

两个概念
**latency**：This is the time taken for a signal or a data packet to travel from the source to the destination. It's often measured in milliseconds (ms). In a pipeline, latency is the time taken from the input of the first stage to the output of the last stage. It represents delay.

**throughput**：This refers to the rate at which data is processed or transferred. It's usually measured in terms of *data units per time* (like Mbps - megabits per second). In a pipeline, throughput is the number of tasks or data units processed per unit of time. High throughput means the system can handle more data or tasks in less time.