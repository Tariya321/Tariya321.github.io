---
title: "Verilator 核心笔记与快速上手指南"
date: 2026-06-05
tags:
  - simulator
  - IC验证
publish: yes
---
## 1. 什么是 Verilator？

**Verilator** 是一个开源的、学术界和工业界广泛使用的 **SystemVerilog 仿真器**。

与传统的事件驱动（Event-driven）仿真器（如 VCS、ModelSim、Questasim）不同，Verilator 采用的是**周期精确（Cycle-accurate）**的编译型仿真机制。它将 Verilog/SystemVerilog 代码**翻译（Verilate）为 C++ 或 SystemC 模型**，然后再通过 C++ 编译器（如 GCC/Clang）编译成一个可执行文件来运行仿真。

### 1.1. Verilator 与传统仿真器的对比

| 对比维度 | Verilator | 传统仿真器 (如 VCS/ModelSim) |
| --- | --- | --- |
| **仿真机制** | 周期精确 (Cycle-accurate)，仅在时钟沿更新状态 | 事件驱动 (Event-driven)，支持任意时间步和延迟 |
| **仿真速度** | 极快 (通常比传统仿真器快 10x - 100x) | 较慢 |
| **测试平台语言** | C++ / SystemC (也可以通过 Cocotb 支持 Python) | SystemVerilog (UVM) / Verilog |
| **时序仿真 (SDF)** | 不支持 | 支持 |
| **代码检查 (Lint)** | 极其严格 (编译时强制进行静态代码检查) | 较宽松 |
| **授权与费用** | 开源免费 (LGPL v3) | 昂贵的商业授权 |

---

## 2. 核心优势与局限性

### 2.1. 优势
- **极致的速度**：由于忽略了门级延迟和毛刺（Glitches），只在时钟沿进行状态求值，因此在大规模数字设计（如 CPU、SoC）的验证中速度极快。
- **无缝对接 C++ 生态**：测试平台（Testbench）直接用 C++ 编写。这意味着你可以轻松地将硬件设计与 C-Model、软件算法库、SystemC 虚拟平台、甚至 OpenCV 等第三方库连接。
- **多线程支持**：支持将设计自动分区并利用多核 CPU 进行并行仿真（通过 `--threads` 参数）。

### 2.2. 局限性
- **不支持非合成语法**：不支持在设计代码中使用 `#10` 这种时间延迟，也不支持大部分用于测试平台的非合成（Non-synthesizable）SystemVerilog 语法。
- **无时序仿真**：无法用于后仿真（Post-layout timing simulation）。
- **不支持完整的 UVM**：由于不支持完整的 SystemVerilog Assertion (SVA) 和受约束的随机化（Constrained Randomization），无法直接运行标准的 UVM 验证环境。

---

## 3. 典型工作流程 (Workflow)

使用 Verilator 进行仿真的标准流程如下：

```
[ 硬件设计 (.v/.sv) ] + [ C++ 测试平台 (sim_main.cpp) ]
                       │
                       ▼ (运行 Verilator 编译)
         [ 自动生成的 C++ 类 (Vtop.h/cpp) ]
                       │
                       ▼ (运行 GCC/Clang 编译)
            [ 可执行仿真程序 (Vtop) ]
                       │
                       ▼ (运行程序)
         [ 仿真结果 / 波形文件 (.vcd/.fst) ]
```

---

## 4. 极简上手示例

下面是一个 4 位计数器的完整仿真示例。

### 4.1. 步骤 1：编写设计代码 `counter.v`
```verilog
module counter (
    input clk,
    input rst_n,
    output reg [3:0] count
);
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n)
            count <= 4'b0;
        else
            count <= count + 1'b1;
    end
endmodule
```

### 4.2. 步骤 2：编写 C++ 测试平台 `sim_main.cpp`
```cpp
#include "Vcounter.h"      // Verilator 根据 counter.v 自动生成的头文件
#include "verilated.h"
#include <iostream>

int main(int argc, char** argv) {
    // 初始化 Verilator 环境
    Verilated::commandArgs(argc, argv);
    
    // 实例化我们的设计顶层模块
    Vcounter* top = new Vcounter;

    // 初始化输入信号
    top->rst_n = 0;
    top->clk = 0;
    top->eval(); // 评估模型状态

    // 仿真循环 (运行 20 个半时钟周期)
    for (int cycle = 0; cycle < 20; cycle++) {
        top->clk = !top->clk; // 翻转时钟
        
        if (cycle == 2) {
            top->rst_n = 1;   // 在第 2 个半周期撤销复位
        }
        
        top->eval();          // 评估模型

        // 打印时钟上升沿时的计数器值
        if (top->clk) {
            std::cout << "Cycle: " << cycle/2 
                      << " | rst_n: " << (int)top->rst_n 
                      << " | count: " << (int)top->count << std::endl;
        }
    }

    // 清理内存
    delete top;
    return 0;
}
```

### 4.3. 步骤 3：一键编译与运行
在终端中运行以下命令：
```bash
# --cc: 生成 C++ 代码
# --exe: 生成可执行文件
# --build: 自动调用 make 进行编译
# -j 4: 使用 4 个 CPU 核心并行编译
verilator --cc --exe --build -j 4 counter.v sim_main.cpp
```
编译完成后，运行生成的程序：
```bash
./obj_dir/Vcounter
```

---

## 5. 进阶技巧与常用命令参数

### 5.1. 导出波形 (Waveform Tracing)
如果需要查看波形（如使用 GTKWave 打开），需要启用波形追踪功能：

- **编译参数**：在 `verilator` 命令中加入 `--trace`。
- **C++ 代码修改**：
  ```cpp
  #include "verilated_vcd_c.h" // 引入波形库
  
  // 在 main 函数中：
  Verilated::traceEverOn(true);
  VerilatedVcdC* tfp = new VerilatedVcdC;
  top->trace(tfp, 99); // 追踪深度
  tfp->open("wave.vcd"); // 保存的文件名
  
  // 在仿真循环中：
  top->eval();
  tfp->dump(contextp->time()); // 写入波形
  
  // 仿真结束时：
  tfp->close();
  ```

### 5.2. 常用命令行参数速查
- `-Wall`：开启所有代码风格和潜在 Bug 警告（强烈推荐）。
- `-Wno-fatal`：即使有警告也不阻止编译（默认情况下警告会被当做错误处理）。
- `--threads N`：开启多线程仿真，`N` 为线程数。
- `-I<dir>`：指定 Verilog 头文件或包含文件的搜索路径。
- `--Mdir <dir>`：指定输出的 C++ 代码目录（默认是 `obj_dir`）。