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
springboot的启动阶段中，关于bean的管理，主要分为两个大阶段：   
1.将指定范围内的class转换为BeanDefinition，并注册到spring容器。
-  扫描得到一批class，将其转换为对应的BeanDefinition对象，然后注册到spring容器中。

2.遍历容器中的所有BeanDefinition，进入bean的实例化阶段并组装bean之间的依赖关系。
-  等到容器中的所有期望管理的bean都被转换为BeanDefinition对象并注册到容器后，开始根据BeanDefinition去实例化对应的目标对象，同时组装目标对象之间的依赖关系等，正式进入bean的实例化、属性填充、初始化等阶段。      

请注意，上述两个阶段中，阶段1整体执行完毕之后，即所有期望的BeanDefinition注册到容器之后，才会整体进入阶段2进行目标bean的实例化操作。    
然而，在阶段1中，spring容器也会优先从容器中根据BeanDefinition，提前实例化一些内部的bean实例处理，比如那一堆processor和其他的组件等，提前实例化的目的是为了调用这些组件对象中的某些方法逻辑，替容器完成一些事情。    

***关键在于如何理解getBean()方法***   
请注意，spring容器启动的任何阶段，只要通过beanFactory调用了getBean()方法，它实际上就是在触发spring去管理目标bean(实例化目标bean以及组装依赖关系)。    
它的逻辑如下：    
-  检查bean的scope，优先去scope对应的缓存容器=中查询，如果有就返回
-  如果没有，则直接调用对应scope生成bean的逻辑，然后将其缓存起来，并返回
因此整个方法，实际上只要被调用了，就是在触发spring容器负责实例化目标bean了！！！


本文优先分析第一个大阶段，开始之前，先回顾一下springboot启动的大致流程。   

-  始于启动类，注意启动类是个main方法，上面标注了@SpringBootApplication注解。
-  run方法中传入了当前启动类的class对象，传入的这个对象在后续有至关重要的作用，这个class对象将作为springboot启动时扫描当前容器的入口。
    -  主要逻辑是解析启动类上的注解，请注意，spring框架底层在解析某个元素上面的注解时，往往都是解析出复合注解，即直接解析出目标元素上标注的所有注解，以及注解上标注的所有注解，简而言之，就是获取到目标元素的所有注解，从中检索出想要的主要。
    -  @SpringBootApplication注解是个复合注解，其中很重要的一个注解是@SpringBootConfiguration，而@SpringBootConfiguration注解也是个复合注解，其中很重要的一个注解是@Configuration。
    -  因此可以看出，run方法传入了当前启动类的class对象进去，后续会在容器启动过程中解析出这个class对象上的@Configuration注解，从而表示启动类是一个java配置类。
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
        this(registry, true);
    }

    public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters) {
        this(registry, useDefaultFilters, getOrCreateEnvironment(registry));
    }

    public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters, Environment environment) {
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
            this.registerDefaultFilters();
        }

        this.setEnvironment(environment);
        this.setResourceLoader(resourceLoader);
    }
```

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

        if (this.isEligible(source)) {
            
            // 构造器阶段初始化的 AnnotatedBeanDefinitionReader 对象，在这个用到了
            this.annotatedReader.register(new Class[]{source});
        }

    }
```

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


