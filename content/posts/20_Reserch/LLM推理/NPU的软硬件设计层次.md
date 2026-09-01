---
title: "NPU的软硬件设计层次"
date: 2026-06-15
tags:
  - npu
publish: yes
---

编译器将由“算子”组合而成的高层 Task 图结构，转化、降维并调度成了由底层硬件“ISA 指令”构成的执行程序

|层次|例子|描述|
|---|---|---|
|Graph|ResNet、Transformer 计算图|描述算子之间的数据流与依赖关系|
|Operator|Conv2D、MatMul、ReLU|模型中的功能级计算操作|
|Kernel|conv_kernel、gemm_kernel|Operator 在特定后端上的具体软件实现|
|ISA|load、store、vfmacc、matrix_mul、dma_start|定义机器可执行的指令及其语义|
|Microarchitecture|MAC/PE 阵列、Vector Unit、Register File、SRAM Buffer、DMA、Pipeline|定义 ISA/计算功能在硬件内部如何组织和执行|
|RTL|Verilog/SystemVerilog 中的 MAC array、DMA controller、SRAM interface|微架构的周期精确硬件实现|
|Circuit / Physical|乘法器、加法器、触发器、SRAM bitcell、标准单元、版图|具体晶体管/逻辑门级实现以及布局布线|



- 算子只定义了**数学功能**，不关心怎么实现
- Kernel 本质上是某个算子的具体执行实现（Implementation）
	- 根据平台、实现方式，一个算子可以有很多个 kernel



```mermaid
graph TD
    %% Style Definitions
    classDef highLevel fill:#e1f5fe,stroke:#03a9f4,stroke-width:2px;
    classDef frontend fill:#fff3e0,stroke:#ff9800,stroke-width:2px;
    classDef backend fill:#e8f5e9,stroke:#4caf50,stroke-width:2px;
    classDef hardware fill:#fce4ec,stroke:#e91e63,stroke-width:2px;

    %% Top Level: Task Input
    Task[High-Level Task<br>e.g., Neural Network with Millions of Parameters]:::highLevel

    %% Compiler Frontend
    subgraph Compiler Frontend: Graph-Level Optimization
        Graph[Computation Graph<br>Nodes and Edges]:::frontend
        
        Op1(Operator: GEMM):::frontend
        Op2(Operator: ReLU Activation):::frontend
        Op3(Operator: Softmax Normalization):::frontend
        
        Fusion[Graph Optimization<br>Operator Fusion & Pruning]:::frontend
        Optimized_IR[Optimized High-Level<br>Intermediate Representation - IR]:::frontend
    end

    %% Compiler Backend
    subgraph Compiler Backend: Lowering and Scheduling
        Loops[Structural Conversion:<br>Nested Loops]:::backend
        
        Schedule_Tiling(Loop Tiling<br>Adapting to SRAM Capacity):::backend
        Schedule_Mem(Memory Allocation & Movement<br>DRAM ↔ SRAM ↔ Registers):::backend
        Schedule_Flow(Data Flow Scheduling<br>Timing Data Entry into Processing Arrays):::backend
    end

    %% Hardware Mapping
    subgraph Hardware Mapping and ISA Generation
        Target_ISA{Generate Hardware Instructions<br>ISA / Configuration Codes}:::hardware
        
        CPU_GPU[General Purpose ISA<br>Scalar/Vector Math, Memory R/W]:::hardware
        DSA[Domain-Specific ISA<br>Tensor/Matrix Instructions, Array Control]:::hardware
        CIM[Physical State Configuration<br>CIM Weight Programming, Crossbar Config]:::hardware
    end

    %% Process Flow
    Task -->|Deconstruct & Parse| Graph
    Graph --> Op1
    Graph --> Op2
    Graph --> Op3
    
    Op1 --> Fusion
    Op2 --> Fusion
    Op3 --> Fusion
    
    Fusion --> Optimized_IR
    
    Optimized_IR -->|Strip Math Semantics,<br>Expose Execution Order| Loops
    
    Loops --> Schedule_Tiling
    Loops --> Schedule_Mem
    Loops --> Schedule_Flow
    
    Schedule_Tiling --> Target_ISA
    Schedule_Mem --> Target_ISA
    Schedule_Flow --> Target_ISA
    
    Target_ISA --> CPU_GPU
    Target_ISA --> DSA
    Target_ISA --> CIM
```


在将高层算子编译为底层指令之前，将会在逻辑和数学层面上减少计算量（算子融合）


