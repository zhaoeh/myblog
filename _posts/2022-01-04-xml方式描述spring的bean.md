---
layout:     post
title:      xml方式描述spring的bean
subtitle:   我们先学习传统的xml方式描述spring的bean
categories: [spring]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. xml中bean定义详解
被spring管理的java对象统称为java bean。   
我们在程序中需要用到很多java对象，之前都是自己new出来然后使用；应用spring之后，spring就是个容器，可以将我们需要用到的java对象列一个清单，交给spring容器。   
spring会帮我们创建、管理、组装清单中的bean对象，放置在IOC容器中。   
当我们需要使用某个java对象时，我们只需要从spring容器中根据目标bean的名称进行获取即可。   
也就是说，当我们向spring容器中提供这份清单时，是需要提供给spring一个目标bean的名称，这样spring才能将实例化好的bean对象和我们提供的bean名称之间进行映射绑定。   
当我们需要使用某个目标bean时，我们只需要将我们提供的目标bean的名称提交给spring容器，spring就可以根据这个bean名称查找到指定的目标bean实例返回给我们使用。   
我们提供给spring的bean名称，在整个spring容器中都必须是唯一的，否则spring容器将不知道该返回哪个bean实例。   

## 1.1 bean xml 配置文件格式
从前面知道，我们提交给spring的bean列表清单文件，支撑xml文件格式和java注解的方式去提供。   
我们先来看xml文件的格式去提供bean配置元数据定义信息。   
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.2.xsd">
    <import resource="引用其他bean xml配置文件"/>
    <bean id="bean名称标识，spring容器中唯一" class="当前bean的类型，即权限定名" />
    <alias name="bean名称表示" alias="bean别名"/>
</beans>
```
beans是根元素，下面可以包含任意数量的import、bean、alias元素。   

## 1.2 bean元素
bean元素用来定义一个bean对象。   
格式如下：   
```xml
<bean id="bean名称标识，spring容器中唯一" name="bean别名" class="当前bean的类型，即权限定名" factory-bean="工厂bean名称" factory-method="工厂方法"/>
```
bean名称:   
每个bean都必须有一个名称，通过id指定，在整个spring容器中bean名称必须唯一，否则spring装载bean会报错。spring通过bean名称可以从其容器中获取对应的bean对象。   
   
bean别名:   
就和人的外号一样，一个人有自己的名称，也可能有很多外号，当别人通过这个人的名称和这个人的外号都可以找到这个人。   
外号就相当于这个人的别名。   
spring管理的bean既可以有自己的名称，也可以有多个别名。spring允许使用者通过名称或者别名获取容器中对应的bean实例对象。   

bean的名称和别名的定义规则:   
bean名称和别名可以通过bean元素中的id和name去定义，具体定义规则如下：   
1.  当id存在时，name不存在，spring在实例化bean时，取id为bean的名称。   
2.  当id不存在，此时需要看name，name的值可以通过<font color="#a52a2a">,;或者空格</font>分割，最后会按照分割符得到一个String数组，数组的第一个元素作为bean的名称，其他的作为bean的别名。   
3.  当id和name都存时，id为bean的名称，name用来定义多个别名。   
4.  当id和name都不指定时，bean名称会自动生成，别名不会生成。   

bean名称和别名的各种写法:   
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.2.xsd">
    <!-- 通过id定义bean名称为person-->
    <bean id="person1" class="com.zeh.main.pojo.Person" />
    <!-- 当id不存在，name存在时，则此时name的第一个名称作为bean名称，此时没有别名。bean名称为person -->
    <bean name="person2" class="com.zeh.main.pojo.Person" />
    <!-- 当id和name都存在，此时id定义bean的名称，name定义bean的别名。bean名称为person，bean别名只有一个person2 -->
    <bean id="person3" name="person3_1" class="com.zeh.main.pojo.Person" />
    <!-- bean名称为person，bean别名有多个，分别为person2,person3,person4,person5 -->
    <bean id="person4" name="person4_1,person4_2;person4_3 person4_5" class="com.zeh.main.pojo.Person" />
    <!-- bean名称为person，bean别名有多个，分别为person2,person3,person4,person5 -->
    <bean name="person5,person5_1,person5_2;person5_3 person5_4" class="com.zeh.main.pojo.Person" />
</beans>
```

写个Java程序获取上面的bean名称和别名:   
```java
package com.zeh.main.controller;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import java.util.Arrays;
/**
 * 功能描述
 *
 * @author zWX5331241
 * @since 2021-04-25
 */
public class PersonController {
    public static void main(String[] args) {
        String beanXml = "classpath:/spring/applicationContext.xml";
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext(beanXml);
        for (String beanName : Arrays.asList("person1", "person2", "person3", "person4", "person5")) {
            // 根据bean名称获取当前bean名称配置的所有别名
            String[] aliases = applicationContext.getAliases(beanName);
            System.out.println(String.format("beanName：%s，别名：[%s]", beanName, String.join(",", aliases)));
        }
        System.out.println("spring容器中所有的bean名称如下：");
        for (String beanName : applicationContext.getBeanDefinitionNames()) {
            // 根据bean名称获取当前bean名称配置的所有别名
            String[] aliases = applicationContext.getAliases(beanName);
            System.out.println(String.format("beanName：%s，别名：[%s]", beanName, String.join(",", aliases)));
        }
        int beanCount = applicationContext.getBeanDefinitionCount();
        System.out.println(String.format("spring容器中的bean总数 is %s", beanCount));
    }
}
```

输出结果:   
```youtrack
beanName：person1，别名：[]
beanName：person2，别名：[]
beanName：person3，别名：[person3_1]
beanName：person4，别名：[person4_1,person4_3,person4_2,person4_5]
beanName：person5，别名：[person5_2,person5_1,person5_4,person5_3]
spring容器中所有的bean名称如下：
beanName：person1，别名：[]
beanName：person2，别名：[]
beanName：person3，别名：[person3_1]
beanName：person4，别名：[person4_1,person4_3,person4_2,person4_5]
beanName：person5，别名：[person5_2,person5_1,person5_4,person5_3]
spring容器中的bean总数 is 5
```
上面程序中的新方法:   
```youtrack
getAliases：通过beanName获取bean的别名。
getBeanDefinitionNames：返回spring容器中所有bean的名称。
getBeanDefinitionCount：返回spring容器中bean的数量。
```

<font color="#FF0000">  当bean xml中的id和name都未指定时: </font>

当id和name都未指定时，bean的名称和别名又是什么呢？   
此时spring容器会自动生成bean的名称和别名。   
spring自动生成的bean的名称规则如下：   
```youtrack
bean的class完整类名#编号
```
上面的编号是从0开始的，相同class类型的bean没有指定名称的依次递增。   
spring自动生成的bean的别名规则如下：
```youtrack
bean的class完整类名
```
相同class类型的bean没有指定名称，只会为第一个，即上面编号为0的bean生成别名，其他的同类型bean的自动别名不生成。   

修改applicationContext.xml文件，添加两个相同类型的bean元素，不指定id和name：   
```xml
<!-- id和name都不指定 -->
<bean class="com.zeh.main.pojo.Person" />
<!-- id和name都不指定 -->
<bean class="com.zeh.main.pojo.Person" />
```
执行程序，结果如下：   
```youtrack
beanName：person1，别名：[]
beanName：person2，别名：[]
beanName：person3，别名：[person3_1]
beanName：person4，别名：[person4_1,person4_3,person4_2,person4_5]
beanName：person5，别名：[person5_2,person5_1,person5_4,person5_3]
spring容器中所有的bean名称如下：
beanName：person1，别名：[]
beanName：person2，别名：[]
beanName：person3，别名：[person3_1]
beanName：person4，别名：[person4_1,person4_3,person4_2,person4_5]
beanName：person5，别名：[person5_2,person5_1,person5_4,person5_3]
beanName：com.zeh.main.pojo.Person#0，别名：[com.zeh.main.pojo.Person]
beanName：com.zeh.main.pojo.Person#1，别名：[]
spring容器中的bean总数 is 7
```
倒数第2第3行，为Person类自动生成了名称，分别为 com.zeh.main.pojo.Person#0 和 com.zeh.main.pojo.Person#1。因为是相同的类型，即Person，所以只会第一个bean实例生成了别名，别名为com.zeh.main.pojo.Person。   

## 1.3 alias元素
alias元素也可以用来给某个bean定义别名，语法为：   
```xml
<alias name="需要定义别名的bean名称" alias="别名"/>
```

修改applicationContext.xml:   
```xml
<!-- alias标签使用 -->
<alias name="person1" alias="person1_11"/>
<alias name="person2" alias="person2_22"/>
<alias name="person3" alias="person3_33"/>
<alias name="person4" alias="person4_44"/>
<alias name="person5" alias="person5_55"/>
```
运行主程序，结果如下：   
```youtrack
beanName：person1，别名：[person1_11]
beanName：person2，别名：[person2_22]
beanName：person3，别名：[person3_33,person3_1]
beanName：person4，别名：[person4_44,person4_1,person4_3,person4_2,person4_5]
beanName：person5，别名：[person5_2,person5_1,person5_4,person5_3,person5_55]
spring容器中所有的bean名称如下：
beanName：person1，别名：[person1_11]
beanName：person2，别名：[person2_22]
beanName：person3，别名：[person3_33,person3_1]
beanName：person4，别名：[person4_44,person4_1,person4_3,person4_2,person4_5]
beanName：person5，别名：[person5_2,person5_1,person5_4,person5_3,person5_55]
beanName：com.zeh.main.pojo.Person#0，别名：[com.zeh.main.pojo.Person]
beanName：com.zeh.main.pojo.Person#1，别名：[]
spring容器中的bean总数 is 7
```

## 1.4 import元素
当系统体量比较大的时候，会分成很多模块，每个模块都会对应一个bean xml文件，这时候我们可以在一个总的bean xml文件中对其他bean xml进行汇总。   
相当于把多个bean xml的内容合并到一个里面了，可以通过import元素引入其他bean xml配置文件。   
语法：   
```xml
<import resource="其他配置文件的位置"/>
```
比如：   
```xml
<import resource="otherBean.xml"/>
```
import元素导入外部bean xml后，就相当于把外部所有的配置引入到当前文件中来了。   