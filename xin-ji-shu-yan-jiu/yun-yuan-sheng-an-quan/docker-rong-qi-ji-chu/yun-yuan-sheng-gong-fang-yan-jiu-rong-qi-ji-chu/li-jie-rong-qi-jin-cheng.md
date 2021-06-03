# 理解容器进程

以下内容来自「[理解Docker容器的进程管理](https://www.cnblogs.com/ilinuxer/p/6188303.html)」，我看了之后深受启发，也由此开启了docker的研究之路，具体内容如下：

## 开始正文

Docker在进程管理上有一些特殊之处，如果不注意这些细节中的魔鬼就会带来一些隐患。另外Docker鼓励“一个容器一个进程\(one process per container\)”的方式。这种方式非常适合以单进程为主的微服务架构的应用。然而由于一些传统的应用是由若干紧耦合的多个进程构成的，这些进程难以拆分到不同的容器中，所以在单个容器内运行多个进程便成了一种折衷方案；此外在一些场景中，用户期望利用Docker容器来作为轻量级的虚拟化方案，动态的安装配置应用，这也需要在容器中运行多个进程。而在Docker容器中的正确运行多进程应用将给开发者带来更多的挑战

今天我们会分析Docker中进程管理的一些细节，并介绍一些常见问题的解决方法和注意事项。

#### 容器的PID namespace（名空间） <a id="1"></a>

在Docker中，进程管理的基础就是Linux内核中的PID名空间技术。在不同PID名空间中，进程ID是独立的；即在两个不同名空间下的进程可以有相同的PID。

Linux内核为所有的PID名空间维护了一个树状结构：最顶层的是系统初始化时创建的root namespace（根名空间），再创建的新PID namespace就称之为child namespace（子名空间），而原先的PID名空间就是新创建的PID名空间的parent namespace（父名空间）。通过这种方式，系统中的PID名空间会形成一个层级体系。父节点可以看到子节点中的进程，并可以通过信号等方式对子节点中的进程产生影响。反过来，子节点不能看到父节点名空间中的任何内容，也不可能通过kill或ptrace影响父节点或其他名空间中的进程。

在Docker中，每个Container都是Docker Daemon的子进程，每个Container进程缺省都具有不同的PID名空间。通过名空间技术，Docker实现容器间的进程隔离。另外Docker Daemon也会利用PID名空间的树状结构，实现了对容器中的进程交互、监控和回收。注：Docker还利用了其他名空间（UTS，IPC，USER）等实现了各种系统资源的隔离，由于这些内容和进程管理关联不多，本文不会涉及。

当创建一个Docker容器的时候，就会新建一个PID名空间。容器启动进程在该名空间内PID为1。当PID1进程结束之后，Docker会销毁对应的PID名空间，并向容器内所有其它的子进程发送SIGKILL。

下面我们来做一些试验，下面我们会利用官方的Redis镜像创建两个容器，并观察里面的进程。  
如果你在Windows或Mac上利用"docker-machine"，请利用`docker-machine ssh default`进入Boot2docker虚拟机

创建名为"redis"的容器，并在容器内部和宿主机中查看容器中的进程信息

```text
docker@default:~$ docker run -d --name redis redis
f6bc57cc1b464b05b07b567211cb693ee2a682546ed86c611b5d866f6acc531c
docker@default:~$ docker exec redis ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
redis        1     0  0 01:49 ?        00:00:00 redis-server *:6379
root        11     0  0 01:49 ?        00:00:00 ps -ef
docker@default:~$ docker top redis
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
999                 9302                1264                0                   01:49               ?                   00:00:00            redis-server *:6379
```

创建名为"redis2"的容器，并在容器内部和宿主机中查看容器中的进程信息

```text
docker@default:~$ docker run -d --name redis2 redis
356eca186321ab6ef4c4337aa0c7de2af1e01430587d6b0e1add2e028ed05f60
docker@default:~$ docker exec redis2 ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
redis        1     0  0 01:50 ?        00:00:00 redis-server *:6379
root        10     0  4 01:50 ?        00:00:00 ps -ef
docker@default:~$ docker top redis2
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
999                 9342                1264                0                   01:50               ?                   00:00:00            redis-server *:6379
```

我们可以使用`docker exec`命令进入容器PID名空间，并执行应用。通过`ps -ef`命令，可以看到每个Redis容器都包含一个PID为1的进程，"redis-server"，它是容器的启动进程，具有特殊意义。

利用`docker top`命令，可以让我们从宿主机操作系统中看到容器的进程信息。在两个容器中的"redis-server"是两个独立的进程，但是他们拥有相同的父进程 Docker Daemon。所以Docker可以父子进程的方式在Docker Daemon和Redis容器之间进行交互。

另一个值得注意的方面是，`docker exec`命令可以进入指定的容器内部执行命令。由它启动的进程属于容器的namespace和相应的cgroup。但是这些进程的父进程是Docker Daemon而非容器的PID1进程。

我们下面会在Redis容器中，利用`docker exec`命令启动一个"sleep"进程

```text
docker@default:~$ docker exec -d redis sleep 2000
docker@default:~$ docker exec redis ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
redis        1     0  0 02:26 ?        00:00:00 redis-server *:6379
root        11     0  0 02:26 ?        00:00:00 sleep 2000
root        21     0  0 02:29 ?        00:00:00 ps -ef
docker@default:~$ docker top redis
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
999                 9955                1264                0                   02:12               ?                   00:00:00            redis-server *:6379
root                9984                1264                0                   02:13               ?                   00:00:00            sleep 2000
```

我们可以清楚的看到exec命令创建的sleep进程属Redis容器的名空间，但是它的父进程是Docker Daemon。

如果我们在宿主机操作系统中手动杀掉容器的启动进程（在上文示例中是redis-server），容器会自动结束，而容器名空间中所有进程也会退出。

```text
docker@default:~$ PID=$(docker inspect --format="{{.State.Pid}}" redis)
docker@default:~$ sudo kill $PID
docker@default:~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
356eca186321        redis               "/entrypoint.sh redis"   23 minutes ago      Up 4 minutes               6379/tcp            redis2
f6bc57cc1b46        redis               "/entrypoint.sh redis"   23 minutes ago      Exited (0) 4 seconds ago                       redis
```

通过以上示例：

* 每个容器有独立的PID名空间，
* 容器的生命周期和其PID1进程一致
* 利用`docker exec`可以进入到容器的名空间中启动进程

此外，自从Docker 1.5之后，`docker run`命令引入了`--pid=host`参数来支持使用宿主机PID名空间来启动容器进程，这样可以方便的实现容器内应用和宿主机应用之间的交互：比如利用容器中的工具监控和调试宿主机进程。

#### 如何指明容器PID1进程 <a id="2"></a>

在Docker容器中的初始化进程（PID1进程）在容器进程管理上具有特殊意义。它可以被Dockerfile中的`ENTRYPOINT`或`CMD`指令所指明；也可以被`docker run`命令的启动参数所覆盖。了解这些细节可以帮助我们更好地了解PID1的进程的行为。

关于ENTRYPOINT和CMD指令的不同，我们可以参见官方的Dockerfile说明和最佳实践

* [https://docs.docker.com/engine/reference/builder/\#entrypoint](https://docs.docker.com/engine/reference/builder/?spm=5176.100239.blogcont5545.13.qOGovX#entrypoint)
* [https://docs.docker.com/engine/reference/builder/\#cmd](https://docs.docker.com/engine/reference/builder/?spm=5176.100239.blogcont5545.14.qOGovX#cmd)

值得注意的一点是：在ENTRYPOINT和CMD指令中，提供两种不同的进程执行方式 shell 和 exec

在 shell 方式中，CMD/ENTRYPOINT指令以如下方式定义

```text
CMD executable param1 param2
```

这种方式中的PID1进程是以`/bin/sh -c ”executable param1 param2”`方式启动的

而在 exec 方式中，CMD/ENTRYPOINT指令以如下方式定义

```text
CMD ["executable","param1","param2"]
```

注意这里的可执行命令和参数是利用JSON字符串数组的格式定义的，这样PID1进程会以 `executable param1 param2` 方式启动的。另外，在`docker run`命令中指明的命令行参数也是以 exec 方式启动的。

为了解释两种不同运行方式的区别，我们利用不同的Dockerfile分别创建两个Redis镜像

"Dockerfile\_shell"文件内容如下，会利用shell方式启动redis服务

```text
FROM ubuntu:14.04
RUN apt-get update && apt-get -y install redis-server && rm -rf /var/lib/apt/lists/*
EXPOSE 6379
CMD "/usr/bin/redis-server"
```

"Dockerfile\_exec"文件内容如下，会利用exec方式启动redis服务

```text
FROM ubuntu:14.04
RUN apt-get update && apt-get -y install redis-server && rm -rf /var/lib/apt/lists/*
EXPOSE 6379
CMD ["/usr/bin/redis-server"]
```

然后基于它们构建两个镜像"myredis:shell"和"myredis:exec"

```text
docker build -t myredis:shell -f Dockerfile_shell .
docker build -t myredis:exec -f Dockerfile_exec .
```

运行"myredis:shell"镜像，我们可以发现它的启动进程\(PID1\)是`/bin/sh -c "/usr/bin/redis-server"`，并且它创建了一个子进程`/usr/bin/redis-server *:6379`。

```text
docker@default:~$ docker run -d --name myredis myredis:shell
49f7fc37f4b7cf1ed7f5296537a93b2ad23b1b6686a05e5c7e40e9a2b2d3665e
docker@default:~$ docker exec myredis ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 08:12 ?        00:00:00 /bin/sh -c "/usr/bin/redis-server"
root         5     1  0 08:12 ?        00:00:00 /usr/bin/redis-server *:6379
root         8     0  0 08:12 ?        00:00:00 ps -ef
```

下面运行"myredis:exec"镜像，我们可以发现它的启动进程是`/usr/bin/redis-server *:6379`，并没有其他子进程存在。

```text
docker@default:~$ docker run -d --name myredis2 myredis:exec
d1df0e4f4e3bbe36fca94f08df9ad3306fa1dee86415c853ddc5593fb9fa5673
docker@default:~$ docker exec myredis2 ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 08:13 ?        00:00:00 /usr/bin/redis-server *:6379
root         8     0  0 08:13 ?        00:00:00 ps -ef
```

由此我们可以清楚的看到，以exec和shell方式执行命令可能会导致容器的PID1进程不同。然而这又有什么问题呢？

原因在于：PID1进程对于操作系统而言具有特殊意义。操作系统的PID1进程是init进程，以守护进程方式运行，是所有其他进程的祖先，具有完整的进程生命周期管理能力。在Docker容器中，PID1进程是启动进程，它也会负责容器内部进程管理的工作。而这也将导致进程管理在Docker容器内部和完整操作系统上的不同。

#### 进程信号处理 <a id="3"></a>

信号是Unix/Linux中进程间异步通信机制。Docker提供了两个命令`docker stop`和`docker kill`来向容器中的PID1进程发送信号。

当执行`docker stop`命令时，docker会首先向容器的PID1进程发送一个SIGTERM信号，用于容器内程序的退出。如果容器在收到SIGTERM后没有结束， 那么Docker Daemon会在等待一段时间（默认是10s）后，再向容器发送SIGKILL信号，将容器杀死变为退出状态。这种方式给Docker应用提供了一个优雅的退出\(graceful stop\)机制，允许应用在收到stop命令时清理和释放使用中的资源。而`docker kill`可以向容器内PID1进程发送任何信号，缺省是发送SIGKILL信号来强制退出应用。

注：从Docker 1.9开始，Docker支持停止容器时向其发送自定义信号，开发者可以在Dockerfile使用`STOPSIGNAL`指令，或`docker run`命令中使用`--stop-signal`参数中指明。缺省是`SIGTERM`

我们来看看不同的PID1进程，对进程信号处理的不同之处。首先，我们使用`docker stop`命令停止由 exec 模式启动的“myredis2”容器，并检查其日志

```text
docker@default:~$ docker stop myredis2
myredis2
docker@default:~$ docker logs myredis2
[1] 11 Feb 08:13:01.631 # Warning: no config file specified, using the default config. In order to specify a config file use /usr/bin/redis-server /path/to/redis.conf
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 2.8.4 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in stand alone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 1
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

[1] 11 Feb 08:13:01.632 # Server started, Redis version 2.8.4
[1] 11 Feb 08:13:01.633 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
[1] 11 Feb 08:13:01.633 * The server is now ready to accept connections on port 6379
[1 | signal handler] (1455179074) Received SIGTERM, scheduling shutdown...
[1] 11 Feb 08:24:34.259 # User requested shutdown...
[1] 11 Feb 08:24:34.259 * Saving the final RDB snapshot before exiting.
[1] 11 Feb 08:24:34.262 * DB saved on disk
[1] 11 Feb 08:24:34.262 # Redis is now ready to exit, bye bye...
docker@default:~$ 
```

我们发现对“myredis2”容器的stop命令几乎立刻生效；而且在容器日志中，我们看到了“Received SIGTERM, scheduling shutdown...”的内容，说明“redis-server”进程接收到了SIGTERM消息，并优雅地退出。

我们再对利用 shell 模式启动的“myredis”容器发出停止操作，并检查其日志

```text
docker@default:~$ docker stop myredis
myredis
docker@default:~$ docker logs myredis
[5] 11 Feb 08:12:40.108 # Warning: no config file specified, using the default config. In order to specify a config file use /usr/bin/redis-server /path/to/redis.conf
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 2.8.4 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in stand alone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 5
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

[5] 11 Feb 08:12:40.109 # Server started, Redis version 2.8.4
[5] 11 Feb 08:12:40.109 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
[5] 11 Feb 08:12:40.109 * The server is now ready to accept connections on port 6379
docker@default:~$
```

我们发现对”myredis”容器的stop命令暂停了一会儿才结束，而且在日志中我们没有看到任何收到SIGTERM信号的内容。原因其PID1进程sh没有对SIGTERM信号的处理逻辑，所以它忽略了所接收到的SIGTERM信号。当Docker等待stop命令执行10秒钟超时之后，Docker Daemon发送SIGKILL强制杀死sh进程，并销毁了它的PID名空间，其子进程redis-server也在收到SIGKILL信号后被强制终止。如果此时应用还有正在执行的事务或未持久化的数据，强制进程退出可能导致数据丢失或状态不一致。

通过这个示例我们可以清楚的理解PID1进程在信号管理的重要作用。所以，

* 容器的PID1进程需要能够正确的处理SIGTERM信号来支持优雅退出。
* 如果容器中包含多个进程，需要PID1进程能够正确的传播SIGTERM信号来结束所有的子进程之后再退出。
* 确保PID1进程是期望的进程。缺省sh/bash进程没有提供SIGTERM的处理，需要通过shell脚本来设置正确的PID1进程，或捕获SIGTERM信号。

另外需要注意的是：由于PID1进程的特殊性，Linux内核为他做了特殊处理。如果它没有提供某个信号的处理逻辑，那么与其在同一个PID名空间下的进程发送给它的该信号都会被屏蔽。这个功能的主要作用是防止init进程被误杀。我们可以验证在容器内部发出的SIGKILL信号无法杀死PID1进程

```text
docker@default:~$ docker start myredis
myredis
docker@default:~$ docker exec myredis kill -9 1
docker@default:~$ docker top myredis
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                3586                1290                0                   08:45               ?                   00:00:00            /bin/sh -c "/usr/bin/redis-server"
root                3591                3586                0                   08:45               ?                   00:00:00            /usr/bin/redis-server *:6379
```

#### 孤儿进程与僵尸进程管理 <a id="4"></a>

熟悉Unix/Linux进程管理的同学对多进程应用并不陌生。

当一个子进程终止后，它首先会变成一个“失效\(defunct\)”的进程，也称为“僵尸（zombie）”进程，等待父进程或系统收回（reap）。在Linux内核中维护了关于“僵尸”进程的一组信息（PID，终止状态，资源使用信息），从而允许父进程能够获取有关子进程的信息。如果不能正确回收“僵尸”进程，那么他们的进程描述符仍然保存在系统中，系统资源会缓慢泄露。

大多数设计良好的多进程应用可以正确的收回僵尸子进程，比如NGINX master进程可以收回已终止的worker子进程。如果需要自己实现，则可利用如下方法：  
1. 利用操作系统的waitpid\(\)函数等待子进程结束并请除它的僵死进程，  
2. 由于当子进程成为“defunct”进程时，父进程会收到一个SIGCHLD信号，所以我们可以在父进程中指定信号处理的函数来忽略SIGCHLD信号，或者自定义收回处理逻辑。

下面这些文章详细介绍了对僵尸进程的处理方法

* [http://www.microhowto.info/howto/reap\_zombie\_processes\_using\_a\_sigchld\_handler.html](http://www.microhowto.info/howto/reap_zombie_processes_using_a_sigchld_handler.html?spm=5176.100239.blogcont5545.15.qOGovX)
* [http://lbolla.info/blog/2014/01/23/die-zombie-die](http://lbolla.info/blog/2014/01/23/die-zombie-die?spm=5176.100239.blogcont5545.16.qOGovX)

如果父进程已经结束了，那些依然在运行中的子进程会成为“孤儿（orphaned）”进程。在Linux中Init进程\(PID1\)作为所有进程的父进程，会维护进程树的状态，一旦有某个子进程成为了“孤儿”进程后，init就会负责接管这个子进程。当一个子进程成为“僵尸”进程之后，如果其父进程已经结束，init会收割这些“僵尸”，释放PID资源。

然而由于Docker容器的PID1进程是容器启动进程，它们会如何处理那些“孤儿”进程和“僵尸”进程？

下面我们做几个试验来验证不同的PID1进程对僵尸进程不同的处理能力

首先在myredis2容器中启动一个bash进程，并创建子进程“sleep 1000”

```text
docker@default:~$ docker restart myredis2
myredis2
docker@default:~$ docker exec -ti myredis2 bash
root@d1df0e4f4e3b:/# sleep 1000

```

在另一个终端窗口，查看当前进程，我们可以发现一个sleep进程是bash进程的子进程。

```text
docker@default:~$ docker exec myredis2 ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 12:21 ?        00:00:00 /usr/bin/redis-server *:6379
root         8     0  0 12:21 ?        00:00:00 bash
root        21     8  0 12:21 ?        00:00:00 sleep 1000
root        22     0  3 12:21 ?        00:00:00 ps -ef
```

我们杀死bash进程之后查看进程列表，这时候bash进程已经被杀死。这时候sleep进程\(PID为21\)，虽然已经结束，而且被PID1进程（redis-server）接管，但是其没有被父进程回收，成为僵尸状态。

```text
docker@default:~$ docker exec myredis2 kill -9 8
docker@default:~$ docker exec myredis2 ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 12:09 ?        00:00:00 /usr/bin/redis-server *:6379
root        21     1  0 12:10 ?        00:00:00 [sleep] <defunct>
root        32     0  0 12:10 ?        00:00:00 ps -ef
docker@default:~$ 

```

这是因为PID1进程“redis-server”没有考虑过作为init对僵尸子进程的回收的场景。

我们来做另一个试验，在用/bin/sh作为PID1进程的myredis容器中，再启动一个bash进程，并创建子进程“sleep 1000”

```text
docker@default:~$ docker start myredis
myredis
docker@default:~$ docker exec -ti myredis bash
root@49f7fc37f4b7:/# sleep 1000
```

查看容器中进程情况，

```text
docker@default:~$ docker exec myredis ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 01:29 ?        00:00:00 /bin/sh -c "/usr/bin/redis-server"
root         5     1  0 01:29 ?        00:00:00 /usr/bin/redis-server *:6379
root         8     0  0 01:30 ?        00:00:00 bash
root        22     8  0 01:30 ?        00:00:00 sleep 1000
root        36     0  0 01:30 ?        00:00:00 ps -ef
```

我们杀死bash进程之后查看进程列表，发现“bash”和“sleep 1000”进程都已经被杀死和回收

```text
docker@default:~$ docker exec myredis kill -9 8
docker@default:~$ docker exec myredis ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 01:29 ?        00:00:00 /bin/sh -c "/usr/bin/redis-server"
root         5     1  0 01:29 ?        00:00:00 /usr/bin/redis-server *:6379
root        45     0  0 01:31 ?        00:00:00 ps -ef
docker@default:~$ 
```

这是因为sh/bash等应用可以自动清理僵尸进程。

关于僵尸进程在Docker中init处理所需注意细节的详细描述，可以在如下文章得到

* [http://www.oschina.net/translate/docker-and-the-pid-1-zombie-reaping-problem](http://www.oschina.net/translate/docker-and-the-pid-1-zombie-reaping-problem?spm=5176.100239.blogcont5545.17.qOGovX)

简单而言，如果在容器中运行多个进程，PID1进程需要有能力接管“孤儿”进程并回收“僵尸”进程。我们可以  
1. 利用自定义的init进程来进行进程管理，比如 [S6](http://blog.tutum.co/2014/12/02/docker-and-s6-my-new-favorite-process-supervisor/?spm=5176.100239.blogcont5545.18.qOGovX) ， [phusion myinit](https://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/?spm=5176.100239.blogcont5545.19.qOGovX)，[dumb-init](https://github.com/Yelp/dumb-init?spm=5176.100239.blogcont5545.20.qOGovX), [tini](https://github.com/krallin/tini?spm=5176.100239.blogcont5545.21.qOGovX) 等  
2. Bash/sh等缺省提供了进程管理能力，如果需要可以作为PID1进程来实现正确的进程回收。

#### 进程监控 <a id="5"></a>

在Docker中，如果`docker run`命令中指明了[restart policy](https://docs.docker.com/engine/reference/run/?spm=5176.100239.blogcont5545.22.qOGovX#restart-policies-restart)，Docker Daemon会监控PID1进程，并根据策略自动重启已结束的容器。

| restart 策略 | 结果 |
| :--- | :--- |
| no | 不自动重启，缺省值 |
| on-failure\[:max-retries\] | 当PID1进程退出值非0时，自动重启容器；可以指定最大重试次数 |
| always | 永远自动重启容器；当Docker Daemon启动时，会自动启动容器 |
| unless-stopped | 永远自动重启容器；当Docker Daemon启动时，如果之前容器不为stoped状态就自动启动容器 |

注意：为防止频繁重启故障应用导致系统过载，Docker会在每次重启过程中会延迟一段时间。Docker重启进程的延迟时间从100ms开始并每次加倍，如100ms，200ms，400ms等等。

利用Docker内置的restart策略可以大大简化应用进程监控的负担。但是Docker Daemon只是监控PID1进程，如果容器在内包含多个进程，仍然需要开发人员来处理进程监控。

大家一定非常熟悉[Supervisor](http://supervisord.org/?spm=5176.100239.blogcont5545.23.qOGovX)，[Monit](https://mmonit.com/monit/?spm=5176.100239.blogcont5545.24.qOGovX)等进程监控工具，他们可以方便的在容器内部中实现进程监控。Docker提供了相应的[文档](https://docs.docker.com/engine/admin/using_supervisord/?spm=5176.100239.blogcont5545.25.qOGovX)来介绍，互联网上也有很多资料，我们今天就不再赘述了。

另外利用Supervisor等工具作为PID1进程是在容器中支持多进程管理的主要实现方式；和简单利用shell脚本fork子进程相比，采用Supervisor等工具有很多好处：

* 一些传统的服务不能以PID1进程的方式执行，利用Supervisor可以方便的适配
* Supervisor这些监控工具大多提供了对SIGTERM的信号传播支持，可以支持子进程优雅的退出

然而值得注意的是：Supervisor这些监控工具大多没有完全提供Init支持的进程管理能力，如果需要支持子进程回收的场景需要配合正确的PID1进程来完成

#### 总结 <a id="6"></a>

进程管理在Docker容器中和在完整的操作系统有一些不同之处。在每个容器的PID1进程，需要能够正确的处理SIGTERM信号来支持容器应用的优雅退出，同时要能正确的处理孤儿进程和僵尸进程。

在Dockerfile中要注意shell模式和exec模式的不同。通常而言我们鼓励使用exec模式，这样可以避免由无意中选择错误PID1进程所引入的问题。

在Docker中“一个容器一个进程的方式”并非绝对化的要求，然而在一个容器中实现对于多个进程的管理必须考虑更多的细节，比如子进程管理，进程监控等等。所以对于常见的需求，比如日志收集，性能监控，调试程序，我们依然建议采用多个容器组装的方式来实现。

