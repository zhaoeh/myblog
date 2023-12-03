---
layout:     post
title:      springboot加载配置项的源码解析以及加载顺序和配置项的优先级
subtitle:   从源码角度分析springboot加载配置项的顺序和优先级
categories: [springboot专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. springboot加载外部配置项的优先级
上一篇文章都是在介绍springboot如何加载外部配置项，即如何从外部配置文件中读取指定的配置项。       