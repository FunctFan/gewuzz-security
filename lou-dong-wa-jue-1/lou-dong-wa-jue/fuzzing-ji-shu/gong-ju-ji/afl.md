---
description: afl
---

# AFL-基于变异的fuzz工具

## 简介

AFL是目前最受欢迎的一个工具，是一个导向型的fuzzing工具。

AFL这个工具出来的一个起因就是AFL的开发者认为盲fuzzing的效率是比较低的；第二个原因就是Charlie Miller和Laurent Gaffié所做的样本筛选的方法是有效果的；还有第三个原因就是符号执行，符号执行在理论是非常不错的，但在实际中经常受到可行性、性能等方面的限制。于是在这样一个背景下，AFL出现了。

AFL有两个关键词：指令插桩和边覆盖。首先AFL是基于插桩的，能够辅助程序分析；其次AFL是基于边覆盖的，是对Charlie Miller等人基于块覆盖用样本筛选的一个改进和提升。

## 工作流程

AFL（American Fuzzy Lop）是由安全研究员Michał Zalewski（[@lcamtuf](https://twitter.com/lcamtuf)）开发的一款基于覆盖引导（Coverage-guided）的模糊测试工具，它通过记录输入样本的代码覆盖率，从而调整输入样本以提高覆盖率，增加发现漏洞的概率。其工作流程大致如下：

①从源码编译程序时进行插桩，以记录代码覆盖率（Code Coverage）；

②选择一些输入文件，作为初始测试集加入输入队列（queue）；

③将队列中的文件按一定的策略进行“突变”；

④如果经过变异文件更新了覆盖范围，则将其保留添加到队列中;

⑤上述过程会一直循环进行，期间触发了crash的文件会被记录下来。

![1.jpg](https://image.3001.net/images/20181207/1544168163_5c0a22e3eedce.jpg!small)

## 参考文献

* AFL漏洞挖掘技术漫谈（一）：用AFL开始你的第一次Fuzzing：[https://www.freebuf.com/articles/system/191543.html](https://www.freebuf.com/articles/system/191543.html)
* AFL漏洞挖掘技术漫谈（二）：Fuzz结果分析和代码覆盖率：[https://www.freebuf.com/articles/system/197678.html](https://www.freebuf.com/articles/system/197678.html)



