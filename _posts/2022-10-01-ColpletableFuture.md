---
layout:     post
title:      spring自动注入依赖
subtitle:   和手动注入依赖相比，自动注入依赖显得方便多了
categories: [java高并发]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 背景了解
## 1.1 什么是CompletableFuture？
CompletableFuture是java8中新增的一个类，算是对Future的一种增强，用起来很方便，也是会经常用到的一个工具类，熟悉一下。

## 1.2 CompletionStage接口
- CompletionStage代表异步计算过程中的某一个阶段，一个阶段完成以后可能会触发另外一个阶段
- 一个阶段的计算执行可以是一个Function，Consumer或者Runnable。比如：stage.thenApply(x -> square(x)).thenAccept(x -> System.out.print(x)).thenRun(() -> System.out.println())
- 一个阶段的执行可能是被单个阶段的完成触发，也可能是由多个阶段一起触发
