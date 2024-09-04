---
layout:     post
title:      BeanPostProcessor
subtitle:   spring架构中最重要的钩子就是BeanPostProcessor
categories: [springboot专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 什么是BeanPostProcessor？
## 1.1 BeanPostProcessor是什么东西？
下面是 BeanPostProcessor 钩子接口的源码：   
```java

package org.springframework.beans.factory.config;

import org.springframework.beans.BeansException;
import org.springframework.lang.Nullable;

public interface BeanPostProcessor {

    // 这个方法名称的关键字是 before 这个单词
    // 这个方法在每一个普通bean的初始化方法执行之前进行调用
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    // 这个方法名称的关键字是 after 这个单词
    // 这个方法在每一个普通bean的初始化方法执行之后进行调用
    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```   
我们可以看到，BeanPostProcessor 钩子接口是属于spring的，而不是springboot的。      
post 表示在....之后，postProcessor的意思表示“后置处理器”，因此所有的BeanPostProcessor都统称为bean的后置处理器。      
所以，所有的BeanPostProcessor都表示 “在有了BeanDefinition之后” 的统一操作。   
这意味着，所有BeanPostProcessor都是在容器中已经加载完所有BeanDefinition信息之后才进行的；所有的BeanPostProcessor都是在遍历容器中的BeanDefinition，开始准备初始化目标bean的前后被触发执行的。       
Initialization表示初始化。      

## 1.2 BeanPostProcessor在bean生命周期的什么阶段进行介入？
我们回顾下spring控制bean的基本生命周期步骤如下：   
```youtrack
1.根据扫描范围或者配置的范围确认那些bean需要被spring容器进行管理
2.全局扫描这些bean，将这些bean的定义封装成对应的beanDefinition对象，全部注册到spring容器中
3.开始遍历这些beanDefinition对象，反射其构造方法按个实例化bean对象
4.反射出一个bean对象后，开始为这个bean填充属性，即确定这个bean依赖的属性值都来自于哪里，然后或者这些属性值，将其填充到这个bean的字段上
5.当一个bean的属性填充完毕后，开始执行这个bean的初始化方法，为这个bean做一些初始化动作，当然初始化逻辑中可以执行任意自己想插入的逻辑都行
6.当bean的初始化方法执行完毕，这个bean就完整的被创建出来了，随后会被注册到spring容器中缓存起来以供后续业务使用
7.创建完一个bean之后开始重复执行3-6步骤，将容器中所有的beanDefinition对象都挨个实例化、填充属性、初始化、然后注册到spring容器中
8.业务应用开始使用spring容器中注册好的完整bean对象
8.业务容器关闭时，开始遍历的销毁bean对象
..........

```
<font color="#dc143c"><b>从spring的bean生命周期可以看出来，BeanPostProcessor主要介入的是bean的初始化阶段，它在每一个bean的初始化阶段前后开始被回调。</b></font>         

## 1.3 为什么需要使用 BeanPostProcessor ？
有时候我们希望spring容器在创建bean对象的过程中，能够执行我们自定义的逻辑，对创建的bean做一些处理，或者执行一些业务（实际上就是拦截bean的创建过程，介入bean的生命周期）。   
实现这种回调逻辑的方案有很多种，比如自定义 bean 的初始化方法等，当bean对象被实例化（通过反射构造器进行实例化），并填充完属性后，紧接着就执行目标bean的初始化方法。      
而 BeanPostProcessor 钩子接口也能够实现类似的功能，并且更通用，因为它能够做到对所有的普通bean都统一插入自定义的逻辑，而不仅仅针对某一个bean。   

如果我们希望spring容器创建的每一个普通bean，在创建的过程中都能够执行一些我们自定义的逻辑，那么我们可以编写一个类，来实现BeanPostProcessor接口，然后将这个类注册到spring容器中。      
spring容器在启动时，会提前创建容器中类型为BeanPostProcessor的所有bean对象（因为我们自定义的类实现了BeanPostProcessor接口，因此所有实现了BeanPostProcessor接口的类，其类型都是BeanPostProcessor），然后在后续创建其他bean对象时，会将正在创建的每一个bean作为参数，遍历回调spring容器中注册的所有 BeanPostProcessor 中的方法。   
而 BeanPostProcessor 接口的方法体逻辑是由我们实现的。   
这样一来就实现了我们让spring容器创建任何bean对象的时候可以回调执行我们插入的自定义逻辑。   

从源码可以看出，BeanPostProcessor 钩子接口只定义了两个方法，而且还都是默认方法，这意味着这个钩子接口如果没有任何子类实现它以覆盖其默认方法的话，将采用接口中提供的默认实现逻辑，而默认实现逻辑就是原模原样返回正在创建的bean。    
并且这两个回调方法参数完全相同，都是spring容器正在创建的目标bean对象（此时bean对象刚创建完成）和目标bean的名称。   
```
bean: 容器正在创建的目标bean对象的引用地址；
beanName: 容器正在创建的目标bean的名称。
```
postProcessBeforeInitialization方法：顾名思义，在每一个bean的初始化方法之前调用;      
postProcessAfterInitialization方法：同理，在每一个bean的初始化方法之后调用。   
那bean的初始化方法是啥呢？   
spring容器创建bean大概分为如下步骤：   
（1）根据bean定义反射实例化bean对象；   
（2）反射调用bean对象的set方法为bean填充属性（如果反射构造器没填入属性的话），或者直接反射bean里面的属性成员，为其强制初始化赋值；   
（3）执行bean的初始化方法进行一些初始化逻辑。   

这里的关键就是执行bean的初始化方法，常见的有3种方式：   
（1）通过@PostConstruct注解标注一个void的非静态方法作为bean的初始化方法；   
（2）目标bean实现InitializingBean接口，实现其 afterPropertiesSet方法；   
（3）创建bean时指定initMethod（xml配置指定标签属性init-method，java配置在@Bean中指定initMethod）。   
上面3种初始化方法的执行优先级是：   
```
@PostConstruct -》 InitializingBean -》 initMethod   
```
而postProcessBeforeInitialization方法就是在这3个方法执行之前运行；   
postProcessAfterInitialization方法在这3个方法执行之后运行。   
如下图：   
![][beanPostProcessor的执行时机]  
上图中标红的两个地方就是BeanPostProcessor中两个方法的执行时机。   
spring容器在创建bean时，如果容器中包含了BeanPostProcessor的实现类对象，那么就会执行这个类的这两个方法，并将当前正在创建的bean的引用地址以及名称作为参数传递进方法中。   
也就是说，BeanPostProcessor的作用域是spring容器中的所有普通bean（不包括一些特殊的bean）。   
我们可以在一个容器中注册多个不同的 BeanPostProcessor 的实现类对象，而每一个普通bean在创建的过程中，都会遍历执行这些对象实现的before和after方法。   
那么自定义的beanPostProcessor的执行顺序如何确定呢？   
spring提供了一个接口Ordered，我们可以让自定义的BeanPostProcessor的实现类实现这个Ordered接口，并实现接口的 getOrder 方法。   
getOrder 方法返回一个Int类型的值，spring容器在启动时会根据order的返回值对多个BeanPostProcessor对象从小到大进行排序，然后在创建其他普通bean时依次遍历执行它们的方法。   
getOrder 方法的返回值越小的 BeanPostProcessor 对象，它所实现的方法将越先被执行。   

总的来说，spring源码中是这样实现的：   
先获取spring容器中挂载的所有beanDefinition；   
然后根据beanDefinition的依赖优先级开始遍历反射构造器，去实例化bean；   
实例化完成，遍历为每一个bean填充属性；   
当前bean属性填充完毕，遍历执行spring容器中挂载的所有beanPostProcessor后置处理器，执行其postProcessBeforeInitialization方法；   
执行当前bean的初始化方法，如果存在的话；   
遍历执行spring容器中挂载的所有beanPostProcessor后置处理器，执行其postProcessAfterInitialization方法；  

<b>spring框架中的很多技术都实现了BeanPostProcessor接口。比如@Autowired、@Value、@EJB等</b>

# 2. 使用BeanPostProcessor容易踩的坑
## 2.1 BeanPostProcessor实现类中依赖的其他bean，不会执行BeanPostProcessor中实现的方法
当我们在BeanPostProcessor的实现类中，依赖了其他的bean对象，那么被依赖的bean对象被创建时，将不会执行它所在的 BeanPostProcessor 实现类实现的方法。   
这其实很好理解，因为我们自定义的BeanPostProcessor依赖另外一个bean，这个bean需要在BeanPostProcessor创建之前就创建完成，因此它不会触发我们自定义的BeanPostProcessor中方法的调用。   
<b>但它会不会触发其他beanPostProcessor的实现类方法的调用，就不好说了，要看当前自定义BeanPostProcessor的实现类在这个beanPostProcessor集合中的优先级是怎样的。</b>   

## 2.1 BeanPostProcessor以及其依赖的bean无法使用AOP
因为Spring中的AOP本身就是作为一种BeanPostProcessor来实现的，也就是说为某个类或者某个接口，使用JDK或者cGlib的方式动态代理生成一个虚拟的class在内存中并反射该class对象，整个动态代理的过程，其实现就是通过一个BeanPostProcessor来完成的。   
所以，自定义的BeanPostProcessor   

# 3. 如何注册BeanPostProcessor？
主要就是两种方式：   
（1）自己编写一个普通类，去实现BeanPostProcessor接口，然后将该类交给spring容器管理（xml方式，各种注解等），这样spring容器在启动时就会优先实例化容器中所有类型为BeanPostProcessor的bean，这些对象自动就会注册到spring容器中了。      
（2）侵入式注册，使用addBeanPostProcessor()方法手动向spring容器中注册一个BeanPostProcessor。使用这种方式注册的BeanPostProcessor，Ordered接口将失效，会按照手动注册的顺序去排序BeanPostProcessor。   
<b>不论是哪种方式注册的BeanPostProcessor，都必须要清楚一点，我们注册的BeanPostProcessor对象的逻辑都会对spring容器中的所有普通bean进行处理。</b>   

# 4. @Bean注解注册BeanPostProcessor的注意点
使用@Bean去标注一个方法去返回一个自定义的BeanPostProcessor对象时，切记，方法返回值的类型必须是BeanPostProcessor类型或者是其子类型，而不能是Ordered类型。   
<b>并且，再扩展一下：@Bean注册的bean的类型，一定是它标注的方法的返回值类型。</b>      
如何理解呢？    
看一段rabbitMQ自动配置的代码：   
```java
        @Bean
        @ConditionalOnSingleCandidate(ConnectionFactory.class)
        @ConditionalOnProperty(
            prefix = "spring.rabbitmq",
            name = {"dynamic"},
            matchIfMissing = true
        )
        @ConditionalOnMissingBean
        // 注意，这个bean实例是RabbitAdmin类型
        // 但是方法的返回值是AmqpAdmin类型
        // 那么注册到容器中的 BeanDefinition 的这个bean的类型就是 AmqpAdmin 类型而不是 RabbitAdmin
        public AmqpAdmin amqpAdmin(ConnectionFactory connectionFactory) {
            return new RabbitAdmin(connectionFactory);
        }
```
通过@Bean注册的 BeanDefinition ，其中目标bean的类型一定是方法中定义的返回值的类型，而不是方法返回对象的实际子类型。    
这和其他方式注册bean是有区别的。   
其他方式是注册bean，都是直接拿到class对象，那么直接就能确定这个bean的类型。    
但是@Bean的bean，它的类型和bean的实例是分开的，类型通过方法的返回类型确定，实例通过方法体去构建。    
因此，如果尝试自动注入上面的RabbitAdmin实例，请注意：   
```java
    // 错误的注入      
    @Autowired
    private RabbitAdmin rabbitAdmin;
```
因为容器中没有RabbitAdmin类型的bean。   
你尝试通过 context.getBeansOfType(RabbitAdmin.class); 去获取RabbitAdmin类型的bean，发现容器中根本没有。    
下面才是正确的注入方式：   
```java
    @Autowired
    private AmqpAdmin amqpAdmin;
```
因为@Bean定义的 BeanDefinition 的类型实际山是 AmqpAdmin 类型，而不是 RabbitAdmin 类型。   
其他同理的操作，一定要注意。   

# 5. BeanPostProcessor 源码
## 5.1 BeanPostProcessor注册到spring容器中
```java
public class Code04ControllerApplication {
    public static void main(String[] args) {
        SpringApplication.run(Code04ControllerApplication.class, args);
    }
}
```
进入run方法：
```java
    public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
        return run(new Class[]{primarySource}, args);
    }
```
```java
    public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
        return (new SpringApplication(primarySources)).run(args);
    }
```
```java
    public ConfigurableApplicationContext run(String... args) {
        long startTime = System.nanoTime();
        DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();
        ConfigurableApplicationContext context = null;
        this.configureHeadlessProperty();
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
        listeners.starting(bootstrapContext, this.mainApplicationClass);

        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
            this.configureIgnoreBeanInfo(environment);
            Banner printedBanner = this.printBanner(environment);
            context = this.createApplicationContext();
            context.setApplicationStartup(this.applicationStartup);
            this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
            
            // 核心方法
            this.refreshContext(context);
            this.afterRefresh(context, applicationArguments);
            Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), timeTakenToStartup);
            }

            listeners.started(context, timeTakenToStartup);
            this.callRunners(context, applicationArguments);
        } catch (Throwable var12) {
            this.handleRunFailure(context, var12, listeners);
            throw new IllegalStateException(var12);
        }

        try {
            Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
            listeners.ready(context, timeTakenToReady);
            return context;
        } catch (Throwable var11) {
            this.handleRunFailure(context, var11, (SpringApplicationRunListeners)null);
            throw new IllegalStateException(var11);
        }
    }
```
进入refresh方法：
```java
    protected void refresh(ConfigurableApplicationContext applicationContext) {
        applicationContext.refresh();
    }
```
进入到spring容器的refresh方法：
```java
    public void refresh() throws BeansException, IllegalStateException {
        Object var1 = this.startupShutdownMonitor;
        synchronized(this.startupShutdownMonitor) {
            StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");
            this.prepareRefresh();
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            
            // 这个方法中注册了一些内置的BeanPostProcessor
            this.prepareBeanFactory(beanFactory);

            try {
                this.postProcessBeanFactory(beanFactory);
                StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
                this.invokeBeanFactoryPostProcessors(beanFactory);
                
                // 注册BeanPostProcessor的核心方法
                this.registerBeanPostProcessors(beanFactory);
                beanPostProcess.end();
                this.initMessageSource();
                this.initApplicationEventMulticaster();
                this.onRefresh();
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
获取容器中的beanPostProcessor对象并排序：
```java
    public static void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {
        // 获取spring容器中所有类型为 BeanPostProcessor 的bean名称
        String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);
        // 计算出当前spring容器中的目标BeanPostProcessor后置处理器的总数。
        // 为什么要+1呢？是因为下面一行手动注册了一个内置处理器BeanPostProcessorChecker，它的个数也直接在这里进行统计了。
        int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
        // 手动增加BeanPostProcessorChecker处理器，用于日志记录和一些校验。
        beanFactory.addBeanPostProcessor(new PostProcessorRegistrationDelegate.BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));
        // 定义存放实现了priorityOrdered接口的处理器集合。
        List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList();
        // 定义存放spring内部实现的处理器集合。
        List<BeanPostProcessor> internalPostProcessors = new ArrayList();
        
        // 定义实现了Ordered接口的处理器的name集合。
        List<String> orderedPostProcessorNames = new ArrayList();
        // 定义没有实现Ordered接口的处理器的name集合。
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
        
        // 手动注册一个内部的beanPostProcessor - ApplicationListenerDetector
        beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
    }
```
## 5.2 BeanPostProcessor后置处理器的触发使用
前面分析了在spring容器启动时，何时注册了内部的BeanPostProcessor，何时又注册了容器中我们自定义的BeanPostProcessor等，那么当这些BeanPostProcessor注册到spring容器中后，我们又是何时使用这些处理器的逻辑的呢？   
从上面的内容我们大概知道，这些BeanPostProcessor的触发使用是在每一个普通bean的初始化方法的前后分别进行的。   

refresh方法中的finishBeanFactoryInitialization方法，如下：   
org.springframework.context.support.AbstractApplicationContext#refresh      
```java
    public void refresh() throws BeansException, IllegalStateException {
        Object var1 = this.startupShutdownMonitor;
        synchronized(this.startupShutdownMonitor) {

            try {
                this.initApplicationEventMulticaster();
                this.onRefresh();
                this.registerListeners();
                
                // 这个方法是开始遍历容器中beanDefinition按个去反射实例化单例bean的核心方法
                this.finishBeanFactoryInitialization(beanFactory);
                this.finishRefresh();
            } 
        }
    }
```   
org.springframework.context.support.AbstractApplicationContext#finishBeanFactoryInitialization：      
```java
    protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
        
        // 获取容器中所有类型为LoadTimeWeaverAware的bean name列表
        String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
        String[] var3 = weaverAwareNames;
        int var4 = weaverAwareNames.length;

        for(int var5 = 0; var5 < var4; ++var5) {
            String weaverAwareName = var3[var5];
            // 调用getBean方法实例化上面所有的 LoadTimeWeaverAware 类型的单例bean对象
            this.getBean(weaverAwareName);
        }
        
        // 正式进入根据容器中的所有beanDefinition去反射实例化出所有单例bean的核心逻辑
        beanFactory.preInstantiateSingletons();
    }
```
org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons：      
```java
public void preInstantiateSingletons() throws BeansException {
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Pre-instantiating singletons in " + this);
        }

        List<String> beanNames = new ArrayList(this.beanDefinitionNames);
        Iterator var2 = beanNames.iterator();

        while(true) {
            String beanName;
            Object bean;
            do {
                while(true) {
                    RootBeanDefinition bd;
                    do {
                        do {
                            do {
                                if (!var2.hasNext()) {
                                    var2 = beanNames.iterator();

                                    while(var2.hasNext()) {
                                        beanName = (String)var2.next();
                                        Object singletonInstance = this.getSingleton(beanName);
                                        if (singletonInstance instanceof SmartInitializingSingleton) {
                                            StartupStep smartInitialize = this.getApplicationStartup().start("spring.beans.smart-initialize").tag("beanName", beanName);
                                            SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton)singletonInstance;
                                            if (System.getSecurityManager() != null) {
                                                AccessController.doPrivileged(() -> {
                                                    smartSingleton.afterSingletonsInstantiated();
                                                    return null;
                                                }, this.getAccessControlContext());
                                            } else {
                                                smartSingleton.afterSingletonsInstantiated();
                                            }

                                            smartInitialize.end();
                                        }
                                    }

                                    return;
                                }

                                beanName = (String)var2.next();
                                bd = this.getMergedLocalBeanDefinition(beanName);
                            } while(bd.isAbstract());
                        } while(!bd.isSingleton());
                    } while(bd.isLazyInit());

                    if (this.isFactoryBean(beanName)) {
                        bean = this.getBean("&" + beanName);
                        break;
                    }
                    
                    // 核心：根据beanName开始调用getBean方法
                    this.getBean(beanName);
                }
            } while(!(bean instanceof FactoryBean));

            FactoryBean<?> factory = (FactoryBean)bean;
            boolean isEagerInit;
            if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                SmartFactoryBean var10000 = (SmartFactoryBean)factory;
                ((SmartFactoryBean)factory).getClass();
                isEagerInit = (Boolean)AccessController.doPrivileged(var10000::isEagerInit, this.getAccessControlContext());
            } else {
                isEagerInit = factory instanceof SmartFactoryBean && ((SmartFactoryBean)factory).isEagerInit();
            }

            if (isEagerInit) {
                this.getBean(beanName);
            }
        }
    }
```

org.springframework.beans.factory.support.AbstractBeanFactory#getBean(java.lang.String)：
```java
    public Object getBean(String name) throws BeansException {
        this.assertBeanFactoryActive();
        return this.getBeanFactory().getBean(name);
    }
```
org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean：
```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) throws BeansException {
        String beanName = this.transformedBeanName(name);
        Object sharedInstance = this.getSingleton(beanName);
        Object beanInstance;
        if (sharedInstance != null && args == null) {
            if (this.logger.isTraceEnabled()) {
                if (this.isSingletonCurrentlyInCreation(beanName)) {
                    this.logger.trace("Returning eagerly cached instance of singleton bean '" + beanName + "' that is not fully initialized yet - a consequence of a circular reference");
                } else {
                    this.logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
                }
            }

            beanInstance = this.getObjectForBeanInstance(sharedInstance, name, beanName, (RootBeanDefinition)null);
        } else {
            if (this.isPrototypeCurrentlyInCreation(beanName)) {
                throw new BeanCurrentlyInCreationException(beanName);
            }

            BeanFactory parentBeanFactory = this.getParentBeanFactory();
            if (parentBeanFactory != null && !this.containsBeanDefinition(beanName)) {
                String nameToLookup = this.originalBeanName(name);
                if (parentBeanFactory instanceof AbstractBeanFactory) {
                    return ((AbstractBeanFactory)parentBeanFactory).doGetBean(nameToLookup, requiredType, args, typeCheckOnly);
                }

                if (args != null) {
                    return parentBeanFactory.getBean(nameToLookup, args);
                }

                if (requiredType != null) {
                    return parentBeanFactory.getBean(nameToLookup, requiredType);
                }

                return parentBeanFactory.getBean(nameToLookup);
            }

            if (!typeCheckOnly) {
                this.markBeanAsCreated(beanName);
            }

            StartupStep beanCreation = this.applicationStartup.start("spring.beans.instantiate").tag("beanName", name);

            try {
                if (requiredType != null) {
                    beanCreation.tag("beanType", requiredType::toString);
                }

                RootBeanDefinition mbd = this.getMergedLocalBeanDefinition(beanName);
                this.checkMergedBeanDefinition(mbd, beanName, args);
                String[] dependsOn = mbd.getDependsOn();
                String[] var12;
                if (dependsOn != null) {
                    var12 = dependsOn;
                    int var13 = dependsOn.length;

                    for(int var14 = 0; var14 < var13; ++var14) {
                        String dep = var12[var14];
                        if (this.isDependent(beanName, dep)) {
                            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
                        }

                        this.registerDependentBean(dep, beanName);

                        try {
                            this.getBean(dep);
                        } catch (NoSuchBeanDefinitionException var31) {
                            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "'" + beanName + "' depends on missing bean '" + dep + "'", var31);
                        }
                    }
                }

                if (mbd.isSingleton()) {
                    sharedInstance = this.getSingleton(beanName, () -> {
                        try {
                            // 核心 createBean 方法
                            return this.createBean(beanName, mbd, args);
                        } catch (BeansException var5) {
                            this.destroySingleton(beanName);
                            throw var5;
                        }
                    });
                    beanInstance = this.getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
                } else if (mbd.isPrototype()) {
                    var12 = null;

                    Object prototypeInstance;
                    try {
                        this.beforePrototypeCreation(beanName);
                        prototypeInstance = this.createBean(beanName, mbd, args);
                    } finally {
                        this.afterPrototypeCreation(beanName);
                    }

                    beanInstance = this.getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
                } else {
                    String scopeName = mbd.getScope();
                    if (!StringUtils.hasLength(scopeName)) {
                        throw new IllegalStateException("No scope name defined for bean '" + beanName + "'");
                    }

                    Scope scope = (Scope)this.scopes.get(scopeName);
                    if (scope == null) {
                        throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
                    }

                    try {
                        Object scopedInstance = scope.get(beanName, () -> {
                            this.beforePrototypeCreation(beanName);

                            Object var4;
                            try {
                                var4 = this.createBean(beanName, mbd, args);
                            } finally {
                                this.afterPrototypeCreation(beanName);
                            }

                            return var4;
                        });
                        beanInstance = this.getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                    } catch (IllegalStateException var30) {
                        throw new ScopeNotActiveException(beanName, scopeName, var30);
                    }
                }
            } catch (BeansException var32) {
                beanCreation.tag("exception", var32.getClass().toString());
                beanCreation.tag("message", String.valueOf(var32.getMessage()));
                this.cleanupAfterBeanCreationFailure(beanName);
                throw var32;
            } finally {
                beanCreation.end();
            }
        }

        return this.adaptBeanInstance(name, beanInstance, requiredType);
    }
```
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])：      
```java
    protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Creating instance of bean '" + beanName + "'");
        }

        RootBeanDefinition mbdToUse = mbd;
        Class<?> resolvedClass = this.resolveBeanClass(mbd, beanName, new Class[0]);
        if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
            mbdToUse = new RootBeanDefinition(mbd);
            mbdToUse.setBeanClass(resolvedClass);
        }

        try {
            mbdToUse.prepareMethodOverrides();
        } catch (BeanDefinitionValidationException var9) {
            throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(), beanName, "Validation of method overrides failed", var9);
        }

        Object beanInstance;
        try {
            beanInstance = this.resolveBeforeInstantiation(beanName, mbdToUse);
            if (beanInstance != null) {
                return beanInstance;
            }
        } catch (Throwable var10) {
            throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName, "BeanPostProcessor before instantiation of bean failed", var10);
        }

        try {
            // doCreateBean 方法
            beanInstance = this.doCreateBean(beanName, mbdToUse, args);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Finished creating instance of bean '" + beanName + "'");
            }

            return beanInstance;
        } catch (ImplicitlyAppearedSingletonException | BeanCreationException var7) {
            throw var7;
        } catch (Throwable var8) {
            throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", var8);
        }
    }
```
doCreateBean：正式创建bean的核心逻辑
```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
        BeanWrapper instanceWrapper = null;
        if (mbd.isSingleton()) {
            instanceWrapper = (BeanWrapper)this.factoryBeanInstanceCache.remove(beanName);
        }

        if (instanceWrapper == null) {
            instanceWrapper = this.createBeanInstance(beanName, mbd, args);
        }

        Object bean = instanceWrapper.getWrappedInstance();
        Class<?> beanType = instanceWrapper.getWrappedClass();
        if (beanType != NullBean.class) {
            mbd.resolvedTargetType = beanType;
        }

        Object var7 = mbd.postProcessingLock;
        synchronized(mbd.postProcessingLock) {
            if (!mbd.postProcessed) {
                try {
                    // 在创建每一个bean之后，首先进行bean定义的合并
                    this.applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
                } catch (Throwable var17) {
                    throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Post-processing of merged bean definition failed", var17);
                }

                mbd.postProcessed = true;
            }
        }

        boolean earlySingletonExposure = mbd.isSingleton() && this.allowCircularReferences && this.isSingletonCurrentlyInCreation(beanName);
        if (earlySingletonExposure) {
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Eagerly caching bean '" + beanName + "' to allow for resolving potential circular references");
            }

            this.addSingletonFactory(beanName, () -> {
                return this.getEarlyBeanReference(beanName, mbd, bean);
            });
        }

        Object exposedObject = bean;

        try {
            
            // 为当前bean填充属性值
            this.populateBean(beanName, mbd, instanceWrapper);
            
            // 初始化bean：执行bean的初始化方法。
            // BeanPostProcessor的after处理器的触发就是在这个方法里面进行的。
            exposedObject = this.initializeBean(beanName, exposedObject, mbd);
        } catch (Throwable var18) {
            if (var18 instanceof BeanCreationException && beanName.equals(((BeanCreationException)var18).getBeanName())) {
                throw (BeanCreationException)var18;
            }

            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Initialization of bean failed", var18);
        }

        if (earlySingletonExposure) {
            Object earlySingletonReference = this.getSingleton(beanName, false);
            if (earlySingletonReference != null) {
                if (exposedObject == bean) {
                    exposedObject = earlySingletonReference;
                } else if (!this.allowRawInjectionDespiteWrapping && this.hasDependentBean(beanName)) {
                    String[] dependentBeans = this.getDependentBeans(beanName);
                    Set<String> actualDependentBeans = new LinkedHashSet(dependentBeans.length);
                    String[] var12 = dependentBeans;
                    int var13 = dependentBeans.length;

                    for(int var14 = 0; var14 < var13; ++var14) {
                        String dependentBean = var12[var14];
                        if (!this.removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                            actualDependentBeans.add(dependentBean);
                        }
                    }

                    if (!actualDependentBeans.isEmpty()) {
                        throw new BeanCurrentlyInCreationException(beanName, "Bean with name '" + beanName + "' has been injected into other beans [" + StringUtils.collectionToCommaDelimitedString(actualDependentBeans) + "] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.");
                    }
                }
            }
        }

        try {
            this.registerDisposableBeanIfNecessary(beanName, bean, mbd);
            return exposedObject;
        } catch (BeanDefinitionValidationException var16) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Invalid destruction signature", var16);
        }
    }
```
initializeBean方法：触发后置处理器方法的执行
```java
 protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
        if (System.getSecurityManager() != null) {
            AccessController.doPrivileged(() -> {
                this.invokeAwareMethods(beanName, bean);
                return null;
            }, this.getAccessControlContext());
        } else {
            this.invokeAwareMethods(beanName, bean);
        }

        Object wrappedBean = bean;
        if (mbd == null || !mbd.isSynthetic()) {
            // 执行BeanPostProcessor的前置方法
            wrappedBean = this.applyBeanPostProcessorsBeforeInitialization(bean, beanName);
        }

        try {
            // 执行初始化方法
            this.invokeInitMethods(beanName, wrappedBean, mbd);
        } catch (Throwable var6) {
            throw new BeanCreationException(mbd != null ? mbd.getResourceDescription() : null, beanName, "Invocation of init method failed", var6);
        }

        if (mbd == null || !mbd.isSynthetic()) {
            // 执行BeanPostProcessor的后置方法
            wrappedBean = this.applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
        }

        return wrappedBean;
    }
```
执行前置方法：applyBeanPostProcessorsBeforeInitialization
```java
    public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName) throws BeansException {
        Object result = existingBean;

        Object current;
        // 获取spring容器中注册的所有beanPostProcessor处理器，然后遍历执行其中的postProcessBeforeInitialization方法
        for(Iterator var4 = this.getBeanPostProcessors().iterator(); var4.hasNext(); result = current) {
            BeanPostProcessor processor = (BeanPostProcessor)var4.next();
            // 如果遇到某个BeanPostProcessor返回的bean是null，则直接跳出循环，后续的BeanPostProcessor不会执行，并将null直接返回。
            // 因此将从spring容器中获取一个null对象。
            current = processor.postProcessBeforeInitialization(result, beanName);
            if (current == null) {
                return result;
            }
        }

        return result;
    }
```
执行后置方法：applyBeanPostProcessorsAfterInitialization
```java
    public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName) throws BeansException {
        Object result = existingBean;

        Object current;
        // 获取spring容器中注册的所有beanPostProcessor处理器，然后遍历执行其中的 postProcessAfterInitialization 方法
        for(Iterator var4 = this.getBeanPostProcessors().iterator(); var4.hasNext(); result = current) {
            BeanPostProcessor processor = (BeanPostProcessor)var4.next();
            // 同理，返回null的话，则终止其他BeanPostProcessor的执行。并将Null返回给spring容器，因此得到的bean是一个null。
            current = processor.postProcessAfterInitialization(result, beanName);
            if (current == null) {
                return result;
            }
        }

        return result;
    }
```

  [beanPostProcessor的执行时机]: data:image/jpeg;base64,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
  
