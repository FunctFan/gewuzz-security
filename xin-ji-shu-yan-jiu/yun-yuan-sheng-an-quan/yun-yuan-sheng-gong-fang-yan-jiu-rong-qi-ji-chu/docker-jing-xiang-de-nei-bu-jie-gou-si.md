---
description: 熟悉Docker镜像内部结构
---

# Docker镜像的内部结构\(四\)

## 一、base镜像

base 镜像简单来说就是不依赖其他任何镜像，完全从0开始建起，其他镜像都是建立在他的之上，可以比喻为大楼的地基，docker镜像的鼻祖。

base 镜像有两层含义：（1）不依赖其他镜像，从 scratch 构建；（2）其他镜像可以之为基础进行扩展。

所以，能称作 base 镜像的通常都是各种 Linux 发行版的 Docker 镜像，比如 Ubuntu, Debian, CentOS 等。

我们以 CentOS 为例查看 base 镜像包含哪些内容。

下载及查看镜像：

```text
root@ubuntu:~# docker pull centos
Using default tag: latest
latest: Pulling from library/centos
d9aaf4d82f24: Pull complete 
Digest: sha256:4565fe2dd7f4770e825d4bd9c761a81b26e49cc9e3c9631c58cfc3188be9505a
Status: Downloaded newer image for centos:latest
```

```text
root@ubuntu:~# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              d123f4e55e12        3 weeks ago         197MB
ubuntu              12.04               5b117edd0b76        7 months ago        104MB
```

我们看到CentOS的镜像大小不到200MB，平时我们安装一个CentOS至少是几个GB，怎么可能才 200MB !

下面我们来解释这个问题，Linux 操作系统由内核空间和用户空间组成。

典型的Linux启动到运行需要两个FS，bootfs + rootfs，如下图：

![Docker&#x955C;&#x50CF;&#x7684;&#x5185;&#x90E8;&#x7ED3;&#x6784;\(&#x56DB;\)](https://s4.51cto.com/images/blog/201711/28/9074aa31f4e98cec2b11b2c62ce815bb.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 1、rootfs

内核空间是 kernel，Linux 刚启动时会加载 bootfs 文件系统，之后 bootfs 会被卸载掉。用户空间的文件系统是 rootfs，包含我们熟悉的 /dev, /proc, /bin 等目录。

对于 base 镜像来说，底层直接用 Host 的 kernel，自己只需要提供 rootfs 就行了。

而对于一个精简的 OS，rootfs 可以很小，只需要包括最基本的命令、工具和程序库就可以了。相比其他 Linux 发行版，CentOS 的 rootfs 已经算臃肿的了，alpine 还不到 10MB。

我们平时安装的 CentOS 除了 rootfs 还会选装很多软件、服务、图形桌面等，需要好几个 GB 就不足为奇了。

### 2、base 镜像提供的是最小安装的 Linux 发行版。

下面是 CentOS 镜像的 Dockerfile 的内容：

![Docker&#x955C;&#x50CF;&#x7684;&#x5185;&#x90E8;&#x7ED3;&#x6784;\(&#x56DB;\)](https://s4.51cto.com/images/blog/201711/27/23115d7a22cd1cf9e8e1364a6adcaa07.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

```text
FROM scratch
ADD centos-7-docker.tar.xz /

LABEL name="CentOS Base Image" \
    vendor="CentOS" \
    license="GPLv2" \
    build-date="20170911"

CMD ["/bin/bash"]
```

第二行 ADD 指令添加到镜像的 tar 包就是 CentOS 7 的 rootfs。在制作镜像时，这个 tar 包会自动解压到 / 目录下，生成 /dev, /porc, /bin 等目录。

### 3、支持运行多种 Linux OS

bootfs \(boot file system\) 主要包含 bootloader 和 kernel, bootloader主要是引导加载kernel, 当boot成功后 kernel 被加载到内存中后 bootfs就被umount了.

rootfs \(root file system\) 包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。

由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别, 因此不同的发行版可以公用bootfs。

比如 Ubuntu 14.04 使用 upstart 管理服务，apt 管理软件包；而 CentOS 7 使用 systemd 和 yum。这些都是用户空间上的区别，Linux kernel 差别不大。

所以 Docker 可以同时支持多种 Linux镜像，模拟出多种操作系统环境。

  
上图 Debian 和 BusyBox（一种嵌入式 Linux）上层提供各自的 rootfs，底层共用 Docker Host 的 kernel。

![](../../../.gitbook/assets/image%20%28124%29.png)

这里需要说明的是：

**base 镜像只是在用户空间与发行版一致，kernel 版本与发型版是不同的。**

CentOS 7 使用 3.x.x 的 kernel，如果 Docker Host 是 Ubuntu 16.04，那么在 CentOS 容器中使用的实际是是 Host 4.x.x 的 kernel。

```text
root@ubuntu:~# uname -r
4.4.0-62-generic
root@ubuntu:~# docker run -ti centos /bin/bash
[root@06f13ef13853 /]# uname -r
4.4.0-62-generic
```

**容器只能使用 Host 的 kernel，并且不能修改。**

所有容器都共用 host 的 kernel，在容器中没办法对 kernel 升级。如果容器对 kernel 版本有要求（比如应用只能在某个 kernel 版本下运行），则不建议用容器，这种场景虚拟机可能更合适。



## 二、镜像的分层结构

Docker 支持通过扩展现有镜像，创建新的镜像。

实际上，Docker Hub 中 99% 的镜像都是通过在 base 镜像中安装和配置需要的软件构建出来的。比如我们现在构建一个新的镜像，Dockerfile 如下：

```text
# Version: 0.0.1
FROM debian                1.新镜像不再是从 scratch 开始，而是直接在 Debian base 镜像上构建。
MAINTAINER wzlinux
RUN apt-get update && apt-get install -y emacs        2.安装 emacs 编辑器。
RUN apt-get install -y apache2             3.安装 apache2。
CMD ["/bin/bash"]              4.容器启动时运行 bash。
```

构建过程如下图所示：

![](../../../.gitbook/assets/image%20%28125%29.png)

  
可以看到，新镜像是从 base 镜像一层一层叠加生成的。每安装一个软件，就在现有镜像的基础上增加一层。

问什么 Docker 镜像要采用这种分层结构呢？

最大的一个好处就是 - 共享资源。

比如：有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像；同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享，我们将在后面更深入地讨论这个特性。

这时可能就有人会问了：如果多个容器共享一份基础镜像，当某个容器修改了基础镜像的内容，比如 /etc 下的文件，这时其他容器的 /etc 是否也会被修改？

答案是不会！  
修改会被限制在单个容器内。  
这就是我们接下来要说的容器 Copy-on-Write 特性。

1. 新数据会直接存放在最上面的容器层。
2. 修改现有数据会先从镜像层将数据复制到容器层，修改后的数据直接保存在容器层中，镜像层保持不变。
3. 如果多个层中有命名相同的文件，用户只能看到最上面那层中的文件。

### 可写的容器层

当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

典型的Linux在启动后，首先将 rootfs 置为 readonly, 进行一系列检查, 然后将其切换为 “readwrite” 供用户使用。在docker中，起初也是将 rootfs 以readonly方式加载并检查，然而接下来利用 union mount 的将一个 readwrite 文件系统挂载在 readonly 的rootfs之上，并且允许再次将下层的 file system设定为readonly 并且向上叠加, 这样一组readonly和一个writeable的结构构成一个container的运行目录, 每一个被称作一个Layer。如下图所示。

![](../../../.gitbook/assets/image%20%28123%29.png)

  
所有对容器的改动，无论添加、删除、还是修改文件都只会发生在容器层中。只有容器层是可写的，容器层下面的所有镜像层都是只读的。

