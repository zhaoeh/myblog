---
layout:     post
title:      springMVC源码
subtitle:   简单分析下springmvc执行源码
categories: [spring MVC专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. spring mvc核心访问流程
springmvc随着springboot启动后，即可立即对外部暴露restful api，接收消费端的http请求，那么一个请求从客户端发起，到restful接收，整个过程是什么样子的呢？   

Spring MVC 是基于 Servlet 的轻量级 Web 框架，整体请求处理流程由核心组件和阶段组成。从接收到 HTTP 请求到生成 HTTP 响应，它经过了多个处理阶段。以下从整体处理流程入手，逐步解析涉及的核心组件及其源码，包括 **拦截器、全局异常处理、`ResponseBodyAdvice`、`ReturnValueHandler`** 等关键点。

---

## **Spring MVC 接收请求的整体处理流程**

1. **请求进入：`DispatcherServlet`**
   - **核心入口：`DispatcherServlet#doDispatch`**
   - 这是 Spring MVC 的中央控制器，它拦截所有符合配置规则的 HTTP 请求，开始整个处理流程。

2. **Handler 映射：`HandlerMapping`**
   - 根据请求路径和方法，找到对应的 `Handler` 和 `HandlerInterceptor`。
   - 返回一个包含 `Handler` 和拦截器链的 `HandlerExecutionChain`。

3. **拦截器链处理：`HandlerExecutionChain`**
   - 执行所有 `preHandle` 方法，决定是否继续后续处理。
   - **核心类：`HandlerExecutionChain`**

4. **Handler 执行：`HandlerAdapter`**
   - 执行请求对应的处理器（通常是控制器方法），并返回处理结果。

5. **视图解析或响应生成**
   - 如果返回值需要直接写入响应（如 `@ResponseBody` 注解），通过 `HttpMessageConverter` 将数据转换为 HTTP 响应。
   - 如果返回视图名，通过 `ViewResolver` 渲染视图。

6. **全局异常处理：`HandlerExceptionResolver`**
   - 在处理过程中如果抛出异常，会交给全局异常处理器处理。

7. **拦截器链清理：`afterCompletion`**
   - 执行所有拦截器的 `afterCompletion` 方法，进行资源清理等操作。

# 2. 源码解读
**整体流程源码解析**   

## 2.1 请求进入 DispatcherServlet

- **入口方法：`doDispatch`**
  - 这是整个 Spring MVC 流程的核心调度方法，接收和处理请求。

  ```java
  protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
      HttpServletRequest processedRequest = request;
      HandlerExecutionChain mappedHandler = null;
      ModelAndView mv = null;

      try {
          // Step 1: 获取 Handler 和拦截器链
          mappedHandler = getHandler(processedRequest);

          // Step 2: 应用拦截器链的 preHandle 方法
          if (!mappedHandler.applyPreHandle(processedRequest, response)) {
              return;
          }

          // Step 3: 调用处理器进行处理
          HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
          mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

          // Step 4: 渲染视图或直接写入响应
          processDispatchResult(processedRequest, response, mappedHandler, mv, null);
      } catch (Exception ex) {
          processHandlerException(processedRequest, response, mappedHandler, ex);
      } finally {
          // Step 5: 执行 afterCompletion
          if (mappedHandler != null) {
              mappedHandler.triggerAfterCompletion(processedRequest, response, null);
          }
      }
  }
  ```

---

## 2.2 Handler 映射阶段

- **定位处理器：`HandlerMapping`**
  - `DispatcherServlet` 调用 `getHandler` 方法，根据请求 URL 和 HTTP 方法找到对应的处理器。
  - **源码位置：`AbstractHandlerMapping#getHandler`**

  ```java
  protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
      for (HandlerMapping hm : this.handlerMappings) {
          HandlerExecutionChain handler = hm.getHandler(request);
          if (handler != null) {
              return handler;
          }
      }
      return null;
  }
  ```

- **结果：`HandlerExecutionChain`**
  - 包含一个 `Handler`（如控制器方法）和拦截器链（`HandlerInterceptor`）。

---

## 2.3 拦截器链处理阶段

- **拦截器处理逻辑：`HandlerExecutionChain`**
  - 每个请求经过拦截器链的多个阶段：
    1. `preHandle`：处理器方法执行前调用，决定是否继续。
    2. `postHandle`：处理器方法执行后调用（视图渲染前）。
    3. `afterCompletion`：请求完成后调用。

  ```java
  public boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
      for (HandlerInterceptor interceptor : this.interceptorList) {
          if (!interceptor.preHandle(request, response, this.handler)) {
              return false;
          }
      }
      return true;
  }
  ```

---

## 2.4 Handler 执行阶段

- **适配处理器：`HandlerAdapter`**
  - `RequestMappingHandlerAdapter` 是最常用的适配器，用于调用控制器方法。
  - **方法：`handleInternal`**
    - 将请求参数解析为控制器方法的入参。
    - 执行控制器方法，获取返回值。

---

## 2.5 返回值处理阶段

- **处理器：`HandlerMethodReturnValueHandler`**
  - 根据返回值类型，找到对应的处理器进行处理。
  - **核心类：`HandlerMethodReturnValueHandlerComposite`**
    - 支持处理多种返回值类型：
      - `ModelAndView`：视图名和模型。
      - `ResponseBody`：直接写入 HTTP 响应。

  ```java
  public void handleReturnValue(Object returnValue, MethodParameter returnType,
                                ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {
      for (HandlerMethodReturnValueHandler handler : this.returnValueHandlers) {
          if (handler.supportsReturnType(returnType)) {
              handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);
              return;
          }
      }
  }
  ```

---

## 2.6 全局异常处理

- **异常解析器：`HandlerExceptionResolver`**
  - 捕获 `DispatcherServlet` 中抛出的异常。
  - **常见实现：**
    - `DefaultHandlerExceptionResolver`：处理标准 Spring 异常。
    - `ResponseStatusExceptionResolver`：解析 `@ResponseStatus` 注解的异常。
    - `ExceptionHandlerExceptionResolver`：解析 `@ExceptionHandler` 注解的方法。

---

## 2.7 ResponseBodyAdvice

- **响应增强器：`ResponseBodyAdvice`**
  - 在响应体写入之前，对数据进行增强处理。
  - 实现类被缓存和查找，并在 `HttpMessageConverter` 写入响应时被调用。

---

## 2.8 视图渲染阶段

- **视图解析：`ViewResolver`**
  - 如果返回的是视图名，通过 `ViewResolver` 查找对应的视图并进行渲染。
  - **常见实现：**
    - `InternalResourceViewResolver`
    - `ThymeleafViewResolver`

---

## **总结**

Spring MVC 的处理流程分为以下关键阶段：

1. **入口**：请求通过 `DispatcherServlet` 进入。
2. **映射**：使用 `HandlerMapping` 找到对应的 `Handler`。
3. **拦截器**：`HandlerExecutionChain` 执行拦截器链。
4. **执行处理器**：使用 `HandlerAdapter` 调用控制器方法。
5. **返回值处理**：`HandlerMethodReturnValueHandler` 处理返回值。
6. **响应增强**：使用 `ResponseBodyAdvice` 对响应数据进行增强。
7. **异常处理**：`HandlerExceptionResolver` 捕获和处理异常。
8. **视图渲染**：通过 `ViewResolver` 渲染视图或直接写入响应。

这些阶段通过 `DispatcherServlet` 调度，层次清晰且高度可扩展。
