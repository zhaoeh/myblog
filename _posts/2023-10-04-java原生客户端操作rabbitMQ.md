---
layout:     post
title:      java原生客户端操作rabbitMQ
subtitle:   通过java原生客户端代码去操作rabbitMQ
categories: [MQ系列]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 什么是rabbitMQ客户端？
对于中间件来说，我们下载安装的是它们的服务端代码，这些中间件都支持各种语言开发的客户端代码去操作它的服务器。   
就比如zookeeper、redis等，它们都有对应的客户端代码去连接它们，进一步操作它们。   
rabbitMQ也是一样。   
因为我们的主业是JAVA，所以我们就使用java语言编写的客户端去操作rabbitMQ。    
这些客户端代码已经是被开源实现好了的，一般来讲，一个中间件产生后，都有对应的客户端代码供我们应用直接去使用。   
我们只需要导入它们的客户端jar包即可以通过客户端API去远程操作这些中间件。   

下面我们将演示，如何通过rabbitMQ提供的原生java客户端去操作它。   

# 2. 使用java语言编写的rabbitMQ客户端去操作
## 2.1 创建一个maven工程
在pom.xml中引入rabbitMQ的java客户端依赖：
```xml
    <dependencies>
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>3.6.5</version>
        </dependency>
    </dependencies>
```
## 2.2 创建生产者
rabbitMQ.demo01.MyProducer1
```java
package rabbitMQ.demo01;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.time.LocalDateTime;
import java.util.concurrent.TimeoutException;

public class MyProducer1 {

    public static final String QUEUE_NAME = "eric";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();

        // 连接到本地rabbit server，端口号已经内部指定了
        connectionFactory.setHost("127.0.0.1");

        // 通过连接工厂创建连接
        Connection connection = connectionFactory.newConnection();

        // 通过连接创建信道
        Channel channel = connection.createChannel();

        // 创建一个名为eric的队列
        // 该队列非持久（rabbitMQ重启后会消失）
        // 非独占（可以用于其他连接）
        // 非自动删除（队列至少被某个消费者连接之后，当再没有其他消费者连接时不会自动删除）
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        // 定义消息内容
        String msg = "hello , 我是Eric." + LocalDateTime.now().toString();

        // 发送消息
        // 参数1：指定交换机名称，""表示发送给默认交换机
        // 参数2：指定发布消息时携带的routing key，这里直接是Queue name，因为我们使用的是默认交换机，它默认会将所有queue绑定，routing key就是queue名称
        // 参数3：指定消息参数
        // 参数4：指定消息内容，需要转换为字节数组
        channel.basicPublish("", QUEUE_NAME, null, msg.getBytes(StandardCharsets.UTF_8));

        System.out.println("生产者发送消息完毕，消息内容为：" + msg);

        // 关闭信道
        channel.close();
        // 关闭连接
        connection.close();
    }
}
```
先通过rabbitMQ的ConnectionFactory配置一下要连接的server host，然后创建一个新的Connection，再通过此Connection创建Channel。   
最终通过这个Channel去创建队列和发送消息。   

创建队列的方法：   
```java
AMQP.Queue.DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete,Map<String, Object> arguments) throws IOException;
```
创建队列的方法中有5个参数：   
```youtrack
queue：表示队列的名称；   
durable：表示是否将此队列持久化，如果true，则服务器重启后该队列还在，否则，该队列消失。   
exclusive：表示该队列是否独占，如果设置为独占队列则此队列仅对首次声明它的连接可见，并在连接断开时自动删除。   
autoDelete：此队列是否自动删除，如果是true，当此队列已经被消费者连接过之后，已经没有任何连接者再与之连接，则自动删除该队列。   
arguments：代表其他额外参数。  
```
这些参数中，durable会经常用到，它表示我们可以对队列做持久化，以保证RabbitMQ服务器重启后此队列也可自行恢复。   

发送消息的方法：   
```java
void basicPublish(String exchange, String routingKey, AMQP.BasicProperties props, byte[] body) throws IOException;
```
发送消息的方法里有4个参数：
```youtrack
exchange：必须指定的转换机，案例中我们传入了""，表示使用rabbitMQ的默认转换机。   
routingKey：该消息携带的routing key，即路由key，案例中我们传入了Queue的名称，因为default exchange默认会将所有queue都绑定，且routing key为各个queue的名称。   
props：发送消息时的额外参数。
body：要发送的消息，字节数组。
```

运行代码：
```youtrack
生产者发送消息完毕，消息内容为：hello , 我是Eric.2023-11-04T16:21:38.370
```

去rabbitMQ查看：   
![](/images/myBlog/2023-10-04-rabbitMQ-queue-summary.png)   
已经创建了名为eric的Queue.   

![](/images/myBlog/2023-10-04-rabbitMQ-queue-binding.png)   
该Queue与默认交换机绑定。   

![](/images/myBlog/2023-10-04-rabbitMQ-queue-getMsg.png)   
成功从该Queue中获取到了生产者发送的消息。   


## 2.3 创建消费者
rabbitMQ.demo01.MyConsumer1
```java
package rabbitMQ.demo01;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

public class MyConsumer1 {

    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();

        // 连接到本地rabbit server，端口号已经内部指定了
        connectionFactory.setHost("127.0.0.1");

        // 通过连接工厂创建连接
        Connection connection = connectionFactory.newConnection();

        // 通过连接创建信道
        Channel channel = connection.createChannel();

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("----------------");
                System.out.println("consumerTag: " + consumerTag);
                System.out.println("Exchange Name：" + envelope.getExchange());
                System.out.println("Routing key：" + envelope.getRoutingKey());
                // 消息
                String msg = new String(body, StandardCharsets.UTF_8);
                System.out.println("消息内容：" + msg);
            }
        };

        // 启动消费者消费指定队列
        channel.basicConsume(MyProducer1.QUEUE_NAME, consumer);
        // 这两句代码要注释掉，因为现在是通过异步监听的方式消费消息，所以要一直保持在连接上监听，rabbitMQ服务器才会一有消息就push到消费者（回调上面的方法）
//        channel.close();
//        connection.close();


        // 还有一种同步消费的方式，消费者主动从Queue中获取消息
        // 这种方式就可以关闭连接，因为这个消费动作是消费者主动发起的，是pull模式
        // 但这种方式一般不使用，因为它没法做到异步消费，当服务器有消息来不会自动push到消费者，需要消费者主动去pull。
        //        GetResponse getResponse = channel.basicGet(MyProducer1.QUEUE_NAME,true);
//        String msg = new String(getResponse.getBody(), StandardCharsets.UTF_8);
//        System.out.println("消息内容：" + msg);
    }
}
```
消费者消费消息，有两种方式：   
（1）同步消费：pull模式，消费者主动去rabbitMQ服务器去获取消息，如果不获取，就拿不到消息。   
```java
GetResponse getResponse = channel.basicGet(MyProducer1.QUEUE_NAME,false);
```
源码如下：
```java
public GetResponse basicGet(String queue, boolean autoAck) throws IOException
```
queue：表示要消费的queue名称。   
autoAck：是否自动确认。true表示自动确认，false表示不自动确认，需要手动确认。   

临时扩展一下，rabbitMQ的消息是需要消费者确认才算成功消费的，只有消费者确认后，这条消息才会真正的从queue中删除，否则，这条消息即便被消费者拿到了，消费者没有确认，queue也不会删除这条消息。   

（2）异步消费：push模式，rabbitMQ中有消息进来就会主动push给订阅它的消费者，回调消费者basicConsume方法传入的consumer对象中的handleDelivery()方法。   
```java
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("----------------");
                System.out.println("consumerTag: " + consumerTag);
                System.out.println("Exchange Name：" + envelope.getExchange());
                System.out.println("Routing key：" + envelope.getRoutingKey());
                // 消息
                String msg = new String(body, StandardCharsets.UTF_8);
                System.out.println("消息内容：" + msg);
            }
        };

        // 启动消费者消费指定队列
        channel.basicConsume(MyProducer1.QUEUE_NAME, consumer);
```
这种异步消费的方式，一定不要关闭连接，否则rabbitMQ就没法push消息到消费者了。 
看下异步消费的源码：   
```java
   public String basicConsume(String queue, Consumer callback) throws IOException {
        return this.basicConsume(queue, false, callback);
    }

    public String basicConsume(String queue, boolean autoAck, Consumer callback) throws IOException {
        return this.basicConsume(queue, autoAck, "", callback);
    }

    public String basicConsume(String queue, boolean autoAck, String consumerTag, Consumer callback) throws IOException {
        return this.basicConsume(queue, autoAck, consumerTag, false, false, (Map)null, callback);
    }

    public String basicConsume(String queue, boolean autoAck, Map<String, Object> arguments, Consumer callback) throws IOException {
        return this.basicConsume(queue, autoAck, "", false, false, arguments, callback);
    }

    public String basicConsume(String queue, boolean autoAck, String consumerTag, boolean noLocal, boolean exclusive, Map<String, Object> arguments, Consumer callback) throws IOException {
        String result = this.delegate.basicConsume(queue, autoAck, consumerTag, noLocal, exclusive, arguments, callback);
        this.recordConsumer(result, queue, autoAck, exclusive, arguments, callback);
        return result;
    }
```
可以看到源码中 basicConsume方法有很多重载，其中Consumer对象实际上就是一个callback对象，用于服务器收到消息后主动回调的对象。   
同时，也可以看到，可以传入参数 autoAck 为 true，在消费的时候自动确认消息的接收。     

运行代码：   
```youtrack
consumerTag: amq.ctag-B7jJX2ejtz9MX2NrWCRkfg
Exchange Name：
Routing key：eric
消息内容：hello , 我是Eric.2023-11-04T16:48:47.968
```  
consumerTag：是这条消息的标识。   
Exchange Name：这条消息所发送exchange的名字，我们发送消息时传入的是空字符串，所以这里也是空字符串。   
Routing key：是这个消息携带的routing key。   

这样我们的程序就一直处在异步消费的状态下，当我们再次调用生产者生产一条新的消息时，消费者就会自动消费该消息。   

## 2.4 消息接收确认（ACK）
上面我们演示了生产者和消费者，生产者发送一条消息，消费者消费一条消息，那么这个时候rabbitMQ中有多少条消息呢？   
理论上来说，发送一条，消费一条，rabbitMQ中的消息数应该是0才对。   
![](/images/myBlog/2023-10-04-rabbitMQ-queue-msgcount.png)   
结果我们发现消息还在。   
我们重新运行下消费者：   
```youtrack
consumerTag: amq.ctag-xaQeuEcNO73VJH8EmSQgSw
Exchange Name：
Routing key：eric
消息内容：hello , 我是Eric.2023-11-04T16:48:47.968
```
发现这条消息还可以正常消费，又打印了一遍出来，而且从时间上看就是我们之前生产的消息。   
也就是说，消费者消费之后，这条消息并没有从队列中删除。   
这种情况出现的原因是因为rabbitMQ中的消息接收确认机制，也就说一条消息被消费者接收之后，需要进行一次确认操作，这条消息才会被服务器删除。   
这种确认机制，避免了消息的丢失。   

rabbitMQ中的消费接收确认是手动的，也可以将其设置为自动的（参见上述代码），但自动确认模式在消费者接收到这条消息后就会通知服务器自动删除这条消息，如果消息处理过程中出现了异常，这条消息就等于没被处理完但也被删掉了，所以一般我们都会使用手动确认。   

消息接收确认（ACK）的代码很简单，只要在原来消费者的代码里加上一句就行了：   
```java
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("----------------");
                System.out.println("consumerTag: " + consumerTag);
                System.out.println("Exchange Name：" + envelope.getExchange());
                System.out.println("Routing key：" + envelope.getRoutingKey());
                // 消息
                String msg = new String(body, StandardCharsets.UTF_8);
                System.out.println("消息内容：" + msg);

                // 消息接收手动确认
                channel.basicAck(envelope.getDeliveryTag(), false);
                System.out.println("消息已经确认");
                
            }
        };
```
basicAck方法的源码如下：
```java
    public void basicAck(long deliveryTag, boolean multiple) throws IOException {
        this.delegate.basicAck(deliveryTag, multiple);
    }
```
deliveryTag：接收标签，一个long值，也就是消息id，表示该条消息的唯一性。   
multiple：是确认该消费者缓存的全部消息，还是只是当前消息。也就是批量操作。比如，当前消费者启动后，在之前已经消费了消息1,2,3,4,但都没有确认，当前消费了消息5，如果将 multiple 设置为true，那么会将之前消费的消息1,2,3,4和当前的5都确认接收，否则只会确认接收5.   
请注意，消费者在一次启动中会缓存消费过的所有消息。   

重启消费者：
```youtrack
consumerTag: amq.ctag-NAx3OpIKOB0cecySMeQKOw
Exchange Name：
Routing key：eric
消息内容：hello , 我是Eric.2023-11-04T16:48:47.968
消息已经确认
```

查看队列的情况：   
![](/images/myBlog/2023-10-04-rabbitMQ-queue-empty.png)   
发现队列中消息已经被删除了。   
其实我们可以猜到，自动删除实际上是在我们的代码逻辑还没有执行的时候就帮我们进行了确认，说白了只要消费者拿到消息就立马返回确认接收。   
这样就导致了消息丢失的可能性。   

我们采用手动确认的方式后，可以现将处理逻辑处理完毕之后（可能出现异常的地方使用try catch），把手动确认的代码放在最后处理，这样如果出现异常情况导致这条消息没有被确认，那么这条消息就不会被Queue删除，它会在之后被重新消费。   
   
# 3.案例2
## 3.1 生产者   
rabbitMQ.demo2.MyProducer2
```java
package rabbitMQ.demo2;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

public class MyProducer2 {

    // 本机服务器ip
    private static String ip = "127.0.0.1";

    // rabbitMQ的默认启动端口，也是客户端程序连接RabbitMQ的端口
    private static int port = 5672;

    // rabbitMQ有一个"/"的虚拟主机
    private static String virtualHost = "/";

    // default exchange
    private static String defaultExchange = "";

    private static String defaultRoutingKey = "eric-daisy";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost(ip);
        connectionFactory.setPort(port);
        connectionFactory.setVirtualHost(virtualHost);

        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        // 通过channel发送5条消息
        for (int i = 0; i < 5; i++) {
            String msg = "Hello RabbitMQ:" + i;

            // 向default exchange发送消息，指定了routing key
            channel.basicPublish(defaultExchange, defaultRoutingKey, null, msg.getBytes(StandardCharsets.UTF_8));
        }

        channel.close();
        connection.close();
    }
}
```
注意，上面的生产者没有创建queue。   
先不着急运行。   

## 3.2 消费者 
rabbitMQ.demo2.MyConsumer2
```java
package rabbitMQ.demo2;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.QueueingConsumer;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

public class MyConsumer2 {

    // 本机服务器ip
    private static String ip = "127.0.0.1";

    // rabbitMQ的默认启动端口，也是客户端程序连接RabbitMQ的端口
    private static int port = 5672;

    // rabbitMQ有一个"/"的虚拟主机
    private static String virtualHost = "/";

    // default exchange
    private static String defaultExchange = "";

    // 定义的队列名称，和生产者中定义的routing key相同
    private static String queue = "eric-daisy";

    public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost(ip);
        connectionFactory.setPort(port);
        connectionFactory.setVirtualHost(virtualHost);

        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        // 在消费者端创建一个队列
        // 1.队列名 2.持久化 3.非独占 4.非自动删除 5.队列参数
        channel.queueDeclare(queue, true, false, false, null);

        // 创建一个消费者：QueueingConsumer是DefaultConsumer的一个继承者，一个内部实现好的消费者回调对象
        QueueingConsumer queueingConsumer = new QueueingConsumer(channel);

        // 使用消费者消费
        // 1.要消费的队列名 2.自动接收确认 3.传入的消费者回调对象
        channel.basicConsume(queue, true, queueingConsumer);

        // 不要关闭连接，监听服务器，等待服务器主动push消息进来
        while (true) {
            // 一个死循环，启动后就一直从queueingConsumer中获取交付物（Delivery）
            QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
            String msg = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println("消息：" + msg);
        }
    }
}

```   
和上一个案例不同的是，QueueingConsumer是一个内置的消费者回调对象，我们可以直接使用。   
源码如下：
```java
package com.rabbitmq.client;

import com.rabbitmq.client.AMQP.BasicProperties;
import com.rabbitmq.utility.Utility;
import java.io.IOException;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

public class QueueingConsumer extends DefaultConsumer {
    private final BlockingQueue<QueueingConsumer.Delivery> _queue;
    private volatile ShutdownSignalException _shutdown;
    private volatile ConsumerCancelledException _cancelled;
    private static final QueueingConsumer.Delivery POISON = new QueueingConsumer.Delivery((Envelope)null, (BasicProperties)null, (byte[])null);

    public QueueingConsumer(Channel ch) {
        this(ch, new LinkedBlockingQueue());
    }

    public QueueingConsumer(Channel ch, BlockingQueue<QueueingConsumer.Delivery> q) {
        super(ch);
        this._queue = q;
    }

    public void handleShutdownSignal(String consumerTag, ShutdownSignalException sig) {
        this._shutdown = sig;
        this._queue.add(POISON);
    }

    public void handleCancel(String consumerTag) throws IOException {
        this._cancelled = new ConsumerCancelledException();
        this._queue.add(POISON);
    }

    public void handleDelivery(String consumerTag, Envelope envelope, BasicProperties properties, byte[] body) throws IOException {
        this.checkShutdown();
        this._queue.add(new QueueingConsumer.Delivery(envelope, properties, body));
    }

    private void checkShutdown() {
        if (this._shutdown != null) {
            throw (ShutdownSignalException)Utility.fixStackTrace(this._shutdown);
        }
    }

    private QueueingConsumer.Delivery handle(QueueingConsumer.Delivery delivery) {
        if (delivery == POISON || delivery == null && (this._shutdown != null || this._cancelled != null)) {
            if (delivery == POISON) {
                this._queue.add(POISON);
                if (this._shutdown == null && this._cancelled == null) {
                    throw new IllegalStateException("POISON in queue, but null _shutdown and null _cancelled. This should never happen, please report as a BUG");
                }
            }

            if (null != this._shutdown) {
                throw (ShutdownSignalException)Utility.fixStackTrace(this._shutdown);
            }

            if (null != this._cancelled) {
                throw (ConsumerCancelledException)Utility.fixStackTrace(this._cancelled);
            }
        }

        return delivery;
    }

    public QueueingConsumer.Delivery nextDelivery() throws InterruptedException, ShutdownSignalException, ConsumerCancelledException {
        return this.handle((QueueingConsumer.Delivery)this._queue.take());
    }

    public QueueingConsumer.Delivery nextDelivery(long timeout) throws InterruptedException, ShutdownSignalException, ConsumerCancelledException {
        return this.handle((QueueingConsumer.Delivery)this._queue.poll(timeout, TimeUnit.MILLISECONDS));
    }

    public static class Delivery {
        private final Envelope _envelope;
        private final BasicProperties _properties;
        private final byte[] _body;

        public Delivery(Envelope envelope, BasicProperties properties, byte[] body) {
            this._envelope = envelope;
            this._properties = properties;
            this._body = body;
        }

        public Envelope getEnvelope() {
            return this._envelope;
        }

        public BasicProperties getProperties() {
            return this._properties;
        }

        public byte[] getBody() {
            return this._body;
        }
    }
}

```
通过nextDelivery()方法可以获取服务器推送到消费者端的交付物（Delivery），然后Delivery对象中有我们需要的消息体。   

## 3.3 测试
### 3.3.1 先启动生产者
访问rabbitMQ web端，如下：   
![](/images/myBlog/2023-10-04-rabbitMQ-queue-pro-notcreate.png)   
发现启动生产者后，web端没有我们需要的队列，这也正常，因为生产者没有创建队列。   

启动消费者：   
发现消费者没有任何输出，web端如下：   
![](/images/myBlog/2023-10-04-rabbitMQ-queue-con-create.png)   
发现启动消费者后，队列eric-daisy被创建了，但是里面没有任何消息。   

这也符合我们代码的编写，因为生产者没有创建队列，只是将消息推送到default exchange，而default exchange根据routing key找不到队列，于是将消息丢弃。   
然后启动消费者，创建了队列，然后从队列中消费消息，但是这个队列是才创建的，里面没有消息。   

### 3.3.2 先启动消费者
接上面的案例，保持上面的消费者启动状态，重新启动一次生产者：   
![](/images/myBlog/2023-10-04-rabbitMQ-consumer-get.png)   
当我们重新启动一次生产者后，发现消费者控制台立马打印5条消息。   
这说明，我们消费者获取消息的方式是异步监听获取的，rabbitMQ的队列中一收到消息就会主动push给监听这个队列的消费者。   

所以说，在设计的时候，我们是不是该考虑下，交换机、队列等的创建是要放在生产者端好？还是放在消费者端好？   
我理解要放在生产者端好一些，以为生产者是生产消息的，有消息后才需要交换机、队列，因此交换机和队列的生命周期应该和生产者保持一致比较好。   

# 4. 通过客户端创建随机队列
每当我们连接到RabbitMQ时，都需要一个全新的空队列。   
为此我们可以让服务器帮我们创建一个具有随机名称的新队列。   
当这个队列被消费者连接过后，再没有任何消费者进行连接的情况下，会自动删除。   

随机队列，实际上是一个具备随机名称、自动删除的队列。   
通过如下方法创建一个随机队列：   
```java
channel.queueDeclare();
```
通过如下方法获取随机队列的名称：
```java
channel.queueDeclare().getQueue();
```
我们点进源码看看，这个随机队列本质到底是个啥？   
```java
    public com.rabbitmq.client.AMQP.Queue.DeclareOk queueDeclare() throws IOException {
        return this.queueDeclare("", false, true, true, (Map)null);
    }
```
可以看到，所谓的随机队列，实际上队列名传递了一个空字符串，durable为false表示非持久化、exclusive为true表示当前连接独占、autoDelete为true表示自动删除。   
```java
    public com.rabbitmq.client.AMQP.Queue.DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments) throws IOException {
        com.rabbitmq.client.AMQP.Queue.DeclareOk ok = this.delegate.queueDeclare(queue, durable, exclusive, autoDelete, arguments);
        RecordedQueue q = (new RecordedQueue(this, ok.getQueue())).durable(durable).exclusive(exclusive).autoDelete(autoDelete).arguments(arguments);
        if (queue.equals("")) {
            q.serverNamed(true);
        }

        this.recordQueue((com.rabbitmq.client.AMQP.Queue.DeclareOk)ok, q);
        return ok;
    }
```
可以看到交给server去生成队列名称。   

## 4.1 生产者
rabbitMQ.demo03.MyProducer3
```java
package rabbitMQ.demo03;

import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.time.LocalDateTime;
import java.util.concurrent.TimeoutException;

public class MyProducer3 {

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();

        connectionFactory.setHost("127.0.0.1");

        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        AMQP.Queue.DeclareOk declareOk = channel.queueDeclare();
        String rondomQueueName = declareOk.getQueue();
        System.out.println("随机队列的名称：" + rondomQueueName);

        String msg = "hello , Random Queue." + LocalDateTime.now().toString();

        channel.basicPublish("", rondomQueueName, null, msg.getBytes(StandardCharsets.UTF_8));

        System.out.println("生产者发送消息完毕，消息内容为：" + msg);

    }
}
```
运行如下：   
```youtrack
随机队列的名称：amq.gen-rhxdRmzSRFJogT2W-mAhQg
生产者发送消息完毕，消息内容为：hello , Random Queue.2023-11-04T20:09:48.311
```
可以看到，运行生产者后，让服务器帮我们创建一个随机队列，然后向该随机队列发送了一条消息。   
请注意，上面的生产者的连接一直没有中断，我删除了如下代码：
```java
channel.close();
connection.close();
```
如果放开上面的代码，那么这个临时队列创建完，连接关闭了，这个队列就消失了。   

登录rabbitMQ的web端查看：  
![](/images/myBlog/2023-10-04-rabbitMQ-queue-rondom.png)   
只要我们创建随机队列的连接不关闭，这个队列就一直存在。   

## 4.2 消费者
rabbitMQ.demo03.MyConsumer3   
```java
package rabbitMQ.demo03;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

public class MyConsumer3 {

    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();

        // 连接到本地rabbit server，端口号已经内部指定了
        connectionFactory.setHost("127.0.0.1");

        // 通过连接工厂创建连接
        Connection connection = connectionFactory.newConnection();

        // 通过连接创建信道
        Channel channel = connection.createChannel();

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumerTag: " + consumerTag);
                System.out.println("Exchange Name：" + envelope.getExchange());
                System.out.println("Routing key：" + envelope.getRoutingKey());
                String msg = new String(body, StandardCharsets.UTF_8);
                System.out.println("消息内容：" + msg);

                channel.basicAck(envelope.getDeliveryTag(), false);
                System.out.println("消息已经确认");
            }
        };

        // 启动消费者消费指定队列
        channel.basicConsume("amq.gen-rhxdRmzSRFJogT2W-mAhQg", consumer);
    }
}

```
启动发现报错了：   
```youtrack
C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\bin\java.exe "-javaagent:C:\D\soft\soft_application\onlyou-app\intellij-idea\IntelliJ IDEA 2018.2.5\lib\idea_rt.jar=61650:C:\D\soft\soft_application\onlyou-app\intellij-idea\IntelliJ IDEA 2018.2.5\bin" -Dfile.encoding=UTF-8 -classpath C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\charsets.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\deploy.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\access-bridge-64.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\cldrdata.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\dnsns.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\jaccess.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\jfxrt.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\localedata.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\nashorn.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\sunec.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\sunjce_provider.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\sunmscapi.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\sunpkcs11.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\zipfs.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\javaws.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\jce.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\jfr.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\jfxswt.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\jsse.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\management-agent.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\plugin.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\resources.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\rt.jar;C:\D\gitsource\zhaoehcode-java\comrabbitmqjava\target\classes;C:\D\soft\soft_application\maven\repo\com\rabbitmq\amqp-client\3.6.5\amqp-client-3.6.5.jar rabbitMQ.demo03.MyConsumer3
Exception in thread "main" java.io.IOException
	at com.rabbitmq.client.impl.AMQChannel.wrap(AMQChannel.java:105)
	at com.rabbitmq.client.impl.AMQChannel.wrap(AMQChannel.java:101)
	at com.rabbitmq.client.impl.ChannelN.basicConsume(ChannelN.java:1118)
	at com.rabbitmq.client.impl.ChannelN.basicConsume(ChannelN.java:1086)
	at com.rabbitmq.client.impl.ChannelN.basicConsume(ChannelN.java:1070)
	at com.rabbitmq.client.impl.ChannelN.basicConsume(ChannelN.java:1063)
	at rabbitMQ.demo03.MyConsumer3.main(MyConsumer3.java:47)
Caused by: com.rabbitmq.client.ShutdownSignalException: channel error; protocol method: #method<channel.close>(reply-code=405, reply-text=RESOURCE_LOCKED - cannot obtain exclusive access to locked queue 'amq.gen-rhxdRmzSRFJogT2W-mAhQg' in vhost '/'. It could be originally declared on another connection or the exclusive property value does not match that of the original declaration., class-id=60, method-id=20)
	at com.rabbitmq.utility.ValueOrException.getValue(ValueOrException.java:66)
	at com.rabbitmq.utility.BlockingValueOrException.uninterruptibleGetValue(BlockingValueOrException.java:32)
	at com.rabbitmq.client.impl.AMQChannel$BlockingRpcContinuation.getReply(AMQChannel.java:360)
	at com.rabbitmq.client.impl.ChannelN.basicConsume(ChannelN.java:1116)
	... 4 more
Caused by: com.rabbitmq.client.ShutdownSignalException: channel error; protocol method: #method<channel.close>(reply-code=405, reply-text=RESOURCE_LOCKED - cannot obtain exclusive access to locked queue 'amq.gen-rhxdRmzSRFJogT2W-mAhQg' in vhost '/'. It could be originally declared on another connection or the exclusive property value does not match that of the original declaration., class-id=60, method-id=20)
	at com.rabbitmq.client.impl.ChannelN.asyncShutdown(ChannelN.java:483)
	at com.rabbitmq.client.impl.ChannelN.processAsync(ChannelN.java:320)
	at com.rabbitmq.client.impl.AMQChannel.handleCompleteInboundCommand(AMQChannel.java:143)
	at com.rabbitmq.client.impl.AMQChannel.handleFrame(AMQChannel.java:90)
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:559)
	at java.lang.Thread.run(Thread.java:745)

```
这是怎么回事？   
这是因为生产者的连接创建的随机队列，其只对首次创建它的连接可见，其他连接无法访问这个队列。   

关闭生产者，发现web端那个随机队列立马被删除了：   
![](/images/myBlog/2023-10-04-rabbitMQ-queue-rondom-delete.png)   

# 5. 客户端代码显式使用交换机
上面的案例，我们使用的都是默认交换机（default exchange），下面我们将来一个完整的案例，自己去创建交换机。   
## 5.1 生产者
rabbitMQ.demo04.MyProducer4
```java
package rabbitMQ.demo04;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.time.LocalDateTime;
import java.util.concurrent.TimeoutException;

public class MyProducer4 {

    private static final String EXCHANGE_NAME = "zeh_love_yu";
    public static final String QUEUE_NAME = "customized.queue.daisy";
    private static final String ROUTING_KEY = "zeh.daisy";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();

        connectionFactory.setHost("127.0.0.1");

        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        // 创建一个exchange
        // 1.exchange name 2.exchange type 3.是否持久化 4.是否自动删除 5.附加参数
        channel.exchangeDeclare(EXCHANGE_NAME, "direct", false, false, null);

        // 创建一个Queue
        // 1.queue name 2.是否持久化 3.是否连接独占 4.是否自动删除 5.附加参数
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        // 创建一个Binding
        // queueBind()方法：1.要绑定的目标Queue 2.当前交换机 3.路由key  4.附加参数
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY, null);

        // 注意还有一个方法比较类似，这个方法用于绑定交换机
        // 将源交换机和目标交换机进行绑定，这个一般用不到
        // exchangeBind()方法：1.要绑定的目标交换机   2.当前的源交换机  3.路由key  4.附加参数
//        channel.exchangeBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY, null);

        // 定义消息内容
        String msg = "hello , i love daisy." + LocalDateTime.now().toString();

        // 向创建的直连交换机发送消息，指定routing key
        channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, null, msg.getBytes(StandardCharsets.UTF_8));

        System.out.println("生产者发送消息完毕，消息内容为：" + msg);

        // 关闭信道
        channel.close();
        // 关闭连接
        connection.close();
    }
}

```
上面代码创建了一个名为zeh_love_yu的direct类型交换机；      
创建了一个名为customized.queue.daisy的queue；      
创建了一个binding，将上面的交换机和队列进行绑定，并指定路由Key为zeh.daisy。   
然后发送消息的代码如下：   
```java
channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, null, msg.getBytes(StandardCharsets.UTF_8));
```
向指定交换机发送消息，指定消息携带的路由key。   

运行如下：   
```youtrack
生产者发送消息完毕，消息内容为：hello , i love daisy.2023-11-04T21:46:16.755
```

web端如下：   
![](/images/myBlog/2023-10-04-rabbitMQ-queue-show-exchange.png)   
可以看到，我们的队列有了。   
![](/images/myBlog/2023-10-04-rabbitMQ-queue-show-exchange2.png)   
交换机也有了。   
![](/images/myBlog/2023-10-04-rabbitMQ-queue-show-binding.png)   
交换机和队列的绑定关系也有了。   

## 5.2 消费者
rabbitMQ.demo04.MyConsumer4
```java
package rabbitMQ.demo04;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

public class MyConsumer4 {
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();

        connectionFactory.setHost("127.0.0.1");

        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumerTag: " + consumerTag);
                System.out.println("Exchange Name：" + envelope.getExchange());
                System.out.println("Routing key：" + envelope.getRoutingKey());
                String msg = new String(body, StandardCharsets.UTF_8);
                System.out.println("消息内容：" + msg);
            }
        };

        // 自动确认
        channel.basicConsume(MyProducer4.QUEUE_NAME, true, consumer);
    }
}
```
上面消费者从目标队列去异步消费消息。   
运行如下：   
```youtrack
consumerTag: amq.ctag-mjiMrpmUO35v6VYwO49YuA
Exchange Name：zeh_love_yu
Routing key：zeh.daisy
消息内容：hello , i love daisy.2023-11-04T21:46:16.755
```

