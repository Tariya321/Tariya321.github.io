---
title: "使用webDav同步zotero"
date: 2025-09-10
tags:
  - webDav
  - zotero
  - 同步
publish: yes
---
## 1. 设置方法
两边服务器校验成功，mac 端上传附件成功
win 端打开 pdf 显示路径不正确，说明尚未正确配置

插件`attager` 的设置
- 原来设置的是 *link* 方式，也即将 paper 集中在某些文件夹中，例如设置了自动将 paper 存储于名为 `{{collection}}` 的子文件夹中，然后 link 到条目下
- 当使用 webDav 同步时，请切换到 *stored copy* 选项
- 并将链接的附件**转换为存储的附件**
{{< figure src="/attachment/%E4%BD%BF%E7%94%A8webDav%E5%90%8C%E6%AD%A5zotero.png" alt="使用webDav同步zotero" width="600" >}}

选中任意的条目，然后选择 `undo move`
{{< figure src="/attachment/%E4%BD%BF%E7%94%A8webDav%E5%90%8C%E6%AD%A5zotero-1.png" alt="使用webDav同步zotero-1" width="600" >}}

原图标
{{< figure src="/attachment/%E4%BD%BF%E7%94%A8webDav%E5%90%8C%E6%AD%A5zotero-3.png" alt="使用webDav同步zotero-3" width="200" >}}
于是链接文件将被转化为存储文件（存储到条目一一对应的子文件夹下）
{{< figure src="/attachment/%E4%BD%BF%E7%94%A8webDav%E5%90%8C%E6%AD%A5zotero-2.png" alt="使用webDav同步zotero-2" width="200" >}}

测试发现，win 端已经可以正常打开阅读了

注意事项
- 两边的设置，除了保存路径之外，最好设置为一致。稳健的办法是通过配置文件

## 2. 问题校验

如果在同步后，另一个设备上无法访问，那么请按照如下进行操作[^1]

{{< figure src="/attachment/%E4%BD%BF%E7%94%A8webDav%E5%90%8C%E6%AD%A5zotero-4.png" alt="使用webDav同步zotero-4" width="500" >}}

## 3. reference
[^1]: [kb:files_not_syncing [Zotero Documentation]](https://www.zotero.org/support/kb/files_not_syncing)
