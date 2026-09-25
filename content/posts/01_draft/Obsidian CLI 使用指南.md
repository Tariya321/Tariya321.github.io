---
title: "Obsidian CLI 使用指南"
date: 2026-08-30
tags:
  - obsidian
  - cli
  - 自动化
publish: true
---
Obsidian CLI 是 Obsidian 的命令行界面，可以从终端控制正在运行的 Obsidian，用于脚本编写、自动化以及和外部工具集成。

> [!info] 版本要求
> CLI 需要 Obsidian 1.12 安装程序；官方中文帮助页目前要求安装程序版本为 **1.12.7 或更高**。CLI 连接的是正在运行的 Obsidian 桌面应用。

## 1. 安装与启用

1. 更新到最新的 Obsidian 安装程序版本。
2. 打开 **设置 → 常规**。
3. 启用 **命令行界面**。
4. 按提示注册 CLI。
5. 注册完成后重启终端，使 PATH 变更生效。

### 1.1. macOS

CLI 注册后通常会创建符号链接：

```shell
ls -l /usr/local/bin/obsidian
```

如果符号链接缺失，可以手动创建：

```shell
sudo ln -sf /Applications/Obsidian.app/Contents/MacOS/obsidian-cli /usr/local/bin/obsidian
```

### 1.2. Windows

Windows 版本会安装终端重定向器 `Obsidian.com`，并通过 CLI 注册将 Obsidian 加入用户 PATH。注册后需要重启终端。

### 1.3. Linux

CLI 通常位于 `~/.local/bin/obsidian`。确保该目录已加入 PATH：

```shell
export PATH="$PATH:$HOME/.local/bin"
```

## 2. 快速上手

### 2.1. 单条命令

```shell
# 查看所有命令
obsidian help

# 查看某条命令的帮助
obsidian help create

# 查看 Obsidian 版本
obsidian version

# 搜索仓库中的文本
obsidian search query="meeting notes"

# 读取当前活动文件
obsidian read
```

### 2.2. 终端界面（TUI）

输入 `obsidian` 可以进入交互式终端界面。进入 TUI 后，后续命令不需要重复输入 `obsidian`：

```shell
obsidian
help
```

TUI 提供命令补全、命令历史、反向搜索和交互式帮助。

## 3. 命令语法

### 3.1. 参数与标志

- 参数使用 `参数=值` 的形式。
- 值包含空格时使用引号。
- 标志不带值，只要出现就表示开启。
- 多行文本使用 `\n` 换行，使用 `\t` 表示制表符。

```shell
# 参数值包含空格
obsidian create name="项目会议" content="会议记录"

# 标志：创建后打开，并允许覆盖已有文件
obsidian create name=Note content="Hello" open overwrite

# 创建多行笔记
obsidian create name=Note content="# 标题\n\n正文内容"
```

### 3.2. 指定 vault

默认使用当前活动 vault。需要操作其他 vault 时，把 `vault=<名称或 ID>` 放在命令前面：

```shell
obsidian vault="ForHigh" search query="Obsidian CLI"
obsidian vault="ForHigh" daily:read
```

在 TUI 中可以使用 `vault:open <名称或 ID>` 切换 vault。

### 3.3. 指定文件

许多命令支持 `file` 或 `path`：

- `file=<名称>`：按 Wiki 链接的方式解析文件名，不需要路径和扩展名。
- `path=<路径>`：从 vault 根目录开始的完整路径。
- 两者都省略时，默认作用于当前活动文件。

```shell
# 文件名唯一时，两种写法等价
obsidian read file=Recipe
obsidian read path="Templates/Recipe.md"
```

### 3.4. 复制输出

在命令后添加 `--copy`，可以把命令输出复制到剪贴板：

```shell
obsidian read --copy
obsidian search query="TODO" --copy
```

## 4. 日常工作流

```shell
# 打开今天的日记
obsidian daily

# 追加一项任务
obsidian daily:append content="- [ ] 整理会议记录"

# 读取日记内容
obsidian daily:read

# 创建一篇新笔记
obsidian create name="旅行计划" content="# 旅行计划\n\n- [ ] 预订交通" open

# 从模板创建笔记
obsidian create name="巴黎旅行" template=Travel

# 搜索笔记
obsidian search query="meeting notes" limit=20

# 列出未完成任务
obsidian tasks todo
```

## 5. 文件与文件夹

| 命令 | 用途 |
| --- | --- |
| `files` | 列出 vault 中的文件；可用 `folder`、`ext`、`total` 筛选或统计 |
| `folders` | 列出文件夹；可用 `folder`、`total` |
| `file` | 显示文件信息，如路径、扩展名、大小、创建和修改时间 |
| `folder` | 显示文件夹信息，可查看文件、子文件夹或大小 |
| `open` | 打开文件；可用 `newtab` 在新标签页打开 |
| `create` | 创建或覆盖文件；支持 `name`、`path`、`content`、`template`、`open`、`newtab` |
| `read` | 读取文件内容 |
| `append` | 在文件末尾追加内容 |
| `prepend` | 在前置元数据之后插入内容 |
| `move` | 移动或重命名文件 |
| `rename` | 重命名文件 |
| `delete` | 删除文件；默认移入回收站，`permanent` 会跳过回收站 |

示例：

```shell
obsidian files folder="01_draft" ext=md
obsidian file path="01_draft/Obsidian CLI 使用指南.md"
obsidian append file=Note content="\n## 新章节\n"
obsidian rename file=旧笔记 name=新笔记
```

## 6. 搜索、链接与大纲

```shell
# 返回匹配文件路径
obsidian search query="关键词" path="01_draft" limit=10

# 返回匹配行及上下文
obsidian search:context query="关键词"

# 打开搜索视图
obsidian search:open query="关键词"

# 查看反向链接和出链
obsidian backlinks file=Note counts
obsidian links file=Note

# 查找未解析链接、孤立文件和死端文件
obsidian unresolved verbose
obsidian orphans total
obsidian deadends total

# 查看当前文件标题结构
obsidian outline file=Note format=tree
```

`search` 默认不区分大小写；需要区分大小写时添加 `case`。搜索结果可以使用 `format=text|json`，部分列表命令也支持 `tsv` 或 `csv`。

## 7. 日记、任务与模板

### 7.1. 日记

| 命令 | 用途 |
| --- | --- |
| `daily` | 打开今天的日记 |
| `daily:path` | 获取今天日记的预期路径，即使文件尚未创建 |
| `daily:read` | 读取今天的日记 |
| `daily:append` | 在日记末尾追加内容 |
| `daily:prepend` | 在日记开头插入内容 |

`daily`、`daily:append` 和 `daily:prepend` 支持 `paneType=tab|split|window`；追加和插入还可以使用 `inline`、`open`。

### 7.2. 任务

```shell
# 列出所有任务
obsidian tasks

# 只列出未完成或已完成任务
obsidian tasks todo
obsidian tasks done

# 按文件筛选，并显示文件路径和行号
obsidian tasks file=Note verbose

# 列出日记中的任务
obsidian tasks daily

# 切换某行任务状态
obsidian task file=Note line=8 toggle

# 直接标记为完成或未完成
obsidian task file=Note line=8 done
obsidian task file=Note line=8 todo
```

任务可以使用 `ref="path/to/Note.md:8"` 定位；也可以使用 `status="?"` 等状态字符筛选。

### 7.3. 模板

```shell
# 列出模板
obsidian templates

# 读取模板
obsidian template:read name=Travel

# 解析 {{date}}、{{time}}、{{title}} 等变量
obsidian template:read name=Travel title="巴黎旅行" resolve

# 将模板插入当前活动文件
obsidian template:insert name=Travel
```

## 8. 属性、标签与字数

```shell
# 列出标签及数量
obsidian tags counts sort=count

# 查看某个标签
obsidian tag name="#project" verbose

# 列出仓库属性及数量
obsidian properties counts sort=count

# 设置和读取属性
obsidian property:set file=Note name=status value=done type=text
obsidian property:read file=Note name=status

# 删除属性
obsidian property:remove file=Note name=status

# 查看别名
obsidian aliases verbose

# 统计字数或字符数
obsidian wordcount file=Note words
obsidian wordcount file=Note characters
```

`property:set` 支持 `text`、`list`、`number`、`checkbox`、`date`、`datetime` 类型。

## 9. 文件历史与同步

```shell
# 列出活动文件的所有版本
obsidian diff

# 比较两个版本
obsidian diff file=Note from=2 to=1

# 只查看本地历史
obsidian history file=Note
obsidian history:read file=Note version=1

# 恢复本地历史版本
obsidian history:restore file=Note version=1

# 查看同步状态
obsidian sync:status

# 暂停或恢复同步
obsidian sync off
obsidian sync on

# 读取或恢复同步版本
obsidian sync:read file=Note version=1
obsidian sync:restore file=Note version=1
```

恢复历史版本和同步版本会修改笔记内容，执行前应确认文件与版本号。

## 10. Bases、书签与工作区

### 10.1. Bases

```shell
# 列出所有 .base 文件
obsidian bases

# 查看当前数据库文件的视图
obsidian base:views file=Projects

# 查询数据库并返回 JSON、CSV、TSV、Markdown 或路径
obsidian base:query file=Projects view="Active" format=json

# 创建数据库条目
obsidian base:create file=Projects view="Active" name="新项目" content="内容"
```

### 10.2. 书签

```shell
obsidian bookmarks
obsidian bookmarks total verbose
obsidian bookmark file="01_draft/Note.md"
```

`bookmark` 也可以添加文件内标题或块、文件夹、搜索查询、URL 和标题。

### 10.3. 工作区

```shell
obsidian workspace
obsidian workspaces
obsidian workspace:save name="写作工作区"
obsidian workspace:load name="写作工作区"
obsidian tabs ids
obsidian recents total
```

## 11. 插件、主题与发布

```shell
# 列出插件
obsidian plugins versions
obsidian plugins:enabled filter=community versions

# 启用、禁用或获取插件信息
obsidian plugin:enable id=plugin-id filter=community
obsidian plugin:disable id=plugin-id filter=community
obsidian plugin id=plugin-id

# 开发插件时重新加载
obsidian plugin:reload id=my-plugin

# 主题与 CSS 片段
obsidian themes versions
obsidian theme name=主题名
obsidian theme:set name=主题名
obsidian snippets:enabled
obsidian snippet:enable name=片段名
```

发布相关命令包括 `publish:site`、`publish:list`、`publish:status`、`publish:add`、`publish:remove` 和 `publish:open`。

安装社区插件、发布文件或执行插件命令前，确认目标 vault 和命令参数。

## 12. 开发者命令

开发插件或主题时，可以用 CLI 完成基本的测试与调试：

```shell
# 打开开发者工具
obsidian devtools

# 重新加载插件
obsidian plugin:reload id=my-plugin

# 显示并清除捕获的 JavaScript 错误
obsidian dev:errors
obsidian dev:errors clear

# 查看控制台错误
obsidian dev:console level=error

# 截图
obsidian dev:screenshot path=screenshot.png

# 查询 DOM 或 CSS
obsidian dev:dom selector=".workspace-leaf" text
obsidian dev:css selector=".workspace-leaf" prop=background-color

# 在 Obsidian 应用上下文中运行 JavaScript
obsidian eval code="app.vault.getFiles().length"
```

另外还有 `dev:debug`、`dev:cdp` 和 `dev:mobile`，分别用于 Chrome DevTools Protocol 调试、执行 CDP 命令和移动端模拟。

## 13. TUI 快捷键

| 操作 | 快捷键 |
| --- | --- |
| 左移 / 右移光标 | `←` / `→`、`Ctrl+B` / `Ctrl+F` |
| 跳到行首 / 行尾 | `Ctrl+A` / `Ctrl+E` |
| 删除到行首 / 行尾 | `Ctrl+U` / `Ctrl+K` |
| 上下浏览历史或建议 | `↑` / `↓`、`Ctrl+P` / `Ctrl+N` |
| 反向搜索命令历史 | `Ctrl+R` |
| 进入或接受自动补全 | `Tab` |
| 执行命令 | `Enter` |
| 清屏 | `Ctrl+L` |
| 退出 TUI | `Ctrl+C` / `Ctrl+D` |

## 14. 踩坑记录：Installer 版本与 Obsidian 应用版本

这次排查暴露出一个容易混淆的版本问题：

- Obsidian 应用版本是 **1.13.7**。
- Obsidian Installer 版本曾经是 **1.8.7**。
- CLI 要求的是 Installer 版本达到官方要求（当前文档要求 **1.12.7 或更高**），不能只看 Obsidian 应用版本。
- 更新 Installer 时**不需要卸载 Obsidian**，直接更新即可；vault、笔记和现有配置不需要迁移。

之前把 `1.8.7` 直接当成 Obsidian 应用版本，是版本概念混淆。排查 CLI 问题时，应分别确认：

```shell
# Obsidian 应用包的版本
plutil -extract CFBundleShortVersionString raw -o - /Applications/Obsidian.app/Contents/Info.plist

# CLI 是否已安装
ls -l /Applications/Obsidian.app/Contents/MacOS/obsidian-cli
```

结论：**CLI 能否工作，首先取决于 Installer 版本和 CLI 注册状态；Obsidian 应用版本与 Installer 版本不是同一个概念。**

## 15. 故障排查

- 确认使用 Obsidian 1.12.7 或更高的安装程序版本。
- 确认 Obsidian 应用正在运行；CLI 连接的是运行中的桌面实例。
- 更新后若 CLI 不工作，可以在设置中关闭并重新启用命令行界面，再重新注册 PATH。
- 注册后重启终端。
- macOS 检查 `/usr/local/bin/obsidian` 是否存在且指向正确的 `obsidian-cli`。
- Linux 检查 `~/.local/bin` 是否已加入 PATH。
- Windows 检查终端是否能找到 `Obsidian.com`，并重启终端使 PATH 生效。

如果目标是不启动桌面应用而进行同步，应查看 Obsidian Headless 相关文档；普通 Obsidian CLI 不是无头同步工具。

## 16. 参考

[Obsidian CLI 官方中文帮助](https://obsidian.md/zh/help/cli)



## 17. 排障记录：Codex 沙盒与 CLI IPC

### 17.1. 现象

Obsidian 应用已运行且版本为 1.13.7，但在 Codex 沙盒内执行以下命令：

```shell
obsidian help
obsidian version
```

仍返回：

```text
The CLI is unable to find Obsidian. Please make sure Obsidian is running and try again.
```

### 17.2. 根因

1. Obsidian 设置中的 **Settings → General → Advanced → Command line interface** 未启用。
2. 启用后还需要点击 **Register**，完成 PATH 注册。
3. 本机的 `/usr/local/bin/obsidian` 已正确指向 `/Applications/Obsidian.app/Contents/MacOS/obsidian-cli`。
4. 即使应用正常运行，Codex 沙盒仍无法访问 Obsidian 的本地 IPC；因此沙盒内的“找不到 Obsidian”不代表应用真的未运行。

### 17.3. 修复步骤

1. 打开 Obsidian 设置。
2. 进入 **General → Advanced**，启用 **Command line interface**。
3. 在出现的提示中点击 **Register**。如果显示 **Already registered in PATH**，说明注册已完成。
4. 在 Codex 中通过沙盒外的系统终端执行 `obsidian ...` 命令；Codex 工具中对应 `sandbox_permissions=require_escalated`。
5. 先用 `obsidian version` 验证连接，再执行具体操作。

### 17.4. 验证结果

```text
obsidian version
1.13.7 (installer 1.13.7)

obsidian vault info=name
ForHigh

obsidian files total
5301
```

结论：CLI 已可用；在本机 Codex 环境中，关键是完成 Obsidian CLI 注册，并在沙盒外执行命令。
