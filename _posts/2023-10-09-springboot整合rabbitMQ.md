---
layout:     post
title:      springboot整合rabbitMQ
subtitle:   我们将主要研究springboot整合MQ
categories: [MQ系列]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. springboot整合MQ和spring整合MQ有啥区别？   

# 2. springboot整合rabbitMQ
## 2.1 创建一个springboot工程
![](/images/myBlog/2023-10-09-springboot-rabbit-project.png)   

## 2.2 pom.xml导入rabbitMQ的stater
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.1</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>zeh.rabbitmq</groupId>
    <artifactId>springboot-rabbitmq</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>my springboot rabbit</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

    </dependencies>

    <repositories>
        <repository>
            <id>com.e-iceblue</id>
            <url>http://repo.e-iceblue.cn/repository/maven-public/</url>
        </repository>
    </repositories>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>

```
别小看 spring-boot-starter-amqp ，很多我们在spring这个rabbitMQ中需要做的配置，都是这个starter底层替我们自动配置了。    

## 2.3 创建启动类
rabbitmq.MySpringApplication   
```java
package rabbitmq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MySpringApplication {
    public static void main(String[] args) {
        SpringApplication.run(MySpringApplication.class, args);

    }
}

```

## 2.4 创建rabbitMQ的配置类
在这个配置类中，我们主要是用来声明交换机、队列、绑定关系。   
rabbitmq.demo01.RabbitConfiguration1
```java
package rabbitmq.demo01;

import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import static rabbitmq.demo01.RabbitConstants1.*;

@Configuration
public class RabbitConfiguration1 {

    // 声明一个非持久化的交换机
    @Bean("myBootExchange01")
    public Exchange bootExchange01() {
        System.out.println("init exchange01...");
        return ExchangeBuilder.topicExchange(EXCHANGE_NAME).durable(false).build();
    }

    // 声明一个持久化的队列
    @Bean("myBootQueue01")
    public Queue bootQueue01() {
        return QueueBuilder.durable(QUEUE_NAME).build();
    }

    // 声明队列和交换机的绑定关系
    @Bean
    public Binding bindQueueToExchange(@Qualifier("myBootQueue01") Queue queue, @Qualifier("myBootExchange01") Exchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(ROUTING_KEY).noargs();
    }
}

```

到目前为止，我们尝试启动运行一下程序。   
启动springboot服务后，日志输出了：   
```youtrack
init exchange01...
```
说明我们声明交换机的程序被执行了，那么我们现在去web管理台去看看：   
![](/images/myBlog/2023-10-09-springboot-console.png)    
<b>神奇的一幕出现了，你可劲儿刷新，发现Exchange、Queue和Binding，竟然都没有！！！    </b>      

这是怎么一回事？   
我们继续往下走。   

## 2.5 创建一个常量类
rabbitmq.demo01.RabbitConstants1   
```java
package rabbitmq.demo01;

public class RabbitConstants1 {

    public static final String EXCHANGE_NAME = "boot_exchange_01";
    public static final String QUEUE_NAME = "boot.queue.01";
    public static final String ROUTING_KEY = "boot.test.01";
}

```
一般都规范的将用到的Exchange、Queue以及Routing key的名称封装到一个常量类中。   

## 2.6 创建一个生产者
rabbitmq.demo01.SpringbootProducer1   
```java
package rabbitmq.demo01;

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

// 生产者1
@RestController
public class SpringbootProducer1 {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/sendMsg1")
    public String sendMessage() {
        rabbitTemplate.convertAndSend(RabbitConstants1.EXCHANGE_NAME, RabbitConstants1.ROUTING_KEY, "Hello rabbitMQ!!!");
        return "ok";
    }
}

```

## 2.7 重启容器，访问生产者
我们重新启动springboot容器，然后用postman访问生产者，通过rabbitTemplate发送一条信息到RabbitMQ服务器中。   
我们神奇发现，web控制台中竟然有了Exchange、Queue和Binding：   
![](/images/myBlog/2023-10-09-springboot-rabbit-exchange.png)    
![](/images/myBlog/2023-10-09-springboot-rabbit-queue.png)   
![](/images/myBlog/2023-10-09-springboot-rabbit-binding.png)    
这说明，springboot在创建Exchange、Queue以及Binding关系时，并不是通过@Bean声明后就创建了。    
底层真正通过rabbitMQ的java客户端去创建这些东西，是需要通过rabbitTemplate对象去操作的。    
在发送消息或者消费消息的时候才真正执行了这些创建操作。   

## 2.8 修改rabbit配置，重复声明相同名称的Exchange和Queue
我们在rabbitmq.demo01.RabbitConfiguration1 中重新生命一组bean，但是和上面的Exchange、Queue还有Binding完全一样。   
```java
    // 声明一个非持久化的交换机
    @Bean("myBootExchange02")
    public Exchange bootExchange02() {
        System.out.println("init exchange02...");
        return ExchangeBuilder.topicExchange(EXCHANGE_NAME).durable(false).build();
    }

    // 声明一个持久化的队列
    @Bean("myBootQueue02")
    public Queue bootQueue02() {
        return QueueBuilder.durable(QUEUE_NAME).build();
    }

    // 声明队列和交换机的绑定关系
    @Bean
    public Binding bindQueueToExchange02(@Qualifier("myBootQueue02") Queue queue, @Qualifier("myBootExchange02") Exchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(ROUTING_KEY).noargs();
    }
```
运行后，发现程序正常启动:   
```youtrack
init exchange01...
init exchange02...
```
打印了两条语句，说明bean被初始化了两次。   

调用生产者发送一条消息，观察web端：   
![](/images/myBlog/2023-10-09-springboot-rabbit-exchange.png)    
![](/images/myBlog/2023-10-09-springboot-rabbit-queue.png)   
![](/images/myBlog/2023-10-09-springboot-rabbit-binding.png)    
发现，当我们重复声明完全相同的Exchange、Queue还有Binding时，其实复用的是rabbitMQ服务器上已经存在的交换机、队列还有绑定关系。   
这一点和原生java客户端的表现完全一致！！！   

我们再来一组相同名称的Exchange和Queue，但修改其中Exchange的属性：   
```java
    // 声明一个持久化的交换机
    @Bean("myBootExchange03")
    public Exchange bootExchange03() {
        System.out.println("init exchange03...");
        return ExchangeBuilder.topicExchange(EXCHANGE_NAME).durable(true).build();
    }

    // 声明一个非持久化的队列
    @Bean("myBootQueue03")
    public Queue bootQueue03() {
        return QueueBuilder.nonDurable(QUEUE_NAME).build();
    }

    // 声明队列和交换机的绑定关系
    @Bean
    public Binding bindQueueToExchange03(@Qualifier("myBootQueue03") Queue queue, @Qualifier("myBootExchange03") Exchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(ROUTING_KEY).noargs();
    }
```
运行程序：   
```youtrack
init exchange01...
init exchange02...
init exchange03...
```
发现3组bean都成功声明了。    
我们调用生产者发送一条消息：   
```youtrack
com.rabbitmq.client.ShutdownSignalException: channel error; protocol method: #method<channel.close>(reply-code=406, reply-text=PRECONDITION_FAILED - inequivalent arg 'durable' for exchange 'boot_exchange_01' in vhost '/': received 'true' but current is 'false', class-id=40, method-id=10)
	at com.rabbitmq.client.impl.ChannelN.asyncShutdown(ChannelN.java:517) ~[amqp-client-5.13.1.jar:5.13.1]
	at com.rabbitmq.client.impl.ChannelN.processAsync(ChannelN.java:341) ~[amqp-client-5.13.1.jar:5.13.1]
	at com.rabbitmq.client.impl.AMQChannel.handleCompleteInboundCommand(AMQChannel.java:182) ~[amqp-client-5.13.1.jar:5.13.1]
	at com.rabbitmq.client.impl.AMQChannel.handleFrame(AMQChannel.java:114) ~[amqp-client-5.13.1.jar:5.13.1]
	at com.rabbitmq.client.impl.AMQConnection.readFrame(AMQConnection.java:739) ~[amqp-client-5.13.1.jar:5.13.1]
	at com.rabbitmq.client.impl.AMQConnection.access$300(AMQConnection.java:47) ~[amqp-client-5.13.1.jar:5.13.1]
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:666) ~[amqp-client-5.13.1.jar:5.13.1]
	at java.lang.Thread.run(Thread.java:745) [na:1.8.0_111]
```
我们发现报错了。   
通过前面的分析，我们清楚在生产者调用rabbitTemplate去发送消息的时候才真正的创建Exchange、Queue和Binding。   
因此当我们发送消息时，尝试创建第三组交换机、队列和绑定关系，发现报错了。   
```youtrack
inequivalent arg 'durable' for exchange 'boot_exchange_01' in vhost '/': received 'true' but current is 'false', class-id=40, method-id=10)
```
这个错误实际上就是java原生客户端抛出来的错误，它和原生客户端的反应一致。    
当我们创建同名的Exchange、Queue时，实际上底层复用的是rabbitMQ服务器上已经存在的。   
如果服务器不存在，则重新创建；   
如果服务器已经存在，就复用；   
但是我们不能修改Exchange、Queue的属性值，否则就无法复用。    
rabbitMQ不允许我们这样做。   

因此，这告诉我们一个现实：当我们通过客户端去创建Exchange、Queue时，必须要注意服务器上是否已经存在相同的。   
如果存在，那么我们定义的属性值必须和服务器上的保持一致。    
否则将报错。   
同理，创建绑定关系时，也必须要求Exchange和Queue必须在服务器上已经创建，否则绑定关系将创建失败。   


# 3. 创建消费者
截止到目前，我们还没有创建消费者，下面将演示如何创建消费者，并通过消费者异步消费。   
为了让程序正常启动，我们注释掉上面错误的第三组Bean:   
```java
    // 声明一个持久化的交换机
//    @Bean("myBootExchange03")
//    public Exchange bootExchange03() {
//        System.out.println("init exchange03...");
//        return ExchangeBuilder.topicExchange(EXCHANGE_NAME).durable(true).build();
//    }
//
//    // 声明一个非持久化的队列
//    @Bean("myBootQueue03")
//    public Queue bootQueue03() {
//        return QueueBuilder.nonDurable(QUEUE_NAME).build();
//    }
//
//    // 声明队列和交换机的绑定关系
//    @Bean
//    public Binding bindQueueToExchange03(@Qualifier("myBootQueue03") Queue queue, @Qualifier("myBootExchange03") Exchange exchange) {
//        return BindingBuilder.bind(queue).to(exchange).with(ROUTING_KEY).noargs();
//    }
```

重启容器：   
```youtrack
init exchange01...
init exchange02...
```
第三组报错的交换机、队列和绑定关系已经被删除了。   

## 3.1 springboot异步消费的第一种方式
@RabbitListener + @RabbitHandler    

编写一个消费者：   
rabbitmq.demo01.SpringbootConsumerA   
```java
package rabbitmq.demo01;

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

// 消费者A
@Component
@RabbitListener(queues = RabbitConstants1.QUEUE_NAME)
public class SpringbootConsumerA {

    @RabbitHandler
    public void consumeMessage1(String message){
        System.out.println("consumeMessageA:" + message);
    }
}

```
重启容器：   
```youtrack
init exchange01...
init exchange02...
```
postman访问生产者发送一条消息：   
```youtrack
consumeMessageA:Hello rabbitMQ!!!
```
客户端正常收到消息了。   

这种组合方式进行异步消费，需要注意：   
（1）@RabbitListener必须放在类上，且指定queues属性为已经存在的队列名称，如果队列不存在，启动将报错。   
（2）@RabbitHandler必须标注在方法上，且要求方法的参数类型必须和MQ服务器上的消息类型兼容，如果类型不兼容，将报错。假如目标队列上之前就有一个Map类型的消息存在，这里通过String类型去消费，那么容器在启动时就会一直报错。假如容器正常启动后，生产者发送了一个Map类型的消息，那么这个接收到消息将直接报错，类型不匹配。   


## 3.2 springboot异步消费的第二种方式



## 3.3 springboot异步消费的第三种方式


# 4. @RabbitListener来创建交换机、队列和绑定关系
我们用@RabbitListener来标注消费者方法时，从第三种方式可以知道，这种方式的健壮性特别强。    
它允许我们在注解中，就维护一组绑定关系。   
在绑定关系中还可以创建交换机和队列等。   
实际上它的原则和之前的原则完全一致：   
（1）当rabbitMQ服务器已经存在同名的交换机时，它指定的交换机实际上复用的是服务器上的交换机，此时要注意，交换机的属性必须和服务器上已存在的保持一直，否则容器启动将报错。如果服务器不存在交换机，此处则创建一个新的交换机。      
（2）队列的创建同理，如果服务器存在同名队列，则此处指定对接的属性必须和服务器上存在的保持一致；否则启动将报错。如果服务器不存在，此处将创建一个新的队列。       
（3）绑定关系也是一样，可以创建多组绑定关系，在创建绑定关系之前，就已经创建了对应的交换机和队列。   

这种方式创建的交换机和队列，什么时候被触发执行？    
类比前面通过配置Bean声明的方式，我们可以看到，配置中声明了Exchange、Queue和Binding，容器启动并不会创建它们，而是等生产者发送消息后才进行创建。    
那@RabbitListener的这种方式是什么时候创建呢？    
看案例：   
rabbitmq.demo01.SpringbootConsumerToCreate   
```java
package rabbitmq.demo01;

import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

// 通过@RabbitListener注解去创建Exchange、Queue和Binding
@Component
public class SpringbootConsumerToCreate {

    private final String PRIVATE_EXCHANGE_NAME = "boot_exchange_c";
    private final String PRIVATE_QUEUE_NAME = "boot.queue.c";
    private final String PRIVATE_ROUTING_KEY = "boot.test.c";

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = PRIVATE_QUEUE_NAME, durable = "false", autoDelete = "true"),
            exchange = @Exchange(value = PRIVATE_EXCHANGE_NAME, type = ExchangeTypes.DIRECT),
            key = PRIVATE_ROUTING_KEY
    ))
    public void consumeMessage(String message) {
        System.out.println("consumeMessage:" + message);
    }

}

```
启动容器：   
```youtrack
init exchange01...
init exchange02...
```
查看web控制台：   
![](/images/myBlog/2023-10-09-springboot-rabbit-queue-run.png)   
![](/images/myBlog/2023-10-09-springboot-rabbit-queue-run2.png)   
![](/images/myBlog/2023-10-09-springboot-rabbit-queue-run3.png)   
神奇的发现，它并不像配置Bean的方式，等到使用rabbitTemplate操作的时候再去创建。    
<font color="#dc143c"><b>通过@RabbitListener去创建Exchange、Queue和Binding，是在容器启动过程中就执行的。容器启动时会扫描该注解，然后使用它的配置去创建交换机、队列，然后创建对应的绑定关系。</b></font>    

# 5. Message.getMessageProperties可能为null吗？
`Message.getMessageProperties()` 方法在Spring中用于获取消息的属性。它返回的是一个 `MessageProperties` 对象，用于表示消息的属性，例如消息的头部信息、内容类型等。通常情况下，如果消息是通过Spring的消息发送机制发送的，那么 `getMessageProperties()` 不应该为null。但是，在某些情况下，如果消息对象没有设置属性或者消息不是通过Spring的消息发送机制发送的，可能会返回null。

因此，你在使用 `getMessageProperties()` 方法时最好进行空指针检查，确保在可能为null的情况下不会出现空指针异常。例如：

```java
MessageProperties messageProperties = message.getMessageProperties();
if (messageProperties != null) {
    // 处理消息属性
} else {
    // 处理无消息属性的情况
}
```

# 6. 我看它源码的toString方法就是直接使用的，为啥它的源码不怕空指针呢？
如果你看到 `Message.getMessageProperties().toString()` 的源码实现是直接使用的，而不会对返回的 `MessageProperties` 对象进行空指针检查，那么这可能是因为在这个具体的实现中，假设了调用 `getMessageProperties()` 方法一定会返回非空对象。

这种假设在某些情况下是合理的，比如在特定的消息发送环境中，确保消息对象始终具有属性信息。但是，在其他情况下，尤其是在处理外部传入的消息或者在不同环境中使用消息时，为空指针检查是很重要的，可以避免潜在的空指针异常。

如果你在自己的代码中使用 `Message.getMessageProperties().toString()`，建议添加空指针检查，以确保代码的健壮性和稳定性。

# 7. 不是，是Message对象的toString方法
哦，明白了！你是指 `Message` 对象的 `toString()` 方法。通常情况下，Spring 中的 `Message` 对象在调用 `toString()` 方法时不会出现空指针异常的情况，因为 `toString()` 方法的实现通常是比较简单的，不会涉及访问可能为null的属性或方法。

Spring 的 `Message` 对象一般用于消息传递，例如在消息队列、消息通道等场景中使用。`Message` 对象的 `toString()` 方法一般会简单地将消息内容转换为字符串输出，而不会涉及到属性对象的调用。

所以，当你调用 `Message.toString()` 时，通常不需要担心空指针异常问题，除非在你的特定实现中对 `Message` 类进行了特殊的定制或重写 `toString()` 方法的逻辑，才有可能导致空指针异常。

# 8. 我是说，Message对象中的toString方法里面，对messageProperties属性直接使用，为啥不怕空指针呢
在理解这个问题之前，需要考虑几个方面：

1. **默认实现不会返回null**：Spring 消息对象 `Message` 在 `toString()` 方法的默认实现中，会先检查 `messageProperties` 是否为null，如果不为null，则调用其 `toString()` 方法输出；如果为null，则直接返回一个空字符串或其他默认值。

2. **稳定的消息对象**：在正常使用的情况下，Spring 的消息传递机制会确保创建的 `Message` 对象是完整的，即使没有显式设置 `messageProperties`，也会有一个默认的空对象或默认值。

3. **内部实现保证不会出现空指针**：Spring 框架的实现通常会对可能出现空指针的情况进行检查和处理，特别是对于内部类和方法之间的调用关系，一般会确保不会因为空指针而导致异常。

因此，在大多数情况下，Spring 框架内部对于 `Message` 对象的 `toString()` 方法会进行适当的空指针检查和处理，以确保在使用过程中不会因为空指针而出现异常。

# 9. spring中的Message对象是amqp中的吗？专门用来封装rabbitmq的消息吗？
是的，Spring 中的 `Message` 对象是来自 `spring-amqp` 包的，它是专门用于封装 RabbitMQ 消息的类。在 Spring AMQP 框架中，`Message` 类用于表示 AMQP 消息（如 RabbitMQ 消息），包含消息的主体和消息属性。

`Message` 类的结构主要包括两个部分：

1. **消息主体（body）**：消息的实际内容，以字节数组的形式存储。
2. **消息属性（MessageProperties）**：包含消息的各种属性信息，例如消息的 `contentType`、`deliveryMode`、`headers` 等，这些属性对应于 RabbitMQ 的消息元数据。

Spring 使用 `Message` 类来标准化 AMQP 消息的处理，方便开发者在 Spring 应用中更好地发送和接收 RabbitMQ 消息。在构造消息时，`MessageProperties` 通常会被自动填充，因此在大部分场景中，`MessageProperties` 是非空的。

**示例：**

```java
byte[] messageBody = "Hello, RabbitMQ!".getBytes();
MessageProperties properties = new MessageProperties();
properties.setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN);
Message message = new Message(messageBody, properties);
```

在这个例子中，`Message` 对象被初始化时，`messageProperties` 属性也被赋予了值，因此 `toString()` 方法可以直接调用 `messageProperties.toString()`，而不需要担心空指针的问题。



