---
title: Special Registerse
categories: ["Computer Science", Hardware]
tags: ["arm basic"]
date: 2023-11-08 22:57 +0900
author: jun
---

## Introduce

While i'm trying to learn iOS mitigation [met](https://blog.siguza.net/KTRR/) the `ttbr1_el1` that part of aarch64 **'system registers'**. That's why i starting write this post.

> This post only deal with arm related knowledge so if you want to know about the intel's special registers, sorry for that.
{: .prompt-warning}

As far as i know, since ARMv8-A or after has many different features that before ARMv7-A or earlier so i seperate section to avoid confusion. You can find more information at [here](https://www.jkelec.co.kr/img/lecture/cortex_arch/cortex_arch_2.html) how they different.

## Special Registers

The ARM processor using many of type registers also including general purpose. But in this post we don't talk about general purpose registers.

### ARMv7-A

> I'm writing based on [ARMv7-A offcial document](https://developer.arm.com/documentation/den0013/d/ARM-Processor-Modes-and-Registers?lang=en) but still not understand everything because it's too complicate to newbie like me. So if you find any wrong statement, please let me know.
{: .prompt-danger}

#### Program status registers (PSRs)

A program status registers has two type that "Current Program Status Registers(CPSR)" and "Saved Program Status Registers(SPSR)". A CPSR is store condition flags, processor mode, Thumb state and other status that related processor and SPSR is copy of CPSR values. A SPSR used to restore when exception handleing.

> In User mode, a restricted form of the CPSR called the _Application Program Status Register_ (APSR) is accessed instead.
{: .prompt-info}

#### System control register (SCTLR)

The SCTLR is one of registers that is accessed using CP15, and control standard memory, system facilities and provides status information for functions implemented in the core. We talk about a CP15 in the next section what is that and what purpose that.

> You can find more detailed description on [here](https://developer.arm.com/documentation/den0013/d/ARM-Processor-Modes-and-Registers/Registers/System-control-register--SCTLR-?lang=en)!
{: .prompt-info}

#### Coprocessor 15 (CP15)

The CP15 used for control "system control register" and other features of the core. I think we don't need to understand everything of CP15 because it's not used in aarch64 but to diffing just remember what their purpose.

---

### AArch64

CP15 is no longer used in Aarch64 to control system registers and it's replaced to `MSR` and `MRS` instruction. 

#### System Registers

A System registers used to control **system configuration** likes 'hypervisor configuration', 'floating-point control' and 'floating-point status' and more. And every register specified their exception level.

For example:

- TTBR0_EL1 is accessible from EL1, EL2, and EL3.

- TTBR0_EL2 is accessible from EL2 and EL3.

> Remember: Privilege levels be higher from EL0 to EL3
{: .prompt-tip}

#### MSR & MRS

- MSR: moving 'immediate value' or 'general purpose register' to 'system register'

- MRS: moving 'system register' value to 'general purpose register'

As you can see, their explain is look so simple but internal structure is not. In case of the moving immediate value to system register by `MSR` instruction, you need to know what is `PSTATE` and how used that.

#### PSTATE

> [*original article*](https://gongpd.tistory.com/9)

In the AArch64 is using "Processor State(**PSTATE**)" instead "CPSR". A PSTATE is including ALU flags, execution state, execution level and more information.

> CPSR and PSTATE is not a perfectly same one but it using for same purpose.
{: .prompt-tip}

#### SPSR

A **Saved Program Status Register** is used to store and restore PSTATE when exception is occured.

## References

- https://medium.com/@deryugin.denis/poring-os-to-aarch64-a0a5dfa38c5d
- https://lioncash.github.io/ARMBook/the_apsr,_cpsr,_and_the_difference_between_them.html
- https://developer.arm.com/documentation/den0024/a/ARMv8-Registers/System-registers?lang=en
- https://gongpd.tistory.com/9
- https://developer.arm.com/documentation/den0013/d/ARM-Processor-Modes-and-Registers/Registers/Program-Status-Registers?lang=en
- https://developer.arm.com/documentation/ddi0602/2023-09/Base-Instructions/MSR--immediate---Move-immediate-value-to-Special-Register-?lang=en
- https://developer.arm.com/documentation/ddi0602/2023-09/Base-Instructions/MSR--register---Move-general-purpose-register-to-System-Register-?lang=en
- https://developer.arm.com/documentation/ddi0602/2023-09/Base-Instructions/MRS--Move-System-Register-to-general-purpose-register-?lang=en