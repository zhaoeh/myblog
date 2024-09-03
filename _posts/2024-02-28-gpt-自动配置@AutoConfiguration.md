---
layout:     post
title:      springboot自动配置原理
subtitle:   springboot自动配置原理源码分析
categories: [gpt]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. springboot的自动配置
始于启动类的 @SpringBootApplication 注解。   

这个复合注解有 @EnableAutoConfiguration 注解。   

这个注解上标注了  @Import({AutoConfigurationImportSelector.class}) 注解。    

走到@Import注解了，就说明启动类上实际上标注了一个@Import注解了，在spring容器的启动过程中就会解析这个@Import注解。   

# 2. org.springframework.boot.autoconfigure.AutoConfigurationImportSelector
@Import({AutoConfigurationImportSelector.class})中核心的实现是：   
```java
org.springframework.boot.autoconfigure.AutoConfigurationImportSelector
```
在这个类中负责解析spring.factories文件中key为 org.springframework.boot.autoconfigure.EnableAutoConfiguratio 的所有实现的全限定名，作为当前classpath下的所有自动配置类实现。    

# 3. 解析classpath下的所有自动配置类，解析后的配置类顺序？   
默认情况下，spring.factories文件中所有自动配置类实现的顺序，加载到内存中，就是容器启动过程中从classpath下扫描加载到的顺序。    
这意味着，哪个自动配置类写在前面，哪个自动配置类就会被优先解析。    
但是，当多个jar包中的spring.factories之间的顺序，是不确定的，具体取决于spring的扫描器的扫描顺序。    

我们可以通过spring内部提供的注解来显式指定自动配置类的顺序。   
```java
@AutoConfigureBefore:表示当前自动配置类在某个自动配置类之前进行解析。   
AutoConfigureAfter：表示当前自动配置类在某个自动配置类之后进行解析。
```   
请注意：上面的注解从springboot 2.x 版本就有了，专门用来指定自动配置类的解析顺序的。    
自动配置类上通过标注上述注解显式指定和其他自动配置类的解析顺序。    
即便自动配置类上标注了上述注解，仍需要标注@Configuration注解来表示这个类是一个自动配置类。    
简而言之：    
@Configuration的作用是标识一个类是自动配置类，只有存在这个注解，这个类才会被spring容器进行扫描解析。   
@AutoConfigureBefore和AutoConfigureAfter注解只是在spring容器解析spring.factoies文件得到所有的自动配置类权限定名称后，对自动配置类权限定名进行排序而已。    

整体的流程是：   
扫描spring.factories文件得到所有自动配置类的权限定名称；   
根据@AutoConfigureBefore和AutoConfigureAfter指定的依赖关系进行排序；    
处理所有的权限定名称，转换成对应的class对象；   
进一步判断对象是否存在指定注解（@Component,@Configuration等），存在的话就进行处理，得到一个BeanDefinition对象。    

# 4.整体上来讲，springboot启动时是如何处理自动配置的？    
springboot扫描解析处理bean，核心类如下：   
```java
org.springframework.context.annotation.ConfigurationClassPostProcessor
```
上面那个类是入口类，本质上是一个实现了 org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor 接口的实现类。   
我们知道，org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor接口是spring容器启动早期的一个接口，它比BeanFactoryPostProcessor执行时机还早。    
主要用于向spring容器中遍历的注册每一个符合规则的BeanDefinition对象。      

在ConfigurationClassPostProcessor中，主要委托了一个default访问级别的 org.springframework.context.annotation.ConfigurationClassParser 对象，负责具体的解析对应的@ComponentScan、@Component、@Import等注解。        
想要了解这些注解被解析的顺序，可以详细阅读org.springframework.context.annotation.ConfigurationClassParser的源码。      
这个类被设计为default级别，意味着只允许同包级别的类进行调用。说白了就是不对外部暴露。      

这个类的核心流程是：    
1.处理@ComponentScan，得到一个扫描范围。   
2.找到这个扫描范围中标记了`@Component`、`@Service`、`@Repository`、`@Controller`的类，然后转换为beanDefinition。   
3.解析标注了@Configuration的类，并处理@Bean方式注册的beanDefinition。   
4.最后处理扫描范围内标注了@Import注解的类，这个注解导入了其他的类。    
上述整个过程会反复执行，直到获取到所有符合条件的beanDefinition对象。   

从上面大概的过程可以分析出来：    
一个应用在启动时，会优先处理当前应用中扫描范围内的`@Component`、`@Service`、`@Repository`、`@Controller`等组件。    
以及当前应用中的@Configuration标注的配置类。    
最后才是当前应用中通过@Import导入的外部配置类（这里就包括springboot自动配置类）。     

这也从源码角度说明了，为什么springboot的自动配置类永远会在当前应用本身的bean定义注册完毕之后才进行处理，就是因为自动配置类是通过@Import注解导入的。    
而导入的组件在流程中是最后才被处理的。   

# 5. 自动配置类顺序和条件注解
spring.factories中配置的所有的自动配置类，都会按照加载到JVM内存中的顺序（这个顺序有可能经过处理，注意上面的@AutoConfigureBefore和AutoConfigureAfter注解）挨个进行解析。     
这里尤其要注意了，自动配置类的解析顺序严重自动配置中的条件注解。    
而且还需要特别注意，自动配置类的解析顺序和容器中bean的实例化顺序（bean的注册顺序）没有必然的联系。    
自动配置类的解析是按照配置类顺序进行解析，且只会解析一次，解析的过程中会将符合条件的class对象转换为对应的beanDefinition对象注册到spring容器中。    
而bean的实例化顺序，则依赖于bean之间是否具备强依赖关系。    

为什么强调这种区别？    
有时候发现自动配置A中的a1对象命名已经进行实例化了，即spring容器中已经有了；    
但是自动配置B中的b1对象有个@ConditionalOnBean(A1.class)，结果发现b1对象始终不会注册。      
这里面的原因就是因为：自动配置B先于自动配置A被解析，解析时评估，此时容器中还没有a1的beanDefinition呢，因此直接将b1废弃了，即b1没有被转换成beanDefinition。    
当a1被创建后注册到spring容器中后，b1始终都不会被创建（解析阶段就评估过了，连beanDefinition都没有，还创建个毛啊）。       

这就说明了条件注解的评估大多数时候是在自动配置类的解析阶段进行评估的，因此，条件注解的评估结果和自动配置类的解析顺序之间存在严重的因果关系。   

# 6. 请注意springboot版本对自动配置的影响
## 6.1 区别一：加载方式的变化
在springboot 2.x版本中，自动配置都是基于扫描解析classpath下的META-INF中的spring.factories文件，找到其中的key为 org.springframework.boot.autoconfigure.EnableAutoConfiguration 的所有实现类的全限定名称。    
在springboot 2.7版本和之后的版本中，提供了一种全新的方式进行自动配置类的加载解析，通过在 META-INF中创建一个spring文件夹，然后创建一个名称为 org.springframework.boot.autoconfigure.AutoConfiguration.imports 的文件，在这个文件中进行自动配置类全限定名称的编写即可。   

## 6.2 加载方式源码的变更
自动配置类的加载过程源码在  org.springframework.boot.autoconfigure.AutoConfigurationImportSelector 中。
在springboot 2.7之前的版本中，解析自动配置类的源码如下：   
```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

在springboot2.7到springboot3.0之间的版本，源码如下：
```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = new ArrayList(SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader()));
    ImportCandidates.load(AutoConfiguration.class, this.getBeanClassLoader()).forEach(configurations::add);
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories nor in META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```
发现在这个阶段，除了从spring.factories文件中解析目标自动配置类的全限定名称外，还提供了一种全新的方式，从META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports文件中进行自动配置类的查找。    
这很明显是处于一个过渡版本。因为springboot官方已经宣布了，将从springboot 3.x版本开始全面废除spring.factories的方式进行自动配置类的配置，而改用META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports的方式。   

而在随后的springboot3.x版本中，已经彻底废除了之前的spring.factories方式，完全使用 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports 进行替代了，源码如下：   
```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = ImportCandidates.load(AutoConfiguration.class, this.getBeanClassLoader()).getCandidates();
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

请注意：在springboot 3.x中只是对自动配置类的方式进行了spring.factories废除，其他的配置依旧保持在spring.factories文件中进行配置，比如 org.springframework.context.ApplicationContextInitializer 的配置等，依旧需要在spring.factories中。   


