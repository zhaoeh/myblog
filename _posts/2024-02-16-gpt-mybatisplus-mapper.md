---
layout:     post
title:      mybatisPlus别名
subtitle:   mybatisPlus别名
categories: [gpt]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. mybatis中的setMapUnderscoreToCameCase设置为true有什么作用呢？具体的含义是什么？
在 MyBatis 中，将 `setMapUnderscoreToCamelCase` 设置为 `true` 可以使得 MyBatis 自动将数据库中的字段名从下划线命名（underscore case）转换为驼峰命名（camel case）。这对于数据库表的列名和 Java 实体类的属性名之间的映射非常有用。

### 具体含义
1. **下划线命名 (underscore case)**：
   - 这是数据库中常用的命名方式，其中单词之间用下划线分隔，例如 `user_name`、`created_at` 等。

2. **驼峰命名 (camel case)**：
   - 这是 Java 中常用的命名方式，其中第一个单词的首字母小写，后续单词的首字母大写，例如 `userName`、`createdAt` 等。

### 示例说明
假设有一个数据库表 `users`，其中有列名 `user_name` 和 `created_at`，对应的 Java 实体类如下：

```java
public class User {
    private String userName;
    private LocalDateTime createdAt;

    // getters and setters
}
```

如果 `setMapUnderscoreToCamelCase` 设置为 `true`，则 MyBatis 会自动将 `user_name` 映射到 `userName` 属性，将 `created_at` 映射到 `createdAt` 属性，而不需要额外的映射配置。

### 配置方法
可以在 MyBatis 配置文件（如 `mybatis-config.xml`）中设置：

```xml
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

或者在 Spring Boot 项目中的 `application.yml` 中设置：

```yaml
mybatis:
  configuration:
    map-underscore-to-camel-case: true
```

### 优点
- **减少代码冗余**：不需要在 MyBatis 的 XML 映射文件或注解中手动指定每个字段的映射关系。
- **代码更简洁**：提高了代码的可读性和维护性。

### 适用场景
适用于数据库字段命名规范和 Java 命名规范不一致的项目，尤其是数据库字段使用下划线命名而 Java 属性使用驼峰命名的情况。

通过设置 `setMapUnderscoreToCamelCase` 为 `true`，可以简化 MyBatis 的配置和映射工作，使开发更加高效。

# 2. 举个案例来说明一下，比如我现在查询的实体字段该如何命名呢
当然，这里有一个具体的案例来说明 `setMapUnderscoreToCamelCase` 设置为 `true` 时如何命名实体字段。

### 数据库表结构
假设有一个 `users` 表，结构如下：

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    user_name VARCHAR(50),
    created_at TIMESTAMP
);
```

### Java 实体类
假设你需要映射这个 `users` 表到一个 Java 实体类 `User`，并且你希望实体类的字段遵循驼峰命名规则。

#### 设置 `setMapUnderscoreToCamelCase` 为 `true`
首先，在 MyBatis 配置文件中设置 `mapUnderscoreToCamelCase`：

##### `mybatis-config.xml`
```xml
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

##### 或者 `application.yml`（Spring Boot 项目）
```yaml
mybatis:
  configuration:
    map-underscore-to-camel-case: true
```

#### 实体类命名
在设置了 `mapUnderscoreToCamelCase` 为 `true` 之后，你可以按照驼峰命名规则来定义实体类的字段名：

```java
public class User {
    private Integer id;
    private String userName;
    private LocalDateTime createdAt;

    // getters and setters
}
```

### MyBatis 映射
假设你使用 MyBatis 的注解来进行映射，映射查询的例子如下：

```java
@Mapper
public interface UserMapper {
    @Select("SELECT id, user_name, created_at FROM users WHERE id = #{id}")
    User getUserById(int id);
}
```

### 使用示例
查询用户时，MyBatis 会自动将 `user_name` 映射到 `userName`，将 `created_at` 映射到 `createdAt`，因为我们设置了 `mapUnderscoreToCamelCase` 为 `true`。

#### 测试代码
```java
public class UserTest {
    private UserMapper userMapper;

    @Test
    public void testGetUserById() {
        User user = userMapper.getUserById(1);
        System.out.println("UserName: " + user.getUserName());
        System.out.println("CreatedAt: " + user.getCreatedAt());
    }
}
```

通过上述配置和示例，MyBatis 会自动处理字段名的映射，你可以直接使用驼峰命名的字段名来定义 Java 实体类的属性，而不需要手动指定每个字段的映射关系。

# 3. setMapUnderscoreToCamelCase 设置为ture，如果执行insert语句，会自动将实体中的驼峰转换为数据库中的_格式么
是的，如果 `setMapUnderscoreToCamelCase` 设置为 `true`，MyBatis 会在执行 `INSERT` 语句时自动将 Java 实体类中的驼峰命名字段转换为数据库中的下划线命名字段。这种设置不仅在查询时有效，在插入和更新操作中同样适用。

### 示例说明

假设仍然使用前面的例子，有一个 `users` 表，结构如下：

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    user_name VARCHAR(50),
    created_at TIMESTAMP
);
```

### Java 实体类

```java
public class User {
    private Integer id;
    private String userName;
    private LocalDateTime createdAt;

    // getters and setters
}
```

### 配置 `setMapUnderscoreToCamelCase`

##### `mybatis-config.xml`
```xml
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

##### 或者 `application.yml`（Spring Boot 项目）
```yaml
mybatis:
  configuration:
    map-underscore-to-camel-case: true
```

### MyBatis Mapper 接口

使用 MyBatis 注解来定义插入操作：

```java
@Mapper
public interface UserMapper {
    @Insert("INSERT INTO users (id, user_name, created_at) VALUES (#{id}, #{userName}, #{createdAt})")
    void insertUser(User user);
}
```

### 使用示例

```java
public class UserTest {
    private UserMapper userMapper;

    @Test
    public void testInsertUser() {
        User user = new User();
        user.setId(1);
        user.setUserName("John Doe");
        user.setCreatedAt(LocalDateTime.now());

        userMapper.insertUser(user);
    }
}
```

在这个示例中，MyBatis 会自动将实体类 `User` 中的 `userName` 字段映射到数据库表 `users` 中的 `user_name` 列，`createdAt` 字段映射到 `created_at` 列。因此，当你执行 `insertUser` 方法时，MyBatis 会自动生成如下的 SQL 语句：

```sql
INSERT INTO users (id, user_name, created_at) VALUES (1, 'John Doe', '2024-06-20T12:34:56')
```

### 总结

通过设置 `setMapUnderscoreToCamelCase` 为 `true`，MyBatis 能够在执行 `INSERT`、`UPDATE` 等操作时自动处理 Java 实体类与数据库表字段之间的命名转换。这大大简化了开发过程，减少了手动映射的工作量，同时保证了代码的整洁和可读性。

# 4. 如果是直接使用mapper.xml的方式会自动进行映射转换么？
如果使用 `mapper.xml` 文件进行映射配置，并且 `setMapUnderscoreToCamelCase` 设置为 `true`，MyBatis 仍然会自动将实体类中的驼峰命名字段转换为数据库中的下划线命名字段。这适用于所有通过 MyBatis 处理的 SQL 操作，包括 `SELECT`、`INSERT`、`UPDATE` 和 `DELETE`。

### 示例说明

假设有一个 `users` 表，结构如下：

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    user_name VARCHAR(50),
    created_at TIMESTAMP
);
```

### Java 实体类

```java
public class User {
    private Integer id;
    private String userName;
    private LocalDateTime createdAt;

    // getters and setters
}
```

### 配置 `setMapUnderscoreToCamelCase`

##### `mybatis-config.xml`
```xml
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

##### 或者 `application.yml`（Spring Boot 项目）
```yaml
mybatis:
  configuration:
    map-underscore-to-camel-case: true
```

### MyBatis Mapper XML

在 `mapper.xml` 文件中定义 SQL 映射：

```xml
<mapper namespace="com.example.mapper.UserMapper">

    <insert id="insertUser" parameterType="com.example.domain.User">
        INSERT INTO users (id, user_name, created_at)
        VALUES (#{id}, #{userName}, #{createdAt})
    </insert>

    <select id="getUserById" parameterType="int" resultType="com.example.domain.User">
        SELECT id, user_name, created_at
        FROM users
        WHERE id = #{id}
    </select>

</mapper>
```

### MyBatis Mapper 接口

```java
public interface UserMapper {
    void insertUser(User user);
    User getUserById(int id);
}
```

### 使用示例

```java
public class UserTest {
    private UserMapper userMapper;

    @Test
    public void testInsertUser() {
        User user = new User();
        user.setId(1);
        user.setUserName("John Doe");
        user.setCreatedAt(LocalDateTime.now());

        userMapper.insertUser(user);
    }

    @Test
    public void testGetUserById() {
        User user = userMapper.getUserById(1);
        System.out.println("UserName: " + user.getUserName());
        System.out.println("CreatedAt: " + user.getCreatedAt());
    }
}
```

### 总结

通过设置 `setMapUnderscoreToCamelCase` 为 `true`，MyBatis 在使用 `mapper.xml` 文件进行映射配置时，依然会自动处理 Java 实体类与数据库表字段之间的命名转换。这意味着你可以在 XML 映射文件中使用驼峰命名的实体类字段名，而 MyBatis 会自动将它们转换为下划线命名的数据库字段名，从而简化了映射配置的编写和维护工作。

# 5. mybatisplus在执行sql时候如何根据实体自动补上某个字段作为条件呢
在 MyBatis-Plus 中，可以通过自定义全局拦截器或条件包装器来实现根据实体自动补上某个字段作为条件的功能。下面提供了几种方法来实现这一需求：

### 方法一：使用 MyBatis-Plus 提供的条件构造器（Wrapper）

MyBatis-Plus 提供了强大的条件构造器，`Wrapper` 和其子类 `QueryWrapper` 和 `UpdateWrapper`，可以灵活地构建动态 SQL。

#### 示例
假设有一个 `User` 实体类，我们希望在查询和更新时自动添加某个字段作为条件，例如 `status` 字段。

##### 实体类
```java
public class User {
    private Integer id;
    private String userName;
    private String status; // 例如，我们希望总是使用 status 作为条件
    private LocalDateTime createdAt;

    // getters and setters
}
```

##### Service 层
```java
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import org.springframework.stereotype.Service;

@Service
public class UserService extends ServiceImpl<UserMapper, User> {

    public List<User> getUsersByStatus(String status) {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("status", status);
        return this.list(queryWrapper);
    }

    public boolean updateUserStatus(User user) {
        UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
        updateWrapper.eq("status", user.getStatus());
        updateWrapper.eq("id", user.getId());
        return this.update(user, updateWrapper);
    }
}
```

### 方法二：自定义全局拦截器

如果希望在每次执行 SQL 时自动添加条件，可以通过自定义 MyBatis 拦截器实现。

#### 自定义拦截器
```java
import org.apache.ibatis.executor.statement.StatementHandler;
import org.apache.ibatis.plugin.*;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.mapping.MappedStatement;

import java.sql.Connection;
import java.util.Properties;

@Intercepts({
    @Signature(type = StatementHandler.class, method = "prepare", args = {Connection.class, Integer.class})
})
public class StatusInterceptor implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        StatementHandler statementHandler = (StatementHandler) invocation.getTarget();
        BoundSql boundSql = statementHandler.getBoundSql();
        MappedStatement mappedStatement = (MappedStatement) statementHandler.getParameterHandler().getParameterObject();
        
        // 在这里可以根据需要修改 SQL
        String originalSql = boundSql.getSql();
        String modifiedSql = originalSql + " AND status = 'ACTIVE'"; // 示例：添加 status 条件

        // 反射设置修改后的 SQL
        Field field = boundSql.getClass().getDeclaredField("sql");
        field.setAccessible(true);
        field.set(boundSql, modifiedSql);

        return invocation.proceed();
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
    }
}
```

#### 配置拦截器
在 MyBatis 配置中注册自定义拦截器：

##### `mybatis-config.xml`
```xml
<configuration>
    <plugins>
        <plugin interceptor="com.example.interceptor.StatusInterceptor"/>
    </plugins>
</configuration>
```

### 方法三：使用 BaseMapper 和自定义方法

扩展 MyBatis-Plus 提供的 `BaseMapper` 接口，通过自定义方法来实现。

#### 自定义方法
```java
public interface UserMapper extends BaseMapper<User> {
    default List<User> selectByStatus(String status) {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("status", status);
        return this.selectList(queryWrapper);
    }

    default int updateByStatus(User user) {
        UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
        updateWrapper.eq("status", user.getStatus());
        updateWrapper.eq("id", user.getId());
        return this.update(user, updateWrapper);
    }
}
```

### 使用示例

```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;

    public List<User> getUsersByStatus(String status) {
        return userMapper.selectByStatus(status);
    }

    public boolean updateUserStatus(User user) {
        return userMapper.updateByStatus(user) > 0;
    }
}
```

通过上述方法，可以在 MyBatis-Plus 执行 SQL 时根据实体自动补上某个字段作为条件，具体选择哪种方法取决于你的应用场景和需求。

# 6. mybatisplus中，关于多租户的处理，每次查询有tenent id的表时都默认加上了租户id作为条件，代码中有一个TenantLineInnerInterceptor
在 MyBatis-Plus 中，多租户的处理可以通过 `TenantLineInnerInterceptor` 来实现。这个拦截器会在每次查询时自动添加租户 ID 作为条件，从而实现多租户数据隔离。以下是一个详细的配置示例：

### 配置 `TenantLineInnerInterceptor`

#### 1. 引入依赖

首先，确保你的项目引入了 MyBatis-Plus 的相关依赖：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3.4</version> <!-- 请根据实际使用的版本号进行调整 -->
</dependency>
```

#### 2. 定义租户处理器

创建一个类实现 `TenantLineHandler` 接口，用于处理租户 ID 的逻辑：

```java
import com.baomidou.mybatisplus.extension.plugins.handler.TenantLineHandler;
import net.sf.jsqlparser.expression.Expression;
import net.sf.jsqlparser.expression.LongValue;

public class MyTenantLineHandler implements TenantLineHandler {
    // 返回租户ID的逻辑，可以从上下文中获取
    @Override
    public Expression getTenantId() {
        // 从上下文或线程中获取当前租户ID
        Long tenantId = TenantContext.getCurrentTenantId();
        return new LongValue(tenantId);
    }

    // 指定租户ID对应的数据库表字段名
    @Override
    public String getTenantIdColumn() {
        return "tenant_id";
    }

    // 指定哪些表需要进行租户过滤
    @Override
    public boolean doTableFilter(String tableName) {
        // 返回 true 表示不进行租户过滤，false 表示进行租户过滤
        return !tableName.equals("your_table_name");
    }
}
```

#### 3. 配置 MyBatis-Plus 拦截器

在 MyBatis-Plus 配置中注册 `TenantLineInnerInterceptor`：

```java
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.TenantLineInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 添加租户拦截器
        interceptor.addInnerInterceptor(new TenantLineInnerInterceptor(new MyTenantLineHandler()));
        return interceptor;
    }
}
```

#### 4. 上下文获取租户 ID

假设我们有一个 `TenantContext` 类来管理当前线程中的租户 ID：

```java
public class TenantContext {
    private static final ThreadLocal<Long> TENANT_ID = new ThreadLocal<>();

    public static void setCurrentTenantId(Long tenantId) {
        TENANT_ID.set(tenantId);
    }

    public static Long getCurrentTenantId() {
        return TENANT_ID.get();
    }

    public static void clear() {
        TENANT_ID.remove();
    }
}
```

在实际使用时，可以在用户登录后将租户 ID 设置到 `TenantContext` 中：

```java
// 用户登录后设置租户ID
TenantContext.setCurrentTenantId(user.getTenantId());
```

#### 5. 示例

假设有一个 `User` 表结构如下：

```sql
CREATE TABLE user (
    id BIGINT PRIMARY KEY,
    name VARCHAR(50),
    tenant_id BIGINT
);
```

在实体类中映射 `tenant_id`：

```java
public class User {
    private Long id;
    private String name;
    private Long tenantId;

    // getters and setters
}
```

在进行查询时，例如：

```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;

    public List<User> getUsers() {
        return userMapper.selectList(null);
    }
}
```

在执行查询时，`TenantLineInnerInterceptor` 会自动添加租户 ID 作为查询条件。例如，如果当前租户 ID 是 `1`，则生成的 SQL 可能是：

```sql
SELECT id, name, tenant_id FROM user WHERE tenant_id = 1;
```

通过这种方式，可以确保每个租户只能访问属于自己的数据，从而实现多租户数据隔离。

# 7. springboot中处理@ConditionalOnProperty注解时，判断的逻辑是什么样的？我想了解它判断的时机，比如它是判断Envrioment中有对应的属性时，还是在判断把对应的属性值注入到某个PropertyBean中之后才做的判断呢？
在 Spring Boot 中，`@ConditionalOnProperty` 注解用于条件性地注册 bean，只有在指定的配置属性满足条件时才会注册该 bean。其判断逻辑主要基于 Spring 的 `Environment`，即它会在配置属性加载到 `Environment` 中时进行判断，而不是在属性注入到某个 `PropertyBean` 之后。

### 具体的判断逻辑

1. **加载配置属性**：Spring Boot 启动时，会加载配置文件（如 `application.properties` 或 `application.yml`）中的属性到 `Environment` 中。

2. **解析注解**：Spring 解析带有 `@ConditionalOnProperty` 注解的 bean 定义，获取注解中的属性值。

3. **检查属性值**：`@ConditionalOnProperty` 会检查 `Environment` 中是否存在指定的属性，并判断其值是否符合要求。

### 判断的时机

`@ConditionalOnProperty` 判断的时机是在 Spring 容器初始化过程中，当 Spring 解析并注册 bean 定义时。这意味着在 Spring Boot 应用启动并加载配置文件之后，但在实际创建 bean 实例之前。

### 示例

假设你有以下配置属性：

```yaml
my.feature.enabled: true
```

定义一个 bean 只有在 `my.feature.enabled` 为 `true` 时才会注册：

```java
@Configuration
public class MyFeatureConfiguration {

    @Bean
    @ConditionalOnProperty(name = "my.feature.enabled", havingValue = "true")
    public MyFeature myFeature() {
        return new MyFeature();
    }
}
```

### 内部实现

`@ConditionalOnProperty` 注解的核心实现是通过 `OnPropertyCondition` 类，该类实现了 `Condition` 接口。其主要逻辑如下：

```java
public class OnPropertyCondition extends SpringBootCondition {

    @Override
    public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 获取注解中的属性
        MultiValueMap<String, Object> attributes = metadata.getAllAnnotationAttributes(ConditionalOnProperty.class.getName());
        if (attributes == null) {
            return ConditionOutcome.noMatch("No attributes found");
        }

        // 读取属性名和期望的值
        String prefix = getAttribute(attributes, "prefix");
        String name = getAttribute(attributes, "name");
        String havingValue = getAttribute(attributes, "havingValue");

        // 从 Environment 中获取实际值
        Environment environment = context.getEnvironment();
        String propertyValue = environment.getProperty(prefix + name);

        // 判断属性是否存在以及值是否匹配
        if (propertyValue != null && propertyValue.equals(havingValue)) {
            return ConditionOutcome.match();
        } else {
            return ConditionOutcome.noMatch("Property " + prefix + name + " does not have expected value " + havingValue);
        }
    }

    private String getAttribute(MultiValueMap<String, Object> attributes, String name) {
        Object value = attributes.getFirst(name);
        return value != null ? value.toString() : null;
    }
}
```

### 结论

`@ConditionalOnProperty` 注解在 Spring Boot 中基于 `Environment` 进行判断。当 Spring 容器启动并加载配置属性到 `Environment` 中之后，但在实际创建 bean 实例之前，Spring 解析并判断 `@ConditionalOnProperty` 注解，根据配置属性决定是否注册该 bean。因此，判断逻辑是在 `Environment` 中检查属性值，而不是在将属性注入到某个 `PropertyBean` 之后。

# 8. sql查询中，order by和select的执行顺序
在 SQL 查询的执行过程中，`ORDER BY` 和 `SELECT` 语句的执行顺序是有固定的执行顺序的。以下是标准的 SQL 查询的执行顺序：

1. **FROM**: 确定要查询的表或视图。
2. **JOIN**: 执行表的连接操作。
3. **WHERE**: 过滤不满足条件的记录。
4. **GROUP BY**: 将记录分组。
5. **HAVING**: 过滤分组后的记录。
6. **SELECT**: 确定要返回的列。
7. **DISTINCT**: 去重操作。
8. **ORDER BY**: 对结果集排序。
9. **LIMIT**: 限制返回的记录数。

因此，`ORDER BY` 是在 `SELECT` 之后执行的。这意味着在排序之前，已经确定了要返回的列和记录。

### 示例

考虑以下示例查询：

```sql
SELECT column1, column2
FROM table_name
WHERE condition
GROUP BY column1
HAVING condition
ORDER BY column2;
```

这个查询的执行步骤如下：

1. **FROM**: 从 `table_name` 表中读取数据。
2. **WHERE**: 过滤满足 `condition` 条件的记录。
3. **GROUP BY**: 按 `column1` 分组。
4. **HAVING**: 过滤分组后的记录，保留满足 `HAVING` 条件的组。
5. **SELECT**: 确定要返回的列 `column1` 和 `column2`。
6. **ORDER BY**: 按 `column2` 对结果集进行排序。

### 实际执行顺序示例

假设有一个名为 `employees` 的表，其中包含以下数据：

| id | name   | department | salary |
|----|--------|------------|--------|
| 1  | Alice  | HR         | 5000   |
| 2  | Bob    | IT         | 7000   |
| 3  | Charlie| HR         | 6000   |
| 4  | David  | IT         | 5500   |
| 5  | Eve    | IT         | 6000   |

执行以下查询：

```sql
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 5500
ORDER BY avg_salary DESC;
```

执行过程如下：

1. **FROM**: 从 `employees` 表中读取所有记录。
2. **GROUP BY**: 按 `department` 列进行分组，得到以下结果：

    | department | salaries        |
    |------------|-----------------|
    | HR         | [5000, 6000]    |
    | IT         | [7000, 5500, 6000] |

3. **HAVING**: 计算每个组的平均薪资，并过滤出平均薪资大于 5500 的组：

    | department | avg_salary |
    |------------|------------|
    | IT         | 6166.67    |

4. **SELECT**: 选择 `department` 和 `avg_salary` 列：

    | department | avg_salary |
    |------------|------------|
    | IT         | 6166.67    |

5. **ORDER BY**: 按 `avg_salary` 降序排序。由于这里只有一个结果，所以排序没有实际影响。

结果是：

| department | avg_salary |
|------------|------------|
| IT         | 6166.67    |

### 总结

在 SQL 查询中，`ORDER BY` 在 `SELECT` 之后执行。这意味着在排序之前，查询已经完成了所有的筛选、分组和列选择。理解这一点对于优化查询和预期结果非常重要。

# 9. 如果select对字段列起别名了，order by使用别名还是数据表字段名呢
在 SQL 查询中，如果 `SELECT` 语句对字段列起了别名，可以在 `ORDER BY` 子句中使用这个别名。大多数数据库管理系统（DBMS）都支持这种用法，这使得查询更简洁和易读。

### 示例

假设有一个名为 `employees` 的表，其中包含以下数据：

| id | name   | department | salary |
|----|--------|------------|--------|
| 1  | Alice  | HR         | 5000   |
| 2  | Bob    | IT         | 7000   |
| 3  | Charlie| HR         | 6000   |
| 4  | David  | IT         | 5500   |
| 5  | Eve    | IT         | 6000   |

#### 使用别名进行排序

```sql
SELECT name, salary AS employee_salary
FROM employees
ORDER BY employee_salary DESC;
```

在这个查询中，`salary` 列被起了一个别名 `employee_salary`，然后在 `ORDER BY` 子句中使用这个别名进行排序。查询将按 `employee_salary` 的降序排列结果：

| name    | employee_salary |
|---------|-----------------|
| Bob     | 7000            |
| Charlie | 6000            |
| Eve     | 6000            |
| David   | 5500            |
| Alice   | 5000            |

### DBMS 支持情况

大多数主流的数据库管理系统（如 MySQL、PostgreSQL、SQL Server、Oracle 等）都支持在 `ORDER BY` 子句中使用 `SELECT` 中的别名。这是一个常见的特性，用于提高查询的可读性。

### 注意事项

- **优先级**：在某些情况下，如果 `SELECT` 中的别名和表中的字段名相同，可能会引起歧义。在这种情况下，明确地使用别名会更加清晰。
- **标准化**：为了确保跨数据库的一致性和避免潜在的兼容性问题，建议在使用别名时保持命名的一致性和唯一性。

### 总结

在 SQL 查询中，可以在 `ORDER BY` 子句中使用 `SELECT` 语句中的别名。这使得查询语句更简洁、更易读，并且是大多数数据库管理系统所支持的常见用法。

# 10. order by不使用select中的别名是不是更合规一些
从 SQL 标准和跨数据库兼容性的角度来看，`ORDER BY` 使用 `SELECT` 中的别名是完全合规的，也是大多数主流数据库管理系统（DBMS）所支持的。然而，是否使用别名主要取决于具体的需求和项目的风格规范。

### 使用别名的优势

1. **可读性**：使用别名可以提高查询的可读性，尤其是在字段名称较长或复杂的情况下。
2. **一致性**：当在多个地方引用同一个计算列时，使用别名可以确保一致性和减少重复代码。
3. **便捷性**：在一些复杂的查询中，别名可以简化 `ORDER BY` 子句的书写。

### 示例

假设我们有一个 `employees` 表：

| id | name   | department | salary |
|----|--------|------------|--------|
| 1  | Alice  | HR         | 5000   |
| 2  | Bob    | IT         | 7000   |
| 3  | Charlie| HR         | 6000   |
| 4  | David  | IT         | 5500   |
| 5  | Eve    | IT         | 6000   |

我们可以通过使用别名来简化 `ORDER BY`：

```sql
SELECT name, salary AS employee_salary
FROM employees
ORDER BY employee_salary DESC;
```

这个查询将按 `employee_salary` 降序排列，结果与原始列名排序一样：

| name    | employee_salary |
|---------|-----------------|
| Bob     | 7000            |
| Charlie | 6000            |
| Eve     | 6000            |
| David   | 5500            |
| Alice   | 5000            |

### 不使用别名的示例

为了最大化兼容性和避免潜在的问题，一些开发者可能选择在 `ORDER BY` 子句中直接使用原始列名或表达式：

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC;
```

或者在更复杂的计算场景中，直接使用计算表达式：

```sql
SELECT name, (salary * 1.1) AS adjusted_salary
FROM employees
ORDER BY (salary * 1.1) DESC;
```

### 兼容性和合规性

大多数现代数据库系统（如 MySQL、PostgreSQL、SQL Server、Oracle 等）都支持在 `ORDER BY` 子句中使用 `SELECT` 中定义的别名。因此，从技术合规性和兼容性角度来看，使用别名是没有问题的。

### 结论

使用别名或原始列名取决于具体的需求和团队的编码规范。使用别名可以提高可读性和便捷性，而直接使用原始列名或表达式可以确保最大兼容性。在实际项目中，选择哪种方式取决于团队的风格指南和对代码可维护性的需求。
