---
description: You don't have 0day?try fuzzing technologies! That's so crazy!
---

# Fuzzing技术

以下内容来自「[北京丁牛科技有限公司- DigApis-简单高效的模糊测试——Fuzzing](https://www.freebuf.com/news/193602.html)」，如有侵权，请及时联系我删除！

## 基本概念

Fuzzing是指通过构造测试输入，对软件进行大量测试来发现软件中的漏洞的一种模糊测试方法。在CTF比赛中，fuzzing可能不常用，但在现实的漏洞挖掘中，fuzzing因其简单高效的优势，成为非常主流的漏洞挖掘方法。

Fuzzing通常由盲fuzzing（blind fuzzing）和导向性fuzzing（guided fuzzing）两种。blind fuzzing生成测试数据的时候不考虑数据的质量，通过大量测试数据来概率性地触发漏洞。Guided fuzzing则关注测试数据的质量，期望生成更有效的测试数据来触发漏洞的概率。比如，通过测试覆盖率来衡量测试输入的质量，希望生成有更高测试覆盖率的数据，从而提升触发漏洞的概率。

## 听听大牛怎么说fuzzing

{% hint style="info" %}
 **Charlie Miller，**他是第一个成功攻击iPhone, G1 Phone的人，是2018-2011年蝉联4届的Pwn2Own冠军，他提出模糊测试只需要5行python代码，并且运行这5行代码，一定可以找到一些你想要的东西。
{% endhint %}

![5 lines of python](../../../.gitbook/assets/image%20%2816%29.png)

{% hint style="info" %}
 **Laurent Gaffié**，他在SMB协议上出了很多成绩，比如在3秒之内就在打好补丁的MS07-063上找到了一个bug，CVE-2009-3103，CVE-2009-3676,  CVE-2010-0270，CVE-2010-0016,CVE-2010-0017，CVE-2010-0470, CVE-2010-0476,CVE-2010-0477，CVE-2010-2550，CVE-2011-1869等都是他发现的漏洞。并且他将自己的模糊测试的方法称为“Home Made Fuzzer”，从这个名称上就可以看出，fuzzing的代码就是真的很简单。
{% endhint %}

![Home Made Fuzzer](../../../.gitbook/assets/image%20%285%29.png)

看他们说的简单，其实在他们简单的背后蕴藏着重要的挖掘经验和思想在里面，没有功底的人是一下无法领略到其中的真正内涵的，但是你有理由相信：只要你具备了一定的：

{% hint style="danger" %}
细心  耐心  会看  会记  懂收集  勤动手  爱学习
{% endhint %}

掌握了正确的方法，那么fuzzing技术确实不是你想象中那么难！





