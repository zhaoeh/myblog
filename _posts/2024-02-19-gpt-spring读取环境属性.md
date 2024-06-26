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

# 5. 
