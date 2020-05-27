---
description: afl
---

# AFL

## [https://www.freebuf.com/articles/system/191543.html](https://www.freebuf.com/articles/system/191543.html)（参考）

## [https://www.freebuf.com/articles/system/197678.html](https://www.freebuf.com/articles/system/197678.html)

## 简介

AFL是目前最受欢迎的一个工具，是一个导向型的fuzzing工具。 Fuzzing通常由盲fuzzing（blind fuzzing）和导向性fuzzing（guided fuzzing）两种。blind fuzzing生成测试数据的时候不考虑数据的质量，通过大量测试数据来概率性地触发漏洞。Guided fuzzing则关注测试数据的质量，期望生成更有效的测试数据来触发漏洞的概率。比如，通过测试覆盖率来衡量测试输入的质量，希望生成有更高测试覆盖率的数据，从而提升触发漏洞的概率。

AFL这个工具出来的一个起因就是AFL的开发者认为盲fuzzing的效率是比较低的；第二个原因就是Charlie Miller和Laurent Gaffié所做的样本筛选的方法是有效果的；还有第三个原因就是符号执行，符号执行在理论是非常不错的，但在实际中经常受到可行性、性能等方面的限制。于是在这样一个背景下，AFL出现了。

AFL有两个关键词：指令插桩和边覆盖。首先AFL是基于插桩的，能够辅助程序分析；其次AFL是基于边覆盖的，是对Charlie Miller等人基于块覆盖用样本筛选的一个改进和提升。

