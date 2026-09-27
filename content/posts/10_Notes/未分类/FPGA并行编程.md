---
title: "FPGA并行编程"
date: 2025-06-04
author: xilinx
attachment: https://xupsh.gitbook.io/pp4fpgas-cn/zheng-wen/01-introduction
publish: yes
tags:
  - FPGA
  - HLS
---

{{< figure src="/attachment/FPGA%E5%B9%B6%E8%A1%8C%E7%BC%96%E7%A8%8B.png" alt="FPGA并行编程" width="600" >}}
## 1. FIR 滤波器
用户可以通过在代码中添加`#pagma HLS pipeline`来指导Vivado HLS工具产生函数流水结构。

**递归**（recurrence），这里是指某个部件的计算需要这个部件之前一轮计算的结果
尽量避免使用含有很多递归的算法


**重建代码**。一般来说现成的算法原代码产出的结构比普通的CPU程序还低效，即使使用流水，展开等方法也没起到太大的作用。所以最好的方法还是自己写出一个等效但适合高层次综合的算法。

​for循环内部的if/else语句效率很低

将嵌套的for循环分解为两个独立循环,从而在每个循环上分别进行优化

> [!NOTE] compile time
如果你的设计在15分钟内不能综合完成，你应该仔细考虑优化效果。当然大型设计可能会花费Vivado HLS大量时间进行综合。但是作为一个初始的用户，你的设计应该相对快速地综合完成。如果花了很长时间，那很可能意味着你以错误的方式使用了一些指令，明显扩展了综合代码。

## 2. optimize synthesis result

Vitis HLS pragmas and directives let you configure the synthesis results for your code.

- HLS Pragmas are added to the source code, enable the optimization or change in the original source code
- directive can be specified as Tcl commands, allowing you to customize the synthesis results for the same code base across different solutions.

{{< figure src="/attachment/FPGA%E5%B9%B6%E8%A1%8C%E7%BC%96%E7%A8%8B-1.png" alt="FPGA并行编程-1" width="500" >}}

