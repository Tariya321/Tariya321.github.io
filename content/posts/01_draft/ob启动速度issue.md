---
title: "ob启动速度issue"
tags:
  - obsidian
  - issue
publish: yes
---
2024-11-27
ob 启动占用较长时间的部分
{{< figure src="/attachment/2024-11-27.png" alt="2024-11-27" width="400" >}}

2024-12-01_23:10，关于 ob 启动有 14 个 tab 的事情，该不会是 homepage 引用的 14 个 link 吧？

2024-12-03
向官方提交 bug：启动时打开引用文件
```
https://forum.obsidian.md/t/obsidian-slow-start-up-if-one-page-linked-many-pages-is-pinned/92662
```

2024-12-07_14:55，论坛管理员回信，但是没有邮件提醒，现在给它开启了
tips：forum 和中文 forum 不是同一个网页，注册用户不互通

2024-12-10_10:59
对方认为这是一个 plugin 引起的问题，建议我用严格模式来测试是哪个引起的

2025-01-08_09:56
关闭了 homepage 插件，并且取消了页面的 pin
但是启动速度还是在 workspace上出现了问题


2025-02-22_14:08
再次提了问题，但这次主要强调是 workspace 启动速度较慢
https://forum.obsidian.md/t/low-workspace-start-up-time/97110
对方提示我可能是 installer 和版本都比较老了
{{< figure src="/attachment/ob%E5%90%AF%E5%8A%A8%E9%80%9F%E5%BA%A6issue.png" alt="ob启动速度issue" width="300" >}}

新版本，直接安装到任意位置，它会自动检测的，老 vault会保存
{{< figure src="/attachment/ob%E5%90%AF%E5%8A%A8%E9%80%9F%E5%BA%A6issue-1.png" alt="ob启动速度issue-1" width="300" >}}

2025-02-23_10:46
坏消息，今天再次启动时发现还是有问题
{{< figure src="/attachment/ob%E5%90%AF%E5%8A%A8%E9%80%9F%E5%BA%A6issue-2.png" alt="ob启动速度issue-2" width="300" >}}
>昨天的优化可能是因为两次启动的间隔较短导致的


2025-02-23_20:59
- 使用 sandbox vault 查看问题是否存在
- 使用安全模式（屏蔽所有第三方插件），查看问题是否存在

2025-02-26_19:44
排除变量：homepage 中的连接增加，start-up time 不会增加，且项目数未增
下一步：关闭更多插件，并取消 pin homepage，等待下一次启动
2025-02-27_10:03
今日启动，依旧缓慢。下一步，直接 sandbox 模式？
或者，启用安全模式（禁用所有插件）？

2025-02-28_13:26
highly suspect it's introduced by `dataview` plugin

2025-03-01_10:25
测试，不是 `dataview` 插件的问题
那么 `homepage` 的可能性就比较大了

