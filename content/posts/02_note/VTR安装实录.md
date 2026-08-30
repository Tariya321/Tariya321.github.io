---
date: 2025-09-23
tags:
  - EDA
publish: yes
---
# VTR安装实录

ee216-可重构计算的第一个 HW，记录一下安装 VTR 工具的过程

## 1. install and config
下载 VTR资源
```
git clone git@github.com:verilog-to-routing/vtr-verilog-to-routing.git
git submodule init
git submodule update
```

VTR requires several system and Python packages to build and run the flow.

system pkg for Debian-based OS (*need sudo*)
```
./install_apt_packages.sh
```
> I'm using Ubuntu22.2

python pkg
```
conda create --name vtr
conda activate vtr
conda install python=3.11
pip install -r requirements.txt
```

before building VTR tool, make sure you are using the right cmake version
```
which cmake
```
then use cmd:
```
make
```
while adding `-j <n>` option can enable the muilt-thread compilation
{{< figure src="/attachment/VTR%E5%AE%89%E8%A3%85%E5%AE%9E%E5%BD%95-1.png" alt="VTR安装实录-1" width="600" >}}

## 2. running VTR flow
```
export VTR_FLOW_DIR=~/zone/vtr_work/quickstart/blink_run_flow
export VTR_ROOT=~/zone/vtr-verilog-to-routing
mkdir -p $VTR_FLOW_DIR
cd $VTR_FLOW_DIR
```
an error occured
```
ModuleNotFoundError: No module named 'prettytable'
```
you can use pip to install it
```
pip3 install prettytable
```

then run by
```
$VTR_ROOT/vtr_flow/scripts/run_vtr_flow.py \
    $VTR_ROOT/doc/src/quickstart/blink.v \
    $VTR_ROOT/vtr_flow/arch/timing/EArch.xml \
    --route_chan_width 100
```
result
{{< figure src="/attachment/VTR%E5%AE%89%E8%A3%85%E5%AE%9E%E5%BD%95.png" alt="VTR安装实录" width="600" >}}

## 3. manually run
firstly
```
export VTR_FLOW_DIR=~/zone/vtr_work/quickstart/vpr_tseng
export VTR_ROOT=~/zone/vtr-verilog-to-routing
mkdir -p $VTR_FLOW_DIR
cd $VTR_FLOW_DIR
```
run by
```
$VTR_ROOT/vpr/vpr \
    $VTR_ROOT/vtr_flow/arch/timing/EArch.xml \
    $VTR_ROOT/vtr_flow/benchmarks/blif/tseng.blif \
    --route_chan_width 100
```
result
{{< figure src="/attachment/VTR%E5%AE%89%E8%A3%85%E5%AE%9E%E5%BD%95-2.png" alt="VTR安装实录-2" width="600" >}}

run visualized implementation
```
$VTR_ROOT/vpr/vpr \
    $VTR_ROOT/vtr_flow/arch/timing/EArch.xml \
    $VTR_ROOT/vtr_flow/benchmarks/blif/tseng.blif \
    --route_chan_width 100 \
    --analysis --disp on
```
result
{{< figure src="/attachment/VTR%E5%AE%89%E8%A3%85%E5%AE%9E%E5%BD%95-3.png" alt="VTR安装实录-3" width="580" >}}

