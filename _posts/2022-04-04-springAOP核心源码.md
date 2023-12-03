---
layout:     post
title:      spring AOP源码
subtitle:   从源码的角度分析AOP
categories: [spring AOP专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. AOP
AOP说白了底层就是代理。   
无非就是动态代理的那几种方式。   
关于代理，请移步![死磕代理设计模式](https://zhaoeh.github.io/myblog/2021/04/09/%E6%AD%BB%E7%A3%95%E4%BB%A3%E7%90%86%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/)   

# 2. AOP源码分析
spring中大名鼎鼎的AOP，我们只知道使用简单的aspectJ进行配置织入，那它底层到底是怎么实现的呢？
spring动态织入aop的能力这么强大，那底层到底是在哪里做了什么手脚，才然我们任意织入的逻辑在指定的目标位置生效，从而达到拦截的效果呢？   

