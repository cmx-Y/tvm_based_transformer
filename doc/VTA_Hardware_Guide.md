VTA Overview 和前面的引言对我帮助不是很大，他们讲的都是一些很高层次的东西，是一种概览，这些概览我在VTA论文里也见了不少，好处特性等等了解一下就行，我现在想了解的是最细节最具体的东西。

## HLS Hardware Source Organization

- 源码在 /vta-hw/hardware/xilinx/src 下

- vta.cc
- vta.h  主要用到了 ap_int

- 一些宏定义在 /vta-hw/include/vta/hw_spec.h
- 配置参数通过 /vta-hw/config/vta_config.json

### HLS Module Example

这一小节介绍了 fetch 模块的 hls 代码实现中的一些细节。

与函数**参数列表**相关有以下几点：

- 值参数被综合成一个内存映射只读寄存器，由 host 向里面写内容，例如 fetch 模块中的 uint32_t insn_count
- 指针参数

- \+ pragma m_axi : 会综合出 AXI 请求接口，进而提供对 DRAM 的 DMA，例如 fetch 模块中的 insns
- \+ pragma bram : 会综合出 BRAM 接口，进而提供对 FPGA block-RAM 的读写

- hls::stream 引用 + pragma axis 会综合出 FIFO 接口，例如 fetch 模块中的 load_queue, gemm_queue, store_queue

关于函数参数被综合成硬件模块接口的具体细节可以参考 ug902/Ch1.High-Level Synthesis/Managing Interfaces。这里简单补充一点基础知识。

函数参数被综合成 RTL 接口的过程叫做 **interface synthesis**，hls 综合生成的 RTL 接口包括以下三类，其中前两类是所有函数都会默认综合出的：

- 时钟和复位信号接口：ap_clk, ap_rst
- 模块级别协议接口：ap_start, ap_done, ap_ready, ap_idle，用于指示当前模块的运行状态
- 端口级别协议接口：根据函数参数综合出的接口，包括默认的 ap_return

接下来根据自己的理解逐行分析 fetch 模块的 hls 代码：

**参数列表及pragma interface部分：**

- insn_count 被综合成 memory-mapped register，因此在 pragma 中会声明offset。详细分析一下pragma：其中 s_axilite 表明 insn_count 会被综合进 AXI4-Lite 接口；bundle 决定将信号综合进哪个 AXI4-Lite 接口，这里就会综合进 s_axi_CONTROL_BUS；offset 表示对信号进行地址分配。 
- insn_T 在 vta.h 中的定义是 :  typedef ap_uint<VTA_INS_WIDTH> insn_T，实际上就是一个包含很多 bit 的指令格式，每个 insn_T 类型的变量存储一条指令。pragma中的 m_axi 表明 insns 会被综合进 AXI4 Master接口。

**代码主体部分：**

包含一个循环：INSN_DECODE，已经添加 pipeline optimizing directive。每一次循环，首先读取 insns[pc] 取出一条指令，通过类型转换得到 VTAGenericInsn 型变量，存储到局部变量VTAInsn insn的generic域。之后就是解码部分，根据 opcode 判断是哪一种指令，判断完后将其写到对应的队列中。当 opcode 为 VTA_OPCODE_LOAD 时，如果加载的是 input 或 weight 数据，将指令写入 load_queue；如果加载的是 micro-op kernel 或寄存器文件，将指令写入 gemm_queue。

## Architectural Overview

### Instruction Set Architecture

### Dataflow Execution

这一小节讲的是 VTA 解决数据冒险的方法：通过依赖队列在模块间进行同步。简单讲就是，一个模块执行一条指令前首先要获取 producer 的 RAW dependence token 和 consumer 的 WAR dependence token，执行完这条指令后，再向自己的 WAR 和 RAW 输入队列写入 token。具体是否需要 token 和是否需要写入 token，都编码在指令的 pop_prev, pop_next, push_prev, push_next 域里。但我比较疑惑的是，为什么 RAW dependence 队头 token 就是模块当前指令需要的 token？？首先，采用了 FIFO 肯定是一个重要原因；其次，我猜 JIT runtime 在生成指令序列的时候也会有特定的限制。总之，具体的细节还是有待后续验证。

### Pipeline Expandability

这一小节主要讲了可以对 VTA 做一些修改，以支持更多段数的流水线。VTA 支持 load-compute-store 三段流水，因为 compute 模块中有 gemm 和 alu 两个模块，也可以将 alu 拆分出来，从而变成 load-compute-activate-store 四段流水，这就更接近 **TPU** 的设计。

## Microarchitectural Overview

### Fetch Module

Fetch模块是 CPU 调用 VTA 的入口，CPU 通过三个寄存器对 Fetch 模块进行控制：写 control 寄存器可以启动 fetch模块，读 control 寄存器可以检查指令流是否执行完毕（所以为什么在 hls 代码中没见 control 寄存器？？）；写 insn_count 寄存器可以设置要执行的指令条数；写 insns 寄存器可以设置指令流在 DRAM 中的起始地址。同时，VTA Runtime 会将指令流存储在 DRAM 上连续的一块上。

### Compute Module

### Load and Store Module

图示的 2D-load 配合伪代码容易看懂。

![img](https://cdn.nlark.com/yuque/0/2023/png/32845095/1696647841037-50f80fef-5338-456c-994a-fe6569c66cb5.png)