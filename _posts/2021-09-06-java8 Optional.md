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
很显然，Optional类支持泛型，其属性value可以是任何对象的实例，也可以是null值。   
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

# 3. 使用分析
## 3.1 创建一个Optional对象并获取它的value值
```java
package zeh.myjavase.code42java8.demo08;

import org.junit.Test;

import java.util.Optional;

public class OptionalRun {

    @Test
    public void testOfAndGet() {
        // of()方法不允许参数为null，否则将抛出空指针异常
        Optional<Integer> optional1 = Optional.of(10);

        // ofNullable()方法的参数无限制，可以是null值，也可以是指定类型的任何值
        Optional<Integer> optional2 = Optional.ofNullable(20);

        // 直接存入null值
        Optional<Integer> optional3 = Optional.ofNullable(null);

        // 直接打印上面的3个optional对象，默认使用Optional对象覆盖的toString()方法的格式来输出
        System.out.println("optional1 is : " + optional1);
        System.out.println("optional2 is : " + optional2);
        System.out.println("optional3 is : " + optional3);

        // 上面我们直接输出的是optional对象，但实际上它只是个容器，我们需要的是它里面包装的真正的value对象
        // 使用get方法可以获取optional容器里面的value属性值
        System.out.println("optioanl1 value is : " + optional1.get());
        System.out.println("optioanl2 value is : " + optional2.get());
        // 注意，当value为null值时，直接get会抛出一个异常：throw new NoSuchElementException("No value present")
        System.out.println("optioanl3 value is : " + optional3.get());
    }
}

```
运行：   
```youtrack
optional1 is : Optional[10]
optional2 is : Optional[20]
optional3 is : Optional.empty
optioanl1 value is : 10
optioanl2 value is : 20

java.util.NoSuchElementException: No value present

	at java.util.Optional.get(Optional.java:135)
	at zeh.myjavase.code42java8.demo08.OptionalRun.testOfAndGet(OptionalRun.java:97)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:47)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:44)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:271)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:70)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:50)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:238)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:63)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:236)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:53)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:229)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:309)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:160)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
```

（1）of方法是Optional对象里面的一个静态方法，接收一个非null值的指定泛型类型的对象，直接返回一个Optional实例：   
```java
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }
```
new Optional<>(value)构造器源码：   
```java
    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }
```
Objects.requireNonNull(value)源码：   
```java
    public static <T> T requireNonNull(T obj) {
        if (obj == null)
            throw new NullPointerException();
        return obj;
    }
```
可以发现，当使用构造器构造value值时，如果为null值，则直接抛出一个空指针异常。    

（2）ofNullable()方法和of方法类似，不同的是它允许传入一个null值，也就是它对传入的值没有任何限制，可以是null值，也可以是指定泛型类型的任何值，使用频率非常高：      
```java
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }
```
empty()源码：   
```java
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
```
EMPTY常量源码：   
```java
private static final Optional<?> EMPTY = new Optional<>();
```
这个方法里面判断了value是不是为null，如果为null，则构建一个空的Optional实例。   

（3）直接输出optional对象，调用的是Optional类覆盖的toString()方法：   
```java
    @Override
    public String toString() {
        return value != null
            ? String.format("Optional[%s]", value)
            : "Optional.empty";
    }
```
如果value不为null值，则按照 “Optional[%s]” 的格式输出；否则直接输出“Optional.empty”。   

（4）get()方法用于获取Optional对象里面存储的value值：   
```java
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
```
可以看出，使用get()方法获取value属性值时，如果value为null值，则直接抛出一个NoSuchElementException异常。   
这也是上面代码最后一行执行后抛出Exception的原因，也正是新手最爱犯的错误。    
正因为会抛出异常，因此这个方法使用频率不高。   

## 3.2 正确获取Optional对象中的value值
上一个案例直接使用get()方法获取Optional对象中的value值，当value不为null时可以直接拿到，但是当value为null时会抛出一个NoSuchElementException异常。   
那么该如何正确的获取Optional对象中的value值呢？   
```java
    // 正确获取Optional对象中的value值
    @Test
    public void testGetValue(){
        Optional<String> optional = Optional.ofNullable(null);

        // 方式1
        System.out.println("value是否存在：" + optional.isPresent());
        if(optional.isPresent()){
            System.out.println("方式1： value is : " + optional.get());
        }

        // 方式2
        System.out.println("方式2：value is : " + optional.orElse("值不存在啊！"));

        // 方式3
        System.out.println("方式3：value is : " + optional.orElseGet(()->{
            System.out.println("orElseGet run");
            return "没有值啊！";
        }));
    }
```
运行：   
```java
value是否存在：false
方式2：value is : 值不存在啊！
orElseGet run
方式3：value is : 没有值啊！
```
（1）isPresent()方法判断Optional容器中的value属性是否不为null值，如果为null则返回false，表示不存在；否则返回true，表示value有值。即当Optional实例的value值非空时返回true，否则返回false.   
```java
    public boolean isPresent() {
        return value != null;
    }
``` 
从日志结果可看出，当value为null时，该方法返回false，因此， System.out.println("方式1： value is : " + optional.get()); 这句没有执行。   

（2）orElse(other)方法，表示如果当前Optional对象中的value不为null值则返回value，否则直接返回给定的other值。    
源码如下：   
```java
    public T orElse(T other) {
        return value != null ? value : other;
    }
```

（3）orElseGet(Supplier)犯法，表示如果当前Optional对象中的value不为null值则直接返回false，否则使用传入的Supplier对象去构建一个返回值。    
```java
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }
```

请注意，尽量使用orElse()或者orElseGet()方法去替代isPresent()方法，为啥这么说？请看下面的案例。   
如果要获取user对象中的name，使用传统的写法如下：   
```java
public static String getName(User u) {
    if (u == null || u.name == null)
        return "Unknown";
    return u.name;
}
```
使用Optional如下：   
```java
public static String getName(User u) {
    Optional<User> user = Optional.ofNullable(u);
    if (!user.isPresent())
        return "Unknown";
    return user.get().name;
}
```
what？这就是Optional？怎么是这幅鬼样子？   
前面不要写成上面那样。   
这样改写非但不简洁，而且实际操作和传统的写法有啥区别？无非就是用 isPresent 方法来替代 u==null 。这样的改写并不是Optional正确的用法，我们再来改写一次。   
```java
public static String getName(User u) {
    return Optional.ofNullable(u)
                    .map(user->user.name)
                    .orElse("Unknown");
}
```
简单吧。这样才是正确使用Optional的姿势。   

暖心建议：   
<font color="#dc143c"><b>尽量避免在程序中直接调用 Optional 对象的 get() 和 isPresent() 方法。</b></font>         

直接调用 get() 方法是很危险的做法，如果 Optional 的值为空，那么毫无疑问会抛出 NoSuchElementException 异常。而为了调用 get() 方法而使用 isPresent() 方法作为空值检查，这种做法与传统的用 if 语句块做空值检查没有任何区别。      

## 3.3 orElse和orElseGet方法到底有啥区别？
从上一个案例可以看出，为了避免直接使用get方法获取value时有可能碰到异常的问题，建议使用orElse或者orElseGet方法去替代get方法，这样，当value为null值时，就可以返回我们指定的值或者使用传入的Supplier函数式对象去构建一个值返回。   
如此看来，这两个方法实现的功能比较相似，那到底这两个方法有啥区别呢？   
来一个案例：   
```java
    @Test
    public void testOrElseGet() {
        System.out.println("when value is null...");
        Optional.ofNullable(null).orElse(get("a"));
        Optional.ofNullable(null).orElseGet(() -> get("b"));
        System.out.println("\nwhen value not is null...");
        Optional.ofNullable("daisy").orElse(get("a"));
        Optional.ofNullable("daisy").orElseGet(() -> get("b"));
    }

    private String get(String str) {
        System.out.println(str + ":i have to be ruined.");
        return str;
    }
```
运行结果：   
```youtrack
when value is null...
a:i have to be ruined.
b:i have to be ruined.

when value not is null...
a:i have to be ruined.
```
从日志可以看出：   
（1）如果Optional容器里面的value是null值，则两者执行原理一模一样。   
（2）当里面的值不为null值时，orElse()方法还是会执行里面获取默认值的逻辑；而orElseGet()方法就不会执行Supplier函数逻辑了。   
这里其实就可以看出函数式编程的底层内核逻辑了，orElse()方法里面接收的始终是一个值，那么至于你如何获取到这个值，那你自己负责实现，但是只要调用了orElse方法，这个值就必须得产生，因此orElse方法无论value是不是null，它接收的值的产生逻辑都必然要执行。   
而orElseGet()方法接收的是一个Supplier函数式对象，传入一个Lambda表达式，只是一个逻辑而已，不代表这个逻辑一定被执行，何时触发，取决于该函数式对象何时调用其方法。   
函数式编程的一大好处就是提供了逻辑的延迟执行能力。   

明白了这种区别的话，为了提升性能，一般建议使用orElseGet()去获取Optional容器里面的value值。   
但凡事不是绝对的，orElse()里面获取默认值的逻辑始终会执行的特性，有时候可以帮我们做一些额外的逻辑处理。   
```java
    private Map<String, String> cache = new ConcurrentReferenceHashMap<>();

    @Test
    public void testOrElse() {
        Optional.ofNullable("Eric").orElse(cacheValue("Marshall"));
    }

    private String cacheValue(String value) {
        cache.put("cache_value", value);
        System.out.println("value is : " + value);
        return value;
    }
```
上面的代码借助了orElse方法的特性，不管容器里面有没有值，获取默认值的逻辑都始终执行，因此，我们可以在这个逻辑里面将我们设置的默认值进行缓存、打印、也可以做任何和默认值无关的逻辑，比如查询DB、发送MQ消息等。   

当然，如果我们想实现一种场景：当Optional容器里面的value为null值时，获取默认值的同时再做一些逻辑处理，这时候就可以借助orElseGet的特性。   
因为orElseGet只有在容器中的value为null时才会执行，因此，刚好可以满足我们这个场景。   
比如从db中查询一条数据，如果这条数据为null的话，那么我们需要发送一个MQ，或者推送一封邮件，此时这个动作就可以写在orElseGet逻辑中！   

## 3.4 orElseThrow方法
和上面的orElse、orElseGet方法不同的是，这个方法当Optional容器中的value不为null值时，返回value；   
否则，允许我们抛出一个自定义异常。    
和orElseGet方法非常类似，也是接收一个Supplier函数式对象，只不过它接收的这个函数式对象返回的元素必须是一个Throwable类型或者其子类型，也就是必须是一个返回Throwable对象的Supplier。   
```java
    // 当value不为null值时返回value；否则通过传入的Supplier构建一个异常对象
    @Test
    public void testOrElseThrow() throws Exception {
        Optional.ofNullable(null).orElseThrow(() -> new Exception("value不存在"));
    }
```
运行：   
```youtrack
java.lang.Exception: value不存在

	at zeh.myjavase.code42java8.demo08.OptionalRun.lambda$testOrElseThrow$3(OptionalRun.java:159)
	at java.util.Optional.orElseThrow(Optional.java:290)
	at zeh.myjavase.code42java8.demo08.OptionalRun.testOrElseThrow(OptionalRun.java:159)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:47)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:44)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:271)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:70)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:50)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:238)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:63)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:236)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:53)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:229)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:309)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:160)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
```
上述代码，value为null值，通过传入的Supplier函数式对象构造了一个自定义的Exception实例。   

orElseThrow方法源码如下：   
```java
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }
```
从源码可以清晰的看出来，的那个value不为null值时，直接返回value；否则通过传入的Supplier对象提供一个异常返回值。    
注意，这里面的Supplier泛型为X，X是这个方法中单独定义的泛型类型，它继承Throwable，因此这里的Supplier对象里面的元素类型必须符合<X extends Throwable>。   

## 3.5 空Optional对象
Optional.empty() 方法是一个静态方法，上面已经看了它的源码，不妨再看一下：   
```java
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
```
```java
private static final Optional<?> EMPTY = new Optional<>();
```

ofNullable方法当传入的value是null值时，也是调用了empty()方法。    
```java
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }
```
因此，直接调用empty()方法和调用ofNullable()方法传入一个null值，最终返回的都是同一个实例对象，那就是下面这个：   
```java
private static final Optional<?> EMPTY = new Optional<>();
```

我们看一个案例：   
```java
    @Test
    public void testOptionalEmpty() {
        Optional<Integer> optional1 = Optional.<Integer>ofNullable(null);
        Optional<Integer> optional2 = Optional.<Integer>ofNullable(null);
        // 相等
        System.out.println("optional1 == optional2 ? " + (optional1 == optional2));
        // 相等
        System.out.println("optional1 == empty() ? " + (optional1 == Optional.<Integer>empty()));

        // 哪怕泛型类型不一样，也相等
        Object obj1 = Optional.<Integer>empty();
        Object obj2 = Optional.<String>empty();
        System.out.println("obj1 == obj2 ? " + (obj1 == obj2));
    }
```
运行结果：   
```youtrack
optional1 == optional2 ? true
optional1 == empty() ? true
obj1 == obj2 ? true
```
验证了源码的逻辑，ofNullable()方法当传入null值时，底层调用的就是empty()方法，最终返回的是一个单例Optional对象，一个特殊的Optional对象，里面的value值永远是null。   
这也就是Optional是个容器的原因，它将null值也进行了包装，封装成了一个 EMPTY 的Optional对象。   
哪怕构建空的Optional对象时指定的泛型类型完全不想通，它也始终复用一个单例 EMPTY。   

## 3.6 ifPresent
上面学习了一个isPresent方法，表示当Optional容器的value不为null时返回true，否则返回false。注意，如果为null值直接get会抛出异常。    
这个这个方法是ifPresent，简直是一字之差，特别像，不要搞混淆了。   
ifPresent(Consumer consumer)：如果option对象保存的值不是null（即value存在的话），则执行consumer对象消费该value，否则不执行。          
```java
    @Test
    public void testIfPresent() {
        System.out.println("begin case1:");
        Optional.ofNullable(null).ifPresent(e -> System.out.println("handel value,value is :" + e));

        System.out.println("begin case2:");
        Optional.<String>ofNullable("Daisy").ifPresent(e -> System.out.println("handel value,value is :" + e));
    }
```
运行：   
```youtrack
begin case1:
begin case2:
handel value,value is :Daisy
```
从日志分析，当value不为null时，回调了Consumer的逻辑对该对象进行了消费；    
如果为null，则不做任何操作。   
这个方法也很有用，如果我们的业务需要在返回的数据存在的情况下，直接对该数据进行某种处理，就可以直接使用这个方法。   

## 3.7 map和flatMap
map()方法，当上游Optional容器中的value值不为null时，根据传入的Function函数式对象将上游的value值映射为另外一个Optional对象，请注意，map()方法接收的Function函数式对象的入参就是上游Optional容器中的value，出参是映射后的另外一个对象（类型可以自定义），但是map方法内部自动对Function的出参做了Optional包装，即将Function逻辑的出参对象自动包装成了一个新的Optional容器。      
flat()方法，和map()方法类似，不同的是它接收的Function函数式对象，出参必须得是一个包装后的Optional对象，也就是说flatMap()方法要求我们传入的Function对象，接收一个上游的value，返回另外一个包装后的Optional对象。它不像map()方法内部会对Function对象执行后的返回值自动做Optional包装，而是要求我们自己在Function对象中处理对于返回值的Optional包装。   

map方法源码：   
```java
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```   
flatMap方法源码：   
```java
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }
```
对比如下：   
![](/images/myBlog/2021-09-06_map_flatmap.jpg)   
从图中明显看出来，map方法对于Function对象的执行结果自动进行了Optional包装；而flatMap需要我们自己在Function对象中主动返回一个Optional包装对象。   

案例：   
```java
    @Test
    public void testMap() {
        // 使用map
        System.out.println(Optional.ofNullable("Daisy").map(e -> e.replace("aisy", "eric")).orElse("Marshall"));

        // 使用flatMap
        System.out.println(Optional.ofNullable("Daisy").flatMap(e -> Optional.ofNullable(e.replace("aisy", "eric"))).orElse("Marshall"));
    }
```
运行：   
```youtrack
Deric
Deric
```
截止到目前，似乎除了从源码层面分析出Optional的map和flatMap方法的区别之外，在使用层面感觉不到到底有啥区别啊。    
无非就是map方法需要我们直接返回原始的映射后的数据就行，其内部会自动替我们包装成Optional对象返回；   
而flatMap方法需要我们自己的Function对象逻辑中奖返回的数据包装成一个Optional对象返回，其内部直接使用我们返回的这个Optional作为方法的返回值。    
仅此而已。   

### 3.7.1 Optional的map和flatMap到底有啥区别？
源码上很好理解，map将传入的Function对象的返回值自动包装成了一个新的Optional对象；而flatMap方法里面没有自动包装，而是需要我们在Function函数式对象逻辑中手动将返回值包装为Optional对象。   
但到底有啥区别呢？什么场景下用map？什么场景下用flatMap？      
看案例，先创建3个相关的级联对象：   
```java
package zeh.myjavase.code42java8.demo08.beans;

import lombok.Data;

import java.util.Optional;

// 学校类
@Data
public class MySchool {

    private String name;

    // 学校里面维护一个老师
    private Optional<MyTeacher> myTeacher;
}

```
```java
package zeh.myjavase.code42java8.demo08.beans;

import lombok.Data;

import java.util.Optional;

// 老师类
@Data
public class MyTeacher {

    private String name;

    // 老师里面维护一个学生
    private Optional<MyStudent> myStudent;
}

```
```java
package zeh.myjavase.code42java8.demo08.beans;

import lombok.Data;

// 学生类
@Data
public class MyStudent {
    private String name;
    private int age;
}

```
（1）MySchool持有一个MyTeacher，但这个MyTeacher是使用Optional包装后的；   
（2）MyTeacher持有一个MyStudent，这个MyStudent也是使用Optional包装后的；   
（3）MyStudent定义了自己的name和age。   

现在我们想从school对象中获取到student的name属性值，我们很容易写出下面的代码：   
```java
Optional.ofNullable(new MySchool()).map(MySchool::getMyTeacher).map(MyTeacher::getMyStudent).map(MyStudent::getName).orElse("not found student");
```
但这行代码在编译器里面直接就是报错的，不信看下图：   
![](/images/myBlog/2021-09-06_map_flatmap_error.jpg)    
这是怎么一回事呢？我们详细分析下：   
（1）Optional.ofNullable(new MySchool())：这行代码传入一个MySchool对象，将其包装成了一个Optional容器；   
（2）.map(MySchool::getMyTeacher)：当MySchool对象不为null值时，执行map方法中传入的Function函数式对象返回一个新的值，那MySchool::getMyTeacher返回的是啥呢？是Optional<MyTeacher>。而map方法会自动将Function函数式对象的返回值再次进行Optional包装，因此这行代码执行完成后拿到的Optional对象就是：   
```java
Optional<Optional<MyTeacher>> myTeacher;
```
它对Function函数式对象返回的Optional<MyTeacher>再次进行了Optional包装，就成了上面的样子。   
（3）.map(MyTeacher::getMyStudent)：从 Optional<Optional<MyTeacher>> myTeacher 中获取value，然后对value进行映射，而获取到的value是啥呢？是Optional<MyTeacher>。   
也就是此处拿到的value实际上是一个Optional对象，而 MyTeacher::getMyStudent 的意思是，给我一个teacher，我返回一个 student。   
现在给我的并不是一个teacher，而是一个  Optional<MyTeacher>   。   
Optional<MyTeacher> 中根本就不存在给我一个 MyTeacher 数据，返回一个 Optional<MyStudent> 的逻辑，因此不匹配了。   
后面的报错也是基于这个原因。    

我们可以想一下，这个编译错误，根本还是因为我们使用map()方法对上游传入的对象自动进行了Optional包装，如果上游传入的对象本身就是一个Optional的话，那再次包装就成了Optional的级联嵌套包装了。   
如何改造呢？   
在map()方法返回的Optional对象后面通过get取出内层的Optional对象就行了。   
```java
Optional.ofNullable(new MySchool()).map(MySchool::getMyTeacher).get().map(MyTeacher::getMyStudent).get().map(MyStudent::getName).orElse("not found student");
```
运行下：   
```java
java.util.NoSuchElementException: No value present

	at java.util.Optional.get(Optional.java:135)
	at zeh.myjavase.code42java8.demo08.OptionalRun.testMapAndFlatMapDiff(OptionalRun.java:179)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:47)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:44)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:271)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:70)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:50)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:238)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:63)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:236)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:53)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:229)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:309)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:160)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
```
发现报错了。    
这个错误很常见，就是因为我们直接使用get，而上游的Optional对象里面的value实际上是null值。   
而且，对于上游Optional容器中得到的value本身就是个Optional对象的这种场景，使用map再结合get()方法手动获取一次真实的value值，很明显这不是Optional的正确使用姿势。   

对于这种情况，一种很好的方式就是直接使用flatMap()方法。   
flatMap()说白了就是Optional的扁平化，如果上游传递下来的value本身就是个Optional对象，flatMap可以直接直接映射这个Optional对象。    
上面的方式采用如下：   
```java
    // map和flatMap到底有啥区别？
    @Test
    public void testMapAndFlatMapDiff() {
        MyStudent myStudent = new MyStudent();
        myStudent.setName("Eric");
        myStudent.setAge(22);
        MyTeacher myTeacher = new MyTeacher();
        myTeacher.setMyStudent(Optional.ofNullable(myStudent));
        MySchool mySchool = new MySchool();
        mySchool.setMyTeacher(Optional.ofNullable(myTeacher));
        System.out.println(Optional.ofNullable(mySchool).flatMap(MySchool::getMyTeacher).flatMap(MyTeacher::getMyStudent).map(MyStudent::getName).orElse("not found student"));
    }
```
运行：   
```youtrack
Eric
```
使用flatMap需要注意：   
（1）我们要明确知道，哪个位置需要流的扁平化，像我们上面的案例，mySchool是需要扁平化处理的，因为mySchool返回的value值本身就是个Optional；myTeacher也是需要扁平化处理的，因为myTeacher返回的value值本身也是个Optional；myStudent就不需要扁平化处理了，因为myStudent持有的value并不是一个Optional容器，因此，只是使用map去映射myStudent容器里面的name即可。   
（2）使用flatMap，一定要注意，对于flatMap()中传入的Function逻辑最终返回的Optional一定要真实存在，而不能返回的是个null。   
从flatMap()源码可以看到这一点：   
```java
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }
```
```java
    public static <T> T requireNonNull(T obj) {
        if (obj == null)
            throw new NullPointerException();
        return obj;
    }
```
可以明显看到，flatMap()接收的Function函数式对象一旦执行完逻辑返回的是个null值的话，则调用flatMap会抛出空指针异常。    
因此在上面的案例中，我们得确保如下事项：   
```youtrack
MySchool::getMyTeacher：该逻辑返回的Optional不能是null，哪怕是个Optional.empty()，都不能是null。
MyTeacher::getMyStudent：同理
```

flatMap实际上就是对Optional容器的扁平化处理，如何理解这个扁平化处理呢？   
扁平化的意思，实际上就是对数据源存在多层Optional嵌套的容器，一步一步将其拆解聚合，最终找到最内层的基本元素。   
对于嵌套层级很多的Optional，一直使用flatMap便可以直接将嵌套层级很多的Optional对象一路扁平化，最终映射成最底层的对象：       
```java
    // flatMap的扁平化处理
    @Test
    public void testFlatMap() {
        System.out.println(Optional.ofNullable(
                Optional.ofNullable(
                        Optional.ofNullable(
                                Optional.ofNullable("Daisy")))).
                flatMap(Function.identity()).
                flatMap(Function.identity()).
                flatMap(Function.identity()).get());
    }
```
运行：   
```java
Daisy
```
（1）数据源是一个4层Optional嵌套的对象，因此，需要从外向内，使用flatMap一步步拆解。   
（2）第一个flatMap拿到的是第二层的Optional；第二个flatMap拿到的是第三层的Optional；第三个flatMap拿到的是第四层的Optional。    
注意第四层的Optional对象是Optional.ofNullable("Daisy")，它的内层value是个String类型，而不是一个Optional。    
我们的目的就是拿到第四层Optional对象中的属性value，因此，直接get或者使用orElse等方法获取其中存储的value值即可。   

这时候我们已经很清楚map和flatMap的应用区别了：   
在通过Optional对象获取嵌套级联属性值时，如果某个对象实例的属性本身就是Optional包装过的类型，那么我们就要使用flatMap()去映射它，直接取出这个包装过的Optional，而不用使用map()让其自动再包装一层Optional后，我们再手动get去拆包装（脱裤子放屁多此一举）。   
对于在映射过程中，上游链路返回的容器属性value是其他类型的，我们就需要使用map去让内部自动为我们包装成一个Optional，而不用使用flatMap()，因为这种情况使用flatMap还需要我们自己在Function逻辑中手动对上游的value进行Optional包装，干脆就直接使用map，让map方法自动替我们包装Optional。      

### 3.7.2 map和flatMap的无限级联
上一个案例清楚了map和flatMap的基本使用姿势，这一节使用map和flatMap的一些高级用法。    
map和flatMap的好处就是可以从一个复杂级联对象中，无限级联操作直到取出最内层的那个非空对象值。    
在使用map或者flatMap进行级联对象属性映射过程中，只要中间任何一个对象为null值，将直接返回设定的默认值（假设最终使用orElse或者orElseGet进行取值的话）。   
下面我们看一个案例：   
先搞几个实体类，组装其中的级联依赖关系，假设有这样一个场景：   
一个公司，有员工，员工有一个卡片，卡片上有一个账号。    
我们要实现，给定一个公司实体，然后最终获取这个员工的账号。   
公司类：   
```java
package zeh.myjavase.code42java8.demo08.ref;

import lombok.Data;

import java.util.Optional;

// 一个公司
@Data
public class Company {

    // 公司持有员工
    private Staff staff;

    // 对应的Optional包装
    private Optional<Staff> optionalStaff;
}
```
员工类：   
```java
package zeh.myjavase.code42java8.demo08.ref;

import lombok.Data;

import java.util.Optional;

// 员工
@Data
public class Staff {

    // 员工持有卡片
    private Card card;

    // 对应的Optional包装
    private Optional<Card> optionalCard;
}

```
卡片类：   
```java
package zeh.myjavase.code42java8.demo08.ref;

import lombok.Data;

import java.util.Optional;

// 卡片
@Data
public class Card {

    // 卡片持有账号
    private NumberSer numberSer;

    // 对应的Optional包装
    private Optional<NumberSer> optionalNumberSer;
}

```
账号类：   
```java
package zeh.myjavase.code42java8.demo08.ref;

import lombok.Data;

// 账号
@Data
public class NumberSer {

    // 最内层的账号，持有一个字符串类型的账号
    private String number;
}


```
现在给定了一个Company对象，我们要获取出这个Company对象里面的number。   
这种对象就是一种级联对象。   
按照java8之前的写法应该是这样的：   
```java
    private String obtainNumber(Company company) {
        if (company != null) {
            Staff staff = company.getStaff();
            if (staff != null) {
                Card card = staff.getCard();
                if (card != null) {
                    NumberSer numberSer = card.getNumberSer();
                    if (numberSer != null) {
                        String number = numberSer.getNumber();
                        if (number != null) {
                            return number;
                        }
                    }
                }
            }
        }
        return "number不存在";
    }
```
这样的代码如果不进行拆解和单独封装，光是这些if非空判断，就足够让人喝一壶的了。    
我们看看如何通过Optional的无限级联去操作它：   
```java
    @Test
    public void testMapOrFlatMap() {
        Company company = new Company();
        System.out.println(obtainNumberByOptional(company));
    }

    private String obtainNumberByOptional(Company company) {
        return Optional.ofNullable(company).map(Company::getStaff).map(Staff::getCard).map(Card::getNumberSer).map(NumberSer::getNumber).orElse("number不存在");
    }
```
看看那一坨map的链式级联调用，优雅不优雅？爽不爽？   
我们尝试运行一下：   
```youtrack
number不存在
```
这是因为我们给了一个空的Company对象。   
接下来，我们封装一个完整的级联对象出来，再进行测试：   
```java
    @Test
    public void testMapOrFlatMap() {
        Company company = new Company();
        System.out.println(obtainNumberByOptional(company));

        NumberSer numberSer = new NumberSer();
        numberSer.setNumber("123456789");
        Card card = new Card();
        card.setNumberSer(numberSer);
        Staff staff = new Staff();
        staff.setCard(card);
        company.setStaff(staff);
        System.out.println(obtainNumberByOptional(company));
    }

    private String obtainNumberByOptional(Company company) {
        return Optional.ofNullable(company).map(Company::getStaff).map(Staff::getCard).map(Card::getNumberSer).map(NumberSer::getNumber).orElse("number不存在");
    }
```
运行：   
```youtrack
number不存在
123456789
```
我们发现最后找到了我们想要的number。   

同理，我们再看看flatMap怎么玩。   
```java
    @Test
    public void testMapOrFlatMap() {
        Company company = new Company();
        System.out.println(obtainNumberByOptional(company));
    }

    private String obtainNumberByOptional(Company company) {
        return Optional.ofNullable(company).flatMap(Company::getOptionalStaff).flatMap(Staff::getOptionalCard).flatMap(Card::getOptionalNumberSer).map(NumberSer::getNumber).orElse("number不存在");
    }
```   
运行，发现报错了：   
```youtrack
java.lang.NullPointerException
	at java.util.Objects.requireNonNull(Objects.java:203)
```
其实我们前面已经说过了使用flatMap需要注意的事情，flatMap()中接收的Function返回的Optional对象不能是null值，这就要求我们的数据源在一开始就要保证它的Optional类型的级联属性必须进行实例化，否则在操作过程中遇到是null的Optional中间层，就直接报空指针了。   
我们将数据源彻底封装一下吧：   
```java
    @Test
    public void testMapOrFlatMap() {
        NumberSer numberSer = new NumberSer();
        numberSer.setNumber("123456789");
        Card card = new Card();
        card.setOptionalNumberSer(Optional.ofNullable(numberSer));
        Staff staff = new Staff();
        staff.setOptionalCard(Optional.ofNullable(card));
        Company company = new Company();
        company.setOptionalStaff(Optional.ofNullable(staff));
        System.out.println(obtainNumberByOptional(company));
    }

    private String obtainNumberByOptional(Company company) {
        return Optional.ofNullable(company).flatMap(Company::getOptionalStaff).flatMap(Staff::getOptionalCard).flatMap(Card::getOptionalNumberSer).map(NumberSer::getNumber).orElse("number不存在");
    }
```
运行：   
```youtrack
123456789
```
这样就取出来了。   
但是没有发现，这样使用起来挺操蛋的吗？   
我们还得必须保证数据源中的Optional类型的级联属性都不能为null，这在真实业务中，根本没法保证。    
而且，在真实的业务操作中，实体的级联属性，没人会对级联字段再使用Optional去包装一层吧。   
因此，实际上flatMap在真是的业务场景中使用的不多。   
但如果级联属性都是自己负责开发的，那么就可以尝试使用这种方式，我们应该修改我们的所有实体中的Optional属性，给它一个默认值。   
```java
@Data
public class Company {

    // 公司持有员工
    private Staff staff;

    // 对应的Optional包装，给定一个空的Optional，如果有真正的值设置进来，就不使用这个空对象；否则使用它
    private Optional<Staff> optionalStaff = Optional.empty();
}
```
```java
@Data
public class Staff {

    // 员工持有卡片
    private Card card;

    // 对应的Optional包装
    private Optional<Card> optionalCard = Optional.empty();
}
```
```java
@Data
public class Card {

    // 卡片持有账号
    private NumberSer numberSer;

    // 对应的Optional包装
    private Optional<NumberSer> optionalNumberSer = Optional.empty();
}

```

我们再来运行上面的程序：   
```java
    @Test
    public void testMapOrFlatMap() {
        Company company = new Company();
        System.out.println(obtainNumberByOptional(company));

        NumberSer numberSer = new NumberSer();
        numberSer.setNumber("123456789");
        Card card = new Card();
        card.setOptionalNumberSer(Optional.ofNullable(numberSer));
        Staff staff = new Staff();
        staff.setOptionalCard(Optional.ofNullable(card));
        company = new Company();
        company.setOptionalStaff(Optional.ofNullable(staff));
        System.out.println(obtainNumberByOptional(company));
    }

    private String obtainNumberByOptional(Company company) {
        return Optional.ofNullable(company).flatMap(Company::getOptionalStaff).flatMap(Staff::getOptionalCard).flatMap(Card::getOptionalNumberSer).map(NumberSer::getNumber).orElse("number不存在");
    }
```
运行：   
```youtrack
number不存在
123456789
```
我们发现，使用flatMap，直接传入一个空的Company对象不再报错了。    

但尽管我们可以使用flatMap这么做，不过还是暖心建议：      
<font color="#dc143c"><b>避免使用 Optional 类型声明实体类的属性。</b></font>         

避免使用 Optional 作为实体类的属性，它在设计的时候就没有考虑过用来作为类的属性，如果你查看 Optional 的源代码，你会发现它没有实现java.io.Serializable 接口，这在某些情况下是很重要的（比如你的项目中使用了某些序列化框架），使用了 Optional 作为实体类的属性，意味着他们不能被序列化。      
<b>不要为了使用flatMap而刻意去操弄出一系列的Optional类型的级联属性，这完全没必要。</b>   

## 3.8 filter操作
filter()方法：当上游Optional传入的value值不为null时，判断该value是否满足给定的 Predicate 条件，如果满足条件，则返回上游传入的Optional；否则返回一个空的Optional。   
源码如下：   
```java
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }
```
可以看到，如果value为null值，则直接返回当前的Optional，相当于什么都不做。   
如果value不为null，则进一步执行predicate对象的条件方法，如果符合条件，则返回当前Optional；否则返回一个空的Optional。   
```java
    @Test
    public void testFilter() {
        System.out.println(Optional.ofNullable(null).filter(v -> v != null).orElse("value 不存在."));
    }
```
运行：   
```youtrack
value 不存在.
```
可以看到，Optional的filter操作比较简单，value如果不存在，则直接返回当前Optional啥也不做，如果value存在，则对value应用Predicate的过滤条件，如果符合条件则返回当前Optional对象，否则返回一个空的Optional。   

尽管filter的操作逻辑比较简单，但我们要多思考思考，能用filter操作来为我们带来什么方便优雅的编程方式呢？    
既然filter是对上游Optional里面的value值进行条件过滤，如果符合条件则返回Optional，否则直接返回一个空的Optional。那这意味着，filter操作实际上替我们执行了一个逻辑表达式，如果逻辑表达式成立，则返回一个value不为null值的Optional；否则，返回一个empty的Optional。    
基于这个原理，我们可以使用filter来替换一些不仅仅用于判断非空的逻辑表达式（对于!=null等非空判断，Optional本身就可以替换，不一定使用filter），最常见的方式是替换if语句。      
比如如下操作：   
```java
        String str = "eric";
        if (StringUtils.isNotBlank(str) && str.contains("r")) {
            System.out.println("eric contains r");
        } 
```
可以使用filter替换为如下：   
```java
Optional.ofNullable(str).filter(e->e.contains("r")).ifPresent(e-> System.out.println("eric contains r"));
```
这两者的效果是一样的。   

如果对于if else结果的改造，可能需要复杂一点。   
```java
        String str = "eric";
        if (StringUtils.isNotBlank(str) && str.contains("r")) {
            System.out.println("eric contains r");
        } else {
            System.out.println("eric not contains r");
        }
```
替换后如下：   
```java
    @Test
    public void testFilter() {
        String str = "eric";
        String get = Optional.ofNullable(str).filter(e -> e.contains("r")).orElseGet(() -> {
            System.out.println("eric not contains r");
            return null;
        });
        if (get != null) {
            System.out.println("eric contains r");
        }
    }
```
运行：   
```youtrack
eric contains r
```
修改eric为daisy，再次运行：   
```java
    @Test
    public void testFilter() {
        String str = "daisy";
        String get = Optional.ofNullable(str).filter(e -> e.contains("r")).orElseGet(() -> {
            System.out.println("eric not contains r");
            return null;
        });
        if (get != null) {
            System.out.println("eric contains r");
        }
    }
```
日志：   
```youtrack
eric not contains r
```
在这个我们看到，对于一般的if....语句，可以直接使用filter来替代它，然后直接使用ifPresent去执行符合if语句条件的case。    
对于包含if....else....的语句，可以使用filter替代它，然后借助orElseGet的特性，当value为null值时才执行orElseGet的Supplier函数式对象，在其逻辑中指定else的逻辑；      
然后再单独对返回的对象做非空判断，在里面做if的逻辑，从而简介的替代了if...else...的场景。   


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

# 5. 综合案例
```java
package zeh.myjavase.code42java8.demo08;

import org.junit.Test;

import java.util.Optional;

public class OptionalRun {

    @Test
    public void testOptionalBase(){
        Optional<String> optional = Optional.ofNullable(null);
        System.out.println("optional 存在非空值?" + optional.isPresent());
//        System.out.println("获取optional对象的value值，如果值为null则报空指针：" + optional.get());
        System.out.println("存在值则返回值，否则为null值则返回指定值：" + optional.orElse("空值"));
        System.out.println("存在值则返回值，否则为null值则根据lambda的供给型逻辑返回指定值：" + optional.orElseGet(() -> "null啊"));
        System.out.println("map方法转换当前optional中的值并返回转换后的optional实例：" + optional.map(e -> e + "!!!").orElse("为null啊"));
        System.out.println("optional 是否存在非空值，存在则执行回调逻辑：");
        optional.ifPresent(System.out::println);
        // 可以使用Optional.empty()创建一个value属性为空的Optional对象。但是该对象并不是空的，它只是里面的值是空的，它本身是一个Optional对象。
        // 如果要构建一个空的对象，new Object()是空的。
        System.out.println("Optional:" + Optional.empty());
    }
}

```
