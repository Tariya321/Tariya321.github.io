---
title: "conda的安装与配置指南"
date: 2025-03-21_18:33
tags:
  - conda
  - linux
  - 指南
publish: yes
---
conda官方下载站点（发送链接到邮件）
https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html
官方指南
https://docs.conda.org.cn/projects/conda/en/stable/user-guide/index.html
安装指南
https://arabelatso.github.io/2020/01/09/Install%20Anaconda%20on%20CentOS%20without%20root/

## 1. install and config

### 1.1. conda 安装和激活

Anaconda
```shell
mkdir -p ~/anaconda3
wget https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh -O ~/anaconda3/anaconda.sh
bash ~/anaconda3/anaconda.sh -b -u -p ~/anaconda3
rm ~/anaconda3/anaconda.sh
```

Miniconda
[Installing Miniconda - Anaconda](https://www.anaconda.com/docs/getting-started/miniconda/install#quickstart-install-instructions)
```shell
mkdir -p ~/miniconda3 
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh 
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3 
rm ~/miniconda3/miniconda.sh
```

activate the conda env by running: 
```
conda init
conda activate
```
this will activate the default  `base` conda env

I think it could be burden for using shell, so i'll
```
conda config --set auto_activate_base false
```
this cmd will let conda un-open default start conda env.

### 1.2. miniforge 安装
[conda-forge/miniforge: A conda-forge distribution.](https://github.com/conda-forge/miniforge)
```
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
echo "source ~/miniforge3/etc/profile.d/conda.sh" >> ~/.bashrc
source ~/.bashrc
conda activate
```

### 1.3. 配置镜像资源站

for conda
编辑 `~/.condarc` 
- 修改为清华镜像源（260812_19:54出于存储压力，不再提供支持）
```
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```
- 修改为浙大镜像源
```
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.zju.edu.cn/anaconda/pkgs/main
  - https://mirrors.zju.edu.cn/anaconda/pkgs/r
  - https://mirrors.zju.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.zju.edu.cn/anaconda/cloud
  msys2: https://mirrors.zju.edu.cn/anaconda/cloud
  bioconda: https://mirrors.zju.edu.cn/anaconda/cloud
  menpo: https://mirrors.zju.edu.cn/anaconda/cloud
  pytorch: https://mirrors.zju.edu.cn/anaconda/cloud
  pytorch-lts: https://mirrors.zju.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.zju.edu.cn/anaconda/cloud
  nvidia: https://mirrors.zju.edu.cn/anaconda-r
```
> 或者其他请参考[网页](https://mdnice.com/writing/ee59139ae3544ea498f6b80ad06b8499)


使配置文件生效
```shell
conda clean -i  # 清除索引缓存
conda update conda
```


for pip
```
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### 1.4. conda 使用
verify for conda path
```shell
conda info
```
{{< figure src="/attachment/server%E6%97%A0sudo%E9%85%8D%E7%BD%AEconda.png" alt="server无sudo配置conda" width="400" >}}

create a py3.11 conda env (3.11 is the most stable version)
```shell
# install python 6.11 only for your env
conda create --name dev python=3.11
# activate
conda activate dev
# verify
python --version
```
or install python 6.11 for conda base(not recommend)
```shell
conda install python=3.11
```
make 3.11 as default version for conda python
```shell
conda config --set default_python 3.11
```
if you want to deactivate
```shell
conda deactivate
```
list all env
```shell
conda env list
```
> please deactivate your venv use `deactivate` at the same time


install conda python library
```
conda install <pkg_name>
```

### 1.5. 常用包

```
conda install  numpy pandas jupyter ipykernel matplotlib tqdm pandas
```

## 2. 进阶-环境复制和再部署

一键创建环境并安装指定pkgs
关键在于一个`yml`格式的文件
导出yml的方式
```
conda env export > environment.yml
```
导入yml文件的方式
```shell
conda env create -f xxx.yml
```

当只是想要安装一些包，而不去管理环境时
```shell
pip install -r requirements.txt
```

使用 conda 管理环境
使用 pip 下载包，因为这时候的 pip 是 conda 环境下的，而不是系统环境

