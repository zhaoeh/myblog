---
layout:     post
title:      spring读取环境配置项
subtitle:   spring读取配置项
categories: [gpt]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. springboot解析配置项，如何注入List<Pojo>，比如我的Pojo中有String类型的name和value
在 Spring Boot 中，可以通过使用 `@ConfigurationProperties` 注解和一个配置类来将配置文件中的数据注入到一个 `List<Pojo>` 中。下面是详细步骤：

### 步骤 1: 创建 POJO 类

首先，创建一个简单的 POJO 类，其中包含 `name` 和 `value` 两个属性：

```java
public class MyPojo {
    private String name;
    private String value;

    // Getters and Setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }
}
```

### 步骤 2: 创建配置类

创建一个配置类，并使用 `@ConfigurationProperties` 注解。这个类将包含一个 `List<MyPojo>` 类型的属性：

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import java.util.List;

@Configuration
@ConfigurationProperties(prefix = "myapp")
public class MyAppProperties {
    private List<MyPojo> pojos;

    // Getters and Setters

    public List<MyPojo> getPojos() {
        return pojos;
    }

    public void setPojos(List<MyPojo> pojos) {
        this.pojos = pojos;
    }
}
```

### 步骤 3: 在配置文件中定义配置项

在 `application.yml` 或 `application.properties` 文件中定义配置项。这里假设使用 `application.yml` 文件：

```yaml
myapp:
  pojos:
    - name: "Pojo1"
      value: "Value1"
    - name: "Pojo2"
      value: "Value2"
```

### 步骤 4: 使配置生效

确保在启动类中启用 `@EnableConfigurationProperties` 注解：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;

@SpringBootApplication
@EnableConfigurationProperties(MyAppProperties.class)
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

### 步骤 5: 使用注入的配置

你可以在任何需要的地方注入 `MyAppProperties` 类，并使用其中的 `List<MyPojo>`：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.List;

@RestController
public class MyController {

    private final MyAppProperties myAppProperties;

    @Autowired
    public MyController(MyAppProperties myAppProperties) {
        this.myAppProperties = myAppProperties;
    }

    @GetMapping("/pojos")
    public List<MyPojo> getPojos() {
        return myAppProperties.getPojos();
    }
}
```

### 总结

通过上述步骤，你可以在 Spring Boot 中将配置文件中的数据注入到一个 `List<Pojo>` 中。关键点在于使用 `@ConfigurationProperties` 注解和配置文件中的正确格式。这样可以方便地管理和访问配置数据。

# 2. 如果yml配置项的值，有空格，需要特殊处理吗
在 YAML 文件中，如果配置项的值包含空格，一般不需要特殊处理，YAML 会正确解析这些值。不过，如果值的开头或结尾包含空格，或者包含特殊字符（如冒号、破折号等），可以使用双引号或单引号将值括起来，以确保 YAML 文件能够正确解析这些值。

### 示例

以下是一些在 YAML 文件中包含空格的配置项示例：

```yaml
myapp:
  pojos:
    - name: "Pojo 1"
      value: "Value 1"
    - name: "Pojo 2 with leading and trailing spaces "
      value: "Value 2 with : special characters"
    - name: 'Pojo 3 with single quotes'
      value: 'Value 3'
    - name: Pojo4
      value: Value4 without quotes
```

### YAML 解析

在上述示例中：

- `Pojo 1` 和 `Value 1` 使用双引号括起来，可以包含空格和其他特殊字符。
- `Pojo 2 with leading and trailing spaces` 使用双引号括起来，以确保开头和结尾的空格能够被正确解析。
- `Value 2 with : special characters` 使用双引号括起来，因为其中包含冒号。
- `Pojo 3 with single quotes` 和 `Value 3` 使用单引号括起来，也可以包含空格和特殊字符。
- `Pojo4` 和 `Value4 without quotes` 没有使用引号，因为它们不包含特殊字符或空格。

### 完整示例

将上述配置与前面的代码结合，形成完整的 Spring Boot 应用配置：

#### 1. `application.yml`

```yaml
myapp:
  pojos:
    - name: "Pojo 1"
      value: "Value 1"
    - name: "Pojo 2 with leading and trailing spaces "
      value: "Value 2 with : special characters"
    - name: 'Pojo 3 with single quotes'
      value: 'Value 3'
    - name: Pojo4
      value: Value4 without quotes
```

#### 2. `MyPojo.java`

```java
public class MyPojo {
    private String name;
    private String value;

    // Getters and Setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }
}
```

#### 3. `MyAppProperties.java`

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import java.util.List;

@Configuration
@ConfigurationProperties(prefix = "myapp")
public class MyAppProperties {
    private List<MyPojo> pojos;

    // Getters and Setters

    public List<MyPojo> getPojos() {
        return pojos;
    }

    public void setPojos(List<MyPojo> pojos) {
        this.pojos = pojos;
    }
}
```

#### 4. 启动类

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;

@SpringBootApplication
@EnableConfigurationProperties(MyAppProperties.class)
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

#### 5. 控制器

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.List;

@RestController
public class MyController {

    private final MyAppProperties myAppProperties;

    @Autowired
    public MyController(MyAppProperties myAppProperties) {
        this.myAppProperties = myAppProperties;
    }

    @GetMapping("/pojos")
    public List<MyPojo> getPojos() {
        return myAppProperties.getPojos();
    }
}
```

通过这种方式，Spring Boot 会正确解析 YAML 文件中的配置项，并将它们注入到 `MyAppProperties` 中，无论这些值是否包含空格或特殊字符。

# 3. spring validation如何支持嵌套校验
在 Spring Validation 中，可以使用嵌套校验来验证复杂的对象图或嵌套的属性。为了实现这一点，需要使用 `@Valid` 注解来标记需要嵌套验证的字段，同时在嵌套对象中使用相应的验证注解。以下是详细步骤和示例：

### 步骤 1: 定义嵌套对象

首先，定义嵌套的 POJO 类，并使用 JSR 303/JSR 380 注解进行验证。

#### Address 类

```java
import javax.validation.constraints.NotBlank;

public class Address {

    @NotBlank(message = "Street cannot be blank")
    private String street;

    @NotBlank(message = "City cannot be blank")
    private String city;

    // Getters and Setters

    public String getStreet() {
        return street;
    }

    public void setStreet(String street) {
        this.street = street;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }
}
```

### 步骤 2: 定义主对象

定义包含嵌套对象的主 POJO 类，并使用 `@Valid` 注解来标记嵌套对象。

#### User 类

```java
import javax.validation.Valid;
import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;

public class User {

    @NotBlank(message = "Name cannot be blank")
    private String name;

    @Email(message = "Email should be valid")
    private String email;

    @Valid
    private Address address;

    // Getters and Setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }
}
```

### 步骤 3: 创建控制器来处理验证

在控制器中使用 `@Valid` 注解来验证传入的主对象。

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;

@RestController
@Validated
public class UserController {

    @PostMapping("/users")
    public ResponseEntity<String> createUser(@Valid @RequestBody User user) {
        // 如果验证通过，执行创建用户的逻辑
        return new ResponseEntity<>("User is valid", HttpStatus.OK);
    }
}
```

### 步骤 4: 处理验证错误

可以使用 `@ExceptionHandler` 注解来处理验证错误，并返回友好的错误消息。

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

import java.util.HashMap;
import java.util.Map;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((org.springframework.validation.FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
    }
}
```

### 完整示例

综合上述步骤，以下是一个完整的 Spring Boot 应用示例，演示了嵌套验证的实现。

#### 1. `Address.java`

```java
import javax.validation.constraints.NotBlank;

public class Address {

    @NotBlank(message = "Street cannot be blank")
    private String street;

    @NotBlank(message = "City cannot be blank")
    private String city;

    // Getters and Setters
}
```

#### 2. `User.java`

```java
import javax.validation.Valid;
import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;

public class User {

    @NotBlank(message = "Name cannot be blank")
    private String name;

    @Email(message = "Email should be valid")
    private String email;

    @Valid
    private Address address;

    // Getters and Setters
}
```

#### 3. `UserController.java`

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;

@RestController
@Validated
public class UserController {

    @PostMapping("/users")
    public ResponseEntity<String> createUser(@Valid @RequestBody User user) {
        // 如果验证通过，执行创建用户的逻辑
        return new ResponseEntity<>("User is valid", HttpStatus.OK);
    }
}
```

#### 4. `GlobalExceptionHandler.java`

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

import java.util.HashMap;
import java.util.Map;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((org.springframework.validation.FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
    }
}
```

通过这种方式，Spring Validation 能够支持嵌套校验，并确保所有嵌套对象中的字段都能正确地进行验证。

# 4. 如果允许address为null呢
如果需要允许 `address` 字段为 `null`，但当 `address` 字段存在时要对其进行验证，可以在 `User` 类中的 `address` 字段上使用 `@Valid` 和 `@Nullable` 注解，并确保验证逻辑能够处理这种情况。

### 修改后的 `User` 类

```java
import javax.validation.Valid;
import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Null;

public class User {

    @NotBlank(message = "Name cannot be blank")
    private String name;

    @Email(message = "Email should be valid")
    private String email;

    @Valid
    @Null
    private Address address;

    // Getters and Setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }
}
```

### 使用 `@Null` 和 `@Valid`

`@Valid` 注解用于触发嵌套对象的验证，而 `@Null` 注解可以用来允许字段为 `null`。这样，当 `address` 字段不为 `null` 时，嵌套的 `Address` 对象将被验证。

### 控制器和异常处理器不需要更改

控制器和异常处理器的代码不需要任何更改。验证逻辑将处理 `address` 字段为 `null` 的情况。

### 完整示例

综合上述步骤，以下是一个完整的 Spring Boot 应用示例，演示了允许 `address` 字段为 `null` 的嵌套验证实现。

#### 1. `Address.java`

```java
import javax.validation.constraints.NotBlank;

public class Address {

    @NotBlank(message = "Street cannot be blank")
    private String street;

    @NotBlank(message = "City cannot be blank")
    private String city;

    // Getters and Setters

    public String getStreet() {
        return street;
    }

    public void setStreet(String street) {
        this.street = street;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }
}
```

#### 2. `User.java`

```java
import javax.validation.Valid;
import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;

public class User {

    @NotBlank(message = "Name cannot be blank")
    private String name;

    @Email(message = "Email should be valid")
    private String email;

    @Valid
    private Address address;

    // Getters and Setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }
}
```

#### 3. `UserController.java`

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;

@RestController
@Validated
public class UserController {

    @PostMapping("/users")
    public ResponseEntity<String> createUser(@Valid @RequestBody User user) {
        // 如果验证通过，执行创建用户的逻辑
        return new ResponseEntity<>("User is valid", HttpStatus.OK);
    }
}
```

#### 4. `GlobalExceptionHandler.java`

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

import java.util.HashMap;
import java.util.Map;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((org.springframework.validation.FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
    }
}
```

通过这种方式，`address` 字段可以为 `null`，但如果它不为 `null`，其内部字段将会被验证。这样可以确保应用的验证逻辑更加灵活，适应更多实际场景。

# 5. springboot如何读取配置项，读取为一个map结构，这个map的key是String，value是一个java对象比如User，User里面有String类型的name和int类型的age
在 Spring Boot 中，可以通过配置属性读取配置文件中的内容，并将其映射到一个 `Map<String, User>` 结构中。可以使用 `@ConfigurationProperties` 注解来实现这一点。

下面是一个具体的示例，展示如何读取配置项，并将其转换为一个 `Map<String, User>` 结构，其中 `User` 对象包含 `name` 和 `age` 属性。

### 1. 配置文件

首先，在 `application.yml` 或 `application.properties` 文件中定义你的配置项。这里使用 `application.yml` 作为示例：

```yaml
users:
  user1:
    name: Alice
    age: 30
  user2:
    name: Bob
    age: 25
```

### 2. 创建 `User` 类

创建一个 `User` 类，用于映射配置文件中的用户信息：

```java
public class User {
    private String name;
    private int age;

    // Getters and setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

### 3. 创建配置类

使用 `@ConfigurationProperties` 注解创建一个配置类，将配置项映射到 `Map<String, User>` 结构：

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

@Configuration
@ConfigurationProperties(prefix = "users")
public class UserConfig {
    private Map<String, User> userMap;

    // Getters and setters

    public Map<String, User> getUserMap() {
        return userMap;
    }

    public void setUserMap(Map<String, User> userMap) {
        this.userMap = userMap;
    }
}
```

### 4. 启用配置属性

在你的 Spring Boot 应用主类中启用配置属性：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;

@SpringBootApplication
@EnableConfigurationProperties(UserConfig.class)
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### 5. 使用配置

现在，你可以在任何需要的地方注入 `UserConfig` 并使用 `userMap`：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

@Component
public class UserPrinter {
    private final UserConfig userConfig;

    @Autowired
    public UserPrinter(UserConfig userConfig) {
        this.userConfig = userConfig;
    }

    @PostConstruct
    public void printUsers() {
        userConfig.getUserMap().forEach((key, user) -> {
            System.out.println("Key: " + key + ", User: " + user);
        });
    }
}
```

### 6. 运行应用

运行你的 Spring Boot 应用，你应该会在控制台看到以下输出：

```
Key: user1, User: User{name='Alice', age=30}
Key: user2, User: User{name='Bob', age=25}
```

### 总结

通过上述步骤，你可以在 Spring Boot 中将配置文件中的内容映射到一个 `Map<String, User>` 结构中，并轻松读取和使用这些配置项。这种方法利用了 Spring Boot 的 `@ConfigurationProperties` 注解和自动绑定特性，使配置管理变得简单和高效。

# 6. 你这个案例中，配置项的命名和类中字段的命名不符合规范啊
好的，了解了。我们可以优化配置项和类中的字段命名，使其更符合 Java 和 YAML 配置文件的命名规范。

### 1. 配置文件

我们使用符合规范的命名方式，在 `application.yml` 中配置用户信息：

```yaml
users:
  user1:
    name: Alice
    age: 30
  user2:
    name: Bob
    age: 25
```

### 2. 创建 `User` 类

创建一个 `User` 类，用于映射配置文件中的用户信息，确保字段名与配置项一致：

```java
public class User {
    private String name;
    private int age;

    // Getters and setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

### 3. 创建配置类

使用 `@ConfigurationProperties` 注解创建一个配置类，将配置项映射到 `Map<String, User>` 结构：

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

@Configuration
@ConfigurationProperties(prefix = "users")
public class UserConfig {
    private Map<String, User> userMap;

    // Getters and setters
    public Map<String, User> getUserMap() {
        return userMap;
    }

    public void setUserMap(Map<String, User> userMap) {
        this.userMap = userMap;
    }
}
```

### 4. 启用配置属性

在你的 Spring Boot 应用主类中启用配置属性：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;

@SpringBootApplication
@EnableConfigurationProperties(UserConfig.class)
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### 5. 使用配置

现在，你可以在任何需要的地方注入 `UserConfig` 并使用 `userMap`：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

@Component
public class UserPrinter {
    private final UserConfig userConfig;

    @Autowired
    public UserPrinter(UserConfig userConfig) {
        this.userConfig = userConfig;
    }

    @PostConstruct
    public void printUsers() {
        userConfig.getUserMap().forEach((key, user) -> {
            System.out.println("Key: " + key + ", User: " + user);
        });
    }
}
```

### 6. 运行应用

运行你的 Spring Boot 应用，你应该会在控制台看到以下输出：

```
Key: user1, User: User{name='Alice', age=30}
Key: user2, User: User{name='Bob', age=25}
```

### 总结

通过上述步骤，我们使用符合规范的命名方式，将配置文件中的内容映射到一个 `Map<String, User>` 结构中，并轻松读取和使用这些配置项。这样不仅保持了代码的清晰和可维护性，还确保了命名规范的一致性。

# 7. userMap不需要和配置项中的名称保持一致吗
在 Spring Boot 中，使用 `@ConfigurationProperties` 注解时，配置类中的字段名称不需要与配置文件中的名称完全一致。Spring Boot 可以自动将 YAML 配置文件中的嵌套结构映射到 Java 对象中的字段。如果有需要，可以使用注解 `@Value` 或者通过提供自定义的 `Binder` 来实现更细粒度的控制。

### 调整示例，使字段名称更加规范

让我们重新整理一下代码，使配置类中的字段名称保持一致，并符合 Java 的命名规范。

### 1. 配置文件

在 `application.yml` 中定义你的配置项：

```yaml
users:
  user1:
    name: Alice
    age: 30
  user2:
    name: Bob
    age: 25
```

### 2. 创建 `User` 类

创建一个 `User` 类，用于映射配置文件中的用户信息：

```java
public class User {
    private String name;
    private int age;

    // Getters and setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

### 3. 创建配置类

使用 `@ConfigurationProperties` 注解创建一个配置类，将配置项映射到 `Map<String, User>` 结构：

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

@Configuration
@ConfigurationProperties(prefix = "users")
public class UserConfig {
    private Map<String, User> map;

    // Getters and setters

    public Map<String, User> getMap() {
        return map;
    }

    public void setMap(Map<String, User> map) {
        this.map = map;
    }
}
```

### 4. 启用配置属性

在你的 Spring Boot 应用主类中启用配置属性：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;

@SpringBootApplication
@EnableConfigurationProperties(UserConfig.class)
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### 5. 使用配置

现在，你可以在任何需要的地方注入 `UserConfig` 并使用 `map`：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

@Component
public class UserPrinter {
    private final UserConfig userConfig;

    @Autowired
    public UserPrinter(UserConfig userConfig) {
        this.userConfig = userConfig;
    }

    @PostConstruct
    public void printUsers() {
        userConfig.getMap().forEach((key, user) -> {
            System.out.println("Key: " + key + ", User: " + user);
        });
    }
}
```

### 6. 运行应用

运行你的 Spring Boot 应用，你应该会在控制台看到以下输出：

```
Key: user1, User: User{name='Alice', age=30}
Key: user2, User: User{name='Bob', age=25}
```

### 总结

通过上述步骤，我们确保了配置文件中的命名和配置类中的字段名一致，同时遵循了 Java 的命名规范。这样不仅保持了代码的清晰和可维护性，还确保了命名规范的一致性。Spring Boot 的 `@ConfigurationProperties` 注解会自动映射配置文件中的嵌套结构到 Java 对象中的字段，使配置管理变得简单和高效。

# 8. @ConfigurationProperties这种方式可以指定默认值吗
`@ConfigurationProperties` 注解本身不直接支持指定默认值。但是，可以通过以下几种方式为配置属性设置默认值：

1. **在字段声明中初始化默认值**：
   直接在字段声明时赋予默认值。

2. **在配置类的构造方法或 `@PostConstruct` 方法中初始化默认值**：
   在类的构造方法或 `@PostConstruct` 方法中设置默认值。

3. **使用 `@Value` 注解**：
   可以在 `@ConfigurationProperties` 类中结合使用 `@Value` 注解为特定字段设置默认值。

### 示例代码

以下示例展示了如何使用这几种方法为配置属性设置默认值。

#### 1. 在字段声明中初始化默认值

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

@Configuration
@ConfigurationProperties(prefix = "users")
public class UserConfig {
    private Map<String, User> map = new HashMap<>(); // 设置默认值为空的 HashMap

    public Map<String, User> getMap() {
        return map;
    }

    public void setMap(Map<String, User> map) {
        this.map = map;
    }
}
```

#### 2. 在构造方法或 `@PostConstruct` 方法中初始化默认值

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;
import java.util.HashMap;
import java.util.Map;

@Configuration
@ConfigurationProperties(prefix = "users")
public class UserConfig {
    private Map<String, User> map;

    public Map<String, User> getMap() {
        return map;
    }

    public void setMap(Map<String, User> map) {
        this.map = map;
    }

    @PostConstruct
    public void init() {
        if (this.map == null) {
            this.map = new HashMap<>();
        }
    }
}
```

#### 3. 使用 `@Value` 注解

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

@Configuration
@ConfigurationProperties(prefix = "users")
public class UserConfig {

    @Value("#{new java.util.HashMap()}")
    private Map<String, User> map;

    public Map<String, User> getMap() {
        return map;
    }

    public void setMap(Map<String, User> map) {
        this.map = map;
    }
}
```

### 用户类 `User`

```java
public class User {
    private String name = "Default Name"; // 默认值
    private int age = 18; // 默认值

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

### 总结

虽然 `@ConfigurationProperties` 本身不直接支持设置默认值，但可以通过字段初始化、构造方法、`@PostConstruct` 方法以及结合 `@Value` 注解等方式来实现默认值设置。这些方法都能有效地确保在配置文件中未指定值时，配置类能够使用默认值。

# 9. 通过这种方式我在postConstruct中debug获取到的配置map为null，是不是和配置项的命名有关系啊？之前不是说会自动映射的吗？不指定规则会自动映射吗
`@ConfigurationProperties` 注解确实会自动将配置文件中的属性映射到配置类中的字段，但映射成功的前提是属性名称与配置类中的字段名称必须匹配。如果在 `@PostConstruct` 中获取到的 `map` 为 `null`，可能是以下几个原因：

1. 配置文件中的属性名称与配置类中的字段名称不匹配。
2. 配置文件中的属性未配置或配置错误。
3. Spring Boot 没有正确加载配置文件。

### 确保属性名称匹配

确保 `application.yml` 中的配置项与 `UserConfig` 类中的字段名称完全匹配。示例：

#### application.yml

```yaml
users:
  user1:
    name: Alice
    age: 30
  user2:
    name: Bob
    age: 25
```

#### UserConfig.java

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

@Configuration
@ConfigurationProperties(prefix = "users")
public class UserConfig {
    private Map<String, User> map;

    // Getters and setters
    public Map<String, User> getMap() {
        return map;
    }

    public void setMap(Map<String, User> map) {
        this.map = map;
    }
}
```

### 使用 `@PostConstruct` 方法

确保在 `@PostConstruct` 方法中检查配置类是否正确注入：

```java
import javax.annotation.PostConstruct;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class UserPrinter {
    private final UserConfig userConfig;

    @Autowired
    public UserPrinter(UserConfig userConfig) {
        this.userConfig = userConfig;
    }

    @PostConstruct
    public void printUsers() {
        if (userConfig.getMap() == null) {
            System.out.println("Map is null");
        } else {
            userConfig.getMap().forEach((key, user) -> {
                System.out.println("Key: " + key + ", User: " + user);
            });
        }
    }
}
```

### 完整示例

确保所有代码正确配置如下：

#### application.yml

```yaml
users:
  user1:
    name: Alice
    age: 30
  user2:
    name: Bob
    age: 25
```

#### User.java

```java
public class User {
    private String name;
    private int age;

    // Getters and setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

#### UserConfig.java

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

@Configuration
@ConfigurationProperties(prefix = "users")
public class UserConfig {
    private Map<String, User> map;

    // Getters and setters
    public Map<String, User> getMap() {
        return map;
    }

    public void setMap(Map<String, User> map) {
        this.map = map;
    }
}
```

#### DemoApplication.java

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;

@SpringBootApplication
@EnableConfigurationProperties(UserConfig.class)
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

#### UserPrinter.java

```java
import javax.annotation.PostConstruct;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class UserPrinter {
    private final UserConfig userConfig;

    @Autowired
    public UserPrinter(UserConfig userConfig) {
        this.userConfig = userConfig;
    }

    @PostConstruct
    public void printUsers() {
        if (userConfig.getMap() == null) {
            System.out.println("Map is null");
        } else {
            userConfig.getMap().forEach((key, user) -> {
                System.out.println("Key: " + key + ", User: " + user);
            });
        }
    }
}
```

### 注意事项

- 确保配置文件正确加载并解析。
- 确保配置类中的字段名称与配置文件中的名称匹配。
- 确保配置类被 Spring Boot 正确管理和注入。

通过上述方式，可以确保 `@ConfigurationProperties` 注解正确映射配置文件中的属性，并在应用启动时正确加载这些属性。如果配置项的命名不规范或未匹配，可能导致字段未正确映射为 `null`。

# 10. spring的nacos解析yml文件时报错，java.nio.charset.MalformedInputException: Input length = 1
这种错误通常表示在读取配置文件时遇到了字符编码问题。在解析 YAML 文件时，可能会遇到不支持的字符编码或不正确的字符编码格式。为了解决这个问题，你可以尝试以下方法：

1. **检查文件编码**：确保你的 YAML 配置文件使用的是支持的字符编码，比如 UTF-8。可以通过文本编辑器查看并确认文件的编码格式。

2. **转换文件编码**：如果确认文件编码不正确，可以使用文本编辑器将文件编码转换为正确的格式，比如将其转换为 UTF-8。

3. **使用正确的字符**：确保在 YAML 文件中使用的字符都是合法的，并且不包含不支持的字符或特殊字符。

4. **检查配置项**：在使用 Spring 的 Nacos 配置时，还要确保配置项的格式和内容符合要求，特别是对于特殊字符或格式要求严格的配置项。

5. **更新依赖**：检查项目中使用的 Spring Boot 和 Nacos 相关依赖是否是最新版本，有时更新到最新版本可以解决一些字符编码相关的问题。

如果你能提供更具体的错误信息或者相关的配置文件内容，我可以更详细地帮助你排查问题。

# 11. 你的意思是说我本地idea的编码不对，还是说nacos上面的文件编码不对？
我指的是你本地编码可能存在问题，也就是你使用的 IDE（比如 IntelliJ IDEA）打开的文件的编码格式可能不正确。这会导致在读取和解析文件时出现编码异常。

首先，你可以在 IntelliJ IDEA 中确认一下文件的编码格式，看看它是否与你的期望一致。可以通过以下步骤进行检查和设置：

1. 打开你的 YAML 配置文件。
2. 在 IDEA 的右下角或者顶部菜单栏的 File -> File Encoding 中查看当前的文件编码格式。
3. 确保文件编码格式设置为 UTF-8 或者与你的文件实际编码格式相符合。

如果你确认本地文件的编码格式正确，但在上传到 Nacos 后仍然出现相同的编码异常，那么可能是 Nacos 服务端或者网络传输过程中出现了编码问题。在这种情况下，你可能需要检查 Nacos 服务端的配置或者和 Nacos 交互的网络环境，确保数据传输过程中编码没有被意外修改或损坏。

# 12. String.format中使用%02d是什么意思？
在 `String.format` 中，`%02d` 表示格式化输出一个整数，其中：

- `%d` 表示整数的占位符；
- `2` 表示输出的整数至少占据两个位置（位数不足时在前面补0）；
- `0` 表示使用0作为补齐的字符。

例如，如果你有一个整数变量 `num` 的值为 `7`，使用 `%02d` 格式化输出这个整数时，结果将会是 `07`，因为数字 `7` 不足两位，所以在前面补上了一个 `0`。如果 `num` 的值是 `12`，则输出为 `12`，因为 `12` 已经占据了两个位置。

这种格式化方式常用于时间、日期等需要补齐位数的情况，确保输出的格式符合预期。
