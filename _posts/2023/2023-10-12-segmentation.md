---
title: "Segmentation"
categories: ["Computer Science", "operating system"]
tags: ["basic knowledge"]
date: 2023-10-12 20:35 +0900
author: jun
---

## Overview

Segmentation is the memory management technique that used in **operating system**. It is divide memory as a variable size parts and each part be a **segment**. The reason why older cpu was worked as a 16-bit mode so it only express until 64kb(0xffff) address. 

## Register

Program has six special register called segment register to access segment memory. Intel 8086 use only four register that code segment(CS), data segment(DS), stack segment(SS), extra segment(ES). After intel 80386 also called IA32, added two segment register FS, GS and their are not defined how to use by the hardware.

### x86 series

When search x86 in google it point two processor intel 8086 and 80386. 8086 processor is 16-bit mode but 80386 is 32-bit mode that’s why 80386 also called IA32. Although their bit are different, the segment registers are same at a 16 bit.

### x86_64

The x86_64 architecture doesn’t use segmentation in **long mode** and this reason CS, SS, DS and ES are forced to base address **zero**. But FS and GS use in OS for special purpose likes ‘Thread Local Storogae’ so they still have **nonzero** base address.

### segment selector

In real mode, segment register is used to calculate memory address and directly access their memory. But protected mode and long mode program is not accesable memory. In this reason, both mode segment register value is pointing **descriptor table** that’s why it called Segment Selector.

---

## References

- [x86 memory segmentation wikipedia](https://en.wikipedia.org/wiki/X86_memory_segmentation)
- [segment selector osdev wiki](https://wiki.osdev.org/Segment_Selector)
