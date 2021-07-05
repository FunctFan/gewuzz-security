---
description: 此分类记录web类型题目解答wp
---

# xctf-攻防世界新手练习区web

题目1：view\_source

![](../../.gitbook/assets/image%20%28160%29.png)

解答过程：直接F12查看源码，得到flag

![](../../.gitbook/assets/image%20%28148%29.png)

题目2：robots

![](../../.gitbook/assets/image%20%28147%29.png)

解答过程：

题目提示Robots，因此直奔robots.txt

![](../../.gitbook/assets/image%20%28138%29.png)

发现了有意思的东西，直接访问该php

![](../../.gitbook/assets/image%20%28158%29.png)

题目3：backup

![](../../.gitbook/assets/image%20%28151%29.png)

解答过程：

题目提示你知道index.php的备份文件名

![](../../.gitbook/assets/image%20%28161%29.png)

因此，直接访问.index.php.swp/.index.php.bak/index.php.bak

![](../../.gitbook/assets/image%20%28162%29.png)

题目4：cookie

![](../../.gitbook/assets/image%20%28134%29.png)

解答过程：

cookie提示,F12查看cookie得知需要访问cookie.php

![](../../.gitbook/assets/image%20%28136%29.png)

访问cookie.php，提示see http response

![](../../.gitbook/assets/image%20%28155%29.png)

题目5：disabled\_button

![](../../.gitbook/assets/image%20%28140%29.png)

解答过程：F12,审计源代码，定位按钮，去掉disable

![](../../.gitbook/assets/image%20%28143%29.png)

题目6：weak\_auth

![](../../.gitbook/assets/image%20%28132%29.png)

解答过程：

![](../../.gitbook/assets/image%20%28135%29.png)

![](../../.gitbook/assets/image%20%28144%29.png)

题目7：simple\_php

![](../../.gitbook/assets/image%20%28154%29.png)

解答过程：

![](../../.gitbook/assets/image%20%28159%29.png)

![](../../.gitbook/assets/image%20%28145%29.png)

题目8：get\_post

![](../../.gitbook/assets/image%20%28156%29.png)

解答过程：

![](../../.gitbook/assets/image%20%28149%29.png)

![](../../.gitbook/assets/image%20%28139%29.png)

![](../../.gitbook/assets/image%20%28146%29.png)

题目9：xff\_referer

![](../../.gitbook/assets/image%20%28157%29.png)

解答过程：

X-Forwarded-For:简称XFF头，它代表客户端，也就是HTTP的请求端真实的IP，只有在通过了HTTP 代理或者负载均衡服务器时才会添加该项

HTTP Referer是header的一部分，当浏览器向web服务器发送请求的时候，一般会带上Referer，告诉服务器我是从哪个页面链接过来的

 请求头添加`X-Forwarded-For: 123.123.123.123`

 请求头内添加`Referer: https://www.google.com`，可获得flag

![](../../.gitbook/assets/image%20%28152%29.png)

题目10：webshell

![](../../.gitbook/assets/image%20%28142%29.png)

解答过程：

![](../../.gitbook/assets/image%20%28141%29.png)

题目11：command\_execution

![](../../.gitbook/assets/image%20%28150%29.png)

解答过程：

![](../../.gitbook/assets/image%20%28137%29.png)

![](../../.gitbook/assets/image%20%28153%29.png)

题目12：simple\_js

![](../../.gitbook/assets/image%20%28133%29.png)

解答过程：

抓包：

```javascript
   function dechiffre(pass_enc){
        var pass = "70,65,85,88,32,80,65,83,83,87,79,82,68,32,72,65,72,65";
        var tab  = pass_enc.split(',');
                var tab2 = pass.split(',');var i,j,k,l=0,m,n,o,p = "";i = 0;j = tab.length;
                        k = j + (l) + (n=0);
                        n = tab2.length;
                        for(i = (o=0); i < (k = j = n); i++ ){o = tab[i-l];p += String.fromCharCode((o = tab2[i]));
                                if(i == 5)break;}
                        for(i = (o=0); i < (k = j = n); i++ ){
                        o = tab[i-l];
                                if(i > 5 && i < k-1)
                                        p += String.fromCharCode((o = tab2[i]));
                        }
        p += String.fromCharCode(tab2[17]);
        pass = p;return pass;
    }
```

\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30

先将字符串用python处理一下，得到数组\[55,56,54,79,115,69,114,116,107,49,50\]，exp如下。

```python
s="\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30" 
print (s)
```

将得到的数字分别进行ascii处理，可得到字符串786OsErtk12，exp如下。

```python
a = [55,56,54,79,115,69,114,116,107,49,50]
c = ""
for i in a:
    b = chr(i)
    c = c + b
print(c)
```





