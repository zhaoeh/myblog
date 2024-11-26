---
layout:     post
title:      自动配置最佳实践-CacheManager
subtitle:   spring缓存管理器详解
categories: [springboot专题]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 什么是CacheManager?
在 Spring 框架中，`CacheManager` 是缓存管理的核心接口，负责管理和提供缓存的抽象，方便开发者通过统一的接口操作不同的缓存实现，如本地Local缓存、caffeine缓存、Redis 缓存、EhCache 等。

### `CacheManager` 的职责

1. **统一缓存操作**：
   - `CacheManager` 负责管理所有缓存的实例，通过 `CacheManager` 可以获取并操作不同的缓存，而不需要直接操作底层的缓存实现（例如 Redis 或本地缓存）。
   - 它提供了一个统一的方式来处理缓存数据，屏蔽底层缓存库的差异，开发者只需关注如何使用缓存，不必关心具体的缓存实现。

2. **缓存管理**：
   - 它负责管理应用中的所有缓存对象，通常以 `Cache` 的形式存在。每个 `Cache` 实例代表一个缓存区域，`CacheManager` 负责返回具体的缓存区域来进行读写操作。

3. **缓存创建**：
   - `CacheManager` 实例会在应用启动时，基于配置自动创建所需的缓存实例。你可以通过配置来控制缓存的行为，如缓存超时、最大容量等。
  
4. **Manager实例**：
   - 一个`CacheManager` 实例只能管理一种类型的多个Cache实例，换言之，如果容器想要支持多种不同类型的缓存，则必修针对每种类型创建对应的CacheManager实例。比如容器需要同时使用redis、ehcache、caffenie这三种类型作为缓存，则针对redis创建一个CacheManager对象，专门管理多个redis实例；针对ehcache创建一个CacheManager对象，专门管理多个ehcache实例；针对caffeine创建一个CacheManager实例，专门管理多个caffeine实例。    

### `CacheManager` 的使用方式

在 Spring 项目中，通常通过注解来使用缓存，而 `CacheManager` 是与这些注解协同工作的：

1. **注解方式**：
   - Spring 提供了一系列缓存相关的注解（例如 `@Cacheable`、`@CachePut`、`@CacheEvict`），这些注解依赖 `CacheManager` 提供缓存管理。
   - 例如，使用 `@Cacheable` 注解可以将某个方法的返回结果缓存起来，供下次调用时直接返回而不再执行该方法。

     ```java
     @Service
     public class MyService {

         @Cacheable("myCache")
         public String fetchData(String key) {
             // 数据库操作或耗时操作
             return "data from db";
         }
     }
     ```

   - 在这个例子中，`@Cacheable("myCache")` 告诉 Spring 使用 `CacheManager` 中管理的 `myCache` 缓存区域来存储和获取方法的返回值。`CacheManager` 会在缓存中查找是否已有缓存的结果，如果没有，就执行方法并将结果存储在缓存中。

2. **配置 `CacheManager`**：
   - `CacheManager` 的具体实现通常是通过自动配置来完成的。Spring Boot 提供了多种缓存实现，如 Redis、EhCache、Caffeine 等，通过配置即可决定使用哪种缓存。
   
   - 例如，如果使用 Redis 作为缓存系统，Spring Boot 自动配置会创建 `RedisCacheManager` 并注册到 Spring 容器中：
     ```yaml
     spring:
       cache:
         type: redis
     ```
   
   - Spring Boot 会自动根据 `spring.cache.type` 的配置，决定使用哪种 `CacheManager`（如 `RedisCacheManager`、`SimpleCacheManager` 等）。开发者通常不需要手动定义 `CacheManager`。

3. **自定义 `CacheManager`**：
   - 如果你有特殊需求，可以自定义 `CacheManager` 实例。通过 `@Bean` 定义自己的 `CacheManager`：
   
     ```java
     @Configuration
     public class CacheConfig {

         @Bean
         public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
             RedisCacheConfiguration cacheConfig = RedisCacheConfiguration.defaultCacheConfig()
                     .entryTtl(Duration.ofMinutes(10)); // 设置缓存的过期时间
             return RedisCacheManager.builder(redisConnectionFactory)
                                     .cacheDefaults(cacheConfig)
                                     .build();
         }
     }
     ```

   - 这个例子展示了如何自定义 `RedisCacheManager`，并设置缓存条目的过期时间。

### `redisTemplate` 和 `CacheManager` 的关系

- **`redisTemplate`**：`RedisTemplate` 是 Redis 的模板类，直接用于操作 Redis 数据结构，比如字符串、哈希、列表、集合等。它允许你手动操作 Redis 数据，而不涉及缓存抽象。开发者可以通过 `redisTemplate` 来执行 Redis 命令，操作缓存数据。

- **`CacheManager`**：`CacheManager` 提供了更高层的缓存抽象，允许开发者通过注解或缓存接口来统一管理缓存操作，而不关心底层缓存的实现。在底层，`CacheManager` 也会使用 `redisTemplate` 或其他类似工具来实际操作 Redis，但开发者通常不直接使用 `redisTemplate` 操作缓存，而是通过 `CacheManager` 的接口。

### `CacheManager` 和 `redisTemplate`的关系

- `CacheManager` 是更高层次的抽象，统一管理环境中的各类缓存，它是抽象的，不关注缓存到底是通过什么方式实现，只提供抽象管理功能。`redisTemplate`只是spring对直接操作redis做的一层包装，它只针对操作redis，更原始，更接近具体的缓存底层。    
  
- **直接使用 `redisTemplate`**：如果项目中直接使用 `redisTemplate` 操作 Redis 数据，这通常是为了执行低级别的 Redis 操作，比如操作 Redis 数据结构（字符串、哈希等），这种操作不走 `CacheManager` 的缓存管理流程。而使用 `@Cacheable` 这样的注解，则依赖 `CacheManager` 来管理缓存。

### 总结

- `CacheManager` 是 Spring 提供的缓存抽象管理器，负责管理和协调不同的缓存实例。
- `CacheManager` 允许通过统一接口操作缓存，并可与注解（如 `@Cacheable`）结合使用。
- `redisTemplate` 是直接操作 Redis 的工具，而 `CacheManager` 提供更高层次的缓存抽象，背后可以使用 `redisTemplate` 来实现缓存操作。
- Spring Boot 自动配置会为你注册适当的 `CacheManager` 实现，所以在大部分情况下不需要手动配置 `CacheManager`。

# 2. CacheManager自动配置
***什么时候cacheManager会自动注册呢？***
默认情况下，只需要在启动类或者其他配置类中标注`@EnableCaching`，springboot就会自动的向容器中注册一个SimpleCacheManager实例。    
如果想向容器中指定我们要创建的CacheManager实例，这需要在配置文件中进行配置：
    ```yaml
     spring:
       cache:
         type: redis
     ```
让然，还需要一些其他条件，比如，容器中比如有redis的某些关键类等。   
也就是说，当我们想要指定CacheManager时，需要完成：   
（1）配置类标注`@EnableCaching`，开启缓存功能。   
（2）导入要指定的目标缓存相关的依赖包。   
（3）在配置文件中显式配置 spring.cache.type 属性。   
满足上述三点，将向容器中注册我们指定的CacheManager。    
如果只满足（1），则默认向容器中注册一个SimpleCacheManager。  
***CacheManager自动配置源码后面会分析***

# 3. CacheManager只能和缓存注解一同使用吗？
实际上，`CacheManager` 并不是只能通过 Spring 的缓存注解（如 `@Cacheable`）来操作缓存，它也提供了面向编程的 API，允许你在代码中手动控制缓存的行为。这样，你可以灵活地决定在什么地方缓存数据、如何更新缓存等，不必依赖注解机制。因此，`CacheManager` 并不是局限于 Spring 内部的一套缓存注解，而是可以通过编程方式进行细粒度的缓存控制。

### 如何通过 `CacheManager` 灵活操作缓存

如果你想通过编程方式灵活地操作缓存，而不是使用注解，你可以直接使用 `CacheManager` 和 `Cache` API 来手动缓存数据、读取缓存、删除缓存等。下面是一个基本的用法示例：

```java
@Service
public class MyService {

    private final CacheManager cacheManager;

    @Autowired
    public MyService(CacheManager cacheManager) {
        this.cacheManager = cacheManager;
    }

    public String getDataWithCache(String key) {
        // 获取指定名称的缓存区域
        Cache cache = cacheManager.getCache("myCache");
        if (cache != null) {
            // 从缓存中获取值
            Cache.ValueWrapper cachedValue = cache.get(key);
            if (cachedValue != null) {
                return (String) cachedValue.get(); // 返回缓存中的数据
            }
        }

        // 缓存中没有值，执行实际的业务逻辑
        String data = fetchDataFromDatabase(key);

        // 将结果手动存入缓存
        if (cache != null) {
            cache.put(key, data);
        }

        return data;
    }

    private String fetchDataFromDatabase(String key) {
        // 模拟数据库查询或其他耗时操作
        return "data from db for key: " + key;
    }
}
```

在上面的例子中，你可以通过 `CacheManager.getCache("myCache")` 获取特定的缓存区域，然后根据业务逻辑手动决定是否缓存数据，而不是通过注解自动处理缓存。

### 主要缓存操作 API

- **`CacheManager.getCache(String name)`**：获取指定名称的缓存区域（`Cache` 对象）。
- **`Cache.get(Object key)`**：从缓存中读取指定 key 的数据。
- **`Cache.put(Object key, Object value)`**：将指定 key 和 value 存入缓存。
- **`Cache.evict(Object key)`**：删除缓存中的指定 key 数据。
- **`Cache.clear()`**：清空整个缓存区域。

### 使用 `CacheManager` 进行灵活的缓存控制

你可以根据需要灵活控制缓存，而不必受限于方法执行完后才缓存结果。通过编程方式操作缓存，你可以在任何地方进行缓存操作，这比注解提供了更多的灵活性。

例如：

- 在一个方法的中间阶段缓存某个计算结果。
- 在某些条件下决定是否更新缓存或删除缓存中的某个条目。
- 手动清理缓存中的数据。

### 示例：灵活的缓存控制

假设你希望在特定条件下更新缓存，而不是在方法结束时自动缓存，可以手动操作缓存：

```java
public void updateCacheConditionally(String key, String newData) {
    Cache cache = cacheManager.getCache("myCache");

    if (cache != null && someConditionMet()) {
        // 只有在满足某些条件时才更新缓存
        cache.put(key, newData);
    }
}
```

### 缓存失效策略

你还可以手动控制缓存的失效策略，比如在某些情况下主动删除缓存数据，而不依赖于 `@CacheEvict` 注解：

```java
public void invalidateCache(String key) {
    Cache cache = cacheManager.getCache("myCache");

    if (cache != null) {
        // 手动删除缓存中的指定 key
        cache.evict(key);
    }
}
```

### 结论

- `CacheManager` 不仅仅是通过注解控制缓存的工具，它也提供了灵活的编程接口，允许你在代码中手动操作缓存。
- 通过 `CacheManager`，你可以手动决定缓存的位置、时机和策略，提供了更大的灵活性，而不是仅依赖注解自动缓存结果。
- 注解（如 `@Cacheable`）是一种便捷的缓存操作方式，而 `CacheManager` 提供了更细粒度的控制，适合需要精细控制缓存行为的场景。

# 4. CacheManager推出的目的

1. **`CacheManager` 的作用**：Spring 通过 `CacheManager` 对各类缓存（如 Redis、Ehcache 等）的操作进行了抽象。这样，无论底层使用的缓存实现是什么，Spring 统一提供了 `CacheManager` 和 `Cache` 接口，使得开发者不需要直接处理 Redis、Ehcache 等缓存实现的具体细节，简化了操作流程。

2. **抽象与实现屏蔽**：通过 `CacheManager` 和 `Cache`，Spring 屏蔽了不同缓存实现的差异，使你能够通过一致的编程接口来操作缓存，而不必了解每种缓存实现的底层细节。这种设计思想解耦了应用和具体缓存实现，使得可以轻松切换不同的缓存提供者。

3. **注解的便捷性**：为了进一步简化缓存操作，Spring 引入了缓存注解（如 `@Cacheable`、`@CachePut`、`@CacheEvict` 等），让开发者只需要在方法上添加注解即可自动完成缓存的读写操作。这是基于 `CacheManager` 的一个更便捷的封装，专门用于处理缓存的常见场景，如方法结果缓存、缓存更新和清除等。

4. **编程式操作的灵活性**：虽然注解提供了很便捷的方式，但它并不适用于所有场景。例如，当你需要对缓存进行复杂的逻辑控制时，编程式的方式更加灵活。通过直接使用 `CacheManager` 和 `Cache` 接口，你可以更细粒度地控制缓存的行为，如动态地决定缓存内容、手动删除缓存条目、缓存中间结果等。

### 总结：
- **`CacheManager`**：统一管理和抽象缓存的访问和操作，屏蔽了不同缓存实现的细节。
- **`Cache` 接口**：提供了缓存的核心操作（如 put、get、evict 等），这些操作通过 `CacheManager` 实例来获取。
- **注解方式**：作为一种便捷的扩展方式，让开发者通过注解轻松实现缓存操作，但背后依然是通过 `CacheManager` 和 `Cache` 来执行缓存操作。
- **编程方式**：提供了比注解更灵活的缓存操作手段，适用于复杂的缓存场景。

总而言之，`CacheManager` 是对缓存的封装，无论你是通过注解还是编程接口进行操作，最终都会经过 `CacheManager` 来实现缓存逻辑，而你不需要关心底层具体的缓存实现。

# 5. 项目中使用了CacheManager，还可以继续使用redisTemplate吗？
虽然 `CacheManager` 提供了统一的缓存抽象，并且通过注解操作缓存变得非常方便，但在实际项目中，许多人仍然倾向于直接使用 `RedisTemplate` 操作 Redis，而不是通过 `CacheManager`。这种现象主要有以下几个原因：

### 1. **操作的灵活性和细粒度控制**
   `RedisTemplate` 提供了直接操作 Redis 的全面 API，使得开发者能够精确控制 Redis 中的操作细节。具体而言，`RedisTemplate` 能直接调用 Redis 的所有功能，包括：
   - 发布订阅 (Pub/Sub)。
   - 管理 Redis 数据结构（如 `Hash`、`List`、`Set` 等）。
   - 精确控制缓存失效策略（如 TTL、过期时间等）。
   - 批量操作、事务控制、管道操作等。

   相比之下，`CacheManager` 主要抽象了对缓存的基本操作（如 `put`、`get`、`evict` 等），提供了较为简单的使用方式，但在某些场景下，无法覆盖 Redis 的全部功能。对于需要精确利用 Redis 强大功能的场景，直接使用 `RedisTemplate` 可以提供更多的灵活性。

### 2. **对 Redis 特性的依赖**
   Redis 作为一个强大的分布式缓存和数据存储解决方案，不仅仅局限于缓存功能。许多项目中，Redis 被用作消息队列、分布式锁、计数器、排行榜等。这些应用场景往往需要使用 Redis 的原生操作，而不仅仅是缓存操作。

   如果项目对 Redis 的这些特性有强依赖，`RedisTemplate` 就成为一个更自然的选择，而不是通过 `CacheManager` 这样较为抽象的缓存框架。

### 3. **复杂的缓存需求**
   在实际项目中，缓存策略往往非常复杂。比如：
   - 缓存的多层级结构（本地缓存 + 分布式缓存）。
   - 精细化的缓存过期策略。
   - 缓存一致性和事务处理。
   
   这些复杂场景可能超出了 `CacheManager` 的能力范围，而直接使用 `RedisTemplate` 可以更好地满足这些需求。

### 4. **性能优化与批量操作**
   `RedisTemplate` 支持 Redis 的批量操作、管道操作和事务控制，这在高性能场景下非常有用。通过 Redis 的管道功能，开发者可以在一次网络请求中批量发送多条命令，大幅提升性能。

   虽然 `CacheManager` 也支持一些基础的批量缓存操作，但对 Redis 的这些高级特性支持较为有限。因此，在高性能场景下，直接操作 `RedisTemplate` 往往是一个更好的选择。

### 5. **传统使用习惯**
   对于某些开发者，直接使用 Redis 的客户端 API（如 `RedisTemplate`）是他们更为熟悉的工作方式。相比于使用 `CacheManager` 的抽象层，直接操作 `RedisTemplate` 让开发者感到更直观、可控，特别是在复杂的业务场景中，他们可能不愿意通过抽象框架来操作缓存。

### 6. **缺少对自定义缓存逻辑的支持**
   `CacheManager` 和注解方式更适合较为简单的缓存需求（比如方法结果缓存），而如果需要在特定逻辑中实现复杂的缓存更新、缓存清理逻辑，或者需要在代码中的某个特定点手动进行缓存操作，`CacheManager` 的注解机制就显得有些受限。

   在这种情况下，`RedisTemplate` 提供的编程式 API 更加灵活，可以完全由开发者根据业务逻辑来决定何时缓存、缓存什么数据、如何更新缓存等。

### 7. **项目早期设计决定**
   一些项目在早期设计时，可能没有使用 Spring 的 `CacheManager` 机制，而是直接使用了 `RedisTemplate` 进行缓存管理。由于项目的技术栈和团队习惯，一旦这种操作模式固定下来，后续的开发者也会继续沿用这种方式。

### 总结
尽管 `CacheManager` 提供了非常方便的缓存抽象和注解方式，但许多实际项目中还是倾向于直接使用 `RedisTemplate`，原因包括：
- 对 Redis 复杂特性和细粒度操作的需求。
- 更高的性能优化需求（如批量操作、管道操作）。
- 复杂的缓存策略和对缓存一致性的精确控制。
- 开发者的习惯和项目早期设计的延续。

所以，**`CacheManager` 更适合简单的、标准化的缓存场景，而 `RedisTemplate` 则适合需要灵活、复杂缓存控制的场景**。

# 6. spring.cache配置项详细解释
`spring.cache` 是 Spring Boot 提供的统一缓存抽象的配置项，允许我们轻松地配置和管理应用中的缓存系统。它支持多种缓存提供者（如 Ehcache、Caffeine、Redis 等），并且通过简单的配置就可以启用和定制缓存行为。

下面是 `spring.cache` 相关的配置项的详细解释：

### 1. **spring.cache.type**
   - **描述**: 指定应用使用的缓存类型。Spring Boot 支持多个缓存提供者，通过该配置项选择具体的缓存实现。
   - **取值**:
     - `none`：禁用缓存。
     - `simple`：使用基于 `ConcurrentHashMap` 的内存缓存（默认配置）。
     - `generic`：一个可以配置自定义 `CacheManager` 的通用缓存。
     - `jcache`：使用 JSR-107（JCache）缓存标准。
     - `ehcache`：使用 Ehcache 缓存。
     - `hazelcast`：使用 Hazelcast 缓存。
     - `infinispan`：使用 Infinispan 缓存。
     - `couchbase`：使用 Couchbase 缓存。
     - `redis`：使用 Redis 缓存。
     - `caffeine`：使用 Caffeine 缓存。
     - `guava`：使用 Guava 缓存。
     - `custom`：自定义缓存类型，通过手动提供 `CacheManager`。
   - **默认值**: `simple`，如果没有明确配置其他缓存类型，会默认使用内存缓存。

### 2. **spring.cache.cache-names**
   - **描述**: 定义缓存的名字，支持多个缓存。设置该属性后，指定的缓存名称会自动创建。
   - **格式**: 逗号分隔的字符串，例如：`users, products, orders`。
   - **默认值**: 无，如果不配置该项，缓存会在运行时动态创建。

### 3. **spring.cache.jcache.config**
   - **描述**: 如果你使用的是 JSR-107（JCache）标准，指定 JCache 的配置文件路径。
   - **取值**: 例如 `classpath:cache-config.xml`，这个 XML 文件按照 JSR-107 的规范定义缓存配置。
   - **默认值**: 无。

### 4. **spring.cache.ehcache.config**
   - **描述**: 使用 Ehcache 缓存时，指定 Ehcache 配置文件路径。
   - **取值**: 例如 `classpath:ehcache.xml`，这个文件定义了 Ehcache 的缓存策略、过期时间等配置。
   - **默认值**: 无。

### 5. **spring.cache.caffeine.spec**
   - **描述**: 使用 Caffeine 缓存时，定义缓存的配置规范（如最大大小、过期策略等）。
   - **取值**: Caffeine 的缓存规范字符串，例如 `maximumSize=500,expireAfterAccess=600s`。
   - **默认值**: 无。

### 6. **spring.cache.redis.time-to-live**
   - **描述**: 配置 Redis 缓存条目的过期时间（TTL，Time To Live）。
   - **取值**: 时间值，例如 `60000ms` 表示 60 秒。可以使用时间单位，例如 `60s`, `5m`, `1h`。
   - **默认值**: 无限制，Redis 中的缓存条目默认不会过期，除非你明确指定。

### 7. **spring.cache.redis.cache-null-values**
   - **描述**: 是否允许缓存 `null` 值。
   - **取值**:
     - `true`：允许缓存 `null` 值。
     - `false`：不缓存 `null` 值。
   - **默认值**: `true`。

### 8. **spring.cache.redis.key-prefix**
   - **描述**: Redis 中缓存条目键的前缀。
   - **取值**: 一个字符串，例如 `myapp:`。这个前缀会添加到每个缓存键的前面，便于区分同一 Redis 实例中不同应用的缓存。
   - **默认值**: 无。

### 9. **spring.cache.redis.use-key-prefix**
   - **描述**: 是否使用指定的键前缀。
   - **取值**:
     - `true`：使用前缀。
     - `false`：不使用前缀。
   - **默认值**: `true`。

### 10. **spring.cache.redis.enable-statistics**
   - **描述**: 是否启用 Redis 缓存的统计信息（例如缓存命中率等）。
   - **取值**:
     - `true`：启用统计。
     - `false`：不启用统计。
   - **默认值**: `false`。

### 11. **spring.cache.redis.serializer**
   - **描述**: 指定 Redis 缓存的序列化方式。
   - **取值**: 可以选择合适的 Redis 序列化器，如 `json` 或 `jdk`。
   - **默认值**: 使用 Spring Boot 默认的 Redis 序列化器。

### 12. **spring.cache.redis.transaction-aware**
   - **描述**: 配置 Redis 缓存是否支持事务感知，确保缓存操作与事务同步。
   - **取值**:
     - `true`：启用事务感知，缓存操作将在事务提交后执行。
     - `false`：不启用事务感知，缓存操作将立即执行。
   - **默认值**: `false`。

### 13. **spring.cache.cache-null-values**
   - **描述**: 是否允许缓存 `null` 值（适用于大多数缓存类型）。
   - **取值**:
     - `true`：允许缓存 `null`。
     - `false`：不缓存 `null`。
   - **默认值**: `true`。

### 14. **spring.cache.default-cache-null-values**
   - **描述**: 是否默认允许所有缓存类型缓存 `null` 值。
   - **取值**: 与 `spring.cache.cache-null-values` 类似，但影响所有缓存类型的默认行为。
   - **默认值**: `true`。

### 配置示例
```yaml
spring:
  cache:
    type: redis  # 使用 Redis 作为缓存实现
    cache-names: users, products  # 定义缓存名称
    redis:
      time-to-live: 60000ms  # 60 秒过期时间
      key-prefix: myapp:  # 键前缀
      use-key-prefix: true  # 启用前缀
      cache-null-values: false  # 不缓存 null 值
      transaction-aware: true  # 启用事务感知
```

### 总结
Spring 的 `spring.cache` 配置项提供了统一的缓存管理抽象，支持多种缓存提供者，并且通过简单的配置就可以启用 Redis、Ehcache、Caffeine 等缓存实现。根据具体需求，你可以配置缓存的名称、过期时间、键前缀、序列化方式等细节，同时还能控制是否缓存 `null` 值、启用事务感知等功能。这种配置灵活性使得 Spring Cache 能够适应各种缓存场景。

# 7. spring.cache.type不支持同时配置多个缓存类型
Spring Boot 默认情况下不直接支持为同一个 `CacheManager` 配置多个类型的缓存（如同时使用 Redis 和 Caffeine 缓存），因为它设计的是通过一个 `CacheManager` 管理所有缓存。每种缓存类型（如 Redis、Caffeine、Ehcache 等）都有自己的 `CacheManager`，默认情况下，应用程序中通常会注册一个主 `CacheManager`，用于管理缓存。

如果你想同时配置多个类型的缓存（例如 Redis 和 Caffeine），你可以采取以下几种策略：

### 1. **配置多个 `CacheManager`**
你可以通过自定义 Bean 来注册多个 `CacheManager`，并根据缓存的用途选择使用哪个 `CacheManager`。

#### 配置示例：
假设你想使用 Caffeine 作为本地缓存，同时使用 Redis 作为远程缓存。

```java
@Configuration
public class CacheConfig {

    @Bean
    public CacheManager caffeineCacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
                .expireAfterWrite(60, TimeUnit.SECONDS)
                .maximumSize(100));
        return cacheManager;
    }

    @Bean
    public CacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory) {
        RedisCacheConfiguration cacheConfig = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10));
        return RedisCacheManager.builder(redisConnectionFactory)
                .cacheDefaults(cacheConfig)
                .build();
    }
}
```

### 2. **使用 `@Primary` 注解指定默认 `CacheManager`**
如果你定义了多个 `CacheManager`，可以使用 `@Primary` 注解指定哪个是默认的 `CacheManager`，没有明确指定时将使用默认的。

```java
@Bean
@Primary
public CacheManager primaryCacheManager() {
    // 配置默认 CacheManager，假设 Redis
}
```

然后，当你在某些场景下需要指定不同的 `CacheManager` 时，可以通过注入特定的 `CacheManager` 来使用：

```java
@Autowired
@Qualifier("caffeineCacheManager")
private CacheManager caffeineCacheManager;

@Autowired
@Qualifier("redisCacheManager")
private CacheManager redisCacheManager;
```

### 3. **自定义 `CacheResolver`**
Spring 提供了 `CacheResolver` 接口，它允许你根据不同的条件选择合适的 `CacheManager` 来处理缓存操作。通过自定义 `CacheResolver`，你可以灵活地选择缓存的实现。

#### 示例：
你可以创建一个 `CacheResolver`，根据缓存名称选择不同的 `CacheManager`。

```java
@Configuration
public class CustomCacheConfig {

    @Bean
    public CacheManager caffeineCacheManager() {
        return new CaffeineCacheManager();
    }

    @Bean
    public CacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory) {
        return RedisCacheManager.builder(redisConnectionFactory).build();
    }

    @Bean
    public CacheResolver cacheResolver(CacheManager caffeineCacheManager, CacheManager redisCacheManager) {
        return new CustomCacheResolver(caffeineCacheManager, redisCacheManager);
    }

    public static class CustomCacheResolver implements CacheResolver {

        private final CacheManager caffeineCacheManager;
        private final CacheManager redisCacheManager;

        public CustomCacheResolver(CacheManager caffeineCacheManager, CacheManager redisCacheManager) {
            this.caffeineCacheManager = caffeineCacheManager;
            this.redisCacheManager = redisCacheManager;
        }

        @Override
        public Collection<? extends Cache> resolveCaches(CacheOperationInvocationContext<?> context) {
            // 根据条件选择不同的缓存
            if (context.getMethod().getName().startsWith("find")) {
                return caffeineCacheManager.getCacheNames().stream()
                        .map(caffeineCacheManager::getCache)
                        .collect(Collectors.toList());
            } else {
                return redisCacheManager.getCacheNames().stream()
                        .map(redisCacheManager::getCache)
                        .collect(Collectors.toList());
            }
        }
    }
}
```

### 4. **指定不同 `CacheManager` 的缓存名称**
你也可以通过配置文件来指定哪些缓存使用特定的 `CacheManager`。例如，某些缓存使用 Redis，而其他缓存使用 Caffeine。你可以在缓存名上加上前缀或特定标识符，然后在代码中根据名称选择不同的 `CacheManager`。

### 总结
- **默认**情况下，Spring Boot 通过一个 `CacheManager` 管理缓存，不直接支持多个类型缓存的配置。
- **解决方案**可以通过定义多个 `CacheManager` Bean、使用 `CacheResolver` 或自定义逻辑来选择和配置不同的缓存类型。
- 多缓存类型适用于需要区分本地缓存和分布式缓存的场景，比如 Caffeine 作为短期本地缓存，而 Redis 作为远程持久化缓存。

# 8. CacheManager和缓存器类型的关系
是的，**CacheManager** 主要是一个缓存管理器，它负责管理同一种类型的多个缓存实例。每个 `CacheManager` 通常对应一种具体的缓存技术，如 Redis、Ehcache、Caffeine 等，它为这些缓存提供统一的操作接口，并将具体的缓存操作（如存取、删除等）委托给底层的缓存实现。

更具体地说：

1. **CacheManager 管理多个缓存实例**：  
   它管理的缓存实例可以通过缓存名称（`Cache` 名称）来区分。每个 `CacheManager` 都可以管理多个不同的缓存区域（`Cache` 实例），并允许根据名称进行缓存操作。每个 `Cache` 实例都有自己的一组缓存条目，存储着不同的数据。

2. **每个 CacheManager 负责一个缓存类型**：  
   每个 `CacheManager` 实际上是与某种特定的缓存类型绑定在一起的。例如，`RedisCacheManager` 管理 Redis 缓存，`CaffeineCacheManager` 管理 Caffeine 缓存。这意味着同一个 `CacheManager` 管理的所有缓存实例都遵循相同的缓存技术实现。

3. **操作统一性**：  
   `CacheManager` 提供了一个统一的接口，开发者不需要关心底层使用的缓存技术是 Redis、Ehcache 还是 Caffeine。所有缓存的操作，比如 `put()`、`get()`、`evict()` 等，都通过 `CacheManager` 和 `Cache` 抽象接口完成，隐藏了具体实现的细节。

4. **灵活扩展**：  
   如果在一个应用中有不同的缓存需求（例如，本地缓存和分布式缓存），可以通过定义多个 `CacheManager` 来管理不同类型的缓存。这样可以根据场景灵活地选择是使用本地缓存还是远程缓存。

### 示例：

假设你有以下两种缓存需求：

- 使用 **Redis** 作为分布式缓存存储用户会话信息。
- 使用 **Caffeine** 作为本地缓存来缓存一些计算结果，减少对外部接口的调用。

你可以为每种缓存技术定义各自的 `CacheManager`，每个 `CacheManager` 会管理与之相关的缓存实例（即多个不同的缓存区域）。

### RedisCacheManager 管理 Redis 缓存示例：
```java
@Bean
public RedisCacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory) {
    return RedisCacheManager.builder(redisConnectionFactory)
        .cacheDefaults(RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofHours(1)))
        .build();
}
```

### CaffeineCacheManager 管理 Caffeine 缓存示例：
```java
@Bean
public CaffeineCacheManager caffeineCacheManager() {
    CaffeineCacheManager cacheManager = new CaffeineCacheManager();
    cacheManager.setCaffeine(Caffeine.newBuilder()
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .maximumSize(1000));
    return cacheManager;
}
```

### 统一操作：
无论你使用 Redis 还是 Caffeine，缓存操作的代码可以保持一致。例如：

```java
@Cacheable(cacheNames = "users")
public User getUserById(String userId) {
    return userRepository.findById(userId);
}
```

这个 `@Cacheable` 注解会自动查找配置的 `CacheManager` 并根据名称（如 `users`）进行缓存操作。

### 总结：
- `CacheManager` 管理同一种类型的缓存实例（即某种缓存技术，如 Redis 或 Caffeine）。
- 每个 `CacheManager` 负责管理多个缓存区域（即不同的缓存实例）。
- `CacheManager` 提供了统一的接口，屏蔽了不同缓存技术的差异，便于操作和扩展。

# 9. CacheManager自动配置源码解析
***首先说下结论***
（1）CacheManager自动配置的首要前提条件是：容器中启动类或者其他被引入的配置类上，需要标注注解@EnableCaching。 如果没有@EnableCaching注解，则  
（2）如果不配置 spring.cache.type，自动配置将根据条件自动创建一个CacheManager对象。这个对象大多数情况下都是SimpleCacheManager类型的；除非在解析过程中，其他类的条件优先被满足，则会实例化第一个被满足条件的CacheManager。      
（3）如果配置了spring.cache.type，自动配置将创建指定类型的CacheManager对象，同时要想成功创建目标对象，还必须满足一些其他条件，比如容器中不存在CacheManager实例的情况下，classpath下必须存在目标类型的类等。      
（4）如果手动想容器中注册了CacheManager实例，那么自动配置将失效。请注意，spring.cache.type等配置只对自动配置有影响，如果是手动配置的CacheManager实例，那么将和spring.cache配置毫无关系。      

***下面将从CacheManager的自动配置源码详细解读***
