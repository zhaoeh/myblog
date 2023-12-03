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
![](/images/myBlog/2022-04-03-01.jpg)   

<b>Invocation接口</b>   
此接口表示程序中的调用，调用是一个连接点，可以被拦截器拦截。   
```java
package org.aopalliance.intercept;

// 此接口表示程序中的调用
// 调用是一个连接点，可以被拦截器拦截。
public interface Invocation extends Joinpoint {
    
    // 将参数作为数组对象获取，可以更改此数组中的元素值以更改参数。
    // 通常用来获取调用目标方法的参数
    Object[] getArguments();
}
```

<b>MethodInvocation接口</b>   
用来表示连接点中方法的调用，可以获取调用过程中的目标方法。   
```java
package org.aopalliance.intercept;
import java.lang.reflect.Method;

// 方法调用的描述，在方法调用时提供给拦截器。
// 方法调用是一个连接点，可以被方法拦截器拦截。
public interface MethodInvocation extends Invocation {
    // 返回正在被调用得方法~~~  返回的是当前Method对象。
    // 此时，效果同父类的AccessibleObject getStaticPart() 这个方法
    Method getMethod();
}
```
<b>ProxyMethodInvocation接口</b>   
表示代理方法的调用   
```java
public interface ProxyMethodInvocation extends MethodInvocation {
    // 获取被调用的代理对象
    Object getProxy();
    
    // 克隆一个方法调用器MethodInvocation
    MethodInvocation invocableClone();
    
    // 克隆一个方法调用器MethodInvocation，并为方法调用器指定参数
    MethodInvocation invocableClone(Object... arguments);
    
    // 设置要用于此链中任何通知的后续调用的参数。
    void setArguments(Object... arguments);
    
    // 添加一些扩展用户属性，这些属性不在AOP框架内使用。它们只是作为调用对象的一部分保留，用于特殊的拦截器。
    void setUserAttribute(String key, @Nullable Object value);
    
    // 根据key获取对应的用户属性
    @Nullable
    Object getUserAttribute(String key);
}
```
通俗点理解：连接点表示方法的调用过程，内部包含了方法调用过程中的所有信息，比如被调用的方法、目标、代理对象、执行拦截器链等信息。   
上面定义都是一些接口，最终有2个实现。   
<b>ReflectiveMethodInvocation</b>      
当代理对象是采用jdk动态代理创建的，通过代理对象来访问目标对象的方法的时，最终过程是由ReflectiveMethodInvocation来处理的，内部会通过递归调用方法拦截器，最终会调用到目标方法。   
<b>CglibMethodInvocation</b>   
功能和上面的类似，当代理对象是采用cglib创建的，通过代理对象来访问目标对象的方法的时，最终过程是由CglibMethodInvocation来处理的，内部会通过递归调用方法拦截器，最终会调用到目标方法。   

这2个类源码稍后详解。   

## 1.2 通知相关的类
通知用来定义需要增强的逻辑。   
![](/images/myBlog/2022-04-03-02.jpg)   

<b>Advice接口</b>   
通知的底层接口   
```java
package org.aopalliance.aop;
public interface Advice {
}
```

<b>BeforeAdvice接口</b>   
方法前置通知，内部空的   
```java
package org.springframework.aop;
public interface BeforeAdvice extends Advice {
}
```

<b>Interceptor接口</b>   
此接口表示通用拦截器   
```java
package org.aopalliance.intercept;
public interface Interceptor extends Advice {
}
```

<b>MethodInterceptor接口</b>    
方法拦截器，所有的通知均需要转换为MethodInterceptor类型的，最终多个MethodInterceptor组成一个方法拦截器链。   
```java
package org.aopalliance.intercept;
@FunctionalInterface
public interface MethodInterceptor extends Interceptor {
    // 拦截目标方法的执行，可以在这个方法内部实现需要增强的逻辑，以及主动调用目标方法
    Object invoke(MethodInvocation invocation) throws Throwable;
}
```

<b>AfterAdvice接口</b>      
后置通知的公共标记接口:      
```java
package org.springframework.aop;
public interface AfterAdvice extends Advice {
}
```

<b>MethodBeforeAdvice接口</b>   
方法执行前通知，需要在目标方法执行前执行一些逻辑的，可以通过这个实现。      
通俗点说：需要在目标方法执行之前增强一些逻辑，可以通过这个接口来实现。before方法：在调用给定方法之前回调。      
```java
package org.springframework.aop;
public interface MethodBeforeAdvice extends BeforeAdvice {
    // 调用目标方法之前会先调用这个before方法
    // method：需要执行的目标方法
    // args：目标方法的参数
    // target：目标对象
    void before(Method method, Object[] args, @Nullable Object target) throws Throwable;
}
```
如同:   
```java
public Object invoke(){
    调用MethodBeforeAdvice#before方法
    return 调用目标方法;
}
```

<b>AfterReturningAdvice接口</b>      
方法执行后通知，需要在目标方法执行之后执行增强一些逻辑的，可以通过这个实现。      
不过需要注意一点：目标方法正常执行后，才会回调这个接口，当目标方法有异常，那么这通知会被跳过。       
```java
package org.springframework.aop;
public interface AfterReturningAdvice extends AfterAdvice {
    // 目标方法执行之后会回调这个方法
    // method：需要执行的目标方法
    // args：目标方法的参数
    // target：目标对象
    void afterReturning(@Nullable Object returnValue, Method method, Object[] args, @Nullable Object target) throws Throwable;
}
```
如同:   
```java
public Object invoke(){
    Object retVal = 调用目标方法;
    调用AfterReturningAdvice#afterReturning方法
    return retVal;
}
```
<b>ThrowsAdvice接口</b>      
```java
package org.springframework.aop;
public interface ThrowsAdvice extends AfterAdvice {
}
```
此接口上没有任何方法，因为方法由反射调用，实现类必须实现以下形式的方法，前3个参数是可选的，最后一个参数为需要匹配的异常的类型。        
```java
void afterThrowing([Method, args, target], ThrowableSubclass);
```
有效方法的一些例子如下：    
```java
public void afterThrowing(Exception ex)
public void afterThrowing(RemoteException)
public void afterThrowing(Method method, Object[] args, Object target, Exception ex)
public void afterThrowing(Method method, Object[] args, Object target, ServletException ex)
```


      



# 2. AOP源码分析
spring中大名鼎鼎的AOP，我们只知道使用简单的aspectJ进行配置织入，那它底层到底是怎么实现的呢？
spring动态织入aop的能力这么强大，那底层到底是在哪里做了什么手脚，才然我们任意织入的逻辑在指定的目标位置生效，从而达到拦截的效果呢？   

# 3. 

