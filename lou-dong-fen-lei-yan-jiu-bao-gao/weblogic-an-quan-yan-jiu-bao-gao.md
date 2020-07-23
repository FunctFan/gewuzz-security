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



