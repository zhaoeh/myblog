---
layout:     post
title:      彻底掌握spring中的事件监听器
subtitle:   从springboot出发完整分析spring容器中的事件监听模型
categories: [springboot专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 监听器
监听器基于观察者模式，简而言之监听器的本质就是一个钩子对象，当服务器发生某个动作时，就会触发该钩子对象的逻辑执行，从而达到该钩子对象貌似在监听服务器的某些行为的表象。   
对于监听器的原理探究，请参考： [监听器和事件驱动模型](https://zhaoehcode.gitee.io/2021/04/06/%E4%BA%8B%E4%BB%B6%E9%A9%B1%E5%8A%A8%E6%A8%A1%E5%9E%8B/)   

# 2. 向springboot中注册ApplicationListener
这里先springboot中注册的监听器，实际上是ApplicationListener对象，该对象是spring容器中定义的接口。   
如果我们想让自定定义的ApplicationListener监听器对象，直接对springboot容器以及整个spring容器中发布的早期事件都感兴趣（感兴趣实际上就是可以对这些事件进行订阅），那么我们自定义的ApplicationListener对象就必须从springboot容器一开始启动就要注册到事件广播器中去。   
目前我们看到，通过在spring.factories文件中配置自己的ApplicationListener实现类，就可以直接将这个对象注册给springboot容器。    

下面我们逐步分析。   

## 2.1 setListeners 
在构造SpringApplication对象时，有如下代码：   
```java
// 反射实例化spring.factories文件中key为ApplicationListener的所有实现类对象并聚合为一个list，然后注册到SpringApplication对象中。
this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
```  
点进去如下：
```java
    public void setListeners(Collection<? extends ApplicationListener<?>> listeners) {
        this.listeners = new ArrayList(listeners);
    }
```
可以看到，是在初始化SpringApplication对象中的 listeners 集合，该集合属性定义如下：   
```java
private List<ApplicationListener<?>> listeners;
```
它是一个集合，类型是 ApplicationListener。我们点进ApplicationListener的源码，如下：
```java
package org.springframework.context;

import java.util.EventListener;
import java.util.function.Consumer;

@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    void onApplicationEvent(E event);

    static <T> ApplicationListener<PayloadApplicationEvent<T>> forPayload(Consumer<T> consumer) {
        return (event) -> {
            consumer.accept(event.getPayload());
        };
    }
}

```
<b><u><font color="#dc143c">可以看到，ApplicationListener是spring时期的接口。</font>  </u></b>         
而且是一个函数式接口，只有一个需要客户端实现的抽象方法：
```java
void onApplicationEvent(E event);
```
且该方法接收一个事件，事件类型为 ApplicationEvent 或者其子类型。   
到这里，有一个疑惑，<b><u><font color="#dc143c">ApplicationListener既然是spring容器的接口，在这里注册到springboot容器的SpringApplication中，那spring容器怎么用？</font>  </u></b>   
既然spring本身暴露了ApplicationListener钩子接口，那意味着要使用它必须得进入到spring的生命周期中才可能回调使用，而不可能在springboot生命周期中就直接回调它。   
具体springboot中注册的ApplicationListener钩子对象集是如何介入到spring的生命周期中的呢？我们一步步分析。   

## 2.2 getSpringFactoriesInstances
getSpringFactoriesInstances方法我们之前分析过了，就是遍历当前classpath和所有jar包里面的classpath，解析其中的META-INF下的spring.factories文件，从该文件中找到指定类型的key-value，然后通过反射实例化指定的对象并返回。    
这种思想实际上借鉴的是java的SPI机制，即服务提供接口。   
我们再来简单分析一下.   

getSpringFactoriesInstances：
```java
    private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
        return this.getSpringFactoriesInstances(type, new Class[0]);
    }
```
getSpringFactoriesInstances:
```java
    private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
        ClassLoader classLoader = this.getClassLoader();
        // 获取目标类型的所有value配置
        Set<String> names = new LinkedHashSet(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
        
        // 反射出目标实现类的所有对象实例
        List<T> instances = this.createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
        AnnotationAwareOrderComparator.sort(instances);
        return instances;
    }
```

loadFactoryNames:
```java
    public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
        ClassLoader classLoaderToUse = classLoader;
        if (classLoader == null) {
            classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
        }

        String factoryTypeName = factoryType.getName();
        // 根据传入的类型的权限定名（作为key），获取对应的value配置（一堆实现类的权限定名集合）
        return (List)loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
    }
```

loadSpringFactories:
```java
 private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
        Map<String, List<String>> result = (Map)cache.get(classLoader);
        if (result != null) {
            return result;
        } else {
            HashMap result = new HashMap();

            try {
                // 通过classLoader加载所有资源下的META-INF/spring.factories文件
                // 将文件中的所有内容都聚合成一个大的map返回
                Enumeration urls = classLoader.getResources("META-INF/spring.factories");

                while(urls.hasMoreElements()) {
                    URL url = (URL)urls.nextElement();
                    UrlResource resource = new UrlResource(url);
                    Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                    Iterator var6 = properties.entrySet().iterator();

                    while(var6.hasNext()) {
                        Entry<?, ?> entry = (Entry)var6.next();
                        String factoryTypeName = ((String)entry.getKey()).trim();
                        String[] factoryImplementationNames = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
                        String[] var10 = factoryImplementationNames;
                        int var11 = factoryImplementationNames.length;

                        for(int var12 = 0; var12 < var11; ++var12) {
                            String factoryImplementationName = var10[var12];
                            ((List)result.computeIfAbsent(factoryTypeName, (key) -> {
                                return new ArrayList();
                            })).add(factoryImplementationName.trim());
                        }
                    }
                }

                result.replaceAll((factoryType, implementations) -> {
                    return (List)implementations.stream().distinct().collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList));
                });
                cache.put(classLoader, result);
                return result;
            } catch (IOException var14) {
                throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var14);
            }
        }
    }
```

createSpringFactoriesInstances:
```java
    private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, ClassLoader classLoader, Object[] args, Set<String> names) {
        List<T> instances = new ArrayList(names.size());
        Iterator var7 = names.iterator();

        // 遍历传入的names权限定名称，通过反射对应的构造器实例化对象
        // 要求目标实现类必须存在无参构造器才能够成功反射出对象实例
        while(var7.hasNext()) {
            String name = (String)var7.next();

            try {
                Class<?> instanceClass = ClassUtils.forName(name, classLoader);
                Assert.isAssignable(type, instanceClass);
                Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
                T instance = BeanUtils.instantiateClass(constructor, args);
                instances.add(instance);
            } catch (Throwable var12) {
                throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, var12);
            }
        }

        return instances;
    }
```

因此，这段逻辑实际上是从所有的spring.factories文件中找出key为org.springframework.context.ApplicationListener的所有value值，然后通过反射实例化出这些实现类的所有对象，聚合到SpringApplication的listeners属性中。   

## 2.3 springboot的spring.factories
一般springboot通过pom会依赖非常基础的几个jar，一个是 spring-boot-xxx.jar；另外一个是 spring-boot-autoconfigure-xxx.jar。   
我们先看spring-boot-xxx.jar中的spring.factories中，关于ApplicationListener的配置：    
```properties
# Application Listeners
org.springframework.context.ApplicationListener=\   
org.springframework.boot.ClearCachesApplicationListener,\   
org.springframework.boot.builder.ParentContextCloserApplicationListener,\   
org.springframework.boot.context.FileEncodingApplicationListener,\   
org.springframework.boot.context.config.AnsiOutputApplicationListener,\   
org.springframework.boot.context.config.DelegatingApplicationListener,\   
org.springframework.boot.context.logging.LoggingApplicationListener,\   
org.springframework.boot.env.EnvironmentPostProcessorApplicationListener   
``` 
再看spring-boot-autoconfigure-xxx.jar中：
```properties
# Application Listeners
org.springframework.context.ApplicationListener=\   
org.springframework.boot.autoconfigure.BackgroundPreinitializer
```
上面的配置类，都是springboot自身默认实现好的ApplicationListener子类，这些子类通过配置到spring.factories文件中就可以自动注册到springboot容器中了。   

截止到目前，通过下面的代码：
```properties
this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
```   
我们已经知道了，SpringApplication对象的listeners已经被初始化了，它的值全部是spring.factories中配置的ApplicationListener实现类对象。   
到这里，spring时期的ApplicationListener对象集已经完全注册到springboot的SpringApplication中了。   

# 3. 如何使用springboot中注册的ApplicationListener呢？
正如上面的疑惑，ApplicationListener是spring时期提供的事件监听器钩子接口，而上面springboot将其注册到SpringApplication对象中了。   
那啥时候使用注册的这些ApplicationListener对象呢？这些注册到springboot容器中的监听器又是如何被spring直接回调使用的呢？      
我们一层一层的来剥开spring事件监听（通知）机制的神秘面纱。   

## 3.1 初始化 SpringApplicationRunListeners 
注意，这里在springboot容器中初始化的对象是 SpringApplicationRunListeners ，它是springboot自己定义的一个类。   

### 3.1.1 run方法
```java
public ConfigurableApplicationContext run(String... args) {
        long startTime = System.nanoTime();
        DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();
        ConfigurableApplicationContext context = null;
        this.configureHeadlessProperty();
        // 这一行代码是在初始化 SpringApplicationRunListeners 对象
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
        listeners.starting(bootstrapContext, this.mainApplicationClass);

        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            
            // 下面传入 listeners 进行使用
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
            this.configureIgnoreBeanInfo(environment);
            Banner printedBanner = this.printBanner(environment);
            context = this.createApplicationContext();
            context.setApplicationStartup(this.applicationStartup);
            
            // 下面传入 listeners 进行使用
            this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
            this.refreshContext(context);
            this.afterRefresh(context, applicationArguments);
            Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), timeTakenToStartup);
            }

            // 调用 listeners 的started方法
            listeners.started(context, timeTakenToStartup);
            this.callRunners(context, applicationArguments);
        } catch (Throwable var12) {
            // 传入 listeners 进行使用
            this.handleRunFailure(context, var12, listeners);
            throw new IllegalStateException(var12);
        }

        try {
            Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
            // 调用 listeners 的ready 方法
            listeners.ready(context, timeTakenToReady);
            return context;
        } catch (Throwable var11) {
            this.handleRunFailure(context, var11, (SpringApplicationRunListeners)null);
            throw new IllegalStateException(var11);
        }
    }
```
### 3.1.2 getRunListeners初始化SpringApplicationRunListeners对象
```java
    // 该方法返回一个SpringApplicationRunListeners对象
    private SpringApplicationRunListeners getRunListeners(String[] args) {
    
        // 指定反射需要的参数类型，第一个参数是SpringApplication类型，第二个参数是字符串数组类型
        Class<?>[] types = new Class[]{SpringApplication.class, String[].class};
        
        // 调用SpringApplicationRunListeners构造器，构造一个SpringApplicationRunListeners对象。
        return new SpringApplicationRunListeners(logger, this.getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args), this.applicationStartup);
    }
```
SpringApplicationRunListeners构造器如下：
```java
    // 接收3个参数
    // 其中，listeners是一个集合，元素类型为 SpringApplicationRunListener ，注意和上面的 ApplicationListener 区分。
    SpringApplicationRunListeners(Log log, Collection<? extends SpringApplicationRunListener> listeners, ApplicationStartup applicationStartup) {
        this.log = log;
        this.listeners = new ArrayList(listeners);
        this.applicationStartup = applicationStartup;
    }
```
SpringApplicationRunListeners是springboot中定义的一个类，这个类用来对SpringApplicationRunListener的运行监听器集合做包装。   

SpringApplicationRunListener：   
SpringApplicationRunListener 同样是springboot中定义的一个钩子接口，俗称运行监听器。   
```java
// 很关键的一点是，这个方法从spring.factories文件中读取类型为SpringApplicationRunListener的所有实现类对象。
this.getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args)
```
再次解读 getSpringFactoriesInstances 方法：
```java
    // 第一个参数：spring.factories中的key，即需要读取的key
    // 第二个参数：根据value配置的权限定名反射实例化时，构造器需要的参数类型
    // 第三个参数：同样，根据参数类型获取到指定构造器后，需要传入实际参数进行对象的实例化
    private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
        ClassLoader classLoader = this.getClassLoader();
        Set<String> names = new LinkedHashSet(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
        List<T> instances = this.createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
        AnnotationAwareOrderComparator.sort(instances);
        return instances;
    }
```
因此，我们再来看：
```java
    private SpringApplicationRunListeners getRunListeners(String[] args) {
        Class<?>[] types = new Class[]{SpringApplication.class, String[].class};
        // 第一个参数：获取目标类型 SpringApplicationRunListener
        // 第二个参数：types，第一个参数类型为SpringApplication，第二个参数类型为 字符串数组
        // 第三个参数：可变参数，作为构造器的实际参数，传入了当前对象 this和 args，this 表示当前的  SpringApplication
        return new SpringApplicationRunListeners(logger, this.getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args), this.applicationStartup);
    }
```
总结 getRunListeners() 方法：   
（1）读取classpath下spring.factories中key为SpringApplicationRunListener的所有实现类的权限定名，然后传入构造器参数类型和实际参数，反射出对应的实例对象并聚合为一个集合。      
（2）传入第（1）步聚合后的SpringApplicationRunListener实例对象，注入到SpringApplicationRunListeners中，构造出一个SpringApplicationRunListeners实例。

### 3.1.3 spring.factories中SpringApplicationRunListener的配置
```properties
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\   
org.springframework.boot.context.event.EventPublishingRunListener
```
我们可以发现，实际上SpringApplicationRunListener只有一个实现类 EventPublishingRunListener 。   

### 3.1.4 EventPublishingRunListener
```java
package org.springframework.boot.context.event;

// EventPublishingRunListener 是springboot内部对运行监听器 SpringApplicationRunListener 的唯一实现。
// 顾名思义，核心功能就是用来在springboot各个生命周期阶段来发布对应的事件的一个监听器。
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered 
```
对应的构造器：  
上面反射 SpringApplicationRunListener 的实现类，目前看只有一个 EventPublishingRunListener。   
通过反射其实现类的构造器，创建它的实例，因此，下面的构造器我们着重看一下。
```java
    private final SpringApplication application;
    private final String[] args;
    private final SimpleApplicationEventMulticaster initialMulticaster;

    // EventPublishingRunListener 构造器接收两个参数：
    // 第一个参数：SpringApplication 类型。
    // 第二个参数：String数组
    // 这和上面反射时传入的构造器参数类型完全匹配：Class<?>[] types = new Class[]{SpringApplication.class, String[].class};
    // 并且在反射实例化时传入的实际参数：this.getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args)就体现在这里。
    public EventPublishingRunListener(SpringApplication application, String[] args) {
        // 反射时传入的this对象，表示 SpringApplication 实例，此处初始化
        this.application = application;
        // 反射时传入的其他实际参数，此处初始化
        this.args = args;
        
        // 关键点：初始化一个事件广播器对象  SimpleApplicationEventMulticaster
        // 这个对象是真正干活的，真正对事件的发布操作都是委托这个对象去做的
        // 注意，这里是在springboot容器中直接new一个广播器对象
        this.initialMulticaster = new SimpleApplicationEventMulticaster();
        
        // 这一行代码至关重要：将springboot中注册的spring时期的监听器对象集在这个转移到了springboot容器中创建的广播器中了。
        // 主要逻辑：从springboot的SpringApplication对象中取出已经注册的所有SpringListener对象集合，然后注册到springboot容器自己创建的事件广播器中了。
        // 前面我们知道在springboot容器构造SpringApplication对象时，就从spring.factories中实例化了所有的ApplicationListener对象并聚合为一个集合。
        // 那些注册到SpringApplication中的 List<ApplicationListener<?>> listeners 就是在这里被springboot容器真正使用的。
        // 此处直接获取SpringApplication中前期注册的所有List<ApplicationListener<?>> listeners，然后遍历每一个 ApplicationListener.
        // 将其添加到springboot容器自己的事件广播器中。
        // 同时我们要注意，SimpleApplicationEventMulticaster类是spring时期的东西；但在目前这个阶段，仍旧属于springboot容器的启动阶段，springboot容器手动创建了一个广播器对象而已。
        Iterator var3 = application.getListeners().iterator();
        while(var3.hasNext()) {
            ApplicationListener<?> listener = (ApplicationListener)var3.next();
            // 遍历ApplicationListener将其添加到springboot自己的事件广播器对象initialMulticaster中。
            this.initialMulticaster.addApplicationListener(listener);
        }
    }
```

### 3.1.5 ApplicationEventMulticaster事件广播器接口
<b><u><font color="#dc143c">SpringApplicationRunListeners、SpringApplicationRunListener、EventPublishingRunListener都是springboot的接口，而ApplicationEventMulticaster广播器是spring的接口。</font>  </u></b>    
因此，springboot容器中注册的ApplicationListener对象集就是通过注册到广播器ApplicationEventMulticaster中，从而通过委托spring容器中的广播器，来注册目前已经实例化的所有监听器。    
当然，实际上：底层对于监听器的注册、以及事件的发布，其实最终都是委托广播器   ApplicationEventMulticaster 来进行管理的。   
```java
package org.springframework.context.event;

import java.util.function.Predicate;
import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.core.ResolvableType;
import org.springframework.lang.Nullable;

public interface ApplicationEventMulticaster {
    // 向广播器中注册一个 ApplicationListener监听器
    void addApplicationListener(ApplicationListener<?> listener);

    // 向广播器中注册一个ApplicationListener的bean权限定名
    // 从这个方法可以看出：广播器不仅仅支持直接注册一个ApplicationListener实例，还支持先注册一个beanName
    // 之所以要这么做，是因为在一些早期的事件中，spring容器中只有bean定义，还没有开始执行实例化bean的动作，因此，为了不遗漏在这个阶段内对某些事件感兴趣的监听器，它允许先将beanName给注册进来
    // 之后，一旦当广播器发布某个指定事件，它也会尝试根据beanName直接从beanFactory容器中获取对应的ApplicationListener对象，如果能获取到，它也会调用它的onApplicationEvent()方法，从而消费这个阶段发布的事件
    void addApplicationListenerBean(String listenerBeanName);

    // 从广播器中移除一个ApplicationListener监听器
    void removeApplicationListener(ApplicationListener<?> listener);

    // 从广播器中移除一个ApplicationListener的bean权限定名
    void removeApplicationListenerBean(String listenerBeanName);

    // 自定义移除ApplicationListener的逻辑
    void removeApplicationListeners(Predicate<ApplicationListener<?>> predicate);

    void removeApplicationListenerBeans(Predicate<String> predicate);

    // 移除所有的ApplicationListener监听器
    void removeAllListeners();

    // 广播事件，参数是一个具体的事件对象
    void multicastEvent(ApplicationEvent event);

    // 广播事件，参数是一个具体的事件对象和对应的事件类型
    void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType);
}

```
上面的 SimpleApplicationEventMulticaster 实际上就是ApplicationEventMulticaster接口的实现类。   
继承体系如下：
```java
// 抽象类AbstractApplicationEventMulticaster实现ApplicationEventMulticaster接口
public abstract class AbstractApplicationEventMulticaster implements ApplicationEventMulticaster, BeanClassLoaderAware, BeanFactoryAware
```
```java
// SimpleApplicationEventMulticaster 继承抽象类 AbstractApplicationEventMulticaster
public class SimpleApplicationEventMulticaster extends AbstractApplicationEventMulticaster
```
回到上面this.initialMulticaster.addApplicationListener(listener);中，点进源码如下：
```java

    // 简单点理解就是ApplicationEventMulticaster中存在一些容器（list，set等）。
    // 然后这些容器对象负责收集注册进来的ApplicationListener，以作后续使用
    public void addApplicationListener(ApplicationListener<?> listener) {
        AbstractApplicationEventMulticaster.DefaultListenerRetriever var2 = this.defaultRetriever;
        synchronized(this.defaultRetriever) {
            Object singletonTarget = AopProxyUtils.getSingletonTarget(listener);
            if (singletonTarget instanceof ApplicationListener) {
                this.defaultRetriever.applicationListeners.remove(singletonTarget);
            }

            this.defaultRetriever.applicationListeners.add(listener);
            this.retrieverCache.clear();
        }
    }
```

### 3.1.6 总结初始化SpringApplicationRunListeners
初始化SpringApplicationRunListeners：
```java
SpringApplicationRunListeners listeners = this.getRunListeners(args);
```
getRunListeners方法返回一个SpringApplicationRunListeners实例：
```java
  private SpringApplicationRunListeners getRunListeners(String[] args) {
        Class<?>[] types = new Class[]{SpringApplication.class, String[].class};
        return new SpringApplicationRunListeners(logger, this.getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args), this.applicationStartup);
    }
```
（1）getRunListeners方法返回一个SpringApplicationRunListeners实例对象。SpringApplicationRunListeners对象是springboot容器的。      
（2）SpringApplicationRunListeners对象构造器初始化了一堆SpringApplicationRunListener运行监听器集合，从spring.factories中反射初始化实现类的，实际上只有一个EventPublishingRunListener实现类。   
（3）反射实例化EventPublishingRunListener对象时，其构造器中创建了一个spring时期的广播器SimpleApplicationEventMulticaster，并且把springboot容器中注册的ApplicationListener事件监听器集注册到了广播器中。   
所以，这一通操作下来，核心的目的就是将前面注册的监听器集合注册到了广播器中了。   

## 3.2 使用SpringApplicationRunListeners
初始化完SpringApplicationRunListeners对象后，就需要在合适的时机使用它。   
<b><u><font color="#dc143c">重点关注3个易混淆的对象：</font>  </u></b>      
（1）SpringApplicationRunListeners：springboot中定义的一个类，其委托SpringApplicationRunListener运行监听器集合干活。   
（2）SpringApplicationRunListener：springboot中定义的一个运行监听器钩子，只有一个实现类EventPublishingRunListener，核心功能就是在springboot的不同生命周期阶段广播不同的事件，但它自己不负责广播，而是委托spring时期的广播器SimpleApplicationEventMulticaster进行广播。   
（3）ApplicationListener：spring中定义的一个事件监听器钩子，当指定的事件被发布后，将会回调执行该监听器逻辑。SimpleApplicationEventMulticaster持有这些监听器对象，并在广播事件后根据事件类型回调对应的监听器逻辑。   

### 3.2.1 SpringApplicationRunListeners源码
```java
// 属于springboot
package org.springframework.boot;

import java.time.Duration;
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.function.Consumer;
import org.apache.commons.logging.Log;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.metrics.ApplicationStartup;
import org.springframework.core.metrics.StartupStep;
import org.springframework.util.ReflectionUtils;

class SpringApplicationRunListeners {
    private final Log log;
    private final List<SpringApplicationRunListener> listeners;
    private final ApplicationStartup applicationStartup;

    SpringApplicationRunListeners(Log log, Collection<? extends SpringApplicationRunListener> listeners, ApplicationStartup applicationStartup) {
        this.log = log;
        
        // 注册SpringApplicationRunListener运行监听器集，实际上只有一个对象EventPublishingRunListener。
        // 当然，也可自己扩展SpringApplicationRunListener，定制属于自己的运行监听器，也会在这里注册到SpringApplicationRunListeners中。
        this.listeners = new ArrayList(listeners);
        this.applicationStartup = applicationStartup;
    }

    // starting阶段，委托SpringApplicationRunListener的starting方法执行。
    void starting(ConfigurableBootstrapContext bootstrapContext, Class<?> mainApplicationClass) {
        this.doWithListeners("spring.boot.application.starting", (listener) -> {
            listener.starting(bootstrapContext);
        }, (step) -> {
            if (mainApplicationClass != null) {
                step.tag("mainApplicationClass", mainApplicationClass.getName());
            }

        });
    }

    // 委托SpringApplicationRunListener的environmentPrepared方法执行
    void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
        this.doWithListeners("spring.boot.application.environment-prepared", (listener) -> {
            listener.environmentPrepared(bootstrapContext, environment);
        });
    }

    // 委托SpringApplicationRunListener的contextPrepared方法执行
    void contextPrepared(ConfigurableApplicationContext context) {
        this.doWithListeners("spring.boot.application.context-prepared", (listener) -> {
            listener.contextPrepared(context);
        });
    }

    // 委托SpringApplicationRunListener的contextLoaded方法执行
    void contextLoaded(ConfigurableApplicationContext context) {
        this.doWithListeners("spring.boot.application.context-loaded", (listener) -> {
            listener.contextLoaded(context);
        });
    }

    // 委托SpringApplicationRunListener的started方法执行
    void started(ConfigurableApplicationContext context, Duration timeTaken) {
        this.doWithListeners("spring.boot.application.started", (listener) -> {
            listener.started(context, timeTaken);
        });
    }

    // 委托SpringApplicationRunListener的ready方法执行
    void ready(ConfigurableApplicationContext context, Duration timeTaken) {
        this.doWithListeners("spring.boot.application.ready", (listener) -> {
            listener.ready(context, timeTaken);
        });
    }

    // 委托SpringApplicationRunListener的failed方法执行
    void failed(ConfigurableApplicationContext context, Throwable exception) {
        this.doWithListeners("spring.boot.application.failed", (listener) -> {
            this.callFailedListener(listener, context, exception);
        }, (step) -> {
            step.tag("exception", exception.getClass().toString());
            step.tag("message", exception.getMessage());
        });
    }

    private void callFailedListener(SpringApplicationRunListener listener, ConfigurableApplicationContext context, Throwable exception) {
        try {
            listener.failed(context, exception);
        } catch (Throwable var6) {
            if (exception == null) {
                ReflectionUtils.rethrowRuntimeException(var6);
            }

            if (this.log.isDebugEnabled()) {
                this.log.error("Error handling failed", var6);
            } else {
                String message = var6.getMessage();
                message = message != null ? message : "no error message";
                this.log.warn("Error handling failed (" + message + ")");
            }
        }

    }

    private void doWithListeners(String stepName, Consumer<SpringApplicationRunListener> listenerAction) {
        this.doWithListeners(stepName, listenerAction, (Consumer)null);
    }

    private void doWithListeners(String stepName, Consumer<SpringApplicationRunListener> listenerAction, Consumer<StartupStep> stepAction) {
        StartupStep step = this.applicationStartup.start(stepName);
        this.listeners.forEach(listenerAction);
        if (stepAction != null) {
            stepAction.accept(step);
        }

        step.end();
    }
}
```
SpringApplicationRunListeners说白了就是一个包装，自己啥活儿都不干，全部委托SpringApplicationRunListener对象干活。   

### 3.2.2 SpringApplicationRunListener和其实现EventPublishingRunListener
SpringApplicationRunListener接口源码：
```java
// 归属于springboot
package org.springframework.boot;

import java.time.Duration;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.ConfigurableEnvironment;

public interface SpringApplicationRunListener {
    // 接收一个ConfigurableBootstrapContext对象，表示当前springboot容器正在启动
    default void starting(ConfigurableBootstrapContext bootstrapContext) {
    }

    // 表示环境已经准备完毕
    default void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
    }

    // 表示上下文已经准备完毕
    default void contextPrepared(ConfigurableApplicationContext context) {
    }

    // 表示上下文已经加载完毕
    default void contextLoaded(ConfigurableApplicationContext context) {
    }

    // 表示容器已经启动，不建议使用
    default void started(ConfigurableApplicationContext context, Duration timeTaken) {
        this.started(context);
    }

    /** @deprecated */
    @Deprecated
    default void started(ConfigurableApplicationContext context) {
    }

    // 表示springboot容器启动完成
    default void ready(ConfigurableApplicationContext context, Duration timeTaken) {
        this.running(context);
    }

    /** @deprecated */
    @Deprecated
    default void running(ConfigurableApplicationContext context) {
    }

    // 表示容器启动失败
    default void failed(ConfigurableApplicationContext context, Throwable exception) {
    }
}
```
主要还是要看实现类EventPublishingRunListener对上述运行阶段方法的实现逻辑：
```java
// 归属于springboot
package org.springframework.boot.context.event;

import java.time.Duration;
import java.util.Iterator;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.boot.ConfigurableBootstrapContext;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringApplicationRunListener;
import org.springframework.boot.availability.AvailabilityChangeEvent;
import org.springframework.boot.availability.LivenessState;
import org.springframework.boot.availability.ReadinessState;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.ApplicationListener;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.event.SimpleApplicationEventMulticaster;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.core.Ordered;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.util.ErrorHandler;

// 从类名可以看出，该运行监听器主要就是用来对springboot容器不同生命周期阶段制定不同的方法
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {
    private final SpringApplication application;
    private final String[] args;
    private final SimpleApplicationEventMulticaster initialMulticaster;

    public EventPublishingRunListener(SpringApplication application, String[] args) {
        this.application = application;
        this.args = args;
        this.initialMulticaster = new SimpleApplicationEventMulticaster();
        Iterator var3 = application.getListeners().iterator();

        while(var3.hasNext()) {
            ApplicationListener<?> listener = (ApplicationListener)var3.next();
            this.initialMulticaster.addApplicationListener(listener);
        }

    }

    public int getOrder() {
        return 0;
    }

    // 在容器启动阶段，委托广播器广播一个ApplicationStartingEvent事件
    public void starting(ConfigurableBootstrapContext bootstrapContext) {
        this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(bootstrapContext, this.application, this.args));
    }

    // 广播一个ApplicationEnvironmentPreparedEvent事件
    public void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
        this.initialMulticaster.multicastEvent(new ApplicationEnvironmentPreparedEvent(bootstrapContext, this.application, this.args, environment));
    }

    // 广播一个ApplicationContextInitializedEvent事件
    public void contextPrepared(ConfigurableApplicationContext context) {
        this.initialMulticaster.multicastEvent(new ApplicationContextInitializedEvent(this.application, this.args, context));
    }

    // 广播一个ApplicationPreparedEvent事件
    public void contextLoaded(ConfigurableApplicationContext context) {
        ApplicationListener listener;
        
        
        // 重点：注意这个for循环，里面实际上做了一件非常重要的事情，将springboot容器中的注册的所有监听器在这个生命周期阶段全部转移注册到spring上下文中了
        // 也就说，在这个逻辑中，对于springboot时期中已经注册的所有ApplicationListener集合，都转移注册到了spring容器上下文中了
        for(Iterator var2 = this.application.getListeners().iterator(); var2.hasNext(); context.addApplicationListener(listener)) {
            listener = (ApplicationListener)var2.next();
            if (listener instanceof ApplicationContextAware) {
                ((ApplicationContextAware)listener).setApplicationContext(context);
            }
        }

        this.initialMulticaster.multicastEvent(new ApplicationPreparedEvent(this.application, this.args, context));
    }

    // 重点：广播一个ApplicationStartedEvent事件
    public void started(ConfigurableApplicationContext context, Duration timeTaken) {
        context.publishEvent(new ApplicationStartedEvent(this.application, this.args, context, timeTaken));
        AvailabilityChangeEvent.publish(context, LivenessState.CORRECT);
    }

    // 重点：广播一个ApplicationReadyEvent事件
    public void ready(ConfigurableApplicationContext context, Duration timeTaken) {
        context.publishEvent(new ApplicationReadyEvent(this.application, this.args, context, timeTaken));
        AvailabilityChangeEvent.publish(context, ReadinessState.ACCEPTING_TRAFFIC);
    }

    // 重点：广播一个ApplicationFailedEvent事件
    public void failed(ConfigurableApplicationContext context, Throwable exception) {
        ApplicationFailedEvent event = new ApplicationFailedEvent(this.application, this.args, context, exception);
        if (context != null && context.isActive()) {
            context.publishEvent(event);
        } else {
            if (context instanceof AbstractApplicationContext) {
                Iterator var4 = ((AbstractApplicationContext)context).getApplicationListeners().iterator();

                while(var4.hasNext()) {
                    ApplicationListener<?> listener = (ApplicationListener)var4.next();
                    this.initialMulticaster.addApplicationListener(listener);
                }
            }

            this.initialMulticaster.setErrorHandler(new EventPublishingRunListener.LoggingErrorHandler());
            this.initialMulticaster.multicastEvent(event);
        }

    }

    private static class LoggingErrorHandler implements ErrorHandler {
        private static final Log logger = LogFactory.getLog(EventPublishingRunListener.class);

        private LoggingErrorHandler() {
        }

        public void handleError(Throwable throwable) {
            logger.warn("Error calling ApplicationEventListener", throwable);
        }
    }
}

```
### 3.2.3 总结SpringApplicationRunListeners对象的使用
通过上面的分析，我们总结下如何使用SpringApplicationRunListeners对象：   
（1）SpringApplicationRunListeners listeners = this.getRunListeners(args);产生了一个SpringApplicationRunListeners对象。   
（2）SpringApplicationRunListeners中封装了各个生命周期阶段的方法，每个方法自己不干活，而是委托持有的SpringApplicationRunListener对象干活。   
（3）SpringApplicationRunListener的实现类EventPublishingRunListener是具体干活的，但是它自己也不干活，而是委托事件广播器SimpleApplicationEventMulticaster负责干活，广播对应的事件出去。   

<b><u><font color="#dc143c">从 EventPublishingRunListener 源码中可以看出，在广播事件时，存在两种方式：</font>  </u></b>   
（1）通过SpringApplicationRunListener自己初始化的SimpleApplicationEventMulticaster广播器直接广播对应的事件出去，比如starting方法：  
```java
    public void starting(ConfigurableBootstrapContext bootstrapContext) {
        this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(bootstrapContext, this.application, this.args));
    }
```
（2）如果当前生命周期阶段已经拿到了spring容器上下文，则通过spring容器上下文进行事件的发布，比如started方法：
```java
    public void started(ConfigurableApplicationContext context, Duration timeTaken) {
        context.publishEvent(new ApplicationStartedEvent(this.application, this.args, context, timeTaken));
        AvailabilityChangeEvent.publish(context, LivenessState.CORRECT);
    }
```
这两种方式有什么区别呢？ 我们跟进它们的源码就可以知道了。       

## <a id="3.3">3.3 spring发布事件的底层实现</a>
纵观spring事件驱动模型中，其前期注册了对应的事件监听器，然后当对应的事件发生后，回调执行对应的事件监听器逻辑，从而达到事件监听的效果。   
这种事件驱动模型实际上是“发布-订阅”模式的一种具体实现，从具体实现细节上来说，它更是对“观察者模式”的一种实现方案。   
在容器的生命周期当中，通过注册不同的事件监听器充当观察者（事件监听器）；   
容器本身充当被观察者（事件源）；   
当容器某个生命周期发布了事件，则主动回调对应的事件监听器，将事件源本身的信息传递给事件监听器执行，从而达到主动通知事件监听感知事件的能力（事件）。   
所以，要触发对应事件监听器的执行，达到“监听”的目的，最主要的是要<b><u><font color="#dc143c">发布事件</font>  </u></b>。   
 
### <a id="3.3.1">3.3.1 SimpleApplicationEventMulticaster广播器发布事件</a>
```java
    public void starting(ConfigurableBootstrapContext bootstrapContext) {
        // 封装了对应的事件ApplicationStartingEvent，传入了bootstrapContext，SpringApplication对象和参数。
        this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(bootstrapContext, this.application, this.args));
    }
```
multicastEvent源码：
```java
    public void multicastEvent(ApplicationEvent event) {
        // resolveDefaultEventType方法：根据事件获取事件对应的类型
        this.multicastEvent(event, this.resolveDefaultEventType(event));
    }
```
点击进去如下：
```java
    public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
        // 获取事件对应的类型
        ResolvableType type = eventType != null ? eventType : this.resolveDefaultEventType(event);
        
        // 获取是否有异步线程执行器
        Executor executor = this.getTaskExecutor();
        
        // 关键：根据事件和事件类型，获取当前订阅该事件的监听器（即根据事件确定哪个监听器绑定了该事件，表示这个监听器对这个事件感兴趣）
        // 就是从前面注册的所有 ApplicationListener 集合中，获取订阅了当前事件的监听器
        Iterator var5 = this.getApplicationListeners(event, type).iterator();

        // 遍历回调执行对当前事件感兴趣的监听器逻辑
        while(var5.hasNext()) {
            ApplicationListener<?> listener = (ApplicationListener)var5.next();
            // 如果存在异步线程池执行器，则异步回调执行
            if (executor != null) {
                executor.execute(() -> {
                    this.invokeListener(listener, event);
                });
            } else {
                // 否则，直接同步回调执行
                this.invokeListener(listener, event);
            }
        }
    }
```
invokeListener方法源码：
```java
    protected void invokeListener(ApplicationListener<?> listener, ApplicationEvent event) {
        ErrorHandler errorHandler = this.getErrorHandler();
        if (errorHandler != null) {
            try {
                // 核心方法：传入监听器和事件，调用 doInvokeListener 干活
                this.doInvokeListener(listener, event);
            } catch (Throwable var5) {
                errorHandler.handleError(var5);
            }
        } else {
            this.doInvokeListener(listener, event);
        }
    }
```
doInvokeListener方法源码：
```java
private void doInvokeListener(ApplicationListener listener, ApplicationEvent event) {
        try {
            // 核心功能：就是传入了事件对象，执行了监听器的onApplicationEvent方法。
            listener.onApplicationEvent(event);
        } catch (ClassCastException var6) {
            String msg = var6.getMessage();
            if (msg != null && !this.matchesClassCastMessage(msg, event.getClass()) && (!(event instanceof PayloadApplicationEvent) || !this.matchesClassCastMessage(msg, ((PayloadApplicationEvent)event).getPayload().getClass()))) {
                throw var6;
            }

            Log loggerToUse = this.lazyLogger;
            if (loggerToUse == null) {
                loggerToUse = LogFactory.getLog(this.getClass());
                this.lazyLogger = loggerToUse;
            }

            if (loggerToUse.isTraceEnabled()) {
                loggerToUse.trace("Non-matching event type for listener: " + listener, var6);
            }
        }
    }
```
到这一步，已经逐步明了了：   
所谓的发布事件，实际上就是根据对应的事件对象去查找对应的监听器，然后触发监听器的回调方法执行。   
说白了就是主动触发客户端实现的监听器ApplicationListener的onApplicationEvent方法的执行。   
<b><u><font color="#dc143c">所以说，发布事件的本质就是服务方主动执行事件监听器的回调钩子。这也是我为什么一直强调，所谓监听器，其实就是一个服务方钩子接口，需要客户端定制回调逻辑。</font>  </u></b>。   

其实整个广播事件的逻辑里面，比较关键的逻辑是，如何根据事件和事件类型获取到对该事件感兴趣的监听器集合呢？   
Iterator var5 = this.getApplicationListeners(event, type).iterator();核心实现：
```java
    protected Collection<ApplicationListener<?>> getApplicationListeners(ApplicationEvent event, ResolvableType eventType) {
        // 先从缓存中获取
        Object source = event.getSource();
        Class<?> sourceType = source != null ? source.getClass() : null;
        AbstractApplicationEventMulticaster.ListenerCacheKey cacheKey = new AbstractApplicationEventMulticaster.ListenerCacheKey(eventType, sourceType);
        AbstractApplicationEventMulticaster.CachedListenerRetriever newRetriever = null;
        AbstractApplicationEventMulticaster.CachedListenerRetriever existingRetriever = (AbstractApplicationEventMulticaster.CachedListenerRetriever)this.retrieverCache.get(cacheKey);
        if (existingRetriever == null && (this.beanClassLoader == null || ClassUtils.isCacheSafe(event.getClass(), this.beanClassLoader) && (sourceType == null || ClassUtils.isCacheSafe(sourceType, this.beanClassLoader)))) {
            newRetriever = new AbstractApplicationEventMulticaster.CachedListenerRetriever();
            existingRetriever = (AbstractApplicationEventMulticaster.CachedListenerRetriever)this.retrieverCache.putIfAbsent(cacheKey, newRetriever);
            if (existingRetriever != null) {
                newRetriever = null;
            }
        }

        if (existingRetriever != null) {
            Collection<ApplicationListener<?>> result = existingRetriever.getApplicationListeners();
            if (result != null) {
                return result;
            }
        }

        // 缓存中不存在对当前事件感兴趣的监听器列表，调用retrieveApplicationListeners方法获取
        return this.retrieveApplicationListeners(eventType, sourceType, newRetriever);
    }
```
retrieveApplicationListeners源码：
```java
private Collection<ApplicationListener<?>> retrieveApplicationListeners(ResolvableType eventType, @Nullable Class<?> sourceType, @Nullable AbstractApplicationEventMulticaster.CachedListenerRetriever retriever) {
        List<ApplicationListener<?>> allListeners = new ArrayList();
        Set<ApplicationListener<?>> filteredListeners = retriever != null ? new LinkedHashSet() : null;
        Set<String> filteredListenerBeans = retriever != null ? new LinkedHashSet() : null;
        AbstractApplicationEventMulticaster.DefaultListenerRetriever var9 = this.defaultRetriever;
        LinkedHashSet listeners;
        LinkedHashSet listenerBeans;
        synchronized(this.defaultRetriever) {
            // 关键逻辑：获取当前广播器持有的applicationListeners和applicationListenerBeans
            listeners = new LinkedHashSet(this.defaultRetriever.applicationListeners);
            
            // 这个listenerBeans集合有什么存在的必要？
            // 别看它好像挺多余，其实一点都不多余。
            // 这个listenerBeans集合是在什么时候被搞到广播器对象中的呢？后面有分析
            // 剧透一下，它是在spring容器实例化好自己的广播器对象后，在registerListeners时期将spring容器中已经存在的ApplicationListener类型的bean名称给注册进来了
            // 那为啥不直接注册ApplicationListener的对象进来呢？因为在registerListeners阶段，spring容器中只有spring bean定义，还没有开始实例化bean呢。
            listenerBeans = new LinkedHashSet(this.defaultRetriever.applicationListenerBeans);
        }

        Iterator var15 = listeners.iterator();

        while(var15.hasNext()) {
            ApplicationListener<?> listener = (ApplicationListener)var15.next();
            // 遍历当前广播器持有的listener，判断其是否订阅了当前事件
            if (this.supportsEvent(listener, eventType, sourceType)) {
                if (retriever != null) {
                    filteredListeners.add(listener);
                }

                allListeners.add(listener);
            }
        }

        // 查找对某个时间感兴趣的所有监听器集合时，除了直接在已经注册到当前广播器中的ApplicationListener对象中找，还在当前广播器中已经注册的beanName集合中去找
        if (!listenerBeans.isEmpty()) {
            // 如果存在listenerBeans，listenerBeans本身是监听器的bean名称
            ConfigurableBeanFactory beanFactory = this.getBeanFactory();
            Iterator var17 = listenerBeans.iterator();

            while(var17.hasNext()) {
                String listenerBeanName = (String)var17.next();

                try {
                    // 如果beanFactory对当前事件感兴趣，哦？怎么是beanFactory对当前事件感兴趣呢？？？我们一会看它的源码
                    if (this.supportsEvent(beanFactory, listenerBeanName, eventType)) {
                        // 注意，这里直接通过beanFactory从spring容器中尝试获取ApplicationListener类型的listenerBeanName
                        // 如果容器中不存在，则会实例化对应的ApplicationListener bean。
                        ApplicationListener<?> listener = (ApplicationListener)beanFactory.getBean(listenerBeanName, ApplicationListener.class);
                        
                        // 如果allListeners不包含该监听器，且该监听器对目标事件感兴趣
                        if (!allListeners.contains(listener) && this.supportsEvent(listener, eventType, sourceType)) {
                            if (retriever != null) {
                                // 如果该beaname是单例的bean，则将该监听器收集到 filteredListeners 
                                if (beanFactory.isSingleton(listenerBeanName)) {
                                    filteredListeners.add(listener);
                                } else {
                                    // 否则，如果是多例的bean，则将该bean name收集到 filteredListenerBeans
                                    filteredListenerBeans.add(listenerBeanName);
                                }
                            }

                            allListeners.add(listener);
                        }
                    } else {
                        Object listener = beanFactory.getSingleton(listenerBeanName);
                        if (retriever != null) {
                            filteredListeners.remove(listener);
                        }

                        allListeners.remove(listener);
                    }
                } catch (NoSuchBeanDefinitionException var13) {
                    ;
                }
            }
        }

        // 对聚合后的监听器列表进行排序（Ordered接口，@Order等之类）
        AnnotationAwareOrderComparator.sort(allListeners);
        if (retriever != null) {
            if (filteredListenerBeans.isEmpty()) {
                retriever.applicationListeners = new LinkedHashSet(allListeners);
                retriever.applicationListenerBeans = filteredListenerBeans;
            } else {
                retriever.applicationListeners = filteredListeners;
                retriever.applicationListenerBeans = filteredListenerBeans;
            }
        }

        // 最终，每当传入一个事件，从当前广播器中获取对目标事件感兴趣的监听器时，都会从当前广播器中已经注册的ApplicationListener监听器集合和已经注册的ApplicationListener的beanName集合中，聚合出一个总的监听器列表返回
        // 返回后，广播器就会迭代执行这些监听器的 onApplicationEvent 方法了，从而达到消费事件的目的
        return allListeners;
    }
```

supportsEvent方法：这个方法有好几个重载，下面的是beanFactory作为参数的重载逻辑。   
```java
    private boolean supportsEvent(ConfigurableBeanFactory beanFactory, String listenerBeanName, ResolvableType eventType) {
        // 根据 listenerBeanName 获取到它的类型
        Class<?> listenerType = beanFactory.getType(listenerBeanName);
        
        // 如果类型不为null，且类型不是GenericApplicationListener，不是SmartApplicationListener的话
        if (listenerType != null && !GenericApplicationListener.class.isAssignableFrom(listenerType) && !SmartApplicationListener.class.isAssignableFrom(listenerType)) {
            // 如果该类型对目标事件不感兴趣，则返回false
            if (!this.supportsEvent(listenerType, eventType)) {
                return false;
            } else {
                
                // 否则，该类型对目标事件感兴趣，当该类型是ApplicationListener时，则返回true
                try {
                    BeanDefinition bd = beanFactory.getMergedBeanDefinition(listenerBeanName);
                    ResolvableType genericEventType = bd.getResolvableType().as(ApplicationListener.class).getGeneric(new int[0]);
                    return genericEventType == ResolvableType.NONE || genericEventType.isAssignableFrom(eventType);
                } catch (NoSuchBeanDefinitionException var7) {
                    // 请注意，如果该bean name在bean定义中不存在，也返回true。
                    return true;
                }
            }
        } else {
            // 否则，默认返回true
            return true;
        }
    }
```
上面的方法，实际上默认返回的就是true，它最大限度的认为该bean name对该事件感兴趣。   

### 3.3.2 spring上下文发布事件
参考started方法源码：
```java
    public void started(ConfigurableApplicationContext context, Duration timeTaken) {
        // 通过ConfigurableApplicationContext上下文对象直接发布一个事件
        context.publishEvent(new ApplicationStartedEvent(this.application, this.args, context, timeTaken));
        AvailabilityChangeEvent.publish(context, LivenessState.CORRECT);
    }
```
我们看一下publishEvent方法的定义：
```java
package org.springframework.context;

// ApplicationEventPublisher接口是spring时期的接口，顾名思义，这个接口就是用来发布事件的。
@FunctionalInterface
public interface ApplicationEventPublisher {
    default void publishEvent(ApplicationEvent event) {
        this.publishEvent((Object)event);
    }

    void publishEvent(Object event);
}
```
那spring容器上下文对象和ApplicationEventPublisher接口有什么关系呢？为什么上下文对象可以直接调用publishEvent方法？   
我们看一下spring容器上下文接口ApplicationContext的继承体系：
```java
// 顶层接口继承了ApplicationEventPublisher接口
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver 
```
因此，spring上下文对象实际上也是ApplicationEventPublisher接口的实现。   
也就是说spring上下文对象本身就是一个事件发布器。   

publishEvent源码：  
AbstractApplicationContext:
```java
    public void publishEvent(Object event) {
        this.publishEvent(event, (ResolvableType)null);
    }
```
点进去如下：
AbstractApplicationContext:
```java
protected void publishEvent(Object event, @Nullable ResolvableType eventType) {
        Assert.notNull(event, "Event must not be null");
        Object applicationEvent;
        if (event instanceof ApplicationEvent) {
            applicationEvent = (ApplicationEvent)event;
        } else {
            applicationEvent = new PayloadApplicationEvent(this, event);
            if (eventType == null) {
                eventType = ((PayloadApplicationEvent)applicationEvent).getResolvableType();
            }
        }

        if (this.earlyApplicationEvents != null) {
            this.earlyApplicationEvents.add(applicationEvent);
        } else {
            // 核心逻辑：它底层竟然也是委托事件广播器将事件广播出去的！！！！！
            this.getApplicationEventMulticaster().multicastEvent((ApplicationEvent)applicationEvent, eventType);
        }

        if (this.parent != null) {
            if (this.parent instanceof AbstractApplicationContext) {
                ((AbstractApplicationContext)this.parent).publishEvent(event, eventType);
            } else {
                this.parent.publishEvent(event);
            }
        }
    }
```
getApplicationEventMulticaster方法源码：
AbstractApplicationContext:
```java
    ApplicationEventMulticaster getApplicationEventMulticaster() throws IllegalStateException {
        // 直接从当前spring上下文容器中返回applicationEventMulticaster对象，如果不存在直接抛出异常
        if (this.applicationEventMulticaster == null) {
            throw new IllegalStateException("ApplicationEventMulticaster not initialized - call 'refresh' before multicasting events via the context: " + this);
        } else {
            return this.applicationEventMulticaster;
        }
    }
```
<b><u><font color="#dc143c">这里留几个疑问：</font>  </u></b>   
<b><u><font color="#dc143c">（1）AbstractApplicationContext上下文中的applicationEventMulticaster是啥时候初始化的呢？</font>  </u></b>   
<b><u><font color="#dc143c">（2）它和EventPublishingRunListener中实例化的SimpleApplicationEventMulticaster对象不是同一个对象吧？</font>  </u></b>   
<b><u><font color="#dc143c">（3）如果这里的广播器和EventPublishingRunListener中实例化的不是同一个对象，那么这两个广播器之间是如何同步本身注册的监听器的呢？还是说各自独立维护自身的监听器？</font>  </u></b>   

## 3.4 总结springboot中注册的ApplicationListener使用
整个流程分析下来，大概如下：   
（1）初始化一个SpringApplicationRunListeners对象，这个对象是对SpringApplicationRunListener运行监听器的包装；SpringApplicationRunListener的实现类EventPublishingRunListener在被构造时，从springboot中获取到了前期注册到springboot容器中的所有ApplicationListener监听器集合，并注册到了其自身（自身指的是springboot容器）负责实例化好的事件广播器SimpleApplicationEventMulticaster中。         
（2）调用SpringApplicationRunListeners的对应方法实际上是在委托EventPublishingRunListener中实例化好的广播器SimpleApplicationEventMulticaster，进行对应事件的广播。   
（3）也可以直接通过spring上下文进行事件的广播，底层也是委托SimpleApplicationEventMulticaster广播器进行事件的广播。   
（4）广播器广播事件的原理很简单，就是从当前广播器中注册的所有ApplicationListener监听器集合中找到订阅了当前事件的目标监听器，然后回调执行目标监听器的逻辑。      

核心就是两个步骤：      
（1）向一个事件广播器对象中注册一堆事件监听器（你只管向一个广播器中注册一堆你想要注册的事件监听器，不用管它什么时候被触发执行，随便注册）；      
（2）在某个生命周期阶段，通过该事件广播器对象发布指定事件，从该事件广播器中检索到订阅当前事件的监听器有哪些，然后遍历回调执行这些监听器的逻辑。      
<b><u><font color="#dc143c">这意味着，最终我们注册监听器，和进行事件广播的广播器，必须得是同一个广播器对象。</font>  </u></b>    
<b>你不能把事件监听器注册到了A广播器上，然后用B广播器去广播一个事件出去，这样肯定无法找到指定的监听器，因为B广播器广播一个事件出去，其自身根据找不到对应的监听器（监听器注册到A广播器上了），也就无法触发监听器的执行了。</b>   

# 4. springboot发布事件和spring发布事件
springboot说白了是在spring的基础之上套了一层外壳，它依赖spring提供的某些扩展机制实现自己的行为，因此springboot有自己的事件发布机制。    
同样的，当springboot拉起spring容器启动后，就进入到了spring容器的生命周期中，spring也有自己的事件发布机制。   
其实，前面我们已经分析过了，不论是springboot还是spring，发布事件的底层逻辑是完全一样的，都是委托一个事件广播器对象SimpleApplicationEventMulticaster将指定的事件广播出去。   
但这里我们要综合分析下，这两种容器阶段，对于事件的处理到底有什么异同点。    

## 4.1 springboot事件处理流程
准备好一个SpringApplicationRunListeners对象：
```java
SpringApplicationRunListeners listeners = this.getRunListeners(args);
```
执行该对象的方法：
```java
listeners.starting(bootstrapContext, this.mainApplicationClass);
// 省略
listeners.started(context, timeTakenToStartup);
```
这些方法最终都进入到EventPublishingRunListener的实现中：
```java
    public EventPublishingRunListener(SpringApplication application, String[] args) {
        this.application = application;
        this.args = args;
        
        // springboot自己负责创建的广播器
        this.initialMulticaster = new SimpleApplicationEventMulticaster();
        // 下面逻辑：将springboot时期的所有ApplicationListener对象都遍历注册到springboot自己的广播器中了
        Iterator var3 = application.getListeners().iterator();
        while(var3.hasNext()) {
            ApplicationListener<?> listener = (ApplicationListener)var3.next();
            this.initialMulticaster.addApplicationListener(listener);
        }
    }

    // springboot容器正在启动，发布一个事件，因为这个时候spring容器还没有refresh，因此只能使用springboot容器自己实例化的广播器去广播
    public void starting(ConfigurableBootstrapContext bootstrapContext) {
        this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(bootstrapContext, this.application, this.args));
    }
    // 省略，同上
    public void contextPrepared(ConfigurableApplicationContext context) {
        this.initialMulticaster.multicastEvent(new ApplicationContextInitializedEvent(this.application, this.args, context));
    }
    // 省略
```
这样一来，在springboot容器阶段，注册到springboot广播器中的所有监听器，就会被该广播器发布的指定事件触发执行。   

## 4.2 spring事件处理流程
同样在EventPublishingRunListener源码中，我们看到，有的事件是通过spring上下文发布的：
```java
    public void started(ConfigurableApplicationContext context, Duration timeTaken) {
        // 这里通过传入进来的spring context来发布事件，而不是使用springboot创建的广播器，很显然，要想使用spring上下文去发布事件，那这个阶段肯定是已经实例化了spring上下文了才行
        // 所以，简单点理解就是：当整个生命周期还没有实例化spring上下文对象时，springboot想发布事件，就只能使用自己实例化好的广播器去发布；
        // 当生命周期已经实例化了spring上下文对象后，发布事件就直接使用spring上下文去发布
        // 那spring上下文对象被实例化后，它的事件怎么来的呢？它能够转移springboot的广播器中的所有监听器到自己的身上吗？
        // 下面深入分析
        context.publishEvent(new ApplicationStartedEvent(this.application, this.args, context, timeTaken));
        AvailabilityChangeEvent.publish(context, LivenessState.CORRECT);
    }
```
让我们再次回到run方法：
```java
    public ConfigurableApplicationContext run(String... args) {
        // 省略...
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
        listeners.starting(bootstrapContext, this.mainApplicationClass);

        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
        // 省略...
        context = this.createApplicationContext();
        this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
        this.refreshContext(context);
        this.afterRefresh(context, applicationArguments);
        // 省略...
        listeners.started(context, timeTakenToStartup);

        listeners.ready(context, timeTakenToReady);
        return context;
    }
```
在spring context对象初始化前，都是调用listeners的对应方法去发布事件，底层是直接通过EventPublishingRunListener中创建的广播器SimpleApplicationEventMulticaster去发布的。   
在spring context对象初始化后，在调用listeners的方法时，都主动传入了spring上下文对象进去，底层通过spring上下文对象去发布事件。  
回到EventPublishingRunListener的源码： 
```java
    // starting，environmentPrepared，contextPrepared，contextLoaded等都是springboot阶段的事件发布，直接通过initialMulticaster广播器负责发布事件
    public void starting(ConfigurableBootstrapContext bootstrapContext) {
        this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(bootstrapContext, this.application, this.args));
    }

    public void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
        this.initialMulticaster.multicastEvent(new ApplicationEnvironmentPreparedEvent(bootstrapContext, this.application, this.args, environment));
    }

    public void contextPrepared(ConfigurableApplicationContext context) {
        this.initialMulticaster.multicastEvent(new ApplicationContextInitializedEvent(this.application, this.args, context));
    }

    public void contextLoaded(ConfigurableApplicationContext context) {
        ApplicationListener listener;
        for(Iterator var2 = this.application.getListeners().iterator(); var2.hasNext(); context.addApplicationListener(listener)) {
            listener = (ApplicationListener)var2.next();
            if (listener instanceof ApplicationContextAware) {
                ((ApplicationContextAware)listener).setApplicationContext(context);
            }
        }

        this.initialMulticaster.multicastEvent(new ApplicationPreparedEvent(this.application, this.args, context));
    }

    // started、ready等都是通过spring上下文来发布事件
    public void started(ConfigurableApplicationContext context, Duration timeTaken) {
        context.publishEvent(new ApplicationStartedEvent(this.application, this.args, context, timeTaken));
        AvailabilityChangeEvent.publish(context, LivenessState.CORRECT);
    }

    public void ready(ConfigurableApplicationContext context, Duration timeTaken) {
        context.publishEvent(new ApplicationReadyEvent(this.application, this.args, context, timeTaken));
        AvailabilityChangeEvent.publish(context, ReadinessState.ACCEPTING_TRAFFIC);
    }
```
详细发布原理参考： [3.3 spring发布事件的底层实现](#3.3)

### <a id="4.2.1">4.2.1 spring上下文的广播器是何时初始化的呢？</a>
也就是上面提到的问题之一：   
AbstractApplicationContext上下文中的applicationEventMulticaster是啥时候初始化的呢？   
run方法：
```java
 public ConfigurableApplicationContext run(String... args) {
        ConfigurableApplicationContext context = null;
        this.configureHeadlessProperty();
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
        listeners.starting(bootstrapContext, this.mainApplicationClass);

        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
        this.configureIgnoreBeanInfo(environment);
        Banner printedBanner = this.printBanner(environment);
        context = this.createApplicationContext();
        context.setApplicationStartup(this.applicationStartup);
        this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
        
        // refreshContext方法开始进入spring生命周期，开始启动spring容器
        this.refreshContext(context);
        this.afterRefresh(context, applicationArguments);
        Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
        if (this.logStartupInfo) {
            (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), timeTakenToStartup);
        }

        listeners.started(context, timeTakenToStartup);
    }
```
我们简化步骤，直接列出spring容器的refresh方法源码：
```java
public void refresh() throws BeansException, IllegalStateException {
        Object var1 = this.startupShutdownMonitor;
        synchronized(this.startupShutdownMonitor) {
            StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");
            
            
            // 这个方法里面，创建了早期监听器集合和早期事件集合
            this.prepareRefresh();
            
            
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            this.prepareBeanFactory(beanFactory);

            try {
                this.postProcessBeanFactory(beanFactory);
                StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
                this.invokeBeanFactoryPostProcessors(beanFactory);
                this.registerBeanPostProcessors(beanFactory);
                beanPostProcess.end();
                this.initMessageSource();
                
                // 核心：这个方法用来初始化spring上下文容器中的广播器
                this.initApplicationEventMulticaster();
                
                
                this.onRefresh();
                
                // 这个方法用来向spring上下文容器中注册事件监听器ApplicationListener集合
                this.registerListeners();
                
                
                this.finishBeanFactoryInitialization(beanFactory);
                this.finishRefresh();
            } catch (BeansException var10) {
                // 省略
            } finally {
            }

        }
    }
```
initApplicationEventMulticaster方法源码：
```java
    protected void initApplicationEventMulticaster() {
        // 如果beanFactory工厂中有广播器，则直接获取直接用
        ConfigurableListableBeanFactory beanFactory = this.getBeanFactory();
        if (beanFactory.containsLocalBean("applicationEventMulticaster")) {
            this.applicationEventMulticaster = (ApplicationEventMulticaster)beanFactory.getBean("applicationEventMulticaster", ApplicationEventMulticaster.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
            }
        } else {
            // 否则，自己new一个然后注册到beanFactory中
            this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
            beanFactory.registerSingleton("applicationEventMulticaster", this.applicationEventMulticaster);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No 'applicationEventMulticaster' bean, using [" + this.applicationEventMulticaster.getClass().getSimpleName() + "]");
            }
        }
    }
```
通过源码分析，spring容器阶段的广播器和springboot容器阶段的是完全不同的两个对象。   
它在spring容器实例化bean对象之前负责初始化一个全新的事件广播器。   

然后在spring上下文对象准备完成后，通过上下文去发布事件时，会直接获取上下文对象中创建的广播器对象来发布事件：
AbstractApplicationContext:
```java
    protected void publishEvent(Object event, @Nullable ResolvableType eventType) {
        Assert.notNull(event, "Event must not be null");
        // 省略
        // 如果存在早期事件，则对早期事件进行收集
        if (this.earlyApplicationEvents != null) {
            this.earlyApplicationEvents.add(applicationEvent);
        } else {
            // 获取当前spring容器中的事件广播器，就是上面在sprign容器refresh阶段初始化的那个广播器
            this.getApplicationEventMulticaster().multicastEvent((ApplicationEvent)applicationEvent, eventType);
        }
    }
```

### <a id="4.2.2">4.2.2 springboot如何同步自己的监听器给spring？</a>
```java
// 注意，listeners是springboot的SpringApplicationRunListeners对象
listeners.started(context, timeTakenToStartup);
```
底层调用的方法是：
```java
    public void started(ConfigurableApplicationContext context, Duration timeTaken) {
        // 通过spring上下文发布事件
        context.publishEvent(new ApplicationStartedEvent(this.application, this.args, context, timeTaken));
        AvailabilityChangeEvent.publish(context, LivenessState.CORRECT);
    }
```
最终spring容器发布事件逻辑落在如下代码上：
```java
// 通过spring容器自己的广播器进行事件的发布
this.getApplicationEventMulticaster().multicastEvent((ApplicationEvent)applicationEvent, eventType);
```
那么问题来了，springboot生命周期中注册的所有事件监听器，最后都注册到了springboot自己的EventPublishingRunListener中的广播器里面了。    
而spring容器中的广播器是自己重新创建的，它没有拥有springboot中注册的这些监听器，那么直接使用它的广播器去发布事件能生效吗？   
<b><u><font color="#dc143c">因此，肯定在通过spring容器发布事件前，有一个地方，将springboot中注册的这些监听器全部同步转移到了spring自己的广播器中了。</font>  </u></b>   

run方法：
```java
    context = this.createApplicationContext();

    // 准备上下文，在这个步骤里，将springboot中的所有监听器同步转移到了spring context中了。
    this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
    
    // 刷新上下文，在这个阶段，spring容器初始化自己的广播器，并向广播器中注册所有符合条件的事件监听器（包括从springboot中同步过来的）
    this.refreshContext(context);
    
    this.afterRefresh(context, applicationArguments);

    listeners.started(context, timeTakenToStartup);
```

prepareContext方法源码：
```java
 private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context, ConfigurableEnvironment environment, SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
        context.setEnvironment(environment);
        // 省略...
        Assert.notEmpty(sources, "Sources must not be empty");
        this.load(context, sources.toArray(new Object[0]));
        
        // 最后这一行，回调执行springboot的contextLoaded方法，表示上下文已经加载完毕
        listeners.contextLoaded(context);
    }
```

contextLoaded方法我们前面已经分析了多次，最终还是进入到了EventPublishingRunListener中：
```java
    public void contextLoaded(ConfigurableApplicationContext context) {
        ApplicationListener listener;
        
        // 这个for循环很关键了，它将springboot中的所有ApplicationListener监听器，都遍历注册到spring上下文中了
        for(Iterator var2 = this.application.getListeners().iterator(); var2.hasNext(); context.addApplicationListener(listener)) {
            listener = (ApplicationListener)var2.next();
            if (listener instanceof ApplicationContextAware) {
                ((ApplicationContextAware)listener).setApplicationContext(context);
            }
        }

        // 同步完springboot的所有监听器后，发布对应的事件
        this.initialMulticaster.multicastEvent(new ApplicationPreparedEvent(this.application, this.args, context));
    }
```

AbstractApplicationContext的addApplicationListener方法：
```java
    public void addApplicationListener(ApplicationListener<?> listener) {
        Assert.notNull(listener, "ApplicationListener must not be null");
        // 如果此时spring容器存在广播器，则同时注册到广播器中（很显然，现在广播器还没有创建）
        if (this.applicationEventMulticaster != null) {
            this.applicationEventMulticaster.addApplicationListener(listener);
        }
        
        // 关键： 将监听器注册到spring容器中
        this.applicationListeners.add(listener);
    }
```

到这里，spring容器中已经同步了springboot中的所有监听器了，它将springboot中的所有监听器拿过来，注册到自己的applicationListeners中了。   
那么什么时候注册到spring容器自己的广播器里面的呢？   
这个问题上面已经提了一笔，在refresh方法中：
```java
   beanPostProcess.end();
   this.initMessageSource();
   
   // 创建spring容器自己的广播器
   this.initApplicationEventMulticaster();
   
   
   this.onRefresh();
   
   
   // 向spring容器自己的广播器中注册所有符合条件的监听器
   this.registerListeners();
```

registerListeners方法源码：
```java
protected void registerListeners() {
        // 遍历spring容器中已经持有的监听器列表，这些列表来源两部分：
        // 1.是从springboot时期转移过来的（springboot转移的过程也是通过2的方式手动添加到spring上下文容器中的）
        // 2.通过context.addApplicationListener(new MyApplicationListener())进行手动添加的ApplicationListener
        Iterator var1 = this.getApplicationListeners().iterator();

        while(var1.hasNext()) {
            // 将这些监听器注册到spring自己的广播器中
            ApplicationListener<?> listener = (ApplicationListener)var1.next();
            this.getApplicationEventMulticaster().addApplicationListener(listener);
        }

        // 从spring容器中获取所有类型为ApplicationListener的bean name，这里获取的bean names不包括上面 getApplicationListeners() 方法获得的 ApplicationListener
        // 注意，这个之所以注册的是spring容器管理的ApplicationListener类型的bean name，而不是直接注册 ApplicationListener 的实例，是因为当前还没有实例
        // 当前只有 bean 定义
        // 在这里将bean name注册到spring容器自己的广播器中后，后续在通过spring上下文去广播事件时，如果该bean name订阅了目标事件，就会主动从spring容器中去创建该监听器，然后回调其逻辑
        // 这里正好说明：我们自定义的一个ApplicationListener的bean，只要交给spring容器进行管理，在这个阶段，它的bean nama就能注册到spring自己的广播器中。
        // 然后通过spring上下文发布指定事件，就能够回调我们自定义的监听器逻辑
        String[] listenerBeanNames = this.getBeanNamesForType(ApplicationListener.class, true, false);
        String[] var7 = listenerBeanNames;
        int var3 = listenerBeanNames.length;
        for(int var4 = 0; var4 < var3; ++var4) {
            String listenerBeanName = var7[var4];
            // 将这些符合条件的bean name注册到spring容器自己的广播器中
            this.getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
        }

        // 如果存在早期事件集合，则遍历发布对应的早期事件
        Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
        this.earlyApplicationEvents = null;
        if (!CollectionUtils.isEmpty(earlyEventsToProcess)) {
            Iterator var9 = earlyEventsToProcess.iterator();

            while(var9.hasNext()) {
                ApplicationEvent earlyEvent = (ApplicationEvent)var9.next();
                this.getApplicationEventMulticaster().multicastEvent(earlyEvent);
            }
        }

    }
```
上面的逻辑核心就干了一件事：将spring容器中已经持有的所有监听器注册到自己的广播器中，同时将beanFactory中所有类型为ApplicationListener的bean name注册到自己的广播器中。   
然后广播器在发布事件时，会根据事件类型，从自身持有的所有监听器对象和监听器bean name中找出对该事件感兴趣的ApplicationListener，聚合为一个整体监听器列表，进行事件的发布，触发对应监听器的执行。   
注意：对于spring广播器中注册的bean name，如果它对目标事件感兴趣，会强制从spring容器中实例化该bean实例，最后和已有的监听器聚合在一起，作为符合条件的监听器集合返回。   

<b><u><font color="#dc143c">这意味着，我们可以定义一个普通bean，只需要直接或者间接实现ApplicationListener接口，然后交给spring管理，就可以自定义一个事件监听器注册到spring容器中了。</font>  </u></b>。      
广播器如何广播事件 请参考： [3.3.1 SimpleApplicationEventMulticaster广播器发布事件](#3.3.1)     

总结一下：   
（1）springboot时期注册的监听器对象，一般通过spring.factories中进行注册，它注册到springboot自己的SpringApplication对象中。     
（2）SpringApplication对象中的监听器集合，通过EventPublishingRunListener的构造器逻辑，同步转移到了springboot自己的广播器中。   
（3）springboot通过 SpringApplicationRunListener 去发布自己的某些早期事件(在spring上下文产生之前的事件，比如starting事件)，底层是通过自己的广播器去发布事件的，当然有可能消费这些事件的，也只能是当前这个广播器中已经注册的监听器集合了。   
（4）随着spring上下文的产生，springboot执行prepareContext方法，在其中通过SpringApplicationRunListeners去发布contextLoaded事件，在这里将springboot中所有注册的监听器都转移到spring上下文中了。   
（5）随着spring容器的refresh执行，在initApplicationEventMulticaster方法中创建了spring自己的广播器。      
（6）在spring容器的registerListeners方法中，会将spring容器中自己已经持有的监听器全部转移到自己的广播器中（当然默认spring当前没有持有任何监听器，但是可以通过侵入式addApplicationListener方法手动注册一个），同时会从spring容器中获取所有类型为ApplicationListener的bean name，注册到spring自己的广播器中。至于为啥注册一个beanName，前面说过原因，在这个阶段，spring容器中只有bean定义，还没有产生bean实例。      
（7）最后就是广播事件，通过广播器去广播事件：如果是spring上下文产生之前的事件，则由springboot容器自己的广播器去广播；如果已经产生了spring上下文了，则由spring容器自己的广播器去广播（反正spring容器最终已经转移了springboot阶段所有的监听器了）   
    广播事件的逻辑很简单：   
    就是找出订阅了当前时间的所有监听器集合，然后统一执行它们的逻辑。   

# 5. ApplicationListenerDetector
其实监听器的注册、转移，广播器的实例化，事件的发布等，前面都已经讲完了。   
但是，ApplicationListenerDetector 又是什么鬼？   
## 5.1 为什么需要ApplicationListenerDetector？
前面分析：    
当spring容器实例化了自己的广播器后（initApplicationEventMulticaster方法中），通过执行registerListeners方法，将自己持有的所有监听器以及spring容器中此时存在的所有类型为ApplicationListener的bean name，分别注册到自己的广播器中。   
其中，已经持有的监听器集合会收集到广播器的：this.defaultRetriever.applicationListeners 字段中，其中元素就是ApplicationListener类型的实例对象；      
spring容器中所有类型为ApplicationListener的bean name会收集到广播器的：this.defaultRetriever.applicationListenerBeans 字段中，其中元素是ApplicationListener类型的bean name，是String类型。   

最后，当通过spring上下文去广播某个指定的事件时，最终会调用到广播器AbstractApplicationEventMulticaster的如下方法：   
```java
private Collection<ApplicationListener<?>> retrieveApplicationListeners(ResolvableType eventType, @Nullable Class<?> sourceType, @Nullable AbstractApplicationEventMulticaster.CachedListenerRetriever retriever) {
        List<ApplicationListener<?>> allListeners = new ArrayList();
        Set<ApplicationListener<?>> filteredListeners = retriever != null ? new LinkedHashSet() : null;
        Set<String> filteredListenerBeans = retriever != null ? new LinkedHashSet() : null;
        AbstractApplicationEventMulticaster.DefaultListenerRetriever var9 = this.defaultRetriever;
        LinkedHashSet listeners;
        LinkedHashSet listenerBeans;
        synchronized(this.defaultRetriever) {
            listeners = new LinkedHashSet(this.defaultRetriever.applicationListeners);
            listenerBeans = new LinkedHashSet(this.defaultRetriever.applicationListenerBeans);
        }

        // 1.如果listeners存在监听器集合（这些监听器集合就是在registerListeners方法中注册的），则直接收集广播器中对当前事件感兴趣的ApplicationListener对象集
        Iterator var15 = listeners.iterator();
        while(var15.hasNext()) {
            ApplicationListener<?> listener = (ApplicationListener)var15.next();
            if (this.supportsEvent(listener, eventType, sourceType)) {
                if (retriever != null) {
                    filteredListeners.add(listener);
                }

                allListeners.add(listener);
            }
        }

        // 2.如果listenerBeans存在监听器的bean name集合（这些监听器bean name集合也是在registerListeners方法中注册的），则直接收集广播器中对当前事件感兴趣的ApplicationListener bean name集
        // 收集完成后会判断该bean name是否是单例，如果是单例则直接从容器中获取该bean name对应的对象
        if (!listenerBeans.isEmpty()) {
            ConfigurableBeanFactory beanFactory = this.getBeanFactory();
            Iterator var17 = listenerBeans.iterator();

            while(var17.hasNext()) {
                String listenerBeanName = (String)var17.next();

                try {
                    if (this.supportsEvent(beanFactory, listenerBeanName, eventType)) {
                        ApplicationListener<?> listener = (ApplicationListener)beanFactory.getBean(listenerBeanName, ApplicationListener.class);
                        if (!allListeners.contains(listener) && this.supportsEvent(listener, eventType, sourceType)) {
                            if (retriever != null) {
                                if (beanFactory.isSingleton(listenerBeanName)) {
                                    filteredListeners.add(listener);
                                } else {
                                    filteredListenerBeans.add(listenerBeanName);
                                }
                            }

                            allListeners.add(listener);
                        }
                    } else {
                        Object listener = beanFactory.getSingleton(listenerBeanName);
                        if (retriever != null) {
                            filteredListeners.remove(listener);
                        }

                        allListeners.remove(listener);
                    }
                } catch (NoSuchBeanDefinitionException var13) {
                    ;
                }
            }
        }

        AnnotationAwareOrderComparator.sort(allListeners);
        if (retriever != null) {
            if (filteredListenerBeans.isEmpty()) {
                retriever.applicationListeners = new LinkedHashSet(allListeners);
                retriever.applicationListenerBeans = filteredListenerBeans;
            } else {
                retriever.applicationListeners = filteredListeners;
                retriever.applicationListenerBeans = filteredListenerBeans;
            }
        }

        return allListeners;
    }
```
重点分析上述方法的第2步。   
第2步主要逻辑如下：   
（1）当通过spring上下文发布某个事件时（底层是通过spring容器自己持有的广播器去发布事件的），它尝试获取spring广播器中持有的前期注册的所有ApplicationListener类型的bean name。   
（2）然后从spring容器中获取这个bean name对应的bean实例，注意，到底这个时候容器中有没有这个bean实例呢？这取决于该事件发布的时机：   
    如果该事件是在spring容器的registerListeners方法之前就发布的，那么根本不会执行到上述方法，因为此时还没有bean name被注册进去；   
    如果该事件是在spring容器的registerListeners方法之后，spring实例化容器中的ApplicationListener类型的bean之前发布，那么此时在这个就会主动触发spring容器去实例化所有bean name符合的ApplicationListener类型的bean对象；   
    如果该事件是在spring容器整体都启动完成后发布，此时所有的bean都交给spring容器实例化完成了，那么此时这个逻辑就直接从spring容器中获取到该bean name对应的ApplicationListener实例即可。   
（3）然后对这些bean name做判断，先判断已有的监听器对象集合中有没有这个bean name对应的bean，如果没有，再看他是否对目标事件感兴趣。    
（4）最后统一返回所有对目标事件感兴趣的ApplicationListener对象集，最终遍历回调其中的 onApplicationEvent 方法，达到消费目标事件的目的。   

上面的逻辑有没有什么缺点呢？   
最大的缺点就在于，如果我们是在spring容器启动完成后去发布一个目标事件的话，那么每次都会执行上面的方法，而每次在业务中发布一个事件，如果存在我们交给spring管理的自定义监听器，则每次发布事件都需要执行如下逻辑：      
```java
 if (!allListeners.contains(listener) && this.supportsEvent(listener, eventType, sourceType)) {
     // ApplicationListenerDetector存在的目的：主要就是将下面这段逻辑给提前了，在ApplicationListenerDetector中已经将spring中的bean注册到广播器的监听器集合中了
     // 因此，条件 !allListeners.contains(listener) 此时就不会满足，下面的逻辑就不会执行
     if (retriever != null) {
         if (beanFactory.isSingleton(listenerBeanName)) {
             filteredListeners.add(listener);
         } else {
             filteredListenerBeans.add(listenerBeanName);
         }
     }

     allListeners.add(listener);
 }
```
判断该bean name对应的bean实例，是不是单例，如果是单例，就将其添加到目标监听器列表中返回。    
如果我们交给spring容器管理的自定义监听器特别多，那在业务中每次发布一个事件，这段逻辑都得遍历执行。     
其实这段逻辑，可不可以提前呢？   
retrieveApplicationListeners方法的目的主要有两步：   
（1）第一步遍历广播器持有的监听器对象列表，遍历判断是否对目标事件感兴趣。   
（2）第二步遍历广播器持有的监听器bean name列表，这个列表就是我们交给spring容器管理的ApplicationListener类型的bean名称，遍历判断是否对目标事件感兴趣。   
最后统一返回对目标事件感兴趣的监听器对象集合。   

我们现在考虑一下，如果在执行这个方法之前，我们能够将我们交给spring管理的ApplicationListener类型的bean name对应的bean实例，就直接给他注册到spring广播器持有的监听器对象集中去，那么上面的第二步就不会执行了。    
spring容器中所有符合条件的ApplicationListener类型的单例监听器对象，此时都会在第一步就执行聚合完毕。   
此时 条件 !allListeners.contains(listener) 不满足，就不会再去挨个判断我们的bean是不是单例bean这一大堆逻辑。    

spring在3.x版本也意识到了这个问题，所以它新增了一个类，负责提前上面的逻辑，这个类就是 ApplicationListenerDetector。   
我们先设想一下，spring最有可能在什么阶段去执行这个逻辑呢？   
我们先把refresh方法源码贴在下面：   
```java
public void refresh() throws BeansException, IllegalStateException {
        Object var1 = this.startupShutdownMonitor;
        synchronized(this.startupShutdownMonitor) {
            StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");
            this.prepareRefresh();
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            this.prepareBeanFactory(beanFactory);

            try {
                this.postProcessBeanFactory(beanFactory);
                StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
                this.invokeBeanFactoryPostProcessors(beanFactory);
                this.registerBeanPostProcessors(beanFactory);
                beanPostProcess.end();
                this.initMessageSource();
                this.initApplicationEventMulticaster();
                this.onRefresh();
                
                // 只能在下面这个时期执行上面的逻辑
                this.registerListeners();
                this.finishBeanFactoryInitialization(beanFactory);
                
                
                this.finishRefresh();
            } catch (BeansException var10) {
                if (this.logger.isWarnEnabled()) {
                    this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var10);
                }

                this.destroyBeans();
                this.cancelRefresh(var10);
                throw var10;
            } finally {
                this.resetCommonCaches();
                contextRefresh.end();
            }

        }
    }
```
其实只能在registerListeners方法之后，在spring容器实例化所有bean之前执行。    
说白了，只能在finishBeanFactoryInitialization方法里面去想办法执行了。    
finishBeanFactoryInitialization方法就是spring容器开始根据所有的bean定义去实例化bean实例的过程。 而这个方法里面在遍历实例化容器中所有目标bean的过程中，大量使用了BeanPostProcessor，因此，我们肯定猜到spring也是通过创建一个新的BeanPostProcessor去做这个事情。    
逻辑也很简单：    
新增一个BeanPostProcessor，当实例化某个bean之后，执行该bean的初始化操作之后，去判断一下，这个bean是不是ApplicationListener类型，如果是，再判断这个bean是不是单例，如果是，就将这个bean注册到spring容器的广播器中。     
为什么必须在执行完该bean的初始化操作之后再执行呢？因为，我们注册到spring广播器中的ApplicationListener对象必须是已经加工完成的对象，即必须确保它的所有初始化逻辑都成功执行完毕才行，所以不能初始化之前做。       

## 5.2 ApplicationListenerDetector的实现
org.springframework.context.support.ApplicationListenerDetector   
```java
package org.springframework.context.support;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.beans.factory.config.DestructionAwareBeanPostProcessor;
import org.springframework.beans.factory.support.MergedBeanDefinitionPostProcessor;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ApplicationEventMulticaster;
import org.springframework.lang.Nullable;
import org.springframework.util.ObjectUtils;

// 该beanPostProcessor实现了 DestructionAwareBeanPostProcessor 接口和  MergedBeanDefinitionPostProcessor 接口
// 而 DestructionAwareBeanPostProcessor 继承了 BeanPostProcessor 接口
// MergedBeanDefinitionPostProcessor也继承了 BeanPostProcessor 接口
class ApplicationListenerDetector implements DestructionAwareBeanPostProcessor, MergedBeanDefinitionPostProcessor {
    private static final Log logger = LogFactory.getLog(ApplicationListenerDetector.class);
    private final transient AbstractApplicationContext applicationContext;
    private final transient Map<String, Boolean> singletonNames = new ConcurrentHashMap(256);

    // 构造器，直接接收spring上下文对象
    public ApplicationListenerDetector(AbstractApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }
    
    // 该方法是 MergedBeanDefinitionPostProcessor 接口的方法，它在合并bean定义之后调用
    // 此处的实现很简单，判断这个bean类型是不是ApplicationListener，如果是，则将beanName作为key，将该bean是否为单例作为value，缓存起来
    public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
        if (ApplicationListener.class.isAssignableFrom(beanType)) {
            this.singletonNames.put(beanName, beanDefinition.isSingleton());
        }

    }

    // 在bean初始化方法之前进行调用
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean;
    }

    // 在bean初始化方法执行之后立即调用，核心逻辑就在这里：
    // 判断目标bean是不是ApplicationListener类型，如果是，判断是不是单例，如果是，则将该bean实例注册到spring上下文中
    // 否则，会打印一个警告日志
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        if (bean instanceof ApplicationListener) {
            Boolean flag = (Boolean)this.singletonNames.get(beanName);
            if (Boolean.TRUE.equals(flag)) {
                this.applicationContext.addApplicationListener((ApplicationListener)bean);
            } else if (Boolean.FALSE.equals(flag)) {
                if (logger.isWarnEnabled() && !this.applicationContext.containsBean(beanName)) {
                    logger.warn("Inner bean '" + beanName + "' implements ApplicationListener interface but is not reachable for event multicasting by its containing ApplicationContext because it does not have singleton scope. Only top-level listener beans are allowed to be of non-singleton scope.");
                }

                this.singletonNames.remove(beanName);
            }
        }

        return bean;
    }

    // 目标bean销毁之前执行
    public void postProcessBeforeDestruction(Object bean, String beanName) {
        if (bean instanceof ApplicationListener) {
            try {
                ApplicationEventMulticaster multicaster = this.applicationContext.getApplicationEventMulticaster();
                multicaster.removeApplicationListener((ApplicationListener)bean);
                multicaster.removeApplicationListenerBean(beanName);
            } catch (IllegalStateException var4) {
                ;
            }
        }

    }

    public boolean requiresDestruction(Object bean) {
        return bean instanceof ApplicationListener;
    }

    public boolean equals(@Nullable Object other) {
        return this == other || other instanceof ApplicationListenerDetector && this.applicationContext == ((ApplicationListenerDetector)other).applicationContext;
    }

    public int hashCode() {
        return ObjectUtils.nullSafeHashCode(this.applicationContext);
    }
}

```   
注释已经写的很清晰了，在构造 ApplicationListenerDetector 对象时直接传入了spring上下文，就可以直接操作spring上下文了。    
那 ApplicationListenerDetector 是什么时候被注册到spring容器中的呢？    
其实我们已经有感觉了，它肯定是在spring注册beanPostProcessor时期被注册进去的：   
```java
    public static void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {
        String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);
        int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
        beanFactory.addBeanPostProcessor(new PostProcessorRegistrationDelegate.BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));
        List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList();
        List<BeanPostProcessor> internalPostProcessors = new ArrayList();
        List<String> orderedPostProcessorNames = new ArrayList();
        List<String> nonOrderedPostProcessorNames = new ArrayList();
        
        String[] var8 = postProcessorNames;
        int var9 = postProcessorNames.length;

        String ppName;
        BeanPostProcessor pp;
        for(int var10 = 0; var10 < var9; ++var10) {
            ppName = var8[var10];
            if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
                // 从spring容器中获取BeanPostProcessor对象，并放入到priorityOrderedPostProcessors中。
                pp = (BeanPostProcessor)beanFactory.getBean(ppName, BeanPostProcessor.class);
                priorityOrderedPostProcessors.add(pp);
                
                // 如果是内部的beanPostProcessor，则放入internalPostProcessors中。
                if (pp instanceof MergedBeanDefinitionPostProcessor) {
                    internalPostProcessors.add(pp);
                }
            } else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
                // 如果实现了Ordered接口，则放入到orderedPostProcessorNames中。
                orderedPostProcessorNames.add(ppName);
            } else {
                // 否则，放入到nonOrderedPostProcessorNames中。
                nonOrderedPostProcessorNames.add(ppName);
            }
        }

        // 根据order对priorityOrderedPostProcessors排序
        sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
        // 注册priorityOrderedPostProcessors到spring容器中。
        registerBeanPostProcessors(beanFactory, (List)priorityOrderedPostProcessors);
        List<BeanPostProcessor> orderedPostProcessors = new ArrayList(orderedPostProcessorNames.size());
        Iterator var14 = orderedPostProcessorNames.iterator();

        // 遍历缓存Ordered接口的beanPostProcessor处理器
        while(var14.hasNext()) {
            String ppName = (String)var14.next();
            BeanPostProcessor pp = (BeanPostProcessor)beanFactory.getBean(ppName, BeanPostProcessor.class);
            orderedPostProcessors.add(pp);
            if (pp instanceof MergedBeanDefinitionPostProcessor) {
                internalPostProcessors.add(pp);
            }
        }

        // 对orderedPostProcessors进行排序
        sortPostProcessors(orderedPostProcessors, beanFactory);
        // 注册 orderedPostProcessors 到spring容器中
        registerBeanPostProcessors(beanFactory, (List)orderedPostProcessors);
        
        // 普通的没有排序的BeanPostProcessor
        List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList(nonOrderedPostProcessorNames.size());
        Iterator var17 = nonOrderedPostProcessorNames.iterator();

        // 遍历缓存普通的beanPostProcessor处理器
        while(var17.hasNext()) {
            ppName = (String)var17.next();
            pp = (BeanPostProcessor)beanFactory.getBean(ppName, BeanPostProcessor.class);
            nonOrderedPostProcessors.add(pp);
            if (pp instanceof MergedBeanDefinitionPostProcessor) {
                internalPostProcessors.add(pp);
            }
        }
        // 注册普通的BeanPostProcessor，因此普通的BeanPostProcessor没有实现Ordered接口，因此没法指定排序，顺序就是获取处理器的随机顺序
        registerBeanPostProcessors(beanFactory, (List)nonOrderedPostProcessors);
        // 对spring内部的处理器 internalPostProcessors 进行排序
        sortPostProcessors(internalPostProcessors, beanFactory);
        // 注册 internalPostProcessors 到spring容器中
        registerBeanPostProcessors(beanFactory, (List)internalPostProcessors);
        
        // 核心：手动注册一个内部的beanPostProcessor - ApplicationListenerDetector
        beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
    }
```
```



# 6. 自定义spring事件模型
上面从源码角度分析了那么多，目的只有一个，那就是我们可以借助spring提供的事件监听机制，来自定义我们自己的事件模型。   
依赖spring的这套事件模型，我们需要实现如下：   
（1）自定义事件监听器，并想办法注册到spring的广播器中（监听器）。    
（2）自定义事件，并让感兴趣的监听器感知订阅到（事件）。   
（3）自定义事件源，在合适的时机发布指定的事件，从而触发监听器的回调执行（事件源）。      

## 5.1 自定义事件监听器
通过前面的分析，要想自定义一个监听器并同时注册到spring容器自身初始化的广播器中，我们猜测有如下方式可以实现：   
（1）
