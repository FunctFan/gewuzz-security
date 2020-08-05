---
description: 包括宿主机docker环境检测，执行容器内部信息获取，运行docker守护进程的宿主机发起测试
---

# 容器安全检测脚本总结

## 在宿主机探测docker环境的自动化脚本

问题：/.dockerenv文件是否存在？ 

操作：执行ls /.dockerenv查看。 

问题：/proc/1/cgroup中是否包括/docker字样？ 

操作：执行grep '/docker' /proc/1/cgroup查看。 

问题：当前环境内的进程数是否少于5个？ 

操作：执行ps aux查看。 

问题：PID为1的进程是否是常规init进程（如systemd或init）？ 

操作：执行ps -p1查看PID为1的进程。 

问题：当前环境是否缺少常见库或工具？ 

操作：执行which xxx来查看常用工具是否存在，例如执行which sudo来查看sudo是否存在。 

```bash
#!/bin/bash

DOCKER_ENV_FILE=/.dockerenv
DOCKER=docker
DOCKER_CGROUP=/proc/1/cgroup

ls -al $DOCKER_ENV_FILE
grep "$DOCKER" $DOCKER_CGROUP
ps aux
ps -p1
which sudo
which apt
which vi
which ping
which ssh
```

## 容器内获取信息

问题：当前用户是？ 

操作：执行id查看当前用户和用户组。 

问题：当前环境中存在哪些用户？ 

操作：读取/etc/passwd数据，例如cat /etc/passwd。 

问题：容器操作系统是？ 

操作：读取/etc/os-release数据，获得操作系统相关信息。 

问题：有哪些进程在容器内运行？ 

操作：执行ps aux查看。 

问题：宿主机内核版本是？ 

操作：执行uname -a查看。 

问题：容器内进程具有哪些权限（capabilities）？ 

操作：执行grep CapEff /proc/self/status获得权限值，在其他系统上执行capsh --decode=value将前面获得的值解析为具体权限。例如： 

![](../.gitbook/assets/image%20%28105%29.png)

问题：当前容器是否运行在特权模式？ 

操作：如果上一步中得到的权限值为0000003fffffffff，那么容器就运行在特权模式，我们就能够进行容器逃逸。 

问题：容器挂载了哪些卷？ 

操作：读取/proc/mounts数据，查看卷挂载情况。 

问题：环境变量中是否存储了敏感信息？ 

操作：执行env命令，列举所有环境变量。 

问题：容器内是否挂载了Docker Socket？ 

操作：检查/proc/mounts是否包含docker.sock或类似文件。通常/run/docker.sock或/var/run/docker.sock会是挂载点。如果发现挂载，我们就能够进行容器逃逸。 

问题：哪些主机是当前环境下网络可达的？ 

操作：有条件的话应该使用nmap探测。还可以先读取/etc/hosts查看容器的IP地址。例如：   


![](../.gitbook/assets/image%20%28104%29.png)

```bash
#!/bin/bash

id
cat /etc/passwd
cat /etc/os-release
ps aux
uname -a
grep CapEff /proc/self/status
CAP=`grep CapEff /proc/self/status | cut  -f 2`
if [ "$CAP" = "0000003fffffffff" ]; then
  echo -e "Container is privileged."
else
  echo -e "Container is not privileged."
fi
cat /proc/mounts
env
cat /proc/mounts | grep docker.sock
cat /etc/hosts
```

## 从运行Docker守护进程的宿主机上发起测试

问题：Docker版本是多少？ 

操作：执行docker --version查看。我们可以据此判断一些已知漏洞是否存在。 

问题：哪些CIS条目没有被正确地配置？ 

操作：运行Docker Bench for Security项目\[1\]来快速发现安全缺陷。笔者在本地环境下的部分测试结果如下图所示： 

![](../.gitbook/assets/image%20%28103%29.png)

问题：哪些用户被允许与Docker Socket交互？ 

操作：执行ls -l /var/run/docker.sock来查看/var/run/docker.sock所属用户和用户组，以及哪些用户对其有读写权限。 

问题：哪些用户在docker用户组中？ 

操作：执行grep docker /etc/group查看。 

问题：Docker客户端工具是否设置了setuid标志位？ 

操作：执行ls -l $\(which docker\)查看。 

问题：当前宿主机上有哪些可用镜像？ 

操作：执行docker images -a查看。 

问题：当前宿主机上有哪些容器？ 

操作：执行docker ps -a查看。 

问题：Docker守护进程是如何启动的（带了哪些参数）？ 

操作：读取配置文件，例如/usr/lib/systemd/system/docker.service和/etc/docker/daemon.json来查看。 

问题：当前宿主机上是否存在一些docker-compose.yaml文件？ 

操作：执行find / -name "docker-compose.\*"查看。 

问题：当前宿主机上是否存在.docker/config.json文件？ 

操作：执行cat /home/\*/.docker/config.json来读取任何可能存在的.docker/config.json文件。 

问题：iptables规则集是否同时为宿主机和容器设置？ 

操作：执行iptables -vnL或iptables -t filter -vnL查看。

```bash
#!/bin/bash

ls -l /var/run/docker.sock
grep docker /etc/group
ls -l $(which docker)
docker images -a
docker ps -a
cat /usr/lib/systemd/system/docker.service
cat /etc/systemd/system/docker.service
cat /etc/docker/daemon.json
find / -name "docker-compose.*"
cat /home/*/.docker/config.json
iptables -t filter -vnL
```



