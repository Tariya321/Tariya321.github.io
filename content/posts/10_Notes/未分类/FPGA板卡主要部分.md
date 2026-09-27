---
title: "FPGA板卡主要部分"
date: 2026-06-14
tags:
  - FPGA
publish: true
---
核心处理芯片
- 可编程逻辑芯片（PL）
- 硬核处理器（PS）

存储芯片
- Flash 闪存(非易失 )，通常作为比特流存储位置，允许重新上电时再组装电路
- DDR 内存

时钟和电源管理
- 时钟源，晶振
- 电源管理芯片 PMIC


调试与通信接口
- JTAG通信
- UART串口通信
- SPI通信协议
- [AXI通信协议](/posts/10_Notes/%E6%8A%80%E6%9C%AF/%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE/AXI%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE/)

