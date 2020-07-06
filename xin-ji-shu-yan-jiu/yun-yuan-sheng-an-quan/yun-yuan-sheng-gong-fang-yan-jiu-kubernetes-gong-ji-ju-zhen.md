---
description: 仿效ATT&CK，微软发布云安全攻击矩阵
---

# 【云原生攻防研究】Kubernetes攻击矩阵

以下内容引自安全客「[天融信阿尔法实验室](https://www.anquanke.com/member/142730)」在其「[ysoserial Java 反序列化系列第一集 Groovy1](https://www.anquanke.com/post/id/202730)」的博文，具体内容已经是精简后的版本：



安全业界近来非常流行用ATT&CK框架设计知识库框架或威胁建模，最近微软也发布了一个开源云编排框架Kubernetes的攻击矩阵。（下图）

![](../../.gitbook/assets/image%20%2855%29.png)

微软希望通过这个矩阵帮助组织识别针对Kubernetes的各种安全威胁的防御能力差距，因为Kubernetes已成长为管理容器化应用程序的全球最受欢迎的开源系统之一。

微软指出，Kubernetes攻击矩阵的灵感来自Mitre ATT＆CK框架。该框架是全球可访问的公开知识库，任何安全行业人士都可以使用该知识库进行威胁建模。

微软Azure安全中心的安全研究软件工程师Yossi Weizman说：

{% hint style="info" %}
尽管Kubernetes具有许多优势，但它也带来了应考虑的新安全挑战。因此，至关重要的是要了解容器化环境中存在的各种安全风险，尤其是在Kubernetes中。
{% endhint %}

根据Weizman的说法，当Azure安全团队开始绘制Kubernetes威胁图时，他们注意到，尽管许多云攻击技术与针对Linux或Windows系统的攻击有所不同，但其策略却是相似的。

Weizman说：

{% hint style="info" %}
因此，我们创建了第一个Kubernetes攻击矩阵：类似于ATT＆CK的矩阵，其中包括与容器编排安全相关的主要技术，重点是Kubernetes。
{% endhint %}

微软的Kubernetes攻击矩阵覆盖九种主要攻击策略步骤：

·初始访问

·执行

·驻留

·提权

·绕过检测

·凭证访问

·发现

·横向移动

·收割

这些策略中的每一步都包含多种技术，攻击者可以使用这些技术来实现不同的目标。

Weizman说：

{% hint style="info" %}
了解容器环境的攻击面是为这些环境构建安全解决方案的第一步。
{% endhint %}

