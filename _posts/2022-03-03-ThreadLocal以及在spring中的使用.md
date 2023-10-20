---
layout:     post
title:      什么是ThreadLocal？
subtitle:   spring中是如何使用ThreadLocal的？其他框架有使用到吗？
date:       2022-03-03
author:     zhaoeh
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - spring
---

# 1. 使用ThreadLocal的源码参考
spring源码中如下类的设计使用了ThreadLocal：
```java
org.springframework.web.context.request.RequestContextHolder
org.springframework.transaction.support.TransactionSynchronizationManager
org.springframework.context.i18n.LocaleContextHolder
```
pageHelper源码中设计如下：
```java
com.github.pagehelper.page.PageMethod
```
如果自己要尝试设计ThreadLocal，可以参考如上代码的设计。