---
description: 容器本质+LXC+三大特色
---

# docker之namespace，cgroup与unionFS

以下内容引自安全客「[JerryWang\_汪子熙](https://www.jianshu.com/u/99b8712e8850)」在其「[Docker技术三大要点：cgroup, namespace和unionFS的理解](https://www.jianshu.com/p/47c4a06a84a4)」的博文，具体内容已经是精简后的版本：

{% hint style="info" %}
作者：JerryWang\_汪子熙  
链接：https://www.jianshu.com/p/47c4a06a84a4  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
{% endhint %}

## 容器本质

Docker其实是容器化技术的具体技术实现之一，采用go语言开发。很多朋友刚接触Docker时，认为它就是一种更轻量级的虚拟机，这种认识其实是错误的，Docker和虚拟机有本质的区别。

容器本质上讲就是运行在操作系统上的一个进程，只不过加入了对资源的隔离和限制。而Docker是基于容器的这个设计思想，基于Linux Container技术实现的核心管理引擎。

## LXC

默认情况下，一个操作系统里所有运行的进程共享CPU和内存资源，如果程序设计不当，最极端的情况，某进程出现死循环可能会耗尽CPU资源，或者由于内存泄漏消耗掉大部分系统资源，这在企业级产品场景下是不可接受的，所以进程的资源隔离技术是非常必要的。

Linux操作系统本身从操作系统层面就支持虚拟化技术，叫做Linux container，也就是大家到处能看到的LXC的全称。

## LXC三大特色

LXC的三大特色：cgroup，namespace和unionFS。

### cgroup

CGroups 全称control group，用来限定一个进程的资源使用，由Linux 内核支持，可以限制和隔离Linux进程组 \(process groups\) 所使用的物理资源 ，比如cpu，内存，磁盘和网络IO，是Linux container技术的物理基础。

### namespace

另一个维度的资源隔离技术，CGroup设计的目的是为了隔离上面描述的物理资源，那么namespace则用来隔离PID\(进程ID\),IPC,Network等系统资源。可以将它们分配给特定的Namespace，每个Namespace里面的资源对其他Namespace都是透明的。不同container内的进程属于不同的Namespace，彼此透明，互不干扰。

如下图所示：父容器有两个子容器，父容器的命名空间里有两个进程，id分别为3和4, 映射到两个子命名空间后，分别成为其init进程，这样命名空间A和B的用户都认为自己独占整台服务器。

![](../../../../.gitbook/assets/image%20%2857%29.png)

Linux操作系统到目前为止支持的六种namespace：

![](../../../../.gitbook/assets/image%20%2856%29.png)

### unionFS

顾名思义，unionFS可以把文件系统上多个目录\(也叫分支\)内容联合挂载到同一个目录下，而目录的物理位置是分开的。要理解unionFS，我们首先要认识bootfs和rootfs。

#### 1. boot file system （bootfs）：

包含操作系统boot loader 和 kernel。用户不会修改这个文件系统。一旦启动完成后，整个Linux内核加载进内存，之后bootfs会被卸载掉，从而释放出内存。同样内核版本的不同的 Linux 发行版，其bootfs都是一致的。

#### 2. root file system （rootfs）

包含典型的目录结构，包括 /dev, /proc, /bin, /etc, /lib, /usr, and /tmp等再加上要运行用户应用所需要的所有配置文件，二进制文件和库文件。这个文件系统在不同的Linux 发行版中是不同的。而且用户可以对这个文件进行修改。

Linux 系统在启动时，roofs 首先会被挂载为只读模式，然后在启动完成后被修改为读写模式，随后它们就可以被修改了。

不同的Linux版本，实现unionFS的技术可能不一样，使用命令docker info查看，比如我的机器上实现技术是overlay2：

![](../../../../.gitbook/assets/image%20%28119%29.png)

看个实际的例子。

新建两个文件夹abap和java，在里面用touch命名分别创建两个空文件：

![](../../../../.gitbook/assets/image%20%28120%29.png)

新建一个mnt文件夹，用mount命令把abap和java文件夹merge到mnt文件夹下，-t执行文件系统类型为aufs：

sudo mount -t aufs -o dirs=./abap:./java none ./mnt

![](../../../../.gitbook/assets/image%20%28118%29.png)

mount完成后，到mnt文件夹下查看，发现了来自abap和java文件夹里总共4个文件：

![](../../../../.gitbook/assets/image%20%28117%29.png)

现在我到java文件夹里修改spring，比如加上一行spring is awesome, 然后到mnt文件夹下查看，发现mnt下面的文件内容也自动被更新了。

![](../../../../.gitbook/assets/image%20%28121%29.png)

![](../../../../.gitbook/assets/image%20%28115%29.png)

那么反过来会如何呢？比如我修改mnt文件夹下的aop文件：

![](../../../../.gitbook/assets/image%20%28116%29.png)

而java文件夹下的原始文件没有受到影响：

![](../../../../.gitbook/assets/image%20%28122%29.png)

实际上这就是Docker容器镜像分层实现的技术基础。如果我们浏览Docker hub，能发现大多数镜像都不是从头开始制作，而是从一些base镜像基础上创建，比如debian基础镜像。

而新镜像就是从基础镜像上一层层叠加新的逻辑构成的。这种分层设计，一个优点就是资源共享。

想象这样一个场景，一台宿主机上运行了100个基于debian base镜像的容器，难道每个容器里都有一份重复的debian拷贝呢？这显然不合理；借助Linux的unionFS，宿主机只需要在磁盘上保存一份base镜像，内存中也只需要加载一份，就能被所有基于这个镜像的容器共享。

当某个容器修改了基础镜像的内容，比如 /bin文件夹下的文件，这时其他容器的/bin文件夹是否会发生变化呢？

根据容器镜像的写时拷贝技术，某个容器对基础镜像的修改会被限制在单个容器内。

这就是我们接下来要学习的容器 Copy-on-Write 特性。

容器镜像由多个镜像层组成，所有镜像层会联合在一起组成一个统一的文件系统。如果不同层中有一个相同路径的文件，比如 /text，上层的 /text 会覆盖下层的 /text，也就是说用户只能访问到上层中的文件 /text。

假设我有如下这个dockerfile：

FROM debian

RUN apt-get install emacs

RUN apt-get install apache2

CMD \["/bin/bash"\]

执行docker build .看看发生了什么。

![](//upload-images.jianshu.io/upload_images/2085791-cca551302dfac53f?imageMogr2/auto-orient/strip|imageView2/2/w/1118/format/webp)

生成的容器镜像如下：

![](//upload-images.jianshu.io/upload_images/2085791-c8466de5394d2a40?imageMogr2/auto-orient/strip|imageView2/2/w/824/format/webp)

当用docker run启动这个容器时，实际上在镜像的顶部添加了一个新的可写层。这个可写层也叫容器层。

![](//upload-images.jianshu.io/upload_images/2085791-404d1347df2eb15a?imageMogr2/auto-orient/strip|imageView2/2/w/809/format/webp)

容器启动后，其内的应用所有对容器的改动，文件的增删改操作都只会发生在容器层中，对容器层下面的所有只读镜像层没有影响。  


