---
layout:     post
title:      bean中的autowire-candidate属性又是干什么的？
subtitle:   autowire-candidate属性的作用分析
categories: [spring专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. autowire-candidate做什么事情？
上一篇文章[primary能解决什么问题](https://zhaoeh.github.io/myblog/2022/01/10/primary%E8%83%BD%E8%A7%A3%E5%86%B3%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98/)中遇到的问题，我们再来回顾一下，当容器中某种类型的bean存在多个实例的时候，此时我们如果从容器中查找这种类型的bean对象，会报下面这个异常：
```youtrack
org.springframework.beans.factory.NoUniqueBeanDefinitionException
```
<font color="#FF0000">原因：当从容器中按照类型查找一个bean对象的时候，容器中却找到了多个匹配到的bean实例，此时spring也蒙圈了不知道该如何选择了，就会报这个异常。 </font>

这种异常主要出现在两种场景中：   
## 1.1 场景1
从容器中查找符合指定类型的bean实例，对应BeanFactory下面的方法：
```java
<T> T getBean(Class<T> requiredType) throws BeansException;
```
## 1.2 场景2
自动注入方式设置为byType的时候，如下：   
```java
package test;
public class NormalBean {
    public interface IService{}
    public static class ServiceA implements IService{}
    public static class ServiceB implements IService{}
    private IService service;
    public void setService(IService service){
        this.service = service;
    }
}
```
xml如下：
```xml
<bean id="serviceA" class="test.NormalBean.ServiceA"/>
<bean id="serviceB" class="test.NormalBean.ServiceB"/>
<bean id="setBean" class="test.NormalBean" autowire="byType"/>
```
setBean的autowire设置为byType，即按照setter方法的参数类型自动注入，setBean的setService的参数类型是IService，而IService类有两个实现类：ServiceA和ServiceB。   
而容器中刚好管理了这两个实现类的bean：serviceA和serviceB实例。   
所以上面代码会报错，不知道该选择哪个实例对象进行注入。   
<font color="#FF0000">我们可以通过primary属性来指定一个主要的bean，当从容器中查找的时候，如果有多个实例bean符合查找的类型，此时容器将返回primary属性为"true"的bean对象。</font>

## 1.3 autowire-candidate
正如上面存在的问题，spring除了使用primary属性来解决，还有一种方法也可以解决这种问题。   
可以设置某个bean是否在自动注入的时候作为候选者bean，通过bean标签的autowire-candidate属性来配置，如下：
```xml
<bean id="serviceA" class="test.NormalBean.ServiceA" autowire-candidate="false"/>
```
autowire-candidate:设置当前bean在被其他对象作为自动注入对象的是偶，是否作为候选者bean，其默认值是true。   
来举例说明一下，以上面的setBean注入的案例说明一下注入的过程：   
<font color="#FF0000">容器在创建setBean的时候，发现其autowire属性为byType，即按类型自动注入，此时会在NormalBean类中查找所有setter方法列表，其中就包含了setService方法，setService方法的参数类型为IService，然后就会去容器中按照IService类型查找所有符合条件的bean列表，此时容器中会返回满足IService这种类型并且autowire-candidate="true"的bean，刚才说过，bean标签的autowire-candidate的默认值是true，所以容器中符合条件的bean有两个：serviceA和serviceB,setService方法只需要一个满足条件的bean，此时会再去看这个bean列表中是否只有一个主要的bean（即bean元素的primary="true"的bean），而bean元素的primary默认值都是false，所以没有primary为true的bean，此时spring容器懵逼了，不知道选择哪个bean注入了，抛出NoUniqueBeanDefinitionException异常！</font>

从上面过程中可以看出将某个bean的primary属性设置为true就可以解决问题了。   
或者只保留一个bean的auto-candidate属性为true，将其余满足条件的bean的autowire-candidate设置为false，此时也可以解决多个bean实例的问题。   
