---
description: 内核相关的一些知识点总结
---

# Linux内核基础

## 内核基本结构

![](https://images2015.cnblogs.com/blog/431521/201605/431521-20160523163606881-813374140.png)

　　如上图所示，从宏观上来看，Linux操作系统的体系架构分为用户态和内核态（或者用户空间和内核空间）。内核从本质上看是一种软件——控制计算机的硬件资源，并提供上层应用程序运行的环境。用户态即上层应用程序的活动空间，应用程序的执行必须依托于内核提供的资源，包括CPU资源、存储资源、I/O资源等。为了使上层应用能够访问到这些资源，内核必须为上层应用提供访问的接口：即系统调用。

## /proc详解

Linux系统上的/proc目录是一种文件系统，即proc文件系统。与其它常见的文件系统不同的是，/proc是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件，用户可以通过这些文件查看有关系统硬件及当前正在运行进程的信息，甚至可以通过更改其中某些文件来改变内核的运行状态。

## SMAP/SMEP

* SMAP\(Supervisor Mode Access Prevention，管理模式访问保护\)
* SMEP\(Supervisor Mode Execution Prevention，管理模式执行保护\)

1. 作用分别是禁止内核访问用户空间的数据和禁止内核执行用户空间的代码。
2. arm里面叫PXN\(Privilege Execute Never\)和PAN\(Privileged Access Never\)。
3. SMEP类似于前面说的NX，不过一个是在内核态中，一个是在用户态中。
4. 和NX一样SMAP/SMEP需要处理器支持，可以通过cat /proc/cpuinfo查看，在内核命令行中添加nosmap和nosmep禁用。



