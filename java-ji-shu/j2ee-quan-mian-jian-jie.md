---
description: J2EE 全面简介
---

# J2EE 全面简介

以下内容引自IBM「刘湛 」在其「[J2EE 全面简介](https://www.ibm.com/developerworks/cn/java/j2ee/index.html)」的博文，具体内容已经是精简后的版本：

## J2EE的概念

目前，Java 2平台有3个版本，它们是适用于小型设备和智能卡的Java 2平台Micro版（Java 2 Platform Micro Edition，J2ME）、适用于桌面系统的Java 2平台标准版（Java 2 Platform Standard Edition，J2SE）、适用于创建服务器应用程序和服务的Java 2平台企业版（Java 2 Platform Enterprise Edition，J2EE）。

J2EE是一种利用Java 2平台来简化企业解决方案的开发、部署和管理相关的复杂问题的体系结构。J2EE技术的基础就是核心Java平台或Java 2平台的标准版，J2EE不仅巩固了标准版中的许多优点，例如"编写一次、随处运行"的特性、方便存取数据库的JDBC API、CORBA技术以及能够在Internet应用中保护数据的安全模式等等，同时还提供了对 EJB（Enterprise JavaBeans）、Java Servlets API、JSP（Java Server Pages）以及XML技术的全面支持。其最终目的就是成为一个能够使企业开发者大幅缩短投放市场时间的体系结构。

J2EE体系结构提供中间层集成框架用来满足无需太多费用而又需要高可用性、高可靠性以及可扩展性的应用的需求。通过提供统一的开发平台，J2EE降低了开发多层应用的费用和复杂性，同时提供对现有应用程序集成强有力支持，完全支持Enterprise JavaBeans，有良好的向导支持打包和部署应用，添加目录支持，增强了安全机制，提高了性能。

### J2EE的优势 <a id="2"></a>

J2EE为搭建具有可伸缩性、灵活性、易维护性的商务系统提供了良好的机制:

1. 保留现存的IT资产: 由于企业必须适应新的商业需求，利用已有的企业信息系统方面的投资，而不是重新制定全盘方案就变得很重要。这样，一个以渐进的（而不是激进的，全盘否定的）方式建立在已有系统之上的服务器端平台机制是公司所需求的。J2EE架构可以充分利用用户原有的投资，如一些公司使用的BEA Tuxedo、IBM CICS, IBM Encina,、Inprise VisiBroker 以及Netscape Application Server。这之所以成为可能是因为J2EE拥有广泛的业界支持和一些重要的'企业计算'领域供应商的参与。每一个供应商都对现有的客户提供了不用废弃已有投资，进入可移植的J2EE领域的升级途径。由于基于J2EE平台的产品几乎能够在任何操作系统和硬件配置上运行，现有的操作系统和硬件也能被保留使用。
2. 高效的开发: J2EE允许公司把一些通用的、很繁琐的服务端任务交给中间件供应商去完成。这样开发人员可以集中精力在如何创建商业逻辑上，相应地缩短了开发时间。高级中间件供应商提供以下这些复杂的中间件服务:
   * 状态管理服务 -- 让开发人员写更少的代码，不用关心如何管理状态，这样能够更快地完成程序开发。
   * 持续性服务 -- 让开发人员不用对数据访问逻辑进行编码就能编写应用程序，能生成更轻巧，与数据库无关的应用程序，这种应用程序更易于开发与维护。
   * 分布式共享数据对象CACHE服务 -- 让开发人员编制高性能的系统，极大提高整体部署的伸缩性。
3. 支持异构环境: J2EE能够开发部署在异构环境中的可移植程序。基于J2EE的应用程序不依赖任何特定操作系统、中间件、硬件。因此设计合理的基于J2EE的程序只需开发一次就可部署到各种平台。这在典型的异构企业计算环境中是十分关键的。J2EE标准也允许客户订购与J2EE兼容的第三方的现成的组件，把他们部署到异构环境中，节省了由自己制订整个方案所需的费用。
4. 可伸缩性: 企业必须要选择一种服务器端平台，这种平台应能提供极佳的可伸缩性去满足那些在他们系统上进行商业运作的大批新客户。基于J2EE平台的应用程序可被部署到各种操作系统上。例如可被部署到高端UNIX与大型机系统，这种系统单机可支持64至256个处理器。（这是NT服务器所望尘莫及的）J2EE领域的供应商提供了更为广泛的负载平衡策略。能消除系统中的瓶颈，允许多台服务器集成部署。这种部署可达数千个处理器，实现可高度伸缩的系统，满足未来商业应用的需要。
5. 稳定的可用性: 一个服务器端平台必须能全天候运转以满足公司客户、合作伙伴的需要。因为INTERNET是全球化的、无处不在的，即使在夜间按计划停机也可能造成严重损失。若是意外停机，那会有灾难性后果。J2EE部署到可靠的操作环境中，他们支持长期的可用性。一些J2EE部署在WINDOWS环境中，客户也可选择健壮性能更好的操作系统如Sun Solaris、IBM OS/390。最健壮的操作系统可达到99.999%的可用性或每年只需5分钟停机时间。这是实时性很强商业系统理想的选择。

### J2EE 的四层模型 <a id="3"></a>

J2EE使用多层的分布式应用模型，应用逻辑按功能划分为组件，各个应用组件根据他们所在的层分布在不同的机器上。事实上，sun设计J2EE的初衷正是为了解决两层模式\(client/server\)的弊端，在传统模式中，客户端担当了过多的角色而显得臃肿，在这种模式中，第一次部署的时候比较容易，但难于升级或改进，可伸展性也不理想，而且经常基于某种专有的协议�D�D通常是某种数据库协议。它使得重用业务逻辑和界面逻辑非常困难。现在J2EE 的多层企业级应用模型将两层化模型中的不同层面切分成许多层。一个多层化应用能够为不同的每种服务提供一个独立的层，以下是 J2EE 典型的四层结构:

* 运行在客户端机器上的客户层组件
* 运行在J2EE服务器上的Web层组件
* 运行在J2EE服务器上的业务逻辑层组件
* 运行在EIS服务器上的企业信息系统\(Enterprise information system\)层软件

![](../.gitbook/assets/image%20%2895%29.png)

#### **J2EE应用程序组件**

J2EE应用程序是由组件构成的.J2EE组件是具有独立功能的软件单元，它们通过相关的类和文件组装成J2EE应用程序，并与其他组件交互。J2EE说明书中定义了以下的J2EE组件:应用客户端程序和applets是客户层组件.Java Servlet和JavaServer Pages\(JSP\)是web层组件.Enterprise JavaBeans\(EJB\)是业务层组件.

#### **客户层组件**

J2EE应用程序可以是基于web方式的,也可以是基于传统方式的.

#### web 层组件

J2EE web层组件可以是JSP 页面或Servlets.按照J2EE规范，静态的HTML页面和Applets不算是web层组件。正如下图所示的客户层那样，web层可能包含某些 JavaBean 对象来处理用户输入，并把输入发送给运行在业务层上的enterprise bean 来进行处理。



![](../.gitbook/assets/image%20%2897%29.png)

#### **业务层组件**

业务层代码的逻辑用来满足银行，零售，金融等特殊商务领域的需要,由运行在业务层上的enterprise bean 进行处理. 下图表明了一个enterprise bean 是如何从客户端程序接收数据，进行处理\(如果必要的话\), 并发送到EIS 层储存的，这个过程也可以逆向进行。有三种企业级的bean: 会话\(session\) beans, 实体\(entity\) beans, 和 消息驱动\(message-driven\) beans. 会话bean 表示与客户端程序的临时交互. 当客户端程序执行完后, 会话bean 和相关数据就会消失. 相反, 实体bean 表示数据库的表中一行永久的记录. 当客户端程序中止或服务器关闭时, 就会有潜在的服务保证实体bean 的数据得以保存.消息驱动 bean 结合了会话bean 和 JMS的消息监听器的特性, 允许一个业务层组件异步接收JMS 消息.

![](../.gitbook/assets/image%20%2898%29.png)

#### **企业信息系统层**

企业信息系统层处理企业信息系统软件包括企业基础建设系统例如企业资源计划 \(ERP\), 大型机事务处理, 数据库系统,和其它的遗留信息系统. 例如，J2EE 应用组件可能为了数据库连接需要访问企业信息系统

### J2EE 的结构 <a id="4"></a>

这种基于组件，具有平台无关性的J2EE 结构使得J2EE 程序的编写十分简单，因为业务逻辑被封装成可复用的组件，并且J2EE 服务器以容器的形式为所有的组件类型提供后台服务. 因为你不用自己开发这种服务, 所以你可以集中精力解决手头的业务问题.

**容器和服务**  
容器设置定制了J2EE服务器所提供得内在支持，包括安全，事务管理，JNDI\(Java Naming and Directory Interface\)寻址,远程连接等服务，以下列出最重要的几种服务：

* J2EE安全\(Security\)模型可以让你配置 web 组件或enterprise bean ,这样只有被授权的用户才能访问系统资源. 每一客户属于一个特别的角色，而每个角色只允许激活特定的方法。你应在enterprise bean的布置描述中声明角色和可被激活的方法。由于这种声明性的方法，你不必编写加强安全性的规则。
* J2EE 事务管理（Transaction Management）模型让你指定组成一个事务中所有方法间的关系，这样一个事务中的所有方法被当成一个单一的单元. 当客户端激活一个enterprise bean中的方法，容器介入一管理事务。因有容器管理事务，在enterprise bean中不必对事务的边界进行编码。要求控制分布式事务的代码会非常复杂。你只需在布置描述文件中声明enterprise bean的事务属性，而不用编写并调试复杂的代码。容器将读此文件并为你处理此enterprise bean的事务。
* JNDI 寻址\(JNDI Lookup\)服务向企业内的多重名字和目录服务提供了一个统一的接口,这样应用程序组件可以访问名字和目录服务.
* J2EE远程连接（Remote Client Connectivity）模型管理客户端和enterprise bean间的低层交互. 当一个enterprise bean创建后, 一个客户端可以调用它的方法就象它和客户端位于同一虚拟机上一样.
* 生存周期管理（Life Cycle Management）模型管理enterprise bean的创建和移除,一个enterprise bean在其生存周期中将会历经几种状态。容器创建enterprise bean，并在可用实例池与活动状态中移动他，而最终将其从容器中移除。即使可以调用enterprise bean的create及remove方法，容器也将会在后台执行这些任务。
* 数据库连接池（Database Connection Pooling）模型是一个有价值的资源。获取数据库连接是一项耗时的工作，而且连接数非常有限。容器通过管理连接池来缓和这些问题。enterprise bean可从池中迅速获取连接。在bean释放连接之可为其他bean使用。

**容器类型**  
J2EE应用组件可以安装部署到以下几种容器中去:

* EJB 容器管理所有J2EE 应用程序中企业级bean 的执行. enterprise bean 和它们的容器运行在J2EE 服务器上.
* Web 容器管理所有J2EE 应用程序中JSP页面和Servlet组件的执行. Web 组件和它们的容器运行在J2EE 服务器上.
* 应用程序客户端容器管理所有J2EE应用程序中应用程序客户端组件的执行. 应用程序客户端和它们的容器运行在J2EE 服务器上.
* Applet 容器是运行在客户端机器上的web浏览器和 Java 插件的结合.

![](../.gitbook/assets/image%20%2892%29.png)

### J2EE的核心API与组件 <a id="5"></a>

J2EE平台由一整套服务（Services）、应用程序接口（APIs）和协议构成，它对开发基于Web的多层应用提供了功能支持，下面对J2EE中的13种技术规范进行简单的描述\(限于篇幅，这里只能进行简单的描述\):

1. JDBC\(Java Database Connectivity\): JDBC API为访问不同的数据库提供了一种统一的途径，象ODBC一样，JDBC对开发者屏蔽了一些细节问题，另外，JDCB对数据库的访问也具有平台无关性。
2. JNDI\(Java Name and Directory Interface\): JNDI API被用于执行名字和目录服务。它提供了一致的模型来存取和操作企业级的资源如DNS和LDAP，本地文件系统，或应用服务器中的对象。
3. EJB\(Enterprise JavaBean\): J2EE技术之所以赢得某体广泛重视的原因之一就是EJB。它们提供了一个框架来开发和实施分布式商务逻辑，由此很显著地简化了具有可伸缩性和高度复杂的企业级应用的开发。EJB规范定义了EJB组件在何时如何与它们的容器进行交互作用。容器负责提供公用的服务，例如目录服务、事务管理、安全性、资源缓冲池以及容错性。但这里值得注意的是，EJB并不是实现J2EE的唯一途径。正是由于J2EE的开放性，使得有的厂商能够以一种和EJB平行的方式来达到同样的目的。
4. RMI\(Remote Method Invoke\): 正如其名字所表示的那样，RMI协议调用远程对象上方法。它使用了序列化方式在客户端和服务器端传递数据。RMI是一种被EJB使用的更底层的协议。
5. Java IDL/CORBA: 在Java IDL的支持下，开发人员可以将Java和CORBA集成在一起。 他们可以创建Java对象并使之可在CORBA ORB中展开, 或者他们还可以创建Java类并作为和其它ORB一起展开的CORBA对象的客户。后一种方法提供了另外一种途径，通过它Java可以被用于将你的新的应用和旧的系统相集成。
6. JSP\(Java Server Pages\): JSP页面由HTML代码和嵌入其中的Java代码所组成。服务器在页面被客户端所请求以后对这些Java代码进行处理，然后将生成的HTML页面返回给客户端的浏览器。
7. Java Servlet: Servlet是一种小型的Java程序，它扩展了Web服务器的功能。作为一种服务器端的应用，当被请求时开始执行，这和CGI Perl脚本很相似。Servlet提供的功能大多与JSP类似，不过实现的方式不同。JSP通常是大多数HTML代码中嵌入少量的Java代码，而servlets全部由Java写成并且生成HTML。
8. XML\(Extensible Markup Language\): XML是一种可以用来定义其它标记语言的语言。它被用来在不同的商务过程中共享数据。XML的发展和Java是相互独立的，但是，它和Java具有的相同目标正是平台独立性。通过将Java和XML的组合，您可以得到一个完美的具有平台独立性的解决方案。
9. JMS\(Java Message Service\): MS是用于和面向消息的中间件相互通信的应用程序接口\(API\)。它既支持点对点的域，有支持发布/订阅\(publish/subscribe\)类型的域，并且提供对下列类型的支持：经认可的消息传递,事务型消息的传递，一致性消息和具有持久性的订阅者支持。JMS还提供了另一种方式来对您的应用与旧的后台系统相集成。
10. JTA\(Java Transaction Architecture\): JTA定义了一种标准的API，应用系统由此可以访问各种事务监控。
11. JTS\(Java Transaction Service\): JTS是CORBA OTS事务监控的基本的实现。JTS规定了事务管理器的实现方式。该事务管理器是在高层支持Java Transaction API \(JTA\)规范，并且在较底层实现OMG OTS specification的Java映像。JTS事务管理器为应用服务器、资源管理器、独立的应用以及通信资源管理器提供了事务服务。
12. JavaMail: JavaMail是用于存取邮件服务器的API，它提供了一套邮件服务器的抽象类。不仅支持SMTP服务器，也支持IMAP服务器。
13. JTA\(JavaBeans Activation Framework\): JavaMail利用JAF来处理MIME编码的邮件附件。MIME的字节流可以被转换成Java对象，或者转换自Java对象。大多数应用都可以不需要直接使用JAF。

**相关主题**

* 《Develop n-tier application using J2EE》- Steven Gould
* 《The Business Benefits of EJB and J2EE Technologies over COM+ and Windows DNA》
* 《The J2EE Tutorial》chapter overview - Monica Pawlan
* 本文所用图片由《The J2EE Tutorial》中的英文图片修改而成.

