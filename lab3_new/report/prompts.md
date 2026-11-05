# Lab 3 AI 提示词与 Spec 记录

> 本文归纳本次开发中使用的任务描述与代码审查要求，为过程摘要而非完整逐字聊天记录；实际错误、修复和验证结果见 `debug.md`。

## 任务记录 1：Trap 入口及恢复

请以我已完成并通过 Lab 2 页面管理测试的 RISC-V64 uCore 工程为基础，建立 Trap 上下文保存-处理-恢复闭环。仅补全 kern/trap/trapentry.S 的 RESTORE_ALL、__alltraps，禁止修改指导书提供的 SAVE_ALL、struct trapframe 和 trap(struct trapframe *) 的签名。布局固定为 32 个 GPR（x0–x31）加 4 个 CSR，占 36 × 8 字节，原 sp 保存到第 2 个槽。按 ABI 通过 a0 传递 TrapFrame 指针给 trap()；按相同布局恢复 GPR、sstatus 和 sepc，最后恢复 sp，通过 sret 返回。要求检查栈对齐、寄存器覆盖风险，展示文件级修改和验证方案；如果框架与课程指导书不符，先列出差异。

## 任务记录 2：中断与异常处理

请按 2026 Lab3 指导书实现 kern/trap/trap.c 中 interrupt_handler 和 exception_handler。按 scause 最高位区分中断和异常，S_TIMER 必须每次重新编程下一次事件并更新 ticks，每 TICK_NUM 个 tick 打印一次统计；累计 SHUTDOWN_AFTER 次输出则通过工程已有 SBI 关机接口关机。其他已知中断简单分类并输出，未知 Trap 需打印完整 TrapFrame。非法指令与断点异常要严格按指定输出文字打印，并结合 RISC-V 指令低位编码决定 epc 加 2 还是 4；其他异常打印完整 TrapFrame。不得破坏之前的 dtb_init、pmm_init 和 Best-Fit/Buddy/SLUB 实现。标明所有未在材料中确定的数值与接口。

## 任务记录 3：静态审查与完善

请审查生成的 Trap 处理代码是否遗漏 print_regs，是否恢复了所有 GPR 以及 sstatus/sepc，是否误恢复 x0、提前恢复 sp、忘记设置下一次时钟事件或将所有异常固定加 4；添加不影响默认测试的条件编译自测项，覆盖 32 位 ebreak、16 位 c.ebreak、32 位非法指令。不要声称执行过 QEMU，除非提供真实日志；如果没有官方 grade.sh，保留课程待核验项。

## 任务记录 4：构建与回归

请在不覆盖 Lab2 原有代码的基础上补齐 Lab3 Makefile、Trap 框架、clock/intr 驱动和必要 SBI 函数。在 RISC-V64 GNU 工具链编译，运行 QEMU 观察 Lab2 回归、100 ticks 重复输出、关机以及可选异常自测。编译或运行报错时应精确记录命令、错误文本、分析与修复，不得虚构运行截图或打分结果。

## 迭代记录

- 审查发现第一个适配版仅打印 CSR 而没有全部 GPR，新增完整 print_regs。
- 补充可选异常自测，验证 2/4 字节 EPC 调整。
- 将官方未确定的 SHUTDOWN_AFTER 数值、异常输出格式和 grader 单独列为待核验项。
