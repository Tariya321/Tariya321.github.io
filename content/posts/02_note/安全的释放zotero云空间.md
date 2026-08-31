---
title: "安全的释放zotero云空间"
date: 2025-09-10
tags:
  - zotero
  - 存储
publish: yes
---
在zotero刚刚上手之际，通常会将zotero设置里的data/file syncing都打开
于是乎zotero附赠的300MB云空间不及就会被pdf填满

此note用于记录如何安全的释放zotero的云空间

1. 在本地zotero app中打开syncing选项卡，取消勾选file syncing下的选项，data syncing继续保持钩上
{{< figure src="/attachment/%E5%AE%89%E5%85%A8%E7%9A%84%E9%87%8A%E6%94%BEzotero%E4%BA%91%E7%A9%BA%E9%97%B4.png" alt="安全的释放zotero云空间" width="500" >}}
2. 审查本地的文献pdf来源，如果有部分来自于云盘sync，那么建议使用插件将文件移动到本地`rename and move`；这里由于我的file都在本地，所以不用做这步了
3. 打开zotero网页的 [storage setting](https://www.zotero.org/settings/storage)，点击红色的 `purge storage in my library` 选项。
{{< figure src="/attachment/%E5%AE%89%E5%85%A8%E7%9A%84%E9%87%8A%E6%94%BEzotero%E4%BA%91%E7%A9%BA%E9%97%B4-1.png" alt="安全的释放zotero云空间-1" width="500" >}}
可以看到我这里的current usgae已经基本降为0了，因为文本条目数据就是很小的
