---
description: 内核相关的一些知识点总结
---

# Linux内核基础

## [/proc详解](https://www.cnblogs.com/liushui-sky/p/9354536.html)

Linux系统上的/proc目录是一种文件系统，即proc文件系统。与其它常见的文件系统不同的是，/proc是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件，用户可以通过这些文件查看有关系统硬件及当前正在运行进程的信息，甚至可以通过更改其中某些文件来改变内核的运行状态。

## SMAP/SMEP

* SMAP\(Supervisor Mode Access Prevention，管理模式访问保护\)
* SMEP\(Supervisor Mode Execution Prevention，管理模式执行保护\)

1. 作用分别是禁止内核访问用户空间的数据和禁止内核执行用户空间的代码。
2. arm里面叫PXN\(Privilege Execute Never\)和PAN\(Privileged Access Never\)。
3. SMEP类似于前面说的NX，不过一个是在内核态中，一个是在用户态中。
4. 和NX一样SMAP/SMEP需要处理器支持，可以通过cat /proc/cpuinfo查看，在内核命令行中添加nosmap和nosmep禁用。



