---
title: "Formality"
tags:
  - verification
  - synopsys
  - EDA
date: 2025-06-09_23:19
publish: yes
---
形式验证


 一致性检查的作用是将EDA工具转化后的网表文件和之前的网表文件进行功能性对比，检查两者是否一致
 
## 1. 了解
---
### 1.1. webpages
---
[形式验证（Formality）- CSDN](https://blog.csdn.net/qq_38315280/article/details/131790997)

[在DC中使用tcl脚本综合和Formality一致性检查 - CSDN](https://blog.csdn.net/weixin_38197667/article/details/90738720)


**两种使用方式**
gui图形化界面：输入`fm`或者`formality`
shell方式：`fm_shell`启动，`source xxx.tcl`

若是在gui界面走了一遍，可以在formality图形界面的命令栏中输入：
```
history > calendar.fms
```
生成脚本文件`calendar.fms`，方便以后直接运行脚本文件执行一致性检查。
之后`source`这个脚本即可
### 1.2. 什么时候需要formality？
---
只要设计发生改变(对代码进行改动)，例如下面的场景：
- 综合前后——RTL比对DC综合网表
- DFT前后——DC综合网表比对DFT网表
- 物理实现前后(PR) ——DFT网表比对PR网表

### 1.3. 形式验证需要什么东西？
---
**参考设计(Reference Design)**        
一般情况下指RTL
**待比较设计(Implementation Design)** 
指综合后的网表
**容器（Containers）**        
放入“r” -参考设计
放入“i” -待比较设计
formality相关约束


## 2. 使用
---
>先读入文件，然后匹配、验证
### 2.1. 形式验证的流程
---
**Step0-引导(Guidance)**：添加综合产生的.svf文件
(在DFT前后与PR前后，不需要该步)

svf文件记录了综合所有的信息，包括：
Object name change
Constant register optimizations
Duplicate and merged refisters
Multiplier and divider architecture types
Datapath transformations
FSM re-encoding(Must be abled in Formality to be used)
Retimming
Register phase inversion

**Step2a-读源RTL**(Read Referenece Design and Libs),并设置顶层
**Step2b-读综合网表**(Read Implementation Design and Libs),并设置顶层

**Step3-设置约束**(Set up)，例如一些路径只走1端，就设置不走0,特别是DFT后，如dft_mode之类的引脚应当对于i文件和r文件都置0

**Step4-匹配**(Match)

**Step5-比对**(Verify)

### 2.2. fms脚本
---
下面是一个formality的脚本文件`calendar.fms`示例
```fms
set_svf -append { ./DSP.svf }

read_verilog -container r -libname WORK -05 { ./RTL/tb.v ./RTL/src/Top/models_pack.v ./RTL/src/Tile/DSP.v}

read_db { ./lib2db/tt0p9v25c.db }

set_top r:/WORK/DSP

read_verilog -container i -libname WORK -05 { ./dc_output/D_netlist.v }

read_db { ./lib2db/tt0p9v25c.db }

set_top i:/WORK/DSP

match
verify
```

### 2.3. 指令
---
`analyze_points -xxx`
-xxx可以是unverified
formality将会给出提示
>甚至告诉你在哪一步中加入哪几行命令

### 2.4. 再探formality 
formality UG
{{< figure src="/attachment/Design%20verification%20process%20flow.png" alt="Design verification process flow" width="500" >}}

**loading guidance**
Guidance is the process by which an implementation tool, such as Design Compiler, provides setup information for formal verification. This is supplied in the form of an automated setup file (.svf). 

**loading design**
To run Formality, you must read in both a reference and an implementation design and any related technology libraries. 
{{< figure src="/attachment/Formality%20Read%20Design%20Process%20Flow.png" alt="Formality Read Design Process Flow" width="500" >}}
