---
layout:     post
title:      springboot启动流程简单分析
subtitle:   对springboot的启动流程源码做简单的分析
categories: [spring]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. springboot主要流程分析
## 1.1 springboot启动入口分析：  
（1）.在主方法中调用 SpringApplication 的run();
```java
//默认扫描当前类所在包和当前子包的所有注解代码
@SpringBootApplication
@EnableAsync
//开启异步执行任务
public class Code04ControllerApplication {
    public static void main(String[] args) {
        SpringApplication.run(Code04ControllerApplication.class, args);
    }
}
```
（2）执行run  
```java
    public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
        return run(new Class[]{primarySource}, args);
    }
```
（3）如下：    
```java
    public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
        return (new SpringApplication(primarySources)).run(args);
    }
```
这个方法就是springboot这个启动的核心方法，返回了一个ConfigurableApplicationContext 对象。  
主要分为两步：  
a.根据primarySources去实例化SpringApplication对象；  
b.实例化好SpringApplication对象后，传入args调用其run()方法得到一个ConfigurableApplicationContext 上下文对象。  

## 1.2 springboot整个启动流程主要包含如下2个大步骤：  
（1）.执行 new SpringApplication(primarySources) 构造方法创建 SpringApplication对象；  
（2）.创建好 SpringApplication 对象后，执行其run()方法。      

# 2. 实例化SpringApplication对象  
## 2.1 创建SpringApplication对象。  
根据primarySources去执行new SpringApplication(primarySources)构造器：  
（1）进入到 SpringApplication 的构造器中：  
```java
    public SpringApplication(Class... primarySources) {
        this((ResourceLoader)null, primarySources);
    }
```
根据传入的primarySources去调用构造方法：  
第1个传入参数是ResourceLoader，传入null；  
第2个参数是primarySources，即主类中一路传递下来的Code04ControllerApplication.class。  

（2）可以看到，传入了动态参数 primarySources，然后ResourceLoader是null，调用了当前的构造器：  
```java
    public SpringApplication(ResourceLoader resourceLoader, Class... primarySources) {
        this.sources = new LinkedHashSet();
        this.bannerMode = Mode.CONSOLE;
        this.logStartupInfo = true;
        this.addCommandLineProperties = true;
        this.addConversionService = true;
        this.headless = true;
        this.registerShutdownHook = true;
        this.additionalProfiles = Collections.emptySet();
        this.isCustomEnvironment = false;
        this.lazyInitialization = false;
        this.applicationContextFactory = ApplicationContextFactory.DEFAULT;
        this.applicationStartup = ApplicationStartup.DEFAULT;
        this.resourceLoader = resourceLoader;
        Assert.notNull(primarySources, "PrimarySources must not be null");
        this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
        this.webApplicationType = WebApplicationType.deduceFromClasspath();
        this.bootstrapRegistryInitializers = new ArrayList(this.getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
        this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
        this.mainApplicationClass = this.deduceMainApplicationClass();
    }
```
该构造方法做了很多事情，初始化了SpringApplication的很多属性。下面简单加上注释进行描述：  
```java
    public SpringApplication(ResourceLoader resourceLoader, Class... primarySources) {
        // 搞一个set容器
        this.sources = new LinkedHashSet();
        
        // 设置banner的模式为console打印
        this.bannerMode = Mode.CONSOLE;
        
        // 设置logStartInfo为true
        this.logStartupInfo = true;
        
        this.addCommandLineProperties = true;
        this.addConversionService = true;
        this.headless = true;
        this.registerShutdownHook = true;
        this.additionalProfiles = Collections.emptySet();
        this.isCustomEnvironment = false;
        this.lazyInitialization = false;
        
        // 重点，初始化 applicationContextFactory 对象。
        // 从名称就可以看出，该工厂对象专门用来创建spring上下文applicationContex的。
        this.applicationContextFactory = ApplicationContextFactory.DEFAULT;
        
        // 设置ApplicationStartup对象
        this.applicationStartup = ApplicationStartup.DEFAULT;
        
        //设置resourceLoader
        this.resourceLoader = resourceLoader;
        
        // 对primarySources对象做断言校验
        Assert.notNull(primarySources, "PrimarySources must not be null");
        
        // 根据传入的primarySources初始化一个Set容器
        this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
        
        // 遍历classpath下的所有class对象，检查是否包含某些特殊类。
        // 其实就是在扫描是否有webFlux需要的类，如果包含则返回WebApplicationType枚举中的REACTIVE，匹配不到返回NONE，否则返回SERVLET。
        // 这个值决定着springboot启动哪种类型的spring上下文，是Servlet很还是Reactive呢？
        this.webApplicationType = WebApplicationType.deduceFromClasspath();
        
        // 初始化bootstrapRegistryInitializers对象，这玩意儿的类型是List<BootstrapRegistryInitializer>，是个集合容器。
        this.bootstrapRegistryInitializers = new ArrayList(this.getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
        
        // 初始化initializers对象，这是一个List<ApplicationContextInitializer<?>>类型的集合
        // 和其他钩子接口一样，ApplicationContextInitializer也是个函数式接口，只有一个抽象方法 initialize
        // 该方法接收一个spring applicationContext对象。
        // 可见，该钩子主要是用来操作spring容器对象的。
        this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
        
        // 找出key为类型ApplicationListener的所有实现，反射实例化后聚合为一个集合返回。
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
        
        // 遍历当前的方法执行堆栈，找到“main"所在的类，返回其class对象。
        this.mainApplicationClass = this.deduceMainApplicationClass();
    }
```

## 2.2 ApplicationContextFactory.DEFAULT
（1）上面构造器中有如下方法：  
```java
this.applicationContextFactory = ApplicationContextFactory.DEFAULT;
```
（2）点击ApplicationContextFactory.DEFAULT进去：
```java
    ApplicationContextFactory DEFAULT = (webApplicationType) -> {
        try {
            switch(webApplicationType) {
            case SERVLET:
                return new AnnotationConfigServletWebServerApplicationContext();
            case REACTIVE:
                return new AnnotationConfigReactiveWebServerApplicationContext();
            default:
                return new AnnotationConfigApplicationContext();
            }
        } catch (Exception var2) {
            throw new IllegalStateException("Unable create a default ApplicationContext instance, you may need a custom ApplicationContextFactory", var2);
        }
    };
```
而且DEFAULT是在ApplicationContextFactory接口中定义的一个默认自身常量对象，ApplicationContextFactory接口的定义如下：  
```java
@FunctionalInterface
public interface ApplicationContextFactory {}
```
ApplicationContextFactory使用 @FunctionalInterface 标注，实际上是一个函数式接口类型，就是一个钩子对象，其只有一个需要客户端实现逻辑的抽象方法：  
```java
ConfigurableApplicationContext create(WebApplicationType webApplicationType);
```
该方法根据传入的webApplicationType去创建对应的spring容器对象。  

而上面的  ApplicationContextFactory DEFAULT 对象，实际上已经通过lambda表达式定义了抽象方法create(WebApplicationType webApplicationType)的回调逻辑，我们简单加上注释再看下：  
```java
    ApplicationContextFactory DEFAULT = (webApplicationType) -> {
        try {
            // 根据传入的webApplicationType去做判断
            switch(webApplicationType) {
                
                // 如果是SERVLET则创建AnnotationConfigServletWebServerApplicationContext容器
            case SERVLET:
                return new AnnotationConfigServletWebServerApplicationContext();
                
                // 如果是REACTIVE则创建AnnotationConfigReactiveWebServerApplicationContext容器
            case REACTIVE:
                return new AnnotationConfigReactiveWebServerApplicationContext();
                
                // 如果都不是则创建AnnotationConfigApplicationContext容器
            default:
                return new AnnotationConfigApplicationContext();
            }
        } catch (Exception var2) {
            throw new IllegalStateException("Unable create a default ApplicationContext instance, you may need a custom ApplicationContextFactory", var2);
        }
    };
```
这个钩子回调对象实现的逻辑如下：  
根据传入的webApplicationType分别去创建不同的springContext对象。  
其中SERVLET类型表示创建传统的servlet容器（spring5之前的），而REACTIVE表示创建Reactive容器（spring5新推出的webFlux交互式web），如果不传入webApplicationType，则默认创建一个AnnotationConfigApplicationContext对象。  

总结：  
```java
this.applicationContextFactory = ApplicationContextFactory.DEFAULT;
```
这行代码的意思很清楚了，在实例化SpringApplication对象时，初始化一个 ApplicationContextFactory 工厂对象，其中的 create(WebApplicationType webApplicationType)方法用来根据webApplicationType去创建对应的spring容器。   
而create(WebApplicationType webApplicationType)是一个抽象方法，其实现需要客户端去做，因此直接给了一个ApplicationContextFactory.DEFAULT对象，该对象是一个lambda表达式，用来表示对create()方法的实现逻辑。   

## 2.3 WebApplicationType.deduceFromClasspath()
（1）上面的构造器有如下方法：
```java
this.webApplicationType = WebApplicationType.deduceFromClasspath();
``` 
（2）点击进去：
```java
    static WebApplicationType deduceFromClasspath() {
        if (ClassUtils.isPresent("org.springframework.web.reactive.DispatcherHandler", (ClassLoader)null) && !ClassUtils.isPresent("org.springframework.web.servlet.DispatcherServlet", (ClassLoader)null) && !ClassUtils.isPresent("org.glassfish.jersey.servlet.ServletContainer", (ClassLoader)null)) {
            return REACTIVE;
        } else {
            String[] var0 = SERVLET_INDICATOR_CLASSES;
            int var1 = var0.length;

            for(int var2 = 0; var2 < var1; ++var2) {
                String className = var0[var2];
                if (!ClassUtils.isPresent(className, (ClassLoader)null)) {
                    return NONE;
                }
            }

            return SERVLET;
        }
    }
```
这个方法实际上是检查classLoader是否可以加载某些特殊类。       
比如能加载 org.springframework.web.reactive.DispatcherHandler 类，且不能加载org.springframework.web.servlet.DispatcherServlet 和 org.glassfish.jersey.servlet.ServletContainer，则返回REACTIVE；    
简单点理解就是根据当前工程中classpath是否存在某些class来判断当前工程是SERVLET，还是REACTIVE，还是NONE。  

此处拿到的 webApplicationType 实际上就是上面的 applicationContextFactory 对象在后面要调用的create()方法需要传入的web类型，用于区分到底创建哪种类型的spring容器。  

## 2.4 向springboot容器注册BootstrapRegistryInitializer钩子集合
（1）上面的构造方法同样存在如下：
```java
this.bootstrapRegistryInitializers = new ArrayList(this.getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
```
（2）getSpringFactoriesInstances()方法如下：
```java
    // 传入一个class对象，然后返回一个该对象类型的集合
    private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
        return this.getSpringFactoriesInstances(type, new Class[0]);
    }
```
（3）getSpringFactoriesInstances
```java
    private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
        ClassLoader classLoader = this.getClassLoader();
        // 获取classpath下所有的sprig.factories文件中配置的key为type指定类型的所有values
        Set<String> names = new LinkedHashSet(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
        // 根据names中聚合的各个权限定名称反射实例化出对应的对象，重新聚合成一个list。
        List<T> instances = this.createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
        // 对聚合后的对象按照order指定的顺序从小到大排序
        AnnotationAwareOrderComparator.sort(instances);
        return instances;
    }
```
（4）loadFactoryNames
```java
    public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
        ClassLoader classLoaderToUse = classLoader;
        if (classLoader == null) {
            classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
        }

        String factoryTypeName = factoryType.getName();
        // 通过这个方法加载classpath下所有的META-INF/spring.factories文件中配置的属性k-v，然后返回一个map；
        // 最后在该map中返回key为factoryTypeName指定的values。
        return (List)loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
    }
```
（5）loadSpringFactories
```java
    private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
        Map<String, List<String>> result = (Map)cache.get(classLoader);
        if (result != null) {
            return result;
        } else {
            HashMap result = new HashMap();

            try {
                // 可以看到，通过classLoader去获取当前工程中classpath下的所有META-INF/spring.factories文件。
                // 注意是所有的该文件，包括任意依赖进的jar包里面的。
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

整理分析出来，即这一步是遍历classpath下的所有META-INF/spring.factories中配置的key-value属性对，从中获取key为BootstrapRegistryInitializer权限定名的value值，其value是一个list，使用逗号分隔。  
value本身也是一堆该类型的实现子类的权限名集合，然后通过反射实例化这些实现子类对象。  

总结：  
<font color="#dc143c"><b><u>第1个向springboot容器注册的钩子接口：BootstrapRegistryInitializer（来源：springboot）</u></b></font>

## 2.5 向springboot容器注册ApplicationContextInitializer钩子集合  
（1）同样存在如下逻辑：  
```java
this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
```
原理类似，初始化initializers对象，这是一个List<ApplicationContextInitializer<?>>类型的集合；遍历整个classpath下的所有 META-INF/spring.factories 文件，找到其中key为ApplicationContextInitializer类型的所有value，然后反射实例化它们后返回。  
着重强调，ApplicationContextInitializer是spring自己的东西，而不是springboot扩展的。  
我们可以看下 ApplicationContextInitializer 的定义：  
```java
// 它所属的包是spring的
package org.springframework.context;

// 它是一个函数式接口，只有一个抽象方法
// 该方法只接收一个spring上下文参数，因此，我们可以通过该钩子对象来直接操作spring上下文
@FunctionalInterface
public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {
    void initialize(C applicationContext);
}
```  
也就是说，spring在设计之初就为开发者预留了一个钩子接口，ApplicationContextInitializer，提供了一个抽象方法，传入spring容器对象，至于该方法何时被触发，拿这个spring上下文做什么，我们先不讨论。  
我们只需要知道，我们可以通过这个钩子接口来介入spring的生命周期，在合适的时机通过插入逻辑来改变spring上下文的行文。  
很多框架整合spring容器时都利用了这个钩子，springboot也是如此，它通过这个钩子来改造spring上下文的行为。  

我们大概看一下springboot本身对这个钩子提供了哪些实现.   
我们先看下 spring-boot-autoconfigure-2.6.2.jar 的/META-INF/spring.factories文件：    
```properties
# Initializers
org.springframework.context.ApplicationContextInitializer=\   
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\   
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener   

# Application Listeners
org.springframework.context.ApplicationListener=\   
org.springframework.boot.autoconfigure.BackgroundPreinitializer   

# Environment Post Processors
org.springframework.boot.env.EnvironmentPostProcessor=\   
org.springframework.boot.autoconfigure.integration.IntegrationPropertiesEnvironmentPostProcessor   

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\   
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener   

# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\   
org.springframework.boot.autoconfigure.condition.OnBeanCondition,\   
org.springframework.boot.autoconfigure.condition.OnClassCondition,\   
org.springframework.boot.autoconfigure.condition.OnWebApplicationCondition   

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\   
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\   
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\   
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\   
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\   
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\   
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\   
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\   
org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration,\   
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\   
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\   
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\   
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRestClientAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.neo4j.Neo4jReactiveDataAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.neo4j.Neo4jReactiveRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcDataAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\   
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\   
org.springframework.boot.autoconfigure.elasticsearch.ElasticsearchRestClientAutoConfiguration,\   
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\   
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\   
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\   
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\   
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\   
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\   
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\   
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\   
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\   
org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\   
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\   
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\   
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\   
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\   
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\   
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\   
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\   
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\   
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\   
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\   
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\   
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\   
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\   
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\   
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\   
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\   
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\   
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\   
org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration,\   
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\   
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\   
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\   
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\   
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\   
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\   
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\   
org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\   
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\   
org.springframework.boot.autoconfigure.neo4j.Neo4jAutoConfiguration,\   
org.springframework.boot.autoconfigure.netty.NettyAutoConfiguration,\   
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\   
org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\   
org.springframework.boot.autoconfigure.r2dbc.R2dbcAutoConfiguration,\   
org.springframework.boot.autoconfigure.r2dbc.R2dbcTransactionManagerAutoConfiguration,\   
org.springframework.boot.autoconfigure.rsocket.RSocketMessagingAutoConfiguration,\   
org.springframework.boot.autoconfigure.rsocket.RSocketRequesterAutoConfiguration,\   
org.springframework.boot.autoconfigure.rsocket.RSocketServerAutoConfiguration,\   
org.springframework.boot.autoconfigure.rsocket.RSocketStrategiesAutoConfiguration,\   
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\   
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\   
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\   
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\   
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\   
org.springframework.boot.autoconfigure.security.rsocket.RSocketSecurityAutoConfiguration,\   
org.springframework.boot.autoconfigure.security.saml2.Saml2RelyingPartyAutoConfiguration,\   
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\   
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\   
org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\   
org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\   
org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\   
org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\   
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\   
org.springframework.boot.autoconfigure.sql.init.SqlInitializationAutoConfiguration,\   
org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\   
org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\   
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\   
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\   
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\   
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.reactive.ReactiveMultipartAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.reactive.WebSessionIdResolverAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\   
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\   
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\   
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\   
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\   
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\   
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration   

# Failure analyzers
org.springframework.boot.diagnostics.FailureAnalyzer=\   
org.springframework.boot.autoconfigure.data.redis.RedisUrlSyntaxFailureAnalyzer,\   
org.springframework.boot.autoconfigure.diagnostics.analyzer.NoSuchBeanDefinitionFailureAnalyzer,\   
org.springframework.boot.autoconfigure.flyway.FlywayMigrationScriptMissingFailureAnalyzer,\   
org.springframework.boot.autoconfigure.jdbc.DataSourceBeanCreationFailureAnalyzer,\   
org.springframework.boot.autoconfigure.jdbc.HikariDriverConfigurationFailureAnalyzer,\   
org.springframework.boot.autoconfigure.jooq.NoDslContextBeanFailureAnalyzer,\   
org.springframework.boot.autoconfigure.r2dbc.ConnectionFactoryBeanCreationFailureAnalyzer,\   
org.springframework.boot.autoconfigure.r2dbc.MissingR2dbcPoolDependencyFailureAnalyzer,\   
org.springframework.boot.autoconfigure.r2dbc.MultipleConnectionPoolConfigurationsFailureAnalzyer,\   
org.springframework.boot.autoconfigure.r2dbc.NoConnectionFactoryBeanFailureAnalyzer,\   
org.springframework.boot.autoconfigure.session.NonUniqueSessionRepositoryFailureAnalyzer   

# Template availability providers
org.springframework.boot.autoconfigure.template.TemplateAvailabilityProvider=\   
org.springframework.boot.autoconfigure.freemarker.FreeMarkerTemplateAvailabilityProvider,\   
org.springframework.boot.autoconfigure.mustache.MustacheTemplateAvailabilityProvider,\   
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAvailabilityProvider,\   
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafTemplateAvailabilityProvider,\   
org.springframework.boot.autoconfigure.web.servlet.JspTemplateAvailabilityProvider   

# DataSource initializer detectors
org.springframework.boot.sql.init.dependency.DatabaseInitializerDetector=\   
org.springframework.boot.autoconfigure.flyway.FlywayMigrationInitializerDatabaseInitializerDetector   

# Depends on database initialization detectors
org.springframework.boot.sql.init.dependency.DependsOnDatabaseInitializationDetector=\   
org.springframework.boot.autoconfigure.batch.JobRepositoryDependsOnDatabaseInitializationDetector,\   
org.springframework.boot.autoconfigure.quartz.SchedulerDependsOnDatabaseInitializationDetector,\   
org.springframework.boot.autoconfigure.session.JdbcIndexedSessionRepositoryDependsOnDatabaseInitializationDetector   
```
我们可以看到，第一行就是ApplicationContextInitializer的相关配置：   
```properties
# Initializers
org.springframework.context.ApplicationContextInitializer=\   
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\   
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener   
```   
当然其他整合的jar包中同样也可能提供了/META-INF/spring.factories文件，其中可能也有自己配置的一些钩子。我们暂时先看看autoconfig包下面的ApplicationContextInitializer。   
我们可以看到配置了两个实现类：   
```java
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer;
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener
```

SharedMetadataReaderFactoryContextInitializer类定义如下：   
```java
// 实现了 ApplicationContextInitializer 接口 和 Ordered 接口
// SharedMetadataReaderFactoryContextInitializer这个实现很重要，spring后期的beanDefinition的加载实际上就是在这个类中为spring容器注册的对应的BeanFactoryPostProcessor。
// 然后这些BeanFactoryPostProcessor在后面做了很多事。这个流程后面再分析，目前只需要记住，这几个实现类很重要！！！
class SharedMetadataReaderFactoryContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext>, Ordered {}
```
实现的initialize方法：   
```java
    // 接收一个ConfigurableApplicationContext类型的spring上下文
    public void initialize(ConfigurableApplicationContext applicationContext) {
        // 在这里，拿到spring容器对象后，其实它自己也不干活，而是创建了一个当前内部类 CachingMetadataReaderFactoryPostProcessor 对象。
        // 这个对象是个 BeanFactoryPostProcessor 类型的，然后将其注册到spring容器中。
        // 具体这个对象怎么使用，我们后面再分析，目前只需要清楚，这个ApplicationContextInitializer的该实现，给spring容器中注册了一个BeanFactoryPostProcessor。
        BeanFactoryPostProcessor postProcessor = new SharedMetadataReaderFactoryContextInitializer.CachingMetadataReaderFactoryPostProcessor(applicationContext);
        applicationContext.addBeanFactoryPostProcessor(postProcessor);
    }
```

ConditionEvaluationReportLoggingListener类定义如下：  
```java
public class ConditionEvaluationReportLoggingListener implements ApplicationContextInitializer<ConfigurableApplicationContext> {}
```
实现的initialize方法：   
```java
    public void initialize(ConfigurableApplicationContext applicationContext) {
        // 该方法将拿到的容器对象直接初始化成自己的一个属性了，后面人家就能使用自己的这个属性了
        this.applicationContext = applicationContext;
        // 然后为spsirng容器对象注册了一个ApplicationListener 监听器。
        applicationContext.addApplicationListener(new ConditionEvaluationReportLoggingListener.ConditionEvaluationReportListener());
        if (applicationContext instanceof GenericApplicationContext) {
            this.report = ConditionEvaluationReport.get(this.applicationContext.getBeanFactory());
        }

    }
```

总结：  
我们可以看到，这些钩子拿到spring容器对象后，实际上自己也不怎么干活，而是使劲儿地往spring上下文上注册一些其他的对象。   
其实最后干活的都是注册的那些对象，委托这些对象在合适的时机进行干活。   
<font color="#dc143c"><b><u>第2个注册的钩子接口：ApplicationContextInitializer（来源：spring）</u></b></font>

## 2.6 向springboot容器注册ApplicationListener集合
（1）存在如下逻辑：
```java
this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
```
同上，找出key为类型ApplicationListener的所有实现，反射实例化后聚合为一个集合返回。  

ApplicationListener接口定义如下：
```java
// 同样属于spring
package org.springframework.context;

import java.util.EventListener;
import java.util.function.Consumer;

// 该接口也是一个函数式接口
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    // 提供了一个抽象方法
    void onApplicationEvent(E event);

    // 同时还实现了一个静态方法
    static <T> ApplicationListener<PayloadApplicationEvent<T>> forPayload(Consumer<T> consumer) {
        return (event) -> {
            consumer.accept(event.getPayload());
        };
    }
}

```
该接口在springboot autoconfig包下的配置：   
```properties
# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer
```

总结：  
<font color="#dc143c"><b><u>第3个向springboot容器注册的钩子接口：ApplicationListener（来源：spring）</u></b></font>

## 2.6 溯源主类
（1）构造器的最后一行代码如下：
```java
// 初始化mainApplicationClass属性，该属性类型为Class<?>
this.mainApplicationClass = this.deduceMainApplicationClass();
```

（2）deduceMainApplicationClass方法如下：
```java
    private Class<?> deduceMainApplicationClass() {
        try {
            // 遍历当前的方法执行堆栈，找到“main"所在的类，返回其class对象。
            StackTraceElement[] stackTrace = (new RuntimeException()).getStackTrace();
            StackTraceElement[] var2 = stackTrace;
            int var3 = stackTrace.length;

            for(int var4 = 0; var4 < var3; ++var4) {
                StackTraceElement stackTraceElement = var2[var4];
                if ("main".equals(stackTraceElement.getMethodName())) {
                    return Class.forName(stackTraceElement.getClassName());
                }
            }
        } catch (ClassNotFoundException var6) {
            ;
        }

        return null;
    }
```
StackTraceElement对象用来描述当前线程执行方法堆栈路径中每一个方法栈。  
其中包含了如下几个内容：  
a.当前线程执行过的方法的名称。  
b.当前线程执行过的方法所在类的权限定名。  
c.当前线程执行过的方法所在java源文件的名称。  
d.当前线程执行过的方法触发下一个方法执行堆栈的位置(即在当前方法中哪一行将流程转发到下一个方法中去了，以行号进行登记)。  

总结： 上述方法就是在遍历整个方法堆栈执行链路，找到main方法所在方法所属的class，然后返回。  

## 2.6 SpringApplication构造器总结
到这里，SpringApplication就已经实例化完成了。  
可以看到，从程序中只传入了一个primarySources进来，但它的构造器去初始化了很多属性。  
这些初始化所需要的参数来源都是遍历classpath下的所有"META-INF/spring.factories"，根据其中的配置实例化了很多对象进来用于初始化设置。  
在构造 SpringApplication 对象的过程中，通过构造器初始化了该对象的很多属性，这些预置的属性将在后面执行run()方法的过程中被使用。  

# 3. 调用SpringApplication对象的run()方法
## 3.1 run方法
根据实例化好的SpringApplication对象，调用其中的run()方法，实现整个springboot的核心启动逻辑。  
（1）进入到run方法：
```java
    public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
        return (new SpringApplication(primarySources)).run(args);
    }
```
（2）run方法的源码如下：  
```java
    public ConfigurableApplicationContext run(String... args) {
        long startTime = System.nanoTime();
        // 创建一个bootstrap上下文对象，里面遍历执行了 BootstrapRegistryInitializer 接口的所有回调实现逻辑。
        DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();
        ConfigurableApplicationContext context = null;
        
        // 将headless的值设置为系统属性 java.awt.headless
        this.configureHeadlessProperty();
        
        // 从sprng.factories中获取SpringApplicationRunListener的所有实现类，然后反射成对象后，封装到一个新的对象SpringApplicationRunListeners中。
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
        // 遍历执行 SpringApplicationRunListener 接口的 starting 方法。
        listeners.starting(bootstrapContext, this.mainApplicationClass);

        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            
            // 准备环境，遍历执行 SpringApplicationRunListener 接口的 environmentPrepared 方法。
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
            this.configureIgnoreBeanInfo(environment);
            Banner printedBanner = this.printBanner(environment);
            
            // 创建spring容器对象，同时会创建一些内置的beanDefinition注入到beanFactory中，不过这些内置的beanDefinition可以不用关注。
            context = this.createApplicationContext();
            
            // 给容器对象注册applicationStartup
            context.setApplicationStartup(this.applicationStartup);
            
            // 准备上下文，这段逻辑遍历执行了 ApplicationContextInitializer 接口的所有回调逻辑。
            // 同时将传递进来的主资源class转换为beanDefinition后注册到beanFactory中了。
            // 我们在前面大概分析了，ApplicationContextInitializer 接口的 initialize 方法主要用来操作spring容器对象.
            // 比如向容器中注册一些postProcessor等，当然可以对spring容器实施的逻辑有很多。
            this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
            
            // 刷新spring容器，正式启动容器，从springboot正式开始进入spring。核心！！！
            this.refreshContext(context);
            
            // afterRefresh()方法没有实现，留给客户端去实现回调逻辑，当容器启动完成之后需要操作的逻辑   
            this.afterRefresh(context, applicationArguments);
            Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), timeTakenToStartup);
            }

            // 遍历执行 SpringApplicationRunListener 所有实现对象的 started 方法
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

## 3.2 createBootstrapContext
（1）创建一个DefaultBootstrapContext对象
```java
DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();
```
（2）遍历执行构造时传入的所有 BootstrapRegistryInitializer 的实例对象的initialize方法。
```java
    private DefaultBootstrapContext createBootstrapContext() {
        // 创建一个DefaultBootstrapContext对象
        DefaultBootstrapContext bootstrapContext = new DefaultBootstrapContext();
        // 遍历bootstrapRegistryInitializers集合，执行其中每个对象的initialize方法。
        this.bootstrapRegistryInitializers.forEach((initializer) -> {
            // initialize方法接收一个bootstrapContext，客户端可以使用该对象做一些回调操作
            initializer.initialize(bootstrapContext);
        });
        return bootstrapContext;
    }
```

总结：  
<font color="#dc143c"><b><u>第1个回调执行的钩子接口：BootstrapRegistryInitializer（来源：springboot）</u></b></font>

## 3.3 this.getRunListeners(args)
（1）获取SpringApplicationRunListeners对象
```java
SpringApplicationRunListeners listeners = this.getRunListeners(args);
```
（2）getRunListeners方法
```java
    private SpringApplicationRunListeners getRunListeners(String[] args) {
        // 实例化一个class数组，里面存在两个class元素，第1个元素是SpringApplication的class，第2个元素是一个String数组的class
        Class<?>[] types = new Class[]{SpringApplication.class, String[].class};
        
        // 获取spring,factories文件中配置的key为SpringApplicationRunListener的所有实现子类对象
        // 将所有的SpringApplicationRunListener对象集合封装在一个SpringApplicationRunListeners对象中返回
        return new SpringApplicationRunListeners(logger, this.getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args), this.applicationStartup);
    }
```
（3）进一步看下 SpringApplicationRunListeners 的构造器：
```java
    // 通过创建一个新的对象，将原来的SpringApplicationRunListener集合进行包装
    SpringApplicationRunListeners(Log log, Collection<? extends SpringApplicationRunListener> listeners, ApplicationStartup applicationStartup) {
        this.log = log;
        this.listeners = new ArrayList(listeners);
        this.applicationStartup = applicationStartup;
    }
```
（4）执行SpringApplicationRunListeners对象的starting方法
```java
listeners.starting(bootstrapContext, this.mainApplicationClass);
```
（5）starting方法
```java
    void starting(ConfigurableBootstrapContext bootstrapContext, Class<?> mainApplicationClass) {
        // 注意这个 doWithListeners 方法
        this.doWithListeners("spring.boot.application.starting", (listener) -> {
            
            // 传入一个lambda表达式
            listener.starting(bootstrapContext);
        }, (step) -> {
            // 传入一个lambda表达式
            if (mainApplicationClass != null) {
                step.tag("mainApplicationClass", mainApplicationClass.getName());
            }

        });
    }
```
（6）doWithListeners方法
```java
    // listenerAction表示一个消费者对象，函数式接口，表示一段逻辑，stepAction同理
    private void doWithListeners(String stepName, Consumer<SpringApplicationRunListener> listenerAction, Consumer<StartupStep> stepAction) {
        StartupStep step = this.applicationStartup.start(stepName);
        
        // 遍历当前的listeners集合，List<SpringApplicationRunListener> listeners
        // 然后执行传入的回调逻辑，从上可以看出，传入的回调逻辑是 listener.starting(bootstrapContext);
        // 所以此处实际上是遍历执行SpringApplicationRunListener对象中的starting(ConfigurableBootstrapContext bootstrapContext)方法。
        this.listeners.forEach(listenerAction);
        if (stepAction != null) {
            stepAction.accept(step);
        }

        step.end();
    }
```

总结：  
<font color="#dc143c"><b><u>第2个回调执行的钩子接口：SpringApplicationRunListener 接口的 starting 方法（来源：springboot）</u></b></font>

## 3.4 this.prepareEnvironment(listeners, bootstrapContext, applicationArguments)
（1）准备环境
```java
ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
```
（2）prepareEnvironment方法
```java
    private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners, DefaultBootstrapContext bootstrapContext, ApplicationArguments applicationArguments) {
        ConfigurableEnvironment environment = this.getOrCreateEnvironment();
        this.configureEnvironment((ConfigurableEnvironment)environment, applicationArguments.getSourceArgs());
        ConfigurationPropertySources.attach((Environment)environment);
        
        // 遍历执行 SpringApplicationRunListener 集合中的 environmentPrepared 方法
        listeners.environmentPrepared(bootstrapContext, (ConfigurableEnvironment)environment);
        DefaultPropertiesPropertySource.moveToEnd((ConfigurableEnvironment)environment);
        Assert.state(!((ConfigurableEnvironment)environment).containsProperty("spring.main.environment-prefix"), "Environment prefix cannot be set via properties.");
        this.bindToSpringApplication((ConfigurableEnvironment)environment);
        if (!this.isCustomEnvironment) {
            environment = this.convertEnvironment((ConfigurableEnvironment)environment);
        }

        ConfigurationPropertySources.attach((Environment)environment);
        return (ConfigurableEnvironment)environment;
    }
```
总结：  
<font color="#dc143c"><b><u>第3个回调执行的钩子接口：SpringApplicationRunListener 接口的 environmentPrepared 方法（来源：springboot）</u></b></font>  
注意，又学习了一招，一个钩子接口有多个方法，不同的方法可以在不同的生命周期阶段进行回调，即通过一个钩子接口就可以插入多个生命周期节点，进行埋点操作。  

## 3.5 this.createApplicationContext()
（1）创建spring容器对象
```java
context = this.createApplicationContext();
```
（2）createApplicationContext
```java
    protected ConfigurableApplicationContext createApplicationContext() {
        // 使用 SpringApplication 对象的applicationContextFactory根据构造器初始化的webApplicationType去创建对应的spring容器对象
        return this.applicationContextFactory.create(this.webApplicationType);
    }
```
我们一般启动的springboot工程都是Servlet工程，即构造器初始化的webApplicationType是SERVLET，所以此处的applicationContextFactory对象实际上是 AnnotationConfigServletWebServerApplicationContext 类型的。  
（3）进入 AnnotationConfigServletWebServerApplicationContext 中的 create 方法：
```java
    static class Factory implements ApplicationContextFactory {
        Factory() {
        }

        // AnnotationConfigServletWebServerApplicationContext类中定义了一个内部类Factory实现了ApplicationContextFactory接口
        // 覆盖了其中的create方法，根据webApplicationType去创建对应的spring容器。
        public ConfigurableApplicationContext create(WebApplicationType webApplicationType) {
            return webApplicationType != WebApplicationType.SERVLET ? null : new AnnotationConfigServletWebServerApplicationContext();
        }
    }
```
（4）进入AnnotationConfigServletWebServerApplicationContext构造器：
```java
    public AnnotationConfigServletWebServerApplicationContext() {
        // 初始化一个缓存注解Class对象的容器
        this.annotatedClasses = new LinkedHashSet();
        // 初始化一个注解的beanDefinition读取器
        this.reader = new AnnotatedBeanDefinitionReader(this);
        // 初始化一个beanDefinition扫描器
        this.scanner = new ClassPathBeanDefinitionScanner(this);
    }
```
AnnotationConfigServletWebServerApplicationContext的继承结构如下：
```java
public class AnnotationConfigServletWebServerApplicationContext extends ServletWebServerApplicationContext implements AnnotationConfigRegistry 
```
ServletWebServerApplicationContext的继承结构如下：
```java
public class ServletWebServerApplicationContext extends GenericWebApplicationContext implements ConfigurableWebServerApplicationContext
```
GenericWebApplicationContext的继承结构如下：
```java
public class GenericWebApplicationContext extends GenericApplicationContext implements ConfigurableWebApplicationContext, ThemeSource
```
（5）最终进入到GenericApplicationContext类的构造器：
```java
    public GenericApplicationContext() {
        this.customClassLoader = false;
        this.refreshed = new AtomicBoolean();
        // 这里直接初始化了spring容器中的beanFactory工厂，可以看到该对象是DefaultListableBeanFactory类型
        this.beanFactory = new DefaultListableBeanFactory();
    }
```
<b>从java知识可以知道，创建一个子类对象，会一层层地往上溯源，直到找到最顶层的父类，然后先创建父类对象，再一层层地沿着子类的方向迭代创建。  
因此，通过 new AnnotationConfigServletWebServerApplicationContext(); 这个构造器，内部就已经执行了很多初始化逻辑了。</b>

总结：
从源码可以看到，我们创建的spring容器对象实际上是 new AnnotationConfigServletWebServerApplicationContext()。  
此时spring容器对象的beanFactory也已经成功实例化了，其类型是DefaultListableBeanFactory。

## 3.6 prepareContext
（1）准备上下文
前面已经创建好了spring容器对象了，这里为spring容器对象做一些准备工作。
```java
this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
```
（2）prepareContext方法：
```java
    private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context, ConfigurableEnvironment environment, SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
        // 为spring容器设置环境
        context.setEnvironment(environment);
        // 为实例化好的spring容器设置一些属性
        this.postProcessApplicationContext(context);
        
        // 传入spring上下文，其中遍历回调了构造器时期注入的所有 ApplicationContextInitializer 对象的initialize方法。
        // 用于对spring上下文插入一些自定义逻辑。
        // 看到了吧，前面springboot时期注册的 ApplicationContextInitializer 的所有实现对象，它们的 initialize 方法就是在这个时候回调执行的。
        // 至于各个实现的initialize方法具体都干了什么事情，由实现者自己决定。
        this.applyInitializers(context);
        
        // 同上面的操作，遍历回调了SpringApplicationRunListener接口的contextPrepared方法。
        listeners.contextPrepared(context);
        
        bootstrapContext.close(context);
        if (this.logStartupInfo) {
            this.logStartupInfo(context.getParent() == null);
            this.logStartupProfileInfo(context);
        }

        // 获取spring容器对象的beanFactory工厂
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        
        // 通过beanFactory工厂注册一个单例bean，名称为 springApplicationArguments
        beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
        if (printedBanner != null) {
            // 如果printedBanner存在，也注册到spring容器中；
            // 注意，此处是直接将一个已经存在的bean对象注册到bean容器中，而不是注册beanDefinition
            beanFactory.registerSingleton("springBootBanner", printedBanner);
        }

        // 设置beanFactory工厂的一些属性
        if (beanFactory instanceof AbstractAutowireCapableBeanFactory) {
            ((AbstractAutowireCapableBeanFactory)beanFactory).setAllowCircularReferences(this.allowCircularReferences);
            if (beanFactory instanceof DefaultListableBeanFactory) {
                ((DefaultListableBeanFactory)beanFactory).setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
            }
        }

        // 如果当前spring容器被设置为懒加载，则为spring容器增加一个BeanFactoryPostProcessor处理器。
        // 该处理器用来对beanFactory工厂对象设置一些回调逻辑，即用来改造beanFactory的行为。
        if (this.lazyInitialization) {
            context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
        }
    
        // 获取传入的所有资源类，就是执行run方法时传入的Class对象，一般是main方法所在的主类
        Set<Object> sources = this.getAllSources();
        Assert.notEmpty(sources, "Sources must not be empty");
        
        // 初始化 BeanDefinitionLoader 加载器，同时初始化BeanDefinitionLoader加载器和扫描器等。
        // 同时还将主资源的beanDefinition注入到beanFactory中了，其实就是run方法传递进来的那个class对象。
        this.load(context, sources.toArray(new Object[0]));
        
        // 遍历回调了构造器时期注入的所有SpringApplicationRunListener对象的contextLoaded方法。
        listeners.contextLoaded(context);
    }
```
我们来总结一下，SpringApplication对象中的prepareContext()方法主要做了哪些事：  
a.遍历回调构造器时期注入的所有 ApplicationContextInitializer 类型对象的initialize方法，该方法接收spring容器对象，用于对spring容器对象插入一些自定义逻辑。  
b.遍历回调构造器时期注入的所有 SpringApplicationRunListener 类型对象的contextPrepared方法，该方法接收spring容器对象，用于对spring容器对象插入一些自定义逻辑。  
c.通过spring容器的beanFactory工厂，将applicationArguments对象注册到spring容器中，名称为springApplicationArguments。   
d.同理，如果printedBanner对象存在，将printedBanner也通过beanFactory工厂注册到spring容器中，名称为springBootBanner。  
e.如果当前容器被设置为懒加载，则为spring容器对象注册一个LazyInitializationBeanFactoryPostProcessor的beanFactory后置处理器。  
f.初始化spring容器的BeanDefinitionLoader加载器，其底层初始化了很多，比如bean定义加载器和扫描器等，然后将主资源转换为BeanDefinition后注册到beanFactory中。    
g.遍历回调构造器时期注入的所有SpringApplicationRunListener类型对象的contextLoaded方法，该方法接收spring容器对象，用于对spring容器对象插入一些自定义逻辑。  

总结：  
<b><u><font color="#dc143c">第4个回调执行的钩子接口：ApplicationContextInitializer 接口的 initialize 方法（来源：springboot）</font>  </u></b>      
<b><u><font color="#dc143c">第5个回调执行的钩子接口：SpringApplicationRunListener 接口的 contextPrepared 方法（来源：springboot）</font>  </u></b>     
<b><u><font color="#dc143c">第4个向springboot容器注册的钩子接口：LazyInitializationBeanFactoryPostProcessor实现（来源：springboot）</font>  </u></b>     
<b><u><font color="#dc143c">第6个回调执行的钩子接口：SpringApplicationRunListener 接口的 contextLoaded 方法（来源：springboot）</font>  </u></b>      

<b>注意，对一些钩子接口实现对象的提前注册，是为了后续在容器中不同的生命周期阶段去回调这些钩子对象的逻辑。</b>  

## 3.7 refreshContext
（1）正式启动spring容器：
```java
// 这一步标志着启动流程正式从springboot的SpringApplication开始进入到spring容器中。
this.refreshContext(context);
```
（2）SpringApplication的refreshContext方法
```java
    private void refreshContext(ConfigurableApplicationContext context) {
        // 如果允许注册停止钩子，则执行SpringApplicationShutdownHook的registerApplicationContext方法，该方法接收一个spring容器对象
        if (this.registerShutdownHook) {
            shutdownHook.registerApplicationContext(context);
        }

        this.refresh(context);
    }
```
（3）SpringApplicationShutdownHook钩子的方法：
```java
    void registerApplicationContext(ConfigurableApplicationContext context) {
        this.addRuntimeShutdownHookIfNecessary();
        Class var2 = SpringApplicationShutdownHook.class;
        synchronized(SpringApplicationShutdownHook.class) {
            this.assertNotInProgress();
            // 为spring容器注册了一个ApplicationContextClosedListener类型的监听器
            context.addApplicationListener(this.contextCloseListener);
            this.contexts.add(context);
        }
    }
```
（4）refresh方法：
```java
    // 执行传递进来的spring容器兑现的refresh方法，正式启动spring容器。
    protected void refresh(ConfigurableApplicationContext applicationContext) {
        applicationContext.refresh();
    }
```

# 4.spring容器refresh启动周期
## 4.1 refresh方法
（1）我们知道前面springboot容器创建的spring容器对象是 AnnotationConfigServletWebServerApplicationContext ，因此refresh方法应该执行的是ServletWebServerApplicationContext中的：   
```java
    public final void refresh() throws BeansException, IllegalStateException {
        try {
            // 调用父类的refresh
            super.refresh();
        } catch (RuntimeException var3) {
            WebServer webServer = this.webServer;
            if (webServer != null) {
                webServer.stop();
            }

            throw var3;
        }
    }
```
（2）super.refresh()
AbstractApplicationContext中实现好的refresh方法：
```java
    public void refresh() throws BeansException, IllegalStateException {
        Object var1 = this.startupShutdownMonitor;
        synchronized(this.startupShutdownMonitor) {
            StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");
            
            // 刷新前准备动作
            this.prepareRefresh();
            
            // 获取spring容器中的beanFactory工厂
            // 此处获取的就是上面springboot中创建spring容器时已经初始化的DefaultListableBeanFactory对象。
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            
            // 为beanFactory做一些准备工作
            // 提前注册一些内置的 beanDefinition，和一些 BeanPostProcessor 等，比如 ApplicationContextAwareProcessor，LoadTimeWeaverAwareProcessor 等。
            this.prepareBeanFactory(beanFactory);

            try {
                // 对拿到的beanFactory工厂对象的一些后置处理动作
                // 注册一个 BeanPostProcessor - WebApplicationContextServletContextAwareProcessor 等
                this.postProcessBeanFactory(beanFactory);
                
                StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
                
                // 回调执行spring容器中的所有BeanFactoryPostProcessor类型对象的postProcessBeanFactory方法。
                // 这个方法很复杂，底层跟套娃一样，一环套一环。
                // 总的来讲，干了两件事：
                // 1.首先判断前面注册的beanFactoryPostProcessor是否是BeanDefinitionRegistryPostProcessor类型，如果是，则执行其postProcessBeanDefinitionRegistry方法。
                // 2.其次判断如果是 BeanFactoryPostProcessor 类型，则执行其postProcessBeanFactory方法。
                // 我们要注意，BeanDefinitionRegistryPostProcessor接口继承了BeanFactoryPostProcessor接口，对后者进行了扩展，增加了一个postProcessBeanDefinitionRegistry方法。
                // 相当于spring容器在回调执行BeanFactoryPostProcessor的所有实现对象时，默认替我们做了优先级，首先会回调所有实现了BeanDefinitionRegistryPostProcessor类型的对象。
                // 其次才是直接实现BeanFactoryPostProcessor类型的对象。
                // 提前剧透一点：spring容器的beanFactory加载容器中的所有beanDefinition定义，就是在这个方法里面实现的。
                // 当然，肯定是某个实现了BeanDefinitionRegistryPostProcessor或者BeanFactoryPostProcessor接口的子类对象负责干的这事。
                // 其实，就是在上面分析的 SharedMetadataReaderFactoryContextInitializer 的 initialize 方法中注册的 CachingMetadataReaderFactoryPostProcessor 对象负责加载的所有beanDefinition。
                // 详情后面分析。
                this.invokeBeanFactoryPostProcessors(beanFactory);
                
                // 向spring容器中注册BeanPostProcessor对象。
                // BeanPostProcessor 的实现类对象来源主要分为两部分：
                // 1.直接在该方法内部手动实例化现有的 BeanPostProcessor 子类对象并通过 addBeanPostProcessor 方法注册到sprign容器中，比如 BeanPostProcessorChecker 和 ApplicationListenerDetector 等。
                // 2.拉起spring容器，让spring容器优先实例化容器内部类型为 BeanPostProcessor 的beanDefinition，然后通过 addBeanPostProcessor 方法注册到spring容器中。
                // 第2点，会优先获取容器中 beanDefinition 为 BeanPostProcessor 类型的所有bean定义，然后spring容器将优先实例化这些bean；
                // 根据beanDefinition信息，通过反射将其实例化成对象后返回，同时注册到spring容器中。
                // 注册时，会分别判断bean是否实现了Order接口之类的，将beanPostProcessor按照优先级进行排序。
                // 同时，还会注册一个 ApplicationListenerDetector 类型的 BeanPostProcessor，该类是spring自己实现的。
                // 也就是说，到这一步，spring容器已经优先实例化了 BeanPostProcessor 类型的所有bean对象。
                
                // 注意，和上面的BeanFactoryPostProcessor进行区分，主要区别如下：
                // 1.前者 BeanFactoryPostProcessor 和 BeanDefinitionRegistryPostProcessor 的实现类对象的实例化，以及实例化后注册到spring容器对象中，这些逻辑基本上都是springboot通过spring.factories中倒挂的ApplicationContextInitializer钩子中去实现的。
                // 而 BeanPostProcessor 类型的实现类对象的实例化，很明显是spring容器负责加载的，也就是说，BeanPostProcessor 类型的bean实际上是完全由spring容器管理的，spring容器加载完所有的beanDefinition后，这些beanDefinition中就有 BeanPostProcessor 类型。
                // 2.前者 BeanFactoryPostProcessor 和 BeanDefinitionRegistryPostProcessor 的实现类对象早就已经通过springboot借助ApplicationContextInitializer的回调方法在回调中向spring容器中注册了，而且在上面的 invokeBeanFactoryPostProcessors()方法中就已经全部回调执行了。
                // 当流程走到这一步，容器中所有的BeanFactoryPostProcessor 和 BeanDefinitionRegistryPostProcessor类型的回调逻辑已经全部执行完毕了。
                // 而在当前方法中才正式开始实例化容器中 BeanPostProcessor 类型的对象，实例化后才开始注册，至于啥时候回调执行，后面在分析。
                // 也就是说，前者 BeanFactoryPostProcessor 的回调逻辑在此处都已经被执行完了，spring容器才开始实例化容器中 BeanPostProcessor 类型的对象。
                // 所以， BeanFactoryPostProcessor 和 BeanDefinitionRegistryPostProcessor 的回调实际要远远早于 BeanPostProcessor。
                // 其实从源码中也可以看出，spring容器注册bean定义本身就是通过容器上下文中注册的 BeanDefinitionRegistryPostProcessor 回调器进行的，因此它的执行必然优先；
                // 只有它执行了，spring容器中才会有普通bean的定义存在，接着才能拿到bean定义去做下一步的动作。
                // 而所有实现 BeanPostProcessor 钩子的普通bean定义这个时候才能拿到，才能从容器中获取所有类型为BeanPostProcessor的bean，拿到bean才能将这些BeanPostProcessor注册到spring容器中。
                // 因此：BeanFactoryPostProcessor 和 BeanDefinitionRegistryPostProcessor 的回调逻辑远远早于 BeanPostProcessor 钩子。
                this.registerBeanPostProcessors(beanFactory);
                beanPostProcess.end();
                
                // 初始化 messageSource 对象。
                // messageSource 对象是用来处理消息的，说白了就是用来读取国际化资源文件的一个bean。
                // 这个后续单独写一篇文章分析。
                this.initMessageSource();
                this.initApplicationEventMulticaster();
                
                // 在spring容器正式利用beanFactory工厂根据beanDefinition去实例化bean前，做一些其他工作
                // 主要是创建web运行环境，即将servlet容器（Servlet、Filter、Listener等）纳入到spring容器中来。
                // 我们知道，spring的web容器其实是Servlet容器内部的一个子容器，它没法管理Servlet容器中的对象；
                // 比如Servlet容器中的servlet、filter、listener等。
                // 传统的Servlet规范，要想向Servlet容器中注册servlet、filter、listener对象等，比如配置web.xml。
                // 然后实现Servlet规范的web服务器厂商（比如tomcat），会遵守Servlet规范，去解析web.xml，拿到web.xml中配置的serlvet，filter等，进行反射，然后将其注册到tomcat容器中。
                // Servlet3.x规范废弃了web.xml，提供了api侵入式来向Servlet容器中注册servlet、filter、listener对象等。
                // springboot依靠Servlet3.x的规范，内置了tomcat容器，并依靠Servlet3.x提供的api来显式的通过代码向Servlet容器中注册serlvet、filter等。
                // 这个方法后面单独写一篇文章分析。
                this.onRefresh();
                
                // 注册springboot传进来的所有 ApplicationListener 对象
                this.registerListeners();
                
                // 完成spring beanFactory工厂对象的初始化工作，注意，真正根据beanDefinition去反射创建bean对象就是在这一步进行的。
                // 前面磨磨唧唧的，都是在向springboot容器中注册钩子，然后回调钩子；
                // 回调钩子时又向spring容器中注册钩子，然后回调钩子....
                // 这一系列操作下来，spring容器中现在已经有了所有内置的bean定义和普通的业务bean定义了（其中一些特殊钩子类型的bean已经被优先创建并回调执行了其钩子逻辑了）
                // 那么剩下的普通bean，就是在下面这个方法里面根据bean定义反射创建并注册到spring容器中的。
                this.finishBeanFactoryInitialization(beanFactory);
                
                // 完成spring容器的启动
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

this.onRefresh();这个方法主要是将tomcat纳入到spring容器中来，请参考：[springboot与tomcat那点事儿](https://zhaoehcode.gitee.io/2022/01/05/BeanPostProcessor/)   

## 4.2 obtainFreshBeanFactory
（1）获取当前spring容器中的beanFactory工厂对象
```java
ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
```
（2）obtainFreshBeanFactory方法：
```java
    protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
        // 为beanFactory工厂设置一些属性，不重要
        this.refreshBeanFactory();
        // 获取当前spring容器中的beanFactory工厂并返回
        return this.getBeanFactory();
    }
```
（3）GenericApplicationContext类的getBeanFactory方法：
```java
    public final ConfigurableListableBeanFactory getBeanFactory() {
        // 直接返回beanFactory
        return this.beanFactory;
    }
```
总结：  
我们知道，spring容器在实例化的时候就已经初始化了其中的beanFactory工厂，是DefaultListableBeanFactory类型的beanFactory，此处只是返回该beanFactory对象而已。  

## 4.3 prepareBeanFactory
（1）准备beanFactory
```java
this.prepareBeanFactory(beanFactory);
```
（2）prepareBeanFactory方法：
```java
    protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        // 为beanFactory设置classLoader
        beanFactory.setBeanClassLoader(this.getClassLoader());
        if (!shouldIgnoreSpel) {
            beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
        }

        beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, this.getEnvironment()));
        
        // 重点，为beanFactory注册一个BeanPostProcessor钩子,ApplicationContextAwareProcessor
        beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
        
        beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
        beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
        beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
        beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
        beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
        beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
        beanFactory.ignoreDependencyInterface(ApplicationStartupAware.class);
        beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
        beanFactory.registerResolvableDependency(ResourceLoader.class, this);
        beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
        beanFactory.registerResolvableDependency(ApplicationContext.class, this);
        
        // 注册一个 BeanPostProcessor 钩子，ApplicationListenerDetector
        beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));
        
        if (!NativeDetector.inNativeImage() && beanFactory.containsBean("loadTimeWeaver")) {
            beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
            beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
        }

        if (!beanFactory.containsLocalBean("environment")) {
            beanFactory.registerSingleton("environment", this.getEnvironment());
        }

        if (!beanFactory.containsLocalBean("systemProperties")) {
            beanFactory.registerSingleton("systemProperties", this.getEnvironment().getSystemProperties());
        }

        if (!beanFactory.containsLocalBean("systemEnvironment")) {
            beanFactory.registerSingleton("systemEnvironment", this.getEnvironment().getSystemEnvironment());
        }

        if (!beanFactory.containsLocalBean("applicationStartup")) {
            beanFactory.registerSingleton("applicationStartup", this.getApplicationStartup());
        }

    }
```

<b>扩展：BeanPostProcessor 钩子用来干啥？   </b>   
请参考：[beanPostProcessor简介](https://zhaoehcode.gitee.io/2022/01/05/BeanPostProcessor/)   

（3）new ApplicationContextAwareProcessor(this)
重点看下，上面为beanFactory工厂对象注册的 ApplicationContextAwareProcessor 对象，这是spring的BeanPostProcessor钩子接口的一个内置实现。  
类继承结构如下：  
```java
class ApplicationContextAwareProcessor implements BeanPostProcessor
```
其实现了BeanPostProcessor钩子接口的postProcessBeforeInitialization方法：
```java
    @Nullable
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        // 判断传入的bean是否是指定的类型，如果不是，则直接返回原始bean，如果是则做处理。
        if (!(bean instanceof EnvironmentAware) && !(bean instanceof EmbeddedValueResolverAware) && !(bean instanceof ResourceLoaderAware) && !(bean instanceof ApplicationEventPublisherAware) && !(bean instanceof MessageSourceAware) && !(bean instanceof ApplicationContextAware) && !(bean instanceof ApplicationStartupAware)) {
            return bean;
        } else {
            AccessControlContext acc = null;
            if (System.getSecurityManager() != null) {
                acc = this.applicationContext.getBeanFactory().getAccessControlContext();
            }

            if (acc != null) {
                AccessController.doPrivileged(() -> {
                    this.invokeAwareInterfaces(bean);
                    return null;
                }, acc);
            } else {
                // 对原始bean做处理
                this.invokeAwareInterfaces(bean);
            }

            return bean;
        }
    }
```
（4）invokeAwareInterfaces方法：
```java
  private void invokeAwareInterfaces(Object bean) {
        if (bean instanceof EnvironmentAware) {
            // 如果bean是EnvironmentAware类型，则回调执行setEnvironment方法
            ((EnvironmentAware)bean).setEnvironment(this.applicationContext.getEnvironment());
        }

        if (bean instanceof EmbeddedValueResolverAware) {
            ((EmbeddedValueResolverAware)bean).setEmbeddedValueResolver(this.embeddedValueResolver);
        }

        if (bean instanceof ResourceLoaderAware) {
            ((ResourceLoaderAware)bean).setResourceLoader(this.applicationContext);
        }

        if (bean instanceof ApplicationEventPublisherAware) {
            ((ApplicationEventPublisherAware)bean).setApplicationEventPublisher(this.applicationContext);
        }

        if (bean instanceof MessageSourceAware) {
            ((MessageSourceAware)bean).setMessageSource(this.applicationContext);
        }

        if (bean instanceof ApplicationStartupAware) {
            ((ApplicationStartupAware)bean).setApplicationStartup(this.applicationContext.getApplicationStartup());
        }

        if (bean instanceof ApplicationContextAware) {
            // 最常用的 ApplicationContextAware 钩子，就是在这个注册的
            ((ApplicationContextAware)bean).setApplicationContext(this.applicationContext);
        }

    }
```
总结：  
在 prepareBeanFactory 方法阶段，注册了一个BeanPostProcessor，是spring内部实现的一个BeanPostProcessor，ApplicationContextAwareProcessor对象。    
这个ApplicationContextAwareProcessor内部实现了回调逻辑，而回调逻辑的核心做了如下几件事情：    
a.正在操作的bean是不是EnvironmentAware类型，如果是，则回调其setEnvironment方法。  
b.如果是EmbeddedValueResolverAware类型，则回调其setEmbeddedValueResolver方法。  
c.如果是ResourceLoaderAware类型，则回调其setResourceLoader方法。  
d.如果是ApplicationEventPublisherAware类型， 则回调其setApplicationEventPublisher方法。  
e.如果是MessageSourceAware类型，则回调其setMessageSource方法。  
f.如果是ApplicationStartupAware类型，则回调其setApplicationStartup方法。  
g.如果是ApplicationContextAware类型，则回调其 setApplicationContext 方法。  
因此可以得出结论：  
通过往spring容器中注册一个spring内部实现的一个BeanPostProcessor处理器-->ApplicationContextAwareProcessor对象，来间接地注册了7种不同类型的Aware接口回调逻辑到容器中了。    
<font color="#dc143c"><b><u>第5个向spring容器注册的钩子接口：ApplicationContextAwareProcessor实现（来源：spring）</u></b></font>    
其间接注册了如下的钩子接口：    
<b><u><font color="#dc143c">第6个向spring容器注册的钩子接口：EnvironmentAware（来源：spring）</font></u></b>      
<b><u><font color="#dc143c">第7个向spring容器注册的钩子接口：EmbeddedValueResolverAware（来源：spring）</font></u></b>   
<b><u><font color="#dc143c">第8个向spring容器注册的钩子接口：ResourceLoaderAware（来源：spring）</font></u></b>     
<b><u><font color="#dc143c">第9个向spring容器注册的钩子接口：ApplicationEventPublisherAware（来源：spring）</font></u></b>   
<b><u><font color="#dc143c">第10个向spring容器注册的钩子接口：MessageSourceAware（来源：spring）</font></u></b>      
<b><u><font color="#dc143c">第11个向spring容器注册的钩子接口：ApplicationStartupAware（来源：spring）</font></u></b>     
<b><u><font color="#dc143c">第12个向spring容器注册的钩子接口：ApplicationContextAware（来源：spring）</font></u></b>      

## 4.4 postProcessBeanFactory
（1）对beanFactory工厂对象的后置处理
```java
this.postProcessBeanFactory(beanFactory);
```
（2）AnnotationConfigServletWebServerApplicationContext类的postProcessBeanFactory方法：
```java
    protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        // 执行父类的方法
        super.postProcessBeanFactory(beanFactory);
        
        // 如果指定了basePackages扫描包路径，则对目标包进行扫描
        if (this.basePackages != null && this.basePackages.length > 0) {
            this.scanner.scan(this.basePackages);
        }

        // 如果注解容器不为空则注册
        if (!this.annotatedClasses.isEmpty()) {
            this.reader.register(ClassUtils.toClassArray(this.annotatedClasses));
        }

    }
```
（3）super.postProcessBeanFactory
ServletWebServerApplicationContext类中的postProcessBeanFactory方法：  
```java
    protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        // 向beanFactory对象注册一个BeanPostProcessor对象->WebApplicationContextServletContextAwareProcessor
        beanFactory.addBeanPostProcessor(new WebApplicationContextServletContextAwareProcessor(this));
        beanFactory.ignoreDependencyInterface(ServletContextAware.class);
        this.registerWebApplicationScopes();
    }
```
（4）我们看下注册的 WebApplicationContextServletContextAwareProcessor 对象覆盖的回调逻辑具体是啥：
```java
public class WebApplicationContextServletContextAwareProcessor extends ServletContextAwareProcessor
```
继续跟进父类ServletContextAwareProcessor：
```java
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (this.getServletContext() != null && bean instanceof ServletContextAware) {
            ((ServletContextAware)bean).setServletContext(this.getServletContext());
        }

        if (this.getServletConfig() != null && bean instanceof ServletConfigAware) {
            ((ServletConfigAware)bean).setServletConfig(this.getServletConfig());
        }

        return bean;
    }
```
从上可以看出：  
注册的这个构造的回调逻辑实际上是向容器中的bean注入servletContext上下文和servletConfig对象。    
<font color="#dc143c"><b><u>第13个注册的钩子接口：WebApplicationContextServletContextAwareProcessor（来源：spring）</u></b></font>   


## 4.5 invokeBeanFactoryPostProcessors
（1）执行beanFactoryPostProcessor后置钩子
```java
    protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
        PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, this.getBeanFactoryPostProcessors());
        if (!NativeDetector.inNativeImage() && beanFactory.getTempClassLoader() == null && beanFactory.containsBean("loadTimeWeaver")) {
            // 注册一个 BeanPostProcessor - LoadTimeWeaverAwareProcessor
            beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
            beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
        }

    }
```
（2）PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors方法：
```java
    // 这个方法是扫描bean定义并注册beanDefinition的核心方法。
    public static void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
        Set<String> processedBeans = new HashSet();
        ArrayList regularPostProcessors;
        ArrayList registryProcessors;
        int var9;
        ArrayList currentRegistryProcessors;
        String[] postProcessorNames;
        if (beanFactory instanceof BeanDefinitionRegistry) {
            BeanDefinitionRegistry registry = (BeanDefinitionRegistry)beanFactory;
            regularPostProcessors = new ArrayList();
            registryProcessors = new ArrayList();
            Iterator var6 = beanFactoryPostProcessors.iterator();

            while(var6.hasNext()) {
                BeanFactoryPostProcessor postProcessor = (BeanFactoryPostProcessor)var6.next();
                if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
                    BeanDefinitionRegistryPostProcessor registryProcessor = (BeanDefinitionRegistryPostProcessor)postProcessor;
                    registryProcessor.postProcessBeanDefinitionRegistry(registry);
                    registryProcessors.add(registryProcessor);
                } else {
                    regularPostProcessors.add(postProcessor);
                }
            }

            currentRegistryProcessors = new ArrayList();
            postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
            String[] var16 = postProcessorNames;
            var9 = postProcessorNames.length;

            int var10;
            String ppName;
            for(var10 = 0; var10 < var9; ++var10) {
                ppName = var16[var10];
                if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
                    currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                    processedBeans.add(ppName);
                }
            }

            sortPostProcessors(currentRegistryProcessors, beanFactory);
            registryProcessors.addAll(currentRegistryProcessors);
            invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
            currentRegistryProcessors.clear();
            postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
            var16 = postProcessorNames;
            var9 = postProcessorNames.length;

            for(var10 = 0; var10 < var9; ++var10) {
                ppName = var16[var10];
                if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
                    currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                    processedBeans.add(ppName);
                }
            }

            sortPostProcessors(currentRegistryProcessors, beanFactory);
            registryProcessors.addAll(currentRegistryProcessors);
            invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
            currentRegistryProcessors.clear();
            boolean reiterate = true;

            while(reiterate) {
                reiterate = false;
                postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
                String[] var19 = postProcessorNames;
                var10 = postProcessorNames.length;

                for(int var26 = 0; var26 < var10; ++var26) {
                    String ppName = var19[var26];
                    if (!processedBeans.contains(ppName)) {
                        currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                        processedBeans.add(ppName);
                        reiterate = true;
                    }
                }

                sortPostProcessors(currentRegistryProcessors, beanFactory);
                registryProcessors.addAll(currentRegistryProcessors);
                invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
                currentRegistryProcessors.clear();
            }

            invokeBeanFactoryPostProcessors((Collection)registryProcessors, (ConfigurableListableBeanFactory)beanFactory);
            invokeBeanFactoryPostProcessors((Collection)regularPostProcessors, (ConfigurableListableBeanFactory)beanFactory);
        } else {
            invokeBeanFactoryPostProcessors((Collection)beanFactoryPostProcessors, (ConfigurableListableBeanFactory)beanFactory);
        }

        String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);
        regularPostProcessors = new ArrayList();
        registryProcessors = new ArrayList();
        currentRegistryProcessors = new ArrayList();
        postProcessorNames = postProcessorNames;
        int var20 = postProcessorNames.length;

        String ppName;
        for(var9 = 0; var9 < var20; ++var9) {
            ppName = postProcessorNames[var9];
            if (!processedBeans.contains(ppName)) {
                if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
                    regularPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
                } else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
                    registryProcessors.add(ppName);
                } else {
                    currentRegistryProcessors.add(ppName);
                }
            }
        }

        sortPostProcessors(regularPostProcessors, beanFactory);
        invokeBeanFactoryPostProcessors((Collection)regularPostProcessors, (ConfigurableListableBeanFactory)beanFactory);
        List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList(registryProcessors.size());
        Iterator var21 = registryProcessors.iterator();

        while(var21.hasNext()) {
            String postProcessorName = (String)var21.next();
            orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
        }

        sortPostProcessors(orderedPostProcessors, beanFactory);
        invokeBeanFactoryPostProcessors((Collection)orderedPostProcessors, (ConfigurableListableBeanFactory)beanFactory);
        List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList(currentRegistryProcessors.size());
        Iterator var24 = currentRegistryProcessors.iterator();

        while(var24.hasNext()) {
            ppName = (String)var24.next();
            nonOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
        }

        invokeBeanFactoryPostProcessors((Collection)nonOrderedPostProcessors, (ConfigurableListableBeanFactory)beanFactory);
        beanFactory.clearMetadataCache();
    }
```
