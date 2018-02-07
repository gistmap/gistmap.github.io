---
title: Java 8 行为参数化
date: 2017-09-28 20:42:36
tags: [java,java8]
categories: java
---
## 前言
既然java9来了，那我们愉快的学习下java8 :)

## 准备
假定我们要完成一个筛选实体对象的方法，我们该怎样做，下面通过逐渐改进的方式来了解**行为参数化**

一个苹果类

```java
public class Apple {
        private String color;
        private Double weight;
        
    public Apple() {}

    public Apple(String color, Double weight) {
        this.color = color;
        this.weight = weight;
}
    //getter and setter
```


## 第一次尝试

> 得到颜色为红色的苹果？

我们可以直接写一个静态方法来完成这个要求


```java
    public static List<Apple> filterRedApples(List<Apple> inventory) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple: inventory){
            if ("red".equals(apple.getColor())) {
                result.add(apple);
            }
        }
        return result;
    }
    //List<Apple> redApples = filterRedApples(inventory);
```

> 那如果得到颜色为绿色的苹果呢？

## 第二次尝试
把颜色抽象出来

```java
    public static List<Apple> filterApplesByColor(List<Apple> inventory,
                                                 String color) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple: inventory){
            if (color.equals(apple.getColor())) {
                result.add(apple);
            }
        }
        return result;
    }
    //List<Apple> greenApples = filterApplesByColor(inventory, "green");
 ```


> 得到重量超过150g的苹果？

```java
    public static List<Apple> filterWeightApples(List<Apple> inventory,
                                                 double weight) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple: inventory){
            if (apple.getWeight() > weight) {
                result.add(apple);
            }
        }
        return result;
    }
```
    

> 得到重量超过150g，红色的苹果？

## 第三次尝试
把颜色和重量都抽出来作为参数

```java
    public static List<Apple> filterApples(List<Apple> inventory,
                                        String color, int weight) {
        List<Apple> result = new ArrayList<Apple>();
        for (Apple apple: inventory){
            if ( (apple.getColor().equals(color)) ||
                     apple.getWeight() > weight) {
                result.add(apple);
            }
        }
        return result;
    }
```

暂时的解决了问题，可随着实体类属性的增长，会变得非常臃肿且丑。。

## 第四次尝试
使用接口定制行为

```java
    public interface ApplePredicate{
        boolean test (Apple apple);
    }
    //筛选重量
    public class AppleHeavyWeightPredicate implements ApplePredicate{
        public boolean test(Apple apple){
            return apple.getWeight() > 150;
        }
    }
    //筛选颜色和重量
    public class AppleRedAndHeavyPredicate implements ApplePredicate{
        public boolean test(Apple apple){
            return "red".equals(apple.getColor()) && apple.getWeight() > 150;
        }
    }
    //筛选方法
    public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p){
        List<Apple> result = new ArrayList<>();
        for(Apple apple: inventory){
            if(p.test(apple)){
                result.add(apple);
            }
        }
        return result;
    }
    //使用
    List<Apple> redAndHeavyApples = filterApples(inventory, 
                                    new AppleHeavyWeightPredicate());
    List<Apple> redAndHeavyApples = filterApples(inventory, 
                                    new AppleRedAndHeavyPredicate());

```



这次我们试用了行为参数化的方式来改写：

 - 定义了一个ApplePredicate接口，包含一个对比的抽象方法
 - 定义一个类实现ApplePredicate接口并根据需求重写test方法
 - 写一个筛选方法，并把ApplePredicate接口作为参数
 - 使用比对方法来筛选合适的苹果

我们把实现了ApplePredicate接口的类中的筛选方法作为一个参数传递给了filterApples方法的第二个参数，我们有几种筛选策略，但在运行时，选择一种来执行。使用起来变得更加灵活,可扩展性也得到提升。

## 第五次尝试
看上去一大坨，我们可能只用一次此种策略的比对方法呢。用内部类改写一下。我们把上次ApplePredicate的2个实现类删除掉，保留其他代码。

```java
    List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
        public boolean test(Apple apple){
            return "red".equals(apple.getColor());
        }
    });
```

虽然短了些，但还是丑。

## 主角登场--lambda

```java
    List<Apple> result = filterApples(inventory, (Apple apple) -> 
                                        "red".equals(apple.getColor()));
    //甚至可以舍弃类型声明
    List<Apple> result = filterApples(inventory, (apple) -> 
                                        "red".equals(apple.getColor()));
    //括号也可以去掉
    List<Apple> result = filterApples(inventory, apple -> 
                                        "red".equals(apple.getColor()));
```


可是我们写了个接口，且只能对Apple类进行操作啊
我们把第四次尝试中的ApplePredicate接口改成如下：



```java
    @FunctionalInterface
    public interface Predicate<T> {
        boolean test(T t);
    }
    //这样使用
    static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple: inventory){
            if (p.test(apple)) {
                result.add(apple);
            }
        }
        return result;
    }
    //这样，即使比对Banana啊，茄子啊，苦瓜 等对象也可以咯
```


@FunctionalInterface 表示此接口是一个函数式接口，只包含一个抽象方法。他不是必须的，但使用它来标注一个接口是函数式接口是一个良好的规范，同时标注了此接口，抽象方法超过一个时，会编译报错。

> 那么@FunctionalInterface有什么卵用？

他和Lambda之间有千丝万缕的联系~自己看~可以看看*Comparator*，*Runnable*，*FileFilter*

## 结束



吐槽一下，我们还是要定义一个用来比对的接口啊。。

其实Java内置了几个非常实用的函数式接口，我们连接口都不用写就可以直接使用~

 - java.util.function.Predicate &lt;T&gt; 
 - java.util.function.Consumer &lt;T&gt;
 - java.util.function.Function &lt;T,R&gt; 

有兴趣的可以点进源码看一看，记方法返回值是一个不错的方法用于区别这三个接口的作用。对了，第一个内置的**Predicate**可直接用于以上的例子。