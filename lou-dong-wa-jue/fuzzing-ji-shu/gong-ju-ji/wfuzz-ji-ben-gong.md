---
description: Web application fuzzer
---

# wfuzz基本功

以下内容来自「[安全脉搏key师傅--Wfuzz基本功](https://www.secpulse.com/archives/81560.html)」，如有侵权，请及时联系我删除！

工具GitHub地址：

        [https://github.com/xmendez/wfuzz](https://github.com/xmendez/wfuzz)

## 爆破文件、目录

wfuzz本身自带字典：

![](../../../.gitbook/assets/image%20%2810%29.png)

进入各类目录查看所有的字典文件如下：

```text
.├── Injections
│   ├── All_attack.txt
│   ├── SQL.txt
│   ├── Traversal.txt
│   ├── XML.txt
│   ├── XSS.txt
│   └── bad_chars.txt
├── general
│   ├── admin-panels.txt
│   ├── big.txt
│   ├── catala.txt
│   ├── common.txt
│   ├── euskera.txt
│   ├── extensions_common.txt
│   ├── http_methods.txt
│   ├── medium.txt
│   ├── megabeast.txt
│   ├── mutations_common.txt
│   ├── spanish.txt
│   └── test.txt
├── others
│   ├── common_pass.txt
│   └── names.txt
├── stress
│   ├── alphanum_case.txt
│   ├── alphanum_case_extra.txt
│   ├── char.txt
│   ├── doble_uri_hex.txt
│   ├── test_ext.txt
│   └── uri_hex.txt
├── vulns
│   ├── apache.txt
│   ├── cgis.txt
│   ├── coldfusion.txt
│   ├── dirTraversal-nix.txt
│   ├── dirTraversal-win.txt
│   ├── dirTraversal.txt
│   ├── domino.txt
│   ├── fatwire.txt
│   ├── fatwire_pagenames.txt
│   ├── frontpage.txt
│   ├── iis.txt
│   ├── iplanet.txt
│   ├── jrun.txt
│   ├── netware.txt
│   ├── oracle9i.txt
│   ├── sharepoint.txt
│   ├── sql_inj.txt
│   ├── sunas.txt
│   ├── tests.txt
│   ├── tomcat.txt
│   ├── vignette.txt
│   ├── weblogic.txt
│   └── websphere.txt
└── webservices
    ├── ws-dirs.txt
    └── ws-files.txt
```

但相对[FuzzDB](https://github.com/fuzzdb-project/fuzzdb)和[SecLists](https://github.com/danielmiessler/SecLists)来说还是不够全面不够强大的，当然如果有自己的字典列表最好～

Wfuzz爆破文件：

```text
wfuzz -w wordlist URL/FUZZ.php
```

Wfuzz爆破目录：

```text
wfuzz -w wordlist URL/FUZZ
```

## 遍历枚举参数值

e.g. 假如你发现了一个未授权漏洞，地址为：http://127.0.0.1/getuser.php?uid=123 可获取uid为123的个人信息

uid参数可以遍历，已知123为三位数纯数字，需要从000-999进行遍历，也可以使用wfuzz来完成：

```text
wfuzz -z range,000-999 http://127.0.0.1/getuser.php?uid=FUZZ
```

使用payloads模块类中的range模块进行生成。

## POST请求测试

e.g. 发现一个登录框，没有验证码，想爆破弱口令账户。

请求地址为：http://127.0.0.1/login.php

POST请求正文为：username=&password=

使用wfuzz测试：

```text
wfuzz -w userList -w pwdList -d "username=FUZZ&password=FUZ2Z" http://127.0.0.1/login.php
```

-d参数传输POST请求正文。  


## Cookie测试

上文 遍历枚举参数值 中说到有未授权漏洞，假设这个漏洞是越权漏洞，要做测试的肯定需要让wfuzz知道你的Cookie才能做测试。

如下命令即可携带上Cookie：

```text
wfuzz -z range,000-999 -b session=session -b cookie=cookie http://127.0.0.1/getuser.php?uid=FUZZ
```

-b参数指定Cookie，多个Cookie需要指定多次，也可以对Cookie进行测试，仍然使用FUZZ占位符即可。

## HTTP Headers测试

e.g. 发现一个刷票的漏洞，这个漏洞需要伪造XFF头（IP）可达到刷票的效果，投票的请求为GET类型，地址为：http://127.0.0.1/get.php?userid=666。

那么现在我想给userid为666的朋友刷票，可以使用wfuzz完成这类操作：

```text
wfuzz -z range,0000-9999 -H "X-Forwarded-For: FUZZ" http://127.0.0.1/get.php?userid=666
```

-H指定HTTP头，多个需要指定多次（同Cookie的-b参数）。

## 测试HTTP请求方法（Method）

e.g. 想测试一个网站\(http://127.0.0.1/\)支持哪些HTTP请求方法

使用wfuzz：

```text
wfuzz -z list,"GET-POST-HEAD-PUT" -X FUZZ http://127.0.0.1/
```

这条命了中多了 -z list 和 -X 参数，-z list可以自定义一个字典列表（在命令中体现），以-分割；-X参数是指定HTTP请求方法类型，因为这里要测试HTTP请求方法，后面的值为FUZZ占位符。

## 使用代理

做测试的时候想使用代理可以使用如下命令：

```text
wfuzz -w wordlist -p proxtHost:proxyPort:TYPE URL/FUZZ
```

-p参数指定主机:端口:代理类型，例如我想使用ssr的，可以使用如下命令：

```text
wfuzz -w wordlist -p 127.0.0.1:1087:SOCKS5 URL/FUZZ
```

多个代理可使用多个-p参数同时指定，wfuzz每次请求都会选取不同的代理进行。

## 认证

想要测试一个需要HTTP Basic Auth保护的内容可使用如下命令：

```text
wfuzz -z list,"username-password" --basic FUZZ:FUZZ URL
```

wfuzz可以通过--basec --ntml --digest来设置认证头，使用方法都一样：

--basec/ntml/digest username:password

## 递归测试

使用-R参数可以指定一个payload被递归的深度\(数字\)。例如：爆破目录时，我们想使用相同的payload对已发现的目录进行测试，可以使用如下命令：

```text
wfuzz -z list,"admin-login.php-test-dorabox" -R 1 http://127.0.0.1/FUZZ
```

如上命令就是使用了自定义字典列表：

```text
admin
login.php
test
dorabox	
```

递归深度为1也就是说当发现某一个目录存在的时候，在存在目录下再递归一次字典。

## 并发和间隔

wfuzz提供了一些参数可以用来调节HTTP请求的线程

e.g. 你想测试一个网站的转账请求是否存在HTTP并发漏洞（条件竞争）

请求地址：http://127.0.0.1/dorabox/race\_condition/pay.php

POST请求正文：money=1

使用如下命令：

```text
wfuzz -z range,0-20 -t 20 -d "money=1" http://127.0.0.1/dorabox/race_condition/pay.php?FUZZ
```

![0x03.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2018/11/0x03-1024x824.png)

测试并发要控制请求次数，在这里为使用range模块生成0-20，然后将FUZZ占位符放在URL的参数后不影响测试即可，主要是用-t参数设置并发请求，该参数默认设置都是10。

另外使用-s参数可以调节每次发送HTTP的时间间隔。

## 保存测试结果

wfuzz通过printers模块来将结果以不同格式保存到文档中，一共有如下几种格式：

```text
raw       | `Raw` output format
json      | Results in `json` format
csv       | `CSV` printer ftw
magictree | Prints results in `magictree` format
html      | Prints results in `html` format
```

将结果以json格式输出到文件的命令如下：

```text
$ wfuzz -f outfile,json -w wordlist URL/FUZZ
```

使用-f参数，指定值的格式为输出文件位置,输出格式。

