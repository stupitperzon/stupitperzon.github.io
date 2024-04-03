---
date: "2024-04-02"
tags: ["cs"]
title: "GDB Process Record"
---

Process record allows one to record the execution of a program and play it back. It logs the machine instruction at each step along with its memory and registers, also known as its instruction status. The logs of the instruction statuses are known as the record list.

Process records need to record instructions and signals. To port recording a new instruction to a new architecture, use `gdbarch_process_record`. This is handled by a special syscall handler in the OS called `record_linux_system_call`. To port recording a new signal to a new architecture, use `gdbarch_process_record_signal`. This is handled by a special signal handler in the OS.

An example of registering a new process record architecture-specific type is using `set_gdbarch_process_record` in `i386-linux-tedp.c`. Then, `gdbarch_process_record_ftype(struct gdbarch *gdbarch, struct regcache *regcache, CORE_ADDR addr)` records a new status and instruction in the record list. The `regcache` contains the value of the registers and memory ranges affected with current values. `addr` is the machine instruction.

[Record-full.c](https://github.com/bminor/binutils-gdb/blob/master/gdb/record-full.c) implements process record. `*-tdep` contains architecture specific process record.

## Resources

<a id="1">[1]</a>
https://sourceware.org/gdb/wiki/ProcessRecord

Process Record

<a id="1">[2]</a>
https://sourceware.org/gdb/wiki/ProcessRecord?action=AttachFile&do=get&target=GDB+Reverse+Debug+and++Process+Record+Target.pdf

Process Record Presentation