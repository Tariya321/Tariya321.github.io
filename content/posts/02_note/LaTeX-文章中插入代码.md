---
title: "LaTeX-文章中插入代码"
date: 2025-06-09_23:11
tags:
  - latex
publish: yes
---
[LaTeX 里「添加程序代码」的完美解决方案 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/65441079)
[不一样的 LaTeX 教程：使用 listings 宏包美化代码 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/464141424)

使用`Listings`宏包
注意若要使用颜色，还需加`xcolor`包

以下代码段，插入导言区

```LaTeX
\usepackage{listings}
\usepackage[dvipsnames]{xcolor}
%\usepackage{ctex}

% 用来设置附录中代码的样式

\lstset{
    basicstyle          =   \sffamily,          % 基本代码风格
    keywordstyle        =   \bfseries,          % 关键字风格
    commentstyle        =   \rmfamily\itshape,  % 注释的风格，斜体
    stringstyle         =   \ttfamily,  % 字符串风格
    flexiblecolumns,                % 别问为什么，加上这个
    numbers             =   left,   % 行号的位置在左边
    showspaces          =   false,  % 是否显示空格，显示了有点乱，所以不现实了
    numberstyle         =   \zihao{-5}\ttfamily,    % 行号的样式，小五号，tt等宽字体
    showstringspaces    =   false,
    captionpos          =   t,      % 这段代码的名字所呈现的位置，t指的是top上面
    frame               =   lrtb,   % 显示边框
}

% Python格式
\lstdefinestyle{Python}{
    language        =   Python, % 语言选Python
    basicstyle      =   \zihao{-5}\ttfamily,
    numberstyle     =   \zihao{-5}\ttfamily,
    keywordstyle    =   \color{blue},
    keywordstyle    =   [2] \color{teal},
    stringstyle     =   \color{magenta},
    commentstyle    =   \color{red}\ttfamily,
    breaklines      =   true,   % 自动换行，建议不要写太长的行
    columns         =   fixed,  % 如果不加这一句，字间距就不固定，很丑，必须加
    basewidth       =   0.5em,
}
```

使用方法
```LaTeX
\begin{lstlisting}
\end{lstlisting}
```

