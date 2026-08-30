---
title: "为什么存算电路中XNOR很常用"
date: 2025-04-01_11:13
tags:
  - CIM
  - XNOR
publish: yes
---
## 1. 背景
这几天准备做ising machine的电路，看了文章里用的存算结构，都提到了XNOR（同或）的逻辑。之前看存算的时候没有重点关注过，为什么XNOR很常用? 

门电路符号
{{< figure src="/attachment/%E4%B8%BA%E4%BB%80%E4%B9%88%E5%AD%98%E7%AE%97%E7%94%B5%E8%B7%AF%E4%B8%ADXNOR%E5%BE%88%E5%B8%B8%E7%94%A8-1.png" alt="为什么存算电路中XNOR很常用-1" width="150" >}}

## 2. 基础知识

开始之前，先让我们复习一下数字电路基础：XNOR的门逻辑
{{< figure src="/attachment/%E4%B8%BA%E4%BB%80%E4%B9%88%E5%AD%98%E7%AE%97%E7%94%B5%E8%B7%AFXNOR%E5%BE%88%E5%B8%B8%E7%94%A8.png" alt="为什么存算电路XNOR很常用" width="300" >}}
好的，可以看出：**两输入相同为1，不同为0**
***Y = A ⊙ B***
This expression can also be written as follows,
***Y = AB + A'B'***

## 3. why
在CIM的应用层次上：两个存储单元的状态异或后进行求和，通过电流对比来确定 XNOR 结果，从而高效完成二值矩阵乘法（Binarized Matrix Multiplication）

以22年jssc中ising机[^1]为例，这个电路的结构是上面一个经典的6T SRAM，中间是两个transmission gate，下面是一个full-adder。
{{< figure src="/attachment/%E4%B8%BA%E4%BB%80%E4%B9%88%E5%AD%98%E7%AE%97%E7%94%B5%E8%B7%AFXNOR%E5%BE%88%E5%B8%B8%E7%94%A8-1.png" alt="为什么存算电路XNOR很常用-1" width="300" >}}
文中指出：“The coefficient value stored in an SRAM bitcell is directly read out from the bitcell internal nodes. **Two transmission gates work as an XNOR-based bitwise signed multiplier.** The multiplication result between an input spin value and the coefficient stored in the SRAM bitcell is sent to the input B of the full adder, located at the bottom of the unit cell.”

核心是要实现两者的乘法电路。可以看到传输门的control信号是neighbor spin state，input信号是存储的coefficient的某个bit。已知传输门的传输函数是out=AB，那么对于输出为相同节点的两个transmission gate而言，实现的是out=AB+A'B'
>关于传输门的基本原理，please refer to 数字集成电路

对于两个single bit相乘，只要它俩都是1的时候输出1，其他时候输出0就好了。听起来似乎直接给个AND门就好了，然而一个AND gate需要4+2=6个管子实现，并且其功耗要高于传输门（模拟switch）。再看回传输门，可以看到，这一对（couple）正好可以满足逻辑。

总结：**使用传输门有减小面积和功耗的优点**

类似的，还可以看24年JSSC的ising机文章[^2]的存算电路
{{< figure src="/attachment/%E4%B8%BA%E4%BB%80%E4%B9%88%E5%AD%98%E7%AE%97%E7%94%B5%E8%B7%AF%E4%B8%ADXNOR%E5%BE%88%E5%B8%B8%E7%94%A8.png" alt="为什么存算电路中XNOR很常用" width="400" >}}
也是一样的，不过它牺牲精度来减小面积了，用的传输管显然会使voltage swing下降

总结：XNOR 门很好的实现了乘法


[^1]: Y. Su, H. Kim, and B. Kim, “CIM-spin: a scalable CMOS annealing processor with digital In-memory spin operators and register spins for combinatorial optimization problems,” _JSSC_, vol. 57, no. 7, pp. 2263–2273, Jul. 2022, doi: [10.1109/JSSC.2021.3139901](https://doi.org/10.1109/JSSC.2021.3139901).

[^2]: Y. Zhou, G. Su, J. Zhou, L. Liao, and Z. Chen, “A compute-in-memory annealing processor with interaction coefficient reuse and sparse energy computation for solving combinatorial optimization problems,” _JSSC_, vol. 59, no. 9, pp. 3094–3105, Sep. 2024, doi: [10.1109/JSSC.2024.3376410](https://doi.org/10.1109/JSSC.2024.3376410).
