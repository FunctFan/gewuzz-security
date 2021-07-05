---
description: 此分类记录web类型题目解答wp
---

# web类型

题目1：view\_source

![](../../.gitbook/assets/image%20%28155%29.png)

解答过程：直接F12查看源码，得到flag

![](../../.gitbook/assets/image%20%28145%29.png)

题目2：robots

![](../../.gitbook/assets/image%20%28144%29.png)

解答过程：

题目提示Robots，因此直奔robots.txt

![](../../.gitbook/assets/image%20%28136%29.png)

发现了有意思的东西，直接访问该php

![](../../.gitbook/assets/image%20%28153%29.png)

题目3：backup

![](../../.gitbook/assets/image%20%28147%29.png)

解答过程：

题目提示你知道index.php的备份文件名

![](../../.gitbook/assets/image%20%28156%29.png)

因此，直接访问.index.php.swp/.index.php.bak/index.php.bak

![](../../.gitbook/assets/image%20%28157%29.png)

题目4：cookie

![](../../.gitbook/assets/image%20%28133%29.png)

解答过程：

cookie提示,F12查看cookie得知需要访问cookie.php

![](../../.gitbook/assets/image%20%28135%29.png)

访问cookie.php，提示see http response

![](../../.gitbook/assets/image%20%28150%29.png)

题目5：disabled\_button

![](../../.gitbook/assets/image%20%28138%29.png)

解答过程：F12,审计源代码，定位按钮，去掉disable

![](../../.gitbook/assets/image%20%28140%29.png)

题目6：weak\_auth

![](../../.gitbook/assets/image%20%28132%29.png)

解答过程：

![](../../.gitbook/assets/image%20%28134%29.png)

![](../../.gitbook/assets/image%20%28141%29.png)

题目7：simple\_php

![](../../.gitbook/assets/image%20%28149%29.png)

解答过程：

![](../../.gitbook/assets/image%20%28154%29.png)

![](../../.gitbook/assets/image%20%28142%29.png)

题目8：get\_post

![](../../.gitbook/assets/image%20%28151%29.png)

解答过程：

![](../../.gitbook/assets/image%20%28146%29.png)

![](../../.gitbook/assets/image%20%28137%29.png)

![](../../.gitbook/assets/image%20%28143%29.png)

题目9：xff\_referer

![](../../.gitbook/assets/image%20%28152%29.png)

解答过程：

X-Forwarded-For:简称XFF头，它代表客户端，也就是HTTP的请求端真实的IP，只有在通过了HTTP 代理或者负载均衡服务器时才会添加该项

HTTP Referer是header的一部分，当浏览器向web服务器发送请求的时候，一般会带上Referer，告诉服务器我是从哪个页面链接过来的

 请求头添加`X-Forwarded-For: 123.123.123.123`

 请求头内添加`Referer: https://www.google.com`，可获得flag

![](../../.gitbook/assets/image%20%28148%29.png)

题目10：webshell

![](../../.gitbook/assets/image%20%28139%29.png)





