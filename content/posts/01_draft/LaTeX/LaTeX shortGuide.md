---
title: "LaTeX shortGuide"
date: 2023-03-02
tags:
  - latex
publish: yes
---
- $\LaTeX$ 是一种着重文字本身的编程语言，format 的事情交给引擎
- 区别于常用的word文档，$\LaTeX$ 并非所见即所得，这就导致了它注定是不直观的，要求写作者需要掌握必要的编辑知识才能入门。

How this package works? You can type below to find the tutorial
```shell
texdoc <package_name>
```

## 1. 基本内容
---
### 1.1. 字符的输入
---
通过在单个字符前输入反斜杠，例如 \$ 可以得到该字符
注意，若欲得到反斜杠，则需要使用 $\backslash$ 语句或者\textbackslash 命令
- 该命令自动忽略之后的空格，因此若使用在句子中时，应加上花括号  {} 后缀
	- 正确输入\LaTeX，大小写的错误将导致命令错误
### 1.2. 导言区设置
---
- 在\documentclass和\begin{document}之间的区域称为导言区
- 使用\usepackage命令调用宏包，还可以进行文档的全局设置
	- \usepackage命令可以一次性调用多个宏包，只需用逗号隔开
	- cmd下调用命令  texdoc <宏包名>  可以查看其说明

- 通常可选参数放置于中括号[]中，而必选参数则放置于大括号{}中
	- 这部分可见文档【The Short Introduction of LaTex2e】:P17-18

class 部分的可选项目
{{< figure src="/attachment/LaTeX%20shortGuide.png" alt="LaTeX shortGuide" width="500" >}}

 一个示例
```tex
\documentclass[11pt,twoside,a4paper]{article}
```

### 1.3. 篇幅处理与input命令
---
○ 对于较长的文章（毕业论文）；或者一个复杂的图、表、代码；内容较多的导言区；等等，将它们单独写在一个文件中使得我们便于对其进行修改和校对
○ 命令`\include`，`\input`都可以将子文件插入主文件。区别是：`\include`命令在子文件前会另起一页，而`\input`命令则是直接插入
§ 注意，插入的文件名中最好不要加空格和特殊符号，以及中文
○ `\includeonly`命令，用于导言区，指定载入某些文件，使得正文中不在其列表范围的\include命令失效
### 1.4. 编译速度
---
○ 调用宏包\<syntonly>，在导言区中使用`\syntaxonly`命令，可使其编译后不生产DVI或者PDF，只排查错误，从而提高编译速度
### 1.5. 中文支持
---
○ 使用ctex宏包，或者通过`\documentclass{ctexart}`即可使用在正文中使用中文
§ 注意源代码应保存为UTF-8格式


## 2. 排版文字
---

### 2.1. 空格和分段
○ 空格键和Tab键被视为一个空格，行末的换行符也被认为是一个空格
○ 连续两个Enter，或者行末使用\par命令，可以实现分段
○ 两个反斜杠  \\  或者 \newline 为手动换行的命令
• 断页
○ \newpage命令
○ \clearpage命令
○ 以上两种命令都可以新起一页。区别体现在双栏排版模式中：\newpage表示另起一栏，而\clearpage则表示另起一页
### 2.2. 注释
○ 用%字符作为注释，该行的所有字符均被视为注释内容
### 2.3. 连字号和破折号
○ 连字号(hyphen)、短破折号(en-dash)和长破折号(em-dash)
○ 连字号  -  用来组成符合词
○ 短破折号 - - 连接数字表示范围
○ 长破折号 - - - 连接单词，类似于中文的破折号
### 2.4. 省略号
○ \ldots命令
○ \dots命令
		


## 3. 文档元素
---
### 3.1. 章节和目录
- \chapter{}     \section{}    \subsection{}：章、节、小节. 默认带编号
- `\tableofcontents` 命令可以产生目录
- 正确生成目录项，一般需要编译两次源代码
- 定制相关内容需要使用 `tocbibind` 等宏包进行修改设置

https://tex.stackexchange.com/questions/198800/how-to-number-and-cross-reference-subsubsection-level-headers
打开第四层次

[LaTeX自定义目录新手教学！【超有用】 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/398759609)

```TeX
\newpage
\tableofcontents
```

### 3.2. 书签
---
上一节已经创建了目录页，为了便于PDF阅读，还可以加上PDF书签
[请教在 LaTeX 中使用带有中文的 PDF 书签的正确姿势？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/25074347)

```tex
\documentclass[hyperref]{ctexart}
```

建议编译两次，之后自动对每个章节添加书签


### 3.3. 标题页
○ 在导言区进行设置
§ 注意：`title`和`author`是必选的，`date`则是可选的，且可以使用`\today`命令自动生成当前日期
§ 可有多为作者，需使用  `\and`  命令隔开
§ 另外，还需要在`\begin{document}`之后加上命令`\maketitle`，否则标题页无法展现
○ 标题页题注
§ 在`\author`命令后使用`\thanks{}`即可
○ 示例代码
```latex
\title{attempt}
\author{Hong Zhilian\thanks{E-mail:hongzhilian@outlook.com}
\and Ted\thanks{Corresponding author}
\and Louis}
\date{August 2022}
```
### 3.4. 交叉引用
○ 部分重要章节、公式、图表可能被引用，使用命令 \label{}  对该位置进行标签化
§ 标签的位置一般在待标签的图标、章节等之后
○ 之后，可使用 \ref{}   \pageref{}  命令，分别生成交叉引用的编号和页码
§ 一般需要编译多次
§ 引用时，将只显示数字编号，而不会显示“图”“节”等，需要自行在文本中添加
- 注意，需要将 label 写在 caption 之后，否则引用出错
§ 注意不要将aux文件清除
### 3.5. 脚注和边注
○ \footnote{} 命令用于生成脚注
§ 个别情况，无法只通过该命令得到脚注，见【The Short Introduction of LaTex2e】：P23
□ 例如在section标题中
○ `\marginpar{}` 命令用于生成边注
§ 其完全形式为 `\marginpar\[\<left-margin>]{\<right-margin>}` 
§ 由于边注较窄，最好设置较小的字号. Be like `\footnotesize`
### 3.6. 特殊环境
---
○ 列表环境
§ 有序列表环境`enumerate`
§ 无序列表环境`itemize`
§ 列表命令
□ `\item`命令可标明每个列表项
® 可使用参数替代原符号
® 可嵌套（最多四层）
○ 对齐环境
§ Center、flushleft 和 flushright 环境分别对应居中、左对齐和右对齐的文本环境
§ 当然，对于浮动体而言，用`\centering`命令即可，没必要加上`center`环境
○ 引用环境
§ `quote`环境用于引用较短的文字，不增加缩进
§ `quotation`用于若干段文字，增加缩进
○ 代码环境
§ `verbatim`环境以等宽字体排版代码
§ `\verb|<>|` 可排版简短的代码或者关键字
>在beamer中使用需要在frame后加上`[fragile]`选项
### 3.7. 图片
○ 导入`graphicx`宏包支持
○ `\includegraphics`命令加载图片
§ 一般用jpg格式比较稳定，图片最好不要超过200kb
○ `\graphicspath{figures/}`，可以直接使用该目录下的图片

### 3.8. 图片占位符
---

```latex
\includegraphics[width=.5\linewidth]{example-image-duck}  % 一只可爱的小鸭子. 用多次每次的鸭子都不一样
```

支持本地调用

并且多次使用可以出现不同的鸭子

### 3.9. 盒子
---
- 水平盒子
	- \\mbox{}
		- 生成一个基本的水平盒子，内容只有一行，此时断行命令失效
	- `\makebox\[\<width>][\<align>]{}`
		- 对齐方式align可选居中c（默认值）、左对齐l、右对齐r和分散对齐s
		- width的单位为em
- 带框的水平盒子
	- \\fbox{}
	- `\framebox[\<width>][\<align>]{}`
		- 可通过\setlength命令调节边框的宽度\fboxrule和内边距\fboxsep（见P45）
- 垂直盒子（见P46）
### 3.10. 浮动体
---
- figure环境: 习惯上放图片。
§ `\begin{figure}[\<placement>]`，参量placement，用以表示浮动体允许排版的位置在使用molticol分栏时，使用宏包，加参数H；否则，可用h
- able环境: 习惯上放表格
§ `\begin{figure}[\<placement>]`
- 标题
§ \caption{}：自动编号
§ \caption*{}：不带编号

## 4. 排版数学公式
---
• 章前准备
○ 引用amsmath宏包
• 单个公式排版基础
○ 行间和行内公式
§ 行内公式：一对$符号包裹
□ 为了与文字相适应，因此行内公式显得较为局促
§ 单独成行的行间公式：由equation环境包裹
□ 自动生成编号，且可以与 \ref 命令、 \label命令 相互联动
◊ 注意应写在公式一行，否则无法识别
□ equation* 环境不会自动生成编号
○ 数学模式
§ 当使用上述的方法输入公式时，实际上LATEX就进入了数学模式，于文本模式相比较，数学模式具有以下的几大特点
□ 空格被忽略。需用 \quad 或 \qquad命令人为引入间距
□ 不可换行。多行公式需用到其他环境

• 数学符号（见P52: 56）
○ 推荐引入bm宏包，这样可以调用`\bm{}` 命令输出粗体数学字符
### 4.1. 多行公式
---
○ 长公式换行
	§ amsmath宏包的`multline`环境
		□ 允许用 \\ 换行
		□ 在该环境下，公式的首行左对齐，末行右对齐，其余行居中
		□ 公式编号将自动放置于最后一行
○ 多行（组）公式
	§ align环境：自动按等号对齐
		□ 在等号左侧添加 & 符号以隔开两部分
		□ 该环境将自动对每行公式进行编号，可以使用`\notag`去除掉不需要的编号（此时需要一些其他的操作，见P57页说明）
		□ 公式之间使用 `\\` 断行
○ 罗列多个公式
	§ gather环境
		□ 不按照等号对齐，只是居中
### 4.2. 数组和矩阵（见P58）
• 公式中的间距（见P59）
○ \quad
○ \qquad
○ \!    减小间距
○ amsmath宏包中的二重积分\iint  三重积分`\iiint`
• 数学符号的字体控制
○ 字体改变
○ 加粗
§ amsmath宏包提供的 `\boldsymbol{}` 命令
§ bm宏包提供的 \bm{} 命令【⭐⭐】
### 4.3. 定理环境
○ 证明环境
§ amsthm宏包提供的 proof 环境，末尾将自动加上一个证毕符号
• 符号表（见P65）


## 5. 排版样式设定
---
• 字体和字号
○ 字号
§ 类似\LARGE的命令将使得其后的所有字符为LARGE，为了使其只在局部生效，则需要用花括号分组
○ 字体命令、字号、文档字号大小  见P73
• 文字装饰和强调
○ 下划线
§ \underline{} 命令：添加下划线
□ 不够灵活、无法换行
§ 基于ulem宏包的 \uline{} 命令
□ 更灵活、自动换行
○ 斜体强调
§ \emph{}命令
### 5.1. 段落格式和间距
○ 长度和长度单位
§ 见P78页
○ 行距
§ `\linespread{<factor>}`
□ factor缺省时为1.2倍字体大小
□ 添加在导言区中，则控制全局格式
□ 添加在局部，则需用 花括号{} 包括，并且用\selectfont命令使该命令生效
® 注意末尾要添加 \par命令. 这是因为行距的改变要直到文字分段才会生效
○ 段落格式
§ 设置缩进间距
□ `\setlength{\leftskip}{<length>}`
□ `\setlength{\rightskip}{<length>}`
□ `\setlength{\parindent}{<length>}`
§ 缩进控制的命令（在段落开头使用）
□ \indent
□ \noindent
○ 水平间距
§ \hspace{} : 加入额外水平间距
§ `\stretch{<n>}`：生成一个特殊弹性长度，参数`<n>`为权重
□ 需要嵌套在`\hspace{}`命令之内
○ 垂直间距
§ \vspace{} ：增加该行垂直间距，不自动分段
§ \\[] : 增加该行垂直间距，自动分段
• 页面和分栏（见P80~83）
• 页眉页脚（见P83~85）
○ `\pagestyle{<page-style>}`：修改全局页眉页脚样式
○ `\thispagestyle{<page-style>}`：修改当前页的页眉页脚格式
§ Page-style参数为样式的名称，latex共预定了四类样式（详见文档）

## 6. 特色工具和功能
---

### 6.1. 版本内容对比
---
[i学堂-玩转 LaTeX排版-苏韩-20230302_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ts4y1L7hB/?spm_id_from=333.337.search-card.all.click&vd_source=aebba21bc4a0f36d05cfff05d5e944b2)

![Screenshot_20230312_132903_tv.danmaku.bili](/attachment/Screenshot_20230312_132903_tv.danmaku.bili.jpg)

### 6.2. 参考文献和BIBTEX工具
○ 方案A：使用LaTex自带的参考文献命令：`\cite[<page>]{<citation>}`
§ citation为参考文献的标签
§ 参考文献由thebibliography环境包裹，示例如下
```tex
\begin{thebibliography}{<widest label>}
\bibitem[<item number>]{<citation>}…
\end{thebibliography}
```
□ widest一般设置为参考文献的数目即可
□ Item number用于自定义文献的序号，如果省略，则为自然顺序
○ 方案B：使用BIBTEX排版参考文献
i. 准备好bib格式的文献数据文件（与代码处于同一目录）
ii. 在源代码中添加命令
1) `\bibliographystyle{<bst-name>}`命令设定文献格式（不带后缀bst）
a) 一般放在导言区\maketitle之后
2) 使用`\bibliography{<bib-name>}`命令添加文献（不带后缀bib）
a) 一般放在`\appendix`之后
### 6.3. 索引和makeindex工具
○ 颜色
§ color或者xcolor宏包
□ 两者的区别在于color宏包颜色少一些
### 6.4. 使用超链接
○ hyperref宏包
§ 为减少冲突，一般将该宏包放在最后调用
○ 超链接
§ `\url{<url>}`：为URL加上了超链接
§ `\nolinkurl{<url>}`：只有URL，无超链接
§ `\herf{<url>}{<text>}`：将一段文字作为超链接
○ PDF书签
§ `\pdfbookmark{<bookmark>}{<anchor>}`
□ bookmark为书签名称，anchor为书签使用的标签
□ 注意：宏包已经提前处理了部分命令，可以直接使用，但仍然存在未被处理好的命令或者公式，此时使用`\texorpadfstring{<LATEX code>}{<PDF bookmark text>}`
○ PDF文档属性

