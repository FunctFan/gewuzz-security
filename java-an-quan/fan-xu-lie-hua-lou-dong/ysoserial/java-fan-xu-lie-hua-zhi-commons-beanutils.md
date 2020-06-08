# Java反序列化之Commons beanUtils

以下内容引自「大专栏」发表于 2020-03-18的「[ysoserial分析系列：一步步写Commons beanUtils利用](https://www.dazhuanlan.com/2020/03/18/5e71a8fec8f35/)」，具体内容已经是精简后的版本：

## 分析及运行环境 <a id="&#x5206;&#x6790;&#x53CA;&#x8FD0;&#x884C;&#x73AF;&#x5883;"></a>

  commons-beanutils:1.9.2，java 1.7

## TemplatesImpl <a id="TemplatesImpl"></a>

  见[JDK7u21分析](https://xiaozhicai.github.io/2020/02/20/jdk7u21/)，目的是调用getOutputProperties方法。

## BeanComparator <a id="BeanComparator"></a>

![](https://xiaozhicai.github.io//images/beanutils_field.png)

  BeanComparator实现了Comparator接口，可以实现javaBean的比较。

  成员变量 String:property指定了用于对比的字段。

### compare方法 <a id="compare&#x65B9;&#x6CD5;"></a>

![](https://xiaozhicai.github.io//images/beanutils.compare.png)

  当对javaBean进行比较时，会自动调用compare方法 ，该方法中会获得property指定的属性。

  例如javaBean中含有成员变量String:name和int：age，若在BeanComparator中指定property为age，则将按照age的取值作为javaBean比较时的依据。

  于是可知，若被比较的javaBean为恶意TemplatesImpl，且设置BeanComparator的成员变量property为outputProperties，在调用栈中会存在TemplatesImpl.getOutputProperties可以触发恶意代码。

### 部分利用链构造与调试 <a id="&#x90E8;&#x5206;&#x5229;&#x7528;&#x94FE;&#x6784;&#x9020;&#x4E0E;&#x8C03;&#x8BD5;"></a>

![](https://xiaozhicai.github.io//images/beanutils.compare_test.png)

  接下来的构造思路，就是找到反序列化时自动调用compare方法的地方

## priorityQueue <a id="priorityQueue"></a>

![](https://xiaozhicai.github.io//images/priorityqueue.png)

  PriorityQueue是一个基于优先级的队列。优先级队列的元素默认按照其自然顺序进行排序。如果构造构造队列时提供的Comparator，则根据Comparator.compare方法进行排序。

### readObject方法 <a id="readObject&#x65B9;&#x6CD5;"></a>

![](https://xiaozhicai.github.io//images/queue.readobject.png)

  该方法在依次对元素进行反序列化后，调用heapify\(\)进行排序。若使用了比较器，则调用siftDownUsingComparator函数进行排序。

### siftDownUsingComparator方法 <a id="siftDownUsingComparator&#x65B9;&#x6CD5;"></a>

![](https://xiaozhicai.github.io//images/sifdown.png)

  该方法调用Comparator.compare，对元素进行比较和排序。在构造队列时，如果使用BeanComparator，则将会调用到BeanComparator.compare。

### 利用链构造与调试 <a id="&#x5229;&#x7528;&#x94FE;&#x6784;&#x9020;&#x4E0E;&#x8C03;&#x8BD5;"></a>

  在上一个调用链的基础上，直接构造使用了BeanComparator的priorityQueue。并将恶意priorityQueue添加到队列中。

![](https://xiaozhicai.github.io//images/beanutils1.png)

  通过运行结果可以看到异常，不能执行到序列化操作。为了避免这个异常，我们需要现在队列中添加其他类型的元素，再通过反射修改为恶意TemplatesImpl。

  构造一个BeanComparator，设置成员变量property的取值为A，  
在priorityQueue中添加的元素要求：拥有名称为A的成员变量，且提供了访问A的方法（如getA）。

  ysoserial使用的中转为BigInteger，该类含有成员变量lowestSetBit，并提供了getLowestSetBit方法。因此设置BeanComparator的property属性为“lowestSetBit”，并向queue中添加BigInteger类型的元素，添加属性时就会调用BeanComparator.compare，按照lowestSetBit对BigInteger对象进行排序。

![](https://xiaozhicai.github.io//images/ysoserialbeanutils.png)

  在较高版本的jdk中，BigInteger的成员变量lowestSetBit即将废弃。为了更好地理解整个利用链，这里换一个中介，直接使用BeanComparator作为中介。

  BeanComparator含有成员变量property，且提供了公共方法setProperty和getProperty访问该变量。

  构造priorityQueue时，使用BeanComparator为比较器并设置property为“property”。在队列中添加的元素为BeanComparator类对象。然后通过反射修改priorityQueue的成员变量queue数组为存放恶意priorityQueue的数组。重新修改比较器的property取值为“outputProperties”。这样在反序列化后对元素排序时，将会调用TemplatesImpl.getOutputProperties，触发恶意代码。

![](https://xiaozhicai.github.io//images/beanutils3.png)

## 完整代码 <a id="&#x5B8C;&#x6574;&#x4EE3;&#x7801;"></a>

  构造含有恶意字节码的TemplatesImpl的代码见[JDK7u21分析](https://xiaozhicai.github.io/2020/02/20/jdk7u21/)

|  |  |
| :--- | :--- |


