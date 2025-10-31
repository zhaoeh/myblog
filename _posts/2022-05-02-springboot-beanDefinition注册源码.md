---
layout:     post
title:      springboot启动后如何注册BeanDefinition对象
subtitle:   只针对springboot的启动流程深入分析BeanDefinition的注册过程
categories: [springboot专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. springboot启动流程简单回顾
***springboot的启动阶段中，关于bean的管理，主要分为两个大阶段：***   
1.将指定范围内的class转换为BeanDefinition，并注册到spring容器。
-  扫描得到一批class，将其转换为对应的BeanDefinition对象，然后注册到spring容器中。

2.遍历容器中的所有BeanDefinition，进入bean的实例化阶段并组装bean之间的依赖关系。
-  等到容器中的所有期望管理的bean都被转换为BeanDefinition对象并注册到容器后，开始根据BeanDefinition去实例化对应的目标对象，同时组装目标对象之间的依赖关系等，正式进入bean的实例化、属性填充、初始化等阶段。      

请注意，上述两个阶段中，**阶段1整体执行完毕之后，即所有期望的BeanDefinition注册到容器之后，才会整体进入阶段2进行目标bean的实例化、属性填充、初始化等操作**。    
然而，在阶段1中，spring容器也会优先从容器中根据BeanDefinition，提前实例化一些内部的bean实例对象作为处理器，比如会优先实例化一部分processor和其他的组件等，提前实例化的目的是为了调用这些组件对象中的某些方法逻辑，替容器完成一些重要的事情。    

***关键在于如何理解getBean()方法***   
请注意，spring容器启动的任何阶段，只要通过beanFactory调用了getBean()方法，它实际上就是在触发spring去管理目标bean(实例化目标bean以及组装依赖关系)。    
它的逻辑如下：    
-  检查bean的scope，优先去scope对应的缓存容器中查询，如果有就返回
-  如果没有，则直接调用对应scope生成bean的逻辑，然后将其缓存起来，并返回
因此整个方法，实际上只要被调用了，就是在触发spring容器负责实例化目标bean了！！！


***本文优先分析第一个大阶段，开始之前，先回顾一下springboot启动的大致流程。***

-  始于启动类，注意启动类是个main方法，上面标注了@SpringBootApplication注解。
-  run方法中传入了当前启动类的class对象，传入的这个对象在后续有至关重要的作用，这个class对象将作为springboot启动时扫描当前容器的入口。
    -  主要逻辑是解析启动类上的注解，请注意，spring框架底层在解析某个元素上面的注解时，往往都是解析出目标对象上的复合注解，即直接解析出目标元素上标注的所有注解，以及注解上递归标注的所有注解，简而言之，就是获取到目标元素的所有注解，从中检索出想要的注解（spring这一套解析复合注解的能力很强大，基本上等同于解决了java中注解不能够继承的限制，相当于一个注解可以继承另外一个注解，从而达到注解的嵌套以及属性的继承与覆盖）。
    -  @SpringBootApplication注解是个复合注解，其中很重要的一个注解是 @SpringBootConfiguration ，而 @SpringBootConfiguration 注解也是个复合注解，其中很重要的一个注解是 @Configuration。
    -  因此可以看出，run方法传入了当前启动类的class对象进去，后续会在容器启动过程中解析出这个class对象上的 @Configuration，以及其他所有注解，从而判断启动类是不是一个java配置类。
 -  @SpringBootApplication中还标注了@ComponentScan注解，主要引入了对应的Filter。   
 -  run方法的主体流程可以参考上一篇文章。核心阶段主要有如下：  
    -  this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
    -  this.refreshContext(context);
    -  this.afterRefresh(context, applicationArguments);
  
****关于@Configuration注解的理解****       
  -  @Configuration注解必须着重强调一下，它本身也是一个复合注解，它上面有标注@Component，就和@Controller、@Service注解一样，都是复合注解，这意味着配置类本身也是一个组件
  -  基于上面的原因，后面不要再往同一个类上面既标注了@Component，又标注了@Configuration，这种尽管可以，但是完全没有必要。
  -  只是通过@Configuration标注的组件明确声明为一个配置类，在其中通过@Bean注解声明bean的注册，才是最佳的实践，而且配置类的解析有独立的流程。
    
# 2.启动类作为class传入，做了什么事情？
## 2.1 进入springboot的prepareContext方法
```java
    private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context, ConfigurableEnvironment environment, SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
        context.setEnvironment(environment);
        this.postProcessApplicationContext(context);
        this.applyInitializers(context);
        listeners.contextPrepared(context);
        bootstrapContext.close(context);
        if (this.logStartupInfo) {
            this.logStartupInfo(context.getParent() == null);
            this.logStartupProfileInfo(context);
        }

        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
        if (printedBanner != null) {
            beanFactory.registerSingleton("springBootBanner", printedBanner);
        }

        if (beanFactory instanceof AbstractAutowireCapableBeanFactory) {
            ((AbstractAutowireCapableBeanFactory)beanFactory).setAllowCircularReferences(this.allowCircularReferences);
            if (beanFactory instanceof DefaultListableBeanFactory) {
                ((DefaultListableBeanFactory)beanFactory).setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
            }
        }

        if (this.lazyInitialization) {
            context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
        }


        // 关键在这一步：获取到构造器阶段注入的所有sources，作为一个Set集合，实际上此时只有启动类那个class对象
        Set<Object> sources = this.getAllSources();
        Assert.notEmpty(sources, "Sources must not be empty");
        // 将sources转换为数组，传入load方法
        this.load(context, sources.toArray(new Object[0]));
        listeners.contextLoaded(context);
    }
```    
## 2.2 load()方法
```java
    protected void load(ApplicationContext context, Object[] sources) {
        if (logger.isDebugEnabled()) {
            logger.debug("Loading source " + StringUtils.arrayToCommaDelimitedString(sources));
        }

        // 这一步很重要，传入了sources数组，构建了一个BeanDefinitionLoader对象
        // 注意，此时还是在springboot的SpringApplication类中
        // 查看源码，BeanDefinitionLoader也是springboot自己封装的一个类，该对象的主要作用就是用来加载BeanDefinition
        BeanDefinitionLoader loader = this.createBeanDefinitionLoader(this.getBeanDefinitionRegistry(context), sources);
        
        // 如果beanName生成器不对null，则设置到loader中
        if (this.beanNameGenerator != null) {
            loader.setBeanNameGenerator(this.beanNameGenerator);
        }

        // 同理
        if (this.resourceLoader != null) {
            loader.setResourceLoader(this.resourceLoader);
        }

        // 同理
        if (this.environment != null) {
            loader.setEnvironment(this.environment);
        }

        // 使用loader对象，进行load
        loader.load();
    }

```
这个方法是protected的，不对外部暴露。    
传入了sources数组，创建了一个BeanDefinitionLoader对象。     
调用BeanDefinitionLoader对象的load()方法。      
-  因此，想要清楚load()方法做了什么事情，先搞清楚createBeanDefinitionLoader()方法是如何创建BeanDefinitionLoader对象的。     

## 2.3 BeanDefinitionLoader对象的创建流程
```java
BeanDefinitionLoader loader = this.createBeanDefinitionLoader(this.getBeanDefinitionRegistry(context), sources);
```
this.getBeanDefinitionRegistry(context):   
```java
    // 传入ApplicationContext对象，根据多态判断类型，将其转换为BeanDefinitionRegistry注册器对象
    private BeanDefinitionRegistry getBeanDefinitionRegistry(ApplicationContext context) {
        if (context instanceof BeanDefinitionRegistry) {
            return (BeanDefinitionRegistry)context;
        } else if (context instanceof AbstractApplicationContext) {
            return (BeanDefinitionRegistry)((AbstractApplicationContext)context).getBeanFactory();
        } else {
            throw new IllegalStateException("Could not locate BeanDefinitionRegistry");
        }
    }
```
上面方法逻辑很简单，就是从ApplicationContext对象中抽象出对应的beanDefinition注册器：BeanDefinitionRegistry      

createBeanDefinitionLoader：
```java
    // 很简单的一个方法，受保护，不对外暴露
    // 直接调用BeanDefinitionLoader构造器，创建了一个BeanDefinitionLoader对象
    // 因此，核心逻辑应该在BeanDefinitionLoader的构造器中
    protected BeanDefinitionLoader createBeanDefinitionLoader(BeanDefinitionRegistry registry, Object[] sources) {
        return new BeanDefinitionLoader(registry, sources);
    }

```

BeanDefinitionLoader源码：
```java

package org.springframework.boot;

import groovy.lang.Closure;
import java.io.IOException;
import java.lang.reflect.Constructor;
import java.util.HashSet;
import java.util.Set;
import java.util.regex.Pattern;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.BeanDefinitionStoreException;
import org.springframework.beans.factory.groovy.GroovyBeanDefinitionReader;
import org.springframework.beans.factory.support.AbstractBeanDefinitionReader;
import org.springframework.beans.factory.support.BeanDefinitionReader;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanNameGenerator;
import org.springframework.beans.factory.xml.XmlBeanDefinitionReader;
import org.springframework.context.annotation.AnnotatedBeanDefinitionReader;
import org.springframework.context.annotation.ClassPathBeanDefinitionScanner;
import org.springframework.core.SpringProperties;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;
import org.springframework.core.type.filter.AbstractTypeHierarchyTraversingFilter;
import org.springframework.util.Assert;
import org.springframework.util.ClassUtils;
import org.springframework.util.ObjectUtils;
import org.springframework.util.StringUtils;

// 这个类是springboot自己封装的，并且访问级别是default，意味着不对包外暴露
class BeanDefinitionLoader {
    private static final boolean XML_ENABLED = !SpringProperties.getFlag("spring.xml.ignore");
    private static final Pattern GROOVY_CLOSURE_PATTERN = Pattern.compile(".*\\$_.*closure.*");
    private final Object[] sources;
    private final AnnotatedBeanDefinitionReader annotatedReader;
    private final AbstractBeanDefinitionReader xmlReader;
    private final BeanDefinitionReader groovyReader;
    private final ClassPathBeanDefinitionScanner scanner;
    private ResourceLoader resourceLoader;


    // 我们先重点观察构造器的逻辑
    BeanDefinitionLoader(BeanDefinitionRegistry registry, Object... sources) {
        // 对beanDefinition注册器和源sources进行校验
        Assert.notNull(registry, "Registry must not be null");
        Assert.notEmpty(sources, "Sources must not be empty");
        // 初始化
        this.sources = sources;
        
        // 这里比较重要了，创建了一个AnnotatedBeanDefinitionReader对象
        // 请注意，这个对象是spring framework中的，spring提供这个对象主要用来向容器中注册注解标注的beanDefinition
        // 说直白点，这个对象提供的大多数方法，都是直接传入一个class对象，然后向容器中注册对应的beanDefinition
        this.annotatedReader = new AnnotatedBeanDefinitionReader(registry);
        this.xmlReader = XML_ENABLED ? new XmlBeanDefinitionReader(registry) : null;
        this.groovyReader = this.isGroovyPresent() ? new GroovyBeanDefinitionReader(registry) : null;
        
        // 这个scanner是扫描器，这个方法也很重要，我们一步步分析
        this.scanner = new ClassPathBeanDefinitionScanner(registry);
        // 这是向扫描器中注册了一个排除过滤器，即指定了一些排除规则，符合排除规则的class将不会被解析为beanDefinition，当然更不会注册了
        this.scanner.addExcludeFilter(new BeanDefinitionLoader.ClassExcludeFilter(sources));
    }
    
    // .... 无用方法先删除

}
```

AnnotatedBeanDefinitionReader：
```java
    // 入口构造器
    public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {
        this(registry, getOrCreateEnvironment(registry));
    }

    // 实际被执行的构造器
    public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
       
        // 初始化beanName生成器
        this.beanNameGenerator = AnnotationBeanNameGenerator.INSTANCE;
        
        // 初始化作用域元数据解析器，负责解析scope注解
        this.scopeMetadataResolver = new AnnotationScopeMetadataResolver();
        Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
        Assert.notNull(environment, "Environment must not be null");
        
        // 绑定beanDefinition注册器
        this.registry = registry;
        
        // 初始化一个对候选BeanDefinition的过滤策略,ConditionEvaluator对象是一个条件评估器
        // 各种基于 @Conditional 注解扩展的条件注解，就是通过这个对象去解析的
        // 和其他属性一样，该对象在 AnnotatedBeanDefinitionReader 构造器中先被初始化，目的是用于后续通过它去评估BeanDefinition的条件是否达到注册的标准
        this.conditionEvaluator = new ConditionEvaluator(registry, environment, (ResourceLoader)null);
        
        // 调用了一个工具类  AnnotationConfigUtils，传入了注册器，向容器中优先注册了一堆Processors
        // 这个逻辑比较重要，我们深入看看
        AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
    }
```

AnnotationConfigUtils工具类的注册逻辑：
```java
    public static void registerAnnotationConfigProcessors(BeanDefinitionRegistry registry) {
        registerAnnotationConfigProcessors(registry, (Object)null);
    }

    public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(BeanDefinitionRegistry registry, @Nullable Object source) {
        // 将注册器转换为 默认的 beanFactory实例
        DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
        if (beanFactory != null) {
            if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
                beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
            }

            if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
                beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
            }
        }

        // 创建一个 BeanDefinitionHolder 集合
        // 请注意，BeanDefinitionHolder 对象用于持有 BeanDefinition 对象，对 BeanDefinition 再度包装做一些属性增强
        Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet(8);
        RootBeanDefinition def;
        
        
        // 我们心心念念的关键处理器 ConfigurationClassPostProcessor ，就是在这个被转换为 BeanDefinition 注册到spring容器中的
        if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalConfigurationAnnotationProcessor")) {
            def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
            def.setSource(source);
            // registerPostProcessor方法，将beanDefiniton和内置的beanName，注册到spring容器中去了
            // 然后将注册后的beanDefinition返回，缓存到beanDefs中
            // 可以看到，spring容器内部的一些内置bean，名称都是携带“internal”作为前缀的，表示是一个内部的bean定义
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalConfigurationAnnotationProcessor"));
        }

        // 下面的这些processor都很重要
        // 它们都实现了spring内部的一些钩子接口，在这个被优先注册到spring容器中
        // 然后在后续的阶段就会被优先实例化
        // 实例化后根据钩子接口的生命周期，在不同的阶段将会触发回调执行不同的方法
        // spring的核心逻辑几乎都是通过这些processor在不同的阶段进行处理的
        if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalAutowiredAnnotationProcessor")) {
            def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalAutowiredAnnotationProcessor"));
        }

        if (jsr250Present && !registry.containsBeanDefinition("org.springframework.context.annotation.internalCommonAnnotationProcessor")) {
            def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalCommonAnnotationProcessor"));
        }

        if (jpaPresent && !registry.containsBeanDefinition("org.springframework.context.annotation.internalPersistenceAnnotationProcessor")) {
            def = new RootBeanDefinition();

            try {
                def.setBeanClass(ClassUtils.forName("org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor", AnnotationConfigUtils.class.getClassLoader()));
            } catch (ClassNotFoundException var6) {
                throw new IllegalStateException("Cannot load optional framework class: org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor", var6);
            }

            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalPersistenceAnnotationProcessor"));
        }

        if (!registry.containsBeanDefinition("org.springframework.context.event.internalEventListenerProcessor")) {
            def = new RootBeanDefinition(EventListenerMethodProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.event.internalEventListenerProcessor"));
        }

        if (!registry.containsBeanDefinition("org.springframework.context.event.internalEventListenerFactory")) {
            def = new RootBeanDefinition(DefaultEventListenerFactory.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.event.internalEventListenerFactory"));
        }

        return beanDefs;
    }
```

registerPostProcessor方法：
```java
    // 这个方法就负责干一件事，将传入的beanName和BeanDefinition对象注册到spring容器中去
    private static BeanDefinitionHolder registerPostProcessor(BeanDefinitionRegistry registry, RootBeanDefinition definition, String beanName) {
        definition.setRole(2);
        registry.registerBeanDefinition(beanName, definition);
        return new BeanDefinitionHolder(definition, beanName);
    }
```

再看另外一个核心的对象 ClassPathBeanDefinitionScanner：
```java
    public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry) {
        // 注意第二个参数，为true，表示是否使用默认的类型过滤器
        this(registry, true);
    }

    public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters) {
        // useDefaultFilters 为true
        this(registry, useDefaultFilters, getOrCreateEnvironment(registry));
    }

    public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters, Environment environment) {
        // useDefaultFilters 为true
        this(registry, useDefaultFilters, environment, registry instanceof ResourceLoader ? (ResourceLoader)registry : null);
    }

    public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters, Environment environment, @Nullable ResourceLoader resourceLoader) {
       
       // 构造器中初始化了很多东西
        this.beanDefinitionDefaults = new BeanDefinitionDefaults();
        this.beanNameGenerator = AnnotationBeanNameGenerator.INSTANCE;
        this.scopeMetadataResolver = new AnnotationScopeMetadataResolver();
        this.includeAnnotationConfig = true;
        Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
        this.registry = registry;
        if (useDefaultFilters) {
            // 此时useDefaultFilters为true，因此执行registerDefaultFilters
            this.registerDefaultFilters();
        }

        this.setEnvironment(environment);
        this.setResourceLoader(resourceLoader);
    }
```

该类继承 ClassPathScanningCandidateComponentProvider 类：
```java
public class ClassPathBeanDefinitionScanner extends ClassPathScanningCandidateComponentProvider
```
ClassPathScanningCandidateComponentProvider，负责从classpath下真正提供扫描候选者组件的供应器。
这个父类中有一个方法 registerDefaultFilters：
```java
    protected void registerDefaultFilters() {
        // includeFilters属性是一个导入过滤器集合，它默认条件是一个基于注解的类型过滤器对象，基于 @Component 注解
        // 啥意思呢，后面会看到，在通过该扫描器扫描classpath下所有候选者时，判断一个候选者是不是期望的组件，就是通过此处的类型过滤器进行判断的
        // 其目的就是将classpath下，所有直接或者间接标注了 @Component 注解的候选者转换为BeanDefinition注册到spring容器中
        // 这意味着，当从@ComponentScan的包路径下遍历扫描组件时，只有直接或者间接标注了@Component注解的类才有资格被转换为BeanDefinition
        this.includeFilters.add(new AnnotationTypeFilter(Component.class));
        ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();


        // 默认还通过反射，尝试添加另外两个类型导入器，即如果classpath下存在 javax.annotation.ManagedBean，或者 javax.inject.Named 注解
        // 则认为直接或者间接标注这两个注解的候选者也是期望的组件
        try {
            this.includeFilters.add(new AnnotationTypeFilter(ClassUtils.forName("javax.annotation.ManagedBean", cl), false));
            this.logger.trace("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
        } catch (ClassNotFoundException var4) {
            ;
        }

        try {
            this.includeFilters.add(new AnnotationTypeFilter(ClassUtils.forName("javax.inject.Named", cl), false));
            this.logger.trace("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
        } catch (ClassNotFoundException var3) {
            ;
        }

    }
```
****includeFilters属性在后续扫描候选者的时候，会用到，用来判断一个候选者是不是spring期望的一个组件，即有没有直接或者间接的标注@Component注解****

ClassPathBeanDefinitionScanner ，主要负责扫描classpath下的所有BeanDefinition，它有一个特别重要的方法：
```java

    // 传入一堆 package路径，然后执行 doScan方法
    public int scan(String... basePackages) {
        int beanCountAtScanStart = this.registry.getBeanDefinitionCount();
        this.doScan(basePackages);
        
        // 如果包含注解配置，则执行registerAnnotationConfigProcessors
        // 这里等于是再次执行一次registerAnnotationConfigProcessors方法，进一步判断容器中是否有那些内部bean定义
        // 如果没有再次兜底注册
        if (this.includeAnnotationConfig) {
            AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
        }

        return this.registry.getBeanDefinitionCount() - beanCountAtScanStart;
    }


    // 这个doScan执行扫描逻辑
    // 核心逻辑就是：
    // 根据传入的一对 package 路径,循环扫描里面的所有class，找到@Component注解的class将其转换为BeanDefinition进行注册
    
    // 当然，这个方法在目前的流程中还没有被触发，我们后续等触发时再详细分析
    protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
        Assert.notEmpty(basePackages, "At least one base package must be specified");
        Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet();
        String[] var3 = basePackages;
        int var4 = basePackages.length;

        for(int var5 = 0; var5 < var4; ++var5) {
            String basePackage = var3[var5];
            Set<BeanDefinition> candidates = this.findCandidateComponents(basePackage);
            Iterator var8 = candidates.iterator();

            while(var8.hasNext()) {
                BeanDefinition candidate = (BeanDefinition)var8.next();
                ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
                candidate.setScope(scopeMetadata.getScopeName());
                String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
                if (candidate instanceof AbstractBeanDefinition) {
                    this.postProcessBeanDefinition((AbstractBeanDefinition)candidate, beanName);
                }

                if (candidate instanceof AnnotatedBeanDefinition) {
                    AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition)candidate);
                }

                if (this.checkCandidate(beanName, candidate)) {
                    BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
                    definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
                    beanDefinitions.add(definitionHolder);
                    this.registerBeanDefinition(definitionHolder, this.registry);
                }
            }
        }

        return beanDefinitions;
    }
```
doScan在目前的启动流程中还没有执行到，后面我们再详细分析！   

总结一下：
- 创建了一个springboot内部的 BeanDefinitionLoader
- BeanDefinitionLoader 构造器初始化了 AnnotatedBeanDefinitionReader 对象和 ClassPathBeanDefinitionScanner 对象。   
    - AnnotatedBeanDefinitionReader 对象负责向spring容器中注册了一堆内置的beanDefinition，其中包括ConfigurationClassPostProcessor。同时它提供向容器中根据class对象注册beanDefinition的能力。   
    - ClassPathBeanDefinitionScanner 对象负责扫描指定包路径，找到指定包中的@Component注解标注的class，将其转换为BeanDefinition注册到容器中。   
    - 同时scanner还注册了一些排除策略
 - 至此，BeanDefinitionLoader对象就创建完毕
 
## 2.4 BeanDefinitionLoader对象的触发流程
创建完 BeanDefinitionLoader 对象后，执行了 load() 方法。   
```java
loader.load();
```
进一步深入：
```java
    // 遍历 sources 数组，按个执行load方法
    // 此时， sources 数组实际上只有一个元素，就是启动类的class
    void load() {
        Object[] var1 = this.sources;
        int var2 = var1.length;

        for(int var3 = 0; var3 < var2; ++var3) {
            Object source = var1[var3];
            this.load(source);
        }

    }

    // 具体的load方法
    private void load(Object source) {
        Assert.notNull(source, "Source must not be null");
        if (source instanceof Class) {
            // class类型，走这个，我们默认就会走这个逻辑
            this.load((Class)source);
        } else if (source instanceof Resource) {
            // resource类型走这个
            this.load((Resource)source);
        } else if (source instanceof Package) {
            // package类型走这个
            this.load((Package)source);
        } else if (source instanceof CharSequence) {
            // 字符串类型走这个
            this.load((CharSequence)source);
        } else {
            throw new IllegalArgumentException("Invalid source type " + source.getClass());
        }
    }
```
重点关注class类型的load()方法流程：
```java
    private void load(Class<?> source) {
        if (this.isGroovyPresent() && BeanDefinitionLoader.GroovyBeanDefinitionSource.class.isAssignableFrom(source)) {
            BeanDefinitionLoader.GroovyBeanDefinitionSource loader = (BeanDefinitionLoader.GroovyBeanDefinitionSource)BeanUtils.instantiateClass(source, BeanDefinitionLoader.GroovyBeanDefinitionSource.class);
            ((GroovyBeanDefinitionReader)this.groovyReader).beans(loader.getBeans());
        }

        // 在对source进行注册之前，存在一个条件 isEligible,用于判断一个class是否满足某些条件
        if (this.isEligible(source)) {
            
            // 构造器阶段初始化的 AnnotatedBeanDefinitionReader 对象，在这个用到了
            this.annotatedReader.register(new Class[]{source});
        }

    }
```

源码如下：
```java
    private boolean isEligible(Class<?> type) {
        return !type.isAnonymousClass() && !this.isGroovyClosure(type) && !this.hasNoConstructors(type);
    }
```
***关于上述isEligible的详细分析***

这个 `isEligible` 方法用于检查一个类是否符合特定的条件。通过以下三个条件来判断 `type` 是否适合某种处理或操作：

1. **`!type.isAnonymousClass()`**:
   - 这个条件检查 `type` 是否是一个匿名内部类。匿名内部类在 Java 中通常是没有名称的类。通过这个检查，`isEligible` 方法排除了所有匿名内部类。

2. **`!this.isGroovyClosure(type)`**:
   - 这个条件检查 `type` 是否是 Groovy 的闭包类。Groovy 闭包类是特定于 Groovy 的概念，Spring 在处理 Groovy 闭包时可能需要特别的处理或忽略它们。这段代码通过这个检查排除了 Groovy 闭包。

3. **`!this.hasNoConstructors(type)`**:
   - 这个条件检查 `type` 是否没有构造函数。如果一个类没有任何构造函数，它可能无法被正确实例化或使用。通过这个检查，`isEligible` 方法排除了那些没有构造函数的类。

总结来说，这个 `isEligible` 方法的目的是确保某个类 `type` 符合特定的条件，从而决定是否可以进行某种操作或处理。具体来说，它排除了匿名内部类、Groovy 闭包类和没有构造函数的类。这个方法可能用于 Spring 框架中某些需要对类进行特定处理的场景。

一般我们的类，都是满足的。   

让我们再次深入spring的AnnotatedBeanDefinitionReader对象的register方法：
```java
    // 逻辑很简单，根据传入的一对class对象，然后遍历进行注册
    public void register(Class... componentClasses) {
        Class[] var2 = componentClasses;
        int var3 = componentClasses.length;

        for(int var4 = 0; var4 < var3; ++var4) {
            Class<?> componentClass = var2[var4];
            this.registerBean(componentClass);
        }

    }
```
底层实际上执行的是这个方法：
```java
    private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name, @Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier, @Nullable BeanDefinitionCustomizer[] customizers) {
        // 将class对象转换为一个普通的beanDefinition对象
        // 此时就是启动类被转换为beanDefinition对象了
        AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
        if (!this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
            // 很重要的 shouldSkip 方法：
            // 这个方法底层其实就是在解析每个源对象上的@Conditional注解，通过它去评估当前BeanDefinition是否应该被注册到spring容器中
            // 传入当前的beanDefinition对象，获取其中的元数据metadata，传递给条件评估器，去判断这个beanDefinition是否应该被注册到容器中
            // 只有当返回结果为“false”时，即表示不应该被跳过时，才进行最终的BeanDefinition的注册
            abd.setInstanceSupplier(supplier);
            
            // 通过 scopeMetadataResolver 解析器，解析当前beanDefinition的scope注解元信息
            ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
            // 设置 scope name
            abd.setScope(scopeMetadata.getScopeName());
            String beanName = name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry);
            AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
            
            // 处理 qualifiers 和 Primary 注解等，以及懒加载
            int var10;
            int var11;
            if (qualifiers != null) {
                Class[] var9 = qualifiers;
                var10 = qualifiers.length;

                for(var11 = 0; var11 < var10; ++var11) {
                    Class<? extends Annotation> qualifier = var9[var11];
                    if (Primary.class == qualifier) {
                        abd.setPrimary(true);
                    } else if (Lazy.class == qualifier) {
                        abd.setLazyInit(true);
                    } else {
                        abd.addQualifier(new AutowireCandidateQualifier(qualifier));
                    }
                }
            }

            if (customizers != null) {
                BeanDefinitionCustomizer[] var13 = customizers;
                var10 = customizers.length;

                for(var11 = 0; var11 < var10; ++var11) {
                    BeanDefinitionCustomizer customizer = var13[var11];
                    customizer.customize(abd);
                }
            }

            // 方法的最后，在这个将启动类class转换后的BeanDefinition对象和自动生成的beanName，包装成 BeanDefinitionHolder
            BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
            
            // 又是这个工具类：AnnotationConfigUtils
            // 处理作用域的代理模型，就是根据scope直接中的值，来判断当前的beanDefinition的代理模型是什么
            // 代理模型定义在 ScopedProxyMode 枚举中
            // 自定义作用域的代理对象就是在这里进行的
            definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
            
            // 调用 BeanDefinitionReaderUtils 工具类注册beanDefinition
            // 此时，springboot启动类就被转换为 BeanDefiniton 注册到spring容器中了
            BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
        }
    }
```

## 2.5 启动类的注册完毕
上面源码分析中，springboot启动类最终被转换成了一个BeanDefinition对象，注册到spring容器中了。    
   -  上述源码分析的只是针对springboot启动类被注册的流程，但实际上那些代码在spring框架中都是解耦的，都是独立作为组件对外提供服务的。它们在容器的其他生命周期阶段也同样提供对应的能力，即它们会在不同的时机被反复的调用。   
   -  启动类被注册到spring容器中，它的BeanDefinition对象会携带它的所有信息。包括所有的注解和解析出来的所有属性，比如scope属性，是否懒加载等，都被维护在它的BeanDefinition对象中。 
   -  流程截止目前，只是启动类被注册到容器中了，它之所以被注册，肯定是要在后续流程中被使用，因此才会被优先注册。
   -  到这个阶段，spring容器中优先注册了一堆Processor，和当前启动类的BeanDefinition对象，都是为了在后续流程中使用。   
   
***启动类被转换为BeanDefinition并注册到spring容器中，和启动类上有没有标注注解没有任何关系，只要run方法中传入了启动类的Class对象，它就会被注册（尽管源码中注册前在isEligible方法中有一些条件，但java中的类基本上都符合）***   
   
# 3. 启动类转为BeanDefinition被注册后，做了什么事？
开始进入 this.refreshContext(context); 环节。   

## 3.1 回顾
我们再回顾一下，如下方法：
```java
this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
```    
这个方法是springboot自己生命周期中的方法，上面详细分析了，这个方法执行完毕，spring容器中注册了一堆beanDefinition。   
如下是我debug列举出的beanDefinition：   
```java
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
code14Application
```   
和源码相比，除了jpa的Processor对象没有被注册，其余5个内置Processor和启动类，都被注册到spring容器中了。   
```java
// jpa的没有被注册，因为有条件限制
org.springframework.context.annotation.internalPersistenceAnnotationProcessor
```

因此，prepareContext()方法执行完毕，后续的流程实际上依靠的就是容器中预先注册的这些beanDefinition了。    
其中各个processor是重点，因为每一个processor都实现了spring容器的不同扩展钩子，它们在容器的不同生命周期中发挥了至关重要的作用。    

## 3.2 springboot的refreshContext方法，正式进入spring容器的启动流程
spring的refresh()方法都分析好多遍了。     
其中关键的逻辑在于上面注册的 ConfigurationClassPostProcessor：   
```java
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
```
这个beanName注册了一个 ConfigurationClassPostProcessor ，它是spring容器提供的一个处理器，专门用来负责处理配置类的一个强大的处理器。     

-  可以看出，springboot就是搭建一套体系，对spring框架进行整合。  
-  springboot几乎自己不负责执行任何核心流程，都是深度整合spring，借助spring本身提供的各种钩子实现、各种处理器等处理具体的事务。
-  springboot无外乎就是借助spring框架内部提供的钩子扩展机制，在ApplicationContext容器对象构建完毕后，容器正式refresh之前，借助这些扩展机制提前往spring容器中注册了几个BeanDefinition。
-  在springboot阶段（ApplicationContext创建之后，refresh正式进入spring生命周期之前）注册的这些BeanDefinition对象（就是上面分析的几种内置的Processors和启动类本身被转换后的BeanDefinition），在后续的spring容器启动阶段发挥着至关重要的作用。  

# 4. 依赖springboot阶段注册的启动类和Processors处理器，开始正式进入spring的生命周期
```java
this.refreshContext(context);
```
## 4.1 回顾下refresh()方法的整体流程
refresh()方法的核心流程请参考上一篇文章。    

## 4.2 关于beanDefinition的注册
核心方法在下面：
```java
this.invokeBeanFactoryPostProcessors(beanFactory);
```
这个方法是在获取了容器中的所有beanFactory类型的bean实例后，然后遍历执行其中的钩子方法。      
我们知道，通过springboot启动阶段，spring容器中已经注册了一个 ConfigurationClassPostProcessor 的beanDefinition。     
而这个钩子，是 BeanDefinitionRegistryPostProcessor 类型，因此，它会在invokeBeanFactoryPostProcessors方法中被回调触发执行。   


# 5. 在容器启动阶段回调执行处理器-ConfigurationClassPostProcessor
请注意，这个处理器至关重要，负责容器中几乎所有BeanDefinition的注册。   
尽管它的名称叫做“ConfigurationClassPostProcessor”，表示它用于处理“配置类”，但是并不意味着它只处理“@Configuration”标注的类。     
言外之意，在spring中，哪些类才称得上是配置类？这只有通过源码才能窥探出来。   
总之，ConfigurationClassPostProcessor 这个处理器就是用来解析并处理所有的配置类的。     
要理解该章节的核心逻辑：首先要理解“什么才是spring认为的配置类？”，请直接参考第6小节。     

## 5.1 入口方法
```java
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
        int registryId = System.identityHashCode(registry);
        if (this.registriesPostProcessed.contains(registryId)) {
            throw new IllegalStateException("postProcessBeanDefinitionRegistry already called on this post-processor against " + registry);
        } else if (this.factoriesPostProcessed.contains(registryId)) {
            throw new IllegalStateException("postProcessBeanFactory already called on this post-processor against " + registry);
        } else {
            this.registriesPostProcessed.add(registryId);
            this.processConfigBeanDefinitions(registry);
        }
    }
```

processConfigBeanDefinitions()方法如下：
```java
    public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    
        // 初始化一个“配置候选者”集合
        List<BeanDefinitionHolder> configCandidates = new ArrayList();
        
        // 获取当前容器中的所有 beanDefinition name
        // 此时容器中的beanDefinition，除了启动类的beanDefinition，剩下的都是一些内置的组件定义
        String[] candidateNames = registry.getBeanDefinitionNames();
        String[] var4 = candidateNames;
        int var5 = candidateNames.length;

        
        // 注意这个for循环，遍历处理所有的beanDefinition
        for(int var6 = 0; var6 < var5; ++var6) {
            String beanName = var4[var6];
            
            // 获取容器中的目标beanDefinition对象
            BeanDefinition beanDef = registry.getBeanDefinition(beanName);
            
            // 这里beanDefinition对象中有个标识属性，如果不为null，说明该配置类已经被处理过了
            // 说白了，只有当某个配置类被处理过了之后，就会给这个属性设置一个值，取值为full或者lite，即是“完整配置类”，还是“部分配置类”
            if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
                if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
                }
            } else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
                // 否则，进行校验，判断目标beanDefinition是不是配置类候选者
                // 如果是，则将其转换为一个BeanDefinitionHolder对象并添加到候选者名单
                configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
            }
        }
        // 总结上面那个for循环，就是将容器当中属于配置类的BeanDefinition作为了下面解析的候选者集合
        // 如何判断是不是配置类候选者呢？ ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory) 这个方法
        // 参考第6小节，如果一个类直接或间接标注了 @Configuration、@Component、@Import、@ComponentScan、@ImportResource、或者定义了@Bean标注的方法，这些了都是“配置类”
        // 正常启动容器的话，容器中的@Configuration配置类，就只有启动类一个
        
        
        // beanDefinition候选者容器不为空，则进行整个流程
        // 正常情况下，启动类就是候选者
        // 这也是为什么springboot启动类上必须标注 @SpringBootApplication 注解的原因，因为这个注解集成了 @Configuration 注解，@ComponentScan 注解等
        // 如果不标注，则容器中的bean就不会被注册
        if (!configCandidates.isEmpty()) {
            
            // 如果有多个候选者，则根据order排序，这意味着，多个配置配候选者可以通过Order接口或者@Order注解进行顺序指定
            configCandidates.sort((bd1, bd2) -> {
                int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
                int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
                return Integer.compare(i1, i2);
            });
            
            
            SingletonBeanRegistry sbr = null;
            // 这个if语句，核心就是获取容器中已经注册的BeanName生成器对象，然后设置给当前类
            if (registry instanceof SingletonBeanRegistry) {
                sbr = (SingletonBeanRegistry)registry;
                if (!this.localBeanNameGeneratorSet) {
                    BeanNameGenerator generator = (BeanNameGenerator)sbr.getSingleton("org.springframework.context.annotation.internalConfigurationBeanNameGenerator");
                    if (generator != null) {
                        this.componentScanBeanNameGenerator = generator;
                        this.importBeanNameGenerator = generator;
                    }
                }
            }

            // 初始化 environment
            if (this.environment == null) {
                this.environment = new StandardEnvironment();
            }


            // 解析启动类的核心逻辑在这里
            // 实例化一个 ConfigurationClassParser 对象，从命名可以看出，这个对象专门负责解析配置类
            ConfigurationClassParser parser = new ConfigurationClassParser(this.metadataReaderFactory, this.problemReporter, this.environment, this.resourceLoader, this.componentScanBeanNameGenerator, registry);
            
            // 转存一道，将配置类候选者转存到一个新的容器中，这个新容器 candidates 变量将在下面的循环中动态变化
            Set<BeanDefinitionHolder> candidates = new LinkedHashSet(configCandidates);
            // 实例化一个已经被解析处理过的配置类集合
            HashSet alreadyParsed = new HashSet(configCandidates.size());

            do {
                StartupStep processConfig = this.applicationStartup.start("spring.context.config-classes.parse");
                
                // 通过解析器解析配置类，这个方法一句话，很简单，底层做的事情很复杂
                // 遍历当前所有配置类候选者
                // 先判断当前配置类候选者是否应该被解析，即通过条件评估器 @Conditional ，判断在 PARSE_CONFIGURATION 阶段，是否应该解析当前配置类候选者
                // 如果不符合条件，则直接跳过当前配置类的解析
                // 
                parser.parse(candidates);
                parser.validate();
                Set<ConfigurationClass> configClasses = new LinkedHashSet(parser.getConfigurationClasses());
                configClasses.removeAll(alreadyParsed);
                if (this.reader == null) {
                    // 这个 ConfigurationClassBeanDefinitionReader 对象，负责将`@Bean`方法注册为BeanDefinition
                    // 这个类在后续负责从已经解析过的配置类的BeanDefinition对象中，进一步解析@Bean标注的方法，然后负责将@Bean标注的方法转换为对应的BeanDefinition对象后注册到IOC容器中
                    // 其中，在解析 @Bean 标注的方法时，会首先进行条件注解的评估，判断当前这个 @Bean 标注的方法是否需要被转换为BeanDefinition然后注册到容器中
                    // 此处对于 @Bean 方法的条件评估，处于 ConfigurationPhase.REGISTER_BEAN 阶段，可以仔细看下这块的源码
                    this.reader = new ConfigurationClassBeanDefinitionReader(registry, this.sourceExtractor, this.resourceLoader, this.environment, this.importBeanNameGenerator, parser.getImportRegistry());
                }

                // 在这里将
                this.reader.loadBeanDefinitions(configClasses);
                alreadyParsed.addAll(configClasses);
                processConfig.tag("classCount", () -> {
                    return String.valueOf(configClasses.size());
                }).end();
                candidates.clear();
                if (registry.getBeanDefinitionCount() > candidateNames.length) {
                    String[] newCandidateNames = registry.getBeanDefinitionNames();
                    Set<String> oldCandidateNames = new HashSet(Arrays.asList(candidateNames));
                    Set<String> alreadyParsedClasses = new HashSet();
                    Iterator var13 = alreadyParsed.iterator();

                    while(var13.hasNext()) {
                        ConfigurationClass configurationClass = (ConfigurationClass)var13.next();
                        alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
                    }

                    String[] var24 = newCandidateNames;
                    int var25 = newCandidateNames.length;

                    for(int var15 = 0; var15 < var25; ++var15) {
                        String candidateName = var24[var15];
                        if (!oldCandidateNames.contains(candidateName)) {
                            BeanDefinition bd = registry.getBeanDefinition(candidateName);
                            if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) && !alreadyParsedClasses.contains(bd.getBeanClassName())) {
                                candidates.add(new BeanDefinitionHolder(bd, candidateName));
                            }
                        }
                    }

                    candidateNames = newCandidateNames;
                }
            } while(!candidates.isEmpty());

            if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
                sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
            }

            if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
                ((CachingMetadataReaderFactory)this.metadataReaderFactory).clearCache();
            }

        }
    }
```

# 6. ConfigurationClassUtils-处理配置类的工具

***关于ConfigurationClassUtils工具类的部分解释***

`CONFIGURATION_CLASS_ATTRIBUTE` 是 Spring 框架内部用于区分 Bean 定义是完全配置类（`full`）还是部分配置类（`lite`）的属性。

### 属性解释：

1. **`full`**：表示该类是一个完整的配置类，也就是被 `@Configuration` 注解标注的类。  
   完整的配置类具有以下特性：
   - 它会通过 CGLIB 动态代理进行增强。
   - 该类中的 `@Bean` 方法会被代理，以确保这些方法不会被多次调用，而是以单例模式创建 bean（即每次调用 `@Bean` 方法时返回的是相同的实例）。
   - 通常是用于完全声明性配置，用于定义并管理 Bean 的生命周期。

   示例：
   ```java
   @Configuration
   public class AppConfig {
       @Bean
       public MyBean myBean() {
           return new MyBean();
       }
   }
   ```

2. **`lite`**：表示该类是部分配置类，即它可能不是通过 `@Configuration` 注解标注的，而是通过 `@Component`、`@Service`、`@Controller`、`@Repository` 等其他注解标注的类，或者是包含 `@Bean` 方法但没有 `@Configuration` 注解的类。
   - 部分配置类不会进行 CGLIB 代理增强，`@Bean` 方法在每次调用时都会创建一个新的实例（除非明确地进行其他处理，如将该 Bean 声明为单例）。
   - 这些类可以包含 `@Bean` 方法，但它们通常不用于全面的配置管理，更多是组件或业务逻辑类。

   示例：
   ```java
   @Component
   public class SomeComponent {
       @Bean
       public OtherBean otherBean() {
           return new OtherBean();
       }
   }
   ```

### `CONFIGURATION_CLASS_ATTRIBUTE` 的作用：

- Spring 会在内部为每个 `BeanDefinition` 设置一个 `CONFIGURATION_CLASS_ATTRIBUTE`，用来标记这个 `BeanDefinition` 是 `full` 还是 `lite` 类型。
- 这个属性是通过 `ConfigurationClassPostProcessor` 解析配置类时设置的。
- 这个标志位的主要作用是决定 Spring 容器是否对该类进行代理处理。

### 如何区分 `full` 和 `lite`？

- **`full`**：当类上有 `@Configuration` 注解时，Spring 将其标记为 `full`，并通过 CGLIB 动态代理增强。
- **`lite`**：当类上没有 `@Configuration` 注解但包含一个或多个 `@Bean` 方法，或是其他 Spring 组件类（如 `@Component`、`@Service` 等）时，Spring 将其标记为 `lite`，并不会代理该类。

### 具体应用场景：

1. **`@Configuration` 完整配置类（full）**：
   - 通过 CGLIB 代理，确保 `@Bean` 方法只调用一次，避免重复创建单例 Bean。
   - 主要用于显式定义 Spring 应用的配置。

2. **`@Component` 或其他组件类（lite）**：
   - 没有进行代理，所以 `@Bean` 方法在每次调用时都会生成新的实例，除非配置了作用域。
   - 用于普通组件、业务类的声明，也可以定义 `@Bean` 方法，但没有 `@Configuration` 类的特性。

### 总结：

- `CONFIGURATION_CLASS_ATTRIBUTE` 属性是 Spring 内部使用的，用于标记 Bean 定义是完整配置类（`full`）还是部分配置类（`lite`）。
- `full` 类通常是使用 `@Configuration` 注解标注的类，并会被代理以管理单例 Bean 的创建。
- `lite` 类通常是普通组件类，不会进行代理增强，每次调用 `@Bean` 方法都会创建新的实例。

***源码如下，注释已经标注。***
```java
package org.springframework.context.annotation;

import java.io.IOException;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.aop.framework.AopInfrastructureBean;
import org.springframework.beans.factory.annotation.AnnotatedBeanDefinition;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.context.event.EventListenerFactory;
import org.springframework.core.Conventions;
import org.springframework.core.annotation.Order;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.lang.Nullable;
import org.springframework.stereotype.Component;

public abstract class ConfigurationClassUtils {

    // spring容器内部定义了两个常量，专门用于将其设置到每个beanDefinition对象中，目的是为了标识目标BeanDefinition的“配置类”标记
    // full 标识目标类是一个“完整配置类”
    static final String CONFIGURATION_CLASS_FULL = "full";
    // lite 标识某表类是一个“部分配置类”
    static final String CONFIGURATION_CLASS_LITE = "lite";
    static final String CANDIDATE_ATTRIBUTE = Conventions.getQualifiedAttributeName(ConfigurationClassPostProcessor.class, "candidate");
    static final String CONFIGURATION_CLASS_ATTRIBUTE = Conventions.getQualifiedAttributeName(ConfigurationClassPostProcessor.class, "configurationClass");
    static final String ORDER_ATTRIBUTE = Conventions.getQualifiedAttributeName(ConfigurationClassPostProcessor.class, "order");
    private static final Log logger = LogFactory.getLog(ConfigurationClassUtils.class);

    // 配置类候选者指标：从这可以看出，一个类有Component注解、ComponentScan注解、Import注解、ImportResource注解，都被是为配置类的候选者指标。
    // 当然判断一个类是不是“配置类”，下面的方法有明确的逻辑。
    private static final Set<String> candidateIndicators = Set.of(Component.class.getName(), ComponentScan.class.getName(), Import.class.getName(), ImportResource.class.getName());

    public ConfigurationClassUtils() {
    }

    public static Class<?> initializeConfigurationClass(Class<?> userClass) {
        Class<?> configurationClass = (new ConfigurationClassEnhancer()).enhance(userClass, (ClassLoader)null);
        Enhancer.registerStaticCallbacks(configurationClass, ConfigurationClassEnhancer.CALLBACKS);
        return configurationClass;
    }

    // 判断一个BeanDefinition是否是配置类
    static boolean checkConfigurationClassCandidate(BeanDefinition beanDef, MetadataReaderFactory metadataReaderFactory) {
        // 获取beanDefinition描述的真实的类名称
        String className = beanDef.getBeanClassName();

        // 一个大条件：只有当beanClassName不为null，且 FactoryMethodName 为null，才进行进一步的判断
        // 意味着，@Bean方式注册的bean，将不被认为是一个配置类；通过其他工厂方法注册的bean，也不被认为是一个配置类
        // 总而言之：只有通过普通方式（@Configuration、@Component等）注册的bean才有可能是一个配置类
        if (className != null && beanDef.getFactoryMethodName() == null) {
            AnnotationMetadata metadata;
            label70: {
                if (beanDef instanceof AnnotatedBeanDefinition) {
                    AnnotatedBeanDefinition annotatedBd = (AnnotatedBeanDefinition)beanDef;
                    if (className.equals(annotatedBd.getMetadata().getClassName())) {
                        metadata = annotatedBd.getMetadata();
                        break label70;
                    }
                }

                if (beanDef instanceof AbstractBeanDefinition) {
                    AbstractBeanDefinition abstractBd = (AbstractBeanDefinition)beanDef;
                    if (abstractBd.hasBeanClass()) {
                        Class<?> beanClass = abstractBd.getBeanClass();
                        if (!BeanFactoryPostProcessor.class.isAssignableFrom(beanClass) && !BeanPostProcessor.class.isAssignableFrom(beanClass) && !AopInfrastructureBean.class.isAssignableFrom(beanClass) && !EventListenerFactory.class.isAssignableFrom(beanClass)) {
                            metadata = AnnotationMetadata.introspect(beanClass);
                            break label70;
                        }

                        return false;
                    }
                }

                try {
                    // 获取目标类的注解元信息
                    MetadataReader metadataReader = metadataReaderFactory.getMetadataReader(className);
                    metadata = metadataReader.getAnnotationMetadata();
                } catch (IOException var7) {
                    if (logger.isDebugEnabled()) {
                        logger.debug("Could not find class file for introspecting configuration annotations: " + className, var7);
                    }
                    // 获取注解失败，则直接返回false
                    return false;
                }
            }

            // 优先获取目标BeanDefinition描述的class上的 Configuration 注解相关的所有属性值
            Map<String, Object> config = metadata.getAnnotationAttributes(Configuration.class.getName());
            // 如果config属性不为null，且proxyBeanMethods属性值不为false
            // 说白了就是目标beanDefinition上有@Configuration注解，且注解中的proxyBeanMethods成员值不为false
            // 这种类就是“完整配置类”，设置 CONFIGURATION_CLASS_ATTRIBUTE 的属性为 “full”
            if (config != null && !Boolean.FALSE.equals(config.get("proxyBeanMethods"))) {
                beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, "full");
            } else {
                // 否则，再次进行判断
                // 如果不存在 @Configuration 注解，且 CANDIDATE_ATTRIBUTE 属性不为true，且isConfigurationCandidate(metadata)返回false
                // 则直接返回false，认为不是一个配置类
                if (config == null && !Boolean.TRUE.equals(beanDef.getAttribute(CANDIDATE_ATTRIBUTE)) && !isConfigurationCandidate(metadata)) {
                    return false;
                }

                // 否则认为是一个“部分配置类”
                beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, "lite");
            }

            // 获取配置类的order值，设置到 beanDefinition 属性中
            Integer order = getOrder(metadata);
            if (order != null) {
                beanDef.setAttribute(ORDER_ATTRIBUTE, order);
            }

            // 返回true
            return true;
        } else {

            // 大条件不满足：直接返回false
            return false;
        }
    }

    // 再看下这个方法，进一步判断某个类是不是“配置类候选者”
    static boolean isConfigurationCandidate(AnnotationMetadata metadata) {
        if (metadata.isInterface()) {
            // 如果一个类元是接口，则直接返回false
            return false;
        } else {

            // 否则，遍历上面的指标（Component注解、ComponentScan注解、Import注解、ImportResource注解）
            Iterator var1 = candidateIndicators.iterator();

            String indicator;
            do {
                // 当指标遍历完成后，还没有匹配上，则返回 hasBeanMethods 方法的执行结果
                if (!var1.hasNext()) {
                    return hasBeanMethods(metadata);
                }

                indicator = (String)var1.next();
                // 当前的指标元素不存在时，一直迭代
                // 意味着如果存在当前指标，则循环终止
            } while(!metadata.isAnnotated(indicator));

            // 如果上述循环没有返回结果，则返回true，认为是一个配置类
            return true;
        }
    }

    // hasBeanMethods方法：很简单，判断目标元素是否存在@Bean标注的工厂方法，只要存在，就认为是配置类
    static boolean hasBeanMethods(AnnotationMetadata metadata) {
        try {
            return metadata.hasAnnotatedMethods(Bean.class.getName());
        } catch (Throwable var2) {
            if (logger.isDebugEnabled()) {
                Log var10000 = logger;
                String var10001 = metadata.getClassName();
                var10000.debug("Failed to introspect @Bean methods on class [" + var10001 + "]: " + var2);
            }

            return false;
        }
    }

    @Nullable
    public static Integer getOrder(AnnotationMetadata metadata) {
        Map<String, Object> orderAttributes = metadata.getAnnotationAttributes(Order.class.getName());
        return orderAttributes != null ? (Integer)orderAttributes.get("value") : null;
    }

    public static int getOrder(BeanDefinition beanDef) {
        Integer order = (Integer)beanDef.getAttribute(ORDER_ATTRIBUTE);
        return order != null ? order : 2147483647;
    }
}
```
上述工具类主要逻辑如下：   
-  判断一个类是否是配置类。
-  配置类的标准是：
  -  直接或者间接存在@Configuration注解的类，是“完整配置类”，直接返回true
  -  不存在 @Configuration注解 ，但直接或间接存在 @Component注解、@ComponentScan注解、@Import注解、@ImportResource注解 之一的，认为是“部分配置类”，直接返回true
  -  不存在上述注解的，但是有@Bean标注的工厂方法的，也认为是“部分配置类”，直接返回true
- 其他场景统一返回false。

从这里就可以看到，一个类被spring认为是“配置类”的标准很明确：
-  如果一个类有@Configuration注解，或者间接标注了@Configuration注解，则是“完整配置类”
-  一个类有@Controller、@Service、@Component、@Import、@ComponentScan、@ImportResource等，则是“部分配置类”
-  一个类没有spring的相关bean注解，但存在@Bean工厂方法，也认为是“部分配置类”；这一点很重要，意味着“@Bean”注解可以标注在一个普通类上，这个类可以不存在任何spring的bean管理注解。

# 7. ConfigurationPhase
这是一个枚举类，里面标注了两个枚举类型：   
```java
package org.springframework.context.annotation;

public interface ConfigurationCondition extends Condition {
    ConfigurationCondition.ConfigurationPhase getConfigurationPhase();

    public static enum ConfigurationPhase {
        PARSE_CONFIGURATION,
        REGISTER_BEAN;

        private ConfigurationPhase() {
        }
    }
}
```
详细解释如上两个枚举类型参与条件评估的阶段：   
PARSE_CONFIGURATION：该类型，表示是在spring容器最早按照Resource的加载顺序一个个加载classpath下面的每一个class对象后，然后判断class对象是否有指定的注解或者否和配置类的标准（即是否存在那些spring的组件注解等），如果存在则这些class就符合spring的配置类规则。    
然后开始解析这些配置类，所谓解析，即将这些配置类从class对象转换为IOC容器内部的BeanDefinition对象后进行注册。    
但是，在开始解析这些配置类前，会判断这些配置类上面的条件注解，即：    
开始解析当前配置类A，如果A上存在条件注解，那么就从IOC容器中去获取条件注解中指定的条件（即那些已经注册到IOC容器中的BeanDefinition）是否通过评估。   
如果通过评估，则开始解析该配置类A，将其转换为BeanDefinition对象后，注册到IOC容器中。    
如果没有通过评估，就不对配置类A进行解析，此时配置类A本身就不会转换为BeanDefinition对象，当然也就不会进行注册。   
所以，PARSE_CONFIGURATION阶段，一般是用来评估一个候选者组件，是否满足条件评估，满足，才开始解析它，将其转换为BeanDefinition后注册，不满足，则不进行任何解析。   
这意味着，当一堆配置类彼此都标注了条件注解时，我们必须保证，当前解析的配置类A，假如条件注解中依赖配置类B，则B必须确保在配置类A之前就被解析，这也是为什么需要@AutoConfigureAfter、@AutoConfigureBefore等注解来标注一堆配置类的解析优先级的原因。     
所以，配置类（包括@Component等标注的组件也算配置类）上的条件注解，是在解析当前配置类的阶段进行评估的。评估失败，则当前配置类就不会被解析为BeanDefinition。   

REGISTER_BEAN：该类型，实际上和上述的阶段类似，只不过该阶段是用于解析配置类里面的@Bean方法时，进行评估的。    
请注意，此时@Bean所属的配置类肯定优先被解析了，才会进入到对@Bean方法的解析。    
而处理@Bean方法的本质，也是将其返回值类型解析为对应的BeanDefinition后，注册到容器中。   
只不过此时，一旦条件评估通过，则这个@Bean标注的方法就会被成功解析为BeanDefinition对象，最后成功注册到IOC容器中。    
如果评估失败，则@Bean标注的方法，最终不会被转换为BeanDefinition对象，容器中也就不会进行注册。    
因此，两者区别的本质就是，前者决定@Bean所在的配置类是否会被解析（即配置类本身是否会被转换为BeanDefinition对象然后注册到容器中），后者决定的是配置类中通过@Bean标注的方法是否会被解析为对应的BeanDefinition然后注册到容器中。   
