---
description: 未知写，焉能防
---

# XXE漏洞写法及防御

        以下内容引自先知社区「Spoock」在其「[JAVA常见的XXE漏洞写法和防御](https://blog.spoock.com/2018/10/23/java-xxe/)」的博文，具体内容已经是精简后的版本：

## 前言

         最近经常看到有Java项目爆出XXE的漏洞并且带有CVE，包括[Spring-data-XMLBean XXE漏洞](https://blog.spoock.com/2018/05/16/cve-2018-1259/)、[JavaMelody组件XXE漏洞解析](https://mp.weixin.qq.com/s/Tca3GGPCIc7FZaubUTh18Q)、[Apache OFBiz漏洞](https://github.com/jamieparfet/Apache-OFBiz-XXE/)。微信支付SDK的XXE漏洞。本质上xxe的漏洞都是因为对xml解析时允许引用外部实体，从而导致读取任意文件、探测内网端口、攻击内网网站、发起DoS拒绝服务攻击、执行系统命令等。

## 不同库的Java XXE漏洞

测试用payload：

```markup
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
        <!ENTITY xxe SYSTEM "dnslog-ip">
        ]>
<evil>&xxe;</evil>
```

### **DocumentBuilderFactory**

**错误的修复方式：**

```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = dbf.newDocumentBuilder();
String FEATURE = null;
FEATURE = "http://javax.xml.XMLConstants/feature/secure-processing";
dbf.setFeature(FEATURE, true);
FEATURE = "http://apache.org/xml/features/disallow-doctype-decl";
dbf.setFeature(FEATURE, true);
FEATURE = "http://xml.org/sax/features/external-parameter-entities";
dbf.setFeature(FEATURE, false);
FEATURE = "http://xml.org/sax/features/external-general-entities";
dbf.setFeature(FEATURE, false);
FEATURE = "http://apache.org/xml/features/nonvalidating/load-external-dtd";
dbf.setFeature(FEATURE, false);
dbf.setXIncludeAware(false);
dbf.setExpandEntityReferences(false);
// 读取xml文件内容
FileInputStream fis = new FileInputStream("path/to/xxexml");
InputSource is = new InputSource(fis);
builder.parse(is);
```

 看似设置得很很全面，但是直接仍然会被攻击，原因就是在于`DocumentBuilder builder = dbf.newDocumentBuilder();`这行代码需要在`dbf.setFeature()`之后才能够生效；

**正确的修复方式**

```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
String FEATURE = null;
FEATURE = "http://javax.xml.XMLConstants/feature/secure-processing";
dbf.setFeature(FEATURE, true);
FEATURE = "http://apache.org/xml/features/disallow-doctype-decl";
dbf.setFeature(FEATURE, true);
FEATURE = "http://xml.org/sax/features/external-parameter-entities";
dbf.setFeature(FEATURE, false);
FEATURE = "http://xml.org/sax/features/external-general-entities";
dbf.setFeature(FEATURE, false);
FEATURE = "http://apache.org/xml/features/nonvalidating/load-external-dtd";
dbf.setFeature(FEATURE, false);
dbf.setXIncludeAware(false);
dbf.setExpandEntityReferences(false);
DocumentBuilder builder = dbf.newDocumentBuilder();
// 读取xml文件内容
FileInputStream fis = new FileInputStream("path/to/xxexml");
InputSource is = new InputSource(fis);
Document doc = builder.parse(is);
```

### SAXBuilder

这个库貌似使用得不是很多。`SAXBuilder`如果使用默认配置就会触发XXE漏洞；如下

```java
SAXBuilder builder = new SAXBuilder();
Document doc = builder.build(InputSource);
```

**修复方法**

* **方式一**

```java
SAXBuilder builder = new SAXBuilder(true);
Document doc = builder.build(InputSource);
```

* **方式二**

```java
SAXBuilder builder = new SAXBuilder();
builder.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
builder.setFeature("http://xml.org/sax/features/external-general-entities", false);
builder.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
builder.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
Document doc = builder.build(InputSource);
```

### SAXParserFactory

同样地，在默认配置下就会存在`XXE`漏洞。

```text
SAXParserFactory spf = SAXParserFactory.newInstance();
SAXParser parser = spf.newSAXParser();
parser.parse(InputSource, (HandlerBase) null);
```

修复方法

```text
SAXParserFactory spf = SAXParserFactory.newInstance();
spf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
spf.setFeature("http://xml.org/sax/features/external-general-entities", false);
spf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
spf.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
SAXParser parser = spf.newSAXParser();
parser.parse(InputSource, (HandlerBase) null);
```

### SAXReader

在默认情况下会出现`XXE`漏洞。

```java
SAXReader saxReader = new SAXReader();
saxReader.read(InputSource);
```

修复方法

```java
SAXReader saxReader = new SAXReader();
saxReader.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
saxReader.setFeature("http://xml.org/sax/features/external-general-entities", false);
saxReader.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
saxReader.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
saxReader.read(InputSource);
```

### SAXTransformerFactory

在默认情况下会出现`XXE`漏洞

```java
SAXTransformerFactory sf = (SAXTransformerFactory) SAXTransformerFactory.newInstance();
StreamSource source = new StreamSource(InputSource);
sf.newTransformerHandler(source);
```

修复方法

```java
SAXTransformerFactory sf = (SAXTransformerFactory) SAXTransformerFactory.newInstance();
sf.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
sf.setAttribute(XMLConstants.ACCESS_EXTERNAL_STYLESHEET, "");
StreamSource source = new StreamSource(InputSource);
sf.newTransformerHandler(source);
```

 通过跟踪源代码发现,`XMLConstants.ACCESS_EXTERNAL_DTD`的内容是`http://javax.xml.XMLConstants/property/accessExternalDTD`,`XMLConstants.ACCESS_EXTERNAL_STYLESHEET`是`http://javax.xml.XMLConstants/property/accessExternalStylesheet`

### SchemaFactory

在默认情况下也会出现`XXE`漏洞。

```java
SchemaFactory factory = SchemaFactory.newInstance("http://www.w3.org/2001/XMLSchema");
StreamSource source = new StreamSource(ResourceUtils.getPoc1());
Schema schema = factory.newSchema(InputSource);
```

修复方法

```java
SchemaFactory factory = SchemaFactory.newInstance("http://www.w3.org/2001/XMLSchema");
factory.setProperty(XMLConstants.ACCESS_EXTERNAL_DTD, "");
factory.setProperty(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");
StreamSource source = new StreamSource(InputSource);
Schema schema = factory.newSchema(source);
```

### TransformerFactory

使用默认的解析方法会存在`XXE`问题。

```java
TransformerFactory tf = TransformerFactory.newInstance();
StreamSource source = new StreamSource(InputSource);
tf.newTransformer().transform(source, new DOMResult());
```

修复方法

```java
TransformerFactory tf = TransformerFactory.newInstance();
tf.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
tf.setAttribute(XMLConstants.ACCESS_EXTERNAL_STYLESHEET, "");
StreamSource source = new StreamSourceInputSource);
tf.newTransformer().transform(source, new DOMResult());
```

### ValidatorSample

使用默认的解析方法会存在`XXE`问题

```java
SchemaFactory factory = SchemaFactory.newInstance("http://www.w3.org/2001/XMLSchema");
Schema schema = factory.newSchema();
Validator validator = schema.newValidator();
StreamSource source = new StreamSource(InputSource);
validator.validate(source);
```

修复方法

```java
SchemaFactory factory = SchemaFactory.newInstance("http://www.w3.org/2001/XMLSchema");
Schema schema = factory.newSchema();
Validator validator = schema.newValidator();
validator.setProperty(XMLConstants.ACCESS_EXTERNAL_DTD, "");
validator.setProperty(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");
StreamSource source = new StreamSource(InputSource);
validator.validate(source);
```

### XMLReader

使用默认的解析方法会存在`XXE`问题

```java
XMLReader reader = XMLReaderFactory.createXMLReader();
reader.parse(new InputSource(InputSource));
```

修复方法

```java
XMLReader reader = XMLReaderFactory.createXMLReader();
reader.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
reader.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
reader.setFeature("http://xml.org/sax/features/external-general-entities", false);
reader.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
reader.parse(new InputSource(InputSource));
```

### Unmarshaller\(唯一的不存在XXE的库\)

使用默认的解析方法不会存在`XXE`问题，这也是唯一一个使用默认的解析方法不会存在`XXE`的一个库。

```java
Class tClass = Some.class;
JAXBContext context = JAXBContext.newInstance(tClass);
Unmarshaller um = context.createUnmarshaller();
Object o = um.unmarshal(ResourceUtils.getPoc1());
tClass.cast(o);
```

## 总结

其实，通过对不同的XML解析库的修复方式可以发现，`XXE`的防护值需要限制带外实体的注入就可以了，修复方式也简单，需要设置几个选项为`false`即可，可能少许的几个库可能还需要设置一些其他的配置，但是都是类似的。

总体来说修复方式都是通过设置feature的方式来防御`XXE`。两种方法分别是：

```text
"http://apache.org/xml/features/disallow-doctype-decl", true 
"http://apache.org/xml/features/nonvalidating/load-external-dtd", false
"http://xml.org/sax/features/external-general-entities", false
"http://xml.org/sax/features/external-parameter-entities", false
```

配置如上。

另外一种是：

```text
XMLConstants.ACCESS_EXTERNAL_DTD, ""
XMLConstants.ACCESS_EXTERNAL_STYLESHEET, ""
```

本质上`XXE`的问题就是一个配置不当的问题，即容易发现也容易防御，但是前提是需要知道有这个漏洞，这也是就是很多开发人员因为不知道`XXE`最终写出了含有漏洞的代码。

