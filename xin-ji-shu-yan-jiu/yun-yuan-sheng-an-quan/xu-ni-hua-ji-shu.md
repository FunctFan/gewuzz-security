---
description: qemu kvm hypervisor
---

# 虚拟化hypervisor简介

## Hypervisor

Hypervisor——一种运行在基础物理服务器和操作系统之间的中间软件层, 可允许多个操作系统和应用共享硬件。也可叫做 VMM（ virtual machine monitor ），即虚拟机监视器。

## kvm、qemu-kvm、ibvirt及openstack之间的关系

KVM是最底层的hypervisor，它是用来模拟CPU的运行，它缺少了对network和周边I/O的支持，所以我们是没法直接用它的。

**KVM 是Linux kernel 的一个模块。可以用命令 modprobe 去加载KVM 模块。加载了模块后，才能进一步通过其他工具创建虚拟机。但仅有KVM 模块是远远不够的，因为用户无法直接控制内核模块去作事情,你还必须有一个运行在用户空间的工具才行。这个用户空间的工具，kvm 开发者选择了已经成型的开源虚拟化软件 QEMU。**

QEMU-KVM就是一个完整的模拟器，它是构建基于KVM上面的，它提供了完整的网络和I/O支持。

Openstack不会直接控制qemu-kvm，它会用一个叫libvirt的库去间接控制qemu-kvm。libvirt提供了跨VM平台的功能，它可以控制除了QEMU之外的模拟器，包括vmware, virtualbox，xen等等。

所以为了openstack的跨VM性，所以openstack只会用libvirt而不直接用qemu-kvm。libvirt还提供了一些高级的功能，例如pool/vol管理。

## KVM-QEMU

Qemu 将KVM 整合进来，通过 ioctl 调用/dev/kvm 接口，将有关CPU 指令的部分交由内核模块来做。kvm 负责 cpu 虚拟化+内存虚拟化，实现了 cpu 和内存的虚拟化，但kvm不能模拟其他设备。qemu 模拟 IO 设备（网卡，磁盘等），kvm 加上 qemu 之后就能实现真正意义上服务器虚拟化。因为用到了上面两个东西，所以称之为qemu-kvm。 Qemu 模拟其他的硬件，如 Network, Disk，同样会影响这些设备的性能，于是又产生了pass through 半虚拟化设备virtio\_blk, virtio\_net，提高设备性能。  


![](../../.gitbook/assets/image%20%28109%29.png)

## LIBVIRT

Libvirt 是管理虚拟机和其他虚拟化功能，比如存储管理，网络管理的软件集合。它包括一个API 库，一个守护程序（libvirtd）和一个命令行工具（virsh）；libvirt 本身构建于一种抽象的概念之上。它为受支持的虚拟机监控程序实现的常用功能提供通用的 API。 libvirt 的主要目标是为各种虚拟化工具提供一套方便、可靠的编程接口，用一种单一的方式管理多种不同的虚拟化提供方式。

## QEMU-KVM

在 QEMU-KVM 中，KVM 运行在内核空间，QEMU 运行在用户空间，实际模拟创建，管理各种虚拟硬件，QEMU 将 KVM 整合了进来，通过 / ioctl 调用 /dev/kvm，从而将 CPU 指令的部分交给内核模块来做。

KVM 实现了 CPU 和内存的虚拟化，但 kvm 不能虚拟其他硬件设备，因此 qemu 还有模拟 IO 设备（磁盘，网卡，显卡等）的作用，KVM 加上 QEMU 后就是完整意义上的服务器虚拟化。当然，由于 qemu 模拟 io 设备效率不高的原因，现在常常采用半虚拟化的 virtio 方式来虚拟 IO 设备。





