---
title: ClassLoad.getResources实践
date: 2017-09-16 22:23:34
tags: [java,api]
categories: java
---

# 实例
要完成一个获取指定包名下的所有类

## 熟悉相关API
以下是getResources源码中的注释：
> Finds all the resources with the given name. A resource is some data(images, audio, text, etc) that can be accessed by class code in away that is independent of the location of the code.
> @return  An enumeration of {@link java.net.URL <tt>URL</tt>} objects for the resource.  If no resources could  be found, the enumeration  will be empty.  Resources that the class loader doesn't have access to will not be in the enumeration.

大致的意思是：根据给定的名称找到其下的数据，返回值是一个Enumeration，我并不知道Enumeration是什么。。
惯例看源码中的注释：
> An object that implements the Enumeration interface generates a series of elements, one at a time. Successive calls to the <code>nextElement</code> method return successive elements of the series.

非常清晰的描述：实现了枚举接口的对象，可生成一连串的元素。需要连续调用nextElement方法来获得其中的元素。
恕我直言，这用法和迭代器Iterator一样。接着往下看。
> NOTE: The functionality of this interface is duplicated by the Iterator
  interface.  In addition, Iterator adds an optional remove operation, and
  has shorter method names.  New implementations should consider using
  Iterator in preference to Enumeration.

先告诉我们这个接口的功能是由迭代器复写的，此外推销了一波迭代器并说新实现应考虑使用迭代器。

以下是Enumeration接口的源码，很短，用法也很明了。就是让我们先判断有没有下一个元素，然后取出来操作，和迭代器一毛一样。
```java
public interface Enumeration<E> {
    /**
     * Tests if this enumeration contains more elements.
     *
     * @return  <code>true</code> if and only if this enumeration object
     *           contains at least one more element to provide;
     *          <code>false</code> otherwise.
     */
    boolean hasMoreElements();

    /**
     * Returns the next element of this enumeration if this enumeration
     * object has at least one more element to provide.
     *
     * @return     the next element of this enumeration.
     * @exception  NoSuchElementException  if no more elements exist.
     */
    E nextElement();
}
```
准备工作完毕，下面开干~

## 开干
```java
public class PackageClass {

    public static void main(String[] args) throws IOException {

        Set<Class<?>> classSet = new HashSet<>();
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        String packageName = "com.gistmap.hello";
        Enumeration<URL> urls  = classLoader.getResources(packageName.replace(".","/"));
        while (urls.hasMoreElements()) {
            URL url = urls.nextElement();
            if (url != null) {
                String protocol = url.getProtocol();
                if (protocol.equals("file")) {
                    String packagePath = url.getPath();
                    addClass(classSet,packagePath,packageName);
                }
            }
        }
        classSet.forEach(c-> System.out.println(c.toString()));
    }

    private static void addClass(Set<Class<?>> classSet, String packagePath, String packageName) {
        File[] files = new File(packagePath).listFiles(
                f -> f.isDirectory() ||
                    (f.isFile() && f.getName().endsWith(".class")));
        for (File file : files) {
            String fileName = file.getName();
            if (file.isFile()) {
                String className = fileName.substring(0, fileName.lastIndexOf("."));
                if (!className.equals("") && className != null) {
                    className = packageName + "." + className;
                }
                doAddClass(classSet,className);
            } else {
                String subPackagePath = fileName;
                if (!subPackagePath.equals("") && subPackagePath != null) {
                    subPackagePath = packagePath + "/" + fileName;
                }
                String subPackageName = fileName;
                if (!subPackageName.equals("") && subPackageName != null) {
                    subPackageName = packageName + "." + subPackageName;
                }
                addClass(classSet, subPackagePath, subPackageName);
            }
        }
    }

    private static void doAddClass(Set<Class<?>> classSet, String className) {
        Class<?> cls = loadClass(className, false);
        classSet.add(cls);
    }

    private static Class<?> loadClass(String className, boolean b) {
        Class<?> cls;
        try {
            cls = Class.forName(className, false, Thread.currentThread().getContextClassLoader());
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
        return cls;
    }

}
```

以上就把一个包名下的类的字节码全放到HashSet里的，就可以直接用Class.newInstance()调用。以上~