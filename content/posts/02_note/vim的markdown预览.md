---
tags:
  - vim
  - plugin
publish: yes
---
# vim的markdown预览

## 1. Linux

[使用 Vim 开始 Markdown 之旅 - 点半九 (dianbanjiu.com)](https://www.dianbanjiu.com/post/%E4%BD%BF%E7%94%A8-vim-%E5%BC%80%E5%A7%8B-markdown-%E4%B9%8B%E6%97%85/)
下载 vim 插件下载器 `vim-plug`
```shell
curl -fLo ~/.vim/autoload/plug.vim --create-dirs  https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```
关于 `vim-plug`，可见 [junegunn/vim-plug: :hibiscus: Minimalist Vim Plugin Manager (github.com)](https://github.com/junegunn/vim-plug)
编辑 `~/.vimrc` 文件
```shell
call plug#begin('~/.vim/plugged') 
	Plug 'iamcco/markdown-preview.nvim', { 'do': 'cd app & yarn install' } 
call plug#end()
```
没有 nodejs 则
```
Plug 'iamcco/markdown-preview.nvim', { 'do': { -> mkdp#util#install() }, 'for': ['markdown', 'vim-plug']}
```

安装插件
```
:PlugInstall
:call mkdp#util#install()
```
报错
```
fatal: unable to access 'https://github.com/iamcco/markdown-preview.nvim.git/': Empty reply from server
```
检索可知是'墙'的原因，重试后仍然报错（作罢）

2025-03-05_10:51
更新，对于无法在线安装的设备，可以进行离线安装
即将文件`plug.vim`下载后，放置到`~/.vim/autoload/`文件夹下
对于vim-plug无法下载插件的情况，可以参考文章‘加速vim-plug安装插件的下载’


在 markdown文件时，使用
```
:MarkdownPreview
```
来开启预览
### 1.1. MarkdownPreview is not an editor command
安装完成后，输入预览指令，报错如题
解决办法：`:set filetype=markdown`

Reference
[MarkdownPreview is not an editor command.不是编辑器命令，但是自动预览可以用。 · Issue #414 · iamcco/markdown-preview.nvim (github.com)](https://github.com/iamcco/markdown-preview.nvim/issues/414)
[vi/vim使用进阶: 开启文件类型检测 - 易水博客 (easwy.com)](https://blog.easwy.com/archives/advanced-vim-skills-filetype-on/)

## 2. Toc生成
写 markdown 怎么能没有预览和标题呢？
已经解决了预览，下面必须整一个标题出来

下载插件 `vim-markdown-toc`
使用 command,生成标题
```
:GenTocGFM
```
>标题也分为多种格式，参见 link 的介绍

Reference
https://github.com/mzlogin/vim-markdown-toc

## 3. Win

下载 vim-plug
```
iwr -useb https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim |` ni $HOME/vimfiles/autoload/plug.vim -Force
```
{{< figure src="/attachment/2024-08-03.png" alt="2024-08-03" width="400" >}}

下面与 linux 下的为相同操作
{{< figure src="/attachment/2024-08-03-1.png" alt="2024-08-03-1" width="400" >}}

在初次打开一个 markdown 文件时，输入指令
```
:call mkdp#util#install()
```
否则很可能在开启预览时报错
```
pre build and node is not found
```
>https://github.com/iamcco/markdown-preview.nvim/issues/7
>作者称“可能是因为网络问题未自动下载”

成功
{{< figure src="/attachment/vim%E7%9A%84markdown%E9%A2%84%E8%A7%88.png" alt="vim的markdown预览" width="500" >}}
