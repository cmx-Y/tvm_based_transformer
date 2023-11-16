# BYOC调研报告

https://tvm.apache.org/docs/dev/how_to/relay_bring_your_own_codegen.html
In this part, we demonstrate how to implement a codegen that generates C code with pre-implemented operator functions. —— 这句话说明 codegen 生成的代码会对预先实现的算子实现进行调用。
总的一个目标是，给定一个子图，遍历子图，生成子图执行函数。
对子图的遍历采用的是访问者模式，需要实现访问不同类型节点时的行为。
给了两种方式，一种是生成 C code，一种是生成特定的 graph representation。
生成 C code的大致流程：生成 subgraph function，进行两层 wrapper 以便 TVM runtime 能够直接进行调用，然后进行注册。
生成 graph representation 的大致流程：遍历子图，生成 specific graph，注册一个 runtime module，输入和输出都传给 TVM runtime module，具体的执行调用自己的 engine。

https://tvm.apache.org/2020/07/15/how-to-bring-your-own-codegen-to-tvm
To share the programming interface with widely used deep learning frameworks, many hardware device providers have attempted to integrate their devices backend to TensorFlow. However, since TensorFlow does not provide an official backend interface for new backends, you have to hack the TensorFlow for registration, which involves many source file changes and makes the future maintenance difficult. —— 这里说明将特定后端对接到 TensorFlow 上是比较困难的。
(上面一个 tutorial 讲的主要是代码生成部分)使用BYOC的整体流程：graph annotation -- code generation -- Implement runtime

paper： Bring Your Own Codegen to Deep Learning Compiler  
To the best of our knowledge, this is the first practical solution that provides a unified, customizable compilation framework for users to integrate their own accelerator code generation to a DLC, and it has been used by multiple commercial accelerators. 

总结起来就是BYOC 没用到 tvm 的schedule 部分，算子实现由 hardware provider 提供，不太像是我们想要走的技术路线。