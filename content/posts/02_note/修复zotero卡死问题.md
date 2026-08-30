---
title: "修复zotero卡死问题"
date: 2025-10-03
tags:
  - zotero
  - debug
publish: yes
---
问题描述
- zotero+其他应用，正常使用
- zotero+远程桌面+其他应用，zotero 频繁卡死
- 关闭远程桌面，zotero 瞬间复活

解决思路：
- 检查网络是否连接正常，有时因为网络导致的无法打开文献库。无效
- 编辑 zotero 的端口，默认是 `23119`, 修改为 `23118`
{{< figure src="/attachment/2025-10-03-1.png" alt="2025-10-03-1" width="500" >}}
- 编辑浏览器拓展插件的交互端口
{{< figure src="/attachment/2025-10-03.png" alt="2025-10-03" width="500" >}}

2025-10-07_15:11，修改回去了，因为发现似乎 zotero 有时候会恢复到默认的 port


- 似乎更加正确的方案：让 RDP 使用 GPU，而不是虚拟渲染驱动
打开 `gpedit.msc` >> Computer Configuration >> Administrative Templates >> Windows Components
Remote Desktop Services>>Remote Desktop Session Host>>Remote Session Environment 
启用 **Use hardware graphics adapters for all Remote Desktop Services sessions**
然后重启下电脑
{{< figure src="/attachment/%E4%BF%AE%E5%A4%8Dzotero%E5%8D%A1%E6%AD%BB%E9%97%AE%E9%A2%98.png" alt="修复zotero卡死问题" width="640" >}}


原因分析：
{{< figure src="/attachment/%E4%BF%AE%E5%A4%8Dzotero%E5%8D%A1%E6%AD%BB%E9%97%AE%E9%A2%98-1.png" alt="修复zotero卡死问题-1" width="620" >}}

参考[^1]

[^1]: [远程桌面优化避坑指南 - 知乎](https://zhuanlan.zhihu.com/p/492662854)
