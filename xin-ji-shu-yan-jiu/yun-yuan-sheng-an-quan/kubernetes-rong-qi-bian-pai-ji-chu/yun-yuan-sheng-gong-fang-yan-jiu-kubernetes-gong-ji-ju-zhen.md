---
description: 仿效ATT&CK，微软发布云安全攻击矩阵
---

# 云原生攻防研究：Kubernetes攻击矩阵

以下内容引自安全客「 [安全牛](javascript:void%280%29;)」在其「[仿效ATT&CK，微软发布云安全攻击矩阵](https://mp.weixin.qq.com/s?__biz=MjM5Njc3NjM4MA==&mid=2651088712&idx=2&sn=085188e6d5a4fc0838d62346a72736be&chksm=bd14d79b8a635e8d615a9286548a7815dc8911fd2519a692c7ee1fdee8f7d155ef244267e120&mpshare=1&scene=1&srcid=&sharer_sharetime=1592283304049&sharer_shareid=e51105992a2df16a51a17def3d0e3b20&key=d9e86545f3b3a92b57c3e9efc5ee7910b4e3b74ee4a4470f776472ed8bd711c50208d30661fcc957c4d0f27d72265a6f8445e9409dd70ce78416ec9a40eb020ec813b3181e559a6f3f622e4e8ee97e1e&ascene=1&uin=MTQxNTg2Mjc4NA%3D%3D&devicetype=Windows+7+x64&version=6209007b&lang=zh_CN&exportkey=ASVqgrg21YvxB8ZbtUDxGLg%3D&pass_ticket=iZIaOuMZAhHldf%2BBzYvKc6sFYuX3vwOT%2BnMe8f4D%2B1huP5rbCSMtXmaeriA64Pby)」的博文，具体内容已经是精简后的版本：

## 正文

安全业界近来非常流行用ATT&CK框架设计知识库框架或威胁建模，最近微软也发布了一个开源云编排框架Kubernetes的攻击矩阵。（下图）

![](../../../.gitbook/assets/image%20%2855%29.png)

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

