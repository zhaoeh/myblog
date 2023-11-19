---
layout:     post
title:      java8 集合操作
subtitle:   jdk团队内部对集合框架大量应用了java8的函数式接口
categories: [JAVA8]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. java8对jdk内部做了哪些扩展？
java8之前没有扩充函数式接口和接口中默认方法、静态方法等。   
java8扩展了function包，下面内置了大量的函数式接口，其中函数式接口都实现了很多默认方法和静态方法，供外部调用。   
java8不仅仅增量追加了function包下的函数式接口，对于jdk原有的一些接口，也追加了某些默认方法，即在接口中通过default关键字指定的实现好的方法。      
默认方法的出现使得可以不在破坏原始子类实现接口的体系下，为接口本身增加大量的默认实现方法，供外部调用。      
简单的例子就是Collection接口和Map接口中增加了很多默认方法，比如foreach()等。   
foreach()方法和Stream中的很多方法实际上都是用来接收一个回调对象的内置方法（一般的内置方法大多数都是用来执行回调对象的）。   

下面我们简单看一个案例，对list和map集合进行遍历（分别使用Java8之前的for循环和java8的lambda表达式）：
```java
package zeh.myjavase.code42java8.demo04;

import org.junit.Test;

import java.util.*;

// Demo04CollectionAndMapForeach
public class CollectionAndMapForeachRun {

    //java8之前,遍历list和map
    @Test
    public void testListForeach() {
        List<String> sourceList = Arrays.asList("Q", "V", "R");
        for (String ele : sourceList) {
            System.out.println("e:" + ele);
        }

        Map<String, String> map = new HashMap<>();
        map.put("name", "Eric");
        for (Map.Entry<String, String> entry : map.entrySet()) {
            System.out.println("key:" + entry.getKey() + ";value:" + entry.getValue());
        }
    }

    //java8之后,遍历list和map。注意foreach()方法是java8为原来的集合接口引入的默认方法。
    @Test
    public void testListForeachJava8() {
        List<String> sourceList = Arrays.asList("Qq", "Vv", "Rr");
        sourceList.forEach((e) -> System.out.println("e:" + e));
        System.out.println("======方法引用方式实现Lambda表达式========");
        sourceList.forEach(System.out::println);
        Map<String, String> map = new HashMap<>();
        map.put("name", "赵二虎");
        map.forEach((k, v) -> System.out.println("key:" + k + ";value:" + v));
    }
}

```
点击集合框架对象的foreach方法进去，发现其定义在Iterable接口中，是自java8之后新增的一个接口默认方法：
```java
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```

# 2. list接口中添加了默认的sort方法
java8的List接口中添加了默认的sort()方法（默认方法接口自己实现），同时也扩展了Comparator比较器接口。   
下面看一个案例：
```java
package zeh.myjavase.code42java8.demo07;

import org.junit.Test;

import java.util.*;

public class SortRun {

    // sort()方法是java8对List扩展的默认方法；
    // Comparator.naturalOrder()方法是一个自然排序算法的实现。
    @Test
    public void test_sort_comparator() {
        List<Integer> integerList = Arrays.asList(2, 7, 3, 9, 1, 7, 0, 4, 3);
        integerList.sort(Comparator.naturalOrder());
        System.out.println("排序后的 integerList:" + integerList);
    }

    // 对集合遍历：使用java8对Collection接口扩展的默认方法foreach()
    @Test
    public void test_foreach() {
        List<String> stringList = Arrays.asList("Eric", "YuRui", "Daisy", "Poppy", "Alisa", "SuSan");
        //asList转换后的list对象不支持 add 和 addAll，如果想使用，外部使用一个新的 list，再将asList的返回对象添加到新的list中，这样使用外部的List即可。
//        stringList.add("4fghjkl;");
        List<String> newList = new ArrayList<>();
        newList.addAll(stringList);
        newList.add("234sdh;");
        stringList.forEach(System.out::println);
        newList.forEach(System.out::println);
    }


    // 自己实现比较器Comparator，指定排序规则进行排序。
    @Test
    public void test_sorted() {
        List<String> stringList = Arrays.asList("Eric", "YuRui", "Daisy", "Poppy", "Alisa", "SuSan");
        stringList.sort((s1, s2) -> {
            return s1.compareTo(s2);
        });
        System.out.println("自定义排序规则后的 stringList:" + stringList);
    }
}
```
同样，我们点击sort方法可以看到，其是在List接口中新增加的default方法：
```java
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }
```

# 3. Collection接口中也增加了默认方法
在Collection接口中增加了如下默认方法：
```java
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
    
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
    

    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
```
其中的stream()方法是最关键的，它返回一个Stream对象，这个对象在Collection单值集合中有非常广泛的应用。   