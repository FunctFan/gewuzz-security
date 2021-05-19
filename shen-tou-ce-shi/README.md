---
description: 渗透测试那些事儿
---

# 渗透测试

#### 1.当我们拿到一个网站的域名或者IP的时候。最先要做的是信息收集。

得到域名后查IP，推荐使用[站长工具](https://whois.chinaz.com/)，选择whois查询和IP反查询，通过ping该域名查看IP。

IP的 WHOIS查询，这样就获得了该网站的基本信息。当然还有DNS历史解析记录 （可能是看服务器是否开启了域间传送策略）

旁站查询，兄弟域名查询

#### 2.得到该网站的基本信息之后，我们再看看该域名下有那些主机

目前我使用的是子域名挖掘机，oneforall，只需要将二级域名输入进去即可，然后默认扫描的是80端口\(Web服务\)，443端口\(https服务\)

我们只需要注意的是如果我们拿到的域名中带有edu\(教育\)、gov\(政府\)等后缀、在进行子域名爆破的时候，**请将edu、gov等后缀原封不动的带上。**  

 如果我们将这些标示去掉，将严重影响我们的爆破成功率

#### 3.爆破出所有相关子域名后，我们将存活的子域名的IP过滤出来

然后使用[Nmap](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247499727&idx=1&sn=93d075b35b0bf1a6a1b7d5912ac3cd1f&chksm=ebeab6e2dc9d3ff4ae93db0782605aad675155a4095ab661c286c19d4a22d77d150731cc4156&token=1557201692&lang=zh_CN&scene=21#wechat_redirect)扫描这些主机上开放了哪些端口。

扫描完端口之后，将这些存活主机的端口记录下来，并分别标注这些端口号代表什么服务，如果有条件的话再将服务器版本号记录上去

我们也可以打开命令行，使用telnet 远程连接服务器，查看服务器是否开启Telnet服务（默认23端口）

如果显示正在连接，则说明23端口已开启，如果端口关闭或无法连接会进行显示如下

常见的几种端口有：

> 21：FTP远程文件传输端口
>
> 22：SSH，远程连接端口
>
> 3389端口：远程桌面连接

在发现这些端口只会我们可以尝试弱口令的爆破

这里推荐（hydra弱口令爆破工具）下载地址：`https://github.com/vanhauser-thc/thc-hydra`

6379端口：redis未授权访问GetShell

（http://blog.knownsec.com/2015/11/analysis-of-redis-unauthorized-of-expolit/）

该链接就是关于6379未授权访问的介绍

**总结下来就是**

hacker利用redis自带的config命令，可以进行写文件操作

然后攻击者将自己的公钥成功写入到目标服务器的/root/.ssh文件夹中的authotrized\_keys文件中

然后攻击者就可以用自己对应的私钥登陆目标服务器。

这里可参考博客 https://blog.csdn.net/sdb5858874/article/details/80484010



27017端口：mongodb默认未授权访问，直接控制数据库。

https://blog.csdn.net/u014153701/article/details/46762627

**总结下就是**

mongodb在刚刚安装完成时，默认数据库admin中一个用户都没有，在没有向该数据库中添加用户之前，

Hacker可以通过默认端口无需密码登陆对数据库任意操作而且可以远程访问数据库！

9200/9300端口：elasticsearch远程命令执行

初学者对这个东西认识不深，感觉就是通过java映射的方法达到攻击目的

你们可以看看下面链接的分析

https://www.secpulse.com/archives/5047.html

80/8080/443端口：常见的Web服务漏洞，这里应该就是我们使用看家本领的地方了

基于常见的web漏洞进行扫描检测，或者对管理后台进行一次弱口令爆破

443端口：心脏出血漏洞（open ssl 1.0.1g 以前的版本）（老师说现在基本没有了。。）

**自己的理解**：攻击者不需要经过身份验证或者其他操作，就可以轻易的从目标机内存中偷来最多64kb的数据

这其中可能包含我们用来登陆的用户名密码,电子邮件密码，或重要的商务消息

下面是大牛的漏洞介绍

https://zhuanlan.zhihu.com/p/19722263?columnSlug=drops

还有一些

jboss弱口令

weblogic 天生SSRF漏洞

resin任意文件读取  

这些东西暂时不看吧，等日后知识储备更多了再了解（后续更细）  附上这些漏洞的分析博文

http://www.hack80.com/thread-22662-1-1.html



