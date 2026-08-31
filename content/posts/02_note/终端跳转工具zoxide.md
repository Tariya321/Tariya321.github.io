---
title: "终端跳转工具zoxide"
date: 2025-11-06
tags:
  - terminal
publish: yes
---
link: [ajeetdsouza/zoxide: A smarter cd command. Supports all major shells.](https://github.com/ajeetdsouza/zoxide)

## 1. install
这个工具是 `fasd` 工具的现代版，具有类似的功能


### 1.1. precompiled-binary

```
wget https://github.com/ajeetdsouza/zoxide/releases/download/v0.9.8/zoxide-0.9.8-x86_64-unknown-linux-musl.tar.gz
```


### 1.2. one-line
for debian based system, use cmd to install
```
curl -sSfL https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | sh
```

## 2. usage
and add this to your bashrc
```
eval "$(zoxide init bash  --cmd)"
```
then try to use `z` instead of `cd`



## 3. debug

如果 logging 时显示
```
Last login: xxx
zoxide: command not found
```
 可以确认一下 zoxide bin 所在目录是否在 path 中
 我这里手动再添加了下
 ```
 export PATH=$HOME/.local/bin:$PATH
 ```

