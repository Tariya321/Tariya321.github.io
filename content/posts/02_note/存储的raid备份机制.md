---
title: "存储的raid备份机制"
date: 2026-08-31
tags:
  - raid
  - 存储
publish: yes
---
raid备份技术细节可参考：[link](https://www.cnblogs.com/guan88/articles/18681220)
不同raid阵列+不同存储盘容量的[存储容量计算](https://www.seagate.com/cn/zh/products/nas-drives/raid-calculator/)

以两槽的 NAS 来讲，可以塞进去两个硬盘，如果正常使用的话，两个硬盘设定会是 Raid 1，也就是**镜像备份**，两个硬盘上的东西是完全一样的。你每次存文件进去的时候，它的文件都会同时复制到两个硬盘里面，所以任何一个硬盘坏掉的时候，另外一个硬盘都还可以把资料找回来。你只要把坏掉的那个拔掉再插一个新的进去，它就会把文件拷贝回去

NAS存储盘的数据丢失风险因素
措施：使用UPS（不间断电源）设备以防止突然断电

[帖子1](https://www.v2ex.com/t/684846)：定期监测 smart，每天一个 short test，每周一个 long test 。失败了就邮件通知，剩下的有 raid 顶着

RAID 机制进行数据的多硬盘备份，防止忽然失效

如果采用了raid机制，那么对于硬盘的购买和使用，就必须遵循一些原则
- 大容量盘至少一下买两个，否则raid情况下会导致容量冗余无法利用

