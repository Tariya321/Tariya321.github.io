---
title: "tcl"
tags:
  - EDA
  - linux
  - tcl
date: 2025-07-04_19:50
publish: yes
---
## 1. why we use TCL?

The Tool Command Language, or Tcl, is an interpreted programming language with variables, procedures (procs), and control structures, to interface to a variety of design tools and to the design data.

The language is easily extended with new function calls, and has been expanded to support new tools and technology since its inception and adoption in the early 1990s. It has been adopted as the standard application programming interface, or API, among most EDA vendors to control and extend their applications.
上世纪 90 年代被引入，如今已经成为大多数 EDA 软件供应商的通用编程接口语言


关于 tcl 的更多介绍，请参见
[数字芯片设计武器 -- TCL脚本语言介绍（上篇） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/416370937)
[IC 极客： IC 为何偏恋三十而立的Tcl - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/35911911)
[Tcl8.6.13/Tk8.6.13 Documentation](https://www.tcl.tk/man/tcl8.6/contents.html)

TCL（Tool Command Language）
>读音：tiko

下面是开发者撰写的经典书籍
{{< figure src="/attachment/v2-bf3c54a4dbd0f94b08ff8bd855365f7e_720w.webp" alt="v2-bf3c54a4dbd0f94b08ff8bd855365f7e_720w" width="300" >}}

- 请提前准备好解释器（在xx_shell中或者在文件中写好解释器路径）
- 太长需要换行时，使用`\`可断行
- 务必在符号之间空格
- `.tcl`是其文件格式

expr求值表达式
### 1.1. 引用
双引号取消空格、制表符、换行和分号的特殊解释；
大括号取消所有特殊字符

### 1.2. 循环和条件判断
foreach
for
if

set定义变量
puts输出
proc定义过程

## 2. 实用主义

- 返回文件夹下所有符合格式的文件： `glob`
>可以避免将冗长的文件名列表写入 tcl脚本

