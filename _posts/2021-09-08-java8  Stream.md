---
layout:     post
title:      java8 Stream
subtitle:   Stream是java8为集合框架新增的一个流式操作
categories: [JAVA8]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. java8 Stream api
java8中除了改造原有的集合框架，往里面新增了一些default方法，以供外部方便调用之外，最值得关注的是，它还新增了一系列新的方法包，其中stream就是亮点之一。   

Stream是java8新增的java.util.stream包下的独立的一个接口（不是函数式接口），并且jdk为Stream接口增加了很多内置实现，目的就是将数据转换成流操作。   
常用于java中各种集合操作。   

# 2. Stream API
<b>Stream API的操作步骤：   
(1)、创建Stream对象：从一个数据源，比如集合或者数组中获取Stream对象（java8对指定的接口比如Collection扩展了直接获取Stream对象的默认方法）；   
(2)、中间操作：一个操作的中间链，对数据源的数据进行操作(起始于Stream对象，调用Stream接口被实现的各种方法，这些方法往往返回的也是一个Stream对象，从而实现链式编程)。   
(3)、终止操作：开始执行中间链操作，并产生结果。   
</b>
我们可以看下Stream的源码：
```java
package java.util.stream;

import java.nio.charset.Charset;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.Arrays;
import java.util.Collection;
import java.util.Comparator;
import java.util.Iterator;
import java.util.Objects;
import java.util.Optional;
import java.util.Spliterator;
import java.util.Spliterators;
import java.util.concurrent.ConcurrentHashMap;
import java.util.function.BiConsumer;
import java.util.function.BiFunction;
import java.util.function.BinaryOperator;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.IntFunction;
import java.util.function.Predicate;
import java.util.function.Supplier;
import java.util.function.ToDoubleFunction;
import java.util.function.ToIntFunction;
import java.util.function.ToLongFunction;
import java.util.function.UnaryOperator;

public interface Stream<T> extends BaseStream<T, Stream<T>> {

    Stream<T> filter(Predicate<? super T> predicate);

    <R> Stream<R> map(Function<? super T, ? extends R> mapper);

    IntStream mapToInt(ToIntFunction<? super T> mapper);

    LongStream mapToLong(ToLongFunction<? super T> mapper);

    DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);

    <R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);

    IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper);

    LongStream flatMapToLong(Function<? super T, ? extends LongStream> mapper);

    DoubleStream flatMapToDouble(Function<? super T, ? extends DoubleStream> mapper);

    Stream<T> distinct();

    Stream<T> sorted();

    Stream<T> sorted(Comparator<? super T> comparator);

    Stream<T> peek(Consumer<? super T> action);

    Stream<T> limit(long maxSize);

    Stream<T> skip(long n);

    void forEach(Consumer<? super T> action);

    void forEachOrdered(Consumer<? super T> action);

    Object[] toArray();

    <A> A[] toArray(IntFunction<A[]> generator);

    T reduce(T identity, BinaryOperator<T> accumulator);

    Optional<T> reduce(BinaryOperator<T> accumulator);

    <U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);

    <R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner);

    <R, A> R collect(Collector<? super T, A, R> collector);

    Optional<T> min(Comparator<? super T> comparator);

    Optional<T> max(Comparator<? super T> comparator);

    long count();

    boolean anyMatch(Predicate<? super T> predicate);

    boolean allMatch(Predicate<? super T> predicate);

    boolean noneMatch(Predicate<? super T> predicate);

    Optional<T> findFirst();

    Optional<T> findAny();

    // Static factories

    public static<T> Builder<T> builder() {
        return new Streams.StreamBuilderImpl<>();
    }

    public static<T> Stream<T> empty() {
        return StreamSupport.stream(Spliterators.<T>emptySpliterator(), false);
    }

    public static<T> Stream<T> of(T t) {
        return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
    }

    @SafeVarargs
    @SuppressWarnings("varargs") // Creating a stream from an array is safe
    public static<T> Stream<T> of(T... values) {
        return Arrays.stream(values);
    }

    public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {
        Objects.requireNonNull(f);
        final Iterator<T> iterator = new Iterator<T>() {
            @SuppressWarnings("unchecked")
            T t = (T) Streams.NONE;

            @Override
            public boolean hasNext() {
                return true;
            }

            @Override
            public T next() {
                return t = (t == Streams.NONE) ? seed : f.apply(t);
            }
        };
        return StreamSupport.stream(Spliterators.spliteratorUnknownSize(
                iterator,
                Spliterator.ORDERED | Spliterator.IMMUTABLE), false);
    }

    public static<T> Stream<T> generate(Supplier<T> s) {
        Objects.requireNonNull(s);
        return StreamSupport.stream(
                new StreamSpliterators.InfiniteSupplyingSpliterator.OfRef<>(Long.MAX_VALUE, s), false);
    }

    public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b) {
        Objects.requireNonNull(a);
        Objects.requireNonNull(b);

        @SuppressWarnings("unchecked")
        Spliterator<T> split = new Streams.ConcatSpliterator.OfRef<>(
                (Spliterator<T>) a.spliterator(), (Spliterator<T>) b.spliterator());
        Stream<T> stream = StreamSupport.stream(split, a.isParallel() || b.isParallel());
        return stream.onClose(Streams.composedClose(a, b));
    }

    public interface Builder<T> extends Consumer<T> {

        @Override
        void accept(T t);

        default Builder<T> add(T t) {
            accept(t);
            return this;
        }

        Stream<T> build();

    }
}
```
Stream API常见方法(大多数内置方法都用来接收一个回调对象从而执行回调逻辑，优秀的是java8中实例化一个函数式接口对象可以采用Lambda表达式方便的进行创建)：  
```java
boolean allMatch(Predicate<? super T> predicate); //Stream所有元素都匹配指定的Predicate规则。   
boolean anyMatch(Predicate<? super T> predicate); //Stream部分元素匹配指定的Predicate规则。   
public static<T> Builder<T> builder();   
void close();   
<R, A> R collect(Collector<? super T, A, R> collector);//将Stream数据流按照指定的收集器规则聚合成目标类型的数据。   
<R> R collect(Supplier<R> supplier,   
BiConsumer<R, ? super T> accumulator,   
BiConsumer<R, R> combiner);   
public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b);   
long count();   
Stream<T> distinct();   
public static<T> Stream<T> empty();   
Stream<T> filter(Predicate<? super T> predicate);   
Optional<T> findAny();   
Optional<T> findFirst();   
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);   
DoubleStream flatMapToDouble(Function<? super T, ? extends DoubleStream> mapper);   
IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper);   
LongStream flatMapToLong(Function<? super T, ? extends LongStream> mapper);   
void forEach(Consumer<? super T> action);   
void forEachOrdered(Consumer<? super T> action);   
public static<T> Stream<T> generate(Supplier<T> s);   
boolean isParallel();   
public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) ;   
Iterator<T> iterator();   
Stream<T> limit(long maxSize);   
<R> Stream<R> map(Function<? super T, ? extends R> mapper);//将集合元素转换为其他对象。   
DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);   
IntStream mapToInt(ToIntFunction<? super T> mapper);   
LongStream mapToLong(ToLongFunction<? super T> mapper);   
Optional<T> max(Comparator<? super T> comparator);   
Optional<T> min(Comparator<? super T> comparator);   
boolean noneMatch(Predicate<? super T> predicate);   
public static<T> Stream<T> of(T t);   
public static<T> Stream<T> of(T... values);   
S onClose(Runnable closeHandler);   
S parallel();   
Stream<T> peek(Consumer<? super T> action);   
Optional<T> reduce(BinaryOperator<T> accumulator);   
T reduce(T identity, BinaryOperator<T> accumulator);//identity表示初始值；参数binaryOperator是一个函数接口，表示二元操作，可用于数学运算。   
<U> U reduce(U identity,   
BiFunction<U, ? super T, U> accumulator,   
BinaryOperator<U> combiner);   
S sequential();   
Stream<T> skip(long n);//跳过前几个元素   
Stream<T> sorted(); //按照默认比较规则对Stream进行排序。   
Stream<T> sorted(Comparator<? super T> comparator);//按照指定比较规则对Stream进行排序。   
Spliterator<T> spliterator();   
Object[] toArray();   
<A> A[] toArray(IntFunction<A[]> generator);   
S unordered();   
```
# 3. 实操演练
下来我们来看下操作Stream的案例。      
先定义一个实体类，用于充当集合中的元素：
```java
package zeh.myjavase.code42java8.demo06;

class ModeDemo06 {
    private String name;
    private Integer age;
    private String country;
    private char sex;

    public ModeDemo06(String name, Integer age, String country, char sex) {
        this.name = name;
        this.age = age;
        this.country = country;
        this.sex = sex;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
    }

    public char getSex() {
        return sex;
    }

    public void setSex(char sex) {
        this.sex = sex;
    }

    public String toString() {
        return "{name=" + this.getName() + ",age=" + this.getAge() + ",country=" + this.getCountry() + ",sex=" + this.getSex() + "}";
    }
}
```

接下来通过Stream API来操作集合。            
创建一个运行类，先组装一个List出来：   
```java
package zeh.myjavase.code42java8.demo06;

import org.junit.Before;
import org.junit.Test;

import java.util.ArrayList;
import java.util.Comparator;
import java.util.IntSummaryStatistics;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class StreamRun {
    private List<ModeDemo06> personList = new ArrayList<>();

    @Before
    public void packageList() {
        personList.add(new ModeDemo06("欧阳雪", 18, "中国", 'F'));
        personList.add(new ModeDemo06("Tom", 24, "美国", 'M'));
        personList.add(new ModeDemo06("Harley", 22, "英国", 'F'));
        personList.add(new ModeDemo06("向天笑", 20, "中国", 'M'));
        personList.add(new ModeDemo06("李康", 22, "中国", 'M'));
        personList.add(new ModeDemo06("小梅", 20, "中国", 'F'));
        personList.add(new ModeDemo06("何雪", 21, "中国", 'F'));
        personList.add(new ModeDemo06("李康", 22, "中国", 'M'));
        
        // 如果年龄为Null值的话，在进行获取后可能报空指针。
//        personList.add(new ModeDemo06("李康", null, "中国", 'M'));
//        personList.add(new ModeDemo06("李康", null, "中国", 'M'));
    }

}
```
## 3.1 Stream的filter方法
Stream的filter()方法用于对流中的元素进行过滤操作。   
（1）Stream的filter方法接受一个Predicate对象，使用Lambda表达式实现Predicate接口的抽象方法。   
（2）返回一个Stream对象，Predicate对象指定规则，返回的Stream是符合规则的过滤后的数据流。   
（3）filter()方法返回一个新的Stream，其和源Stream完全独立。   
（4）Collection集合在Java8中也增加了一个stream()默认方法，该方法返回Stream类型对象，用于将List转换为Stream。      
```java
    @Test
    public void testStream_filter() {
        // 1）找到年龄大于18岁的人并输出；
        // filter()是Stream接口中的方法，接收一个Predicate对象；
        // Predicate接口是函数式接口，所以使用Lambda表达式实现其对象并传递进去；
        // forEach()也是Stream接口的方法，接收默认的Consumer对象，同样Consumer接口是函数式接口，通过方法引用实现简写的Lambda表达式；
        // Predicate接口最开始是apache官方的第三方包，java8借鉴后自己也搞了一个。
        // 创建流：personList.stream()；中间操作：filter()；终止操作：foreach()。
        personList.stream().filter((p) -> p.getAge() > 18).forEach(System.out::println);

        // filter()方法是返回一个新的Stream，不会改变源数据。
        System.out.println("-------------------------------------------" + personList);

        // 2）找出所有中国人的数量
        //创建流：personList.stream()；中间操作：filter()；终止操作：count()。
        long chinaPersonNum = personList.stream().filter((p) -> p.getCountry().equals("中国")).count();
        System.out.println("中国人有：" + chinaPersonNum + "个");
    }
```
运行：   
```youtrack
{name=Tom,age=24,country=美国,sex=M}
{name=Harley,age=22,country=英国,sex=F}
{name=向天笑,age=20,country=中国,sex=M}
{name=李康,age=22,country=中国,sex=M}
{name=小梅,age=20,country=中国,sex=F}
{name=何雪,age=21,country=中国,sex=F}
{name=李康,age=22,country=中国,sex=M}
-------------------------------------------[{name=欧阳雪,age=18,country=中国,sex=F}, {name=Tom,age=24,country=美国,sex=M}, {name=Harley,age=22,country=英国,sex=F}, {name=向天笑,age=20,country=中国,sex=M}, {name=李康,age=22,country=中国,sex=M}, {name=小梅,age=20,country=中国,sex=F}, {name=何雪,age=21,country=中国,sex=F}, {name=李康,age=22,country=中国,sex=M}]
中国人有：6个
```
## 3.2 Stream的map方法
Stream接口的map()方法用于对流中的元素进行转换，它接收流中的每一个元素，返回转换后的值，通过它转换后，源流中的所有元素都将被映射为它转换后的值了。   
（1）map()方法接收一个Function对象，使用Lambda表达式实现Function接口的抽象方法,所谓map，从字面理解就是映射,这里指的是对象关系的映射,将上游Stream的每一个元素按照逻辑进行元素转换。   
（2）该方法返回一个Stream对象。Function对象指定上游数据中每一个元素的转换规则，返回的Stream是转换后的数据流。    
（3）查看map()方法的实现：map()默认处理的是上流Stream中的每一个元素对象，将每一个元素按照实现的Function接口的抽象方法实现规则（使用Lambda）进行转换后返回到一个新的Stream中。   
（4）总结：map()就是按照指定的实现逻辑对传入的每一个元素对象进行转换后返回到新的Stream中。      
```java
@Test
public void testStream_map() {
    personList.stream().map((element) -> {
        //将每一个元素的age加上20后返回每一个元素到Stream中，最后使用Stream的foreach()方法进行遍历。
        element.setAge(element.getAge() + 20);
        return element;
    }).forEach(System.out::println);
}
```
运行：   
```youtrack
{name=欧阳雪,age=38,country=中国,sex=F}
{name=Tom,age=44,country=美国,sex=M}
{name=Harley,age=42,country=英国,sex=F}
{name=向天笑,age=40,country=中国,sex=M}
{name=李康,age=42,country=中国,sex=M}
{name=小梅,age=40,country=中国,sex=F}
{name=何雪,age=41,country=中国,sex=F}
{name=李康,age=42,country=中国,sex=M}
```
## 3.3 Stream的reduce方法
reduce()方法用于对上游Stream中每一个元素进行聚合运算。可以求和、求平均数、求个数等。   
（1）SQL中类似 sum()、avg() 或者 count() 的聚集函数，实际上就是 reduce 操作，因为它们接收多个值并返回一个值。   
（2）它接收一个 BinaryOperator 对象，这个对象也是一个函数式接口，它继承BiFunction接口，因此，它接收两个参数，返回一个值。   
（3）返回值是Optional，使用get()方法获取对应的统计结果。   
（4）注意：上游Stream传入的数据中，每一个元素的数据类型必须要符合要求，如果不符合，应该先使用map()进行转换后再对整个Steam进行统计。
```java
    @Test
    public void testStream_reduce() {
        //因为上游Stream传入的一堆Person，而reduce()方法接受的类型默认就是传入的类型，而默认传入的类型就是上游stream中的每一个元素类型，即Person。
        //很明显Person没法进行聚合，所以需要先使用map()将一堆Person类型先进行转换。
//        Long total = personList.stream().reduce((sum, element) -> {
//            return sum.getAge() + element.getAge();
//        }).get();
        Integer totalAge = personList.stream().map((element) -> element.getAge()).reduce((sum, element) -> sum + element).get();
        System.out.println("统计所有人的年龄之和：" + totalAge);
    }
```
运行：   
```youtrack
统计所有人的年龄之和：169
```
## 3.4 Stream的collect方法
collect()方法接受一个Collector收集器对象，将上游stream数据流转换成自己Collectors收集器指定的数据。   
目标就是将指定的上游stream流数据聚合成对应的集合对象。   

```java
    @Test
    public void testStream_collect() {
        List<String> resultList = personList.stream().map((element) -> element.getName()).collect(Collectors.toList());
        System.out.println("resultList:" + resultList);
    }
```
运行：   
```youtrack
resultList:[欧阳雪, Tom, Harley, 向天笑, 李康, 小梅, 何雪, 李康]
```
## 3.5 filter结合collect
使用filter()方法指定回调过滤接口规则；使用collect()将数据按照指定收集器规则进行收集。   
再强调一遍：filter()方法会产生一个符合过滤规则的新列表，而不会更改源列表。   
```java
    @Test
    public void testStream_filter_collect() {
        List<ModeDemo06> resultList = personList.stream().filter(e -> e.getAge() > 18).collect(Collectors.toList());
        System.out.println("resultList:" + resultList);
    }
```
运行：   
```youtrack
resultList:[{name=Tom,age=24,country=美国,sex=M}, {name=Harley,age=22,country=英国,sex=F}, {name=向天笑,age=20,country=中国,sex=M}, {name=李康,age=22,country=中国,sex=M}, {name=小梅,age=20,country=中国,sex=F}, {name=何雪,age=21,country=中国,sex=F}, {name=李康,age=22,country=中国,sex=M}]
```
## 3.6 Collectors的joining方法
（1）对列表中每一个元素使用函数等，Collectors.joining():使用上游Stream中的各个元素拼接目标字符串使之成为一个新的字符串。   
（2）最后一个元素将不会在末尾拼接。   
```java
    @Test
    public void testStream_function() {
        String resultStr = personList.stream().map(x -> x.getName().toUpperCase()).collect(Collectors.joining("---"));
        System.out.println("resultStr:" + resultStr);
    }
```
运行：   
```youtrack
resultStr:欧阳雪---TOM---HARLEY---向天笑---李康---小梅---何雪---李康
```
### 进一步理解joining方法：
在 Java 8 中，你可以使用 `Stream` API 和 `Collectors.joining()` 来将 `List` 中的元素拼接成一个字符串，并且可以控制分隔符（如逗号 `,`），而且最后一个元素不需要分隔符。

以下是实现示例：

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("apple", "banana", "orange", "grape");

        // 使用 Collectors.joining(",") 进行拼接
        String result = list.stream()
                            .collect(Collectors.joining(","));
        
        System.out.println(result);
    }
}
```

### 代码说明：
1. **`list.stream()`**：将 `List` 转换为流。
2. **`Collectors.joining(",")`**：将流中的元素用 `,` 拼接起来，默认最后一个元素不会加上额外的 `,`。

### 输出结果：
```
apple,banana,orange,grape
```

### 其他可选拼接方式：
- 你还可以自定义分隔符、前缀和后缀：
```java
String result = list.stream()
                    .collect(Collectors.joining(",", "[", "]"));
System.out.println(result);
```

输出：
```
[apple,banana,orange,grape]
```

这种方式可以优雅地将列表中的元素拼接起来，同时确保最后一个元素不会多余加上分隔符。

## 3.7 Stream的distinct方法
对目标Stream进行结果去重复。   
（1）distinct方法返回一个新的Stream，不会影响源Stream。   
（2）distinct()对Stream数据流进行去重复，默认是根据元素对应的引用地址去判断是否是重复元素的(和hashSet一样，如果要真正去重复必须实现equals()方法和hashCode()方法)。   
（3）注意，即便元素为null值，它也同样参与去重复比较。
```java
    @Test
    public void testStream_distinct() {
        //对stream数据流进行去重复，stream中保存的是一堆对象
        List<ModeDemo06> persons = personList.stream().distinct().collect(Collectors.toList());
        System.out.println("去重复后的：" + persons);

        //转换成每一个年龄的集合，不去除重复
        List<Integer> resultList = personList.stream().map(e -> e.getAge()).collect(Collectors.toList());
        System.out.println("resultList:" + resultList);

        //转换成每一个年龄的集合，去除重复
        List<Integer> resultList1 = personList.stream().map(e -> e.getAge()).distinct().collect(Collectors.toList());
        System.out.println("resultList1:" + resultList1);

        Long nullCount = personList.stream().map(e -> e.getAge()).filter(e -> e == null).count();
        System.out.println("nullCount:" + nullCount);

        Long count = personList.stream().map(e -> e.getAge()).distinct().count();
        System.out.println("去重复后的age属性的count：" + count);
        System.out.println("去重复前的age属性的count：" + persons.size());
    }
```
运行：   
```youtrack
去重复后的：[{name=欧阳雪,age=18,country=中国,sex=F}, {name=Tom,age=24,country=美国,sex=M}, {name=Harley,age=22,country=英国,sex=F}, {name=向天笑,age=20,country=中国,sex=M}, {name=李康,age=22,country=中国,sex=M}, {name=小梅,age=20,country=中国,sex=F}, {name=何雪,age=21,country=中国,sex=F}, {name=李康,age=22,country=中国,sex=M}]
resultList:[18, 24, 22, 20, 22, 20, 21, 22]
resultList1:[18, 24, 22, 20, 21]
nullCount:0
去重复后的age属性的count：5
去重复前的age属性的count：8
```
## 3.8 Stream的mapToInt方法
mapToInt()：直接将上游元素每一个转换成int类型的。它的返回值是IntStream。       
IntStream、LongStream 和 DoubleStream 等流的类中，有个非常有用的方法叫做 summaryStatistics() 。   
可以返回 IntSummaryStatistics、LongSummaryStatistics 或者 DoubleSummaryStatistic s，描述流中元素的各种摘要数据。   
在本例中，我们用这个方法来计算列表的最大值和最小值。它也有 getSum() 和 getAverage() 方法来获得列表的所有元素的总和及平均值。
```java
    @Test
    public void testStream_IntStream() {
        IntSummaryStatistics summaryStatistics = personList.stream().map(e -> e.getAge()).mapToInt(x -> x).summaryStatistics();
        System.out.println("最大值：" + summaryStatistics.getMax());
        System.out.println("最小值：" + summaryStatistics.getMin());
        System.out.println("求个数：" + summaryStatistics.getCount());
        System.out.println("求和：" + summaryStatistics.getSum());
        System.out.println("求平均值：" + summaryStatistics.getAverage());
    }
```
运行：   
```youtrack
最大值：24
最小值：18
求个数：8
求和：169
求平均值：21.125
```

### mapToInt方法实际操作：Map中有多个key-value，其中每一个value是一个List,现在想通过java8立即得到整个map中所有value的size，注意是将所有value聚合后的list的总size。该怎么操作呢？   
在 Java 8 中，可以使用 `Stream` API 来实现这个需求。你可以将 `Map` 中所有 `List` 类型的 `value` 聚合起来，计算出这些 `List` 的元素总数。

具体步骤如下：
1. 获取 `Map` 中所有 `value`，即 `List` 的集合。
2. 使用 `Stream` 展开每个 `List` 并计算其大小。
3. 最终将这些 `List` 的大小累加，得到总数。

以下是实现代码示例：

```java
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        // 创建一个包含 List 作为 value 的 Map
        Map<String, List<Integer>> map = new HashMap<>();
        map.put("list1", Arrays.asList(1, 2, 3));
        map.put("list2", Arrays.asList(4, 5));
        map.put("list3", Arrays.asList(6, 7, 8, 9));

        // 使用 Stream API 计算所有 List 的总大小
        int totalSize = map.values().stream()
                .mapToInt(List::size) // 计算每个 List 的大小
                .sum(); // 汇总所有大小

        System.out.println("Total size of all lists: " + totalSize);
    }
}
```

### 代码说明：
1. `map.values().stream()`：获取 `Map` 中所有 `value`（即 `List`）并将其转换为流。
2. `mapToInt(List::size)`：将每个 `List` 的大小映射为整数。
3. `sum()`：将所有 `List` 的大小累加，得到总的元素数量。

### 输出结果：
```
Total size of all lists: 9
```

### 优点：
- 代码简洁，利用流式 API 实现复杂操作。
- 可以轻松处理 `Map` 中多个 `List` 的聚合需求。

这种写法不仅简洁，还能很好地扩展到更多的聚合操作。


## 3.9 Stream的limit方法
limit()：Stream的方法，对上游数据流进行截取。   
同样，它返回一个新的Stream。   
```java
    // 指定过滤规则后，选取前面2个。
    @Test
    public void testStream_limit() {
        personList.stream().filter(e -> e.getAge() > 18).limit(2).forEach(e -> System.out.println(e));
    }
```
运行：   
```youtrack
{name=Tom,age=24,country=美国,sex=M}
{name=Harley,age=22,country=英国,sex=F}
```
## 3.10 Stream的of方法
of()：Stream的静态方法，创建一个指定数据集合的Stream流，该方法通常用于我们快速的创建一个Stream对象。   
它接收一个T类型的可变参数，实际上本质就是一个T类型的数组。   
因此，我们可以传递任意类型的一个数组对象进去，将它转换为Stream。      
```java
    @Test
    public void testStream_of() {
        Stream<String> stringStream = Stream.of("Eric", "Daisy", "Poppy", "Sam");
        stringStream.forEach(e -> System.out.println(e));
    }
```
运行：    
```youtrack
Eric
Daisy
Poppy
Sam
```
## 3.11 Stream的max和min方法
max():从上游Stream中获取到指定比较元素对应的最大值的元素，注意返回的是对应规则的最大元素，即返回的是Person。   
min():从上游Stream中获取到指定比较元素对一个的最小值的元素。   
它接收一个Comparator比较器对象，该对象也被改造为函数式接口了，其中只有一个抽象方法 compare()，用于指定两个对象的比较规则。   
max和min返回一个Optional。   
```java
    @Test
    public void testStream_max_min() {
        ModeDemo06 maxPerson = personList.stream().max(Comparator.comparing(a -> a.getAge())).get();
        System.out.println("年龄最大的人是：" + maxPerson);
        ModeDemo06 minPerson = personList.stream().min(Comparator.comparing(a -> a.getAge())).get();
        System.out.println("年龄最小的人是：" + minPerson);
    }
```
运行：   
```youtrack
年龄最大的人是：{name=Tom,age=24,country=美国,sex=M}
年龄最小的人是：{name=欧阳雪,age=18,country=中国,sex=F}
```
## 3.12 Stream的allMatch
allMatch()：上游Stream各个元素是否全部匹配指定的规则。      
（1）接收一个Predicate，表示一个条件表达式。      
（2）返回值是boolean，判断所有的元素是否都符合该条件表达式，如果符合则为true，如果有一个不符合，则为false。     
```java
    @Test
    public void testStream_allMatch() {
        Boolean result = personList.stream().allMatch(e -> e.getName().startsWith("T"));
        System.out.println("是否匹配：" + result);
    }
```
运行：   
```youtrack
是否匹配：false
```
## 3.13 Stream的anyMatch
和allMatch刚好相反，只要上游Stream中有一个元素符合条件规则，则返回true，否则返回false。   
anyMatch()：上游Stream各个元素是否任意匹配指定的规则。
```java
    @Test
    public void testStream_anyMatch() {
        Boolean result = personList.stream().anyMatch(e -> e.getName().startsWith("T"));
        System.out.println("是否匹配：" + result);
    }
```
运行：   
```youtrack
是否匹配：true
```

## 3.14 IntStream 代替传统的for循环
在 Java 8 中，`IntStream` 提供了一种优雅的方式来替代传统的 `for` 循环。它可以通过流式 API 执行固定次数的操作。使用 `IntStream.range()` 或 `IntStream.rangeClosed()`，我们可以轻松地执行指定次数的循环操作，并结合 `Stream` API 来编写更加简洁的代码。

以下是一个使用 `IntStream` 替代传统 `for` 循环的示例：

### 传统的 `for` 循环：
```java
for (int i = 0; i < 5; i++) {
    System.out.println("Iteration: " + i);
}
```

### 使用 `IntStream` 替代：
```java
import java.util.stream.IntStream;

public class Main {
    public static void main(String[] args) {
        IntStream.range(0, 5).forEach(i -> System.out.println("Iteration: " + i));
    }
}
```

### 代码说明：
1. **`IntStream.range(0, 5)`**：生成一个从 0（含）到 5（不含）的整型流，相当于 `for (int i = 0; i < 5; i++)`。
2. **`forEach()`**：对每个流中的元素执行指定的操作，类似于 `for` 循环体内的代码。

### `range` 与 `rangeClosed` 的区别：
- **`IntStream.range(start, end)`**：生成的流范围是 `[start, end)`，即不包含 `end`。
- **`IntStream.rangeClosed(start, end)`**：生成的流范围是 `[start, end]`，即包含 `end`。

### 复杂操作示例：
除了简单的迭代输出外，还可以进行更复杂的操作，比如累加求和或多线程并行执行：

#### 累加求和：
```java
int sum = IntStream.range(1, 11).sum(); // 计算1到10的和
System.out.println("Sum: " + sum); // 输出: Sum: 55
```

#### 并行执行任务：
```java
IntStream.range(0, 5).parallel().forEach(i -> {
    System.out.println("Parallel Task: " + i + " - " + Thread.currentThread().getName());
});
```
这里使用 `parallel()` 可以将循环任务并行化，充分利用多核 CPU。

### 优点：
1. **简洁优雅**：相比传统的 `for` 循环，流式 API 更加简洁可读。
2. **并行化**：通过 `parallel()` 轻松实现多线程并行执行，提高性能。
3. **函数式编程风格**：使用 `forEach()` 等函数式接口，使代码更具表达性。

这类写法不仅使代码简洁清晰，还能帮助开发者更好地利用 Java 8 的函数式编程能力。


## 3.15 综合案例
```java
    @Test
    public void testStream_zonghe() {
        List<ModeDemo06> allList = personList.stream().filter(e -> e.getName().startsWith("T")).sorted(Comparator.comparing(s -> s.getName())).collect(Collectors.toList());
        System.out.println("allList:" + allList);
    }
```
运行：   
```youtrack
allList:[{name=Tom,age=24,country=美国,sex=M}]
```
# 4. Stream的collect()方法和Collectors收集器
collect是一个将管道流的结果集到一个list中的结束操作。   
collect是一个将数据流缩减为一个值的归约操作。这个值可以是集合、映射，或者一个值对象。   
你可以使用collect达到以下目的：   
(1)、将数据流缩减为一个单一值：一个流执行后的结果能够被缩减为一个单一的值。单一的值可以是一个Collection，或者像int、double等的数值，再或者是一个用户自定义的值对象。   
(2)、将一个数据流中的元素进行分组：根据任务类型将流中所有的任务进行分组。这将产生一个Map<TaskType, List>的结果，其中每个实体包含一个任务类型以及与它相关的任务。   
你也可以使用除了列表以外的任何其他的集合。   
如果你不需要与一任务类型相关的所有的任务，你可以选择产生一个Map<TaskType, Task>。这是一个能够根据任务类型对任务进行分类并获取每类任务中第一个任务的例子。   
(3)、分割一个流中的元素：你可以将一个流分割为两组——比如将任务分割为要做和已经做完的任务。   

下面看案例：   
先搞一个实体作为流中的元素：   
```java
package zeh.myjavase.code42java8.demo09;

import java.math.BigDecimal;

// 测试类 Apple
class AppleDemo09 {
    private Integer id;
    private String name;
    private BigDecimal money;
    private Integer num;

    public AppleDemo09(Integer id, String name, BigDecimal money, Integer num) {
        this.id = id;
        this.name = name;
        this.money = money;
        this.num = num;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public BigDecimal getMoney() {
        return money;
    }

    public void setMoney(BigDecimal money) {
        this.money = money;
    }

    public Integer getNum() {
        return num;
    }

    public void setNum(Integer num) {
        this.num = num;
    }

    public String toString() {
        return "[id = " + this.getId() + ",name = " + this.getName() + ",money = " + this.getMoney() + ",num = " + this.getNum() + "]";
    }
}
```
测试Stream:   
```java
package zeh.myjavase.code42java8.demo09;

import org.junit.Before;
import org.junit.Test;

import java.math.BigDecimal;
import java.util.*;
import java.util.function.Function;
import java.util.stream.Collectors;

public class CollectorsRun {
    List<AppleDemo09> appleList = new ArrayList<>();//存放apple对象集合

    @Before
    public void beforeAppleList() {
        AppleDemo09 apple1 = new AppleDemo09(1, "苹果1", new BigDecimal("3.25"), 10);
        AppleDemo09 apple12 = new AppleDemo09(1, "苹果2", new BigDecimal("1.35"), 20);
        AppleDemo09 apple2 = new AppleDemo09(2, "香蕉", new BigDecimal("2.89"), 30);
        AppleDemo09 apple3 = new AppleDemo09(3, "荔枝", new BigDecimal("9.99"), 40);
        appleList.add(apple1);
        appleList.add(apple12);
        appleList.add(apple2);
        appleList.add(apple3);
    }
}
```

## 4.1 Collectors.toList() 
```java

    // 将数据收集进一个list列表。
    // toList收集器使用了ArrayList作为列表的实现。
    @Test
    public void testCollectors_list() {
        List<String> nameList = appleList.stream().map(AppleDemo09::getName).collect(Collectors.toList());
        nameList.forEach(System.out::println);
    }

```

## 4.2 Collectors.toSet()
```java
    // 将数据收集进一个set集合。
    // toSet方法使用了HashSet作为集合的实现来存储结果集。
    // 注意：Set集合不能重复，并且hashSet集合是散列表顺序，如果需要确保收集的东西是唯一且不在乎顺序，可以使用toSet收集器。
    @Test
    public void testCollectors_set() {
        Set<AppleDemo09> appleSet = appleList.stream().collect(Collectors.toSet());
        appleSet.forEach(System.out::println);
    }
```
## 4.3 Collectors.toMap
### 4.3.1 最简单的toMap
```java

    // 将Stream数据流收集进一个map映射，即将list集合转换成map.
    // toMap()：有两个参数和三个参数的。
    // 第一个参数：目标map的key；第二个参数：目标map的value；第三个参数：目标map当key重复时，取值哪个key。
    // 需要注意的是：
    // toMap 如果集合对象有重复的key，会报错Duplicate key ....
    // apple1,apple12的id都为1。
    // 可以用 (k1,k2)->k1 来设置，如果有重复的key,则保留key1,舍弃key2
    // @Test
    public void testCollectors_toMap1() {
        //按照name作為key收集到一个map映射中
        Map<String, AppleDemo09> appleMap = appleList.stream().collect(Collectors.toMap(AppleDemo09::getName, e -> e));
        System.out.println("appleMap:" + appleMap);
    }
```
### 4.3.2 使用 Function.identity() 改进
```java
    // 通过使用Function接口中的默认方法identity来改进上面展示的代码，如下所示，这样可以让代码更加简洁，并更好地传达开发者的意图。
    @Test
    public void testCollectors_toMap2() {
        //按照name作為key收集到一个map映射中
        Map<String, AppleDemo09> appleMap = appleList.stream().collect(Collectors.toMap(AppleDemo09::getName, Function.identity()));
        System.out.println("appleMap:" + appleMap);
    }
```
### 4.3.3 list转Map如果遇到重复key怎么办？
```java
    // 如果收集的map中指定的key存在重复，将报错。
    // 通过使用toMap方法的另一个变体来处理重复问题，它允许我们指定一个合并方法。
    // 这个合并方法允许用户他们指定想如何处理多个值关联到同一个键的冲突。
    // 在下面展示的代码中，我们只是使用了新的值，当然你也可以编写一个智能的算法来处理冲突。
    @Test
    public void testCollectors_toMap3() {
        //按照id作为key收集到一个map映射中。id存在重复，所以toMap()方法中指定合并方法确定选择的key。
        Map<Integer, AppleDemo09> appleMap = appleList.stream().collect(Collectors.toMap(AppleDemo09::getId, a -> a, (key1, key2) -> key2));
        System.out.println("appleMap:" + appleMap);
    }
```
### 4.3.4 转换为map时可以指定单独的map存储容器
```java
    // toMap()将Stream数据流收集到map中时，还可以指定特定的map实例进行存储。
    // 比如指定LinkedHashMap。
    @Test
    public void testCollectors_toMap4() {
        Map<Integer, AppleDemo09> appleMap = appleList.stream().collect(Collectors.toMap(AppleDemo09::getId, Function.identity(), (k1, k2) -> k2, LinkedHashMap::new));
        System.out.println("appleMap:" + appleMap);
    }
```
### 4.3.5 toConcurrentMap
```java
    // 类似于toMap收集器，也有toConcurrentMap收集器，它产生一个ConcurrentMap而不是HashMap。
    @Test
    public void testCollectors_toMap5() {
        Map<Integer, AppleDemo09> appleMap = appleList.stream().collect(Collectors.toConcurrentMap(AppleDemo09::getId, Function.identity(), (k1, k2) -> k2));
        System.out.println("appleMap:" + appleMap);
    }
```

## 4.4 Collectors.toCollection
```java
    // 将stream数据流收集进其他的收集容器中。
    // 像toList和toSet这类特定的收集器不允许你指定内部的列表或者集合实现。
    // 当你想要将结果收集到其它类型的集合中时，你可以像下面这样使用toCollection收集器。
    @Test
    public void testCollectors_toOthers() {
        //将数据流收集到LinkedHashSet中。
        Set<AppleDemo09> appleSet = appleList.stream().collect(Collectors.toCollection(LinkedHashSet::new));
        System.out.println("appleSet:" + appleSet);
    }
```

## 4.5 Collectors.groupingBy
```java
    // 对list集合按照某个属性进行分组，分组后是一个Map，其中map的key为进行分组的id，value为源list进行分组后的多个list。
    @Test
    public void testCollectors_groupBy() {
        Map<Integer, List<AppleDemo09>> groupByMap = appleList.stream().collect(Collectors.groupingBy(AppleDemo09::getId));
        System.out.println("groupByMap:" + groupByMap);
    }
```

## 4.6 Collectors.partitioningBy
```java
    // partitioningBy(分割) 和 groupingBy 的作用相似。
    // 不同的是他的key值为true/false。
    @Test
    public void testCollectors_partitioningBy() {
        Map<Boolean, List<AppleDemo09>> partioningByMap = appleList.stream().collect(Collectors.partitioningBy(e -> e.getId() > 1));
        System.out.println("partioningByMap:" + partioningByMap);
    }
```

## 4.7 Collectors.collectingAndThen方法详解
`Collectors.collectingAndThen` 是 Java 8 中 `Collectors` 类的一个静态方法，它用于对收集器的结果进行进一步处理或转换。具体来说，它可以在使用一个收集器收集数据后，执行一个额外的转换操作。

### 方法签名：
```java
public static <T, A, R, RR> Collector<T, A, RR> collectingAndThen(
    Collector<T, A, R> downstream, 
    Function<R, RR> finisher
)
```

### 参数说明：
1. **`downstream`**：一个收集器，用于收集数据，通常是已有的内置收集器，比如 `Collectors.toList()`、`Collectors.toSet()` 等。
2. **`finisher`**：一个 `Function`，用于对收集器生成的结果进行最终处理或转换。

### 返回值：
返回一个新的 `Collector`，该收集器先执行 `downstream` 收集数据，然后对结果应用 `finisher` 函数来生成最终的输出。

### 场景和用法：
`collectingAndThen` 常用于当你想对已有的 `Collector` 进行进一步的处理时。例如，将一个 `List` 通过 `Collectors.toList()` 收集起来后，再将其转换为不可修改的 `List`。

### 示例 1：将 `List` 转换为不可修改的 `List`

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("apple", "banana", "orange", "grape");

        // 使用 Collectors.collectingAndThen 收集 List 并转换为不可修改的 List
        List<String> unmodifiableList = list.stream()
                .collect(Collectors.collectingAndThen(
                        Collectors.toList(),          // 下游收集器：收集到 List
                        Collections::unmodifiableList // 转换函数：将 List 转换为不可修改的 List
                ));

        System.out.println(unmodifiableList);

        // 尝试修改不可修改的 List 会抛出异常
        // unmodifiableList.add("new fruit"); // 会抛出 UnsupportedOperationException
    }
}
```

### 代码解释：
1. **`Collectors.toList()`**：这是一个下游收集器，将流中的元素收集到一个 `List` 中。
2. **`Collections::unmodifiableList`**：这是一个转换函数（`finisher`），将 `List` 转换为不可修改的 `List`。
3. **`Collectors.collectingAndThen()`**：它将上述两个步骤组合在一起，先收集元素到 `List`，然后将 `List` 转换为不可修改的 `List`。

### 示例 2：收集元素并计算结果（例如统计 `List` 中元素的个数）

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("apple", "banana", "orange", "grape");

        // 使用 Collectors.collectingAndThen 收集 List 并计算 List 的大小
        int size = list.stream()
                .collect(Collectors.collectingAndThen(
                        Collectors.toList(),  // 下游收集器：收集到 List
                        List::size           // 转换函数：对 List 计算大小
                ));

        System.out.println("List size: " + size);
    }
}
```

### 代码解释：
1. **`Collectors.toList()`**：将流中的元素收集到 `List` 中。
2. **`List::size`**：将收集到的 `List` 进一步处理，获取它的大小。

### `collectingAndThen` 的使用场景：
- **转换收集后的结果**：在数据收集完成后，对结果进行一些不可变的操作或转换。例如将 `Set` 转换为不可修改的 `Set`，或者将收集到的 `List` 转换为某种特定类型的集合。
- **计算最终的统计数据**：你可以先收集数据，然后对收集到的结果进行计算或统计（如求总和、求大小等）。
- **不可变集合**：常用于确保收集后的集合是不可修改的。

### 小结：
`Collectors.collectingAndThen` 提供了一种灵活的方式，可以在使用收集器收集数据后立即对结果进行进一步的处理或转换。它结合了函数式编程的思想，将数据收集和后续处理分离，提高了代码的可读性和可维护性。

## 4.8 Collectors.counting()
```java
    // 计算Stream数据流中元素的个数。
    @Test
    public void testCollectors_count() {
        Long count = appleList.stream().collect(Collectors.counting());
        System.out.println("Stream数据流中元素的个数：" + count);
    }
```

## 4.9 reduce统计总数
```java
    // 统计stream数据流中的金额总数:本案例借助的是BigDecimal类中的初始值和自增统计方法add。
    @Test
    public void testCollectors_sum() {
        BigDecimal totalMoney = appleList.stream().map(AppleDemo09::getMoney).reduce(BigDecimal.ZERO, BigDecimal::add);
        System.out.println("总花费：" + totalMoney);
    }
```
## 4.10 Comparator.comparing
```java
    // 使用收集器获取数据流中的最大值对应的元素对象，返回结果是一个Optional对象。
    @Test
    public void testCollectors_max_min() {
        Optional<AppleDemo09> maxResult = appleList.stream().collect(Collectors.maxBy(Comparator.comparing(AppleDemo09::getId)));
        maxResult.ifPresent(System.out::println);
        Optional<AppleDemo09> minResult = appleList.stream().collect(Collectors.minBy(Comparator.comparing(AppleDemo09::getId)));
        minResult.ifPresent(System.out::println);
    }
```

# 5. 使用Stream求两个集合的交集差集并集
直接上案例：
```java
package zeh.myjavase.code42java8.demo10;

import org.junit.Test;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

// java8计算交集、并集、差集
public class JiaoBingChaJiRun {
    @Test
    public void testJiaoJi() {
        List<Integer> integerList1 = Arrays.asList(1, 2, 3);
        List<Integer> integerList2 = Arrays.asList(3, 4, 5, 6);
        System.out.println("求交集：");
        List<Integer> jiaojiList = integerList1.stream().filter(e -> integerList2.contains(e)).collect(Collectors.toList());
        jiaojiList.forEach(System.out::println);
    }

    @Test
    public void testChaJi() {
        List<Integer> integerList1 = Arrays.asList(1, 2, 3);
        List<Integer> integerList2 = Arrays.asList(1, 2, 3, 4, 5, 6);
        System.out.println("求差集（list1-list2）:");
        List<Integer> chajiList = integerList1.stream().filter(e -> !integerList2.contains(e)).collect(Collectors.toList());
        System.out.println("chajiList:" + chajiList.size());
        chajiList.forEach(System.out::println);
    }

    @Test
    public void testChaJi2() {
        List<Integer> integerList1 = Arrays.asList(1, 2, 3);
        List<Integer> integerList2 = Arrays.asList(3, 4, 5, 6);
        System.out.println("求差集（list2-list1）:");
        List<Integer> chajiList = integerList2.stream().filter(e -> !integerList1.contains(e)).collect(Collectors.toList());
        chajiList.forEach(System.out::println);
    }

    @Test
    public void testBingJi() {
        List<Integer> list = new ArrayList<>();
        List<Integer> integerList1 = Arrays.asList(1, 2, 3);
        List<Integer> integerList2 = Arrays.asList(3, 4, 5, 6);
        list.addAll(integerList1);
        list.addAll(integerList2);
        System.out.println("求并集（不去重）：");
        list.forEach(System.out::println);
    }


    @Test
    public void testBingJi2() {
        List<Integer> list = new ArrayList<>();
        List<Integer> integerList1 = Arrays.asList(1, 2, 3);
        List<Integer> integerList2 = Arrays.asList(3, 4, 5, 6);
        list.addAll(integerList1);
        list.addAll(integerList2);
        System.out.println("求并集（去重）：");
        list.stream().distinct().collect(Collectors.toList()).forEach(System.out::println);
    }
}

```

# 6.Stream的skip和limit进行集合的行列去除
Java 8 Stream中的两个方法：skip()和limit()。这两个方法是Stream很常用的，不仅各自会被高频使用，还可以组合出现，并能实现一些小功能，如subList和分页等。   

## 6.1 skip()方法  
见名知义，skip()方法用于跳过前面n个元素，然后再返回新的流，如图所示：   
![][stream_skip]
来看看代码：
```java
    List<Integer> result = Stream.of(1, 2, 3, 4, 5, 6).skip(4).collect(Collectors.toList());
    List<Integer> expected = asList(5, 6);
    assertEquals(expected, result);
```
方法skip()的参数n的四种情况：   
（1）当n<0时，抛IllegalArgumentException异常；   
（2）当n=0时，相当没有跳过任何元素，原封不动、完璧归赵；   
（3）当0<n<length时，跳过n个元素后，返回含有剩下的元素的流；   
（4）当n>=length时，跳过所有元素，返回空流。   

## 6.2 limit()方法
对于limit()方法，它是用于限制流中元素的个数，即取前n个元素，返回新的流，如图所示：   
![][stream_limit]
代码如下：
```java
    List<Integer> result = Stream.of(1, 2, 3, 4, 5, 6).limit(4).collect(Collectors.toList());
    List<Integer> expected = asList(1, 2, 3, 4);
    assertEquals(expected, result);
```

方法limit()的参数n的四种情况：   
（1）当n<0时，抛IllegalArgumentException异常；   
（2）当n=0时，不取元素，返回空流；      
（3）当0<n<length时，取前n个元素，返回新的流；   
（4）当n>=length时，取所有元素，原封不动、完璧归赵。   

## 6.3 对无限流的操作
流Stream分为有限流和无限流，前面的例子我们都是使用的有限流，与Java集合类不同，流是可以无限的。对于无限流，skip()和limit()表现出了极大的差异，先上代码：   
```java
Stream.iterate(1, i -> i + 1).filter(num -> (num & (num - 1)) == 0).limit(10).forEach(System.out::println);
System.out.println("----------------");
Stream.iterate(1, i -> i + 1).filter(num -> (num & (num - 1)) == 0).skip(10).forEach(System.out::println);
```
执行后发现，limit()是可以将无限流转化为有限流的，所以我们也可以认为它是一个短路操作。而skip()则不行，不管你跳过了前面多少个元素，总还是会有源源不断的元素过来，无法收敛。   
上述代码的结果是：   
通过limit()输出了前十个2的n次方值：   
1, 2, 4, 8, 16, 32, 64, 128, 256, 512   
而skip()跳过了前10个，继续输出，但会不断执行下去（会有int的溢出现象）：   
1024, 2048, 4096, 8192, 16384, 32768...   

## 6.4 组合应用
除了两者各自有各自的功能外，我们通过组合使用，可以实现其它功能。   
### 6.4.1 与subList的替换
集合类如List是有subList()这个方法的，可以截取List中的某一部分，这个功能还可以通过组合skip()和limit()使用得到，如下面代码：   
```java
    List<Integer> list = asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
    List<Integer> expected = list.subList(3, 7);
    
    List<Integer> result = list.stream()
      .skip(3)
      .limit(7 - 3)
      .collect(Collectors.toList());
    assertEquals(expected, result);
```
### 6.4.2 分页 
以通过组合使用skip()和limit()进行分页，如下面代码：
```java
    int pageSize = 10;
    int pageIndex = 7;
    
    List<Integer> expected = asList(61, 62, 63, 64, 65, 66, 67, 68, 69, 70);
    List<Integer> result = Stream.iterate(1, i -> i + 1)
      .skip((pageIndex - 1) * pageSize)
      .limit(pageSize)
      .collect(Collectors.toList());
    
    assertEquals(expected, result);
```
上面代码例子是获取了第七页数据，每页大小为10。   

## 6.5  总结
介绍了Java 8的Stream接口中两个常用的方法：skip()和limit()，比较简单易懂，也介绍了怎么组合使用。需要注意的是，如果Stream过大或是无限流，小心skip()会有性能问题。   

# 7. 综合案例
实体类：
```java
package zeh.myjavase.code42java8.demo11;

// 论文详情表
public class RequirePaperDetailVO {

    // 论文详情ID（主键）
    private Integer paperDetailId;
    
    // 论文详情中文标题
    private String detailTitleZH;
    
    // 论文详情中文内容
    private String detailContentZH;
    
    // 论文详情英文标题
    private String detailTitleEN;
    
    // 论文详情英文内容
    private String detailContentEN;
    
    // 论文选题信息ID
    private Integer paperId;
    
    // 章节ID
    private Integer chapterId;
    
    // 章节标题类别
    private String chapterType;

	public Integer getPaperDetailId() {
		return paperDetailId;
	}

	public void setPaperDetailId(Integer paperDetailId) {
		this.paperDetailId = paperDetailId;
	}

	public String getDetailTitleZH() {
		return detailTitleZH;
	}

	public void setDetailTitleZH(String detailTitleZH) {
		this.detailTitleZH = detailTitleZH;
	}

	public String getDetailContentZH() {
		return detailContentZH;
	}

	public void setDetailContentZH(String detailContentZH) {
		this.detailContentZH = detailContentZH;
	}

	public String getDetailTitleEN() {
		return detailTitleEN;
	}

	public void setDetailTitleEN(String detailTitleEN) {
		this.detailTitleEN = detailTitleEN;
	}

	public String getDetailContentEN() {
		return detailContentEN;
	}

	public void setDetailContentEN(String detailContentEN) {
		this.detailContentEN = detailContentEN;
	}

	public Integer getPaperId() {
		return paperId;
	}

	public void setPaperId(Integer paperId) {
		this.paperId = paperId;
	}

	public String getChapterType() {
		return chapterType;
	}

	public void setChapterType(String chapterType) {
		this.chapterType = chapterType;
	}

	public Integer getChapterId() {
		return chapterId;
	}

	public void setChapterId(Integer chapterId) {
		this.chapterId = chapterId;
	}
	
}
```
测试Stream:
```java
package zeh.myjavase.code42java8.demo11;

import com.alibaba.fastjson.JSON;
import org.apache.commons.collections.CollectionUtils;
import org.junit.Test;

import java.util.*;
import java.util.stream.Collectors;

public class Java8Run {

    @Test
    public void test1() {
        List<Map<String, Object>> chapterMapList = new ArrayList<>();
        List<RequirePaperDetailVO> requirePaperDetailVOList2 = new ArrayList<>();
        Map map1 = new HashMap();
        Map map2 = new HashMap();
        Map map3 = new HashMap();
        map1.put("key1", "value1");
        map2.put("key1", "value1");
        map3.put("chapterId", 1234);

        chapterMapList.add(map1);
        chapterMapList.add(map2);
        chapterMapList.add(map3);

        RequirePaperDetailVO vo1 = new RequirePaperDetailVO();
        vo1.setChapterId(1234);
        vo1.setChapterType("zhao");
        vo1.setDetailContentEN("中文");
        vo1.setDetailContentEN("yingwen");
        requirePaperDetailVOList2.add(vo1);

        RequirePaperDetailVO vo2 = new RequirePaperDetailVO();
        vo2.setChapterId(1234);
        vo2.setChapterType("zhao");
        vo2.setDetailContentEN("中文");
        vo2.setDetailContentEN("yingwen");
        requirePaperDetailVOList2.add(vo2);

        List<Map<String, Object>> chapterMapList1 = new ArrayList<>();
        if (CollectionUtils.isNotEmpty(chapterMapList) && CollectionUtils.isNotEmpty(requirePaperDetailVOList2)) {
            chapterMapList.stream().map(chapter -> requirePaperDetailVOList2.stream()
                    .filter(paperDetail -> Objects.equals(chapter.get("chapterId"), paperDetail.getChapterId()))
                    .findFirst().map(paperDetail -> {
                        chapter.put("paperDetailId", paperDetail.getPaperDetailId());
                        chapter.put("chapterNameZH", paperDetail.getDetailTitleZH());
                        chapter.put("chapterContent", paperDetail.getDetailContentZH());
                        chapter.put("chapterNameEN", paperDetail.getDetailTitleEN());
                        chapter.put("chapterContentEN", paperDetail.getDetailContentEN());
                        chapter.put("chapterType", "3");
//      chapterMapList1.add(chapter);
                        System.out.println(JSON.toJSONString(chapter));
                        chapterMapList1.add(chapter);
                        return chapter;
                    }).orElse(null)).filter(Objects::nonNull).collect(Collectors.toList());
            System.out.println(JSON.toJSONString(chapterMapList1));
        }

    }
}
```



 [stream_skip]: data:image/png;base64,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
 [stream_limit]: data:image/png;base64,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QAIkQAIkQAIkQAIkQAIkQAKmIUCJYJqo2SgJkAAJkAAJkAAJkAAJkAAJkAAJ9I0AJULf+PFuEiABEiABEiABEiABEiABEiABEjANAUoE00TNRkmABEiABEiABEiABEiABEiABEigbwQoEfrGj3eTAAmQAAmQAAmQAAmQAAmQAAmQgGkIUCKYJmo2SgIkQAIkQAIkQAIkQAIkQAIkQAJ9I0CJ0Dd+vJsESIAESIAESIAESIAESIAESIAETEOAEsE0UbNREiABEiABEiABEiABEiABEiABEugbAUqEvvHj3SRAAiRAAiRAAiRAAiRAAiRAAiRgGgKUCKaJmo2SAAmQAAmQAAmQAAmQAAmQAAmQQN8IUCL0jR/vJgESIAESIAESIAESIAESIAESIAHTEKBEME3UbJQESIAESIAESIAESIAESIAESIAE+kaAEqFv/Hg3CZAACZAACZAACZAACZAACZAACZiGACWCaaJmoyRAAiRAAiRAAiRAAiRAAiRAAiTQNwKUCH3jx7tJgARIgARIgARIgARIgARIgARIwDQEKBFMEzUbJQESIAESIAESIAESIAESIAESIIG+EaBE6Bs/3k0CJEACJEACJEACJEACJEACJEACpiFAiWCaqNkoCZAACZAACZAACZAACZAACZAACfSNACVC3/jxbhIgARIgARIgARIgARIgARIgARIwDQFKBNNEzUZJgARIgARIgARIgARIgARIgARIoG8EKBH6xo93kwAJkAAJkAAJkAAJkAAJkAAJkIBpCFAimCZqNkoCJEACJEACJEACJEACJEACJEACfSNAidA3frybBEiABEiABEiABEiABEiABEiABExDgBLBNFGzURIgARIgARIgARIgARIgARIgARLoGwFKhL7x490kQAIkQAIkQAIkQAIkQAIkQAIkYBoClAimiZqNkgAJkAAJkAAJkAAJkAAJkAAJkEDfCFAi9I0f7yYBEiABEiABEiABEiABEiABEiAB0xCgRDBN1GyUBEiABEiABEiABEiABEiABEiABPpGgBKhb/x4NwmQAAmQAAmcEAGb1QJPDyssFqC5tR3t7R0ndL9ZP2y1WpAYPQDRYT6wwIKq2hbsyqlFR4fr+Ikxk+P8EBnsDTFKxYEWZOTWoUP+/3iRAAmQAAmQgDkJUCKYM3d2TQIkQAIk4AYC0aE+GJEYgMFRfvKL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RFAzN1ZNAiRAAiRAAiRAAiRAAiRAAiRAApoToETQHDkHJAESIAESIAESIAESIAESIAESIAE1CVAiqJkbqyYBEiABEiABEiABEiABEiABEiABzQlQImiOnAOSAAmQAAmQAAmQAAmQAAmQAAmQgJoEKBHUzI1VkwAJkAAJkAAJkAAJkAAJkAAJkIDmBCgRNEfOAUmABEiABEiABEiABEiABEiABEhATQKUCGrmxqpJgARIgARIgARIgARIgARIgARIQHMClAiaI+eAJEACJEACJEACJEACJEACJEACJKAmAUoENXNj1SRAAiRAAiRAAiRAAiRAAiRAAiSgOQFKBM2Rc0ASIAESIAESIAESIAESIAESIAESUJMAJYKaubFqEiABEiABEiABEiABEiABEiABEtCcACWC5sg5IAmQAAmQAAmQAAmQAAmQAAmQAAmoSYASQc3cWDUJkAAJkAAJkAAJkAAJkAAJkAAJaE6AEkFz5ByQBEiABEiABEiABEiABEiABEiABNQkQImgZm6smgRIgARIgARIgARIgARIgARIgAQ0J0CJoDlyDkgCJEACJEACJEACJEACJEACJEACahKgRFAzN1ZNAiRAAiRAAiRAAiRAAiRAAiRAApoToETQHDkHJAESIAESIAESIAESIAESIAESIAE1CVAiqJkbqyYBEiABEiABEiABEiABEiABEiABzQlQImiOnAOSAAmQAAmQAAmQAAmQAAmQAAmQgJoEKBHUzI1VkwAJkAAJkAAJkAAJkAAJkAAJkIDmBCgRNEfOAUmABEiABEiABEiABEiABEiABEhATQKUCGrmxqpJgARIgARIgARIgARIgARIgARIQHMClAiaI+eAJEACJEACJEACJEACJEACJEACJKAmAUoENXNj1SRAAiRAAiRAAiRAAiRAAiRAAiSgOQFKBM2Rc0ASIAESIAESIAESIAESIAESIAESUJOAbiSCb2AUPL391aRosKotFitsnj6weXrD3tYCe2sTOtrtBuuS7eiVgNXmAZuHDyw2D/nu2duagY4OvZbLuhQnIP6+s3p4wcNrANrtLWhraeTfd4pnqlr5Ng9veHj7oaOjHS0N1aqVz3pJgARIgARMSKCuMk/+3HTvtadi0TnjnCZg6ehw7qf8sRf/3elBeaNrCFhtXhBSxzcwEk115WisKYW9tdE1g/GpJHAYAU/v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/8fnnq0s64DpCnU2GznRRGaLVCqUjI8IEAEiQATmGwEKIsyjGWMvz9F4Gv3jfkx5WOpvFE5PlFs1DsfSSGZySGWynIAZe6cRifjcy7FMLIRSJuY+1GxmJWpNatjMKm7116xXUtBgHtlAJbqazuS4lTq2Quw8Zncub4xT3Gc2xzIV0uk893HHXpqZzbHVNGZ37IPNYlTCbp5O12Up4szuaLWtEjM5/+7JAgUsgMW2OzjdETh90z7P42c+LzXt89I5zuexAIRQyOe2wDD7UkhFMGhknK877vPYFhm7UQURCXTOP2OoQI9Z4NQXTGCIbXfgfF8ETk8M/mAC0eR0llYqnUc2Oy0EKhYLTvg+puNi0ilOeuaq0cYyXdQyLrBABxEgAkSACBCB+UyAggjzbPbYSzV7seFW44ol7jf7ObEqclImAluOY6u73Mocl4lwbFWYxzuxOsyyE6je9DwzgjJ3l32cnZz1Mr0SV0KRZcCUcCz75YOVYGZPbEWOx60GM1s7lgXDn7Y7oYDZHOUdlHka5+3tPvB5JW6l+CMrwSf7vA9nwhyzPS4bhq0KC/hcRhaZ37w1h7J2nGVYMT+XOykDhnvessy/YxkIzAceS8U6kYlwPBuL2Rrn/45l/03bH/m+sk4i3YwIEAEiQATmhAAFEeYEKzVKBIgAESACRIAIEAEiQASIABEgAkRg4RGgIMLCm1MaEREgAkSACBABIkAEiAARIAJEgAgQgTkhQEGEOcFKjRIBIkAEiAARIAJEgAgQASJABIgAEVh4BCiIsPDmlEZEBIgAESACRIAIEAEiQASIABEgAkRgTghQEGFOsFKjRIAIEAEiQASIABEgAkSACBABIkAEFh4BCiIsvDmlEREBIkAEiAARIAJEgAgQASJABIgAEZgTAhREmBOs1CgRIAJEgAgQASJABIgAESACRIAIEIGFR4CCCAtvTmlERIAIEAEiQASIABEgAkSACBABIkAE5oQABRHmBCs1SgSIABEgAkSACBABIkAEiAARIAJEYOERoCDCwptTGhERIAJEgAgQASJABIgAESACRIAIEIE5IUBBhDnBSo0SASJABIgAESACRIAIEAEiQASIABFYeAQoiLDw5pRGRASIABEgAkSACBABIkAEiAARIAJEYE4IUBBhTrBSo0SACBABIkAEiAARIAJEgAgQASJABBYeAQoiLLw5pRERASJABIgAESACRIAIEAEiQASIABGYEwIURJgTrNQoESACRIAIEAEiQASIABEgAkSACBCBhUeAgggLb05pRESACBABIkAEiAARIAJEgAgQASJABOaEAAUR5gQrNUoEiAARIAJEgAgQASJABIgAESACRGDhEaAgwsKbUxoRESACRIAIEAEiQASIABEgAkSACBCBOSFAQYQ5wUqNEgEiQASIABEgAkSACBABIkAEiAARWHgEKIiw8OaURkQEiAARIAJEgAgQASJABIgAESACRGBOCFAQYU6wUqNEgAgQASJABIgAESACRIAIEAEiQAQWHgEKIiy8OaUREQEiQASIABEgAkSACBABIkAEiAARmBMC/x9pLWJeu4/riAAAAABJRU5ErkJggg==
