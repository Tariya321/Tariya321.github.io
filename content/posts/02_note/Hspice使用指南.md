---
title: "Hspice使用指南"
tags:
  - Hspice
publish: yes
date: 2023-09-30
---
2025-09-30_01:03今天在做 virtuoso 的仿真，发现曲线一直有问题，于是回来看看电源的参数设置，常看常新~

{{< figure src="/attachment/Pasted%20image%2020230924110527.png" alt="Pasted image 20230924110527" width="500" >}}

- 电路元器件
```Hspice
MXX  ND NG NS NB MNAME ; MXX为, 漏/栅/源/衬底, 最后是mos管名称
```

## 1. 电源
---
* 分段线性源
```
VXXX N+ N- PWL (T1 V1 <T2 V2 T3 V3 ……>)    ;其中每对值（T1，V1）确定了时间t＝T1是分段线性源的值V1
Vpwl 3 0 PWL（0 1 10n 1.5）    ;表示节点3和0之间的分段线性源
```

- 直流源
```
VXXX N1 N2 DC value  ; 在节点N1和N2之间施加大小为value的直流电压
IXXX N1 N2 DC value  ; 在节点N1和N2之间施加大小为value的直流电流
```

脉冲源 Vpluse 的使用
{{< figure src="/attachment/Hspice%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97.png" alt="Hspice使用指南" width="400" >}}
描述参数有
- V1 初始电压
- V2 脉冲电压
- td 延迟时间（和周期无关，纯粹是从 V1 到开始周期 V2的时间）
- PW 脉宽
- period 周期长度

正弦波源Vsin 使用
```
VXXXX N+ N- SIN(V0 VA FREQ TD THETA PHASE)
V0:偏置，VA:幅度，FREQ: 频率 ，TD :延迟，THETA: 阻尼因子，PHASE:相位
```
幅度：在 dc 之上的最大幅度


## 2. 组成结构
- 子电路

```
; 子电路的定义
.SUBCKT OPAMP 1 2 3   ; 名为OPAMP的子电路，节点为1、2、3
<description...>      ; 表达电路连接关系
.ENDS

; 调用子电路
Xop 3 5 7 OPAMP   ；调用子电路OPAMP，名称为Xop，外部节点为3、5、7依次对应123
```
注意调用名只能以X开头，作为伪元件名

- 注释语句
星号 * 为注释符号

## 3. 电路的分析类型
---
分析类型描述语句由定义电路分析类型的描述语句和一些控制语句组成，如直流分析
（.OP），瞬态分析（.TRAN）等分析语句，以及初始状态设置（.IC）,选择项设置(.OPTIONS)等控制语句。它的位置可在标题语句和结束语句之间的任何地方。
### 3.1. 仿真时长
---
```
.tran 1n 200n ; 设置仿真总时长为200ns, 步长为1ns
```

### 3.2. 文本打印
---
```
.PRINT TYPE OutputValue_1 OutputValue_2 OutputValue_3
* type表示打印数据的类型（瞬态、直流……）
* OutputValue表示打印节点x，节点y
```

## 4. 其他
---

### 4.1. 调整曲线
---
手动调整
按下ctrl+shift+c

固定模式调整
tools-preference-analysis
![Pasted image 20091006101453](/attachment/Pasted%20image%2020091006101453.png)

## 5. 错误
---
edit SL不亮时，表明生成错误，Hspice网表文件有问题

