# QEMU/TCG 的 RISC-V 后端开发 Walkthrough

0. Intro

QEMU/TCG 后端是将 TCG 的中间表示 TCG ops 翻译成 host code 的过程。
其主要分为以下三个部分的工作：
 - 声明后端的能力
 - 添加必要的指令、寄存器约束
 - 实现各个声明的 TCG ops 的功能。
其中，前两者用来帮助 TCG 生成后端易于实现的 TCG ops。后者则是将 TCG ops 翻译成对应的 host code 的过程。

1. 声明后端的能力

TCG 很聪明，会根据你的后端能力来生成不同的 TCG ops。所以，你需要在 `tcg/tcg-target.h` 中声明你的后端能力。

通常来说，可选的能力都会有一个 `#define TCG_TARGET_HAS_XXX` 的宏，你只需要将其定义为 1 即可。

2. 添加必要的指令、寄存器约束

首先，你需要在 `tcg/tcg-target-con-str.h` 中声明指令参数的约束。

分两类约束：

- 寄存器约束

形如 `REGS('x', val)` 这样的形式。
其中 `x` 是一个之后会被用到的字母简写，val 是一个数字，是可以用的寄存器的 mask。（mask 的是 TCGReg enum 的值，比如 `0xffffffff` 表示所有的序号为 0-31 的寄存器）val 可以是一个在 `tcg/tcg-target.c.inc` 中定义的宏。

- 常数约束（立即数约束）

形如 `CONSTS('i', val)` 这样的形式。
其中 `i` 是一个之后会被用到的字母简写，val 是一个数字，是可以用的立即数的 mask。随后在 tcg_target_const_match() 函数中会使用这个 mask 来检查是否是一个合法的立即数。

随后，你可以在 `tcg/tcg-target-con-set.h` 中去声明指令格式的约束。

这个文件里面有一段注释来说明怎么添加一个指令格式的约束。
```
C_On_Im(...) defines a constraint set with <n> outputs and <m> inputs.
Each operand should be a sequence of constraint letters as defined by
tcg-target-con-str.h; the constraint combination is inclusive or.
```
也就是说，`C_On_Im(...)` 是一个有 `n` 个输出和 `m` 个输入的约束。
举例说明的话，`C_O1_I1(v, vr)` 意味着可以有 1 个输出和 1 个输入，第一个 `v` 意味着第一个*输出*可以是 v 类型，第二个 `vr` 意味着第一个*输入*可以是 v 类型或是一个 r 类型。具体的字母所代表的约束类型可以在 `tcg/tcg-target-con-str.h` 中找到。字母之间是*或*的关系。

3. 实现各个声明的 TCG ops 的功能

这一步是最重要的一步，也是最复杂的一步。
首先需要补全所有已经声明了的 TCG ops 对应的函数（形如 `void tcg_out_op_xxx(TCGContext *s, ...)` 的函数）。你要做的事情是，首先查阅 QEMU 的文档，了解这个 TCG ops 的功能，然后根据这个功能，将其翻译成对应的 host code。你可以通过 `tcg_outxxx()` 函数来输出 host code。在 RISC-V 中用的最多的是 `tcg_out32`，因为 RISC-V 的指令长度通常是 32 位的长度。

所以其实是需要我们自己来拼出每个指令对应的二进制编码的！这时候就需要翻阅 RISC-V 的手册了。RISC-V 的手册可以在这里找到：https://riscv.org/specifications/，在这里，我们需要查阅 RISC-V 的指令集手册，来了解每个指令的二进制编码。或者，也可以 https://wiki.riscv.org/display/HOME/Recently+Ratified+Extensions 在这里查阅需要使用到的扩展相关的信息。建议随手备着一张 RISC-V 指令集的表格或者正在使用的扩展的 spec pdf，方便查阅。

需要检验自己的是否正确，可以通过给 QEMU 添加 `-d out_asm` 参数来查看 QEMU 的输出，看看自己的指令是否正确。需要注意的是，输出默认会输出到 `stderr`，有必要的话可以通过 `-D` 参数来指定输出的文件，或者通过 `2>` 来重定向输出（比如 `2>&1` 重定向到 `stdout`）。也可以通过比对 godbolt.org 或者直接写一个对应指令的程序并通过 `objdump` 比对，来查看自己的编码的实现是否正确。https://github.com/Swung0x48/riscv-insn-playground 这个仓库里面是一个非常简单的测试环境，可以用来快速开始编写和测试 RISC-V 的汇编程序。

4. 运行和调试

最后的最后，你需要将你的后端编译进 QEMU 中，然后运行 QEMU 来测试你的后端是否正确。
如果出现了任何运行时的问题需要调试，可以通过 `gdb` 来调试 QEMU。如果你很不幸需要和我一样，测试的后端没有直接的硬件支持，那可能需要一个 QEMU-on-QEMU 的环境。调试环境配置的方法可以参考(这里)[../HowTo/调试运行在 chroot 环境中的 qemu-user.md]。

掌握了以上这些，我想你应该可以开始为 QEMU/TCG 后端贡献代码了。祝你好运！