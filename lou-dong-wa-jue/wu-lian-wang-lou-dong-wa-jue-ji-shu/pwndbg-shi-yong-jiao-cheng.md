---
description: 熟悉常用的调试命令
---

# pwndbg调试工具使用教程

## 内存布局

![](../../.gitbook/assets/image%20%28106%29.png)

进程地址空间从低地址开始依次是代码段\(Text\)、数据段\(Data\)、BSS段、堆、内存映射段\(mmap\)、栈。

![](../../.gitbook/assets/image%20%28107%29.png)

在进程被载入内存中时，基本上被分裂成许多小的节（section）。我们比较关注的是6个主要的节： 

（1） .text 节 （2）.data 节 （3）.bss 节 （4） 堆节 （5） 栈节 （6）环境/参数节 环境/参数节（environment/arguments section）用来存储系统环境变量的一份复制文件， 进程在运行时可能需要。例如，运行中的进程，可以通过环境变量来访问路径、shell 名称、主机名等信息。 该节是可写的，因此在格式串（format string）和缓冲区溢出（buffer overflow）攻击中都可以使用该节。 另外，命令行参数也保持在该区域中。





