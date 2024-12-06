---
layout:     post
title:      CompletableFuture
subtitle:   深入学习高级的java异步任务编排框架CompletableFuture
categories: [java高并发]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 背景了解
## 1.1 什么是CompletableFuture？
CompletableFuture是java8中新增的一个类，算是对Future的一种增强，用起来很方便，也是会经常用到的一个工具类，熟悉一下。

**CompletionStage接口**
- CompletionStage代表异步计算过程中的某一个阶段，一个阶段完成以后可能会触发另外一个阶段
- 一个阶段的计算执行可以是一个Function，Consumer或者Runnable。比如：stage.thenApply(x -> square(x)).thenAccept(x -> System.out.print(x)).thenRun(() -> System.out.println())
- 一个阶段的执行可能是被单个阶段的完成触发，也可能是由多个阶段一起触发

**CompletableFuture类**
- 在Java8中，CompletableFuture提供了非常强大的Future的扩展功能，可以帮助我们简化异步编程的复杂性，并且提供了函数式编程的能力，可以通过回调的方式处理计算结果，也提供了转换和组合 CompletableFuture 的方法。
- 它可能代表一个明确完成的Future，也有可能代表一个完成阶段（ CompletionStage ），它支持在计算完成以后触发一些函数或执行某些动作。
- 它实现了Future和CompletionStage接口

# 2.常见的方法
**runAsync 和 supplyAsync方法**
CompletableFuture 提供了四个静态方法来创建一个异步操作。   
```java
public static CompletableFuture<Void> runAsync(Runnable runnable)
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```
没有指定Executor的方法会使用ForkJoinPool.commonPool() 作为它的线程池执行异步代码。如果指定线程池，则使用指定的线程池运行。以下所有的方法都类同。   
- runAsync方法不支持返回值。
- supplyAsync可以支持返回值。

***示例代码***
```java
//无返回值
public static void runAsync() throws Exception {
    CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
        }
        System.out.println("run end ...");
    });
    future.get();
}
//有返回值
public static void supplyAsync() throws Exception {         
    CompletableFuture<Long> future = CompletableFuture.supplyAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
        }
        System.out.println("run end ...");
        return System.currentTimeMillis();
    });
    long time = future.get();
    System.out.println("time = "+time);
}
```

**计算结果完成时的回调方法**
当CompletableFuture的计算结果完成，或者抛出异常的时候，可以执行特定的Action。主要是下面的方法：   
```java
public CompletableFuture<T> whenComplete(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
public CompletableFuture<T> exceptionally(Function<Throwable,? extends T> fn)
```
可以看到Action的类型是BiConsumer<? super T,? super Throwable>它可以处理正常的计算结果，或者异常情况。   

***whenComplete 和 whenCompleteAsync 的区别：***
- whenComplete：是执行当前任务的线程执行继续执行 whenComplete 的任务。
- whenCompleteAsync：是执行把 whenCompleteAsync 这个任务继续提交给线程池来进行执行。

***示例代码***
```java
public static void whenComplete() throws Exception {
    CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
        }
        if(new Random().nextInt()%2>=0) {
            int i = 12/0;
        }
        System.out.println("run end ...");
    });
    future.whenComplete(new BiConsumer<Void, Throwable>() {
        @Override
        public void accept(Void t, Throwable action) {
            System.out.println("执行完成！");
        }
    });
    future.exceptionally(new Function<Throwable, Void>() {
        @Override
        public Void apply(Throwable t) {
            System.out.println("执行失败！"+t.getMessage());
            return null;
        }
    });
    TimeUnit.SECONDS.sleep(2);
}
```

**thenApply 方法**
当一个线程依赖另一个线程时，可以使用 thenApply 方法来把这两个线程串行化。   
```java
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
```
Function<? super T,? extends U>   
T：上一个任务返回结果的类型   
U：当前任务的返回值类型   
***示例代码***
```java
private static void thenApply() throws Exception {
    CompletableFuture<Long> future = CompletableFuture.supplyAsync(new Supplier<Long>() {
        @Override
        public Long get() {
            long result = new Random().nextInt(100);
            System.out.println("result1="+result);
            return result;
        }
    }).thenApply(new Function<Long, Long>() {
        @Override
        public Long apply(Long t) {
            long result = t*5;
            System.out.println("result2="+result);
            return result;
        }
    });
    long result = future.get();
    System.out.println(result);
}
```
第二个任务依赖第一个任务的结果。   

**handle 方法**
handle 是执行任务完成时对结果的处理。   
handle 方法和 thenApply 方法处理方式基本一样。不同的是 handle 是在任务完成后再执行，还可以处理异常的任务。thenApply 只可以执行正常的任务，任务出现异常则不执行 thenApply 方法。   
```java
public <U> CompletionStage<U> handle(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn,Executor executor);
```
***示例代码***
```java
public static void handle() throws Exception{
    CompletableFuture<Integer> future = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int i= 10/0;
            return new Random().nextInt(10);
        }
    }).handle(new BiFunction<Integer, Throwable, Integer>() {
        @Override
        public Integer apply(Integer param, Throwable throwable) {
            int result = -1;
            if(throwable==null){
                result = param * 2;
            }else{
                System.out.println(throwable.getMessage());
            }
            return result;
        }
     });
    System.out.println(future.get());
}
```
从示例中可以看出，在 handle 中可以根据任务是否有异常来进行做相应的后续处理操作。而 thenApply 方法，如果上个任务出现错误，则不会执行 thenApply 方法。

**thenAccept 消费处理结果**
接收任务的处理结果，并消费处理，无返回结果。   
```java
public CompletionStage<Void> thenAccept(Consumer<? super T> action);
public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action);
public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action,Executor executor);
```
***示例代码***
```java
public static void thenAccept() throws Exception{
    CompletableFuture<Void> future = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            return new Random().nextInt(10);
        }
    }).thenAccept(integer -> {
        System.out.println(integer);
    });
    future.get();
}
```
从示例代码中可以看出，该方法只是消费执行完成的任务，并可以根据上面的任务返回的结果进行处理。并没有后续的输错操作。   

**thenRun 方法**
跟 thenAccept 方法不一样的是，不关心任务的处理结果。只要上面的任务执行完成，就开始执行 thenAccept 。   
```java
public CompletionStage<Void> thenRun(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action,Executor executor);
```
***示例代码***
```java
public static void thenRun() throws Exception{
    CompletableFuture<Void> future = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            return new Random().nextInt(10);
        }
    }).thenRun(() -> {
        System.out.println("thenRun ...");
    });
    future.get();
}
```
该方法同 thenAccept 方法类似。不同的是上个任务处理完成后，并不会把计算的结果传给 thenRun 方法。只是处理玩任务后，执行 thenAccept 的后续操作。   

**thenCombine 合并任务**
thenCombine 会把 两个 CompletionStage 的任务都执行完成后，把两个任务的结果一块交给 thenCombine 来处理。   
```java
public <U,V> CompletionStage<V> thenCombine(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn,Executor executor);
```
***示例代码***
```java
private static void thenCombine() throws Exception {
    CompletableFuture<String> future1 = CompletableFuture.supplyAsync(new Supplier<String>() {
        @Override
        public String get() {
            return "hello";
        }
    });
    CompletableFuture<String> future2 = CompletableFuture.supplyAsync(new Supplier<String>() {
        @Override
        public String get() {
            return "hello";
        }
    });
    CompletableFuture<String> result = future1.thenCombine(future2, new BiFunction<String, String, String>() {
        @Override
        public String apply(String t, String u) {
            return t+" "+u;
        }
    });
    System.out.println(result.get());
}
```

**thenAcceptBoth**
当两个CompletionStage都执行完成后，把结果一块交给thenAcceptBoth来进行消耗   
```java
public <U> CompletionStage<Void> thenAcceptBoth(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action,     Executor executor);
```
***示例代码***
```java
private static void thenAcceptBoth() throws Exception {
    CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f1="+t);
            return t;
        }
    });
    CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f2="+t);
            return t;
        }
    });
    f1.thenAcceptBoth(f2, new BiConsumer<Integer, Integer>() {
        @Override
        public void accept(Integer t, Integer u) {
            System.out.println("f1="+t+";f2="+u+";");
        }
    });
}
```

**applyToEither 方法**
两个CompletionStage，谁执行返回的结果快，我就用那个CompletionStage的结果进行下一步的转化操作。   
```java
public <U> CompletionStage<U> applyToEither(CompletionStage<? extends T> other,Function<? super T, U> fn);
public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn);
public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn,Executor executor);
```
***示例代码***
```java
private static void applyToEither() throws Exception {
    CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f1="+t);
            return t;
        }
    });
    CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f2="+t);
            return t;
        }
    });
    CompletableFuture<Integer> result = f1.applyToEither(f2, new Function<Integer, Integer>() {
        @Override
        public Integer apply(Integer t) {
            System.out.println(t);
            return t * 2;
        }
    });
    System.out.println(result.get());
}
```

**acceptEither 方法**
两个CompletionStage，谁执行返回的结果快，我就用那个CompletionStage的结果进行下一步的消耗操作。   
```java
public CompletionStage<Void> acceptEither(CompletionStage<? extends T> other,Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action,Executor executor);
```
***示例代码***
```java
private static void acceptEither() throws Exception {
    CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f1="+t);
            return t;
        }
    });
    CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f2="+t);
            return t;
        }
    });
    f1.acceptEither(f2, new Consumer<Integer>() {
        @Override
        public void accept(Integer t) {
            System.out.println(t);
        }
    });
}
```

**runAfterEither 方法**
两个CompletionStage，任何一个完成了都会执行下一步的操作（Runnable）   
```java
public CompletionStage<Void> runAfterEither(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action,Executor executor);
```
***示例代码***
```java
private static void runAfterEither() throws Exception {
    CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f1="+t);
            return t;
        }
    });
    CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f2="+t);
            return t;
        }
    });
    f1.runAfterEither(f2, new Runnable() {
        @Override
        public void run() {
            System.out.println("上面有一个已经完成了。");
        }
    });
}
```

**runAfterBoth**
两个CompletionStage，都完成了计算才会执行下一步的操作（Runnable）   
```java
public CompletionStage<Void> runAfterBoth(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action,Executor executor);
```
***示例代码***
```java
private static void runAfterBoth() throws Exception {
    CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f1="+t);
            return t;
        }
    });
    CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f2="+t);
            return t;
        }
    });
    f1.runAfterBoth(f2, new Runnable() {
        @Override
        public void run() {
            System.out.println("上面两个任务都执行完成了。");
        }
    });
}
```

**thenCompose 方法**
thenCompose 方法允许你对两个 CompletionStage 进行流水线操作，第一个操作完成时，将其结果作为参数传递给第二个操作。   
```java
public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn);
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn) ;
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn, Executor executor) ;
```
***示例代码***
```java
private static void thenCompose() throws Exception {
        CompletableFuture<Integer> f = CompletableFuture.supplyAsync(new Supplier<Integer>() {
            @Override
            public Integer get() {
                int t = new Random().nextInt(3);
                System.out.println("t1="+t);
                return t;
            }
        }).thenCompose(new Function<Integer, CompletionStage<Integer>>() {
            @Override
            public CompletionStage<Integer> apply(Integer param) {
                return CompletableFuture.supplyAsync(new Supplier<Integer>() {
                    @Override
                    public Integer get() {
                        int t = param *2;
                        System.out.println("t2="+t);
                        return t;
                    }
                });
            }
        });
        System.out.println("thenCompose result : "+f.get());
    }
```

# 3. CompletableFuture如何保证线程安全？   
首先明确一点，不论是使用传统的线程Thread、还是线程池thread pool、还是更优雅的Future、CompletableFuture等工具，其底层只是对传统的thread进行了管理。   
换言之，线程池是负责管理线程对象的容器，而Future和CompletableFuture在线程池的基础之上，提供了更优雅的控制异步任务、编排异步任务的能力。   
因此，无论千变万化，线程对象本身才是最核心的东西。   

CompletableFuture本身并不能保证线程安全，它只是将异步任务负责编排，按照约定的规则提交给线程池而已。   
至于这些异步任务提交到线程池中，被某些线程调度，在调度的过程中是否存在线程安全的问题，主要取决于是否存在多个线程同时修改同一个共享变量的场景。   
如果提交的任务中存在可能被多个线程对象共享修改的情况，那么CompletableFuture就不是线程安全的。    
如果不存在这种场景，那么CompletableFuture对于任务的操作就是线程安全的。    

因此，得出结果：不论是使用线程、线程池、Future、还是CompletableFuture去操作某些对象，是否存在线程安全问题，取决于这些对象本身是否存在共享修改操作。   

# 4. delayedExecutor
在 Java 的 **CompletableFuture** 中，`delayedExecutor` 是一种工具，用于在指定的延迟之后执行任务。它返回一个 `Executor`，这个 `Executor` 会在给定的延迟时间后调度任务的执行，而不是立即执行任务。

## 4.1 `delayedExecutor` 方法定义
```java
public static Executor delayedExecutor(long delay, TimeUnit unit)
public static Executor delayedExecutor(long delay, TimeUnit unit, ScheduledExecutorService executor)
```

**参数说明**
1. **`delay`**: 延迟时间。
2. **`unit`**: 延迟时间的单位，例如 `TimeUnit.SECONDS`。
3. **`executor`** *(可选)*: 一个 `ScheduledExecutorService`，用于调度任务。如果未提供，默认使用全局的 `ForkJoinPool`。


**工作原理**
- **延迟执行**：当你将任务提交给由 `delayedExecutor` 创建的 `Executor` 时，任务不会立刻执行，而是在指定的延迟时间后才会执行。
- **线程管理**：如果没有提供自定义的 `ScheduledExecutorService`，`delayedExecutor` 会利用全局的 `ForkJoinPool`，或内部使用 `CompletableFuture` 的默认线程池。

**常见用法**

**1. 使用默认线程池**
以下代码会在 3 秒后执行任务：
```java
Executor delayedExecutor = CompletableFuture.delayedExecutor(3, TimeUnit.SECONDS);

CompletableFuture.runAsync(() -> {
    System.out.println("Task executed after 3 seconds");
}, delayedExecutor);
```

**2. 使用自定义的 `ScheduledExecutorService`**
你可以提供自己的线程池进行调度：
```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

Executor delayedExecutor = CompletableFuture.delayedExecutor(5, TimeUnit.SECONDS, scheduler);

CompletableFuture.runAsync(() -> {
    System.out.println("Task executed after 5 seconds using custom scheduler");
}, delayedExecutor);

scheduler.shutdown(); // 用完记得关闭线程池
```

**典型场景**
1. **实现超时控制**：在 `CompletableFuture` 中，结合 `orTimeout` 或 `completeOnTimeout` 使用，用于实现操作的延时触发或超时机制。
2. **模拟延迟**：用于测试和模拟延时操作，例如网络请求或耗时操作。
3. **定时任务**：实现一些简单的定时任务调度。

**完整示例：超时控制**
结合 `delayedExecutor` 和 `supplyAsync` 实现超时机制：
```java
Executor delayedExecutor = CompletableFuture.delayedExecutor(3, TimeUnit.SECONDS);

CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(1000); // 模拟耗时任务
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
    return "Task completed";
});

// 3 秒后触发超时，任务未完成时将使用默认值
future.completeOnTimeout("Timeout occurred", 3, TimeUnit.SECONDS)
      .thenAccept(System.out::println);
```

**注意事项**
1. **线程池管理**：如果使用自定义的 `ScheduledExecutorService`，务必在适当的时候关闭线程池以释放资源。
2. **任务堆积问题**：在高并发情况下，如果大量任务被提交到延迟的 `Executor` 中，可能会导致任务堆积，需要监控线程池的容量和使用率。

**总结**
`delayedExecutor` 是一种强大的工具，可以轻松实现延时任务的调度。在异步编程中，尤其是需要结合超时处理或定时操作的场景下，它可以大大简化代码的实现。

# 5. delayedExecutor针对的是任务
`delayedExecutor` 的延迟是针对**任务**的，而不是针对自定义的线程池。具体来说：

- **延迟针对任务**：任务被提交到由 `delayedExecutor` 创建的 `Executor` 时，会按照指定的延迟时间调度任务的执行，而不会立即运行任务。
- **线程池的作用**：如果提供了自定义的 `ScheduledExecutorService`，它仅作为任务调度的底层实现，而延迟逻辑由 `delayedExecutor` 管理，与线程池的配置无关。

**运行机制解析**
1. 当调用 `delayedExecutor` 创建的 `Executor` 时，任务不会立即交给线程池执行。
2. `delayedExecutor` 会记录下延迟时间，并通过提供的 `ScheduledExecutorService` 或默认的调度器，在延迟时间到达后再调度任务。
3. 延迟只针对每个任务的调度，而不会改变线程池的运行方式或行为。

**关键点：任务与线程池的关系**
**如果没有提供自定义线程池**
`delayedExecutor` 使用全局的 `ForkJoinPool` 或默认的线程池：
```java
Executor delayedExecutor = CompletableFuture.delayedExecutor(2, TimeUnit.SECONDS);

CompletableFuture.runAsync(() -> {
    System.out.println("Task executed after 2 seconds");
}, delayedExecutor);
```
- **延迟行为**：任务将在 2 秒后开始执行。
- **线程池行为**：`ForkJoinPool` 管理线程的实际执行，但延迟逻辑由 `delayedExecutor` 管理。

**如果提供了自定义线程池**
延迟逻辑仍然在任务级别，而线程池只是负责具体的任务执行：
```java
ScheduledExecutorService customScheduler = Executors.newScheduledThreadPool(1);

Executor delayedExecutor = CompletableFuture.delayedExecutor(3, TimeUnit.SECONDS, customScheduler);

CompletableFuture.runAsync(() -> {
    System.out.println("Task executed after 3 seconds using custom scheduler");
}, delayedExecutor);

customScheduler.shutdown();
```
- **延迟行为**：任务将在 3 秒后开始调度。
- **线程池行为**：`customScheduler` 只是负责实际运行任务的线程，延迟逻辑仍由 `delayedExecutor` 控制。

**验证：多个任务是否独立延迟**
每个任务的延迟时间是独立的，与线程池的行为无关。

示例：
```java
ScheduledExecutorService customScheduler = Executors.newScheduledThreadPool(2);

Executor delayedExecutor = CompletableFuture.delayedExecutor(2, TimeUnit.SECONDS, customScheduler);

// 提交多个任务
CompletableFuture.runAsync(() -> System.out.println("Task 1 executed at " + System.currentTimeMillis()), delayedExecutor);
CompletableFuture.runAsync(() -> System.out.println("Task 2 executed at " + System.currentTimeMillis()), delayedExecutor);

customScheduler.shutdown();
```
输出（假设开始时间为 1000 毫秒）：
```
Task 1 executed at 3000
Task 2 executed at 3000
```
说明两个任务都独立延迟了 2 秒，即每个任务都有自己的延迟，不会因线程池行为受到影响。

**总结**
1. **延迟针对任务**：`delayedExecutor` 控制任务的延迟调度，而不是改变线程池的行为。
2. **线程池只是执行**：自定义的 `ScheduledExecutorService` 仅用于运行任务，延迟逻辑独立于线程池的配置。
3. **任务独立延迟**：即使多个任务使用相同的 `delayedExecutor`，它们的延迟时间也是相互独立的，不受其他任务或线程池的影响。
