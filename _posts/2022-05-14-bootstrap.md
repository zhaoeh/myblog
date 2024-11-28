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

# 3. application.yml等文件以及环境变量等属性何时被spring容器解析？
`ConfigFileApplicationListener` 被废弃，改为使用 `ConfigDataApplicationListener` 是因为 Spring Boot 从 2.4.0 开始引入了新的配置数据加载机制，旨在提高配置文件的灵活性和可扩展性。以下是具体原因和改动的背景：

---

### 1. **新配置加载机制的目标**
   - **统一配置加载**：`ConfigDataApplicationListener` 引入了统一的配置加载流程，支持从多种来源加载配置（如文件、环境变量、远程服务器等）。
   - **增强的灵活性**：新机制允许在应用程序上下文刷新之前解析配置，这为配置的优先级、动态性和依赖关系提供了更大的灵活性。
   - **模块化设计**：新机制采用模块化设计，便于扩展，例如可以自定义配置数据源。

---

### 2. **新机制的主要特性**
   - **支持多种配置来源**：不仅支持 `application.properties` 和 `application.yml` 文件，还支持远程配置服务（如 Spring Cloud Config）。
   - **配置文件分层**：支持加载多个配置文件并合并配置值，例如通过 `spring.config.import` 导入文件。
   - **配置导入**：提供 `spring.config.import` 属性来加载嵌套的或外部的配置文件。
   - **新的配置绑定**：提供更好的配置绑定和验证支持。

---

### 3. **为什么废弃 `ConfigFileApplicationListener`**
   - **复杂的配置需求**：`ConfigFileApplicationListener` 主要用于从文件加载配置，但随着 Spring Boot 应用的复杂化，仅支持文件不足以满足需求。
   - **扩展性不足**：旧的机制在扩展其他配置源时需要大量定制，难以灵活地适应动态配置需求。
   - **代码维护性**：新机制通过 `ConfigDataEnvironment` 和 `ConfigDataLoader` 提供了更清晰的 API，简化了代码维护。

---

### 4. **过渡期的兼容性**
   - 如果你的项目从 `ConfigFileApplicationListener` 升级到 `ConfigDataApplicationListener`，通常不需要额外修改代码，只需按照 Spring Boot 的配置指南确保正确设置配置文件即可。
   - 如果需要兼容旧配置，可以通过启用 `spring.config.use-legacy-processing=true` 使用旧的配置文件处理逻辑（仅限过渡期）。

---

### 5. **如何适应新的配置机制**
   - **检查配置文件**：确保使用了新的配置选项，例如 `spring.config.import`。
   - **理解优先级**：新的加载机制引入了更细粒度的优先级控制，需要了解 `spring.config.name` 和 `spring.config.location` 的行为。
   - **测试远程配置**：尝试使用远程配置源验证新机制的灵活性。

---

### 6. **总结**
`ConfigFileApplicationListener` 的废弃是为了适应更加现代化和动态化的配置需求。`ConfigDataApplicationListener` 提供了更强大的功能和更清晰的逻辑，可以更好地支持复杂场景和未来扩展。对于开发者来说，理解和适应新机制能够充分利用 Spring Boot 的强大能力。

# 4. spring cloud如何确保解析bootstrap文件早于解析其他环境属性呢？以及和nacos篡改配置的关系
您的问题非常深入，涉及 Spring Boot 和 Spring Cloud 配置机制的设计哲学和实现细节。下面逐一分析 **优先级控制** 和 **设计取向** 两个核心问题。

---

### **1. `BootstrapConfigFileApplicationListener` 如何优先于 `ConfigDataApplicationListener` 执行**

#### **1.1 优先级控制的关键**
Spring Boot 和 Spring Cloud 通过监听器的注册顺序与执行优先级控制机制来确保 `BootstrapConfigFileApplicationListener` 在 `ConfigDataApplicationListener` 之前执行。

具体机制：
1. **`SpringApplicationRunListeners`**：
   - Spring Boot 会通过 `META-INF/spring.factories` 文件加载 `ApplicationListener`。
   - `BootstrapConfigFileApplicationListener` 是 Spring Cloud 定义的特殊监听器，通过扩展 Spring Boot 的 `ApplicationListener`，并明确其加载优先级。

2. **优先级声明：`@Order` 或 `Ordered` 接口**：
   - `BootstrapConfigFileApplicationListener` 明确声明了较高的优先级（较低的数值表示更高的优先级）。
   - `ConfigDataApplicationListener` 的优先级较低，通常在 Spring Boot 内部排序机制中位于后面。

3. **`spring.factories` 的注册顺序**：
   - Spring Cloud 在 `spring.factories` 中注册了 `BootstrapConfigFileApplicationListener`，它会优先加载并执行。
   - 配置文件和监听器的执行顺序基于 `SpringApplicationRunListeners` 的加载策略。

#### **1.2 为什么需要优先于 `ConfigDataApplicationListener`**
- `BootstrapConfigFileApplicationListener` 是 Spring Cloud 的核心，主要用于加载 `bootstrap.yml` 和 `bootstrap.properties`，这些配置通常包含应用程序初始化的关键信息（如注册中心地址、加密信息等）。
- **配置时机的关键性**： 
  - 如果不在 `ConfigDataApplicationListener` 之前执行，就无法确保 `Environment` 中的必要配置被优先加载，可能导致应用启动失败。

---

### **2. 为什么 Nacos 选择 `ApplicationContextInitializer` 改变环境？**

#### **2.1 两种方式的比较**
- **通过 `EnvironmentPostProcessor` 篡改 `Environment`**
  - Spring Boot 提供的扩展机制，允许在应用启动时修改 `Environment`。
  - 在 Spring Boot 2.4+ 中，这种机制主要被 `ConfigDataApplicationListener` 取代，用于加载配置数据。
  - 适用于需要调整 `Environment` 或引入配置数据的场景。

- **通过 `ApplicationContextInitializer` 篡改上下文**
  - Spring 容器的扩展机制，允许在 `ApplicationContext` 刷新之前做初始化操作。
  - 可以更早地控制上下文初始化过程，例如注册 BeanFactory 后置处理器、调整上下文配置等。
  - 适用于需要更深层次调整的场景，例如定制化的上下文刷新逻辑。

#### **2.2 为什么 Nacos 选择 `ApplicationContextInitializer`**
1. **启动时机的考虑**：
   - Nacos 的部分配置依赖于 `ApplicationContext` 初始化的更早阶段，而不是单纯的 `Environment` 配置。
   - `ApplicationContextInitializer` 提供的扩展点更靠近 Spring 容器的核心，可以在配置加载之前做出必要的调整。

2. **动态注册能力**：
   - Nacos 可能需要动态注入一些与配置管理相关的组件，例如配置监听器、动态属性更新支持。
   - 通过 `ApplicationContextInitializer`，可以直接与 `ApplicationContext` 交互，方便实现这些功能。

3. **与 Spring Cloud 的契合**：
   - Spring Cloud 和 Nacos 的设计目标是为微服务提供配置中心服务，`ApplicationContextInitializer` 更符合这种集成需求，便于扩展注册中心、动态刷新等功能。

4. **与 Spring Boot 加载机制的兼容性**：
   - Spring Boot 的 `ConfigDataApplicationListener` 已经接管了许多配置加载的职责。通过 `ApplicationContextInitializer`，可以避免与 `ConfigDataApplicationListener` 的加载流程冲突。

#### **2.3 设计哲学上的差异**
- **`EnvironmentPostProcessor` 的哲学**：
  - 偏向于 *数据驱动* 的设计，主要专注于配置加载的扩展。
  - 面向简单的配置调整需求。
- **`ApplicationContextInitializer` 的哲学**：
  - 偏向于 *上下文控制* 的设计，更适合复杂的场景。
  - 面向动态注册组件、深度集成第三方框架等高级需求。

---

### **3. 总结**

#### **优先级的设计意图**
- `BootstrapConfigFileApplicationListener` 的高优先级确保 Spring Cloud 的核心配置可以优先加载，满足微服务框架的需求。
- `ConfigDataApplicationListener` 则专注于 Spring Boot 2.4+ 的配置机制，用于整合文件和远程配置源。

#### **设计哲学**
- Spring Boot 的设计倾向模块化、职责分离，`EnvironmentPostProcessor` 是基于配置加载的职责延伸。
- Spring Cloud 和 Nacos 偏向于灵活性和深度控制，`ApplicationContextInitializer` 提供了更大的操作自由度，适合高度定制化的需求。

两种机制在不同场景中各有优势，而 Nacos 选择 `ApplicationContextInitializer` 的背后，是为了实现更强的控制能力和集成深度。
