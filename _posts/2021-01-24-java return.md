---
layout:     post
title:      Java return关键字
subtitle:   return用于结束当前线程的调用，返回到被调用处
categories: [Java基础知识]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 返回调用方需要的值
return总是和方法之间的调用密切相关，对程序而言，方法之间的调用实际上是一种消息通信。  
当调用方的方法需要被调用方返回一个指定的类型值时，那么被调用方必须明确指定返回值类型，且必须在每一个case里都使用return返回指定的类型值。  

# 2. 结束对当前被调用方的方法调用，直接返回到调用方处
当调用方调用被调用方的方法时，被调用方可以选择在任何部位结束调用方对当前方法的调用，直接返回到被调用方处，即return的作用就是结束方法的调用，让流程直接离开当前方法，返回到调用方处。  
