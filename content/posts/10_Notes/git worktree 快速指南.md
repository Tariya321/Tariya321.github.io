---
title: "Git-Worktree-short-guide"
date: 2026-09-28
tags:
  - git
publish: true
---
## 1. 什么是 Git Worktree

Git Worktree 是 Git 提供的一种机制，允许一个 Git 仓库同时拥有多个工作目录，每个目录对应不同的 branch。

传统 Git：

```text
repository
    |
    └── 当前只能 checkout 一个 branch
```

切换 branch：

```bash
git checkout main
git checkout feature
```

会修改同一个工作目录中的文件。

使用 Worktree 后：

```text
              Git Repository
                    |
      -------------------------------
      |              |              |
    main          feature       experiment

    dir A          dir B          dir C
```

特点：

- 多个 branch 可以同时存在
- 每个 worktree 拥有独立工作目录
- 所有 worktree 共享 Git 历史和对象数据
- 不需要为每个 branch 单独 clone 仓库
- 适合多方向并行开发


---

## 2. Worktree操作

### 2.1. 查看已有 Worktree

进入任意 Git worktree：

```bash
git worktree list
```

示例：

```text
/home/user/project          abc123 [main]
/home/user/feature-dev      def456 [feature]
```

含义：

```text
路径                    当前 branch
--------------------------------
project                 main
feature-dev             feature
```

---

### 2.2. 创建 Worktree

创建新的 branch 并建立 worktree：

```bash
git worktree add <path> -b <branch-name>
```


使用已有 branch：

```bash
git worktree add <path> <branch-name>
```


---

### 2.3. 删除 Worktree

删除 worktree：

```bash
git worktree remove <path>
```

注意：

- 删除 worktree 不会删除对应 branch
- branch 和 commit 历史仍然保留

如果目录已经被手动删除：

```bash
rm -rf <path>
```

需要清理 Git 中的记录：

```bash
git worktree prune
```

---

## 3. Worktree 工作原理

普通 Git clone：

```text
project/
├── .git/
├── src/
└── README.md
```

Worktree：

```text
main repository

project/
├── .git/
│   ├── objects/
│   ├── refs/
│   └── worktrees/
│
└── src/


extra worktree

feature-dev/
├── .git
└── src/
```

额外 worktree 中的 `.git` 不是完整 Git 仓库，而是一个引用文件：

```text
gitdir: /path/to/project/.git/worktrees/feature-dev
```

因此：

- 工作目录相互独立
- Git 数据共享
- 节省磁盘空间

### 3.1. Worktree 限制

**同一个 branch 不能同时被多个 worktree 使用**

例如：

```text
main branch
    |
    └── 已被 main-worktree 使用
```

不能再次创建：

```bash
git worktree add ../another-main main
```

---

## 4. Worktree 与 Git Push

Worktree 中常见情况：

本地 branch：

```text
feature-worktree
```

远程 branch：

```text
origin/feature
```

两者名称可以不同。

查看 branch 与 upstream 关系：

```bash
git branch -vv
```

如果本地 branch 名称和远程 branch 名称不同：

不要直接依赖：

```bash
git push
```

推荐显式指定目标：

```bash
git push <remote> HEAD:<remote-branch>
```

例如：

```bash
git push origin HEAD:feature
```

含义：

```text
当前 worktree HEAD
        |
        v
远程 feature branch
```

这样可以避免 worktree 中本地 branch 与远程 branch 名称不一致导致的 push 错误。

---

## 5. 常用命令速查

| 功能 | 命令 |
| --- | --- |
| 查看所有 worktree | `git worktree list` |
| 创建 worktree | `git worktree add PATH BRANCH` |
| 创建新 branch + worktree | `git worktree add PATH -b BRANCH` |
| 删除 worktree | `git worktree remove PATH` |
| 清理失效记录 | `git worktree prune` |
| 查看 branch 关系 | `git branch -vv` |
| 推送指定远程 branch | `git push REMOTE HEAD:BRANCH` |

---

## 6. 常见使用场景

```text
project/
├── main-worktree          # 稳定版本
├── feature-worktree       # 功能开发
└── experiment-worktree    # 实验测试
```

适用于：

- 大型代码库
- 多分支并行开发
- 长周期实验
- 需要保持主分支稳定的开发流程

