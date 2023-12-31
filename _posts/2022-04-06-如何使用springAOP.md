---
layout:     post
title:      spring AOP该怎么使用呢？
subtitle:   快速上手使用spring AOP的能力
categories: [spring AOP专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 什么是AOP？

# 2. 什么是Aspectj?

# 3.使用Aspectj实现AOP
请注意，spring内部并没有使用AspectJ静态织入的方式实现AOP，它导入AspectJ的jar包实际上只是支持了AspectJ提供的注解，但它通过解析AspectJ的注解来实现AOP仍旧是使用jdk或者cglib提供的动态织入的方式。   

（1）在spring工程pom中引入AspectJ的jar包   
可能和spring的版本有关系，高版本可能已经自动集成了aspectJ的jar。   
同样,springboot中默认集成了aspectJ，因此在springboot中不需要显式导入。   
```xml
        <!-- aspectj aop注解开发 -->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.9</version>
        </dependency>
```

（2）在spring xml配置中开启使用AspectJ的注解方式：   
```xml
    <!-- spring bean xml中要想使用切面注解@aspectJ，必须添加如下配置 -->
    <aop:aspectj-autoproxy/>
```
（3）使用AspectJ提供的注解标识一个切面   
```java
package zeh.myspring.spring2annotation.code02annotationaop.aspect;

import com.alibaba.fastjson.JSONObject;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

// 通过AspectJ注解方式配置的Aspect切面类
// 注解方式不用实现指定的Advice通知接口,注解方式不用在springContext.xml中配置切入点Pointcut和通知Advice
@Component
// 注解方式标注为一个组件，将自动实例化该切面类的bean对象。
@Aspect
// 注解方式标注此类为Aspect切面类。
public class UseAnnotationAspect {
    // 目标方法前通知的切面
    // 目标方法执行之前执行下面的切面方法
    // 下述切面中的方法只要被指定的注解标注了， 就确认了切入点Pointcut和通知Advice
    // 下面的方法名称可以任意命名
    // execution()表达式是切面编程中用来定义切入点Pointcut的，execution()表达式的语法很灵活。
    @Before("execution(* zeh.myspring.spring2annotation.code02annotationaop.service.AopUseAnnotationServiceImpl.withAop()) ")
    public Object beforeMethod(JoinPoint jp) throws Throwable {
        System.out.println("====>进入Before方法前切面");
        System.out.println("@Before：执行目标方法之前的切面方法....");
        System.out.println("@Before：目标方法为--" + jp.getSignature().getName());
        System.out.println("====>结束Before方法前切面");
        return jp;
    }

    // --目标方法正常执行完毕后执行下面的切面方法
    // 下面的方法名称可以任意命名
    @AfterReturning(value = "execution(* zeh.myspring.spring2annotation.code02annotationaop.service.AopUseAnnotationServiceImpl.withAop()) ", returning = "result")
    public Object afterReturningMethod(JoinPoint jp, Object result)
            throws Throwable {
        System.out.println("====>进入AfterReturning方法后切面，目标方法正常执行完毕");
        System.out.println("@AfterReturning：目标方法正常执行完毕后的切面方法....");
        System.out.println("@AfterReturning：目标方法为--"
                + jp.getSignature().getName() + ";目标方法的返回结果 --" + result);
        System.out.println("====>结束AfterReturning方法后切面，目标方法正常执行完毕");
        return jp;
    }

    // 目标方法执行完毕后执行下面的切面方法,不管目标方法是否出现异常
    // 下面的方法名称可以任意命名
    @After("execution(* zeh.myspring.spring2annotation.code02annotationaop.service.AopUseAnnotationServiceImpl.withAop()) ")
    public Object afterMethod(JoinPoint jp) throws Throwable {
        System.out.println("====>进入After方法后切面");
        System.out.println("@After：目标方法执行完毕后执行下面的切面方法,不管目标方法是否出现异常后的切面方法....");
        System.out.println("@After：目标方法为--" + jp.getSignature().getName());
        System.out.println("====>结束After方法后切面");
        return jp;
    }

    // 目标方法出现异常后执行下面的切面方法
    // 下面的方法名称可以任意命名
    @AfterThrowing(value = "execution(* zeh.myspring.spring2annotation.code02annotationaop.service.AopUseAnnotationServiceImpl.withAop()) ", throwing = "e")
    public Object afterThorwingMethod(JoinPoint jp, Exception e)
            throws Throwable {
        System.out.println("====>进入AfterThrowing异常切面");
        System.out.println("@AfterThrowing：目标方法正常执行完毕后的切面方法....");
        System.out.println("@AfterThrowing：目标方法为--"
                + jp.getSignature().getName() + ";目标方法的异常信息--" + e);
        System.out.println("====>结束AfterThrowing异常切面");
        return jp;
    }

    // 环绕目标方法的通知的切面
    // 下面的方法名称可以任意命名
    // ProceedingJoinPoint类只支持@Around注解这个Advice
    // @Around注解是环绕注解，可以统一方法前的Pointcut、方法后Pointcut和异常Pointcut。因为ProceedingJoinPoint类可以直接确定何时执行目标方法。
    @Around("execution(* zeh.myspring.spring2annotation.code02annotationaop.service.AopUseAnnotationServiceImpl.withAop()) ")
    public Object around(ProceedingJoinPoint jp) throws Throwable {
        String methodName = jp.getSignature().getName();// 通过jp对象获取当前目标方法的名称，该名称就是@Around()注解中配置的目标方法名称；

        // 目标方法执行结果
        JSONObject result = null;
        try {
            System.out.println("====>进入Around环绕切面");
            System.out.println("@Around：目标方法执行前的切面方法....，目标方法名称：" + methodName);
            /* 执行目标方法 */
            System.out.println("@Around：开始执行目标方法....");
            result = (JSONObject) jp.proceed();
            System.out.println("@Around：目标方法执行结束....");
            System.out.println("@Around：目标方法正常执行完毕后的切面方法....，目标方法名称："
                    + methodName);
        } catch (Exception e) {
            System.out.println("@Around：目标方法出现异常时的切面方法....，目标方法名称："
                    + methodName);
        }
        System.out.println("@Around：目标方法执行完毕后的切面方法，不管是否出现异常....，目标方法名称："
                + methodName);
        System.out.println("====>结束Around环绕切面");

        JSONObject logObject = new JSONObject();
        logObject.put("requestLog", "我是业务响应的日志");
        result.put("logResult", logObject);
        return result;
    }

}

```

（4）定义aop要代理的目标接口   
注意，因为我们要代理的真实对象存在接口，因此，spring底层会默认选择以jdk的方式来实现动态代理。   
```java
package zeh.myspring.spring2annotation.code02annotationaop.service;

import com.alibaba.fastjson.JSONObject;

// aop业务接口
public interface IAopUseAnnotationService {

    JSONObject withAop() throws Exception;

    String withOutAop() throws Exception;
}

```

（5）定义aop业务接口的实现   
```java
package zeh.myspring.spring2annotation.code02annotationaop.service;

import com.alibaba.fastjson.JSONObject;
import org.springframework.stereotype.Component;

// aop业务接口的实现
// 将service的实现类交给IOC容器管理，Ioc容器将采用反射自动实例化该实例
@Component("aopUseAnnotation")
public class AopUseAnnotationServiceImpl implements IAopUseAnnotationService {
    @Override
    public JSONObject withAop() throws Exception {
        System.out.println("withAop方法执行...");
        JSONObject result = new JSONObject();
        result.put("result", "我是withAop()方法，我被绑定了切面，所以我是目标方法");
        return result;
    }

    @Override
    public String withOutAop() throws Exception {
        System.out.println("withOutAop方法执行...");
        return "我是withOutAop()方法，我没有被绑定切面，所以我不是目标方法";
    }
}

```

（6）测试运行：   
```java
package zeh.myspring.spring2annotation.code02annotationaop;

import com.alibaba.fastjson.JSONObject;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import zeh.myspring.spring2annotation.code02annotationaop.service.IAopUseAnnotationService;

// 使用注解的方式实现aop
// 之前通过传统的bean xml方式实现aop，比较繁琐，现在使用注解的方式实现。
public class RunAopByAnnotationJunit {

    @Test
    public void testAopByAnnotation() {
        String beanXml = "springannotation/code02-applicationContext-annotation-aop.xml";
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext(beanXml);
        // 获取bean对象，此时的bean对象是代理对象
        IAopUseAnnotationService service = applicationContext.getBean("aopUseAnnotation", IAopUseAnnotationService.class);
        System.out.println(String.format("bean class is %s", service.getClass()));
        try {
            JSONObject result1 = service.withAop();// 执行配置了切面注解的目标方法
            String result2 = service.withOutAop();// 执行没有配置切面注解的目标方法
            System.out.println("最终响应结果result1:" + result1.toString());
            System.out.println("最终响应结果result2:" + result2);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

（7）运行结果：   
```
C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\bin\java.exe -ea -Didea.test.cyclic.buffer.size=1048576 "-javaagent:C:\D\soft\soft_application\onlyou-app\intellij-idea\IntelliJ IDEA 2018.2.5\lib\idea_rt.jar=63540:C:\D\soft\soft_application\onlyou-app\intellij-idea\IntelliJ IDEA 2018.2.5\bin" -Dfile.encoding=UTF-8 -classpath "C:\D\soft\soft_application\onlyou-app\intellij-idea\IntelliJ IDEA 2018.2.5\lib\idea_rt.jar;C:\D\soft\soft_application\onlyou-app\intellij-idea\IntelliJ IDEA 2018.2.5\plugins\junit\lib\junit-rt.jar;C:\D\soft\soft_application\onlyou-app\intellij-idea\IntelliJ IDEA 2018.2.5\plugins\junit\lib\junit5-rt.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\charsets.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\deploy.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\access-bridge-64.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\cldrdata.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\dnsns.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\jaccess.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\jfxrt.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\localedata.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\nashorn.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\sunec.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\sunjce_provider.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\sunmscapi.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\sunpkcs11.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\ext\zipfs.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\javaws.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\jce.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\jfr.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\jfxswt.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\jsse.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\management-agent.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\plugin.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\resources.jar;C:\D\soft\soft_application\jdk\jdk_window\jdk1.8\jre\lib\rt.jar;C:\D\zhaoehcode-java\mySpring\target\classes;C:\D\soft\soft_application\maven\repo\org\springframework\spring-context\4.3.13.RELEASE\spring-context-4.3.13.RELEASE.jar;C:\D\soft\soft_application\maven\repo\org\springframework\spring-aop\4.3.13.RELEASE\spring-aop-4.3.13.RELEASE.jar;C:\D\soft\soft_application\maven\repo\org\springframework\spring-beans\4.3.13.RELEASE\spring-beans-4.3.13.RELEASE.jar;C:\D\soft\soft_application\maven\repo\org\springframework\spring-core\4.3.13.RELEASE\spring-core-4.3.13.RELEASE.jar;C:\D\soft\soft_application\maven\repo\commons-logging\commons-logging\1.2\commons-logging-1.2.jar;C:\D\soft\soft_application\maven\repo\org\springframework\spring-expression\4.3.13.RELEASE\spring-expression-4.3.13.RELEASE.jar;C:\D\soft\soft_application\maven\repo\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\D\soft\soft_application\maven\repo\junit\junit\4.11\junit-4.11.jar;C:\D\soft\soft_application\maven\repo\org\hamcrest\hamcrest-core\1.3\hamcrest-core-1.3.jar;C:\D\soft\soft_application\maven\repo\com\alibaba\fastjson\1.2.12\fastjson-1.2.12.jar;C:\D\soft\soft_application\maven\repo\org\aspectj\aspectjweaver\1.8.9\aspectjweaver-1.8.9.jar" com.intellij.rt.execution.junit.JUnitStarter -ideVersion5 -junit4 zeh.myspring.spring2annotation.code02annotationaop.RunAopByAnnotationJunit,testAopByAnnotation
log4j:WARN No appenders could be found for logger (org.springframework.core.env.StandardEnvironment).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
bean class is class com.sun.proxy.$Proxy12
====>进入Around环绕切面
@Around：目标方法执行前的切面方法....，目标方法名称：withAop
@Around：开始执行目标方法....
====>进入Before方法前切面
@Before：执行目标方法之前的切面方法....
@Before：目标方法为--withAop
====>结束Before方法前切面
withAop方法执行...
@Around：目标方法执行结束....
@Around：目标方法正常执行完毕后的切面方法....，目标方法名称：withAop
@Around：目标方法执行完毕后的切面方法，不管是否出现异常....，目标方法名称：withAop
====>结束Around环绕切面
====>进入After方法后切面
@After：目标方法执行完毕后执行下面的切面方法,不管目标方法是否出现异常后的切面方法....
@After：目标方法为--withAop
====>结束After方法后切面
====>进入AfterReturning方法后切面，目标方法正常执行完毕
@AfterReturning：目标方法正常执行完毕后的切面方法....
@AfterReturning：目标方法为--withAop;目标方法的返回结果 --{"result":"我是withAop()方法，我被绑定了切面，所以我是目标方法","logResult":{"requestLog":"我是业务响应的日志"}}
====>结束AfterReturning方法后切面，目标方法正常执行完毕
withOutAop方法执行...
最终响应结果result1:{"result":"我是withAop()方法，我被绑定了切面，所以我是目标方法","logResult":{"requestLog":"我是业务响应的日志"}}
最终响应结果result2:我是withOutAop()方法，我没有被绑定切面，所以我不是目标方法

Process finished with exit code 0

```

# 4. AOP实现源码

