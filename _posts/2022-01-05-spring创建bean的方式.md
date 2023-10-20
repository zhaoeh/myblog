---
layout:     post
title:      spring创建bean的方式
subtitle:   spring管理bean对象，都通过哪些方式创建bean实例呢？
date:       2022-01-05
author:     zhaoeh
header-img: img/post-bg-myself9.jpg
catalog: true
tags:
    - spring
---

# 1. spring创建bean实例
## 1.1 spring创建bean实例的方式
spring容器内部创建bean实例对象常见的有如下4种方式：   
```youtrack
1.  通过反射调用构造方法创建bean对象。
2.  通过静态工厂方法创建bean对象。
3.  通过实例工厂方法创建bean对象。
4.  通过FactoryBean创建bean对象。
```
## 1.2 constructor-arg标签
<constructor-arg></constructor-arg>标签是<bean></bean>标签下的子标签。   
其主要是用来定义构造器参数的，有如下3种使用方式：   
1.  通过构造器参数序号，来指定构造器参数的初始化方式。   
```xml
<constructor-arg index="当前参数在构造器中的序号，指定0表示第一个参数，1表示第二个参数" value="当前属性如果是简单类型则表示当前属性的值" ref="当前属性如果是引用类型则表示引用的bean名称"/>
```
2.  通过构造器参数名称，来指定构造器参数的初始化方式。   
```xml
<constructor-arg name="当前参数在构造器中的参数名称" value="当前属性如果是简单类型则表示当前属性的值" ref="当前属性如果是引用类型则表示引用的bean名称"/>
```
3.  通过构造器参数类型，来指定构造器参数的初始化方式。   
```xml
<constructor-arg type="当前参数在构造器中的参数类型" value="当前属性如果是简单类型则表示当前属性的值" ref="当前属性如果是引用类型则表示引用的bean名称"/>
```
其中，value和ref只能二选一，当参数是简单类型时，通过value指定参数实际值；当参数是引用类型时，通过ref指定引用的bean名称。   

# 2. spring 4种创建对象方式详解
## 2.1 通过反射调用构造方法创建bean对象？
调用bean的指定构造器来反射创建bean实例，是用的最多的方式。   
这种方式只需要在bean xml中的bean元素里面指定class属性，spring容器内部会根据constructor-arg去自动匹配指定参数索引、或者指定参数名称或者指定参数类型的构造器去创建bean实例。   
如果不指定constructor-arg元素表示调用的是默认无参构造器，则此时要求被创建的bean必须存在无参构造器。   

语法:   
```xml
<bean id="bean名称" name="bean名称或者别名" class="bean的完整类路径">
    <constructor-arg index="0" value="bean的简单属性的值" type="该参数的类型" name="bean的属性名称" ref="该属性引用的其他bean名称"/>
    <constructor-arg index="1" value="bean的简单属性的值" type="该参数的类型" name="bean的属性名称" ref="该属性引用的其他bean名称"/>
    <constructor-arg index="2" value="bean的简单属性的值" type="该参数的类型" name="bean的属性名称" ref="该属性引用的其他bean名称"/>
    ...
    <constructor-arg index="n" value="bean的简单属性的值" type="该参数的类型" name="bean的属性名称" ref="该属性引用的其他bean名称"/>
</bean>
```
其中，constructor-arg用于指定构造方法参数的值。   
index、name、type三选一，可以选择使用参数索引来匹配，也可以使用参数名称来匹配，也可以使用参数类型来匹配。index指定构造方法中参数的位置，从0开始，依次递增即可。   
value和ref二选一，当构造器指定参数是简单类型时，使用value设置值；当是其他引用bean时，使用ref设置值。   

案例，bean xml配置:
```xml
<bean id="person1" class="com.zeh.main.pojo.Person" />
<bean id="person2" class="com.zeh.main.pojo.Person">
    <constructor-arg name="name" value="eric" type="java.lang.String" />
    <constructor-arg name="age" value="22" type="int" />
</bean>
<bean id="person3" class="com.zeh.main.pojo.Person">
    <constructor-arg index="0" value="daisy" type="java.lang.String" />
    <constructor-arg index="1" value="18" type="int" />
</bean>
```
其中 constructor-arg 元素用来指定目标bean的构造器中对应的构造参数，传入指定构造参数，spring会通过反射目标bean的指定构造器去创建目标bean对象。   

测试程序：   
```java
public class PersonController {
    public static void main(String[] args) {
        String beanXml = "classpath:/spring/applicationContext.xml";
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext(beanXml);
        for (String beanName : applicationContext.getBeanDefinitionNames()) {
            System.out.println(String.format("beanName is %s; bean is [%s]", beanName, applicationContext.getBean(beanName)));
        }
    }
}
```

执行结果:
```youtrack
beanName is person1; bean is [name is null，age is 0]
beanName is person2; bean is [name is eric，age is 22]
beanName is person3; bean is [name is daisy，age is 18]
```

## 2.2 通过静态工厂创建bean对象
我们可以创建一个静态的工厂，内部提供一些静态方法专门用来帮我们生成我们需要的对象，然后将这些静态方法生成的bean实例交给spring管理即可。   

语法:   
```xml
<bean id="bean名称" name="bean名称或者别名" class="静态工厂完整类路径" factory-method="静态工厂的方法">
    <constructor-arg index="0" value="静态方法中的简单属性的值" type="该参数的类型" name="静态方法中的属性名称" ref="该属性引用的其他bean名称"/>
    <constructor-arg index="1" value="静态方法中的简单属性的值" type="该参数的类型" name="静态方法中的属性名称" ref="该属性引用的其他bean名称"/>
    <constructor-arg index="2" value="静态方法中的简单属性的值" type="该参数的类型" name="静态方法中的属性名称" ref="该属性引用的其他bean名称"/>
    ...
    <constructor-arg index="n" value="静态方法中的简单属性的值" type="该参数的类型" name="静态方法中的属性名称" ref="该属性引用的其他bean名称"/>
</bean>
```
class：指定静态工厂类的权限定名。   
factory-method：静态工厂中的静态方法，返回需要的bean对象。   
constructor-arg：用于指定静态方法中的参数值，用法和前面一样。      
spring会自动调用静态工厂的静态方法获取指定的bean实例对象，并将其放入spring容器中。      

案例，创建静态工厂类：
```java
   public class PersonStaticFactory {
   // 静态无参方法创建无参数bean实例
   public static Person buildPerson() {
       return new Person();
   }
   // 静态有参方法创建有参数bean实例
   public static Person buildPerson2(String name, int age) {
       return new Person(name, age);
   }
}
```
bean xml配置:
```xml
<!-- 通过静态工厂的无参静态方法创建bean对象 -->
<bean id="createPersonByStaticFactoryMethod1" class="com.zeh.main.PersonStaticFactory"
      factory-method="buildPerson" />
<!-- 通过静态工厂的有参静态方法创建有参bean对象 -->
<bean id="createPersonByStaticFactoryMethod2" class="com.zeh.main.PersonStaticFactory"
      factory-method="buildPerson2">
    <constructor-arg index="0" value="daisy" type="java.lang.String" />
    <constructor-arg index="1" value="18" type="int" />
</bean>
```
上述配置文件中，spring容器启动时会自动调用PersonStaticFactory工厂的工厂方法buildPerson去创建Person对象，将其作为createPersonByStaticFactoryMethod1名称对应的bean实例放入到spring容器中。   
会自动调用PersonStaticFactory工厂的工厂方法buildPerson2，并且传入两个指定参数，去创建Person对象，将其作为createPersonByStaticFactoryMethod2名称对应的bean实例放入到spring容器中。   

执行结果:   
```youtrack
beanName is createPersonByStaticFactoryMethod1; bean is [name is null，age is 0]
beanName is createPersonByStaticFactoryMethod2; bean is [name is daisy，age is 18]
```

## 2.3 通过实例工厂方法创建bean对象
spring通过调用某些实例bean的某些实例方法来创建bean对象然后放在spring容器中以供使用。   
语法:   
```xml
<bean id="bean名称" name="bean名称或者别名" factory-bean="需要调用的实例bean名称" factory-method="实例bean的实例方法">
    <constructor-arg index="0" value="实例方法中的简单属性的值" type="该参数的类型" name="实例方法中的属性名称" ref="该属性引用的其他bean名称"/>
    <constructor-arg index="1" value="实例方法中的简单属性的值" type="该参数的类型" name="实例方法中的属性名称" ref="该属性引用的其他bean名称"/>
    <constructor-arg index="2" value="实例方法中的简单属性的值" type="该参数的类型" name="实例方法中的属性名称" ref="该属性引用的其他bean名称"/>
    ...
    <constructor-arg index="n" value="实例方法中的简单属性的值" type="该参数的类型" name="实例方法中的属性名称" ref="该属性引用的其他bean名称"/>
</bean>
```
spring容器以factory-bean指定的bean名称去查找到对应的bean对象，然后调用该bean对象中factory-method指定的方法，将这个方法返回的bean实例对象作为当前bean对象放在容器中以供使用。   

案例，创建实例工厂类：
```java
public class PersonInstanceFactory {
    // 实例无参方法创建无参数bean实例
    public Person buildPerson() {
        return new Person();
    }
    // 实例有参方法创建有参数bean实例
    public Person buildPerson2(String name, int age) {
        return new Person(name, age);
    }
}
```
bean xml配置:
```xml
<!-- 创建实例工厂对象 -->
<bean id="personInstanceFactory" class="com.zeh.main.PersonInstanceFactory"/>
<!-- 通过实例工厂的无参静态方法创建bean对象 -->
<bean id="createPersonByInstanceFactoryMethod1" factory-bean="personInstanceFactory"
      factory-method="buildPerson" />
<!-- 通过静态工厂的有参静态方法创建有参bean对象 -->
<bean id="createPersonByInstanceFactoryMethod2" factory-bean="personInstanceFactory"
      factory-method="buildPerson2">
    <constructor-arg index="0" value="zhangsan" type="java.lang.String" />
    <constructor-arg index="1" value="22" type="int" />
</bean>
```
执行结果:
```youtrack
beanName is createPersonByInstanceFactoryMethod1; bean is [name is null，age is 0]
beanName is createPersonByInstanceFactoryMethod2; bean is [name is zhangsan，age is 22]
```

## 2.4 通过FactoryBean来创建bean对象
之前学习过了BeanFactory接口，BeanFactory接口是spring的顶层接口。   
而这里我们说的是FactoryBean，也是一个接口，这俩接口很容易搞混淆。   
FactoryBean可以让spring容器通过这个接口的实现类来创建我们需要的bean对象。   

FactoryBean接口源码:
```java
package org.springframework.beans.factory;
public interface FactoryBean<T> {
    // 返回创建好的bean对象
    T getObject() throws Exception;
    // 返回需要创建的bean对象的类型
    Class<?> getObjectType();
    // bean对象是否是单例的
    boolean isSingleton();
}
```
从FactoryBean的应用场景来看，这个接口实际上是spring暴露给客户端的一个回调接口。   
getObject方法由开发者去实现对象的创建逻辑，实现了之后return即可，便可以将创建好的bean对象返回给spring容器。   
getObjectType方法需要开发者去实现，返回我们创建的bean的类型，即对象的class对象。   
isSingleton方法需要开发者实现，表示通过这个接口的实现类创建的bean是否是单例的；如果自己覆盖为true，则创建的bean对象就是单例的；如果自己覆盖之后返回false，则客户端每次从spring容器中获取对象时都会调用这个接口的getObject()方法去生成bean对象。   

语法:
```xml
<bean id="bean名称" class="FactoryBean接口实现类" />
```

案例，创建FactoryBean接口实现类:   
```java
public class FactoryBeanImpl implements FactoryBean<Person> {
    @Override
    public Person getObject() {
        Person person = new Person("Lisi", 19);
        System.out.println("我是通过FactoryBean创建的bean对象");
        return person;
    }
    @Override
    public Class<?> getObjectType() {
        return Person.class;
    }
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

bean xml配置：
```xml
<bean id="createByFactoryBean" class="com.zeh.main.FactoryBeanImpl" />
```
测试程序:
```java
public class PersonController {
    public static void main(String[] args) {
        String beanXml = "classpath:/spring/applicationContext.xml";
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext(beanXml);
        // 多次从spring容器中获取目标bean createByFactoryBean
        Object bean1 = applicationContext.getBean("createByFactoryBean");
        Object bean2 = applicationContext.getBean("createByFactoryBean");
        System.out.println(String.format("bean1 is [%s]", bean1));
        System.out.println(String.format("bean2 is [%s]", bean2));
        System.out.println(bean1 == bean2);
    }
}
```
执行结果:
```youtrack
我是通过FactoryBean创建的bean对象
bean1 is [name is Lisi，age is 19]
bean2 is [name is Lisi，age is 19]
true
```
多次从spring容器中获取这个bean对象，发现获取的bean对象引用地址完全相同，说明多次获取的bean对象是同一个bean对象，即这个bean实例是单例的。   

修改isSingleton()方法，让其返回false，运行结果如下:   
```youtrack
我是通过FactoryBean创建的bean对象
我是通过FactoryBean创建的bean对象
bean1 is [name is Lisi，age is 19]
bean2 is [name is Lisi，age is 19]
false
```
当isSingleton()被我们实现返回false后，多次从spring容器中获取当前bean对象，每获取一次就会执行一次getObject()方法，而且多次获取的bean对象是两个完全不同的对象。   
说明此时的bean对象是多例的，每从spring容器中获取一次就重新创建一次。   

# 3. 总结
spring容器提供了4种创建bean对象的方式，除了构造方法的方式外，其他3种方式都可以让我们手动去控制bean对象的创建。   