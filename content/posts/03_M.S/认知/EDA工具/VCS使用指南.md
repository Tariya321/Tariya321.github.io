---
title: "VCS使用指南"
tags:
  - verification
  - synopsys
  - IC验证
date: 2023-12-01
publish: yes
---
Verilog Compiler Simulator，用来仿真的

[‌﻿⁢⁡‬‍‍​​⁢⁤⁣⁤‬⁤‌‬﻿‌﻿‬‍⁡⁣‬⁢⁡⁡‌⁤​‍﻿﻿⁣⁣⁡‍‍‌⁢⁢‬⁣﻿​‌‬﻿​VCS使用说明 - 飞书云文档 (feishu.cn)](https://inr4q2kmnf.feishu.cn/docx/WA4SdwCMpooIimx4I3Ach54Hnre)

[VCS入门教程(一) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/127335447)

`.f`文件

把要编译的文件的路径保存在在`.f`文件里边，编译的时候用`-f`参数调出来编译

VCS与Verdi联调Verdi
[VCS与Verdi工具初体验 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/274783452)

### 0.1. VCS指令
---
编译命令的格式：（编译选项用来控制编译过程）
```shell
# sourceFile每个.v文件要空格
vcs sourcefile <compile_time_option>
# 另一种方式
	# -l readme.log用于将编译产生的信息放在log文件内
	# -f verilog_file.f
	# -debug_all用于产生debug所需的文件
vcs -f verilog_file.f -l readme.log -debug_all
```

执行仿真命令格式：
```shell
# syntax
./simv <run_time_option>
# 另一种方式
	# -l run.log 记录终端上产生的信息
	# -gui图形化界面
./simv -gui -l run.log
```

### 0.2. 进阶使用：makefile
---
当使用到VCS更多其他功能时，编译选项会变得很长，在终端上一个一个敲变得十分不方便，我们便可以使用`makefile`来帮助我们编译仿真。
在工作目录下新建一个`makefile`文件

```
.PHONY:com sim clean 
OUTPUT = your_top 
VCS = vcs -full64 -timescale=1ns/1ns 
	-debug_all 
	-o ${OUTPUT} 
	-l compile.log 
	-f verilog_filelist.f
SIM = ./${OUTPUT} -l run.log 

com: 
	${VCS} 
sim: 
	${SIM} 
clean: 
	rm -rf ./csrc *.daidir *.log simv* *.key  
```

下面是具体的命令流程，terminal输入：`source makefile`
之后开始vcs的东西
```shell
# 开始编译
make com
# 开始仿真
make sim
# 清除中间文件
make clean
```


>由于每一个tile的filelist 现在只有DC所需要文件，其中不包含`testbench`文件。所以我希望在原有的filelist基础上，再加一行`testbench.v`文件

算啦，不如直接copy一份filelist算了

