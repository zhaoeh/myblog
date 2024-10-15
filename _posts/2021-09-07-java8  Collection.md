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

# 4. Map.of方法，进行封装时，value不能为null吗？
是的，`Map.of()` 方法在封装键值对时，**不允许 `value` 为 `null`**。这是 `Java 9` 引入的 `Map.of()` 方法的一个特点。它用于创建不可变的 `Map`，并且对键和值的要求较为严格：

- **`null` 键**和**`null` 值**都不允许。
- 如果你尝试在 `Map.of()` 中传入 `null` 值，会抛出 `NullPointerException`。

### 示例

```java
import java.util.Map;

public class MapOfExample {
    public static void main(String[] args) {
        // 这段代码会抛出 NullPointerException，因为 value 为 null
        Map<String, String> map = Map.of("key1", "value1", "key2", null);  // 会报错
    }
}
```

输出：

```
Exception in thread "main" java.lang.NullPointerException
	at java.base/java.util.Objects.requireNonNull(Objects.java:221)
	at java.base/java.util.ImmutableCollections$MapN.<init>(ImmutableCollections.java:930)
	at java.base/java.util.Map.of(Map.java:1085)
```

### 解决方案
如果你有 `null` 值需要放入 `Map`，可以使用以下几种替代方案：

#### 1. 使用 `HashMap` 或其他允许 `null` 值的 `Map` 实现

你可以使用 `HashMap` 来存储 `null` 值，因为 `HashMap` 允许 `null` 键和值。

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapExample {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        map.put("key1", "value1");
        map.put("key2", null);  // 允许null值

        System.out.println(map);
    }
}
```

输出：

```
{key1=value1, key2=null}
```

#### 2. 使用 `Map.ofNullable()`（伪代码：Java 未提供这样的API）

目前 Java 没有提供类似 `Map.ofNullable()` 这样的 API，但你可以手动编写工具类来处理 `null` 值。

#### 3. 使用 `Optional` 进行封装

如果你的逻辑能够允许使用 `Optional` 来处理 `null` 值，可以使用 `Optional` 对值进行封装。

```java
import java.util.Map;
import java.util.Optional;

public class MapOptionalExample {
    public static void main(String[] args) {
        Map<String, Optional<String>> map = Map.of(
            "key1", Optional.of("value1"),
            "key2", Optional.ofNullable(null)  // Optional可以包裹null
        );

        System.out.println(map);
    }
}
```

输出：

```
{key1=Optional[value1], key2=Optional.empty}
```

### 总结
- `Map.of()` 不能包含 `null` 键或 `null` 值。
- 使用 `HashMap` 或其他支持 `null` 的实现，如果必须存储 `null` 值。
- 或者考虑使用 `Optional` 来优雅地处理可能为空的值。

  # 5. 还有其他方式吗？
  除了使用 `HashMap` 或 `Optional` 之外，处理 `null` 值的方式还有几种可选方法。这里介绍几种处理方案，根据实际场景可以灵活选择。

### 1. 使用 `Stream` 和 `Collectors` 构建 Map

如果你有一个集合或数组，想过滤掉 `null` 值然后创建 `Map`，可以使用 `Stream` API 来实现。`Stream` API 提供了强大的过滤和收集能力，可以轻松地避免 `null` 值。

```java
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class StreamMapExample {
    public static void main(String[] args) {
        Map<String, String> map = Stream.of(
                new AbstractMap.SimpleEntry<>("key1", "value1"),
                new AbstractMap.SimpleEntry<>("key2", null),  // 可以包含null
                new AbstractMap.SimpleEntry<>("key3", "value3")
        )
        .filter(entry -> entry.getValue() != null)  // 过滤掉null值
        .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));

        System.out.println(map);  // {key1=value1, key3=value3}
    }
}
```

这种方式通过 `filter()` 方法可以轻松去除 `null` 值。

### 2. 使用 `Guava` 的 `ImmutableMap`

如果你正在使用 Google 的 Guava 库，它提供了 `ImmutableMap` 类，可以创建不可变的 Map。在 Guava 中，也不允许 `null` 键或值，但你可以使用 `Maps.filterValues()` 来过滤掉 `null` 值。

```java
import com.google.common.collect.ImmutableMap;
import com.google.common.collect.Maps;

import java.util.Map;

public class GuavaMapExample {
    public static void main(String[] args) {
        Map<String, String> map = ImmutableMap.of("key1", "value1", "key2", null);  // 抛出 NullPointerException

        // 过滤掉 null 值
        Map<String, String> filteredMap = Maps.filterValues(map, value -> value != null);

        System.out.println(filteredMap);
    }
}
```

请注意，`Guava` 里的 `ImmutableMap` 也不允许 `null` 值，但结合 `Maps.filterValues()`，你可以创建并过滤 `null` 值。

### 3. 使用 `Apache Commons Collections` 的 `MapUtils`

`Apache Commons Collections` 提供了许多实用工具类，其中 `MapUtils` 可以帮助你处理 `null` 值。`MapUtils.safeAddToMap()` 可以避免 `null` 值并且不抛出异常。

```java
import org.apache.commons.collections4.MapUtils;

import java.util.HashMap;
import java.util.Map;

public class CommonsMapExample {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        MapUtils.safeAddToMap(map, "key1", "value1");  // 正常添加
        MapUtils.safeAddToMap(map, "key2", null);      // 忽略null值

        System.out.println(map);  // {key1=value1}
    }
}
```

在这个例子中，`safeAddToMap()` 会安全地处理 `null` 值，不会抛出异常，同时不会将 `null` 值添加到 `Map` 中。

### 4. 自定义工具方法

如果你希望灵活控制 `null` 值的处理逻辑，可以编写一个自定义工具方法，封装你的 `Map` 构建逻辑，并根据需求处理 `null` 值。

```java
import java.util.HashMap;
import java.util.Map;

public class CustomMapUtils {
    public static <K, V> Map<K, V> createMap(K key1, V value1, K key2, V value2) {
        Map<K, V> map = new HashMap<>();
        if (value1 != null) map.put(key1, value1);
        if (value2 != null) map.put(key2, value2);
        return map;
    }

    public static void main(String[] args) {
        Map<String, String> map = createMap("key1", "value1", "key2", null);
        System.out.println(map);  // {key1=value1}
    }
}
```

在这个例子中，`createMap()` 可以过滤掉 `null` 值，并且允许你灵活地添加更多逻辑来处理不同情况。

### 5. 使用 `Optional` + 默认值

你可以结合 `Optional` 设置默认值来避免 `null`，这样即使值为 `null`，也会被替换为默认值。

```java
import java.util.Map;
import java.util.Optional;

public class OptionalDefaultValueExample {
    public static void main(String[] args) {
        Map<String, String> map = Map.of(
            "key1", Optional.ofNullable("value1").orElse("default"),
            "key2", Optional.ofNullable(null).orElse("default")  // 使用默认值替代null
        );

        System.out.println(map);  // {key1=value1, key2=default}
    }
}
```

### 总结

除了使用 `HashMap` 或 `Optional` 外，你还可以通过：
1. 使用 `Stream` 和 `Collectors` 动态过滤 `null` 值。
2. 使用 Guava、Apache Commons 这些工具库提供的过滤或安全添加方法。
3. 编写自定义工具方法，根据需求处理 `null`。
4. 使用 `Optional` 提供默认值避免 `null` 值。

根据你的具体场景，选择最适合的方式来处理 `null` 值。

# 6. 那ofentries中允许value为null吗？
在 `Java 9` 引入的 `Map.ofEntries()` 方法中，**同样不允许 `null` 键或 `null` 值**。如果你尝试使用 `Map.ofEntries()` 创建包含 `null` 值的 `Map`，将会抛出 `NullPointerException`。

### 示例

```java
import java.util.Map;
import java.util.Map.Entry;

public class MapOfEntriesExample {
    public static void main(String[] args) {
        // 这段代码会抛出 NullPointerException，因为 value 为 null
        Map<String, String> map = Map.ofEntries(
                Entry.of("key1", "value1"),
                Entry.of("key2", null)  // 不允许 null 值
        );
    }
}
```

输出：

```
Exception in thread "main" java.lang.NullPointerException
	at java.base/java.util.Objects.requireNonNull(Objects.java:221)
	at java.base/java.util.ImmutableCollections$MapN.<init>(ImmutableCollections.java:930)
	at java.base/java.util.Map.ofEntries(Map.java:1148)
```

### `Map.ofEntries()` 与 `Map.of()` 的相似性
- **`Map.of()`** 和 **`Map.ofEntries()`** 都不允许 `null` 键或 `null` 值。
- 它们创建的 `Map` 都是不可变的，因此不能修改。
- 如果你试图插入 `null` 值，都会抛出 `NullPointerException`。

### 如果需要允许 `null` 值的替代方案
如果你确实需要允许 `null` 值，可以考虑以下几种方式：

#### 1. 使用 `HashMap`

`HashMap` 是一个允许 `null` 值的 `Map` 实现，可以手动将 `Entry` 添加到 `HashMap` 中。

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapExample {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        map.put("key1", "value1");
        map.put("key2", null);  // 允许null值

        System.out.println(map);  // {key1=value1, key2=null}
    }
}
```

#### 2. 使用 `Stream` 和 `Collectors.toMap()`

通过 `Stream` 构建 `Map` 并使用 `Collectors.toMap()` 可以灵活地控制 `null` 值的处理逻辑。例如，你可以允许 `null` 值，也可以选择过滤掉它们。

```java
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;
import java.util.AbstractMap;

public class StreamMapExample {
    public static void main(String[] args) {
        Map<String, String> map = Stream.of(
                new AbstractMap.SimpleEntry<>("key1", "value1"),
                new AbstractMap.SimpleEntry<>("key2", null)  // 可以允许null值
        ).collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));

        System.out.println(map);  // {key1=value1, key2=null}
    }
}
```

### 总结
- **`Map.ofEntries()`** 和 **`Map.of()`** 不允许 `null` 值和 `null` 键。
- 如果你需要允许 `null`，可以使用 `HashMap` 或 `Stream` + `Collectors.toMap()` 的方式来实现。

# 7. hashmap允许key有多个null吗？
在 `Java` 中，`HashMap` **允许多个键为 `null` 的值**，但只能有一个 `null` 键。这是因为 `HashMap` 通过键的 `hashCode()` 来决定键值对的存储位置，而 `null` 键是特殊处理的。

具体来说，`HashMap` 对 `null` 键会有一套独特的逻辑：
- `HashMap` 内部使用链表或者树来存储元素，`null` 键会被放在索引为 0 的位置，其他键则根据其 `hashCode()` 值计算索引。
- 因为只能有一个 `null` 键，任何对 `null` 键的重复 `put` 操作会覆盖之前的值。

### 示例

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapNullKeyExample {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        map.put(null, "value1");  // 第一个 null 键
        map.put(null, "value2");  // 第二次插入 null 键，会覆盖之前的值

        System.out.println(map);  // 输出 {null=value2}
    }
}
```

输出：

```
{null=value2}
```

在上面的示例中，`HashMap` 允许插入 `null` 键，但是当第二次插入 `null` 键时，`value1` 被 `value2` 覆盖。因此，`HashMap` 中始终只有一个 `null` 键。

### 总结
- `HashMap` **允许** `null` 键，但只能存储一个 `null` 键。
- 当多次 `put(null, value)` 时，后面的值会覆盖之前的值。
- 允许多个值为 `null`，即 `null` 可以作为 `HashMap` 中的值存在多个。

  # 8. List.of中的，也不能为null是吧？
  是的，`Java 9` 引入的 `List.of()` **不允许包含 `null` 元素**。如果你尝试在 `List.of()` 中传入 `null`，将会抛出 `NullPointerException`。

### 示例

```java
import java.util.List;

public class ListOfExample {
    public static void main(String[] args) {
        // 这段代码会抛出 NullPointerException，因为包含 null 元素
        List<String> list = List.of("a", "b", null);
    }
}
```

输出：

```
Exception in thread "main" java.lang.NullPointerException
	at java.base/java.util.Objects.requireNonNull(Objects.java:246)
	at java.base/java.util.ImmutableCollections$ListN.<init>(ImmutableCollections.java:229)
	at java.base/java.util.List.of(List.java:876)
```

在 `List.of()` 中，**不能包含任何 `null` 元素**。`List.of()` 创建的列表是不可变的（immutable），并且对 `null` 值有严格的限制。

### 替代方案

如果你需要一个允许 `null` 的列表，可以使用 `ArrayList` 或者其他可变的 `List` 实现类，比如 `LinkedList`。

#### 使用 `ArrayList`

```java
import java.util.ArrayList;
import java.util.List;

public class ArrayListExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("a");
        list.add("b");
        list.add(null);  // ArrayList 允许 null 元素

        System.out.println(list);  // [a, b, null]
    }
}
```

#### 使用 `Arrays.asList()`

`Arrays.asList()` 也允许 `null` 值，但它返回的列表是一个固定大小的列表，不能添加或删除元素，只能修改现有的元素。

```java
import java.util.Arrays;
import java.util.List;

public class ArraysAsListExample {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("a", "b", null);  // 允许 null 值

        System.out.println(list);  // [a, b, null]
    }
}
```

不过请注意，`Arrays.asList()` 返回的列表是固定大小的，不能进行增删操作。

### 总结

- `List.of()` 不允许 `null` 元素。
- 如果你需要允许 `null` 值，可以使用 `ArrayList` 或 `Arrays.asList()`。
- `List.of()` 创建的列表是不可变的，而 `ArrayList` 是可变的，可以动态添加、删除和修改元素。

  # 9. 
