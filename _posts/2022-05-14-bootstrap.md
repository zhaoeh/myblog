---
layout:     post
title:      bootstrap.yml
subtitle:   spring cloud解析bootstrap.yml源码
categories: [springboot专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 解析bootstrap.yml源码分析
Spring Cloud 的 `bootstrap.yml` 是在应用的启动阶段**非常早**被加载的。它的加载机制与 Spring Boot 的配置文件加载流程紧密相关，但 `bootstrap.yml` 是在应用主配置文件 (`application.yml` 或 `application.properties`) 之前加载的，用于配置应用上下文初始化所需的环境属性，如**外部配置中心**（如 Spring Cloud Config Server）的地址和认证信息。

以下是 `bootstrap.yml` 的加载流程及源码分析：

---

### **1. 核心概念与入口**
- Spring Cloud 的 `bootstrap.yml` 加载是通过 **`BootstrapApplicationListener`** 实现的，这是 Spring Cloud 定义的一个 `ApplicationListener`。
- 入口方法为 `BootstrapApplicationListener` 的 `onApplicationEvent` 方法。

#### 关键类
1. **`BootstrapApplicationListener`**
   - 它负责创建一个 `BootstrapContext`，加载 `bootstrap.yml` 配置并将其加入到 `Environment` 中。
   
2. **`ConfigFileApplicationListener`**
   - 它是 Spring Boot 负责加载主配置文件的监听器（加载 `application.yml` 和 `application.properties`），`bootstrap.yml` 的加载优先于它。

---

### **2. 加载流程详细解析**

#### 1. **`BootstrapApplicationListener` 的注册**
Spring Cloud 注册 `BootstrapApplicationListener` 是通过 `META-INF/spring.factories` 文件实现的，代码如下：

```properties
org.springframework.context.ApplicationListener=\
org.springframework.cloud.bootstrap.BootstrapApplicationListener
```

Spring Boot 启动时，会扫描 `META-INF/spring.factories` 文件并加载这个 `ApplicationListener`。

---

#### 2. **`onApplicationEvent` 方法解析**

`BootstrapApplicationListener` 的 `onApplicationEvent` 方法负责解析 `bootstrap.yml` 的核心逻辑，关键源码如下：

```java
@Override
public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
    ConfigurableEnvironment environment = event.getEnvironment();
    if (!environment.getProperty("spring.cloud.bootstrap.enabled", Boolean.class, true)) {
        return;
    }

    // 创建 Bootstrap Context
    ConfigurableApplicationContext bootstrapContext = bootstrapServiceContext(environment);

    // 将 Bootstrap Environment 合并到主环境
    mergeBootstrapProperties(environment, bootstrapContext);
}
```

流程解析：
1. **检查是否启用**  
   通过 `spring.cloud.bootstrap.enabled` 属性判断是否启用 `bootstrap.yml` 加载。
   
2. **创建 Bootstrap Context**  
   调用 `bootstrapServiceContext` 方法，加载 `bootstrap.yml` 中的配置属性并创建一个独立的 `ApplicationContext`，该上下文专用于解析 `bootstrap.yml`。

   ```java
   private ConfigurableApplicationContext bootstrapServiceContext(ConfigurableEnvironment environment) {
       // 初始化 Spring ApplicationContext，用于加载 bootstrap.yml
       StandardEnvironment bootstrapEnvironment = new StandardEnvironment();
       MutablePropertySources bootstrapProperties = bootstrapEnvironment.getPropertySources();
       bootstrapProperties.addFirst(new MapPropertySource("bootstrap", Collections.emptyMap()));

       SpringApplicationBuilder builder = new SpringApplicationBuilder()
           .environment(bootstrapEnvironment)
           .sources(BootstrapImportSelectorConfiguration.class);

       return builder.run();
   }
   ```

3. **合并属性到主环境**  
   加载完 `bootstrap.yml` 后，其属性会通过 `mergeBootstrapProperties` 合并到主 `Environment` 中。

---

#### 3. **配置文件加载顺序**

`bootstrap.yml` 的加载顺序早于 `application.yml`，这一点体现在 Spring Boot 的 `EnvironmentPostProcessor` 执行流程中：

1. `BootstrapApplicationListener` 提前注册并加载 `bootstrap.yml`。
2. 加载完成后，Spring Boot 的 `ConfigFileApplicationListener` 再加载主配置文件 `application.yml`。

---

### **3. 完整流程图**

1. 启动 Spring Boot 应用，触发监听器加载。
2. `BootstrapApplicationListener` 被触发。
3. 检查 `spring.cloud.bootstrap.enabled` 配置。
4. 加载 `bootstrap.yml` 并初始化一个独立的 `ApplicationContext`。
5. 将加载的属性合并到主 `Environment`。
6. 后续 `application.yml` 或 `application.properties` 加载覆盖（或补充）部分属性。

---

### **4. 注意点**

- `bootstrap.yml` 的典型用途：
  1. 配置外部配置中心（如 Spring Cloud Config）。
  2. 配置加密解密所需的信息。
  3. 配置应用初始化必须的系统级属性。

- 加载顺序控制：  
  `bootstrap.yml` 的属性优先级低于 `application.yml`，因此后者可以覆盖前者。

---

### **源码路径**
- **`BootstrapApplicationListener`**: [Spring Cloud Context](https://github.com/spring-cloud/spring-cloud-context)
- **`ConfigFileApplicationListener`**: [Spring Boot](https://github.com/spring-projects/spring-boot) 

若有更多需求，可进一步深入！

# 2. 触发回调流程分析
### **`BootstrapApplicationListener` 被触发回调的流程详解**

`BootstrapApplicationListener` 的触发和回调是 Spring Boot 的事件机制驱动的，具体涉及 **Spring 应用启动阶段的事件发布与监听机制**。以下是完整的详细解析：

---

### **1. Spring Boot 启动流程中的事件机制**

Spring Boot 在应用启动过程中会触发多个生命周期事件，其中一个关键事件是 **`ApplicationEnvironmentPreparedEvent`**。  
`BootstrapApplicationListener` 正是通过监听该事件来完成 `bootstrap.yml` 的加载和属性注入。

#### **启动事件触发顺序**
1. **`ApplicationStartingEvent`**: 应用启动的最早阶段，未初始化 `Environment` 和 `ApplicationContext`。
2. **`ApplicationEnvironmentPreparedEvent`**: `Environment` 准备完成后发布。
3. **`ApplicationContextInitializedEvent`**: `ApplicationContext` 初始化完成后发布。
4. **`ApplicationPreparedEvent`**: 上下文已刷新但尚未加载 Bean。
5. **`ApplicationStartedEvent`**: 上下文已完全启动，但尚未完成 CommandLineRunner 或 ApplicationRunner。
6. **`ApplicationReadyEvent`**: 应用启动完成并已准备好提供服务。

`BootstrapApplicationListener` 在 **`ApplicationEnvironmentPreparedEvent`** 阶段回调。

---

### **2. 触发回调的详细流程**

#### 1. **Spring 启动时加载监听器**
Spring Boot 会通过 `SpringApplication` 类加载所有在 `META-INF/spring.factories` 文件中声明的 `ApplicationListener`。  
`BootstrapApplicationListener` 被声明为一个 `ApplicationListener`。

`META-INF/spring.factories` 中的配置：
```properties
org.springframework.context.ApplicationListener=\
org.springframework.cloud.bootstrap.BootstrapApplicationListener
```

Spring Boot 会扫描并注册该类为全局监听器。

---

#### 2. **事件发布到监听器触发回调**

当 Spring Boot 在启动过程中完成 `Environment` 的准备后，会发布 **`ApplicationEnvironmentPreparedEvent`**，触发所有监听该事件的 `ApplicationListener`，包括 `BootstrapApplicationListener`。

`SpringApplication#run` 的关键代码片段：
```java
// 发布 ApplicationEnvironmentPreparedEvent
listeners.environmentPrepared(environment);
```

这里的 `listeners` 包括所有注册的 `ApplicationListener`。

---

#### 3. **`BootstrapApplicationListener#onApplicationEvent` 方法触发**

`ApplicationEnvironmentPreparedEvent` 发布时，会回调 `BootstrapApplicationListener` 的 `onApplicationEvent` 方法，完成如下操作：

```java
@Override
public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
    ConfigurableEnvironment environment = event.getEnvironment();

    // 检查是否启用 bootstrap.yml 加载
    if (!environment.getProperty("spring.cloud.bootstrap.enabled", Boolean.class, true)) {
        return;
    }

    // 创建 Bootstrap 环境上下文
    ConfigurableApplicationContext bootstrapContext = bootstrapServiceContext(environment);

    // 将 Bootstrap 属性合并到主环境
    mergeBootstrapProperties(environment, bootstrapContext);
}
```

---

### **3. 具体执行逻辑解析**

#### **(1) 检查是否启用**
代码：
```java
if (!environment.getProperty("spring.cloud.bootstrap.enabled", Boolean.class, true)) {
    return;
}
```
- 默认情况下，`spring.cloud.bootstrap.enabled` 为 `true`。
- 如果用户显式关闭该功能（设置为 `false`），将跳过 `bootstrap.yml` 加载。

---

#### **(2) 创建 Bootstrap 上下文**
代码：
```java
ConfigurableApplicationContext bootstrapContext = bootstrapServiceContext(environment);
```

`bootstrapServiceContext` 方法：
```java
private ConfigurableApplicationContext bootstrapServiceContext(ConfigurableEnvironment environment) {
    // 初始化新的 Bootstrap 环境
    StandardEnvironment bootstrapEnvironment = new StandardEnvironment();

    // 添加空的 PropertySource（将被填充为 bootstrap.yml 的内容）
    MutablePropertySources bootstrapProperties = bootstrapEnvironment.getPropertySources();
    bootstrapProperties.addFirst(new MapPropertySource("bootstrap", Collections.emptyMap()));

    // 创建 Spring ApplicationContext
    SpringApplicationBuilder builder = new SpringApplicationBuilder()
        .environment(bootstrapEnvironment)
        .sources(BootstrapImportSelectorConfiguration.class);

    // 启动新的 Bootstrap Context
    return builder.run();
}
```

流程：
1. 创建一个独立的 `StandardEnvironment`，用于解析 `bootstrap.yml`。
2. 将解析到的属性存入 `PropertySource`。
3. 使用 `SpringApplicationBuilder` 构建一个新的 `ApplicationContext` 并加载 `bootstrap.yml`。

---

#### **(3) 合并属性到主环境**
代码：
```java
mergeBootstrapProperties(environment, bootstrapContext);
```

`mergeBootstrapProperties` 方法：
```java
private void mergeBootstrapProperties(ConfigurableEnvironment environment, ConfigurableApplicationContext context) {
    ConfigurableEnvironment bootstrapEnvironment = context.getEnvironment();
    for (PropertySource<?> source : bootstrapEnvironment.getPropertySources()) {
        if (!environment.getPropertySources().contains(source.getName())) {
            environment.getPropertySources().addLast(source);
        }
    }
}
```

流程：
1. 获取 `bootstrap.yml` 加载到的属性。
2. 检查主 `Environment` 是否已有同名属性。
3. 如果没有，将属性添加到主环境的 `PropertySources` 中，确保后续的 `application.yml` 或代码中的配置能够覆盖这些属性。

---

### **4. 触发流程完整时序图**

1. **SpringApplication 初始化**：
   - 加载 `ApplicationListener`（包括 `BootstrapApplicationListener`）。
2. **发布 `ApplicationEnvironmentPreparedEvent`**：
   - `BootstrapApplicationListener` 被触发。
3. **解析 `bootstrap.yml`**：
   - 初始化独立的 `Bootstrap Context`。
   - 通过 `SpringApplicationBuilder` 解析 `bootstrap.yml`。
4. **属性合并**：
   - 将 `bootstrap.yml` 中的属性合并到主 `Environment`。
5. **后续事件和流程**：
   - `ConfigFileApplicationListener` 加载 `application.yml`。
   - Bean 初始化和容器启动完成。

---

### **5. 总结**

`BootstrapApplicationListener` 的回调流程结合了 Spring Boot 的事件机制和独立的 `Bootstrap Context` 创建：
- 它确保在最早阶段加载外部配置源所需的关键属性。
- 合并逻辑保证了 `bootstrap.yml` 的优先级低于 `application.yml`，但高于默认配置。

如果还有疑问或需要具体场景代码解析，可以进一步探讨！
