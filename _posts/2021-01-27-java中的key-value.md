---
layout:     post
title:      java中常见的key-value
subtitle:   java中经常会遇到一些key-value
date:       2021-01-27
author:     zhaoeh
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Java基础知识
---

# 1. 常见的key-value格式
1.  Java中的Map(Object o1,Object o2);  
2.  Java中的Properties类(String s1,String s2);  
3.  JavaScript(json)中的对象(String s,Object o);  
4.  jsp中的四大属性(String s,Object o);  
5.  jsp中的Cookie(String s,String s2);  
6.  Java中的session,setAttribute(String s,Object o);  

# 2. 注意点
Java中的cookie实际上内部并不是map结构，cookie本身就是一个简单的实体对象，其中有name,value,maxAge,domain等字段。    
一个cookie对象只能保存平行的一对或者几个属性而已，而不是保存一堆属性。  
cookie就类似一个Person，而不是类似map；一个cookie对象最常见的是保存一对name-value的数据，而不能保存多对。    
要想向cookie中保存多对name-value数据，就需要实例化多个cookie对象，分别向每个cookie对象中各保存一对name-value数据。    
