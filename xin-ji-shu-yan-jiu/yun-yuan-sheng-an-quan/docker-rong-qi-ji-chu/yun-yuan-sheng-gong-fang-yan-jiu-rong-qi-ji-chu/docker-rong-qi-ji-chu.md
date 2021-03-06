# Docker容器基础

## 使用虚拟机部署应用程序的年代

### **什么是虚拟化技术**

谈到计算机的虚拟化技术，我们直接想到的便是虚拟机，虚拟机允许我们在一台物理计算机模拟出多台机器，简单地理解，虚拟化技术就是在一台物理计算机上，通过中间虚拟软件层Hypervisor隔离CPU、内存等硬件资源，虚拟出多台虚拟服务器，这样做的话，一台物理服务器便可以安装多个应用程序，达到资源利用的最大化，而且多个应用之间相互隔离，如下图所示：

![&#x865A;&#x62DF;&#x673A;&#x4E0A;&#x90E8;&#x7F72;&#x5E94;&#x7528;&#x793A;&#x610F;&#x56FE;](http://dockone.io/uploads/article/20190822/744b54144d0800aacb09dcf16e03e71b.png)

### **虚拟机的优点**

* 可以把资源分配到不同的虚拟机，达到硬件资源的最大化利用
* 与直接在物理机上部署应用，虚拟更容易扩展应用。
* 云服务：通过虚拟机虚拟出不同的物理资源，可以快速搭建云服务。

### **虚拟机的不足之处**

虚拟机的不足之外来自于对物理服务器资源的消耗，当我们在物理服务器创建一台虚拟机时，便需要虚拟出一套硬件并在上面运行完整的操作系统，每台虚拟机都占用许多的服务器资源。  


## Docker是什么？

相对于虚拟机的笨重，Docker则更显得轻量化，因此不会占用太多的系统资源。  
  
Docker是使用时下很火的Golang语言进行开发的，其技术核心是Linux内核的Cgroup，Namespace和AUFS类的Union FS等技术，这些技术都是Linux内核中早已存在很多年的技术，所以严格来说并不是一个完全创新的技术，Docker通过这些底层的Linux技术，对Linux进程进行封装隔离，而被隔离的进程也被称为容器，完全独立于宿主机的进程。  
所以Docker是容器技术的一种实现，也是操作系统层面的一种虚拟化，与虚拟机的通过一套硬件再安装操作系统完全不同。

![Docker&#x5BB9;&#x5668;&#x4E0E;&#x7CFB;&#x7EDF;&#x5173;&#x7CFB;&#x793A;&#x610F;&#x56FE;](http://dockone.io/uploads/article/20190822/ade832620bdb618760c46cb1c6392ab8.png)

### **Docker与虚拟机之间的比较**

Docker是在操作系统进程层面的隔离，而虚拟机是在物理资源层面的隔离，两者完全不同，另外，我们也可以通过下面的一个比较，了解两者的根本性差异。  
从上面的容器与虚拟机的对比中，我们明白了容器技术的优势。

![&#x5BB9;&#x5668;&#x4E0E;&#x865A;&#x62DF;&#x673A;&#x7684;&#x6BD4;&#x8F83;&#x3010;&#x6458;&#x81EA;&#x300A;Docker-&#x4ECE;&#x5165;&#x95E8;&#x5230;&#x5B9E;&#x8DF5;&#x300B;&#x3011;](http://dockone.io/uploads/article/20190822/add19471351ab02a597dea48a976213b.png)

#### **容器解决了开发与生产环境的问题**

开发环境与生产环境折射的是开发人员与运维人员之间的矛盾，也许我们常常会听到开发人员对运维人员说的这样一句话：“在我的电脑运行没问题，怎么到了你那里就出问题了，肯定是你的问题”，而运维人员是认为是开发人员的问题。  
  
开发人员需要在本机安装各种各样的测试环境，因此开发的项目需要软件越多，依赖越多，安装的环境也就越复杂。  
  
同样的，运维人员需要为开发人员开发的项目提供生产环境，而运维人员除了应对软件之间的依赖，还需要考虑安装软件与硬件之间的兼容性问题。  
  
就是这样，所以我们经常看到开发与运维相互甩锅，怎么解决这个问题呢？  
  
容器就是一个不错的解决方案，容器能成为开发与运维之间沟通的语言，因为容器就像一个集装箱一样，提供了软件运行的最小化环境，将应用与其需要的环境一起打包成为镜像，便可以在开发与运维之间沟通与传输。

![5.jpg](http://dockone.io/uploads/article/20190822/4a2f457e3816bf6b41b74607cac7d811.jpg)

### Docker的版本

Docker分为社区版（CE）和企业版（EE）两个版本，社区版本可以免费使用，而企业版则需要付费使用，对于我们个人开发者或小企业来说，一般是使用社区版的。

Docker CE有三个更新频道，分别为stable、test、nightly，stable是稳定版本，test是测试后的预发布版本，而nightly则是开发中准备在下一个版本正式发布的版本，我们可以根据自己的需求下载安装。

### 如何安装Docker?

好了，通过前面的介绍，我们应该对Docker有了初步的了解，下面开始进入Docker的学习之旅了。  
  
而学习Docker的第一步，从安装Docker运行环境开始，我们以Docker的社区版本（CE）安装为例，Docker社区版本提供了Mac OS，Microsoft Windows和Linux（CentOS，Ubuntu，Fedora，Debian）等操作系统的安装包，同时也支持在云服务器上的安装，比如AWS Cloud。

### SUSE12安装docker

由于内网隔离，需要离线安装docker。

步骤如下：

互联网下载docker和docker-compose

docker下载地址：[https://download.docker.com/linux/static/stable/x86\_64/](https://download.docker.com/linux/static/stable/x86_64/)

docker-compose下载地址：[https://github.com/docker/compose/releases](https://github.com/docker/compose/releases)

选择的版本分别为 docker-18.06.3-ce.tgz 和 docker-compose-Linux-x86\_64

将下列代码保存为setDockerEnv.sh文件。

```bash
#!/bin/bash
set -x

#  本脚本为127.0.0.1执行

#setup
tar zxf  docker-18.06.3-ce.tgz && mv docker/* /usr/bin/ && rm -rf docker/

#systemd config
cat >/etc/systemd/system/docker.service <<-EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
EOF

#start
chmod +x /etc/systemd/system/docker.service
systemctl daemon-reload && systemctl start docker && systemctl enable docker.service

#testing
systemctl status docker && docker -v

# 安装docker-compose
chmod 777 docker-compose-Linux-x86_64

sudo mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose -v
```

### 使用官方安装脚本自动安装

安装命令如下：

```text
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

也可以使用国内 daocloud 一键安装命令：

```text
curl -sSL https://get.daocloud.io/docker | sh
```

## **在Linux上安装**

在Linux操作系统上的安装，主要以CentOS 7为例，其他Linux系统的发行版本，如Ubuntu，Debian，Fedora等，可以自行查询Docker的官方文档。  
  
**删除旧的Docker版本**  
  
可能有些Linux预先安装Docker，但一般版本比较旧，所以可以先执行以下代码来删除旧版本的Docker。  


```text
$ sudo dnf remove docker \
              docker-client \
              docker-client-latest \
              docker-common \
              docker-latest \
              docker-latest-logrotate \
              docker-logrotate \
              docker-selinux \
              docker-engine-selinux \
              docker-engine
```

###  **指定安装版本** 

```text
$ sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```

###  **使用yum安装Docker** 

```text
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

###  **启动Docker服务器** 

```text
# 启动docker守护进程
$ sudo systemctl start docker
```

### **测试安装是否成功**

#### 通过上面几种方式安装了Docker之后，我们可以通过下面的方法来检测安装是否成功。  **打印Docker版本** 

```text
# 打印docker版本
$ docker version 
```

####  **拉取镜像并运行容器** 

```text
# 拉取hello-world镜像
docker pull hello-world

# 使用hello-world运行一个容器
docker run hello-world
```

  
运行上面的命令之后，如果有如下图所示的输出结果，则说明安装已经成功了。  


![9.png](http://dockone.io/uploads/article/20190822/bbeb0a5d66bf409d130f8b24689a307b.png)

## Docker的基本概念

镜像（Image）、容器（Container）与仓库（Repository），这三个是Docker中最基本也是最核心的概念，对这三个概念的掌握与理解，是学习Docker的关键。  


### **镜像（Image）**

什么是Docker的镜像？  
  
Docker本质上是一个运行在Linux操作系统上的应用，而Linux操作系统分为内核和用户空间，无论是CentOS还是Ubuntu，都是在启动内核之后，通过挂载Root文件系统来提供用户空间的，而Docker镜像就是一个Root文件系统。  
  
Docker镜像是一个特殊的文件系统，提供容器运行时所需的程序、库、资源、配置等文件，另外还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。  
  
镜像是一个静态的概念，不包含任何动态数据，其内容在构建之后也不会被改变。  
  
下面的命令是一些对镜像的基本操作，如下：  
  
**查看镜像列表**  


```text
# 列出所有镜像
docker image ls
```

  
由于我们前面已经拉取了hello-world镜像，所以会输出下面的内容：

```text
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
hello-world             latest              fce289e99eb9        7 months ago        1.84kB
```

  
下面的命令也一样可以查看本地的镜像列表，而且写法更简洁。

```text
# 列表所有镜像
docker images
```

  
**从仓库拉取镜像**  
  
前面我们已经演示过使用docker pull命令拉取了hello-world镜像了，当然使用docker image pull命令也是一样的。  
  
一般默认是从Docker Hub上拉取镜像的，Docker Hub是Docker官方提供的镜像仓库服务（Docker Registry），有大量官方或第三方镜像供我们使用，比如我们可以在命令行中输入下面的命令直接拉取一个CentOS镜像：  


```text
docker pull centos
```

  
docker pull命令的完整写法如下：  


```text
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

  
拉取一个镜像，需要指定Docker Registry的地址和端口号，默认是Docker Hub，还需要指定仓库名和标签，仓库名和标签唯一确定一个镜像，而标签是可能省略，如果省略，则默认使用latest作为标签名，另外，仓库名则由作者名和软件名组成。  
  
那么，我们上面使用CentOS，那是因为省略作者名，则作者名library，表示Docker官方的镜像，所以上面的命令等同于：  


```text
docker pull library/centos:latest
```

  
因此，如果拉取非官方的第三方镜像，则需要指定完整仓库名，如下：  


```text
docker pull mysql/mysql-server:latest
```

  
**运行镜像**  
  
使用docker run命令，可以通过镜像创建一个容器，如下：  


```text
docker run -it centos /bin/bash
```

  
**删除镜像**  
  
当本地有些镜像我们不需要时，那我们也可以删除该镜像，以节省存储空间，不过要注意，如果有使用该镜像创建的容器未删除，则不允许删除镜像。  


```text
# image_name表示镜像名，image_id表示镜像id
dockere image rm image_name/image_id
```

  
删除镜像的快捷命令：  


```text
docker rmi image_name/image_id
```

  
好了，关于Docker镜像的相关知识，我们就简单地介绍到这里，有机会的话，我们单独写一篇文章来谈谈，特别构建Docker镜像部分的相关知识，有必要深入再学习一下。

### **容器（Container）**

Docker的镜像是用于生成容器的模板，镜像分层的，镜像与容器的关系，就是面向对象编程中类与对象的关系，我们定好每一个类，然后使用类创建对象，对应到Docker的使用上，则是构建好每一个镜像，然后使用镜像创建我们需要的容器。  
  
**启动和停止容器**  
  
启动容器有两种方式，一种是我们前面已经介绍过的，使用docker run命令通过镜像创建一个全新的容器，如下：

```text
docker run hello-world
```

  
另外一种启动容器的方式就是启动一个已经停止运行的容器：

```text
# container_id表示容器的id
docker start container_id
```

  
要停止正在运行的容器可以使用docker container stop或docker stop命令，如下：

```text
# container_id表示容器的id
docker stop container_id
```

  
**查看所有容器**  
  
如果要查看本地所有的容器，可以使用docker container ls命令：

```text
# 查看所有容器
docker container ls
```

  
查看所有容器也有简洁的写法，如下：

```text
# 查看所有容器
docker ps
```

  
**删除容器**  
  
我们也可以使用docker container rm命令，或简洁的写法docker rm命令来删除容器，不过不允许删除正在运行的容器，因此如果要删除的话，就必须先停止容器。

```text
# container_id表示容器id，通过docker ps可以看到容器id
$ docker rm container_id
```

  
当我们需要批量删除所有容器，可以用下面的命令：

```text
# 删除所有容器
docker rm $(docker ps -q)
```



 当我们需要批量删除所有容器，可以用下面的命令：

```text
# 删除所有容器
docker rm $(docker ps -q)
```

```text
# 删除所有退出的容器
docker container prune
```

  
**进入容器**

```text
# 进入容器，container_id表示容器的id，command表示Linux命令，如/bin/bash
docker exec -it container_id command
```

### **仓库（Repository）**

在前面的例子中，我们使用两种方式构建镜像，构建完成之后，可以在本地运行镜像，生成容器，但如果在更多的服务器运行镜像呢？很明显，这时候我们需要一个可以让我们集中存储和分发镜像的服务，就像Github可以让我们自己存储和分发代码一样。  
  
Docker Hub就是Docker提供用于存储和分布镜像的官方Docker Registry，也是默认的Registry，其网址为[https://hub.docker.com](https://hub.docker.com/)，前面我们使用docker pull命令便从Docker Hub上拉取镜像。  
  
Docker Hub有很多官方或其他开发提供的高质量镜像供我们使用，当然，如果要将我们自己构建的镜像上传到Docker Hub上，我们需要在Docker Hub上注册一个账号，然后把自己在本地构建的镜像发送到Docker Hub的仓库当中，Docker Registry包含很多个仓库，每个仓库对应多个标签，不同标签对应一个软件的不同版本。

## 使用 docker-compose 替代 docker run

### 使用 docker run 运行镜像 <a id="&#x4F7F;&#x7528;-docker-run-&#x8FD0;&#x884C;&#x955C;&#x50CF;"></a>

要运行一个 docker 镜像， 通常都是使用 `docker run` 命令， 在运行的镜像的时候， 需要指定一些参数， 例如：容器名称、 映射的卷、 绑定的端口、 网络以及重启策略等等， 一个典型的 `docker run` 命令如下所示：

```text
docker run \
  --detach \
  --name registry \
  --hostname registry \
  --volume $(pwd)/app/registry:/var/lib/registry \
  --publish 5000:5000 \
  --restart unless-stopped \
  registry:latest
```

为了保存这些参数， 可以将这个 `run` 命令保存成 shell 文件， 需要时可以重新运行 shell 文件。 对于只有单个镜像的简单应用， 基本上可以满足需要了。 只要保存对应的 shell 文件， 备份好卷的内容， 当容器出现问题或者需要迁移活着需要重新部署时， 使用 shell 文件就可以快速完成。

不过不是所有的应用都倾向于做成单个镜像， 这样的镜像会非常复杂， 而且不符合 docker 的思想。 因为 docker 更倾向于简单镜像， 即： 一个镜像只有一个进程。 一个典型的 web 应用， 至少需要一个 web 服务器来运行服务端程序， 同时还需要一个数据库服务器来完成数据的存储， 这就需要两个镜像， 一个是 web ， 一个是 db ， 如果还是按照上面的做法， 需要两个 shell 文件， 或者是在一个 shell 文件中有两个 `docker run` 命令：

```text
# PostGIS DB
docker run \
  --datach \
  --publish 5432:5432 \
  --name postgis \
  --restart unless-stopped \
  --volume $(pwd)/db/data:/var/lib/postgresql/data \
  beginor/postgis:9.3

# GeoServer Web
docker run \
  --detach \
  --publish 8080:8080 \
  --name geoserver \
  --restart unless-stopped \
  --volume $(pwd)/geoserver/data_dir:/geoserver/data_dir \
  --volume $(pwd)/geoserver/logs:/geoserver/logs \
  --hostname geoserver \
  --link postgis:postgis \
  beginor/geoserver:2.11.0
```

在上面的例子中， web 服务器使用的是 geoserver ， db 服务器使用的是 postgis ， web 服务器依赖 db 服务器， 必须先启动 db 服务器， 再启动 web 服务器， 这就需要编写复杂的 shell 脚本， 需要的镜像越多， 脚本越复杂， 这个问题被称作 docker 的编排。

> 关于 docker run 的各个参数的使用方法， 请参阅 docker 网站的[说明文档](https://docs.docker.com/edge/engine/reference/commandline/run/)。

### 使用 docker-compose 编排镜像 <a id="&#x4F7F;&#x7528;-docker-compose-&#x7F16;&#x6392;&#x955C;&#x50CF;"></a>

docker 提供了一个命令行工具 `docker-compose` 帮助完成镜像的编排， 要使用 `docker-compose` ， 需要先编写一个 `docker-compose.yml` 文件， `yaml` 是一种常用配置文件格式， 维基百科中对 `yaml` 描述如下：

> YAML 是一个可读性高，用来表达数据序列的格式。YAML参考了其他多种语言，包括：C语言、Python、Perl，并从XML、电子邮件的数据格式（RFC 2822）中获得灵感。

如果想了解详细信息， 请[参考 YAML 官方网站](https://yaml.org/)或者[维基百科](https://zh.wikipedia.org/wiki/YAML)。

docker 网站上提供了 docker-compose 的[入门教程](https://docs.docker.com/compose/gettingstarted/)， 如果不熟悉的话可以去学习一下。

上面的脚本转换成对应的 docker-compose.yml 文件如下所示：

```text
version: "3"
services:
  web:
    image: beginor/geoserver:2.11.1
    container_name: geoserver-web
    hostname: geoserver-web
    ports:
      - 8080:8080
    volumes:
      - ./web/data_dir:/geoserver/data_dir
      - ./web/logs:/geoserver/logs
    restart: unless-stopped
    links:
      - database:database
  database:
    image: beginor/postgis:9.3
    container_name: postgis
    hostname: postgis
    ports:
      - 5432:5432
    volumes:
      - ./database/data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: 1q2w3e4R
    restart: unless-stopped
```

上面的 `docker-compose.yml` 文件定义了两个服务 web 和 database， 一个服务在运行时对应一个容器的实例， 上面的文件表示要启动两个实例。

在部署时， 通常将 `docker-compose.yml` 文件放到一个目录， 表示一个应用， docker 会为这个应用创建一个独立的网络， 便于和其它应用进行隔离。

要运行这个程序， 只要在这个目录下执行 `docker-compose up -d` 命令， 就会按照上面的配置启动两个容器的实例:

```text
$ docker-compose up -d
Creating network "geoserver_default" with the default driver
Creating geoserver_database_1
Creating geoserver_web_1
```

要停止上面的容器， 只需要输入 `docker-compose down` 命令：

```text
$ docker-compose down
Stopping geoserver_web_1 ... done
Stopping geoserver_database_1 ... done
Removing geoserver_web_1 ... done
Removing geoserver_database_1 ... done
Removing network geoserver_default
```

从上面的命令可以看出， docker-compose 不仅可以根据配置文件 `docker-compose.yml` 自动创建网络， 启动响应的容器实例， 也可以根据配置文件删除停止和删除容器实例， 并删除对应的网络， 确实是 `docker run` 命令更加方便， 因此推荐在测试环境或者生产环境中使用。

## Docker的组成与架构

### Docker架构

#### a、Docker客户端和服务端

  Docker是客户－服务器\(C/S\)架构的程序。Docker客户端只需向Docker服务器或守护进程发出请求，服务器或守护进程将完成所有工作并返回结果。Docker提供了一个命令行工具docker以及一整套RESTful API。你可以在同一台宿主机上运行Docker守护进程和客户端，也可以从本地的Docker客户端连接到运行在另一台宿主机上的远程Docker守护进程。下图描绘了Docker的架构：  


![](../../../../.gitbook/assets/image%20%28114%29.png)

### docker 启动的调用链

docker-client -&gt; dockerd -&gt; docker-containerd -&gt; docker-containerd-shim -&gt; runc（容器外） -&gt; runc（容器内） -&gt; containter-entrypoint

安装 docker ，其实是安装了 docker 客户端、dockerd 等一系列的组件，其中比较重要的有下面几个。

**Docker CLI\(docker\)**  
docker 程序是一个客户端工具，用来把用户的请求发送给 docker daemon\(dockerd\)。该程序的安装路径为：

```text
/usr/bin/docker
```

**Dockerd**  
docker daemon\(dockerd\)，一般也会被称为 docker engine。该程序的安装路径为：

```text
/usr/bin/dockerd
```

**Containerd**  
详情请参考《[Containerd 简介](http://www.cnblogs.com/sparkdev/p/9063042.html)》。该程序的安装路径为：

```text
/usr/bin/docker-containerd
```

**Containerd-shim**  
它是 containerd 的组件，是容器的运行时载体，我们在 docker 宿主机上看到的 shim 也正是代表着一个个通过调用 containerd 启动的 docker 容器。该程序的安装路径为：

```text
/usr/bin/docker-containerd-shim
```

**RunC**  
详情请参考《[RunC 简介](http://www.cnblogs.com/sparkdev/p/9032209.html)》。该程序的安装路径为：

```text
/usr/bin/docker-runc
```

从Docker 1.11之后，Docker Daemon被分成了多个模块以适应OCI标准。拆分之后，结构分成了以下几个部分。

其中，containerd独立负责容器运行时和生命周期（如创建、启动、停止、中止、信号处理、删除等），其他一些如镜像构建、卷管理、日志等由Docker Daemon的其他模块处理。  


![](../../../../.gitbook/assets/image%20%28113%29.png)

Docker的模块块拥抱了开放标准，希望通过OCI的标准化，容器技术能够有很快的发展。

这是一个很抽象也很容器理解的过程，但是我们还想知道更多：docker daemon 是如何创建并运行容器的？  
其实容器部分的操作和管理都被 dockerd 外包给 containerd 了，下图描述了运行一个容器时各个组件之间的关系：

![](../../../../.gitbook/assets/image%20%28108%29.png)

#### b、Docker架构图

![](../../../../.gitbook/assets/image%20%28112%29.png)

![](../../../../.gitbook/assets/image%20%28111%29.png)

#### c、Docker run 运行流程图

![](../../../../.gitbook/assets/image%20%28110%29.png)

在安装好并启动了Docker之后，我们可以使用在命令行中使用Docker命令操作Docker，比如我们使用如下命令打印Docker的版本信息。

```text
docker verion
```

  
其结果如下：

![10.png](http://dockone.io/uploads/article/20190822/eb4d2cdd28df19ab7b0f27349ebd6c8d.png)

  
从上面的图中，我们看到打出了两个部分的信息：Client和Server。  
  
这是因为Docker跟大部分服务端软件一样（如MySQL），都是使用C/S的架构模型，也就是通过客户端调用服务器，只是我们现在刚好服务端和客户端都在同一台机器上而已。  
  
因此，我们可以使用下面的图来表示Docker的架构，DOCKER\_HOST是Docker Server，而Client便是我们在命令中使用Docker命令。

![11.png](http://dockone.io/uploads/article/20190822/b49136d13143db9013b25e99416b75c6.png)

**Docker Engine**

Docker Server为客户端提供了容器、镜像、数据卷、网络管理等功能，其实，这些功能都是由Docker Engine来实现的。

1. dockerd：服务器守护进程。
2. Client docker Cli：命令行接口
3. REST API：除了cli命令行接口，也可以通过REST API调用Docker

  
下面是Docker Engine的示例图：  
  


![12.png](http://dockone.io/uploads/article/20190822/c6df70f0762eb7f84ceec5119ec03936.png)

## 小结

作为一名开发人员，在学习或开发过程中，总需要安装各种各样的开发环境，另外，一个技术团队在开发项目的过程，也常常需要统一开发环境，这样可能避免环境不一致引发的一些问题。  
  
虽然使用虚拟机可以解决上面的问题，但虚拟机太重，对宿主机资源消耗太大，而作为轻量级容器技术，Docker可以简单轻松地解决上述问题，让开发环境的安装以及应用的部署变得非常简单，而且使用Docker，比在虚拟机安装操作系统，要简单得多。  
  
链接：[https://juejin.im/post/5d4522c1f265da03e05af5f5](https://juejin.im/post/5d4522c1f265da03e05af5f5)，作者：张君鸿

