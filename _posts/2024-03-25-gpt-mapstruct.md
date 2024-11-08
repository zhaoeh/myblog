---
layout:     post
title:      mapstruct
subtitle:   mapstruct如何使用
categories: [gpt]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. cglib和mapstruct原理上有什么区别呢？
CGLIB（Code Generation Library）和 MapStruct 是两种不同的 Java 库，尽管它们都与代码生成有关，但它们的用途和工作原理有很大的不同。

### CGLIB

**用途**: CGLIB 是一个高效的字节码生成库，主要用于动态代理和增强类。它通常用于 AOP（面向切面编程）、懒加载、动态类增强等场景。

**原理**:
1. **动态字节码生成**: CGLIB 通过 ASM 库在运行时生成目标类的子类，并在子类中添加所需的逻辑（如方法拦截）。
2. **子类代理**: 与 Java 标准库中的 `java.lang.reflect.Proxy` 不同，CGLIB 可以为任意类创建代理，而不仅限于接口。CGLIB 生成的代理类是目标类的子类，因此可以代理没有接口的类。
3. **MethodInterceptor**: CGLIB 使用 `MethodInterceptor` 接口来定义拦截逻辑。开发者实现这个接口中的 `intercept` 方法，当代理类的方法被调用时，这个 `intercept` 方法会被执行，从而实现方法拦截和增强。

**优缺点**:
- **优点**:
  - 可以代理没有接口的类。
  - 性能较好，因为生成的代理类是直接的字节码操作。
- **缺点**:
  - 不能代理 `final` 类和 `final` 方法。
  - 比基于接口的代理（如 JDK 动态代理）更复杂。

### MapStruct

**用途**: MapStruct 是一个代码生成器，用于在编译时生成类型安全的对象映射代码。它主要用于 DTO（数据传输对象）和实体类之间的映射。

**原理**:
1. **注解驱动**: 开发者通过在接口上使用 MapStruct 提供的注解（如 `@Mapper`）来定义映射关系。
2. **编译时代码生成**: MapStruct 在编译时（而非运行时）生成映射实现类。这个实现类包含了将一个对象转换为另一个对象的详细逻辑。
3. **静态代码生成**: MapStruct 使用注解处理器在编译时生成静态的、类型安全的映射代码。这意味着生成的代码在性能上与手动编写的映射代码无异。

**优缺点**:
- **优点**:
  - 高性能，因为映射逻辑在编译时生成，运行时无额外开销。
  - 类型安全，避免了运行时的类型转换错误。
  - 易于维护，注解驱动的映射配置简洁明了。
- **缺点**:
  - 不支持复杂的动态映射逻辑（需要使用其他工具或手动实现）。

### 对比总结

- **用途不同**:
  - CGLIB 用于动态代理和类增强，主要在 AOP 和类似场景中使用。
  - MapStruct 用于对象映射，主要在 DTO 与实体类之间的数据传输中使用。

- **工作时机不同**:
  - CGLIB 在运行时生成代理类。
  - MapStruct 在编译时生成映射代码。

- **实现方式不同**:
  - CGLIB 使用字节码操作生成代理类，并通过方法拦截实现功能增强。
  - MapStruct 使用注解处理器生成静态映射代码，完全在编译期完成。

- **应用场景不同**:
  - CGLIB 适用于需要动态代理和增强类的方法，如 AOP、懒加载等。
  - MapStruct 适用于需要高效、安全地在不同对象间进行数据映射的场景，如 DTO 和实体类的转换。

### 示例代码

#### CGLIB 示例

```java
public class SampleClass {
    public String test(String input) {
        return "Hello " + input;
    }
}

public class SampleInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("Before method: " + method.getName());
        Object result = proxy.invokeSuper(obj, args);
        System.out.println("After method: " + method.getName());
        return result;
    }
}

public class Main {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(SampleClass.class);
        enhancer.setCallback(new SampleInterceptor());
        
        SampleClass proxy = (SampleClass) enhancer.create();
        System.out.println(proxy.test("world"));
    }
}
```

#### MapStruct 示例

```java
@Mapper
public interface CarMapper {
    CarMapper INSTANCE = Mappers.getMapper(CarMapper.class);

    @Mapping(source = "numberOfSeats", target = "seatCount")
    CarDto carToCarDto(Car car);
}

public class Car {
    private int numberOfSeats;

    // getters and setters
}

public class CarDto {
    private int seatCount;

    // getters and setters
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.setNumberOfSeats(4);

        CarDto carDto = CarMapper.INSTANCE.carToCarDto(car);
        System.out.println("Car DTO seat count: " + carDto.getSeatCount());
    }
}
```

希望这些信息能帮助你理解 CGLIB 和 MapStruct 的原理和区别。如果有更多问题，欢迎随时提问！

# 2. 也就是说mapstruct实际上是介入java的编译器执行时机，在编译阶段生成了class文件；cglib是在java运行器执行阶段动态生成了class文件？
是的，你的理解是正确的。MapStruct 和 CGLIB 在生成代码的时机和机制上有明显的区别。

### MapStruct

- **时机**: 编译阶段
- **机制**: 使用 Java 的注解处理器在编译时生成映射代码。
- **过程**:
  - 开发者在代码中定义映射接口，并使用 MapStruct 的注解（如 `@Mapper`）。
  - 在编译时，MapStruct 的注解处理器会扫描这些注解，生成对应的实现类。
  - 生成的实现类会包含所需的映射逻辑，这些实现类在项目编译完成后成为项目的一部分。
  - 生成的代码是静态的、类型安全的，因此在运行时不会有额外的性能开销。

### CGLIB

- **时机**: 运行阶段
- **机制**: 通过字节码操作在运行时动态生成类。
- **过程**:
  - 当需要代理一个类时，CGLIB 使用字节码生成库（如 ASM）在运行时创建目标类的子类。
  - 这个子类会覆盖目标类的方法，并插入自定义的拦截逻辑（通过 `MethodInterceptor` 实现）。
  - 每次调用代理类的方法时，拦截逻辑会被执行，从而实现方法增强或其他动态行为。
  - 这种方式适合在需要动态改变行为的场景中使用，如 AOP（面向切面编程）中的方法拦截。

### 对比总结

- **MapStruct**:
  - **生成时机**: 编译阶段。
  - **生成方式**: 使用注解处理器生成静态映射代码。
  - **适用场景**: 需要高效、类型安全的对象映射，例如 DTO 和实体类之间的数据传输。
  - **性能**: 无运行时开销，编译时生成的代码在性能上与手动编写的代码无异。

- **CGLIB**:
  - **生成时机**: 运行阶段。
  - **生成方式**: 动态字节码生成，创建目标类的子类并进行方法拦截。
  - **适用场景**: 需要动态代理和类增强的场景，如 AOP 中的方法拦截、懒加载等。
  - **性能**: 有一定的运行时开销，因为需要在运行时生成字节码并进行方法拦截。

这两种技术虽然都是通过生成代码来实现特定功能，但它们的应用场景和生成时机大不相同，各有优劣。MapStruct 更适合编译时生成静态代码的场景，而 CGLIB 更适合运行时需要动态代理和增强功能的场景。

# 3. cglib在运行期间操纵字节码，是会在内存中生成class文件吗？
是的，CGLIB 在运行时操作字节码，会在内存中生成新的类（class），但这些类不会生成实际的 `.class` 文件存储在文件系统中，而是直接加载到 JVM 的内存中。

### 详细机制

1. **动态生成子类**:
   - CGLIB 通过 ASM 等字节码操作库，在运行时生成目标类的子类。这个子类会包含拦截逻辑，用于方法增强。
   
2. **字节码操作**:
   - 生成的字节码通过 `ClassLoader` 动态加载到 JVM 中。CGLIB 通常会使用自定义的 `ClassLoader` 来加载这些动态生成的类。
   
3. **方法拦截**:
   - 通过实现 `MethodInterceptor` 接口，开发者可以定义拦截逻辑。当代理类的方法被调用时，这个拦截逻辑会被执行。

4. **内存中的类**:
   - 动态生成的类存储在 JVM 的内存中，使用时从内存中直接调用。这避免了生成中间文件的开销和复杂性，但也意味着在 JVM 退出后，这些动态生成的类不会保留。

### 示例代码

以下是一个简单的示例，展示了 CGLIB 如何在运行时生成动态代理类：

```java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class SampleClass {
    public String test(String input) {
        return "Hello " + input;
    }
}

public class SampleInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("Before method: " + method.getName());
        Object result = proxy.invokeSuper(obj, args);
        System.out.println("After method: " + method.getName());
        return result;
    }
}

public class Main {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(SampleClass.class);
        enhancer.setCallback(new SampleInterceptor());
        
        SampleClass proxy = (SampleClass) enhancer.create();
        System.out.println(proxy.test("world"));
    }
}
```

在这个示例中：
1. **`SampleClass`** 是一个普通的 Java 类，包含一个 `test` 方法。
2. **`SampleInterceptor`** 实现了 `MethodInterceptor` 接口，定义了方法调用前后要执行的逻辑。
3. **`Enhancer`** 是 CGLIB 提供的一个类，用于生成动态代理类。它通过设置目标类（`SampleClass`）和拦截器（`SampleInterceptor`），在运行时生成一个代理类。
4. **代理类** 在内存中生成，当调用 `proxy.test("world")` 时，拦截器的逻辑会被执行。

### 内存管理

由于动态生成的类只存在于内存中，JVM 会负责管理这些类的生命周期。当这些类不再被引用时，JVM 的垃圾收集器（GC）会自动回收这些类所占用的内存。

总结来说，CGLIB 通过在运行时动态生成类，并将这些类加载到内存中，从而实现动态代理和方法增强的功能。这种方式避免了生成中间 `.class` 文件的复杂性，同时提供了灵活的运行时动态代理能力。
