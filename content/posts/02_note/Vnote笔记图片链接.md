---
title: "Vnote笔记图片链接"
tags:
  - OneClickSolution
  - python
  - ob仓库
publish: yes
date: 2023-02-11
---
链接
图片链接一旦移动位置（我的文件体系中附件在根目录下的一个固定文件夹），其链接无法自动更新
打算手修这个问题了，比较费功夫
打算写个脚本来做了，因为规模比较大。。

## 1. 问题归因
找到问题了，是因为在vnote中的一个格式，调整图像大小导致的路径错误，从而更新链接失败
{{< figure src="/attachment/Pasted%20image%2020240331204812.png" alt="Pasted image 20240331204812" width="300" >}}

下面打算使用脚本来操作
{{< figure src="/attachment/Pasted%20image%2020240331211321.png" alt="Pasted image 20240331211321" width="500" >}}

首先做一个测试，看能不能work，再运用到数据库

打印变量files，结果表明其监测到了。
那么就应该写这个过程不对

用python写脚本吧，担心这个控制不了

经过一系列调整，最终得到一个python脚本
>这个过程，我觉得要清晰的表达需求，并且对其给出的脚本进行校验。

## 2. work脚本
```python
import os  
import re  
  
# 定义要遍历的文件夹路径  
folder_path = "D:/test/"  
  
# 获取文件夹中的所有 .md 文件  
for root, dirs, files in os.walk(folder_path):  
    for file in files:  
        if file.endswith(".md"):  
            file_path = os.path.join(root, file)  
            with open(file_path, "r", encoding="utf-8") as f:  
                content = f.read()  
  
            # 使用正则表达式删除匹配字符串  
            new_content = re.sub(r'\s*=\w*x', '', content)  
  
            # 将处理后的内容写回文件（使用 UTF-8 编码）  
            with open(file_path, "w", encoding="utf-8") as f:  
                f.write(new_content)
```

函数功能
现在，这个更新后的 Python 脚本会在 `D:/test/` 文件夹及其子文件夹中遍历所有的 `.md` 文件，并删除每个文件中行内包含以` =`开头，以 `x` 结尾的字符串。在写回文件时，使用 UTF-8 编码。

