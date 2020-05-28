---
description: teenage mutant ninja turtles V 1.5
---

# tmnt-Web应用漏洞fuzzer

以下内容来自「[FreeBuf博主open的Web应用漏洞fuzz工具 – teenage mutant ninja turtles V 1.5](https://www.freebuf.com/sectool/5623.html)」，如有侵权，请立即联系我删除！具体内容如下：

![](https://image.3001.net/uploads/image/20120913/20120913150459_65350.jpg)

**现在很多大中型网站都在服务器前段架设了WAF、IPS等过滤设备，常见的SQL注入、XSS、XXE等都会在这一层面被过滤掉，或者给渗透测试造成了各种不方便。**

该工具主要就是生成混淆的测试代码，绕过常见的过滤设备，使用了如下一些常见的混淆方法：

```text
Using Case Variation
Using SQL Comments
Using URL Encoding
Using Dynamic Query Execution
Using Null Bytes
Nesting Stripped Expressions
Exploiting Truncation
Using Non-Standard Entry Points
Combine all techniques above 
```

teenage-mutant-ninja-turtles\(忍者神龟\)该软件主要包含三个测试项目

```text
1：基于fuzzdb的一些测试Payload
2：Web应用程序错误的数据库（比如返回的错误信息等）
3：Web应用程序中Payload部分可自行修改
```

[下载地址](http://code.google.com/p/teenage-mutant-ninja-turtles/downloads/list)（需要翻墙）

