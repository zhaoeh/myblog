---
layout:     post
title:      springboot加载配置项的源码解析以及加载顺序和配置项的优先级
subtitle:   从源码角度分析springboot加载配置项的顺序和优先级
categories: [springboot专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. springboot加载外部配置项的优先级
上一篇文章都是在介绍springboot如何加载外部配置项，即如何从外部配置文件中读取指定的配置项。  
当然，springboot中负责维护各种属性配置的核心对象是Environment。而且，这个对象中被注册进去的各个属性配置来源，并不仅仅局限于当前classpath下的配置文件。     
具体来说，它还可以从环境变量中加载属性配置、从启动参数中加载属性配置、从远程配置中心中加载属性配置等。      
还可以从自定义的PropertySource中加载属性配置。    
