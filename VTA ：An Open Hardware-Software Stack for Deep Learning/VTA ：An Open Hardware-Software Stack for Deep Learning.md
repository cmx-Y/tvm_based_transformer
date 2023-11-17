# VTA ：An Open Hardware-Software Stack for Deep Learning

## 一、论文信息

1. 标题 : VTA ：An Open Hardware-Software Stack for Deep Learning
2. 作者 : Thierry Moreau, Tianqi Chen, Ziheng Jiang, Luis Ceze, Carlos Guestrin, Arvind Krishnamurthy
2. 机构 : University of Washington
3. 刊物/会议 ：axriv
4. 时间 : 2018
5. 链接 : arxiv.org/abs/1807.04188

## 二、 摘要&Introduction

1. ### What is VTA(pronunced vita)?

   - **Versatile Tensor Accelerator**
   - **A common deep learning system stack**
     - VTA is a programmable accelerator that exposes **a RISC-like programming abstraction** to describe operations at the tensor level. 
     - VTA is **an end-to-end solution** —— Drivers，JIT runtime,an Optimizing compiler stack
     - a behavioral hardware simulator &  the infrastructure to deploy VTA on low-cost FPGA development boards for fast prototyping(**current release**)

2. ### What is VTA for?

   - **Provide a common deep learning system stack** for hardware, compilers, and systems researchers alike to incorporate state-of-the-art optimizations and co-design techniques.

   - **Lower the barrier** of entry for machine learning practitioners to experiment with novel network architectures,operators and data representations that require specialized hardware support.

3. ### Use-case Scenerios

   - Hardware Designers and Computer Architects
   - Optimizing Compilers Researchers
   - Deep Learning Researchers

4. ### Stack Overview

   - NNVM Intermediate Representation
   - TVM Intermediate Representation
   - VTA JIT Runtime
   - VTA Instruction Set Architecture
   - VTA Micro-Architecture
   
   ![Overview of VTA stack](C:\Users\acer\Desktop\基于TVM的Transformer计算优化\论文笔记\VTA ：An Open Hardware-Software Stack for Deep Learning\overview_of_VTA_stack.png)
   
   ​																						Figure 1 : Overview of  VTA Stack

## 三、主要内容

1. ### VTA Hardware Architecture

   ![VTA Hardware Organization](C:\Users\acer\Desktop\基于TVM的Transformer计算优化\论文笔记\VTA ：An Open Hardware-Software Stack for Deep Learning\VTA_Hardware_Organization.png)

   ​																					 				Figure 2 : The VTA Hardware Organization

   

   - VTA Design Overview
     - Fetch Module : (1)loading an instruction stream from DRAM;(2) decoding the instruction;
     - Load Module : loading input and weight tensors from DRAM into memories;
     - Compute Module :  (1)Compute : dense linear algebra computation with  GEMM core & general computation with tensor ALU;(2)Loading : DRAM to Register File & Micro-OP Cache；
     - Store Module : Store Computing results from Compute Module to DRAM;
     
   - VTA ISA
     - High Level Instruction Set (CISC) :Complex Instruction Set Computer
     
     - LOAD/GEMM/ALU/STOR![The_VTA_CISC_instruction_fields.jpg](C:\Users\acer\Desktop\基于TVM的Transformer计算优化\论文笔记\VTA ：An Open Hardware-Software Stack for Deep Learning\The_VTA_CISC_instruction_fields.jpg)
     
       ​																		Figure 3 : The VTA CISC instruction fields
     
   - Task-Level Pipeline Parallelism(TLPP)
   
     - Lantency Hiding
   
     ![Task-level pipeline parallelism](C:\Users\acer\Desktop\基于TVM的Transformer计算优化\论文笔记\VTA ：An Open Hardware-Software Stack for Deep Learning\Task-level pipeline parallelism.png)
   
     ​																				Figure 4 : Task-level pipeline parallelism
   
     - Data Dependences : Read-After-Write(RAW)/Write-After-Read(WAR)
   
     ![RAW & WAR.png](C:\Users\acer\Desktop\基于TVM的Transformer计算优化\论文笔记\VTA ：An Open Hardware-Software Stack for Deep Learning\RAW & WAR.png)
   
     ​																									Figure 5 : RAW & WAR
   
     - Dataflow Execution 
     
     ![image-20231017131217799](C:\Users\acer\Desktop\基于TVM的Transformer计算优化\论文笔记\VTA ：An Open Hardware-Software Stack for Deep Learning\the_hardware_module.png)
     
     ​																			Figure 6 : The Hardware Organization
     
     - Pipeline Expandability : load-gemm-activate-store
     
   - VTA Compute Core
   
     - Micro-Ops : **ALU**/**GEMM** Ops && executes micro-ops sequences inside a two-level nested loop
     - GEMM :  Matrix
     - ALU : Vector
     
   - VTA Memory Subsystem
   
     - **One Level** On-chip Memory Hiearchy && **Data-specialized** && Each buffer has a **Single Reader** and a **Single Writer** to allow **Coherent Excution**
     - Bandwidth Considerations : Divergence in Bandwidth Requirements - why **Data-specialized**
     - Memory Access Latency Hiding : DMA transfers from DRAM to SRAM / from SRAM to DRAM
     - Tiled Access Patterns : **strided 2D accesses** &&  insert 2D **padding on-the-fly**
   
     ![load_module_with_strided_access_and_padding_on-the-fly.png](C:\Users\acer\Desktop\基于TVM的Transformer计算优化\论文笔记\VTA ：An Open Hardware-Software Stack for Deep Learning\load_module_with_strided_access_and_padding_on-the-fly.png)
   
     ​									Figure 9 : the Load Module with Strided Access and 2D Padding on-the-fly
   
2. ## VTA Runtime System

   - Compilation Overview

     ![TVM_Compilation_Flow.png](C:\Users\acer\Desktop\基于TVM的Transformer计算优化\论文笔记\VTA ：An Open Hardware-Software Stack for Deep Learning\TVM_Compilation_Flow.png)

     ​																						Figure 10 : TVM Compilation Flow

   - JIT Runtime 

     - Dynamic memory allocation and buffer management.
     - Direct Memory Access (DMA) transfers between main memory (DRAM) and accelerator memory (SRAM).
     - Micro-op kernel generation and caching.
     - Explicit dependence management in the instruction stream.
     - Synchronization between the target CPU and VTA .

3. ### TVM Support for VTA

   - Explicit Memory Management
   - Tensorization
   - Explicit Memory Lantency Hiding

1. ## Evaluation

   - Notes

     - A Preliminary Evaluation on Pynq

     - Emphasize the **full stack at work**, Not **the peak performance**

   - Platform

     - ARM Cortex A9 dual core CPU (667MHz)
     - Artix-7 FPGA
     - 8bit value & 16×16 matrix-vector units(100MHz) & 32bit register for accumulation
     - Peak Throughout 51GOPS (Giga Operations Per Second)
     - 16kB microkernel cache & 32kB activation storage & 256kB parameters storage & 128kB register file
     - On-chip buffers **NOT LARGE ENOUGH ** for single layer of ResNet —> effective memory reuse and memory access latency hiding

   - Benchmark

     - MxNet+ResNet-18 & 32bit float to 8bit fixed & Accuracy 63%
     - H : Height W : Width IC : Input Channels OC : Outout Channels K : Kenerl Size S : Stride
     - Padding : "SAME"

     ![Configurations of all Conv2d.png](C:\Users\acer\Desktop\基于TVM的Transformer计算优化\论文笔记\VTA ：An Open Hardware-Software Stack for Deep Learning\Configurations_of_all_Conv2d.png)

     ​																						table 1 : Configurations of all Conv2d

     - CPU : C1 && Max Pooling Layer && Fully Connect Layer
     - FPGA ： C2-C12 && Activation && Batch Normalization

   - Resource Utilization Efficiency

     - Latency Hiding —> Much closer to the roofline, **Higher Compute and Momery Bandwidth Efficiency**

     ![Arithmetic_Intensity.png](C:\Users\acer\Desktop\基于TVM的Transformer计算优化\论文笔记\VTA ：An Open Hardware-Software Stack for Deep Learning\Arithmetic_Intensity.png)

     ​															Figure 15 : Roofline	 of an FPGA-based deep learning accelerator running ResNet inference

     - **Each Layer has a different Arithmetic Intensity** : Left half - **Bandwidth Limited**; Right half - **Compute Limited**;

   - End-to-end ResNet Evaluation

     - The FPGA provide a **40×** acceleration on **offload convolution layers(*ONLY*)** over Cortex A9![ResNet18_Inference_Time.png](C:\Users\acer\Desktop\基于TVM的Transformer计算优化\论文笔记\VTA ：An Open Hardware-Software Stack for Deep Learning\ResNet18_Inference_Time.png)

       ​																				Figure 16 : ResNet18 Inference Time Comparison

2. ## Conclusion

   - The **Versatile Tensor Accelerator(VTA)**, an **open**, **generic**, and **customizable** deep learning accelerator for **cross-stack** deep learning research and optimization.
   - Demonstrate **the complete VTA design and TVM stack working together**

## 四、贡献

1. **Provide a common deep learning system stack** for hardware, compilers, and systems researchers alike to incorporate state-of-the-art optimizations and co-design techniques.
2. **Lower the barrier** of entry for machine learning practitioners to experiment with novel network architectures,operators and data representations that require specialized hardware support.
3. Understand what **performance bottlenecks** and **Amdahl limitations** stand in the way

## 五、讨论

（优劣之处、应用场景、改进方向等）



