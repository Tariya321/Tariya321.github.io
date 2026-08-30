---
tags:
  - visio
  - office
  - 指南
publish: yes
---
# Visio

## 1. 快捷键

| Keybinding         | Action    |
| ------------------ | --------- |
| `Ctrl + Shift + A` | Auto Size |
| `Ctrl + Shift + G` | Group     |
| `Ctrl + Shift + U` | Ungroup   |
| `Ctrl + Shift + N` | New Page  |
| `Ctrl + Shift + X` | Cut       |
| `Ctrl + Shift + C` | Copy      |
| `Ctrl + Shift + V` | Paste     |
| `Ctrl + Z`         | Undo      |
| `alt+F9`           | 打开“对齐与吸附” |
| `ctrl+H`           | 沿 y 轴翻转   |
| `ctrl+L`           | 逆时针旋转     |


## 2. 和 office 套件的交互

将 visio 中绘制的图形，以可编辑的方式粘贴到 ppt 中
1. 在 visio 中选中图形并复制
2. 在 ppt 中"开始"-"粘贴"-"选择性粘贴"-"visio 绘图对象"

{{< figure src="/attachment/PixPin_2026-03-22_14-05-33.png" alt="PixPin_2026-03-22_14-05-33" width="416" >}}


## 3. 导出清晰的图片
---
分辨率选择——打印机
大小选择——源


### 3.1. 绘制折线

画完一部分直线后，在端点处先按下shift，然后鼠标拖动即可绘制折线


## 4. 模具

我的模具位于"D:\\创作\\visio FPGA 模板"

## 5. 显示参考网格线

{{< figure src="/attachment/PixPin_2026-04-16_17-12-02.png" alt="PixPin_2026-04-16_17-12-02" width="543" >}}

## 6. 导出无白边的 pdf

[如何从visio导出不带白边而且图片适应尺寸的pdf_visio图片导出pdf合适大小-CSDN博客](https://blog.csdn.net/XIA_RU/article/details/131558309)

- 打开开发者模式
- 显示“shapesheet”中的页
- 设置 margin 为 0
- 导出为 pdf 时候关闭“辅助功能文档结构标记”


{{< figure src="/attachment/PixPin_2026-04-16_22-26-01.png" alt="PixPin_2026-04-16_22-26-01" width="644" >}}

{{< figure src="/attachment/PixPin_2026-04-16_22-25-33.png" alt="PixPin_2026-04-16_22-25-33" width="542" >}}

使用宏一键导出，参考VBA脚本-一键导出当前visio活动页


## 7. 修改默认线宽

[Microsoft Visio 专业版2019 默认字体和线条粗细自定义 - 知乎](https://zhuanlan.zhihu.com/p/257928840)

1. 打开开发人员模式
- 打开"绘图资源管理器"
- 右键主题，选择"自定义样式"
- 选择形状，修改线宽

{{< figure src="/attachment/PixPin_2026-04-16_14-41-36.png" alt="PixPin_2026-04-16_14-41-36" width="458" >}}

{{< figure src="/attachment/PixPin_2026-04-16_14-48-32.png" alt="PixPin_2026-04-16_14-48-32" width="513" >}}

{{< figure src="/attachment/PixPin_2026-04-16_14-50-43.png" alt="PixPin_2026-04-16_14-50-43" width="472" >}}

