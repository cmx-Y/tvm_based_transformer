## hw_spec.h

这个头文件中主要定义了各种指令的结构体，包括 VTAGenericInsn, VTAMemInsn, VTAGemInsn, VTAAluInsn, VTAUop。

**VTAGenericInsn**

- opcode : 容易理解，用来指明是哪条指令
- pop_prev_dep : 记录指令间依赖信息
- pad_0 : 用来padding

**VTAMemInsn**

- memory_type : 区分 input 和 weight
- sram_base : 容易理解，sram的基地址
- x_size, y_size : 原矩阵块的size
- x_stride : 大矩阵的x轴size
- y_pad_0 : 目的矩阵块的四个方向pad

**VTAInsn**

- 是一个共用体，包含 generic, mem, gemm, alu 四种指令

## vta.cc