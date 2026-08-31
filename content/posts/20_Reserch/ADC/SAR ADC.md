---
title: "SAR ADC"
tags:
  - ADC
date: 2023-11-23
publish: yes
---

## 1. info
---
2023-11-23今天开了这个专题，打算补一下相关的知识

所有参考的webpage都在这里
[SAR ADC设计 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/585492142)

清华李福乐老师的集成电路实践课程PPT，有一章专门讲SAR ARC，非常详细

eetop上李福乐老师的基于tsmc 0.5mm的8bit同步SAR ADC模型
[集成电路设计实践课件+实验教程-清华大学-李福乐-2013年春 - Analog/RF IC 资料共享 - EETOP 创芯网论坛 (原名：电子顶级开发网) -](https://bbs.eetop.cn/thread-396169-1-1.html)

SAR ADC大师
- 西电的朱樟明低功耗CMOS SAR ADC

2024-06-01向段俊同学请教SAR ADC基础：
1. 二进制码的位数越多，意味着在一个电压幅值下，可细分的电压值越多，因此位数也可称为精度。
2. 理论上位数越多，精度越高，ADC转换的数字结果越接近模拟输入。然而由于电路复杂度的提升，精度可能下降（热噪声...）

[10. 逐次逼近型ADC原理_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1LN4y1g7yt/?spm_id_from=333.788.recommend_more_video.-1&vd_source=aebba21bc4a0f36d05cfff05d5e944b2)学习下SAR ADC的基本原理是什么
例子天平称重可以帮助理解其原理


## 2. SAR ADC 基本工作原理

Circuit schemetic
{{< figure src="/attachment/SAR%20ADC%E5%9F%BA%E6%9C%AC%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.png" alt="SAR ADC基本工作原理" width="300" >}}
>类比二分法。

- S/H 为 sample/Hold 的缩写，表示对模拟输入进行采样并保持
- Vsh 和 Vdac 为比较器的两个输入端，Comp 为 compare 结果信号：高或者低
- 在对比周期内，一个 CLK 有效电平将根据 COMP 信号结果依次输出 Bn 为有效或者无效；
- DAC 电路根据 SAR 处理电路输出的二进制数 $B_{n-1}B_{n-2}...B_2B_1$ 输出转换电平 `V_DAC`
- `V_DAC` 信号再与采样信号对比，如此循环

例如输入的采样信号范围是 0~15 mV，那么可以使用位宽为 4 的 SAR 控制器（0000 对应 0 mV, 1111 对应 15 mV）。
- 假设输入信号为 5 mV，若最高位为 1，则输出值 $V_{DAC}$ 为 8 mV
- 比较器输出使得最高位为 0，然后次高位设为 1，则输出值为 4 mV
- 比较器输出使得次高位为 1，接下来次次高位设为 1，....
如此循环下去，直到最后一位，可以逐渐逼近结果，这也是**逐次逼近** ADC 名称的由来。

## 3. SAR ADC 电路
---

### 3.1. 采样时钟生成电路
---

### 3.2. 比较器
---

### 3.3. SAR逻辑电路
---
SAR（successive approximation register）ADC，逐次逼近型模数转换模块

