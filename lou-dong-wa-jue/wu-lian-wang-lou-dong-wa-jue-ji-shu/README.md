---
description: 工业互联网，物联网，车载网安全
---

# 物联网漏洞挖掘技术

## 分析环境部署

安装必备：qemu,binwalk,pwndbg,gdb-multidbg

### 某实验室开源分析环境：

[https://www.qiling.io/](https://www.qiling.io/)

[https://github.com/qilingframework/qiling](https://github.com/qilingframework/qiling)

### mips环境部署

[https://ray-cp.github.io/archivers/MIPS\_Debug\_Environment\_and\_Stack\_Overflow\#mips-%E6%B1%87%E7%BC%96%E5%9F%BA%E7%A1%80](https://ray-cp.github.io/archivers/MIPS_Debug_Environment_and_Stack_Overflow#mips-%E6%B1%87%E7%BC%96%E5%9F%BA%E7%A1%80) 不看里面的buidroot和网络配置

[https://wzt.ac.cn/2019/09/10/QEMU-networking](https://wzt.ac.cn/2019/09/10/QEMU-networking) 网络照这个配置

qemu-kvm

## GDBserver

编译好的各架构的gdbserver:

{% embed url="https://github.com/e3pem/embedded-toolkit/tree/master/prebuilt\_static\_bins/gdbserver" %}

## 入门资料必备

关于对智能设备如何进行安全分析，请参考： 

绿盟 - 智能设备安全分析手册：[https://book.yunzhan365.com/tkgd/lzkp/mobile/index.html](https://book.yunzhan365.com/tkgd/lzkp/mobile/index.html)

这个是新版的attifyOS 集成了固件分析的工具:

{% embed url="https://github.com/adi0x90/attifyos" %}

总结的IOT资料 [https://zybuluo.com/H4l0/note/1524758](https://zybuluo.com/H4l0/note/1524758) 密码是1286

![&#x5165;&#x95E8;&#x4E8C;&#x8FDB;&#x5236;&#x6F0F;&#x6D1E;&#x5206;&#x6790;&#x8111;&#x56FE;](../../.gitbook/assets/ru-men-er-jin-zhi-lou-dong-fen-xi-nao-tu-.png)

## 常用命令

chroot . ./mydemo /usr/bin/tddp

./gdbserver \*:12345 --attach $\(pgrep smb\)

gdb-peda:c b n ni s target remote ip:port

binwalk -Me  \*.bin

file smb

## qemu模拟

$ sudo tunctl -t tap0 -u `whoami` \# 为了与 QEMU 虚拟机通信，添加一个虚拟网卡                                            $ sudo ifconfig tap0 10.10.10.1/24 \# 为添加的虚拟网卡配置 IP 地址                                                                   $ qemu-system-arm -M vexpress-a9 -kernel vmlinuz-3.2.0-4-vexpress -initrd initrd.img-3.2.0-4-vexpress -drive if=sd,file=debian\_wheezy\_armhf\_standard.qcow2 -append "root=/dev/mmcblk0p2 console=ttyAMA0" -net nic -net tap,ifname=tap0,script=no,downscript=no -nographic 123



