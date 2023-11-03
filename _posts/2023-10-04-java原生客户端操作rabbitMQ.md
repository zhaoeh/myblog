---
layout:     post
title:      java原生客户端操作rabbitMQ
subtitle:   通过java原生客户端代码去操作rabbitMQ
categories: [rabbitMQ]
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
