# weblogic漏洞研究经验笔记

### weblogic cve漏洞捡漏指南 <a id="activity-name"></a>

{% embed url="https://mp.weixin.qq.com/s/HlG\_Jd8zu0dR4rQw25hfDg" %}



1.在idea中，覆盖一个类最简单的办法是，在自己的项目中创建一个与被覆盖的类包名相同类名相同的类，然后修改你想要修改的代码即可。例如在这个反序列化中，我要修改 writeExternal方法。直接强行替换成cve-2020-2555的gadget。

2.weblogic 12已经不支持在jdk版本低于1.8的jdk下运行。

3.weblogic中的补丁，其实是一个个编译好的class文件，我们直接使用idea打开补丁文件夹，就可以利用idea的反编译功能区分析补丁。

补丁中，每个class文件，都对应weblogic 实际目录中的一个class文件，打补丁你可以认为强行替换weblogic 中相关jar包中的class文件。

4. 直接看黑名单`files\oracle.wls.jrf.tenancy.common.sharedlib\12.2.1.4.0\wls.common.symbol\modules\com.bea.core.utils.jar\weblogic\utils\io` 文件夹下的`WeblogicFilterConfig.class` 文件

5.因为weblogic 的漏洞，绝大多数都是T3协议、IIOP协议的java反序列化漏洞。而weblogic为了修复该漏洞，最简单的办法是设置反序列化黑名单并添加黑名单列表。如果反序列化时遇到的类存在于黑名单中，则中止反序列化过程。

我们只需要diff黑名单列表，自己研究构造poc即可。有的时候不一定是rce，也有可能是其他问题。

6.如果触发反序列化的类在正常业务中可能需要，或者因为其他原因不能屏蔽，weblogic的修复方法为直接修改相关的类。

如果一个类在readObject方法中，自己私自调用ObjectInputStream去执行反序列化操作而不是用weblogic提供的FilterInputStream执行反序列化操作，这样的话会导致weblogic的黑名单失效。这也就是反序列化中的反序列化漏洞，这种漏洞在weblogic中挺常见的。

7.光复现还不够，缺乏的是总结思路，知道他是怎么修复的





