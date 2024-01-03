---
layout:     post
title:      SmartInitializingSingleton钩子
subtitle:   SmartInitializingSingleton钩子算是spring生命周期比较靠后的一个钩子了
categories: [springboot专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. SmartInitializingSingleton接口
SmartInitializingSingleton 是 Spring 框架中的一个接口，它的源码如下：   
```java
package org.springframework.beans.factory;

// 可以看到，它就是spring生命周期当中一个完全独立的钩子接口
// 没有和其他的BeanFactoryProcessor或者BeanPostProcessor等钩子接口有任何关系
public interface SmartInitializingSingleton {
    
    // 这个钩子只有一个抽象方法需要实现类实现
    // 从方法的命名就可以看出，这个方法的回调时机应该是：当容器中的某个单例bean完全被实例化完毕之后进行调用
    void afterSingletonsInstantiated();
}

```

# 2. SmartInitializingSingleton 钩子的回调时机
afterSingletonsInstantiated方法什么时候会被触发回调呢？        
SmartInitializingSingleton是spring 4.1中引入的新特效，与InitializingBean的功能类似，都是bean实例化后执行自定义初始化，都是属于spring bean生命周期的增强。      
但是，SmartInitializingSingleton的定义及触发方式方式上有些区别，它的定义不在当前的bean中。      

## 2.1 SmartInitializingSingleton是什么？
和 InitializingBean相比的话，InitializingBean 是在每一个 bean 初始化完成后调用；多例的情况下每初始化一次就调用一次。      
SmartInitializingSingleton 是所有的非延迟的、单例的 bean 都初始化后调用，只调用一次。如果是多例的bean实现，不会调用。      

说明：   
```youtrack
（1）实现SmartInitializingSingleton的bean 要是单例。      
（2）在所有非延迟的单例的bean初始化完成后调用。     

```
SmartInitializingSingleton 接口中的 afterSingletonsInstantiated() 方法将会在所有的非惰性单实例Bean初始化完成之后进行回调。可以避免早期初始化的意外副作用。SmartInitializingSingleton可以看作是在所有的Bean初始化完成结束后的InitializingBean接口的替代。      

## 2.2 SmartInitializingSingleton&InitializingBean的区别      
（1）SmartInitializingSingleton接口只能作用于非惰性单实例Bean，InitializingBean接口无此要求。      
（2）SmartInitializingSingleton接口是在所有非惰性单实例bean初始化完成之后进行激活回调，InitializingBean接口是在每一个Bean实例初始化完成之后进行激活回调。      

## 2.3 使用场景
主要应用场合就是在所有单例 bean 创建完成之后，可以在该回调中做一些事情。      

## 2.4 执行时机
SmartInitializingSingleton主要用于在Spring容器启动完成时进行扩展操作，即afterSingletonsInstantiated()；      
实现SmartInitializingSingleton接口的bean的作用域必须是单例，afterSingletonsInstantiated()才会触发；      
afterSingletonsInstantiated()触发执行时，非懒加载的单例bean已经完成实现化、属性注入以及相关的初始化操作；      
afterSingletonsInstantiated()的执行时机是在DefaultListableBeanFactory#preInstantiateSingletons()；       

扩展：关于Spring bean有七种作用域：默认singleton(单例)、prototype、request、session、globalSession、application、websocket；      
```youtrack
1、singleton(单例)：Spring容器只会创建一个bean对象；
2、prototype：每次获取bean都会重新创建一个bean对象；
3、request：对于每一个http请求，在同一个请求内Spring容器只会创建一个bean对象，若请求结束，bean也随之销毁；
4、session：在同一个http会话里，Spring容器只会创建一个bean对象，若传话结束，也随之销毁；
5、globalSession：globalSession作用域的效果与session作用域类似，但是只适用于基于portlet的web应用程序中
6、application：在servlet程序中，该作用域的bean将会作为ServletContext对象的属性，被全局访问，与singleton的区别就是，singleton作用域的bean在Spring容器中只一；application作用域的bean在ServletContex中唯一；
7、websocket：为每个websocket对象创建一个实例。仅在Web相关的ApplicationContext中生效。   

```
# 3.源码分析
还是老样子，前期的源码请参考其他文章，我们直接进入到AbstractApplicationContext#refresh()方法中开始分析：       
org.springframework.context.support.AbstractApplicationContext#refresh:     
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
                this.registerListeners();
                
                // finishBeanFactoryInitialization阶段，开始遍历spring容器中的所有beanDefinition，挨个的去实例化、填充属性、初始化 对应的非延迟加载的单例bean
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
org.springframework.context.support.AbstractApplicationContext#finishBeanFactoryInitialization:   
```java
    protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
        if (beanFactory.containsBean("conversionService") && beanFactory.isTypeMatch("conversionService", ConversionService.class)) {
            beanFactory.setConversionService((ConversionService)beanFactory.getBean("conversionService", ConversionService.class));
        }

        if (!beanFactory.hasEmbeddedValueResolver()) {
            beanFactory.addEmbeddedValueResolver((strVal) -> {
                return this.getEnvironment().resolvePlaceholders(strVal);
            });
        }

        String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
        String[] var3 = weaverAwareNames;
        int var4 = weaverAwareNames.length;

        for(int var5 = 0; var5 < var4; ++var5) {
            String weaverAwareName = var3[var5];
            this.getBean(weaverAwareName);
        }

        beanFactory.setTempClassLoader((ClassLoader)null);
        
        // 设置容器冻结，执行到这里，已经不会有新的bean definition进来了。
        // 设置冻结状态，表示可以从缓存中读取bean了。
        beanFactory.freezeConfiguration();
        
        // 准备开始所有非懒加载的单例bean的实例化了（反射指定构造器去实例化）
        beanFactory.preInstantiateSingletons();
    }
```

继续往下分析，实际上是调用了DefaultListableBeanFactory对象的preInstantiateSingletons方法。      
单独看这个方法，逻辑相对简单，主要做了两件事情：      
```youtrack
（1）单例bean的实例化、属性填充、相关初始化逻辑；
（2）找出所有实现了SmartInitializingSingleton钩子接口的实现对象，遍历并执行afterSingletonsInstantiated()方法。   

```
源码如下：       
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

                                    // 下面这个循环就是在遍历处理所有单例bean，判断是否是SmartInitializingSingleton类型
                                    // 如果是，则循环回调其中的 afterSingletonsInstantiated 方法
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
从上面源码分析可以得出：     
如果要在业务开发中使用SmartInitializingSingleton扩展点，需要特别注意实现这个接口的bean应该是非懒加载的单例bean；      
执行时机是在所有的bean完成实例化、属性注入、相关初始化操作以及BeanPostProcessor的postProcessAfterInitialization方法执行完毕后，才被触发执行。      