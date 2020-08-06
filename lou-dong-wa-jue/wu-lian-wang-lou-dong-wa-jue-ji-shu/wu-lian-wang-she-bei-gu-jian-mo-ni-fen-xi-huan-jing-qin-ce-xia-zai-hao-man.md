---
description: 搭建自己的IOT分析环境
---

# 物联网设备固件模拟分析环境\*（亲测下载好慢,不建议使用该教程）

**在这篇文章中，我们将看看如何才能完成特定物联网设备的固件模拟。固件模拟可以用于许多不同的目的，例如更好地分析固件，进行开发，执行远程调试等。**  


通过这种技术，您可以模拟最初想在不同架构上运行的固件，并与其进行交互，即使没有物理物联网设备。固件模拟较早方法之一是创建Qemu镜像，然后将固件文件系统的内容复制到Qemu镜像上，然后启动镜像。

但是，存在一个更简单的方案，它在模拟固件时不容易出现问题。我们来看一下。

### 您需要的工具： <a id="h2-1"></a>

> * [AttifyOS VM](https://github.com/adi0x90/attifyos/)或任何基于Linux的映像
> * 固件分析工具包（[https://github.com/attify/firmware-analysis-toolkit）](https://github.com/attify/firmware-analysis-toolkit%EF%BC%89)
> * 您想要模拟的固件（例如：[Netgear WNAP320](http://www.downloads.netgear.com/files/GDC/WNAP320/WNAP320%20Firmware%20Version%202.0.3.zip)）

### 设置 <a id="h2-2"></a>

一旦你拥有了上述的三个软件，第一步就是设置固件分析工具包。

固件分析工具包只是实际项目[Firmadyne](https://github.com/firmadyne/firmadyne)的一个包装，可以自动模拟新固件。

要下载和安装FAT，只需简单地克隆git存储库，如下所示：

```text
git clone --recursive https://github.com/attify/firmware-analysis-toolkit.git
```

![](https://image.3001.net/images/20180421/15243085783053.png!small)

接下来，我们需要设置Binwalk，Firmadyne和Firmware-Mod-Kit等独立工具。

### 设置Binwalk <a id="h2-3"></a>

要设置Binwalk，只需先安装如下的依赖关系，然后安装该工具：

```text
cd firmware-analysis-toolkit/binwalk
sudo ./deps.sh
sudo python setup.py install
```

如果一切顺利，您将能够运行`binwalk`并看到如下所示的输出。

![](https://image.3001.net/images/20180421/15243086123946.png!small)

### 设置Firmadyne <a id="h2-4"></a>

要设置Firmadyne，需要到Firmadyne文件夹并打开`firmadyne.config`。如下图所示。

![](https://image.3001.net/images/20180421/15243086386752.png!small)

取消该行`FIRMWARE_DIR=/home/vagrant/firmadyne/`注释，并将地址修改为Firmadyne的当前路径。在我的案例中修改的行如下所示。

![](https://image.3001.net/images/20180421/15243086668566.png!small)

一旦更新了路径，下一步就是下载Firmadyne工作所需的额外二进制文件。这可能需要一段时间（1-2分钟，所以在这个时候我们可以喝杯咖啡或啤酒）。

![](https://image.3001.net/images/20180421/15243086839860.png!small)

完成后，下一步是安装Firmadyne的其余依赖项：

```text
sudo -H pip install git+https://github.com/ahupp/python-magic
sudo -H pip install git+https://github.com/sviehb/jefferson
sudo apt-get install qemu-system-arm qemu-system-mips qemu-system-x86 qemu-utils
```

此时还要根据Firmadyne官方[wiki](https://github.com/firmadyne/firmadyne/)的说明来设置PostgreSQL数据库。

**提示输入密码时，数据库的密码应该是`firmadyne`（避免以后出现问题）。**

```text
sudo apt-get install postgresql
sudo -u postgres createuser -P firmadyne
sudo -u postgres createdb -O firmadyne firmware
sudo -u postgres psql -d firmware < ./firmadyne/database/schema
```

到此Firmadyne设置完成。

#### 设置固件分析工具包 <a id="h3-1"></a>

我们要做的第一件事是移动`fat.py`和`reset.py`到`firmadyne`文件夹下。

完成后，打开`fat.py`并修改`root`密码（以便在运行脚本时不会要求您输入密码），并指定firmadyne的路径，如下所示。

![](https://image.3001.net/images/20180421/15243087351508.png!small)

这就是所有的设置。确保我们的postgresql数据库已启动并正在运行。

![](https://image.3001.net/images/20180421/15243087569324.png!small)

### 模拟固件镜像 <a id="h2-5"></a>

为了模拟固件，我们需要运行`./fat.py`并指定固件名称。在这个案例中，我们指定WNAP320.zip固件。

对于`Brand`，我们可以指定任何参数。

我们的输出如下所示：![](https://image.3001.net/images/20180421/15243088595513.png!small)

一旦完成了固件的初始设置过程，它将为我们提供一个IP地址。在这个案例中固件运行Web服务器，我们能够访问Web界面，以及通过SSH与固件交互进行一些基于网络的开发。

![](https://image.3001.net/images/20180421/15243088813494.png!small)

现在让我们打开Firefox，看看我们是否能够访问Web界面。

![](https://image.3001.net/images/20180421/15243089068417.png!small)

恭喜！ - 我们已经成功地模拟了一个固件（最初是为MIPS Big Endian架构设计的），甚至可以在固件内部访问Web服务器！

[原文链接](https://blog.attify.com/getting-started-with-firmware-emulation/)

由于译者水平有限，希望业内同行批评指正。

