---
title: Codex Remote SSH Windows socket hang up 修复
tags:
  - Codex
  - SSH
  - Windows
  - 故障排查
date: 2026-09-27
publish: true
---
# ChatGPT/Codex Remote SSH 连接 Windows 时出现 socket hang up：根因与修复

## 1. 症状与结论

ChatGPT/Codex Desktop 通过 Remote SSH 连接 Windows 主机 时，界面显示 `socket hang up`。当时 SSH、Tailscale 和目标端口均可连通。

排查发现两个问题。Windows OpenSSH 的默认 Shell 无法执行 Remote SSH 所需的 POSIX 命令。修复默认 Shell 后，Codex app-server 仍因控制目录的访问控制列表（ACL）过宽而无法启动。这是 `socket hang up` 的直接原因。

## 2. OpenSSH 默认 Shell 不兼容

Codex Remote SSH 的启动脚本需要 POSIX Shell 命令。Windows OpenSSH 默认进入 CMD 或 PowerShell 时，`sh`、`command -v` 等命令无法按预期执行。

在远端的管理员 PowerShell 中，将 OpenSSH 默认 Shell 改为 Git Bash，并重启 `sshd`。如果 Git 的安装路径不同，请替换为实际路径。

```powershell
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\App\Git\bin\bash.exe" -PropertyType String -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShellCommandOption -Value "-c" -PropertyType String -Force
Restart-Service sshd
```

在本机验证：

```bash
ssh wins 'echo "shell=$SHELL"; command -v sh; command -v git; command -v codex; codex --version'
```

本次验证得到以下结果：

- Git Bash：`/c/app/git/bin/bash.exe`
- `sh`：`/usr/bin/sh`
- Git：`/mingw64/bin/git`
- Codex：`/c/Users/user/AppData/Roaming/npm/codex`
- Codex 版本：`codex-cli 0.157.1`

这些结果确认 SSH 启动脚本和 CLI 查找正常。app-server 仍需单独检查。

## 3. 根因：app-server 控制目录的 ACL 过宽

Codex 在 `%USERPROFILE%\.codex\app-server-control` 下创建控制 socket。这个目录继承的 Windows ACL 过宽，不符合 Codex 对私有目录的要求。app-server 报错：

```text
Error: socket directory is not private to the current user
```

app-server 因此无法就绪。客户端连接 `app-server-control.sock` 时出现 `os error 10061`，Desktop 最终显示 `socket hang up`。排查时，先查看 app-server 的 ACL 错误。

## 4. 修复方法

先完全退出 ChatGPT/Codex Desktop，并确认 Remote SSH 的实际登录用户

在 Windows PowerShell 中执行以下命令。先核对 `$env:USERPROFILE`，再停止残留进程并重命名旧目录。如果无法停止进程，请使用有权管理该进程的 PowerShell。

```powershell
whoami
$env:USERPROFILE
Get-Process codex -ErrorAction SilentlyContinue | Stop-Process -Force
$dir = Join-Path $env:USERPROFILE '.codex\app-server-control'
if (Test-Path $dir) {
    Rename-Item -Path $dir -NewName "app-server-control.bad-$(Get-Date -Format yyyyMMdd-HHmmss)"
}
Test-Path $dir
```

最后一行应返回 `False`。不要用 Git Bash、Desktop 或 `mkdir` 预先创建目录。让原生 Windows `codex.exe` 创建目录及其私有 ACL：

```powershell
codex.exe -c features.code_mode_host=true app-server --listen unix://
```

命令保持运行属于正常情况。另开一个同用户的 PowerShell 窗口，检查目录和 ACL：

```powershell
$dir = Join-Path $env:USERPROFILE '.codex\app-server-control'
Get-ChildItem -Force $dir
$acl = Get-Acl $dir
$acl | Format-List Owner,AreAccessRulesProtected,Sddl
$acl.Access | Format-Table IdentityReference,FileSystemRights,IsInherited
```

确认目录由原生 Windows Codex 创建，`AreAccessRulesProtected` 为 `True`，且 ACL 仅授予当前用户所需权限。然后在运行 app-server 的窗口按 `Ctrl+C`。**保留 `app-server-control` 目录**，重新启动 Desktop 并连接 Remote SSH。本次连接已恢复。

## 5. 快速诊断流程

1. 确认 SSH 别名和登录用户，并测试 Tailscale 与目标端口。
2. 运行上面的 SSH 验证命令。如果找不到 `sh` 或 `codex`，先修复 OpenSSH 默认 Shell 和远端 PATH。
3. 如果 CLI 可用，但 Desktop 仍显示 `socket hang up`，检查 app-server 的错误输出。重点查找 `socket directory is not private to the current user`。
4. 如果出现该错误，核对登录用户的 `%USERPROFILE%`，按“修复方法”重建控制目录，并检查 ACL。
5. 重启 Desktop，确认 Remote SSH 已恢复。

## 6. 可复制命令

本机确认 SSH 实际登录用户：

```bash
ssh wins 'whoami; echo "$HOME"; command -v sh; command -v codex; codex --version'
```

远端 PowerShell 核对控制目录：

```powershell
$dir = Join-Path $env:USERPROFILE '.codex\app-server-control'
Write-Host "User: $(whoami)"
Write-Host "Profile: $env:USERPROFILE"
if (Test-Path $dir) {
    Get-Acl $dir | Format-List Owner,AreAccessRulesProtected,Sddl
} else {
    Write-Host "Control directory does not exist: $dir"
}
```

`remote_control removed false` 与此次故障无关。无需启用旧的 feature flag。
