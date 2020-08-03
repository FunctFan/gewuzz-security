---
description: 知道创宇404 ScanV 安全服务团队
---

# TP-Link SR20 本地网络远程代码执行漏洞

## 简述

3月26号 Google 安全开发人员 Matthew Garrett在 Twitter 上公布了 TP-Link Smart Home Router \(SR20\) 的远程代码执行漏洞，公布的原因是他去年 12 月份将相关漏洞报告提交给 TP-Link后没有收到任何回复，于是就公开了，该漏洞截至目前官方修复，在最新固件中漏洞仍然存在，属于 0day 漏洞，当我看到漏洞证明代码\(POC\)后决定尝试重现此漏洞

  
TP-Link SR20 是一款支持 Zigbee 和 Z-Wave 物联网协议可以用来当控制中枢 Hub 的触屏 Wi-Fi 路由器，此远程代码执行漏洞允许用户在设备上以 root 权限执行任意命令，该漏洞存在于 TP-Link 设备调试协议\(TP-Link Device Debug Protocol 英文简称 TDDP\) 中，TDDP 是 TP-Link 申请了专利的调试协议，基于 UDP 运行在 1040 端口

![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190902175944477.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzgwNTQ5,size_16,color_FFFFFF,t_70)

TP-Link SR20 设备运行了 V1 版本的 TDDP 协议，V1 版本无需认证，只需往 SR20 设备的 UDP 1040 端口发送数据，且数据的第二字节为 `0x31` 时，SR20 设备会连接发送该请求设备的 TFTP 服务下载相应的文件并使用 LUA 解释器以 root 权限来执行，这就导致存在远程代码执行漏洞  


![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190902180003492.png)

## 漏洞复现

根据[文章](https://paper.seebug.org/879/)的描述，漏洞的基理为：TP-Link SR20 设备运行了 V1 版本的 TDDP 协议，V1 版本无需认证，只需往 SR20 设备的 UDP 1040 端口发送数据，且数据的第二字节为 `0x31` 时，SR20 设备会连接发送该请求设备的 TFTP 服务下载相应的文件并使用 LUA 解释器以 root 权限来执行，这就导致存在远程代码执行漏洞。

首先是对漏洞进行复现，后面再对漏洞原理进行分析。

首先是固件下载，固件可在[官网](https://www.tp-link.com/us/support/download/sr20/#Firmware)进行下载。最新的固件版本为[SR20\(US\)\_V1\_190401](https://static.tp-link.com/2019/201904/20190402/SR20%28US)\_V1\_190401.zip\)，此为已经修复漏洞的版本。存在漏洞的版本为[SR20\(US\)\_V1\_180518](https://static.tp-link.com/2018/201806/20180611/SR20%28US)\_V1\_180518.zip\)。将两个版本的固件都下下来，后续还会使用bindiff对二者进行比对，来看是如何修复该漏洞的。

接着是环境搭建，最主要的是qemu和binwalk的安装。环境搭建的过程可以参考之前的[文章](https://ray-cp.github.io/archivers/MIPS_Debug_Environment_and_Stack_Overflow)，同时一键安装iot环境的[脚本](https://github.com/ray-cp/Tool_Script/blob/master/iot_env_install.md)，也可以用用，虽然不全，但是也包含了一些，还需要手动操作的就是以系统模式运行qemu的时候还需要配置下网卡。

固件和环境都配好了以后，接下来就是解压固件，使用以下命令将漏洞版本的文件系统提取出来：

```text
binwalk -Me sr20.bin
```

然后查看文件类型：

```text
$ file ./squashfs-root/bin/busybox
./squashfs-root/bin/busybox: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-, stripped
```

可以看到文件是基于arm 32位的小端ELF文件。

### qemu分析环境模拟

接着使用qemu系统模式运行起来一个arm虚拟机，虚拟机的下载地址为[https://people.debian.org/~aurel32/qemu/armhf/](https://people.debian.org/~aurel32/qemu/armhf/)，运行命令为（需配置好网络，可参考[文章](https://ray-cp.github.io/archivers/MIPS_Debug_Environment_and_Stack_Overflow#qemu%E6%A8%A1%E6%8B%9F%E8%BF%90%E8%A1%8Cmips%E7%B3%BB%E7%BB%9F)）：

ARM CPU 有两个矢量浮点（软浮点和硬浮点）具体区别可以查看 Stackoverflow，本次选择使用硬浮点 armhf

从 Debian 官网下载 QEMU 需要的 Debian ARM 系统的三个文件:

* debian\_wheezy\_armhf\_standard.qcow2 2013-12-17 00:04 229M
* initrd.img-3.2.0-4-vexpress 2013-12-17 01:57 2.2M
* vmlinuz-3.2.0-4-vexpress 2013-09-20 18:33 1.9M

把以上三个文件放在同一个目录执行以下命令

```text
$ sudo tunctl -t tap0 -u `whoami`  # 为了与 QEMU 虚拟机通信，添加一个虚拟网卡
$ sudo ifconfig tap0 10.10.10.1/24 # 为添加的虚拟网卡配置 IP 地址
$ qemu-system-arm -M vexpress-a9 -kernel vmlinuz-3.2.0-4-vexpress -initrd initrd.img-3.2.0-4-vexpress -drive if=sd,file=debian_wheezy_armhf_standard.qcow2 -append "root=/dev/mmcblk0p2 console=ttyAMA0" -net nic -net tap,ifname=tap0,script=no,downscript=no -nographic
123
```

![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190902180458501.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzgwNTQ5,size_16,color_FFFFFF,t_70)

![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190902180507266.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzgwNTQ5,size_16,color_FFFFFF,t_70)

虚拟机启动成功后会提示登陆

用户名和密码都为 `root`

![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190902180525642.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzgwNTQ5,size_16,color_FFFFFF,t_70)

配置网卡IP

```text
ifconfig eth0 10.10.10.2/24
```

![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190902180544225.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzgwNTQ5,size_16,color_FFFFFF,t_70)

此时 QEMU 虚拟机可以与宿主机进行网络通信

### 固件打包上传到qemu虚拟机

在宿主机中压缩文件系统并启动web服务：

```text
tar jcf tar -jcf squashfs-root.tar.bz2 squashfs-root
python -m SimpleHTTPServer 80
```

然后在qemu虚拟机中下载文件系统:

```text
wget http://192.168.10.1/squashfs-root.tar.bz2
tar jxf squashfs-root.tar.bz2
```

接着使用 chroot 切换根目录固件文件系统。

```text
mount -o bind /dev ./squashfs-root/dev/
mount -t proc /proc/ ./squashfs-root/proc/
chroot squashfs-root sh # 切换根目录后执行新目录结构下的 sh shell
```

使用 chroot 后，系统读取的是新根下的目录和文件，也就是固件的目录和文件。 chroot 默认不会切换 /dev 和 /proc, 因此切换根目录前需要现挂载这两个目录。

到此可以看到已经切换到了该固件的环境

```text
root@debian-armhf:~/work# mount -o bind /dev ./squashfs-root/dev/
root@debian-armhf:~/work# mount -t proc /proc/ ./squashfs-root/proc/
root@debian-armhf:~/work# chroot squashfs-root sh


BusyBox v1.19.4 (2018-05-18 20:52:39 PDT) built-in shell (ash)
Enter 'help' for a list of built-in commands.

/ #
```

然后宿主机中安装ftp服务器：

```text
sudo apt install atftpd
```

配置ftp服务：

```text
vim /etc/default/atftpd
# 修改USE_INETD=true 改为 USE_INETD=false
# 修改修改/srv/tftp为相应的ftp目录，我这里为/opt/ftp
```

配置目录

```text
sudo mkdir /opt/ftp_dir
sudo chmod 777 /opt/ftp_dir
```

启动服务

```text
sudo systemctl start atftpd
```

使用`sudo systemctl status atftpd`可查看服务状态。如果执行命令 `sudo systemctl status atftpd` 查看 atftpd 服务状态时，提示 `atftpd: can't bind port :69/udp` 无法绑定端口，可以执行 `sudo systemctl stop inetutils-inetd.service` 停用 `inetutils-inetd` 服务后，再执行 `sudo systemctl restart atftpd` 重新启动 atftpd 即可正常运行 atftpd。

前面都是准备环境的环节，接着就是复现漏洞的真正操作部分了。

