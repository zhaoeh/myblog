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

# 1. AOP原理
原理比较简单，主要就是使用jdk动态代理和cglib代理来创建代理对象，通过代理对象来访问目标对象，而代理对象中融入了增强的代码，最终起到对目标对象增强的效果。      

aop相关的一些类：   
1.连接点（JoinPoint）相关类   
2.通知（Advice）相关的类   
3.切入点（Pointcut）相关的类   
4.切面（Advisor）相关的类   

## 1.1 连接点（JoinPoint）相关类
### 1.1.1 JoinPoint接口
这个接口表示一个通用的运行时连接点（在AOP术语中）   
```java
package org.aopalliance.intercept;
public interface Joinpoint {
    
    // 转到拦截器链中的下一个拦截器
    Object proceed() throws Throwable;
    
    // 返回保存当前连接点静态部分【的对象】，这里一般指被代理的目标对象
    Object getThis();
    
    // 返回此静态连接点  一般就为当前的Method(至少目前的唯一实现是MethodInvocation,所以连接点得静态部分肯定就是本方法)
    AccessibleObject getStaticPart();
}
```
几个重要的子接口和实现类，如下：   


# 2. AOP源码分析
spring中大名鼎鼎的AOP，我们只知道使用简单的aspectJ进行配置织入，那它底层到底是怎么实现的呢？
spring动态织入aop的能力这么强大，那底层到底是在哪里做了什么手脚，才然我们任意织入的逻辑在指定的目标位置生效，从而达到拦截的效果呢？   

# 3. 

