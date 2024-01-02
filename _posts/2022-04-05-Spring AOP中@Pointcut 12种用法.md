---
layout:     post
title:      Spring AOP中@Pointcut 12种用法
subtitle:   顺便学习创建AOP代理对象的第三种手动方式-AspectJProxyFactory
categories: [spring AOP专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. AspectJProxyFactory
目前手动Aop中三种方式已经介绍2种了，本文将介绍另外一种：AspectJProxyFactory，可能大家对这个比较陌生，但是@Aspect这个注解大家应该很熟悉吧，通过这个注解在spring环境中实现aop特别的方便。      
