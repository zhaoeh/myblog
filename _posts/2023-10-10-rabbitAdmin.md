---
layout:     post
title:      rabbitAdmin主要干啥
subtitle:   从源码层面分析RabbitAdmin
categories: [MQ系列]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. RabbitAdmin
在spring整合rabbitMQ中，已经接触了这个类。   
它主要用来管理rabbitMQ服务器端资源的，比如操作rabbitMQ服务器，去创建Exchange、Queue以及Binding等。   

我们从源码层面来解读一下：   
首先告诉大家，spring中对于交换机、队列、Binding等一系列的创建实际上是委托AmqpAdmin对象去做的。   
```java
package org.springframework.amqp.core;

import java.util.Properties;
import org.springframework.lang.Nullable;

public interface AmqpAdmin {
    void declareExchange(Exchange var1);

    boolean deleteExchange(String var1);

    @Nullable
    Queue declareQueue();

    @Nullable
    String declareQueue(Queue var1);

    boolean deleteQueue(String var1);

    void deleteQueue(String var1, boolean var2, boolean var3);

    void purgeQueue(String var1, boolean var2);

    int purgeQueue(String var1);

    void declareBinding(Binding var1);

    void removeBinding(Binding var1);

    @Nullable
    Properties getQueueProperties(String var1);

    @Nullable
    QueueInformation getQueueInfo(String var1);

    default void initialize() {
    }
}

```
这个接口主要就是用来操作rabbitMQ服务器的，比如创建交换机、队列、绑定关系等。   
这个接口在spring中有一个实现：   
org.springframework.amqp.rabbit.core.RabbitAdmin    
RabbitAdmin的类结构：   
```java
public class RabbitAdmin implements AmqpAdmin, ApplicationContextAware, ApplicationEventPublisherAware, BeanNameAware, InitializingBean 
```
RabbitAdmin的底层实现：   
（1）从 Spring 容器中获取 Exchange、Bingding、Routingkey 以及Queue 的 @Bean 声明   
（2）然后使用 rabbitTemplate 的 execute 方法进行执行对应的声明、修改、删除等一系列 RabbitMQ 基础功能操作。例如添加交换机、删除一个绑定、清空一个队列里的消息等等    

类图如下：   
![](/images/myBlog/2023-10-10-rabbitAdmin-class.png)   
RabbitAdmin实现了5个接口：   
```youtrack
AmqpAdmin
ApplicationContextAware
ApplicationEventPublisherAware
BeanNameAware
InitializingBean

```
RabbitAdmin借助于 ApplicationContextAware 和 InitializingBean来获取我们在配置类中声明的exchange, queue, binding beans等信息并调用channel的相应方法来声明。   
（1）首先,RabbitAdmin借助于ApplicationContextAware来获取ApplicationContext applicationContext   
（2）然后，借助于InitializingBean以及上面的applicationContext来实现rabbitMQ entity的声明   

这里面重点是InitializingBean接口，RabbitAdmin对象被实例化之后，会回调初始化方法afterPropertiesSet()。    
对于容器中声明的Exchange、Queue、Binding等的创建就是在这个方法里面完成的。   
```java
public void afterPropertiesSet() {
        Object var1 = this.lifecycleMonitor;
        synchronized(this.lifecycleMonitor) {
            if (!this.running && this.autoStartup) {
                if (this.retryTemplate == null && !this.retryDisabled) {
                    this.retryTemplate = new RetryTemplate();
                    this.retryTemplate.setRetryPolicy(new SimpleRetryPolicy(5));
                    ExponentialBackOffPolicy backOffPolicy = new ExponentialBackOffPolicy();
                    backOffPolicy.setInitialInterval(1000L);
                    backOffPolicy.setMultiplier(2.0D);
                    backOffPolicy.setMaxInterval(5000L);
                    this.retryTemplate.setBackOffPolicy(backOffPolicy);
                }

                if (this.connectionFactory instanceof CachingConnectionFactory && ((CachingConnectionFactory)this.connectionFactory).getCacheMode() == CacheMode.CONNECTION) {
                    this.logger.warn("RabbitAdmin auto declaration is not supported with CacheMode.CONNECTION");
                } else {
                    AtomicBoolean initializing = new AtomicBoolean(false);
                    
                    // 这个连接监听器是关键，它向ConnectionFactory工厂中注册了一个ConnectionListener
                    this.connectionFactory.addConnectionListener((connection) -> {
                        if (initializing.compareAndSet(false, true)) {
                            try {
                                if (this.retryTemplate != null) {
                                    this.retryTemplate.execute((c) -> {
                                        this.initialize();
                                        return null;
                                    });
                                } else {
                                    this.initialize();
                                }
                            } finally {
                                initializing.compareAndSet(true, false);
                            }

                        }
                    });
                    this.running = true;
                }
            }
        }
    }
```
org.springframework.amqp.rabbit.connection.ConnectionListener：   
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.amqp.rabbit.connection;

import com.rabbitmq.client.ShutdownSignalException;

@FunctionalInterface
public interface ConnectionListener {
    void onCreate(Connection var1);

    default void onClose(Connection connection) {
    }

    default void onShutDown(ShutdownSignalException signal) {
    }

    default void onFailed(Exception exception) {
    }
}

```
这个rabbitMQ连接监听器是个函数式接口，从它的方法定义可以看出它是一个运行监听器。    
外部传入一个lambda表达式来实现onCreate()方法，当与rabbitMQ服务器创建连接时会回调该方法，执行里面的逻辑。    

<font color="#dc143c"><b>注意上面的分析是关键，这个监听器只有在应用和rabbitMQ服务器创建连接的时候才会回到执行onCreate()方法。这也是为什么在之前的案例中，发现仅仅通过@Bean生命了队列交换机，启动spring容器，发现控制台根本没有创建对应队列的原因，因为此时没有和rabbitMQ服务器建立连接，因此这个方法不会被触发。</b></font>      

好了，当创建rabbitMQ连接时，onCreate()方法才会被触发执行。    
我们看下传入的Lambda表达式到底是啥：   
```java
                    AtomicBoolean initializing = new AtomicBoolean(false);
                    this.connectionFactory.addConnectionListener((connection) -> {
                        if (initializing.compareAndSet(false, true)) {
                            try {
                                if (this.retryTemplate != null) {
                                    this.retryTemplate.execute((c) -> {
                                        this.initialize();
                                        return null;
                                    });
                                } else {
                                    this.initialize();
                                }
                            } finally {
                                initializing.compareAndSet(true, false);
                            }

                        }
                    });
                    this.running = true;
```    
关键就是下面这段：   
```java
this.retryTemplate.execute((c) -> {
    this.initialize();
    return null;
```
然后看下initialize()方法做了什么事情？   
```java
public void initialize() {
    if (this.applicationContext == null) {
        this.logger.debug("no ApplicationContext has been set, cannot auto-declare Exchanges, Queues, and Bindings");
    } else {
        this.logger.debug("Initializing declarations");
        // 从spring容器中获取所有Exchange类型的bean
        Collection<Exchange> contextExchanges = new LinkedList(this.applicationContext.getBeansOfType(Exchange.class).values());
        // 获取所有Queue类型的bean
        Collection<Queue> contextQueues = new LinkedList(this.applicationContext.getBeansOfType(Queue.class).values());
        // 获取所有Binding类型的Bean
        Collection<Binding> contextBindings = new LinkedList(this.applicationContext.getBeansOfType(Binding.class).values());
        
        // 获取所有 DeclarableCustomizer 类型的bean，这个接口是个函数式接口，继承Function函数，传入一个Declarable类型对象，返回一个Declarable类型对象
        // 主要用来创建自定义的声明
        Collection<DeclarableCustomizer> customizers = this.applicationContext.getBeansOfType(DeclarableCustomizer.class).values();
        
        // 将所有的交换机、队列和绑定，通过这个方法去聚合
        this.processDeclarables(contextExchanges, contextQueues, contextBindings);
        
        // 对聚合后形成的3种类型的集合，分别做一些过滤操作，具体逻辑读者可以自己研究下
        Collection<Exchange> exchanges = this.filterDeclarables(contextExchanges, customizers);
        Collection<Queue> queues = this.filterDeclarables(contextQueues, customizers);
        Collection<Binding> bindings = this.filterDeclarables(contextBindings, customizers);
        
        // 下面这个大循环，就一件事情，遍历上面的exchanges、queues和bindings
        // 然后调用rabbitTemplate 的execute() 方法去创建
        Iterator var8 = exchanges.iterator();
        while(true) {
            Exchange exchange;
            do {
                if (!var8.hasNext()) {
                    var8 = queues.iterator();
                    while(true) {
                        Queue queue;
                        do {
                            if (!var8.hasNext()) {
                                if (exchanges.size() == 0 && queues.size() == 0 && bindings.size() == 0 && this.manualDeclarables.size() == 0) {
                                    this.logger.debug("Nothing to declare");
                                    return;
                                }
                                this.rabbitTemplate.execute((channel) -> {
                                    this.declareExchanges(channel, (Exchange[])exchanges.toArray(new Exchange[exchanges.size()]));
                                    this.declareQueues(channel, (Queue[])queues.toArray(new Queue[queues.size()]));
                                    this.declareBindings(channel, (Binding[])bindings.toArray(new Binding[bindings.size()]));
                                    return null;
                                });
                                if (this.manualDeclarables.size() > 0) {
                                    Map var13 = this.manualDeclarables;
                                    synchronized(this.manualDeclarables) {
                                        this.logger.debug("Redeclaring manually declared Declarables");
                                        Iterator var15 = this.manualDeclarables.values().iterator();
                                        while(var15.hasNext()) {
                                            Declarable dec = (Declarable)var15.next();
                                            if (dec instanceof Queue) {
                                                this.declareQueue((Queue)dec);
                                            } else if (dec instanceof Exchange) {
                                                this.declareExchange((Exchange)dec);
                                            } else {
                                                this.declareBinding((Binding)dec);
                                            }
                                        }
                                    }
                                }
                                this.logger.debug("Declarations finished");
                                return;
                            }
                            queue = (Queue)var8.next();
                        } while(queue.isDurable() && !queue.isAutoDelete() && !queue.isExclusive());
                        if (this.logger.isInfoEnabled()) {
                            this.logger.info("Auto-declaring a non-durable, auto-delete, or exclusive Queue (" + queue.getName() + ") durable:" + queue.isDurable() + ", auto-delete:" + queue.isAutoDelete() + ", exclusive:" + queue.isExclusive() + ". It will be redeclared if the broker stops and is restarted while the connection factory is alive, but all messages will be lost.");
                        }
                    }
                }
                exchange = (Exchange)var8.next();
            } while(exchange.isDurable() && !exchange.isAutoDelete());
            if (this.logger.isInfoEnabled()) {
                this.logger.info("Auto-declaring a non-durable or auto-delete Exchange (" + exchange.getName() + ") durable:" + exchange.isDurable() + ", auto-delete:" + exchange.isAutoDelete() + ". It will be deleted by the broker if it shuts down, and can be redeclared by closing and reopening the connection.");
            }
        }
    }
}
```
processDeclarables 方法：   
```java
    private void processDeclarables(Collection<Exchange> contextExchanges, Collection<Queue> contextQueues, Collection<Binding> contextBindings) {
    
        // 获取容器中所有类型为Declarables的bean
        Collection<Declarables> declarables = this.applicationContext.getBeansOfType(Declarables.class, false, true).values();
        
        // 遍历所有的Declarables bean，然后分别判断其中的类型是交换机、还是队列、还是绑定关系
        // 将对应类型的Declarable聚合到对应的容器中
        declarables.forEach((d) -> {
            d.getDeclarables().forEach((declarable) -> {
                if (declarable instanceof Exchange) {
                    contextExchanges.add((Exchange)declarable);
                } else if (declarable instanceof Queue) {
                    contextQueues.add((Queue)declarable);
                } else if (declarable instanceof Binding) {
                    contextBindings.add((Binding)declarable);
                }

            });
        });
    }
```
核心步骤：   
（1）springboot自动配置RabbitAdmin，将其交给spring容器管理。   
（2）RabbitAdmin在实例化完成后，执行初始化方法afterPropertiesSet()。   
（3）该方法首先向rabbitMQ连接工厂中注册了一个连接监听器，这个连接监听器只有当客户端和rabbitMQ服务器创建连接时才回调执行 onCreate() 方法。   
（4）当客户端和rabbitMQ服务器建立连接时，执行 initialize() 方法。   
（5）initialize()方法从spring容器中分别获取5种类型的Bean:    
```youtrack
Exchange.class
Queue.class
Binding.class
DeclarableCustomizer.class
Declarables.class
```
（6）对这些bean重新做规整，将Exchange统一聚合，将Queue统一聚合，将Binding统一聚合。   
（7）聚合之后分别做一些逻辑过滤。   
（8）对聚合后的三种类型的集合，调用rabbitTemplate的execute()方法，分别执行对应的创建工作。   

到这里，springboot通过@Bean声明的方式就能自动创建Exchange、Queue和Binding的源码基本就分析完毕了。   

# 2. Declarables.class
从上面看到了一个新的类： Declarables    
这个类之前没见过，是干嘛的？   
这个类的源码也贼简单：   
```java
package org.springframework.amqp.core;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collection;
import java.util.List;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.stream.Collectors;
import java.util.stream.Stream;
import org.springframework.util.Assert;
import org.springframework.util.ObjectUtils;

public class Declarables {
    private final Collection<Declarable> declarables = new ArrayList();

    public Declarables(Declarable... declarables) {
        if (!ObjectUtils.isEmpty(declarables)) {
            this.declarables.addAll(Arrays.asList(declarables));
        }

    }

    public Declarables(Collection<Declarable> declarables) {
        Assert.notNull(declarables, "declarables cannot be null");
        this.declarables.addAll(declarables);
    }

    public Collection<Declarable> getDeclarables() {
        return this.declarables;
    }

    public <T> List<T> getDeclarablesByType(Class<T> type) {
        Stream var10000 = this.declarables.stream();
        type.getClass();
        return (List)var10000.filter(type::isInstance).map((dec) -> {
            return dec;
        }).collect(Collectors.toList());
    }

    public String toString() {
        return "Declarables [declarables=" + this.declarables + "]";
    }
}

```
可以看到，这个类就是一个聚合 Declarable 类型的容器！   
通过它可以持有一堆 Declarable 对象。   

那Declarable又是啥呢？    
实际上我们看了源码也能猜到，这个Declarable.class实际上就是对Exchange、Queue、Binding这三种类型的顶层抽象。    
说白了，Exchange、Queue、Binding这三种类型的顶层接口就是Declarable。    
源码如下：   
```java
package org.springframework.amqp.core;

import java.util.Collection;
import org.springframework.lang.Nullable;

public interface Declarable {
    boolean shouldDeclare();

    Collection<?> getDeclaringAdmins();

    boolean isIgnoreDeclarationExceptions();

    void setAdminsThatShouldDeclare(Object... var1);

    default void addArgument(String name, Object value) {
    }

    @Nullable
    default Object removeArgument(String name) {
        return null;
    }
}

```
这个接口的实现类如下：   
![](/images/myBlog/2023-10-10-rabbitAdmin-Declarable.png)   
看到这里，是不是恍然大悟了。    
原来我们一直创建的Exchange、Queue、Binding，它们其实都是一种类型，那就是  Declarable ！   

这意味着我们在@Bean中通过下面的方式声明是完全合理的：   
```java
    @Bean("myBootExchange01")
    public Declarable bootExchange01() {
        System.out.println("init exchange01...");
        return ExchangeBuilder.topicExchange(EXCHANGE_NAME).durable(false).build();
    }

    @Bean("myBootQueue01")
    public Declarable bootQueue01() {
        return QueueBuilder.durable(QUEUE_NAME).build();
    }

    @Bean
    public Declarable bindQueueToExchange(@Qualifier("myBootQueue01") Declarable queue, @Qualifier("myBootExchange01") Declarable exchange) {
        return BindingBuilder.bind((Queue) queue).to((Exchange) exchange).with(ROUTING_KEY).noargs();
    }
```
这看着也就是很简单了，但是实际操作中不建议将这些对应的Exchange、Queue、和Binding都使用Declarable去接收，因为这看起来没有业务含义，容易让人误解。   
还是应该是交换机就使用Exchange，是队列就使用Queue，是绑定关系就使用Binding去声明。   

那spring抽象出这么一个顶层类型，同时还搞了一个持有它的容器类 Declarables ，意欲何为？    

# 3. Declarables 和 Declarable 的使用
如果第一节的源码看的足够仔细，这里它的目的也能猜测个八九不离十。   
我们在通过spring容器生命Exchange、Queue、Binding时，容易发现两个比较尴尬的问题：   
（1）如果有很多个Exchange、Queue、Binding，那一个配置类中要通过@Bean声明好多个方法，几乎满篇都是。   
（2）如果我们的Exchange、Queue、Binding要通过某个配置项动态生成，比如有一个配置项通过;分隔了好多个Exchange Name，它期望容器根据它分割后的字符串数组个数来声明对应的Exchange，这种仅仅通过上面@Bean的方式根本行不同嘛。    

因此，Declarables 目的就是包装了一堆 Declarable 对象，我们只需要将我们要声明的所有Exchange、Queue、Binding，都统一实例化好放入 Declarable 容器中即可。   
而且，这里也给了我们自己实例化 Declarable 的便利，我们可以自己决定根据动态配置实例化多少个Declarable对象，最后聚合后通过@Bean声明Declarables的方式统一交给Spring容器。    
而这一点，是单独通过@Bean显式直接交给spring容器所无法实现的。    

## 3.1 springboot配置中增加配置项
```properties
spring.application.name=rabbitMQ-springboot

spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.virtual-host=/

# 分别增加一;分隔的多个交换机、队列和路由键名称
my.exchange.names=daisy_01;daisy_02;daisy_03
my.queue.names=eric.dan.01;eric.dan.02;eric.dan.03
my.routing.key.names=funing.01;funing.02;funing.03
```
在rabbitmq.demo01.RabbitConfiguration1 中增加 Declarables bean声明的方式，可以定制化、统一声明 Declarable。   

## 3.2 修改rabbitMQ配置类
```java
    @Value("${my.exchange.names}")
    private String myExchangeNames;

    @Value("${my.queue.names}")
    private String myQueueNames;

    @Value("${my.routing.key.names}")
    private String myRoutingKeyNames;

    @Bean
    public Declarables declarablesBath() {
        // 根据配置项，动态声明Exchange
        List<Exchange> myExchanges = Arrays.asList(myExchangeNames.split(";")).stream().map(name -> {
            Exchange exchange = ExchangeBuilder.topicExchange(EXCHANGE_NAME).durable(false).build();
            return exchange;
        }).collect(Collectors.toList());


        // 动态声明Queue
        List<Queue> myQueues = Arrays.asList(myQueueNames.split(";")).stream().map(name -> {
            Queue queue = QueueBuilder.durable(name).build();
            return queue;
        }).collect(Collectors.toList());

        // 动态声明 Binding
        // 此处为了简单，只将上面的第一个交换机和第一个队列，遍历的通过routing key绑定了
        List<Binding> myBinding = Arrays.asList(myRoutingKeyNames.split(";")).stream().map(name -> {
            Binding binding = BindingBuilder.bind(myQueues.get(0)).to(myExchanges.get(0)).with(name).noargs();
            return binding;
        }).collect(Collectors.toList());

        List<Declarable> all = new ArrayList<>();
        all.addAll(myExchanges);
        all.addAll(myQueues);
        all.addAll(myBinding);

        // 通过 declarables 统一聚合
        Declarables declarables = new Declarables();
        declarables.getDeclarables().addAll(all);
        return declarables;
    }
```
上面的注释已经写的很清楚了。    
Declarables、@Bean单独声明，@RabbitListener声明（标注后容器启动就会创建）、手动通过RabbitAdmin去创建，这些方式都可以使用，也可以组合使用。   