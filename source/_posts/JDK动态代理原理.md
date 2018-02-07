---
title: 动态代理那些事
date: 2017-12-9 23:23:34
tags: [java,api]
categories: java
---

上回，简单的看了JDK动态代理与CGLIB动态代理的应用于区别，也了解了下Proxy.newProxyInstance()这个类参数与作用。这次我们接着往下看并提出几个疑问来引导下面的内容。
> *  这个代理类到底哪来的？
> * invoke()方法到底做了什么，怎么和我们的方法建立联系的？
> * 生成的代理类是什么样的？

### 再次探索Proxy.newProxyInstance
抽出主要的3条关键语句（可在IDEA中通过双击关键词来观察代码的走向）
分别是：克隆-生成指定的代理类-得到构造器-返回代理类
可以得知：生成代理类的内容在第二句话中，那么点进去~
```java
final Class<?>[] intfs = interfaces.clone();
Class<?> cl = getProxyClass0(loader, intfs);
final Constructor<?> cons = cl.getConstructor(constructorParams);
return cons.newInstance(new Object[]{h});
```
方法头注释我没加进来，意思是欲生成代理类，必先调用checkProxyAccess方法来执行权限检查，而在newProxyInstance方法的前几行遍执行了这个检查。
下面的3行单行注释意为：这个东西是有缓存的，如果有就不要用ProxyClassFactory再生成。
这边只有一个可以进入的方法，进入他！
```java
private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
      if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }

        // If the proxy class defined by the given loader implementing
        // the given interfaces exists, this will simply return the cached copy;
        // otherwise, it will create the proxy class via the ProxyClassFactory
        return proxyClassCache.get(loader, interfaces);
}
```

这个方法大量是对缓存的操作，从高频的get()，replace()，和putIfAbsen()，我们都不管。进入一个参数正好为之前传递进来的key和parameter的apply方法。
apply方法是BiFunction<T, U, R>接口的一个定义方法，他是一个函数式接口，在《Java8 in Action》P47也有他的身影。这种定义通常是表示为(T,U)->R，意为通过2个类型产生一个类型。这个方法有2个实现，幸好我们上面再注释中看到了ProxyClassFactory，进入他！
```java
Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
```

前面就是通过对接口的遍历来验证接口，在方法的尾部我们看到一个byte[]字节数组，并从注释得知他的作用是生成指定的代理类。我们点进generateProxyClass方法，发现他有几个参数不同的重载方法。


```java
/*
 * Generate the specified proxy class.
 */
byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
    proxyName, interfaces, accessFlags);
try {
    return defineClass0(loader, proxyName,
                        proxyClassFile, 0, proxyClassFile.length);
} catch (ClassFormatError e) {
    throw new IllegalArgumentException(e.toString());
}
```

下面我们用generateProxyClass方法把返回的字节码写入到文件中来查看其中的东西。
```java
public class GenerateProxy {
    public static void main(String[] args) {
        Talking talking = new JDKProxy(new TalkingImpl()).getProxy();
        talking.say("二狗子");
        String path = "C:/$Proxy0.class";
        byte[] classFile = ProxyGenerator.generateProxyClass("$Proxy0",
                TalkingImpl.class.getInterfaces());
        try (FileOutputStream out = new FileOutputStream(path)){
            out.write(classFile);
            out.flush();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
因为IDEA自带反编译功能，直接把生成的$Proxy0.class文件拖入IDEA即可。
```java
public final class $Proxy0 extends Proxy implements Talking {
    private static Method m1;
    private static Method m2;
    private static Method m3;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return ((Boolean)super.h.invoke(this, m1, new Object[]{var1})).booleanValue();
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void say(String var1) throws  {
        try {
            super.h.invoke(this, m3, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final int hashCode() throws  {
        try {
            return ((Integer)super.h.invoke(this, m0, (Object[])null)).intValue();
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m3 = Class.forName("com.company.proxy.Talking").getMethod("say", Class.forName("java.lang.String"));
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```
可以看到代理类继承于Proxy并实现了Talking接口，并可以看到定义了4个方法与静态块通过反射用来初始化。到这里，一切都豁然开朗了。

接下来回答上面3个问题：
> * 是通过ProxyGenerator.generateProxyClass（String, Class<?>[]）来生成的字节码
> * invoke方法即建立生成代理类与原被代理接口之间的联系
> * 生成的代理类就是酱紫的~