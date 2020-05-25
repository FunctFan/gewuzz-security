---
description: serialization or deserialization
---

# 序列化/反~

        以下内容引自CSDN「 [**ssooking**](https://www.cnblogs.com/ssooking)」在其「[Java反序列化漏洞分析](https://www.cnblogs.com/ssooking/p/5875215.html)」的博文，具体内容已经是精简后的版本：

## 基本概念

* 序列化：把对象的状态信息转换为字节序列\(即可以存储或传输的形式\)的过程 ；
* 反序列化：即序列化的逆过程，由字节流还原成对象的过程；

{% hint style="info" %}
注： 字节序是指多字节数据在计算机内存中存储或者网络传输时各字节的存储顺序。
{% endhint %}

## 用途

1. 把对象的字节序列永久地保存到硬盘上，通常存放在一个文件中；
2. 在网络上传送对象的字节序列。

## **应用场景：**

1. 一般来说，服务器启动后就不会随时关闭了，但是如果系统升级和维护需要重启，而用户会话还在进行相应的操作，这时就需要使用序列化技术将session信息保存起来放在硬盘，待服务器重启之后，又通过反序列化技术重新加载。这样就保证了用户信息不会丢失，实现永久化保存。
2. 在很多应用中，需要对某些对象进行序列化，让它们离开内存空间，入住物理硬盘，以便减轻内存压力或便于长期保存。

## 漏洞产生

        据此，如果Java应用对用户输入，即不可信数据做了反序列化处理，那么攻击者可以通过构造恶意输入，让反序列化产生非预期的对象，非预期的对象在产生过程中就有可能造成任意代码执行。

## 反序列化漏洞挖掘

* 漏洞触发场景
  * 在java编写的web应用与web服务器间java通常会发送大量的序列化对象例如以下场景： 
  * HTTP请求中的参数，cookies以及Parameters。
  * RMI协议，被广泛使用的RMI协议完全基于序列化
  * JMX 同样用于处理序列化对象
  * 自定义协议：用来接收与发送原始的java对象
* 漏洞挖掘
  * 1.确定反序列化输入点

    * [ ] 首先应找出readObject方法调用，在找到之后进行下一步的注入操作，一般可以通过以下方法进行查找：
      * 源码审计：寻找可以利用的“靶点”，即确定调用反序列化函数readObject的调用地点。
      * 对该应用进行网络行为抓包，寻找序列化数据，如wireshark,tcpdump等

             **注： java序列化的数据一般会以标记（ac ed 00 05）开头，base64编码后的特征为rO0AB。**

  * 2.考察应用的Class Path中是否包含Apache Commons Collections库
  * 3.生成反序列化的payload
  * 4.提交我们的payload数据

## 总结

### 挖掘思路

* 首先要找到反序列化入口（source）
* 寻找调用链（gadget） 
* 触发漏洞的目标方法（sink） 

  反序列化漏洞挖掘的本质其实就是已知了source和sink，走通整个调用流程，即调用链gadget。

### 挖掘内容

1. source反序列化入口可以包括：Java原生的反序列化以及专有格式的反序列化。
   1. 1 Java原生的反序列化，就是通过原生方法ObjectInputStream.readObject\(\)来处理二进制格式内容，进而得到Java对象；
   2. 1 专有格式的反序列化，比如通过Fastjson, Xstream等第三方库，处理json, xml等格式内容，得到Java对象 
2. 执行目标sink，可以包括：
   1. 1 Runtime.exec\(\)，这种最为简单直接，即直接在目标环境中执行命令；
   2. 1 通过调用FileOutputStream.write\(\)实现任意文件创建；
   3. 1 Method.invoke\(\)，这种需要适当地选择方法和参数，通过反射调用来实现方法执
   4. 1 RMI/JNDI/JRMP等，主要通过引用远程对象，间接实现任意代码执行；
3. 调用链（gadget）

   从source出发，递归检查其所有方法调用，如果能够执行到sink，那就形成了一条gadget。

## 参考文献

* Java反序列化漏洞分析：[https://www.cnblogs.com/ssooking/p/5875215.html](https://www.cnblogs.com/ssooking/p/5875215.html)
* 玩转Ysoserial-CommonsCollection的七种利用方式分析：[https://www.freebuf.com/articles/web/214096.html](https://www.freebuf.com/articles/web/214096.html)
* [http://www.freebuf.com/vuls/90840.html](http://www.freebuf.com/vuls/90840.html)
* [https://security.tencent.com/index.php/blog/msg/97](https://security.tencent.com/index.php/blog/msg/97) 
* [http://www.tuicool.com/articles/ZvMbIne](http://www.tuicool.com/articles/ZvMbIne) [http://www.freebuf.com/vuls/86566.html](http://www.freebuf.com/vuls/86566.html) 
* [http://sec.chinabyte.com/435/13618435.shtml](http://sec.chinabyte.com/435/13618435.shtml) 
* [http://www.myhack58.com/Article/html/3/62/2015/69493\_2.htm](http://www.myhack58.com/Article/html/3/62/2015/69493_2.htm) 
* [http://blog.nsfocus.net/java-](http://blog.nsfocus.net/java-deserialization-vulnerability-comments/)[deserialization-vulnerability-comments/](http://blog.nsfocus.net/java-deserialization-vulnerability-comments/) 
* [http://www.ijiandao.com/safe/cto/18152.html](http://www.ijiandao.com/safe/cto/18152.html) 
* [https://www.iswin.org/2015/11/13/Apache-CommonsCollections-Deserialized-Vulnerability/](https://www.iswin.org/2015/11/13/Apache-CommonsCollections-Deserialized-Vulnerability/) 
* [http://www.cnblogs.com/dongchi/p/4796188.html](http://www.cnblogs.com/dongchi/p/4796188.html)
* [https://blog.chaitin.com/2015-11-11\_java\_unserialize\_rce/?from=timeline&isappinstalled=0\#h4\_漏洞利用实例](https://blog.chaitin.com/2015-11-11_java_unserialize_rce/?from=timeline&isappinstalled=0#h4_漏洞利用实例)



