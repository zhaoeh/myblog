---
layout:     post
title:      spring的bean scope详解
subtitle:   spring管理bean对象，bean是存在作用域的
categories: [spring专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 什么是bean scope？
spring容器帮我们创建bean对象，有时候我们需要这个bean对象在整个spring容器中都是唯一的，有时候我们又希望每次使用时都是重新创建的新对象。   
spring对我们的这种需求也提供了支持，在spring中这种诉求叫做bean的作用域，xml在定义bean的时候，可以通过bean元素的scope属性来指定bean的作用域。   
比如：   
```xml
<bean id="createByFactoryBean" class="com.zeh.main.FactoryBeanImpl" scope="当前bean的作用域"/>
```
spring容器中scope常见有5种，singleton、prototype、request、session、application。   

# 2. 5种bean作用域详解
## 2.1 singleton作用域
当scope的取值为singleton的时候，整个spring容器中只会存在一个bean实例，通过容器多次查找bean的时候（调用BeanFactory的getBean方法或者bean之间自动注入依赖的bean对象的时候），返回的都是同一个bean对象。   
singleton是scope的默认值，所以spring容器中默认创建的bean对象都是单例的。   
通常spring容器在启动的时候，会将scope为singleton的bean提前创建好放在spring缓存容器中（有个特殊的情况，当bean的lazy被设置为true时，表示该bean为懒加载，那么在从容器中获取bean的时候才会创建），当从容器中获取时直接返回。   

案例：   
bean xml配置singleton作用域：   
```xml
<bean id="singletonBean" class="com.zeh.main.pojo.BeanScopeModel" scope="singleton">
    <constructor-arg index="0" value="singleton" />
</bean>
```
上述文件制定目标bean为BeanScopeModel，指定它的scope为singleton。并且通过构造方法的方式去反射实例化目标bean，传入了一个参数作为当前bean的作用域描述。   

BeanScopeModel代码:   
```java
public class BeanScopeModel {
    public BeanScopeModel(String beanScope) {
        System.out.println(String.format("create beanScopeModel，[scope = %s],[this = %s]", beanScope, this));
    }
}
```
上述构造方法中输出了一段文字，一会我们可以根据输出观察这个bean是什么时候创建的，是从容器中获取bean的时候创建的还是容器启动的时候就创建的。   

测试代码:   
```java
public class BeanScopeModelController {
    public static void main(String[] args) {
        System.out.println("spring容器准备启动...");
        String beanXml = "classpath:/spring/applicationContext.xml";
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext(beanXml);
        System.out.println("spring容器启动完成...");
        System.out.println("开始从容器中获取bean...");
        BeanScopeModel beanScopeModel1 = applicationContext.getBean("singletonBean", BeanScopeModel.class);
        BeanScopeModel beanScopeModel2 = applicationContext.getBean("singletonBean", BeanScopeModel.class);
        BeanScopeModel beanScopeModel3 = applicationContext.getBean("singletonBean", BeanScopeModel.class);
        System.out.println(String.format("beanScopeModel1 == beanScopeModel2 ? %s",(beanScopeModel1 == beanScopeModel2)));
        System.out.println(String.format("beanScopeModel1 == beanScopeModel3 ? %s",(beanScopeModel1 == beanScopeModel3)));
        System.out.println("从容器中获取bean结束...");
    }
}
```
测试代码中先启动了spring容器，然后根据spring上下文对象多次获取了同一个bean id。   

测试结果:   
```youtrack
spring容器准备启动...
create beanScopeModel，[scope = singleton],[this = com.zeh.main.pojo.BeanScopeModel@71318ec4]
spring容器启动完成...
开始从容器中获取bean...
beanScopeModel1 == beanScopeModel2 ? true
beanScopeModel1 == beanScopeModel3 ? true
从容器中获取bean结束...
```

结果分析:   
从结果可以看出来：   
1.  没有设置延迟加载的bean，都是在spring容器启动时就调用构造方法去实例化目标bean的，实例化好之后放在容器中缓存着。   
2.  对于单例bean，在获取bean实例对象时，是从缓存中获取到的同一个目标bean对象，因此验证了singleton的bean是同一个单例bean。   

单例bean的注意事项:   
单例bean是整个spring容器共享的，所以需要考虑到线程安全问题。   
之前在搞springmvc时，因为springmvc的controller默认是单例的，有些开开发者在controller中创建了一些成员变量，那么这些变量实际上就变成共享的了，controller可能会被很多线程同时访问，这些线程去并发修改controller中的共享变量，可能会出现数据错乱的问题。   
因此，在使用单例bean的时候要特别注意多线程安全问题。   

## 2.2 prototype作用域
如果一个bean的scope被设置成prototype类型的了，表示这个bean是多例的，通过容器每次获取的bean都是不同的实例。   
多例的bean是每次在获取时都会重新创建一个新的bean实例对象（也就是单例bean是容器启动时预加载缓存到容器中的，而多例bean是明确获取bean时才加载进行实例化，即懒加载）。   
```youtrack
懒加载机制只对单例bean有作用，对于多例bean设置懒加载没有意义，多例bean本身就是懒加载的。
懒加载只是延后了对象创建的时机，对象仍然是单例的。
```

bean xml配置:   
```xml
<bean id="singletonBean" class="com.zeh.main.pojo.BeanScopeModel" scope="prototype">
    <constructor-arg index="0" value="singleton" />
</bean>
```

执行结果:
```youtrack
spring容器准备启动...
spring容器启动完成...
开始从容器中获取bean...
create beanScopeModel，[scope = singleton],[this = com.zeh.main.pojo.BeanScopeModel@67784306]
create beanScopeModel，[scope = singleton],[this = com.zeh.main.pojo.BeanScopeModel@6ddf90b0]
create beanScopeModel，[scope = singleton],[this = com.zeh.main.pojo.BeanScopeModel@57536d79]
beanScopeModel1 == beanScopeModel2 ? false
beanScopeModel1 == beanScopeModel3 ? false
从容器中获取bean结束...
```

结果分析:   
从结果分析：   
1.  prototype的多例bean是延迟加载的，即容器启动不预加载，而是等到从容器中获取bean实例时才进行bean的加载创建。   
2.  多例bean每次都是重新创建一个新的bean对象，即多例bean不会放入spring缓存中。   

多例bean的注意事项:   
多例bean每次获取的时候都会重新创建，如果目标bean比较复杂，创建时间比较长，会影响系统的性能，而且频繁创建很多bean，对内存也是一种浪费，这一点需要注意。   

## 2.3 request、session、application作用域
request、session、application这三种bean scope作用域都是在spring web容器环境中才会有的。   
request作用域:   
当一个bean的作用域为request，表示在一次http请求中，一个bean对应一个实例；对每个http请求都会创建一个bean实例，request结束的时候，这个bean也就结束了。   
request作用域用在spring容器的web环境中，spring容器中有个web容器接口WebApplicationContext，这个里面对request作用域提供了支持，配置方式如下：   
```xml
<bean id="" class="" scope="request"/>
```

session作用域:   
session作用域和request作用域类似，也是用在web环境中，session级别共享的bean，每一个会话对应一个bean实例，不同的session对应不同的bean实例。   
```xml
<bean id="" class="" scope="session"/>
```

application作用域:   
全局web应用级别的作用域，也是在web环境中使用的，一个web应用程序对应一个bean实例，通常情况下和singleton效果类似。   
不过也有不一样的地方，singleton是每个spring容器中只有一个bean实例，一般我们的应用只有一个spring容器，但是一个web应用可以创建多个spring容器。   
不同的spring容器中可以存在同名的bean，当scope=application时，不管web应用中存在多少个spring容器，这个应用中同名的bean只有一个。   

# 3. 自定义scope
## 3.1 为啥要自定义scope？
有时候，spring内置的几种scope都无法满足我们的需求，这个时候我们可以自定义scope作用域。    
自定义scope作用域分为3步骤。   

## 3.2 如何自定义scope作用域？
### 3.2.1 第一步：实现scope接口
```java
package org.springframework.beans.factory.config;
import org.springframework.beans.factory.ObjectFactory;
public interface Scope {
    // 返回当前作用域中name对应的bean对象
    // name：需要检索的bean的名称
    // objectFactory：如果name对应的bean在当前作用域中没有找到，那么可以调用ObjectFactory来创建这个对象
    Object get(String name, ObjectFactory<?> objectFactory);
    // 将name对应的bean从当前作用域中移除
    Object remove(String name);
    // 用于注册销毁回调，如果想要销毁相应的对象，则由spring容器注册相应的销毁回调，而由自定义作用域选择是不是要销毁相应的对象
    void registerDestructionCallback(String name, Runnable callback);
    // 用于解析相应的上下文数据，比如request作用域将返回request中的属性
    Object resolveContextualObject(String var1);
    // 作用域的会话标识，比如session作用域将是sessionId
    String getConversationId();
}
```
### 3.2.2 第二步：将自定义的scope注册到容器
需要调用  org.springframework.beans.factory.config.ConfigurableBeanFactory 的registerScope()方法，该方法如下：   
```java
// 向容器中注册自定义的scope
// scopeName：自定义的scope名称
// scope：自定义的scope对象
void registerScope(String scopeName, Scope scope);
```
### 3.2.3 第三步：使用自定义的作用域
定义bean时，指定bean的scope属性为自定义的作用域名称。   

## 3.3 自定义scope作用域案例演示
### 3.3.1 需求
实现一个线程级别的bean作用域，同一个线程中同名的bean是同一个实例，不同线程中的bean是不同的实例。   
分析:   
需求中要求bean在同一个线程中是共享的，所以可以通过ThreadLocal来实现，ThreadLocal可以实现同一个线程中数据的共享。   

### 3.3.2 自定义ThreadScope
```java
package com.zeh.main.scope;
import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.config.Scope;
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;
/**
 * 功能描述 自定义本地线程级别的bean scope，不同的线程对象中的bean实例是不同的，同一个线程对象中的同名bean实例是同一个实例。
 *
 * @since 2021-05-07
 */
public class ThreadScope implements Scope {
    // 定义一个常量，表示当前自定义作用域的名称，在定义bean的时候可以给scope属性使用
    public static final String THREAD_SCOPE = "thread";
    // 定义一个ThreadLocal成员，用来作为每个线程对象中的缓存容器
    private ThreadLocal<Map<String, Object>> beanMap = new ThreadLocal() {
        @Override
        protected Map<String, Object> initialValue() {
            return new HashMap<String, Object>();
        }
    };
    @Override
    public Object get(String s, ObjectFactory<?> objectFactory) {
        Object object = beanMap.get().get(s);
        if (Objects.isNull(object)) {
            object = objectFactory.getObject();
            beanMap.get().put(s, object);
        }
        return object;
    }
    @Override
    public Object remove(String s) {
        return this.beanMap.get().remove(s);
    }
    @Override
    public void registerDestructionCallback(String s, Runnable runnable) {
        // bean作用域结束的时候调用该方法，用于bean清理
        System.out.println("当前bean的作用域结束，bean is " + s);
    }
    @Override
    public Object resolveContextualObject(String s) {
        return null;
    }
    @Override
    public String getConversationId() {
        return Thread.currentThread().getName();
    }
}
```

### 3.3.3 bean xml配置   
```xml
<bean id="threadBean" class="com.zeh.main.pojo.BeanScopeModel" scope="thread">
    <constructor-arg index="0" value="thread" />
</bean>
```

### 3.3.4 BeanScopeModel
```java
package com.zeh.main.pojo;
public class BeanScopeModel {
    public BeanScopeModel(String beanScope) {
        System.out.println(String.format("线程:%s create beanScopeModel，[scope = %s],[this = %s]", Thread.currentThread(), beanScope, this));
    }
}
```

### 3.3.5 测试程序
```java
package com.zeh.main.controller;
import com.zeh.main.scope.ThreadScope;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import java.util.concurrent.TimeUnit;

public class BeanScopeModelController {
    public static void main(String[] args) throws InterruptedException {
        String beanXml = "classpath:/spring/applicationContext.xml";
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext();
        // 设置配置文件位置
        applicationContext.setConfigLocation(beanXml);
        // 启动容器
        applicationContext.refresh();
        // 向容器中注册自定义的scope
        applicationContext.getBeanFactory().registerScope(ThreadScope.THREAD_SCOPE, new ThreadScope());
        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread() + ";" + applicationContext.getBean("threadBean"));
                System.out.println(Thread.currentThread() + ";" + applicationContext.getBean("threadBean"));
            }).start();
            TimeUnit.SECONDS.sleep(1);
        }
    }
}
```
### 3.3.6 测试结果
```youtrack
线程:Thread[Thread-0,5,main] create beanScopeModel，[scope = thread],[this = com.zeh.main.pojo.BeanScopeModel@17ebea32]
Thread[Thread-0,5,main];com.zeh.main.pojo.BeanScopeModel@17ebea32
Thread[Thread-0,5,main];com.zeh.main.pojo.BeanScopeModel@17ebea32
线程:Thread[Thread-1,5,main] create beanScopeModel，[scope = thread],[this = com.zeh.main.pojo.BeanScopeModel@10e827ac]
Thread[Thread-1,5,main];com.zeh.main.pojo.BeanScopeModel@10e827ac
Thread[Thread-1,5,main];com.zeh.main.pojo.BeanScopeModel@10e827ac
```
结果分析:   
可以看到，对于同一个线程创建的相同名称的bean，其是共享的，即为单例bean。   
而对于不同线程创建的bean，是不同的bean。   

# 4. 总结
1.  spring容器自带有2种作用域，分别是singleton和prototype。还有3种是spring web容器环境中才支持的request、session、application。   
2.  singleton是spring容器默认的作用域，一个spring容器中同名的bean实例只有一个，多次获取得到的是同一个bean；单例bean需要考虑线程安全问题。   
3.  prototype是多例的bean，每次从容器中获取同名的bean时都会重新创建一个新的bean实例；多例bean使用时要考虑对性能的影响。   
4.  一个web应用中可以有多个spring容器。   
5.  自定义scope分为3个步骤：实现Scope接口、将实现类注册到spring容器、bean xml中使用自定义的scope。   