---
title: "vivado使用入门"
date: 2025-06-09_23:20
tags:
  - vivado
  - EDA
  - FPGA
publish: yes
---
## 1. sim

![|375](/attachment/251080221254866.png)
![|350](/attachment/77000121257364.png)
这里我们添加一个处理模块
添加后，需要等候updating
![|346](/attachment/223482621236107.png)
再添加一个testbench，将它设置为top
![|350](/attachment/21042821258547.png)
下面运行时序仿真
![|625](/attachment/445165721230893.png)
ops，产生了一个错误
![|575](/attachment/528115021247229.png)
注意看message
![|500](/attachment/322585321240363.png)
这里说：在例化时未找到该模块
看了一下调用的模块名，是写错了
> 应注意文件名和模块名的一致，可以避免这种问题

再次运行仿真，可见我们的仿真界面
![|500](/attachment/85305921233397.png)
但是好像并未按照我们设计的思路跑，查找问题
发现其实不是这样的，只是目前波形非常小，鼠标单击右滑即可波形放大
![|650](/attachment/127110222242344.png)
稍微修改一下代码，显示正确了
![|625](/attachment/40370922235229.png)
接下来，查看其RTL电路图
![|253](/attachment/205911222262793.png)
![|475](/attachment/538661222245006.png)
点击这个加号
![|375](/attachment/340731322234304.png)
![|475](/attachment/595701322230555.png)
这样，内部门电路就可以看见了

接下来是综合
![](/attachment/8811022235838.png)

## 2. block design

[高效的VIVADO BlockDesign设计方法 - 米联客(milianke) - 博客园](https://www.cnblogs.com/milianke/p/17935537.html)


## 3. IO 约束

- IO 约束=管脚约束和延迟约束
- 管脚约束：指定管脚的 PACKAGE_PIN 和 IOSTANDARD 两个属性的值,前者指定了管脚的位置,后者指定了管脚对应的电平标准。
- 延迟约束用的是`set_input_delay`和`set_output_delay`，分别用于input端和output端，其时钟源可以是时钟输入管脚，也可以是虚拟时钟。
	- 但需要注意的是，这个两个约束并不是起延迟的作用
	- 这个约束是告诉vivado我们的输入信号和输入时钟之间的延迟关系，从而去做 P&R

[vivado分配引脚 - 心比天高xzh - 博客园](https://www.cnblogs.com/xzh-personal-issue/p/17317625.html)
[关于XILINX的XDC约束文件编写 - szfpga - 博客园](https://www.cnblogs.com/fpga-design/p/18890997)
[FPGA时序约束理论篇之IO约束 - 知乎](https://zhuanlan.zhihu.com/p/88186632)
https://zhuanlan.zhihu.com/p/646721508
两种方法
- vivado gui 界面进行分配（之后可以 gen 一个 xdc 文件）
- 直接写 xdc 文件


