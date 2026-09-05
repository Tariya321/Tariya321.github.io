---
title: "强大的rsync文件备份工具"
date: 2025-06-14_21:23
tags:
  - linux
  - rsync
  - 文件备份
publish: yes
---
rsync的强大之处
- 保留了文件信息
- 拥有关系不变
- 增量更新

option 解读
- -a (archive)，等价于 `-r`，递归拷贝目录，且保留权限、时间、符号
- -z (compress)，传输中压缩数据，对于文本类传输非常有效
- --dry-run，仅演示，不会真正执行
- -e ssh, 使用 ssh 传输通道
- --progress，显示进度

https://www.ruanyifeng.com/blog/2020/08/rsync.html
默认情况下，rsync 只确保源目录的所有内容（明确排除的文件除外）都复制到目标目录。它不会使两个目录保持相同，并且不会删除文件。如果要使得目标目录成为源目录的镜像副本，则必须使用`--delete`参数，这将删除只存在于目标目录、不存在于源目录的文件。
```shell
rsync -av --delete <source> <destination>
```

Note: 待备份的文件夹，请不要加最后的斜杠

例如我的，在两个目录之间备份传输
```shell
rsync -av --delete ~/sw_data/ob_vaults/ForHigh /Volumes/hot_data/ob_vaults/
```


from server to local
```shell
rsync -avzhe "ssh -p <port>" user@192.168.ip.xx:/path/unbacked_dir /local/dir
# 传输xxx内的文件到yyy目录下
rsync -avz hostname:xxxx /local/yyy
```




