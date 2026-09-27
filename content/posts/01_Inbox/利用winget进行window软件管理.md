---
title: "利用winget进行window软件管理"
date: 2025-10-05
tags:
  - winget
  - window
  - 包管理
publish: true
---
winget（**Windows Package Manager**）是微软官方的命令行软件包管理器，类似于：
- Linux 上的 apt（Debian/Ubuntu）
- macOS 上的 brew（Homebrew）



install PowerWell[^1]
```
winget install --id Microsoft.PowerShell.Preview --source winget
```

uninstall software
```
 winget uninstall <sw_id>
```

windows 上好用的下载和升级软件工具 `winget`，常用 cmd
```
winget list --upgrade-available
winget upgrade --id <software_ID>
winget upgrade --all
```

例如安装固定 version 的 工具
```
winget install Git.Git --version 2.43.1
```

install FFmpeg with winget
```
winget install -e --id Gyan.FFmpeg
```

利用 `pin` 关闭某些文件的升级
```
winget pin add <ID>
```
这将使该包不出现在可升级列表中



利用 yaml 格式文件批量部署
```yaml
# setup.yml
sources:
  - name: winget
    type: microsoft
packages:
  - id: Python.Python.3.11
    version: 3.11.4
  - id: Git.Git
    version: 2.43.1
  - id: VisualStudioCode
  	version: 1.97.2
```
then
```
winget import setup.yml
```






[^1]: https://learn.microsoft.com/zh-cn/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5
