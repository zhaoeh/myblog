---
layout:     post
title:      rabbitMQ 管理台操作
subtitle:   安装好rabbitMQ后，可以登录管理台手动操作它
categories: [MQ系列]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. RabbitMQ管理台界面简介
在第一篇文章[rabbitMQ 安装](https://zhaoeh.github.io/myblog/2023/10/01/rabbitMQ%E5%AE%89%E8%A3%85/)中安装完RabbitMQ后，访问管理台地址：   
http://localhost:15672/   
输入默认用户名和密码： guest/guest   
就能登录RabbitMQ的管理台页面了，如下：   
![](/images/myBlog/2023-10-03-rabbitMQ-console.png)

我们主要关注Exchanges页面和Queue页面，因为大多数我们通过管理台手动操作，只是为了手动配置Exchange和Queue并建立绑定关系。   

# 2. Exchanges页面
![](/images/myBlog/2023-10-03-rabbitMQ-exchanges.png)
在这个页面可以新增一个交换机，并指定交换机名称和类型等。   

我们可以看到，rabbitMQ默认就已经内置了7种交换机。如下图：
![](/images/myBlog/2023-10-03-rabbitMQ-exchanges-default.png)
（1）AMQP default：是rabbitMQ的默认交换机（default exchange），它实际上由rabbitMQ预先创建好，并且名字为一个空字符串("")的直连（Direct）交换机。它有一个特殊的属性使得它对简单应用特别友好：那就是为每一个新创建的Queue都会自动绑定到该默认Exchange上，绑定的Routing key的名称就是新建的Queue的名称。   
这意味着，当你创建一个Queue后，并没有为该Queue绑定任何交换机时，rabbitMQ默认在你创建完Queue后就自动将该Queue绑定到default Exchange上，绑定的Routing key的名称就是该Queue的名称。   

比如：当你创建了一个名称"hello"的Queue，rabbitMQ会自动将其绑定到default Exchange上，绑定的Routing key的名称也是"hello"。因此，当携带着名称"hello"的Routing key的message被生产者执行发送操作的时候，如果指定交换机名称为""的话，该消息会被发送到default Exchange，而default Exchange会将该消息路由到名为"hello"的Queue中去。即默认交换机看起来能够直接投递消息到Queue中去，对于简单的应用来说，可以不用创建任何交换机，在发送消息的时候，也不用指定交换机的名称（指定为""即为默认交换机），只需要指定Queue的名称（此时Queue的名称实际上是消息携带的Routing key的名称），就能够将消息直接投递到目标Queue中去。   
这一系列原理，实际上就是这个default Exchange在背后默默为我们提前做好的。   

（2）下面6个以"amq."开头的交换机：这6个交换机也是rabbitMQ自动创建好的内置交换机，这些交换机被预留做rabbitMQ内部使用，应用不能直接使用，否则爬出403（ACCESS_REFUSED）错误。   

交换机的四种类型：direct，topic，fanout，headers。   

![](/images/myBlog/2023-10-03-rabbitMQ-exchange-durable.png)
Durability:；两个选项。可以指定交换机特征，是临时的还是持久的。   
Durable：表示该交换机是持久的，当RabbitMQ服务器重启后，该交换机依然存在。   
Transient：表示该交换机是临时的，当RabbitMQ服务器重启后，该交换机不存在。   

![](/images/myBlog/2023-10-03-rabbitMQ-exchange-autodelete.png)
可以指定该交换机是否自动删除，默认是No。   
如果选择Yes，则当没有任何队列与该交换机绑定时，该交换机将自动删除。   

![](/images/myBlog/2023-10-03-rabbitMQ-exchanges-inter.png)
可以指定该交换机是否为内部交换机，默认为No。   
如果选择Yes，则该交换机无法被客户端直接使用，需要通过一个外部交换机与该交换机进行绑定，才能将消息从外部交换机发送到该交换机上来。   
一般都选择No，即创建该交换机就是要让外部客户端直接连接使用的。   

![](/images/myBlog/2023-10-03-rabbitMQ-exchanges-params.png)
可以指定该交换机参数。   
alternate：备用的。   
顾名思义，可以通过参数 alternate-exchange 来指定一个备用交换机。   
上图的解释也很清楚了，如果无法以其他方式路由到此交换器的消息，请将它们发送到此处指定的备用交换器。   
通过alternate-exchange参数来指定一个备用交换机。       
这个Arguments参数，主要是扩展参数，用于扩展AMQP协议定制使用。   

# 3. Queues and Streams页面
![](/images/myBlog/2023-10-03-rabbitMQ-queue.png)
默认没有任何Queue。   
我们可以在这个页面新增Queue。   

![](/images/myBlog/2023-10-03-rabbitMQ-queue-config.png)
Virtual host:该Queue对应的虚拟机路径，只能是/。   
Type: 该Queue对应的类型，一般选择Default for virtual host即可。   

![](/images/myBlog/2023-10-03-rabbitMQ-queue-durable.png)   
Durability:；两个选项。可以指定Queue特征，是临时的还是持久的。     
Durable：表示该队列是持久的，当RabbitMQ服务器重启后，该队列依然存在。      
Transient：表示该队列是临时的，当RabbitMQ服务器重启后，该队列不存在。     

![](/images/myBlog/2023-10-03-rabbitMQ-queue-params.png)
在创建Queue时，可以设置一些参数。   
```youtrack
Auto expire：How long a queue can be unused for before it is automatically deleted (milliseconds).
            (Sets the "x-expires" argument.)
是指queue的过期时间，到达过期时间后这个queue将被自动删除。         

Message TTL:How long a message published to a queue can live before it is discarded (milliseconds).
            (Sets the "x-message-ttl" argument.)
发布到队列的消息的过期时间，到达过期时间后该消息将被丢弃。               

            
Overflow behaviour:Sets the queue overflow behaviour. This determines what happens to messages when the maximum length of a queue is reached. Valid values are drop-head, reject-publish or reject-publish-dlx. The quorum queue type only supports drop-head and reject-publish.
设置队列的溢出行为，当队列中的元素达到队列的最大元素数后，那些消息该如何被处理？
默认有几种策略：
drop-head
reject-publish
reject-publish-dlx

Single active consumer:If set, makes sure only one consumer at a time consumes from the queue and fails over to another registered consumer in case the active one is cancelled or dies.
                       (Sets the "x-single-active-consumer" argument.)
使用 x-single-active-consumer 参数设置。 
指定该队列的单个活跃消费者。如果设置，请确保一次只有一个消费者从该队列中消费。                          
                       
Dead letter exchange:Optional name of an exchange to which messages will be republished if they are rejected or expire.
                     (Sets the "x-dead-letter-exchange" argument.)
使用 x-dead-letter-exchange 参数设置。
死信交换，消息被拒绝发布或者消息过期后，将重新发布到该参数指定的死信交换机上。
通过该参数可以设置该队列绑定的死信交换机。                        
                     
Dead letter routing key:Optional replacement routing key to use when a message is dead-lettered. If this is not set, the message's original routing key will be used.
                        (Sets the "x-dead-letter-routing-key" argument.)
使用 x-dead-letter-routing-key 参数设置。
死信交换机的Routing key，当该队列中的消息为死信时，设置死信Routing key，该消息将被发送到死信交换机中，死信交换机将通过该Routing key将该死信发送到死信队列中。 
如果不指定该参数，将使用该消息原始的Routing key作为死信路由key，但原始key应保证已经在死信交换机绑定死信队列时指定了该key，否则将无法路由到死信队列中。                           
                        
Max length:How many (ready) messages a queue can contain before it starts to drop them from its head.
           (Sets the "x-max-length" argument.)
使用 x-max-length 参数设置。
队列的最大长度，该队列中可以缓存的最大元素个数。           
           
Max length bytes:Total body size for ready messages a queue can contain before it starts to drop them from its head.
                 (Sets the "x-max-length-bytes" argument.)
使用  x-max-length-bytes 参数设置。
队列的最大容量大小，即该队列中可以缓存的最大字节容量。                 
                 
Leader locator:Set the rule by which the queue leader is located when declared on a cluster of nodes. Valid values are client-local (default) and balanced.
领导者定位器：设置在集群节点上声明时队列领导者的定位规则。 有效值为client-local (default)和balanced。                                                                                      

```

# 3. Binding（指定Routing key绑定Queue到Exchange中）
要绑定Queue到Exchange中，那么我们首先需要创建一个Queue和Exchange。   
请注意：前面我们说过，如果我们只是创建了一个Queue，而没有额外将该Queue绑定到某个Exchange上的话，那么该Queue将默认绑定到default Exchange上，并且Routing key 默认为该Queue name。   

## 3.1 创建一个Exchange
![](/images/myBlog/2023-10-03-rabbitMQ-exchange-create.png)

## 3.2 创建一个Queue
![](/images/myBlog/2023-10-03-rabbitMQ-queue-create.png)

## 3.3 将Queue绑定到Exchange中
进入到自己创建的Exchange中。如下：
![](/images/myBlog/2023-10-03-rabbitMQ-binding.png)

创建好一个Exchange后，我们可以看到，这个页面有很多能力。   
为该Exchange增加一个Binding，即绑定一个Queue，指定Routing key。    
![](/images/myBlog/2023-10-03-rabbitMQ-binding-toqueue.png)
从页面可以看到，binding功能实际上是被复用的。   
因为在增加一个binding时，它有To queue还是To exchange的选项。   
这说明，增加一个binding，可以从Exchange的角度出发，为Exchange增加一个binding，即将一个Exchange绑定到一个Queue上（To Queue）；   
也可以从Queue的角度出发，为Queue增加一个binding，即将一个Queue绑定到一个Exchange上（To Exchange）。   
看你以谁为维度去操作了，反正目的就是为Exchange和Queue之间通过Routing增加一个绑定关系。   
为该Exchange绑定一个Queue：
![](/images/myBlog/2023-10-03-rabbitMQ-binding-bindings.png)
可以看到，增加一个binding后，里面展示出了这个绑定关系。   
我们还可以继续为该Exchange增加其他的绑定关系。   
即一个Exchange可以绑定多个Queue（相反一个Queue也可以绑定多个Exchange，一会我们从Queue的角度再看一下binding）。      

可以发布消息，指定消息的Routing key和消息内容以及其他属性等。


可以删除该Exchange。   

   



