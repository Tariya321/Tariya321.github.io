---
title: "obsidian使用指南"
tags:
  - obsidian
  - 知识系统
publish: true
date: 2023-10-14
---
---
使用  $\verb|![[]]|$ 可以在当前page下把其它位置的图片贴过来

`[name](links)`插入网页链接

使用$\verb|[#](#section)|$可以引用其它文章的大标题

按键|作用
--|--
ctrl+L|todo项目
ctrl+G|全局关系图谱

在分割线下无法直接输出表格，需要隔开一行

鼠标悬浮链接，支持预览

小特性：在标号前输入数字自动被大纲忽略

## 1. 知识体系管理
---
创建、整理、归档，笔记的生命线

创建一个笔记的原因：一件事情、一个文件的阅读等等~

关于笔记系统的迁徙，随着笔记生态库中更多 app 的加入，记录者们拥有了更多选择的权利
我的笔记库比较杂糅，首先需要把其它库中的笔记迁移过来，开启专栏说明
Build my library

## 2. 安装插件
---
>接口中搜索需要打开系统代理模式；需要关闭安全模式

外部下载插件方法请见[这里](https://blog.csdn.net/fonrye/article/details/128144903)，下载latest版本的zip文件，解压即可

插件安装位置
>我是"D:\MyWorkEnv\ForHigh\.obsidian\plugins"

重启obsidian后可以看见新的插件

## 3. 核心插件
---
### 3.1. 使用模板
---
定义一个page，例如我的日记模板模板-02日记
在任意一个page里点击侧边栏的templater，选中定义的模板即可

### 3.2. 调整布局
---
需要打开`布局`功能
然后鼠标拖动吸附，之后命名并保存调整好的布局

### 3.3. 斜杠命令
---
输入斜杠`/`可以呼出命令面板，输入关键字进行命令检索

### 3.4. 幻灯片
---
支持在选项中对文本进行幻灯片演示

使用`---`作为幻灯片之间的分隔符号

### 3.5. 日记
---
- 支持打开obsidian后默认创建并打开当日日记
- 配合模板可以实现高度自动化


## 4. 语法
---
### 4.1. Callout
---

**一行**
语法
```
>[!note] your text here...
```
示例
>[!note] 你好
>

>[!hint] 不错的提案

**折叠**
语法
```
>[!note]- 参考
>文献1
>文献2
>...
```
示例
>[!quote]- 参考文献
>《不是我的》
>《是你的》


可替换的字符
```
- note
- abstract, summary, tldr
- info, todo
- tip, hint, important
- success, check, done
- question, help, faq
- warning, caution, attention
- failure, fail, missing
- danger, error
- bug
- example
- quote, cite
```

### 4.2. 调节图片大小
---
本地图片调整大小，示例
```python
![[xxxx.jpg|width]]    # width调整大小，一般输入500
```

或者使用插件"**Mousewheel Image Zoom**"，该插件可实现"*alt+鼠标滚轮*"控制obsidian图片的视图大小


2026-03-16_16:39 it's time to uninstall this plugin, since the latest version has introduced resize officially.

## 5. 主题
---
使用minimal主题

并下载对应的插件，调节主题

### 5.1. css snipets
一个优化 yaml 到正文距离的 css 脚本
```css
/* Hide redundant Properties UI */
.metadata-properties-heading,
.metadata-add-button {
    display: none !important;
}

/* Compact Properties spacing */
.markdown-preview-view .metadata-container {
    margin-block-end: 8px !important;
    padding-block-end: 0 !important;
}

.markdown-preview-view .metadata-content {
    margin-block-end: 0 !important;
    padding-block-end: 0 !important;
}

.markdown-preview-view .metadata-container.is-collapsed {
    margin-block-end: 8px !important;
    padding-block-end: 0 !important;
}
```


# 2 社区插件

## 1. 文件夹icon化
---

## 2. floating TOC
---
支持左侧悬浮标题栏目

## 3. 快速插入日期
---
**Naturally Language Dates** 插件

使用方法：文本中输入 @ 即可召唤
{{< figure src="/attachment/Pasted%20image%2020231014212455.png" alt="Pasted image 20231014212455" width="200" >}}

## 4. 插入表情
---
插件 Emoji Toolbar

按下`ctrl+alt+E`可以打开表盘
或者使用斜杠命令，输入emoji查询

>这个插件是从twitter上拉取的，在无VPN时may failure

## 5. 可视化Dataview
---


修改记录
2024-10-07 修改日期格式以优化 dataview汇总效果
{{< figure src="/attachment/Obsidian.png" alt="Obsidian" width="600" >}}

### 5.1. 帮助页面
---

| webpages                                                                               | 推荐程度           |
| -------------------------------------------------------------------------------------- | -------------- |
| [Dataview (blacksmithgu.github.io)](https://blacksmithgu.github.io/obsidian-dataview/) | 官方文档           |
| [Obsidian 插件之 Dataview - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/373623264)       | 进阶             |
| [Obsidian插件dataview基本使用 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/543891539)      | 参数介绍           |
| [Obsidian DataView 入门保姆级引导手册 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/614881764) | 非常详细           |
| [Dataview常用语法速查🚀](https://obsidian.vip/zh/dataview/dataview-advanced-f.html)          | 可读性较好的文档       |
| https://blacksmithgu.github.io/obsidian-dataview/reference/functions/                  | Query function |

YAML格式，设置`文档属性`

### 5.2. 使用指南
---
相关键值对一定要写在文件头部

参数名 建议为英文
数值 允许中文

需要在设置中 Dataview插件下把`Enable Javascript Queries`和`Enable Inline Javascript Queries`这两个功能打开（默认是关闭的）

默认的表头为英文，使用例如`file.mtime as 创建时间`可以替换为`创键时间`


### 5.3. template
---
对应的模板
```text
​```dataview
[list|table|task] field1, (field2 + field3) as myfield, ..., fieldN
from #tag or "folder" or [[link]] or outgoing([[link]])
where field [>|>=|<|<=|=|&|'|'] [field2|literal value] (and field2 ...) (or field3...)
sort field [ascending|descending|asc|desc] (ascending is implied if not provided)
​```
```
>注意代码格式为`dataview`，否则无法识别


在文件夹XX中列出包含文件名YY的所有页面，并按`ctime`时间排序，另一个表头是YAML属性mood（我自己定义的）
```text
table file.mtime, mood
from "XX"
where contains(file.name,"YY")
sort file.ctime asc
```
我的模板-月报就是这样的

日记中使用了task
这样，每次有一个todo-list直接task命令即可自动汇总

## 6. Numbers Heading
---
文件内标题自动添加数字索引
支持自动对标题进行数字标号增添，便于阅读

>[! tip] 支持移动章节后自动更新

## 7. Comment
---
选中需要批注的文本，按下`Ctrl` + `P`键呼出命令面板，搜索`comment`，选中`add comment`即可

在阅读模式下，可以看见高亮的文本，点击后显示批注




