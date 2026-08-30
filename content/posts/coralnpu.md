---
date: 2026-08-15
tags:
  - npu
  - google
publish: yes
---
# coralnpu

- coral-npu，google研究团队开源的软硬件方案
- CoralNPU Nexus FPGA emulation boards (e.g., Board 09, Board 10)
- 运行于edge端，例如手机、手表等电子设备中
{{< figure src="/attachment/ai-accelerator%20for%20inference.png" alt="ai-accelerator for inference" width="346" >}}
![345](https://pic4.zhimg.com/v2-9e3e1552315645b7e5785f026512369d_r.jpg)
https://github.com/google-coral/coralnpu


## 1. 项目组件


标量核、向量执行单元、矩阵运算单元
- 标量核：负责程序控制流、分支判断和系统级调度的通用处理核心。
- 向量执行单元：通过 SIMD 方式对多数据元素进行并行处理的计算单元。
- 矩阵运算单元：基于 MAC 阵列对矩阵/张量乘加运算进行高并行加速的专用计算单元。


{{< figure src="/attachment/PixPin_2026-05-25_22-38-36.png" alt="PixPin_2026-05-25_22-38-36" width="714" >}}

soc 描述组件
![PixPin_2026-05-26_13-37-53](/attachment/PixPin_2026-05-26_13-37-53.jpg)
仔细看 sv 的代码，其实就是把核心子系统和一堆外设连接起来了

hdl 目录中分为 chisel 和 verilog 下的硬件代码，chisel 代码是核心
verilog 下的硬件代码的作用
{{< figure src="/attachment/PixPin_2026-05-26_14-51-44.jpg" alt="PixPin_2026-05-26_14-51-44" width="611" >}}

综上，一个基本硬件和代码对应的结构图为：
{{< figure src="/attachment/PixPin_2026-05-26_14-58-00.jpg" alt="PixPin_2026-05-26_14-58-00" width="601" >}}

**三个核心处理单元：**

| 单元 | 文件 | 功能 |
|------|------|------|
| **Scalar Core** | `scalar/SCore.scala` | 控制流 + 基本 ALU |
| **Vector Core** | `rvv/RvvCore.scala` | 向量/SIMD 操作 (256 位) |
| **Matrix Unit** | `scalar/Mlu.scala` | 乘法累加 (256 MACs/cycle) |


关于用到的总线
{{< figure src="/attachment/PixPin_2026-05-26_15-17-25.jpg" alt="PixPin_2026-05-26_15-17-25" width="636" >}}

### 1.1. compiler 路径


提供了两种将模型部署到 npu 上的方法
- MLIR/IREE
	- 适合精细化定制
- LiteRT (原 TFLite)
	- 比较死板的格式转换器
	- 首先得把模型转成 TensorFlow 格式
	- 硬件算子需要匹配，否则回退到 cpu 计算
{{< figure src="/attachment/ai-accelerator%20for%20inference-9.png" alt="ai-accelerator for inference-9" width="557" >}}

MLIR/IREE 的路径示意图
{{< figure src="/attachment/ai-accelerator%20for%20inference-10.png" alt="ai-accelerator for inference-10" width="786" >}}


LiteRT 的路径
{{< figure src="/attachment/ai-accelerator%20for%20inference-11.png" alt="ai-accelerator for inference-11" width="792" >}}



### 1.2. hardware design hierarchy demenstration

https://deepwiki.com/google-coral/coralnpu/7-fpga-implementation
![ai-accelerator for inference-12](/attachment/ai-accelerator%20for%20inference-12.png)

```mermaid
graph TD
    %% 样式定义
    classDef config fill:#FFF2CC,stroke:#D6B656,stroke-width:2px;
    classDef chisel fill:#DAE8FC,stroke:#6C8EBF,stroke-width:2px;
    classDef verilog fill:#D5E8D4,stroke:#82B366,stroke-width:2px;
    classDef generated fill:#F8CECC,stroke:#B85450,stroke-width:2px,stroke-dasharray: 5 5;
    classDef fpga fill:#E1D5E7,stroke:#9673A6,stroke-width:2px;

    %% 层次 1
    subgraph L1 [层次 1：配置文件 Parameters.scala]
        P[参数中心: enableRvv, itcmSize, dtcmSize...]:::config
    end

    %% 层次 2
    subgraph L2 [层次 2：Chisel Scala源代码]
        CS[Chisel DSL 源码: Core.scala, SCore.scala, Alu.scala, RvvCore.scala...]:::chisel
    end

    %% 层次 3
    subgraph L3 [层次 3：手写性能优化 Verilog]
        WV[手写关键路径优化: RstSync.sv, fifo_flopped.sv, rvv_backend_alu_unit.sv...]:::verilog
    end

    %% 层次 5
    subgraph L5 [层次 5：处理器核心 生成 ]
        GenCore[自动生成 Core.sv<br>100-150k 行<br>4 lane RISC-V + RVV]:::generated
    end

    %% 层次 4
    subgraph L4 [层次 4：FPGA IP 核]
        IP[外设 IP: ispyocto, ddr4_stub, i2c_master, spi_dpi_master...]:::verilog
    end

    %% 层次 6
    subgraph L6 [层次 6：完整 SoC 生成 ]
        SoC_Scala[SoC 集成描述: CoralNPUChiselSubsystem.scala]:::chisel
        GenSoC[自动生成 CoralNPUChiselSubsystem.sv<br>200-300k 行<br>Core + 外设 + 总线仲裁 + ISP]:::generated
    end

    %% 层次 7
    subgraph L7 [层次 7：FPGA 板级集成]
        FPGA[板级顶层包装: chip_nexus.sv, coralnpu_soc.sv, clkgen_wrapper.sv]:::fpga
    end

    %% 关系连接
    P -->|控制硬件配置| CS
    P -->|控制SoC配置| SoC_Scala
    CS -->|1.Bazel 编译| GenCore
    WV -->|2.直接例化/调用| GenCore
    GenCore -->|集成到总线| SoC_Scala
    SoC_Scala -->|Chisel 编译| GenSoC
    GenSoC -->|作为核心总线例化| FPGA
    IP -->|作为外设 IP 连线| FPGA

    %% 阶段标注
    linkStyle 0,1 stroke:#D6B656,stroke-width:2px;
    linkStyle 2,5 stroke:#6C8EBF,stroke-width:2px;
    linkStyle 3 stroke:#82B366,stroke-width:2px;

```


### 1.3. sw2hw co-work design flow
```mermaid
flowchart TD
  %% ==================== 样式定义 ====================
  classDef swStyle fill:#EBF3FF,stroke:#3178C6,stroke-width:2px,color:#1E3A8A;
  classDef simStyle fill:#FFFBEB,stroke:#D97706,stroke-width:2px,color:#78350F;
  classDef hwStyle fill:#F0FDF4,stroke:#16A34A,stroke-width:2px,color:#14532D;
  classDef extStyle fill:#FAFAFA,stroke:#737373,stroke-width:1px,color:#404040,stroke-dasharray: 3 3;

  %% ==================== 软件层 ====================
  subgraph SW[软件层]
    direction TB
    SRC["examples/*.cc<br>(应用代码)"] --> BLD["coralnpu_v2_binary<br>(工具链 + linker script)"]
    BLD --> ELF["ELF 可执行文件"]
    BLD --> BIN["BIN / VMEM"]
  end
  class SW,SRC,BLD,ELF,BIN swStyle;

  %% ==================== 加载与仿真层 ====================
  subgraph SIM[加载与仿真层]
    direction TB
    SIMRUN["core_mini_axi_sim.cc<br>(SystemC + Verilator)"] --> TB["CoreMiniAxi_tb"]
    TB --> LOAD["LoadElfSync / Reset<br>/ ClockGate"]
    LOAD --> EXEC["运行程序 / 产生访存事务"]
  end
  class SIM,SIMRUN,TB,LOAD,EXEC simStyle;

  %% ==================== 硬件RTL层 ====================
  subgraph HW[硬件RTL层]
    direction TB
    
    %% 硬件生成与顶层
    GEN["Chisel EmitCore"] --> SV["CoreMiniAxi.sv<br>/ RvvCoreMiniAxi.sv"]
    SV --> COREAXI["CoreAxi (最外层封装)"]
    
    %% 核心架构垂直解耦，降低宽度
    COREAXI --> AXIM["AXI master / slave"]
    COREAXI --> CORE["Core"]
    
    %% Core 内部并排或下沉
    CORE --> SCORE["SCore (标量核心)"]
    CORE -. 可选 .-> RVV["RvvCore (向量核心)"]
    
    %% 流水线与内存
    SCORE --> PIPE["流水线<br>(Fetch/Dispatch/LSU/CSR...)"]
    PIPE --> MEM["内部内存<br>(ITCM / DTCM / CSR Fabric)"]
  end
  class HW,GEN,SV,COREAXI,CORE,SCORE,RVV,PIPE,MEM,AXIM hwStyle;

  %% ==================== 外部组件 ====================
  EXT["外部内存 / 外设"]
  class EXT extStyle;

  %% ==================== 跨层数据流 ====================
  ELF -->|加载| LOAD
  EXEC -->|指令/数据流| MEM
  EXEC -->|总线事务| AXIM
  AXIM --> EXT
```

### 1.4. rvv 设计结构

```mermaid
flowchart LR
  subgraph Chisel["Chisel/Scala HDL"]
    C1["hdl/chisel/src/coralnpu/rvv\nRvvCore.scala, RvvDecode.scala, RvvAlu.scala"]
    C2["hdl/chisel/src/common\nAligner.scala"]
  end

  subgraph SV["SystemVerilog HDL"]
    V1["hdl/verilog/rvv/design\nRvvCore.sv, rvv_backend_*.sv"]
    V2["hdl/verilog/rvv/inc\n*.svh headers"]
    V3["hdl/verilog/\nClockGate.sv, RstSync.sv, SRAMs"]
  end

  C1 -->|Bazel emit_verilog| G["Generated SystemVerilog"]
  C2 -->|resources: Aligner.sv| V1
  V1 --> F["Final RTL build / simulation"]
  V2 --> F
  V3 --> F
  G --> F
```


### 1.5. MAC 阵列

- 外积 MAC 使用 8bit×8bit → 32bit 累加，256 MAC/周期
- zvt 矩阵计算单元，含有 PE 和 systolic array



## 2. 运行实例

### 2.1. quick start


- bazel 7.4.1
>  please upgrade to 8.6.0

```bash
# Ensure that test suite passes
bazel run //tests/cocotb:core_mini_axi_sim_cocotb

# Build a binary
bazel build //examples:coralnpu_v2_hello_world_add_floats

# Build the Simulator (non-RVV for shorter build time):
bazel build //tests/verilator_sim:core_mini_axi_sim

# Run the binary on the simulator:
bazel-bin/tests/verilator_sim/core_mini_axi_sim --binary bazel-out/k8-fastbuild-ST-dd8dc713f32d/bin/examples/coralnpu_v2_hello_world_add_floats.elf
```

test: 构建并运行一个 **cocotb + Verilator 的 RTL 仿真测试**
把 `CoreMiniAxi` 这个硬件顶层编译成一个可执行的 verilator仿真模型，然后使用 python cocotb 作为驱动
run result
{{< figure src="/attachment/PixPin_2026-05-22_20-22-44.png" alt="PixPin_2026-05-22_20-22-44" width="806" >}}




### 2.2. run mobileNet

- 由`BUILD`文件控制行为、工具链
	- 将TFLite 模型调用工具进行编译，产生mobilenet的binary data
- 脚本 **`run_full_mobilenet_v1.cc`** 编译产生 `.elf` 文件，在 coralNPU simulator 上作为可执行程序
	- 该脚本 link 了 生成的 model binary数据文件 `mobilenet_v1_025_224_int8_dummy.h`
- 脚本 **`npusim_run_mobilenet.py`** 作为 simulator 的控制器，会加载 elf 文件，作为中介传入 simulator

> [!cite] What is ELF file?
ELF (Executable and Linkable Format)，一种封装机器码的格式


一键运行指令
```
bazel run tests/npusim_examples:npusim_run_mobilenet
```
BUILD 文件解析
{{< figure src="/attachment/ai-accelerator%20for%20inference-2.png" alt="ai-accelerator for inference-2" width="713" >}}

运行结果
{{< figure src="/attachment/ai-accelerator%20for%20inference-3.png" alt="ai-accelerator for inference-3" width="624" >}}

这里值得注意的是显示 "fallback kernel"，表示没有 offload 到 npu
这是因为走到的 **Conv2D 算子形状没有命中这个优化内核支持的条件，所以代码主动退回到了 reference kernel**。
而 `conv.cc` 针对优化的是 4x4 (height x width)的卷积核，
{{< figure src="/attachment/ai-accelerator%20for%20inference-4.png" alt="ai-accelerator for inference-4" width="601" >}}

将 python 中的 `output_data` 个数修改为更大的例如 20，100，将可以看到其他 case
{{< figure src="/attachment/ai-accelerator%20for%20inference-5.png" alt="ai-accelerator for inference-5" width="615" >}}


refs:
- 如何使用pytorch
PyTorch和TensorFlow之对比




