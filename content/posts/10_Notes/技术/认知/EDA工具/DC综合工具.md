---
title: "DC综合工具"
tags:
  - synopsys
  - synthesis
publish: yes
date: 2023-11-17
---
逻辑综合工具

DC主要完成将设计的RTL代码转换为门级网表的过程

## 1. 综合图解
---
**综合简图**
{{< figure src="/attachment/RTL%E6%A8%A1%E5%9D%97%E7%9A%84%E7%BB%BC%E5%90%88%E8%BF%87%E7%A8%8B.png" alt="RTL模块的综合过程" width="500" >}}

1. 读入的HDL代码首先由synopsys自带的GTECH库转换成Design Compiler内部交换的格式，此时可以导出一个unmapped综合网表
GTECH库是Synopsys公司提供的通用的、独立于工艺的元件库
2. 然后经过映射到工艺库和优化生成mapped综合网表
target_library即为设计所用的元件库

**DC synthesis flow**
{{< figure src="/attachment/DC_synthesisFlow.jpg" alt="DC_synthesisFlow" width="400" >}}

对于DC的综合效率而言，RTL代码的质量是最关键的
>不能指望工具帮助我们把一段垃圾代码代码综合出像样的电路来，即所谓的Garbage in --> Garbage out

## 2. DC流程
---
DC可分为下面四个流程：
[pre-synthesis processes](#pre-synthesis-processes)
[constrainting the design](#constrainting-the-design)
[synthesis the design](#synthesis-the-design)
[post-synthesis process](#post-synthesis-process)
### 2.1. pre-synthesis processes
---
预综合过程是指在综合过程之前的一些为综合作准备的步骤，包括Design Compiler的启动、设置各种库文件、创建启动脚本文件、读入设计文件、DC中的设计对象、各种模块的划分以及Verilog的编码等等。

预综合的流程如下：
[启动DC](#启动dc)
[库文件](#库文件)
[读入设计对象](#读入设计对象)
[设置启动文件.synopsys_dc.setup](#设置启动文件synopsys-dcsetup)
[设计划分](#设计划分)
#### 2.1.1. 启动DC
---
(dc_shell-t命令行模式)
```
dc_shell-t
```
之后执行`.tcl`脚本
```
dc_shell-t> source XX.tcl
```

启动DC后，将自动source setup文件
#### 2.1.2. 库文件
---
他们是工艺库、链接库、符号库以及综合库
>对于所有DC可能用到的库，我们都需要在`link_library`中指定，其中也包括要用到的IP

**工艺库**
综合后电路网表要最终映射到的库

**链接库**
`link_library`设置模块或者单元电路的引用

**符号库**
`symbol_library`是定义了单元电路显示的Schematic的库；
符号库的后缀是`.sdb`，若未设置，DC会用默认的符号库取代；
设置符号库的命令是`set symbol_library`；

**综合库**
DesignWare库`.sldb`文件用于实现Verilog描述的运算符，类似于自己的ip核心；
在初始化 DC 的时候，不需要设置标准的 DesignWare 库 standard.sldb 用于实现 Verilog 描述的运算符，对于扩展的 DesignWare，需要在 synthetic_library 中设置，同时需要在 link_library 中设置相应的库以使得在链接的时候 DC 可以搜索到相应运算符的实现。
#### 2.1.3. 读入设计对象
---
**1. shell环境直接read方式**
```tcl
# 读取XX格式的文件
read_XX YY.XX
```
`read`完之后，需要使用`link`命令将读到Design Compiler存储区中的模块或实体连接起来；

如果在使用`link`命令之后，出现`unresolved design reference`的警告信息，需要重新读取该模块；

**2. setup文件配置方式**
在`.synopsys_dc.setup`文件中添加`link_library`，告诉DC到库中去找这些模块，同时还要注意`search_path`中的路径是否指向该模块或单元电路所在的目录

或者可以配合使用`analyze`命令和`elaborate`命令：`analyze`是分析HDL的源程序并将分析产生的中间文件存于`work`(用户也可以自己指定)的目录下；`elaborate`则在产生的中间文件中生成verilog的模块或者VHDL的实体，缺省情况下，`elaborate`读取的是`work`目录中的文件。

#### 2.1.4. 设置启动文件
---
.synopsys_dc.setup
启动DC后，自动在下面三个路径中搜索setup文件
{{< figure src="/attachment/Pasted%20image%2020231114120702.png" alt="Pasted image 20231114120702" width="600" >}}
1. `$SYNOPSYS/admin/setup`目录下，DC安装的标准初始化文件。
2. 当前用户的`$HOME`目录下，一般用于设置一些用户本人使用的变量以及一些个性化设置。
3. DC启动所在的目录下，一般用于与所在设计相关的设置。

最后设置的`.setup`文件的优先级最高

`.setup` 文件主要包括库文件的设置、工作路径的设置以及一些常用命令别名的设置等等

启动DC后，将自动source当前工程目录下`.setup`文件
#### 2.1.5. 一个tcl脚本
---
```tcl
# 设置工艺库
set target_library {my_tech.db}
# 设置链接库
# 带星号*的原因可见P24
set link_library {* my_tech.db}
# 添加链接库的上级，因为DC默认在DC一级路径下搜索
lappend search_path {bob}

read_verilog /../xx.v

link
```

#### 2.1.6. 语句含义
---
一般要这样写
```
search_path "$search_path  $my_tcl_path ..."
```

这样做的目的是不把原path给over-write掉

 "\* $path1"表示designs in DC Memory

"$path1 $path2"双引号定义了可用变量的list
有时列表过长需要换行，请使用右斜线 \ 断行

". path1/dir1"点表示current working directory

对于`run_dc.tcl`脚本，应当将除generic设置之外的参数，写在这个tcl脚本里

#### 2.1.7. 设计划分
---
P30~P35
DC中可以再对模块之间的关系做调节

**模块划分四原则**
原则一. 不要让一个组合电路穿越过多的模块
原则二. 将所有的输出寄存起来
原则三. 根据综合时间长短控制模块大小
原则四. 将同步逻辑部分与其他部分分离

事实上除了通过HDL中的模块体现划分，我们还可以运用DC的两个命令（Group及Ungroup）来调整设计划分。

1. 用group创建新层次
{{< figure src="/attachment/%E7%94%A8group%E5%88%9B%E5%BB%BA%E6%96%B0%E5%B1%82%E6%AC%A1.png" alt="用group创建新层次" width="600" >}}
2. 用ungroup去除层次关系
{{< figure src="/attachment/%E7%94%A8ungroup%E5%8E%BB%E9%99%A4%E5%B1%82%E6%AC%A1%E5%85%B3%E7%B3%BB.png" alt="用ungroup去除层次关系" width="600" >}}

### 2.2. constrainting the design
---
这些约束主要包括：时序和面积约束、电路的环境属性、时序和负载在不同模块之间的分配以及时序分析


#### 2.2.1. 时序和面积
---
current_design

**定义时钟并set_dont_touch**
create_clock，创建综合用的时钟源（clk_period参考对象是外部时钟，）
对所有定义的时钟网络设置为don’t_touch，即综合的时候不对Clk信号优化。如果不加这句，DC会根据Clk的负载自动对他产生Buffer，而在实际的电路设计中，时钟树(Clock Tree)的综合有自己特别的方法，它需要考虑到实际布线后的物理信息，所以DC不需要在这里对它进行处理，就算处理了也不会符合要求。


**约束输入/输出路径**
[set_input_delay/set_output_delay - zhihu.com](https://zhuanlan.zhihu.com/p/337532021)
定义的输入延时是指被综合模块外的寄存器触发的信号在到达被综合模块之前经过的延时。
外部器件和本级芯片的时序连接关系，如果不设置，DC就不知道。
如果有外部器件的DataSheet，应当遵循；若没有，则需要和designer确认信息
一般的紧张约束是，外面设置70%，内部留30%

**总结**
至此，面积、时钟、IO都施加了约束，可使用命令`report_port -verbose`，报告在当前设计中所有的输入输出端口属性和施加的约束值

#### 2.2.2. 环境属性
---
环境属性分类
{{< figure src="/attachment/%E8%AE%BE%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%B1%9E%E6%80%A7.png" alt="设置环境属性" width="600" >}}

**设置输出负载**
DC默认输出负载为0。在实际情况下，由于输出负载的存在，transition time将延长
{{< figure src="/attachment/Pasted%20image%2020240117203734.png" alt="Pasted image 20240117203734" width="600" >}}

**设置输入驱动**
DC默认输入驱动为无穷大，对应信号的transition time为零。
可设置指定源信号先经过某元件库单元进而驱动当前设计电路

**设置工作条件**
所谓工作条件即PVT。在默认情况下，Design Compiler不会自动指定工作条件。
可通过`report_lib`命令来列出在当前的工艺库里提供了哪几种工作条件

**设置连线负载模型**
综合中连线延时是通过设置连线负载模型(wire load model)确定的。命令`report_lib`可查看由foundry提供的负载模型
在默认情况下，DC自动根据综合出来的模块的大小选择负载模型。下图是`report_design`报告中的结果
{{< figure src="/attachment/DC%E8%87%AA%E5%8A%A8%E6%A0%B9%E6%8D%AE%E7%BB%BC%E5%90%88%E9%9D%A2%E7%A7%AF%E9%80%89%E6%8B%A9%E8%B4%9F%E8%BD%BD%E6%A8%A1%E5%9E%8B.png" alt="DC自动根据综合面积选择负载模型" width="500" >}}

下面是以上设置的命令
```shell
# Apply default drive strengths and typical loads for I/O ports
set_driving_cell -lib_cell xxxx [all_inputs]
set_load {expr {load_of .../../xxx_cell} * 3} [all_outputs]

# Set operating conditions
set_operating_conditions xxx

# Turn on auto wire load selection
# (library must support this feature)
set auto_wire_load_selection true
# set_wire_load_model
```

**检查环境属性是否施加**
`write_script`将施加的约束和属性写到一个文件，用于检查
#### 2.2.3. 时序分析
---
**打印报告**
report_timing >  ./dc_output/report_timing.rpt
report_area > ./dc_output/report_area.rpt
report_constrant -all > ./dc_output/report_constraint.rpt

**使用report_timing检查时序**
{{< figure src="/attachment/Pasted%20image%2020231117203705.png" alt="Pasted image 20231117203705" width="600" >}}
显示了路径的基本信息
{{< figure src="/attachment/Pasted%20image%2020231117203718.png" alt="Pasted image 20231117203718" width="600" >}}
显示了最长路径的各单元延时情况，path表示至该节点的路径总延时
{{< figure src="/attachment/Pasted%20image%2020231117203903.png" alt="Pasted image 20231117203903" width="600" >}}
显示了constraint文件的具体约束时间，这里的clock clk的incr就是这里一个时钟节拍的时间，也即允许的最大路径延时
{{< figure src="/attachment/Pasted%20image%2020231117204118.png" alt="Pasted image 20231117204118" width="600" >}}
这是报告的最后部分，显示了时序裕量

#### 2.2.4. set_false_path和set_disable_timing的区别

set_false_path 只对data path起作用。**设置成false_path的数据路径，EDA工具仍然会计算累加这条路径上的timing arc延时，但是不优化和报告这条数据路径上的setup/hold时序违例**，会继续优化和报告这条数据路径上的逻辑DRC。

set_disable_timing 对timing arc起作用。**设置disable_timing之后，所有经过这个timing arc的timing path（data path/clock path），工具都不会去计算和分析。**

- 对于timing-loop而言，需要使用set_disable_timing 来break掉这个loop

#### 2.2.5. Commands for Setting Design Constraints
---
可见DC ug：P757

**set_false_path**
Marks paths between specified points as false. This command eliminates the selected paths from timing analysis.

**group_path**
Groups a set of paths or endpoints for cost function calculation. This command is used to create path groups, to add paths to existing groups, or to change the weight of existing groups.

#### 2.2.6. Using Floorplan Physical Constraints
---
>Design Compiler in topographical mode supports high-level physical constraints. DC启用topographic模式可支持高层次物理约束

两种方式：
1. 从布局布线工具抽取
	1. APR工具导出DEF文件后导入DC
	2. APR工具导出floorplan信息后导入DC
2. 手动创建

```shell
# APR工具ICC导出def信息
write_def -xxx -yyy my_physical_data.def
# APR工具innovus导出def信息
defOut my_physical_data.def
# DC载入def信息
extract_physical_constraints xx.def xxx.def
```

DC工具可以吐出DEF文件（需要license支持），但是只支持5.7版本
### 2.3. synthesis the design 
---
>对于同样的约束文件，每一次跑DC综合出来的结果都不尽相同。

#### 2.3.1. Compile Vs. Compile ultra
---
**compile_ultra** 命令，所有DesignWare 层次被自动取消。原本是以IP核/module形式进行综合的，但是一旦ultra完之后，这些模块之间的明显层次关系就消失了。对于综合结果的优化程度而言，compile ultra更加优越。

**compile** 命令

#### 2.3.2. 编译策略
---
分析报告，调整策略

使用的主要命令

命令|意义
--|--
report_constraint -all_violators|报告未满足的约束条件
report_timing -delay max|基于建立时间的关键路径
report_timing -delay min|基于保持时间的关键路径

#### 2.3.3. 层次化设计的编译
---


**多例化模块的编译**
uniquify
compile + dont_touch 




### 2.4. post-synthesis process
---
调整综合方法、修正约束违反、给submodule时序和负载预算、预估整体设计

#### 2.4.1. 层次化编译
----
对于规模不大的设计，优先采用top-down的方法；若使用bottom-up的方法，必须仔细考虑各个模块之间的时序和负载

**top-down模式**
将整个设计一次性读入，施加顶层约束后直接进行编译。
优点是：无需考虑各个子模块之间的依赖关系
{{< figure src="/attachment/Pasted%20image%2020231124103653.png" alt="Pasted image 20231124103653" width="600" >}}

**bottom-up模式**
先一个个编译比较底层的子模块，给它们加入时序和负载预算，然后在顶层将各个子模块整合起来。
优点是：“分而治之”的策略对于不可能一次编译完成的大型设计是十分有用的；对于工作站的要求不那么高；
缺点是：步骤较多，对各个模块之间的时序和负载要求较高，处理不好容易violated
{{< figure src="/attachment/Pasted%20image%2020231124103643.png" alt="Pasted image 20231124103643" width="600" >}}
具体的方法是：子模块综合完了，写ddc出来；在top层综合时link，再compile 


#### 2.4.2. 第二阶段编译
---
对于bottom-up模式而言，出现顶层时序违反时，可采用
```
compile -top
```
命令，其仅修正顶层子模块之间的路径，速度较compile -inc更快



#### 2.4.3. 设计预估（设计方法学）
---
design exploration

不同步骤的自由度Vs修改代价
{{< figure src="/attachment/%E4%B8%8D%E5%90%8C%E6%AD%A5%E9%AA%A4%E7%9A%84%E8%87%AA%E7%94%B1%E5%BA%A6%20Vs%20%E4%BF%AE%E6%94%B9%E4%BB%A3%E4%BB%B7.png" alt="不同步骤的自由度 Vs 修改代价" width="600" >}}

WNS (Worst Negative Slack) ：最差负时序裕量

**设计约束**


## 3. log信息
---
DC综合结束之后查阅timing_rpt，说检测到timing loop
```
Information: Timing loop detected. (OPT-150)

Warning: Disabling timing arc between pins 'A1' and 'ZN' on cell inst3/U3' to break a timing loop. (OPT-314)
```
loop 就是startpoint和endpoint都是一个flipflop的path

D到Q再通过logic再到D，在分析的时候，tools会自动break这个loop，或者人为的set_disable_timing 

timing loop在STA的时候可以减少skew来多点margin，因为launch 和capture path都是一样的，没有必要计算skew

startpoint和endpoint是同一个DFF的情况，不算是loop，也就是没有问题。组合逻辑出去回到组合逻辑输入的，才是有问题的loop

## 4. 输出Formality需要的文件
---
DC需要输出给formality用于网表一致性检查的的文件
- `.svf`文件
- `netlist`网表文件

**脚本**
```tcl
write -format verilog -hierarchy -output ./dc_output/${DESIGN_NAME}_netlist.v
write -format ddc     -hierarchy -output ./dc_output/${DESIGN_NAME}_post.ddc
write_sdf ./dc_output/${DESIGN_NAME}.sdf
write_sdc ./dc_output/${DESIGN_NAME}.sdc
set_svf -off
```

