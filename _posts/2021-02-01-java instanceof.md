---
layout:     post
title:      java instanceof关键字
subtitle:   java的instanceof关键字用来判断类型所属
date:       2021-02-01
author:     zhaoeh
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - Java初级进阶
---

# 1. instanceof详解
## 1.1 什么是instanceof？
1.  instanceof是java中的关键字，因此不能使用instanceof去命名常量和变量。  
2.  instanceof严格说是java中的一个双目运算符，用来判断一个对象是否是另外一个类的实例。  
3.  instanceof的作用就是判断左边的对象是否是右边的类型的一个实例，如果是就返回true，否则返回false。  
4.  用法为：  
    ```
    boolean result = obj instanceof TargetClass;
    ```
    左边obj是一个实例化对象，TargetClass表示一个类或者接口。  
    当 obj 为 TargetClass 的对象，或者是其直接或间接子类的对象，或者是其接口的实现类的对象，结果result 都返回 true，否则返回false。  
5.  编译器会检查instanceof左边的实例对象是否能够强制转换为右边TargetClass类型（大多数都会向上转型，偶尔会发生向下转型），如果不能强制转换则报错；如果不能确定obj的类型，则可以通过编译，具体运行时再确定结果。  
6.  说白了，instanceof就是用来判断“我能不能强制转换成你的类型”。  

## 1.2 左边的obj对象必须是引用类型
instanceof关键字只能用作引用对象的判断，当obj为基本类型时，编译报错。  
```java
int i = 0;

//编译不通过
System.out.println(i instanceof Integer);
//编译不通过
System.out.println(i instanceof Object);
```
## 1.3 obj为null值，直接返回false
```java
// 返回false，null是一种特殊的值，不属于任何类型。
null instanceof Object; 
```

## 1.4 obj为目标类的实例
当obj为右边目标类的实例化对象时，直接返回true。  
```java
Person per = new Person();

// 返回true
per instanceof Person; 
```

## 1.5 obj为目标类的子类实例或者是目标接口的实现
当obj为右边目标类的子类实例化对象，或者为右边目标接口的实现类的实例化对象时，直接返回true。  
```java
// 子类对象直接向上转型
IPerson person = new Student();
// 返回true
person instanceof IPerson; 

// 子类对象向上转型后再向下转型
Student student = (Student)person;
// 返回true
person instanceof IPerson; 
```

## 1.6 obj为目标类父类实例
当obj为右边目标类的父类实例化对象，则需要分情况。  
说白了就是如果左边的父类实例能够强转成右边子类，则返回true；否则返回false。  
```java
// 发生了对应的向上转型
MyInterface myInterface = new ClassA();

// 则此时 myInterface 对象可以强转成继承链中的任意一个子类型，所以下面返回true
System.out.println(myInterface instanceof ClassA);

// 因为 ClassD 和 MyInterface 的继承链中没有发生过对应的向上转型;
// 因此，myInterface对象不能强转成ClassD，所以下面返回false
System.out.println(myInterface instanceof ClassD);
```

## 1.7 存在的问题
前面我们说过编译器会检查 obj 是否能转换成右边的class类型，如果不能转换则直接报错，如果不能确定类型，则通过编译，具体看运行时定。  

看如下几个例子：
```java
Person p1 = new Person();

//编译报错
System.out.println(p1 instanceof String);
//false
System.out.println(p1 instanceof List);
//false
System.out.println(p1 instanceof List<?>);
//编译报错
System.out.println(p1 instanceof List<Person>);
```
按照我们上面的说法，这里就存在问题了，Person 的对象 p1 很明显不能转换为 String 对象，那么自然 Person 的对象 p1 instanceof String 不能通过编译。  
但为什么 p1 instanceof List 却能通过编译呢？而 instanceof List<Person> 又不能通过编译了？  
<b>深究:</b>  
如果用伪代码描述：  
```java
boolean result;
if (obj == null) {
  result = false;
} else {
  try {
      T temp = (T) obj; // checkcast
      result = true;
  } catch (ClassCastException e) {
      result = false;
  }
}
```
也就是说有表达式 obj instanceof T，instanceof 运算符的 obj 操作数的类型必须是引用类型或者null值; 否则，会发生编译时错误。  
如果 obj 转换为 T 时发生编译错误，则关系表达式的 instanceof 同样会产生编译时错误。  
在运行时，如果 T 的值不为null，并且 obj 可以转换为 T 而不引发ClassCastException，则instanceof运算符的结果为true。  
简单来说就是：如果 obj 不为 null 并且 (T) obj 不抛 ClassCastException 异常则该表达式值为 true ，否则值为 false 。  
所以对于上面提出的问题就很好理解了，为什么 p1 instanceof String 编译报错，因为(String)p1 是不能通过编译的，而 (List)p1 可以通过编译。  

# 2. instanceof底层原理  
instanceof底层原理使用的就是转型的原理。左边对象能够强制转型为右边的类型，则instanceof就返回true，否则返回false。  