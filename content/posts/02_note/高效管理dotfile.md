---
title: "高效管理dotfile"
tags:
  - config
  - dotfile
  - linux
publish: yes
date: 2025-06-09_23:15
---

dotfile 是什么？
- one specifical file folder, which allocate your global dotfile like `.gitconfig`, `.vim` and `.bashrc` .etc.
- 可以上载到仓库，便于在多设备情形下同步软件配置


迁移到 dotfile 后，系统如何感知变化？
- manual方法，使用软链接绑定标识符
```
ln -sf /abs_path/.../target_link symbol_name
```
>note:  target link please use absolute path.

如何自动化这一过程？ we do not want to do it one by one
- use symlink farm manager "Stow"
- and create one simple script

Reference: https://htoopyaelwin.medium.com/organizing-your-dotfiles-e059090a4bf5

2025-03-24_16:47
回归，要将dotfile及时的更新

