---
description: qemu kvm hypervisor
---

# 虚拟化技术

## QEMU-KVM

在 QEMU-KVM 中，KVM 运行在内核空间，QEMU 运行在用户空间，实际模拟创建，管理各种虚拟硬件，QEMU 将 KVM 整合了进来，通过 / ioctl 调用 /dev/kvm，从而将 CPU 指令的部分交给内核模块来做。

KVM 实现了 CPU 和内存的虚拟化，但 kvm 不能虚拟其他硬件设备，因此 qemu 还有模拟 IO 设备（磁盘，网卡，显卡等）的作用，KVM 加上 QEMU 后就是完整意义上的服务器虚拟化。当然，由于 qemu 模拟 io 设备效率不高的原因，现在常常采用半虚拟化的 virtio 方式来虚拟 IO 设备。

## Hypervisor

Hypervisor——一种运行在基础物理服务器和操作系统之间的中间软件层, 可允许多个操作系统和应用共享硬件。也可叫做 VMM（ virtual machine monitor ），即虚拟机监视器。

