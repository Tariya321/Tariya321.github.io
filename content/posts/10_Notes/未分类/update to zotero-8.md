---
title: "update to zotero-8"
date: 2026-01-26
tags:
  - zotero
  - update
publish: yes
---
![PixPin_2026-01-26_11-20-22](/attachment/PixPin_2026-01-26_11-20-22.png)

260128_15:14
已知 BUG：通过 http 协议的 webdav 无法 PUT 数据（写数据到 webdav）
旧版 zotero7 功能正常，目前尚且不知道究竟算是谁的问题

报错信息：
```
Your WebDAV server returned an HTTP 0 error for a PUT request.

If you receive this message repeatedly, check your WebDAV server settings or contact your WebDAV server administrator.
```

可以看到论坛中也有众多相同问题
{{< figure src="/attachment/PixPin_2026-01-28_15-21-40.png" alt="PixPin_2026-01-28_15-21-40" width="580" >}}

260128_23:05
有论坛回复到，采用 https 连接，并给予自认证/SSL 证书可以解决问题

