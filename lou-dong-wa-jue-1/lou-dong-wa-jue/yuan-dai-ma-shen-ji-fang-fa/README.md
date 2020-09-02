---
description: idea可要会用哦
---

# 源代码审计方法

以下内容来自「 [安百科技](http://www.anbai.com/)-[凌天实验室](http://www.absec.cn/)-\[[园长](http://javaweb.org/) [iiusky](http://javaweb.org.cn/) [su18](https://su18.org/) [ver007](http://www.ver007.org/)\]的GitBook [javasec.org](http://javasec.org/)」，我看了之后深受启发，也由此开启了正确的java安全之路，具体内容如下：

## Java 代码审计 <a id="java-&#x4EE3;&#x7801;&#x5BA1;&#x8BA1;"></a>

通俗的说Java代码审计就是通过审计Java代码来发现Java应用程序自身中存在的安全问题，由于Java本身是编译型语言，所以即便只有class文件的情况下我们依然可以对Java代码进行审计。对于未编译的Java源代码文件我们可以直接阅读其源码，而对于已编译的class或者jar文件我们就需要进行反编译了。

Java代码审计其本身并无多大难度，只要熟练掌握审计流程和常见的漏洞审计技巧就可比较轻松的完成代码审计工作了。但是Java代码审计的方式绝不仅仅是使用某款审计工具扫描一下整个Java项目代码就可以完事了，一些业务逻辑和程序架构复杂的系统代码审计就非常需要审计者掌握一定的Java基础并具有具有一定的审计经验、技巧甚至是对Java架构有较深入的理解和实践才能更加深入的发现安全问题。

本章节讲述Java代码审计需要掌握的前置知识以及Java代码审计的流程、技巧。

### 准备环境和辅助工具 <a id="&#x51C6;&#x5907;&#x73AF;&#x5883;&#x548C;&#x8F85;&#x52A9;&#x5DE5;&#x5177;"></a>

在开始Java代码审计前请自行安装好Java开发环境，建议使用MacOS、Ubuntu操作系统。

所谓“工欲善其事，必先利其器”，合理的使用一些辅助工具可以极大的提供我们的代码审计的效率和质量！

强烈推荐下列辅助工具：

1. `Jetbrains IDEA(IDE)`
2. `Sublime text(文本编辑器)`
3. `JD-GUI(反编译)`
4. `Fernflower(反编译)`
5. `Bytecode-Viewer`
6. `Eclipse(IDE)`
7. `NetBeans(IDE)`

![code-tools](https://javasec.org/images/code-tools.png)

