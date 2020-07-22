---
description: 可能是你能找到的最详细的WebLogic安全相关中文文档
---

# WebLogic安全研究报告

以下内容来自「[iso60001-WebLogic安全研究报告](https://nosec.org/home/detail/2859.html)」，我看了之后深受启发，也由此开启了WebLogic正确的漏洞挖掘之路，具体内容如下：

## 序言

        从我还未涉足安全领域起，就知道WebLogic的漏洞总会在安全圈内变成热点话题。WebLogic爆出新漏洞的时候一定会在朋友圈刷屏。在从事安全行业之后，跟了几个WebLogic漏洞，写了一些分析，也尝试挖掘新漏洞和绕过补丁。但因为能力有限，还需要对WebLogic，以及Java反序列化有更深入的了解才能在漏洞挖掘和研究上更得心应手。因此决定写这样一篇长文把我所理解的WebLogic和WebLogic漏洞成因、还有这一切涉及到的相关知识讲清楚，也是自己深入WebLogic的过程。因此，本文不是一篇纯漏洞分析，而主要在讲“是什么”、“什么样”、“为什么”。希望把和WebLogic漏洞有关的方方面面都讲一些，今后遇到类似的问题有据可查。

## 中间件（Middleware）

{% hint style="info" %}
中间件是一种独立的系统软件或服务程序,分布式应用软件借助这种软件在不同的技术之间共享资源, 中间件位于客户机服务器的操作系统之上，管理计算资源和网络通信。
{% endhint %}

中间件是指连接软件组件或企业应用程序的软件。中间件是位于操作系统和分布式计算机网络两侧的应用程序之间的软件层。它可以被描述为“软件胶水。通常，它支持复杂的分布式业务软件应用程序。

![1.png](https://nosec.org/avatar/uploads/attach/image/c6b0cc21c218a76fd271dbba7dbea2df/1.png)

Oracle定义中间件的组成包括Web服务器、应用程序服务器、内容管理系统及支持应用程序开发和交付的类似工具，它通常基于可扩展标记语言（xml）、简单对象访问协议（SOAP）、Web服务、SOA、Web 2.0和轻量级目录访问协议（LDAP）等技术。

### Oracle融合中间件（Oracle Fusion Middleware）

Oracle融合中间件是Oracle提出的概念，Oracle融合中间件为复杂的分布式业务软件应用程序提供解决方案和支持。Oracle融合中间件是一系列软件产品并包括一系列工具和服务， 如：符合Java Enterprise Edition 5（Java EE）的开发和运行环境、商业智能、协作和内容管理等。Oracle融合中间件为开发、部署和管理提供全面的支持。Oracle融合中间件通常提供以下图中所示的解决方案：

![2.png](https://nosec.org/avatar/uploads/attach/image/f091f59946e56a85e0f55baa560eaffd/2.png)

Oracle融合中间件提供两种类型的组件：

* Java组件

Java组件用于部署一个或多个Java应用程序，Java组件作为域模板部署到Oracle WebLogic Server域中。这里提到的Oracle WebLogic Server域在后面会随着Oracle WebLogic Server详细解释。

* 系统组件

系统组件是被Oracle Process Manager and Notification \(OPMN\)管理的进程，其不作为Java应用程序部署。系统组件包括Oracle HTTP Server、Oracle Web Cache、Oracle Internet Directory、Oracle Virtual Directory、Oracle Forms Services、Oracle Reports、Oracle Business Intelligence Discoverer、Oracle Business Intelligence。

## Oracle WebLogic Server（WebLogic）

Oracle WebLogic Server（以下简称WebLogic）是一个可扩展的企业级Java平台（Java EE）应用服务器。其完整实现了Java EE 5.0规范，并且支持部署多种类型的分布式应用程序。

在前面Oracle融合中间件的介绍中，我们已经发现了其中贯穿着WebLogic的字眼，且Oracle融合中间件和WebLogic也是我在漏洞分析时经常混淆的。实际上WebLogic是组成Oracle融合中间件的核心。几乎所有的Oracle融合中间件产品都需要运行WebLogic Server。因此，**本质上，WebLogic Server不是Oracle融合中间件，而是构建或运行Oracle融合中间件的基础，Oracle融合中间件和WebLogic密不可分却在概念上不相等。**

### Oracle WebLogic Server域

Oracle WebLogic Server域又是WebLogic的核心。Oracle WebLogic Server域是一组逻辑上相关的Oracle WebLogic Server资源组。域包括一个名为Administration Server的特殊Oracle WebLogic Server实例，它是配置和管理域中所有资源的中心点。也就是说无论是Web应用程序、EJB（Enterprise JavaBeans）、Web服务和其他资源的部署和管理都通过Administration Server完成。

![3.png](https://nosec.org/avatar/uploads/attach/image/550baf11d17cad07b919f720b75a5be7/3.png)

### Oracle WebLogic Server集群

WebLogic Server群集由多个同时运行的WebLogic Server服务器实例组成，它们协同工作以提供更高的可伸缩性和可靠性。因为WebLogic本身就是为分布式设计的中间件，所以集群功能也是WebLogic的重要功能之一。也就有了集群间通讯和同步，WebLogic的众多安全漏洞也是基于这个特性。

## WebLogic的版本

WebLogic版本众多，但是现在我们经常见到的只有两个类别：10.x和12.x，这两个大版本也叫WebLogic Server 11g和WebLogic Server 12c。

根据Oracle官方下载页面[https://www.oracle.com/technetwork/middleware/weblogic/downloads/wls-for-dev-1703574.html](https://www.oracle.com/technetwork/middleware/weblogic/downloads/wls-for-dev-1703574.html)（从下向上看）：

10.x的版本为Oracle WebLogic Server 10.3.6，这个版本也是大家用来做漏洞分析的时候最喜欢拿来用的版本。P牛的vulhub\([https://github.com/vulhub/vulhub](https://github.com/vulhub/vulhub)\)中所有WebLogic漏洞靶场都是根据这个版本搭建的。

12.x的主要版本有：

* Oracle WebLogic Server 12.1.3
* Oracle WebLogic Server 12.2.1
* Oracle WebLogic Server 12.2.1.1
* Oracle WebLogic Server 12.2.1.2
* Oracle WebLogic Server 12.2.1.3

值得注意的是，**Oracle WebLogic Server 10.3.6支持的最低JDK版本为JDK1.6， Oracle WebLogic Server 12.1.3支持的最低JDK版本为JDK1.7，Oracle WebLogic Server 12.2.1及以上支持的最低JDK版本为JDK1.8**。因此由于JDK的版本不同，尤其是反序列化漏洞的利用方式会略有不同。同时，**不同的Oracle WebLogic Server版本依赖的组件\(jar包\)也不尽相同，因此不同的WebLogic版本在反序列化漏洞的利用上可能需要使用不同的Gadget链（反序列化漏洞的利用链条）。**但这些技巧性的东西不是本文的重点，请参考其他文章。如果出现一些PoC在某些时候可以利用，某些时候利用不成功的情况，应考虑到这两点。

