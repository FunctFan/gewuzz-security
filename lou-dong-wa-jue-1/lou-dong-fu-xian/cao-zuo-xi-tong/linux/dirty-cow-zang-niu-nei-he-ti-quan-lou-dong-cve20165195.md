---
description: 内核页写时复制特性
---

# Dirty CoW脏牛内核提权漏洞（CVE-2016-5195）

以下内容引自简书博主ch3ckr的文章，如有侵权，请联系删除，具体如下：

{% hint style="info" %}
作者：ch3ckr  
链接：https://www.jianshu.com/p/df72d1ee1e3e  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
{% endhint %}

## **漏洞范围：**

Linux kernel &gt;= 2.6.22 && Linux kernel &lt;= 4.8（2007年发行，到2016年10月18日才修复）

## **危害：**

低权限用户利用该漏洞可以在众多Linux系统上实现本地提权

## **简要分析：**

该漏洞具体为，get\_user\_page内核函数在处理Copy-on-Write\(以下使用COW表示\)的过程中，可能产出竞态条件造成COW过程被破坏，导致出现写数据到进程地址空间内只读内存区域的机会。修改su或者passwd程序就可以达到root的目的。具体分析请查看官方分析。

## **测试工具：**

官方放出的EXP:[链接](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fdirtycow%2Fdirtycow.github.io)  
 提权为root权限的EXP一：[链接](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FFireFart%2Fdirtycow)  
 提权为root权限的EXP二：[链接](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fgbonacini%2FCVE-2016-5195)

## **测试过程（EXP一）：**

本次环境使用 bee-box虚拟机进行测试。  
 假设一开始获得的是普通用户bee的权限。

![](//upload-images.jianshu.io/upload_images/1258400-d3b16383259ba455.png?imageMogr2/auto-orient/strip|imageView2/2/w/553/format/webp)

1.使用`uname -a`命令查看linux内核信息，发现在脏牛漏洞范围内，可以进行测试。

![](//upload-images.jianshu.io/upload_images/1258400-f92aedc1336b4462.png?imageMogr2/auto-orient/strip|imageView2/2/w/595/format/webp)

2.将exp一下载到本地，使用`gcc -pthread dirty.c -o dirty -lcrypt`命令对dirty.c进行编译，生成一个dirty的可执行文件。

![](//upload-images.jianshu.io/upload_images/1258400-cad9e9b14787bae7.png?imageMogr2/auto-orient/strip|imageView2/2/w/599/format/webp)

3.执行`./dirty 密码`命令，即可进行提权。

![](//upload-images.jianshu.io/upload_images/1258400-55d83ee29e3c50e7.png?imageMogr2/auto-orient/strip|imageView2/2/w/597/format/webp)

4.此时使用上图中的账号密码即可获取root权限。

![](//upload-images.jianshu.io/upload_images/1258400-e26f0d4c2c2f4cf5.png?imageMogr2/auto-orient/strip|imageView2/2/w/605/format/webp)

## **测试过程（EXP二）**

由于bee-box的gcc版本较低，就不进行具体的测试了，具体步骤如下：  
 1.下载exp到靶机，解压并进入文件夹，执行`make`后会生成一个dcow的可执行文件。  
 2.执行`./dcow -s` 如果成功的话会返回一个root的 shell，失败则返回fail。

## **总结：**

1. 内核版本需要在2.6.22以上，并且未打补丁
2. gcc版本可能有要求（exp2），需先升级再测
3. 成败取决于环境

