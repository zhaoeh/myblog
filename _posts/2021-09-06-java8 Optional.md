---
layout:     post
title:      java8 Optional
subtitle:   Optional主要用来避免空指针和适当的if替换
categories: [JAVA8]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 什么是Optional？
NullPointerException （空指针异常）相信每个JAVA程序员都不陌生，是JAVA应用程序中最常见的异常。   
之前，Google Guava项目曾提出用Optional类来包装对象从而解决 NullPointerException （空指针异常）。受此影响，JDK8的类中也引入了Optional类，在新版的SpringData Jpa和Spring Redis Data中都已实现了对该方法的支持。     

Optional 类是一个属性value可以为null值的容器对象，它可能包含空值，也可能包含非空值。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。   
Optional 是个容器：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。   
Optional 类的引入很好的解决空指针异常。   

# 2. Optional API
```java
// 无参构造，构造一个空Optional
private Optional();

// 根据传入的非空value构建Optional
private Optional(T value); 

// 返回一个空的Optional，该实例的value为空
public static<T> Optional<T> empty();

// 根据传入的非空value构建Optional，与Optional(T value)方法作用相同
public static <T> Optional<T> of(T value);

// 与of(T value)方法不同的是，ofNullable(T value)允许你传入一个空的value
// 当传入的是空值时其创建一个空Optional，当传入的value非空时，与of()作用相同
// 这个包装器方法非常常用
public static <T> Optional<T> ofNullable(T value)

// 返回Optional的值，如果容器为空，则抛出NoSuchElementException异常
public T get();

// 判断当前Optional是否已设置了值
public boolean isPresent();

// 判断当前Optional是否已设置了值，如果有值，则调用Consumer函数式接口进行处理
public void ifPresent(Consumer<? super T> consumer);

// 如果设置了值，且满足Predicate的判断条件，则返回该Optional，否则返回一个空的Optional
public Optional<T> filter(Predicate<? super T> predicate);

// 如果Optional设置了value，则调用Function对值进行处理，并返回包含处理后值的Optional，否则返回空Optional
public<U> Optional<U> map(Function<? super T, ? extends U> mapper);

// 与map()方法类型，不同的是它的mapper结果已经是一个Optional，不需要再对结果进行包装
public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper);

// 如果Optional值不为空，则返回该值，否则返回other
public T orElse(T other);

// 如果Optional值不为空，则返回该值，否则根据other另外生成一个
public T orElseGet(Supplier<? extends T> other);


// 如果Optional值不为空，则返回该值，否则通过supplier抛出一个异常
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X
```

# 3. 一个简单的案例
```java
package zeh.myjavase.code42java8.demo08;

import org.junit.Test;

import java.util.Optional;

public class OptionalRun {

    public static void main(String[] args) {
        Optional<String> optional = Optional.ofNullable(null);
        System.out.println("optional 存在非空值?" + optional.isPresent());
        System.out.println("存在值则返回值，否则为null值则返回指定值：" + optional.orElse("空值"));
        System.out.println("存在值则返回值，否则为null值则根据lambda的供给型逻辑返回指定值：" + optional.orElseGet(() -> "null啊"));
        System.out.println("map方法转换当前optional中的值并返回转换后的optional实例：" + optional.map(e -> e + "!!!").orElse("为null啊"));
        System.out.println("optional 是否存在非空值，存在则执行回调逻辑：");
        optional.ifPresent(System.out::println);
        optional.ifPresent(System.out::println);
        
        // 可以使用Optional.empty()创建一个value属性为空的Optional对象。但是该对象并不是空的，它只是里面的值是空的，它本身是一个Optional对象。
        // 如果要构建一个空的对象，new Object()是空的。
        System.out.println("Optional:" + Optional.empty());
    }
}

```

# 4. 介绍其他博主的文章
[原始链接](https://segmentfault.com/a/1190000012263070)    
本篇介绍的Optional虽然并不是一个函数式接口，但是也是一个极其重要的类。   
Optional并不是我们之前介绍的一系列函数式接口，它是一个class，主要作用就是解决Java中的NPE（NullPointerException）。空指针异常在程序运行中出现的频率非常大，我们经常遇到需要在逻辑处理前判断一个对象是否为null的情况。   
```java
if(null != person){
    Address address = person.getAddress();
    if(null != address){
        ......
    }
}
```
实际开发中我们经常会按上面的方式进行非空判断，接下来看下使用Optional类如何避免空指针问题:   
```java
String str = "hello";
Optional<String> optional = Optional.ofNullable(str);
optional.ifPresent(s -> System.out.println(s));//value为hello，正常输出
```
首先，ofNullable方法接收一个可能为null的参数，将参数的值赋给Optional类中的成员变量value，ifPresent方法接收一个Consumer类型函数式接口实例，再将成员变量value交给Consumer的accept方法处理前，会校验成员变量value是否为null，如果value是null，则什么也不会执行，避免了空指针问题。下方是ifPresent源码:   
```java
public void ifPresent(Consumer<? super T> consumer) {
    if (value != null)
        consumer.accept(value);
}
```
如果传入的内容是空，则什么也不会执行，也不会有空指针异常.   
```java
String str = null;
Optional<String> optional = Optional.ofNullable(str);
optional.ifPresent(s -> System.out.println(s));//不会输出任何内容
```
如果为空时想返回一个默认值:   
```java
String str = null;
Optional<String> optional = Optional.ofNullable(str);
System.out.println(optional.orElseGet(() -> "welcome"));
```
orElseGet方法接收一个Supplier，还记得前面介绍的Supplier么，不接受参数通过get方法直接返回结果，类似工厂模式，上面代码就是针对传入的str变量，如果不为null那正常输出，如果为null，那返回一个默认值"welcome"    
orElseGet方法源码:   
```java
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}
```