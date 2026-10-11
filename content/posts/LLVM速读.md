---
title: "LLVM速读"
date: 2026-10-01
tags:
  - IR
  - backend
publish: true
---
> [!summary] 一句话理解
> LLVM 是一套模块化、可复用的编译器基础设施：前端负责理解源语言，LLVM IR 负责承载和优化程序，后端负责生成目标硬件代码。

**LLVM**（原名 *Low Level Virtual Machine*）是一个**模块化、可复用的编译器和工具链技术集合**。

它是 Rust、Swift、Clang/C++、Julia 等现代语言背后的“**编译引擎底座**”。

## 1. LLVM 解决什么问题？

在 LLVM 出现之前，传统编译器通常将某门语言直接编译为特定 CPU 的机器码。

如果有 $M$ 门编程语言（C、Rust、Swift 等），要支持 $N$ 种硬件架构（x86、ARM、RISC-V 等），就需要编写 $M \times N$ 个编译器，开发和维护成本很高。

LLVM 引入了**三段式架构（Three-Phase Architecture）**：

{{< figure src="/attachment/LLVM%E9%80%9F%E8%AF%BB.png" alt="LLVM速读" width="625" >}}
- interesting! 对于只写高层次算子的开发者而言，LLVM 是它们的后端
- 核心优势是，新写一门语言只需开发前端，想支持新硬件只需开发后端，从而将整体工作量从 $M \times N$ 降到了约 $M + N$


## 2. 编译流程：从源代码到机器码

![LLVM-IR-flow-gemini-gen](/attachment/LLVM-IR-flow-gemini-gen.jpeg)

| 阶段                      | 主要职责                       | 示例               |
| ----------------------- | -------------------------- | ---------------- |
| **前端（Frontend）**        | 解析源语言、进行语法和语义检查，生成 LLVM IR | Clang、Rustc      |
| **优化器（Optimizer/Core）** | 对 IR 进行与语言和硬件无关的优化         | 死代码消除、循环展开、常量折叠  |
| **后端（Backend）**         | 将优化后的 IR 转换为目标平台的汇编或机器码    | x86、ARM64、RISC-V |
|                         |                            |                  |

> [!tip] 关键点
> 前端和后端通过 LLVM IR 解耦。前端不需要了解每种硬件，后端也不需要理解每种源语言。

## 3. 核心概念：LLVM IR

LLVM 的灵魂在于 **LLVM IR（Intermediate Representation，中间表示）**。它可以看作一种面向编译器的通用汇编语言，具有以下特点：

- **强类型：** 指令和数据类型明确，便于检查和优化。
- **静态单赋值（SSA）：** 每个变量只赋值一次，便于分析数据流。
- **硬件相对无关：** 无论上层是 C、Swift 还是 Julia，都可以转换为统一的 IR 结构。
- **适合分层优化：** 自动向量化、函数内联等高级优化大多在 IR 层完成。



## 4. LLVM 生态

![LLVM速读-1](/attachment/LLVM%E9%80%9F%E8%AF%BB-1.png)

### 4.1. 编译器与语言

- **Clang：** 基于 LLVM 的高性能 C/C++/Objective-C 编译器，也是 GCC 的强力竞争对手。
- **Rust、Swift、Julia、Kotlin/Native：** 这些现代语言的官方编译器均使用 LLVM 作为底层后端。

### 4.2. 工具链与扩展项目

- **MLIR（Multi-Level IR）：** LLVM 社区衍生出的多层次 IR 项目，广泛用于机器学习和 AI 编译器，例如 TensorFlow、PyTorch 的硬件加速。
- **LLD：** 基于 LLVM 的高速链接器（Linker）。
- **LLDB：** 基于 LLVM 的调试器（Debugger）。

## 5. LLVM 的核心优势

- **模块化与库化：** LLVM 由一组可调用的 C++ API 和库组成，可以嵌入自己的工程，例如实现 JIT（即时编译）。
- **强大的代码优化能力：** 汇集了 Apple、Google、Intel、AMD、Nvidia 等厂商和开发者共同维护的优化算法。
- **跨平台与硬件无关性：** 同一套前端和优化流程可以面向多种硬件平台。

## 6. 记忆框架

```text
源语言
  ↓
前端（解析与检查）
  ↓
LLVM IR（统一表示）
  ↓
优化器（改进程序）
  ↓
后端（适配硬件）
  ↓
汇编 / 机器码
```

**核心关系：** LLVM 不只是一个编译器，而是一套围绕 LLVM IR 构建的编译器基础设施和工具链生态。
