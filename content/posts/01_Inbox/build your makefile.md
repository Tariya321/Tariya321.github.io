---
title: "build your makefile"
date: 2026-04-01
tags:
  - linux
  - cmd
publish: yes
---
## 1. path

如何在 makefile 中调用当前 PATH？
```
SHELL := /bin/bash -lc
```
其中 `-l` 表示使用 login shell


## 2. target

例如 `clean` 就是个 target
```
make clean
```
- 单个 target 占用一个 shell 环境
- target 之间占用不同的 shell 环境

target 可以存在依赖
例如下面的 target setup 就需要先去 clean
```
setup: clean
	source xxx.sh
```

