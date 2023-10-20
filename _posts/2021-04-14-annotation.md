---
layout:     post
title:      玩一玩Java注解
subtitle:   各种框架都使用了java注解，我们研究一下它的原理
categories: [Java高级知识]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
    
# 玩转java注解  
关于注解，下面的几个问题你都会吗？  
1.  注解是干嘛的？  
2.  同一个注解可以使用很多次吗？如果可以，怎么使用？  
3.  @Inherited注解是干什么用的？  
4.  @Target注解中的“TYPE_PARAMETER”和“TYPE_USE”用在什么地方啊？  
5.  泛型你了解吧，泛型中如何使用注解？  
6.  注解的定义可以实现继承吗？  
7.  spring对注解有增强吗？@Aliasfor注解是干嘛的？  
    
# 1. java注解的定义和本质
我们都了解代码中的注释，注释是给开发人员看的，可能是开发者自己写的给自己看，也可能是给同组其他同事看的。  
代码中的注释可以提升代码的可读性和可维护性，但是对于java编译器和java虚拟机来说注释是没有意义的，编译后的class字节码文件中是没有注释信息的。  
注解和注释有点类似，唯一的区别就是注释是给人看的，而注解是给java编译器和java虚拟机看的。编译器和虚拟机在运行过程中可以获取注解信息，然后根据这些信息做各种想做的事。  
比如：@Override注解我们都了解。加载方法上，标注当前方法重写了父类的方法，当编译器编译代码时，会对@Override标注的方法进行验证，验证其父类中是否存在相同签名的方法，否则报错。  
<font color="#a52a2a">总结就是：注解本身是对代码的增强，可以在代码编译或者运行期间获取注解的信息，然后根据这些信息做各种牛掰的事。</font>  

## 1.1 注解的定义
1.  @Annotation是java中引入的注解机制。  
2.  注解功能是jdk1.5之后的jdk版本提供的。jdk内置三种注解，称为三种内置注解。    
3.  Annotation可以修饰接口、类、方法和属性等，而且Annotation不影响程序运行，无论是否使用Annotation程序都可以正常运行。  
4.  java.lang.annotation.Annotation是Annotation的接口，只要是定义的Annotation编译器都默认实现了该接口。  
5.  jdk内置的注解和自定义的注解编译器都默认实现了java.lang.annotation.Annotation接口。  

## 1.2 注解的本质
1.  可拆卸化的功能，可以标注在类级别、接口级别、成员级别、方法级别等。具体的注解要看具体的实现，注解本质类似于pojo，其中可以存放多个对象，每个对象可以是基本数据类型、String、Class、枚举类型、注解类型,也可以是这些类型对应的数组。其底层是一个容器，里面可以存放很多属性值（类似Person对象有name，有age），其中每一个属性值可以是一个简单值，也可以是个数组。  
2.  注解容器里面可以没有注解成员，也可以定义一个注解成员，也可以定义多个注解成员（每一个注解成员是基本数据类型、String、Class、枚举类型、注解类型或者其对应的的数组），具体看注解是如何实现的。  

# 2. jdk的3种内置注解
## 2.1 什么是内置注解？  
jdk1.5之后，内置了3种注解，用户直接使用即可。  
1. @Override：声明方法覆盖的Annotation。  
2. @Deprecated：不赞成使用的Annotation。  
3. @SuppressWarnings：压制安全警告的Annotation。  

## 2.2 详解jdk的3个内置注解
上述的3个内置注解是在java.lang包中定义的，java.lang包是自动导入的，因此可以直接使用。  
### 2.2.1 第1个：@Override
定义如下：   
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```
注意事项:  
（1）、使用范围：@Target(ElementType.METHOD)表示@Override注解只能在方法上标注，其他地方比如类、接口或者成员属性上是不能标注该注解的。  
（2）、作用：子类继承父类或者实现接口时，用来标注在子类覆盖或者实现的对一个方法上。目的是保证方法的覆盖是正确的（同名同参同返回），如果使用了这个注解，java编译器就会检查子类覆盖的方法是否和其父类中的方法具有相同的签名，一旦不符合方法覆盖负责，编译器将报错。  
（3）、@Retention(RetentionPolicy.SOURCE)表示该注解只在java源文件阶段有效。  
### 2.2.2 第2个：@Deprecated
定义如下：  
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.CONSTRUCTOR, ElementType.FIELD, ElementType.LOCAL_VARIABLE, ElementType.METHOD, ElementType.PACKAGE, ElementType.PARAMETER, ElementType.TYPE})
public @interface Deprecated {
}
```
注意事项：  
（1）、使用范围：@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})表示@Deprecated注解可以标注在构造器、成员属性、本地变量、方法、包、参数、类|接口|枚举|注解上。  
（2）、作用：@Deprecated注解用来标注一个不建议使用的构造器、成员属性、本地变量、方法、包、参数、类|接口|枚举|注解等，如果使用了这些，编译时将会出现警告（注意不是编译报错，仅仅是警告而已）。  
（3）、@Retention(RetentionPolicy.RUNTIME)表示该注解在java源文件阶段、字节码阶段和jvm运行阶段都有效。  
### 2.2.3 第3个：@SuppressWarnings
定义如下：  
```java
@Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.CONSTRUCTOR, ElementType.LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```
注意事项:  
（1）、使用范围：@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})表示@SuppressWarnings注解可以标注在类|接口|枚举|注解、成员属性、参数、构造器、本地变量上。  
（2）、作用：@SuppressWarnings注解需要传递压制参数，如果需要压制一条警告信息，则传递一个字符串即可；如果需要压制多条警告信息，则传递一个字符串数组即可。作用是压制调用方调用被调用方时编译器提示的警告信息。  
（3）、@Retention(RetentionPolicy.SOURCE)表示该注解只在java源文件阶段有效。  
（4）、@SuppressWarnings注解定义的成员可以接收的参数：  
    deprecation：使用了不赞成使用的类或方法时的警告  
    unchecked：执行了未检查的转换时警告，比如，泛型操作中没有指定泛型类型  
    fallthrough：当使用switch操作时，case后未加入break操作，而导致程序继续执行其他case语句时出现的警告  
    path：当设置了一个错误的类路径、源文件路径时出现的警告  
    serial：当在可序列化的类上缺少serialVersionUID定义时出现的警告  
    finally：任何finally子句不能正常完成时出现的警告  
    all：关于以上所有情况的警告  
  
# 3. 元注解-标注注解的注解
## 3.1 什么是元注解？
不论是jdk的内置注解还是自定义的注解，jdk在声明注解时同样提供了其他的注解来标注当前声明的注解，这些用来标注注解的注解称为元注解。  
元注解的目的是规范一个注解的使用限制。  
元注解只能在声明注解的时候用来标注注解，而不能标注普通类和方法、属性等。  
我们始终记住，元注解是用来标注注解的注解，不能用来标注其他目标。  

## 3.2 jdk提供的5种元注解
### 3.2.1 第1个：@Documented，使注解包含在javadoc中
1.定义如下：
 ```java
package java.lang.annotation;
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Documented {
}
```
2.注解成员：  
该元注解没有注解成员。用该元注解标注的注解将被包含在javadoc中。
### 3.2.2 第2个：@Retention(RetentionPolicy.XXX)，指定注解的保留策略
首先先解下java程序的3个过程：  
```
源码阶段  
源码被编译为字节码之后变成class文件  
字节码被虚拟机加载然后执行 
``` 
那么自定义的注解会被保留在上面哪个阶段呢？可以通过@Retention指定。  
1.定义如下：  
```java
package java.lang.annotation;
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Retention {
    RetentionPolicy value();
}
```
2.注解成员：  
该元注解定义的成员是RetentionPolicy类型的一个对象，并且没有指定默认值，意味着使用该元注解时必须设置注解参数值。  
RetentionPolicy是一个枚举，定义如下：  
```java
package java.lang.annotation;
public enum RetentionPolicy {
    SOURCE,
    CLASS,
    RUNTIME;
    private RetentionPolicy() {
    }
}
```
3.注解成员含义：  
RetentionPolicy.SOURCE：该元注解标注的注解，去标注其他目标时，其有效范围只保存在程序源文件*.java中，编译后的字节码文件*.class中将不再保留注解信息，即在编译后直接将失效，所以一般不用。  
RetentionPolicy.CLASS：该元注解标注的注解，去标注其他目标时，其有效范围保存在源文件*.java和编译后的字节码文件*.class中，随着class文件被JVM装载进内存后，注解信息将失效。一般也不使用，如果一个Annotation在声明时没有指定有效范围，则默认是此范围。  
RetentionPolicy.RUNTIME：该元注解标注的注解，去标注其他目标时，其有效范围保存在源文件*.java、编译后的字节码文件*.class和JVM内存中，一般使用注解就是为了在JVM运行中获取到注解信息然后做各种牛逼的事情，所以一般使用这个有效范围。  
<font color="#a52a2a">正因为SOURCE和CLASS无法保留到JVM中，因此实用性不强，这2者一般都是java编译器和java虚拟机负责解析的。一般自定义注解都设置为RUNTIME，否则通过发射无法获取注解信息。</font>  

4.使用  
```java
@Retention(RetentionPolicy.SOURCE)
public @interface MyAnnotation{
}
```
上面指定了<font color="#a52a2a">MyAnnotation</font>注解只保留到源码阶段，后面的2个阶段注解会丢失。  

### 3.2.3 第3个：@Target({ElementType.XXX,ElementType.XXX})，指定注解的使用范围
1.定义如下：  
```java
package java.lang.annotation;
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Target {
    ElementType[] value();
}
```
2.注解成员：  
该元注解定义的成员是ElementType类型的一个对象数组，并且没有指定默认值，意味着使用该元注解时必须设置注解参数值。  
ElementType是一个枚举，定义如下：  
```java
package java.lang.annotation;
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,
    /** Field declaration (includes enum constants) */
    FIELD,
    /** Method declaration */
    METHOD,
    /** Formal parameter declaration */
    PARAMETER,
    /** Constructor declaration */
    CONSTRUCTOR,
    /** Local variable declaration */
    LOCAL_VARIABLE,
    /** Annotation type declaration */
    ANNOTATION_TYPE,
    /** Package declaration */
    PACKAGE,
    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,
    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```
3.注解成员含义：  
ElementType.TYPE：作用于接口、类、枚举、注解上。  
ElementType.FIELD：作用于成员变量、枚举常量上。  
ElementType.METHOD：作用于方法上。  
ElementType.PARAMETER：作用于方法参数上。  
ElementType.CONSTRUCTOR：作用于构造方法上。  
ElementType.LOCAL_VARIABLE：作用于本地方法变量上。  
ElementType.ANNOTATION_TYPE：作用于注解上。  
ElementType.PACKAGE：作用于包上。  
ElementType.TYPE_PARAMETER：作用于泛型变量类型上，注意只能作用于声明的泛型变量上，必须在<>里面的泛型变量前标注。  
ElementType.TYPE_USE：作用于泛型类型的实际类型上，泛型类型的实际类型可能是任何类型，言外之意只能作用于<>里面的实际类型前。  

4.使用  
```java
@Target({ElementType.TYPE,ElementType.METHOD})
public @interface MyAnnotation{
}
```    
如上表示自定义的注解MyAnnotation只允许标注在接口、类、枚举、注解和方法上。  
<font color="#a52a2a">如果自定义一个注解的时候，没有使用@Target元注解指定标注目标，则声明后的注解默认可以在任何目标上使用（上述第3点的所有目标位置）。</font>  

### 3.2.4 第4个：@Inherited，实现类之间的注解继承
1.定义如下：
```java
package java.lang.annotation;
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Inherited {
}
```

2.注解成员：  
该元注解没有注解成员。  
用该元注解标注的注解，在后续标注普通类等目标时，如果该类有子类，则其子类可以继承父类的注解。即如果一个自定义注解想要被标注的类的子类继承，则自定义注解应该使用@Inherited元注解来标注。  
<font color="#a52a2a">注意，@Inherited 在父类中标注的注解可以被子类继承，如果接口中也使用了@Inherited 标注的注解，接口的实现类是无法继承这个注解的。</font>  

3.使用
```java
@Inherited
public @interface MyAnnotation{
}
@MyAnnotation
class A{
}
class B extends A{
}
```
如上表示，B类会自动继承A类上的@MyAnnotation注解，因为该注解使用了@Inherited标注。  

### 3.2.5 第5个：@Repeatable(Annotation或其实现类.class)，让目标注解可以重复使用
1. 定义如下：  
```java
package java.lang.annotation;
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Repeatable {
    Class<? extends Annotation> value();
}
```
2.注解成员：  
该元注解定义的成员是一个实际类型必须是Annotation或者其实现类的class对象，说白了就是一个注解的class对象，并且没有指定默认值，意味着使用该元注解时必须设置注解参数值。  
  
3.注解成员含义：  
@Repeatable的注解成员是一个容器注解。容器注解也是一个注解，其中的成员是一个注解类型的数组，用来保存一堆注解。  

4.使用  
要想重复使用目标注解，按照如下步骤进行：  
```
>1.定义一个容器注解
>2.定义目标注解，使用@Repeatable标准，@Repeatable中的value值是上面定义的容器注解
>3.对目标元素重复标注目标注解
```
重复使用注解的详细过程请参考下面的案例！！！  

# 4. 如何使用注解？
3个步骤：  
1. 定义注解（声明注解）：注解需要先定义，定义之后才能够使用（jdk内置注解相当于已经定义好了可以直接拿来使用）。  
2. 标注注解：在需要使用注解的地方，使用<font color="gray">@注解名称(注解参数，如果有参数的话)</font>来标注注解。  
3. 获取注解信息做各种牛逼plus的事情：注解要想生效，则需要有一个组件通过反射机制去获取标注在某个目标上的注解，获取注解信息后进行具体的业务逻辑处理（jdk内置的注解已经实现了反射去调用了，自定义的注解要想生效则必须自己通过反射去调用）。  

注意：自定义的注解，单纯定义了注解并且在指定目标上标注了注解并不能使注解生效，必须使用反射进行注解的获取才能决定标注该注解后应该执行的业务逻辑。  
对于jdk和其他框架的注解，内部已经通过反射去获取指定目标上的注解了。我们只需要在指定目标上标注注解即可让注解生效。  

# 5. 自定义注解
## 5.1 如何自定义注解？
自定义注解需要自己实现注解的定义，定义完注解之后才能标注注解。  
java中注解相关的类和接口都定义在<font color="#a52a2a">java.lang.annotation</font> 包中。  
注解语法：  
```
[public] @interface Annotation名称{   // 如果一个java源文件已经存在public修饰的类，则此处就不需要加public
    [[public] 数据类型 成员变量名称1() [default 成员默认值];]  // 如果注解不需要定义成员参数，则此处不用定义成员变量
    [[public] 数据类型 成员变量名称2() [default 成员默认值];]
    [[public] 数据类型 成员变量名称3() [default 成员默认值];]
}
```
注解中有没有注解成员都可以，也可以定义多个注解成员，注解成员也称为注解参数。    
1.  只要使用@interface声明注解，那么该注解在编译后默认继承了java.lang.annotation.Annotation接口。    
2.  注解使用@interface进行声明，但是在编译后这个直接实际上是个接口，里面的注解成员实际上是个抽象方法。    
3.  在定义注解时，可以定义各种成员变量，但是注意变量后面必须有()。()不是定义方法参数，也不能在括号里定义任何参数，仅仅是个特殊的语法。    
4.  注解成员的访问修饰符必须是public，不写默认是public。    
5.  注解成员的类型只能是基本数据类型、String、Class、枚举类型、注解类型（体现了注解的嵌套效果）以及上述类型对应的数组类型。    
6.  注解成员的命名一般为名词，如果注解中只定义了一个注解成员，请将其命名为value，当注解中只有一个名称为value的成员时，在标注注解时可以省略参数名称，直接设置其值。      
7.  <font color="#a52a2a">default</font>表示默认值，值类型必须和相应的注解成员的类型匹配。      
8.  如果某个注解成员没有指定默认值，代表后续使用该注解标注时必须给该注解参数设置值。  

如下定义了具有一个注解成员的注解，其默认值是“zhangsan”：  
```java
public @interface MyAnnotation{
    public String name() default "zhangsan";
}
```

## 5.2 自定义注解并使用jad进行反编译
<b>自定义注解如下： </b> 
```java
package zeh.test.demo.com.class1;

@interface MyAnnocation {
    String value() default "100";
}
```
<b>使用jad反编译如下：</b>
```java
package zeh.test.demo.com.class1;
import java.lang.annotation.Annotation;
interface MyAnnocation
    extends Annotation
{
    public abstract String value();
}
```

# 6. 自定义注解案例
## 6.1 案例1：声明一个不接受任何注解参数的注解
<b>定义注解：</b>
```java
package com.zeh.main.annotation.myannotation;
// 自定义注解不使用@Retention(RetentionPolicy.XXX)元注解，则默认是@Retention(RetentionPolicy.CLASS)
// 如果不使用@Target(ElementType.XXX)元注解，则默认可以标注在任何目标上
public @interface MyDefaultAnnotationNonePram {

    // 自定义注解，不定义任何注解成员，以为这不接受任何参数。
    // 要想使用这个自定义注解，需要定义其他的类或者接口等目标，在其类、接口、枚举、成员变量或者成员方法等目标上标注该注解，并且通过反射去获取注解信息然后做各种吊炸天的事情。
}
```

<b>标注注解：</b>
```java
package com.zeh.main.annotation.mytarget;
import com.zeh.main.annotation.myannotation.MyDefaultAnnotationNonePram;
// 只是在该处标注注解，并没有什么卵用，要想真正使用注解，比如反射获取注解信息才能做后续处理
@MyDefaultAnnotationNonePram
public class AnnotationNonePram {
}
```

## 6.2 案例2：声明一个接受一个注解参数的注解
<b>定义注解：</b>
```java
package com.zeh.main.annotation.myannotation;

public @interface MyDefaultAnnotationOneParam {
    // 定义一个String类型的注解成员变量来接受字符串类型的注解参数
    public String value();
}
```

<b>标注注解：</b>
```java
package com.zeh.main.annotation.mytarget;
import com.zeh.main.annotation.myannotation.MyDefaultAnnotationOneParam;
// 对于自定义注解有且只有一个成员并且成员名称为value，则可以直接设置成员值。
// @MyDefaultAnnotationOneParam("eric")
// 当然也可以显式指定成员的名称通过 key = value 形式进行赋值。比如 value =
@MyDefaultAnnotationOneParam(value = "eric")
public class AnnotationOneParam {
}
```

## 6.3 案例3：声明一个接受多个注解参数的注解
<b>定义注解：</b>
```java
package com.zeh.main.annotation.myannotation;

public @interface MyDefaultAnnotationMoreParam {
    // 定义一个String类型的注解成员变量接受字符串参数
    public String key();
    // 定义一个String类型的注解成员变量接受字符串参数
    public String value();
}
```

<b>标注注解：</b>
```java
package com.zeh.main.annotation.mytarget;
import com.zeh.main.annotation.myannotation.MyDefaultAnnotationMoreParam;
// 当自定义注解有多个成员定义时，不论其中是否有value命名的成员，都必须显式使用 key = value 的形式进行注解参数赋值（如果其中其他不叫value的成员都有默认值的话，只设置value值时也可直接设置）
@MyDefaultAnnotationMoreParam(key = "name", value = "eric")
public class AnnotationMoreParam {
}
```

## 6.4 案例4：声明一个接受一个注解参数的注解，但是这个注解参数是个数组容器
<b>定义注解：</b>
```java
package com.zeh.main.annotation.myannotation;

public @interface MyDefaultAnnotationOneParamArray {
    // 定义一个String数组类型的注解成员变量用于接收一堆String类型的注解参数。
    String[] value();
}
```

<b>标注注解：</b>
```java
package com.zeh.main.annotation.mytarget;
import com.zeh.main.annotation.myannotation.MyDefaultAnnotationOneParamArray;
// 当自定义注解中有且仅有一个注解成员且其名称为value时（或者有多个注解成员，但是其他成员都有默认值，则单独设置value值时也可以直接设置），可以直接设置值，也可以通过key=value的形式设置值。
// 当其成员是数组时，如果数组只有一个元素，则可以省略{}；如果有多个元素，则需要添加{}
// @MyDefaultAnnotationOneParamArray({"eric","daisy"})
// @MyDefaultAnnotationOneParamArray("zhangsan")
// @MyDefaultAnnotationOneParamArray(value = "lisi")
// @MyDefaultAnnotationOneParamArray(value = {"zhangsan"})
@MyDefaultAnnotationOneParamArray(value = {"zhangsan", "lisi"})
public class AnnotationOneParamArray {
}
```

## 6.5 案例5：声明一个接受多个注解参数的注解，且声明时注解参数指定默认值
<b>定义注解：</b>
```java
package com.zeh.main.annotation.myannotation;

public @interface MyDefaultAnnotationMoreParamDefaultValue {
    // 定义注解成员同时指定注解成员的默认值，如果不指定默认值则标注注解时必须显式指定注解参数值，否则编译报错；如果此处指定了默认值，则标注注解时可以传递实际参数值，这时取此处的默认值
    String name1() default "eric";
    String name2() default "daisy";
    // 数组类型设置默认值使用{}
    String name3()[] default {"A", "B"};
    // 如果数组类型默认值只有一个的话则省略{}
    int[] score() default 99;
}
```
<b>标注注解：</b>
```java
package com.zeh.main.annotation.mytarget;
import com.zeh.main.annotation.myannotation.MyDefaultAnnotationMoreParamDefaultValue;
// 自定义注解有默认值的成员，可以不用设置值，此时取的是默认值
// @MyDefaultAnnotationMoreParamDefaultValue
// 也可以对其中部分注解参数进行设置，此时取的是设置进去的值
@MyDefaultAnnotationMoreParamDefaultValue(name1 = "zhangsan", name2 = "lisi")
public class AnnotationMoreParamDefaultValue {
}
```

## 6.6 案例6：声明一个接受多个注解参数的注解，且使用枚举限制注解参数的取值范围
<b>定义枚举：</b>
```java
package com.zeh.main.annotation.myenum;

// 2021-05-24
public enum MyEnum {
    // 自定义一个枚举类，保存如下枚举常量
    // 每个枚举常量默认是public的、static的、final的
    // 每个枚举常量都是当前枚举类的实例对象
    ERIC, DAISY, ZHAOEH;
}
```

<b>定义注解：</b>
```java
package com.zeh.main.annotation.myannotation;
import com.zeh.main.annotation.myenum.MyEnum;
public @interface MyDefaultAnnotationMoreParamUseEnum {
    // 定义两个MyEnum类型的注解成员接收注解参数，则外部传入注解参数的取值范围只能是MyEnum枚举类型的那几个枚举常量。
    MyEnum name1();
    // 限制name2成员的取值只能是MyEnum类型，即只能是MyEnum枚举类的几个枚举常量值，并且name2指定了默认取值为MyEnum枚举的枚举常量对象DAISY。
    MyEnum name2() default MyEnum.DAISY;
}
```

<b>标注注解：</b>
```java
package com.zeh.main.annotation.mytarget;
import com.zeh.main.annotation.myannotation.MyDefaultAnnotationMoreParamUseEnum;
import com.zeh.main.annotation.myenum.MyEnum;
// name1和name2都得从MyEnum枚举常量中取值，name2有默认值所以此处可以不进行设置
@MyDefaultAnnotationMoreParamUseEnum(name1 = MyEnum.ERIC)
public class AnnotationMoreParamUseEnum {
}
```

## 6.7 案例7：声明一个接受多个注解参数的注解，且在声明注解时使用元注解（先使用4种，重复注解下个案例看）
<b>定义注解：</b>
```java
package com.zeh.main.annotation.myannotation;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

// 该元注解没有注解参数
// 使用该元注解标注自定义注解表示自定义注解将会被保存在javadoc中
@Documented
// 该元注解有注解参数，在标注时必须设置注解参数
// 使用该元注解标注自定义注解表示自定义注解的有效保留范围
// 可以是源文件、class字节码或者JVM内存中
// 如果不使用该注解，则默认保存范围是CLASS
@Retention(RetentionPolicy.RUNTIME)
// 该元注解有注解参数，在标注时比如设置注解参数
// 使用该元注解标注自定义注解表示自定义注解的有效标注目标
// 可以是类、接口、注解、方法、成员、本地变量等
// 如果不使用该元注解标注自定义注解，则默认自定义注解可以在任何目标上进行标注
@Target({ElementType.TYPE,ElementType.METHOD})
// 该元注解没有注解参数
// 使用该元注解标注自定义注解表示自定义注解标注父类后，该父类的后续子类都会默认继承该自定义注解
@Inherited
public @interface MyDefaultAnnotationMoreParamMeta {
    String name1();
    String name2();
}
```

<b>标注注解：</b>
```java
package com.zeh.main.annotation.mytarget;
import com.zeh.main.annotation.myannotation.MyDefaultAnnotationMoreParamMeta;
// 多个注解参数设置值，只能使用key = value的形式进行设置
@MyDefaultAnnotationMoreParamMeta(name1 = "daisy", name2 = "eric")
public class AnnotationMoreParamMeta {
}
```

## 6.8 案例8：注解重复使用，元注解@Repeatable
<b>第1步：定义容器注解MyDefaultAnnotationRepeats</b>
```java
package com.zeh.main.annotation.myannotation;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

// 定义一个容器注解
@Documented
@Target({ElementType.TYPE, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyDefaultAnnotationRepeats {
    // 该容器注解成员表示：保存一堆MyDefaultAnnotationRepeat注解。
    MyDefaultAnnotationRepeat[] value();
}
```

<b>第2步：定义目标注解MyDefaultAnnotationRepeat</b>
```java
package com.zeh.main.annotation.myannotation;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

// 定义一个要使用的目标注解，并为当前注解通过@Repeatable指定容器注解
// 要让当前注解可以重复使用，需在当前注解上加上@Repeatable注解，@Repeatable中的value值是自定义的容器注解。
@Repeatable(MyDefaultAnnotationRepeats.class)
@Documented
@Target({ElementType.TYPE, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyDefaultAnnotationRepeat {
    String name();
}
```

<b>第3步：在目标AnnotationRepeat上重复标注MyDefaultAnnotationRepeat注解</b>
```java
package com.zeh.main.annotation.mytarget;
import com.zeh.main.annotation.myannotation.MyDefaultAnnotationRepeat;
import com.zeh.main.annotation.myannotation.MyDefaultAnnotationRepeats;
// 重复标注注解方式1：直接在目标上标注相同的注解多次
@MyDefaultAnnotationRepeat(name = "zhangsan")
@MyDefaultAnnotationRepeat(name = "zhangsan")
public class AnnotationRepeat {
    // 重复标注注解方式2：在目标上标注容器注解，容器注解的成员设置一个目标注解数组进去
    @MyDefaultAnnotationRepeats({@MyDefaultAnnotationRepeat(name = "wangwu"),@MyDefaultAnnotationRepeat(name = "lisi")})
    private String field;
}
```

# 7. 注解信息的获取
## 7.1 如何获取注解？
上面的案例都是使用注解3步骤中的前2步：定义注解、标注注解。  
而使用注解的关键在第3步，即获取注解信息然后做各种牛逼的事儿。  
为了运行时能够准确的获取注解信息，java在<font color="#a52a2a">java.lang.reflect</font>反射包下新增了AnnotatedElement接口，它主要用于表示目前正在虚拟机运行的程序中已经使用的注解的元素。  
通过该接口可以利用反射技术读取注解信息。  

<b>AnnotatedElement接口的子接口和子类有：</b>  
```
Package：表示包的信息。
Class：表示类的信息。
Constructor：表示构造方法信息。
Field：表示类中属性信息。
Method：表示方法信息。
Parameter：表示方法参数信息。
TypeVariable：表示泛型变量类型信息，比如：类上声明的泛型类型变量，方法上声明的泛型类型变量。
```

<b>AnnotatedElement接口常用方法：</b>
```java
<A extends Annotation> getAnnotation(Class<A> annotationClass)：目标元素如果存在指定注解，则返回指定注解，否则返回null。
Annotation[] getAnnotations()：返回目标元素上的所有注解，包括从父类继承的。
boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)：如果指定注解存在于目标元素上，则返回true，否则返回false。
Annotation[] getDeclaredAnnotations()：返回直接存在于目标元素上的所有注解，注意，不包括从父类继承的注解，调用者可以随意修改返回的注解数组，这不会对其他调用者返回的注解数组产生任何影响，没有则返回长度为0的空数组。
```

## 7.2 综合案例：完整使用注解
<b>定义需要标注的注解MyDefaultAnnotationReflect1</b>
```java
package com.zeh.main.annotation.myannotation.all;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE,
        ElementType.FIELD,
        ElementType.METHOD,
        ElementType.PARAMETER,
        ElementType.CONSTRUCTOR,
        ElementType.LOCAL_VARIABLE,
        ElementType.ANNOTATION_TYPE,
        ElementType.PACKAGE,
        ElementType.TYPE_PARAMETER,
        ElementType.TYPE_USE})
@Inherited
// 该注解可以重复使用，其依赖的容器注解是 MyDefaultAnnotationReflectRepeats
@Repeatable(MyDefaultAnnotationReflectRepeats.class)
public @interface MyDefaultAnnotationReflect1 {
    String value();
}
```

<b>为上面的注解定义容器注解MyDefaultAnnotationReflectRepeats</b>
```java
package com.zeh.main.annotation.myannotation.all;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
// 综合案例：容器注解
@Documented
// 容器注解的使用范围应该和其托管的注解的范围一致，否则如果目标注解可以标注在方法参数上，而容器注解不能标注的话，那么目标注解是无法在方法参数上重复标注的
@Target({ElementType.TYPE,
        ElementType.FIELD,
        ElementType.METHOD,
        ElementType.PARAMETER,
        ElementType.CONSTRUCTOR,
        ElementType.LOCAL_VARIABLE,
        ElementType.ANNOTATION_TYPE,
        ElementType.PACKAGE,
        ElementType.TYPE_PARAMETER,
        ElementType.TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface MyDefaultAnnotationReflectRepeats {
    // 使用容器注解托管自定义注解，便可以重复使用自定义注解了
    MyDefaultAnnotationReflect1[] value();
}
```

<b>再定义一个需要使用的注解MyDefaultAnnotationReflect1，该注解不重复标注</b>
```java
package com.zeh.main.annotation.myannotation.all;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE,
        ElementType.FIELD,
        ElementType.METHOD,
        ElementType.PARAMETER,
        ElementType.CONSTRUCTOR,
        ElementType.LOCAL_VARIABLE,
        ElementType.ANNOTATION_TYPE,
        ElementType.PACKAGE,
        ElementType.TYPE_PARAMETER,
        ElementType.TYPE_USE})
@Inherited
@MyDefaultAnnotationReflect1("标注在注解上；对应ElementType.ANNOTATION_TYPE")
public @interface MyDefaultAnnotationReflect1_1 {
    int value();
}
```
<b>上面定义了两个注解MyDefaultAnnotationReflect1和MyDefaultAnnotationReflect1_1，其中MyDefaultAnnotationReflect1使用了容器注解MyDefaultAnnotationReflectRepeats，因此它可以重复标注。</b>

<b>定义一个类AnnotationReflect，标注上述注解</b>
```java
package com.zeh.main.annotation.mytarget.all;
import com.zeh.main.annotation.myannotation.all.MyDefaultAnnotationReflect1;
import com.zeh.main.annotation.myannotation.all.MyDefaultAnnotationReflect1_1;
import java.util.Map;
@MyDefaultAnnotationReflect1("标注在类上；对应ElementType.TYPE")
@MyDefaultAnnotationReflect1("标注在类上；重复标注")
@MyDefaultAnnotationReflect1_1(1)
public class AnnotationReflect<
        @MyDefaultAnnotationReflect1(value = "标注在泛型变量类型T1上，T1是在类上声明的一个泛型变量类型；对应ElementType.TYPE_PARAMETER")
                @MyDefaultAnnotationReflect1(value = "标注在泛型变量类型T1上，重复标注")
                @MyDefaultAnnotationReflect1_1(2) T1,
        @MyDefaultAnnotationReflect1(value = "标注在泛型变量类型T2上，T2是在类上声明的一个泛型变量类型；对应ElementType.TYPE_PARAMETER")
                @MyDefaultAnnotationReflect1(value = "标注在泛型变量类型T2上，重复标注")
                @MyDefaultAnnotationReflect1_1(3) T2> {
    @MyDefaultAnnotationReflect1("标注在成员属性上；对应ElementType.FIELD")
    @MyDefaultAnnotationReflect1("标注在成员属性上；重复标注")
    @MyDefaultAnnotationReflect1_1(4)
    private String name;
    private Map<@MyDefaultAnnotationReflect1("标注在泛型类型的实际类型上；对应ElementType.TYPE_USE")
    @MyDefaultAnnotationReflect1("标注在泛型类型的实际类型上；重复标注") String,
            @MyDefaultAnnotationReflect1_1(5) ?> map;
    @MyDefaultAnnotationReflect1("标注在构造方法上；对应ElementType.CONSTRUCTOR")
    @MyDefaultAnnotationReflect1("标注在构造方法上；重复标注")
    @MyDefaultAnnotationReflect1_1(10)
    public AnnotationReflect() {
    }
    @MyDefaultAnnotationReflect1("标注在方法method1上；对应ElementType.METHOD")
    @MyDefaultAnnotationReflect1("标注在方法method1上；重复标注")
    @MyDefaultAnnotationReflect1_1(78)
    public <@MyDefaultAnnotationReflect1("标注在泛型变量类型T3上，T3是在类上声明的一个泛型变量类型；对应ElementType.TYPE_PARAMETER")
            @MyDefaultAnnotationReflect1("标注在泛型变量类型T3上，重复标注")
            @MyDefaultAnnotationReflect1_1(12) T3,
            @MyDefaultAnnotationReflect1("标注在泛型变量类型T4上，T4是在类上声明的一个泛型变量类型；对应ElementType.TYPE_PARAMETER")
                    @MyDefaultAnnotationReflect1("标注在泛型变量类型T4上，重复标注")
                    @MyDefaultAnnotationReflect1_1(18) T4>
    T3 method1(@MyDefaultAnnotationReflect1("标注在方法参数上；对应ElementType.PARAMETER")
               @MyDefaultAnnotationReflect1("标注在方法参数上；重复标注")
               @MyDefaultAnnotationReflect1_1(0) T3 t3,
               @MyDefaultAnnotationReflect1("标注在方法参数上；对应ElementType.PARAMETER")
               @MyDefaultAnnotationReflect1("标注在方法参数上；重复标注")
               @MyDefaultAnnotationReflect1_1(19) T4 t4) {
        return t3;
    }
}
```
<b>获取注解 TestMyAnnotationReflect</b>
```java
package com.zeh.main.annotation.test;
import com.zeh.main.annotation.myannotation.all.MyDefaultAnnotationReflect1;
import com.zeh.main.annotation.myannotation.all.MyDefaultAnnotationReflect1_1;
import com.zeh.main.annotation.mytarget.all.AnnotationReflect;
import org.junit.Test;
import java.lang.annotation.Annotation;
import java.lang.reflect.AnnotatedParameterizedType;
import java.lang.reflect.AnnotatedType;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.Parameter;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.lang.reflect.TypeVariable;

// 解析注解做各种牛逼的事儿
// 对于Jdk和其他框架实现好的注解，只需要在指定的目标上标注即可，不用关注如何解析。因为jdk内部和框架底层已经实现了通过反射去自动扫描注解并获取注解信息进行后续的逻辑处理。
public class TestMyAnnotationReflect {

    // 解析标注在类上的注解：对应ElementType.TYPE
    @Test
    public void getAnnotationFromClass() {
        Class&lt;AnnotationReflect> annotationReflectClass = AnnotationReflect.class;
        System.out.println("标注在" + annotationReflectClass.getName() + "类上的注解如下：");
        Annotation[] annotations = annotationReflectClass.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation);
        }
    }
    
    // 解析标注在成员上的注解：对应ElementType.FIELD
    @Test
    public void getAnnotationFromField() throws NoSuchFieldException {
        Field nameField = AnnotationReflect.class.getDeclaredField("name");
        System.out.println("标注在" + nameField.getName() + "属性上的注解如下：");
        Annotation[] annotations = nameField.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println("annotation is :" + annotation);
            System.out.println("当前 annotation 的类型：" + annotation.getClass());
            if (annotation instanceof MyDefaultAnnotationReflect1) {
                // 如果当前注解是 MyDefaultAnnotationReflect1 类型，则强转后获取注解中的属性值
                MyDefaultAnnotationReflect1 myDefaultAnnotationReflect = (MyDefaultAnnotationReflect1) annotation;
                System.out.println("该注解的value：" + myDefaultAnnotationReflect.value());
            }
            if (annotation instanceof MyDefaultAnnotationReflect1_1) {
                // 如果当前注解是 MyDefaultAnnotationReflect1_1 类型，则强转后获取注解中的属性值
                MyDefaultAnnotationReflect1_1 myDefaultAnnotationReflect1_1 = (MyDefaultAnnotationReflect1_1) annotation;
                System.out.println("该注解的value：" + myDefaultAnnotationReflect1_1.value());
            }
        }
    }

    // 解析标注在构造方法上的注解：对应ElementType.CONSTRUCTOR
    @Test
    public void getAnnotationFromConstructor() throws NoSuchMethodException {
        // 反射获取无参构造器
        Constructor&lt;AnnotationReflect> annotationReflectConstructor = AnnotationReflect.class.getConstructor();
        // 手动获取构造方法上的指定注解。
        System.out.println("标注在" + annotationReflectConstructor.getName() + "构造方法上的注解如下：");
        // 如果当前构造器上存在 MyDefaultAnnotationReflect1 注解标注的话
        if (annotationReflectConstructor.isAnnotationPresent(MyDefaultAnnotationReflect1.class)) {
            // 获取构造器上 MyDefaultAnnotationReflect1 类型的注解
            MyDefaultAnnotationReflect1 myDefaultAnnotationReflect1 = annotationReflectConstructor.getAnnotation(MyDefaultAnnotationReflect1.class);
            System.out.println("annotation is :" + myDefaultAnnotationReflect1);
            System.out.println("当前 annotation 的类型：" + myDefaultAnnotationReflect1.getClass());
            System.out.println("该注解的value：" + myDefaultAnnotationReflect1.value());
        }
        // 如果当前构造器上存在 MyDefaultAnnotationReflect1_1 注解标注的话
        if (annotationReflectConstructor.isAnnotationPresent(MyDefaultAnnotationReflect1_1.class)) {
            // 获取构造器上 MyDefaultAnnotationReflect1_1 类型的注解
            MyDefaultAnnotationReflect1_1 myDefaultAnnotationReflect1_1 = annotationReflectConstructor.getAnnotation(MyDefaultAnnotationReflect1_1.class);
            System.out.println("annotation is :" + myDefaultAnnotationReflect1_1);
            System.out.println("当前 annotation 的类型：" + myDefaultAnnotationReflect1_1.getClass());
            System.out.println("该注解的value：" + myDefaultAnnotationReflect1_1.value());
        }
    }

    // 解析标注在类上声明的泛型变量类型上的注解：对应ElementType.TYPE_PARAMETER
    @Test
    public void getAnnotationFromTypeVariableOnClass() {
        // 获取在类上声明的泛型变量类型列表，类上可能声明了多个泛型变量类型，因此得到的是一个数组
        TypeVariable&lt;Class&lt;AnnotationReflect>>[] classTypeVariable = AnnotationReflect.class.getTypeParameters();
        // 遍历类上声明的所有泛型变量类型列表
        for (TypeVariable&lt;Class&lt;AnnotationReflect>> typeVariable : classTypeVariable) {
            System.out.println("标注在" + AnnotationReflect.class.getSimpleName() + "类上声明的" + typeVariable.getName() + "泛型变量类型上的注解如下：");
            for (Annotation annotation : typeVariable.getAnnotations()) {
                System.out.println("annotation is :" + annotation);
                System.out.println("当前 annotation 的类型：" + annotation.getClass());
                if (annotation instanceof MyDefaultAnnotationReflect1) {
                    // 如果当前注解是 MyDefaultAnnotationReflect1 类型，则强转后获取注解中的属性值
                    MyDefaultAnnotationReflect1 myDefaultAnnotationReflect = (MyDefaultAnnotationReflect1) annotation;
                    System.out.println("该注解的value：" + myDefaultAnnotationReflect.value());
                }
                if (annotation instanceof MyDefaultAnnotationReflect1_1) {
                    // 如果当前注解是 MyDefaultAnnotationReflect1_1 类型，则强转后获取注解中的属性值
                    MyDefaultAnnotationReflect1_1 myDefaultAnnotationReflect1_1 = (MyDefaultAnnotationReflect1_1) annotation;
                    System.out.println("该注解的value：" + myDefaultAnnotationReflect1_1.value());
                }
            }
            System.out.println("------------------------------------------");
        }
    }

    // 解析标注在方法上声明的泛型变量类型上的注解：对应ElementType.TYPE_PARAMETER
    @Test
    public void getAnnotationFromTypeVariableOnMethod() throws NoSuchMethodException {
        // 获取method1方法对象，对于方法参数是泛型变量类型的，其对应的参数类型传入Object.class即可，因为泛型在JVM中实际上是被擦除了的，替换成了Object
        Method method1 = AnnotationReflect.class.getDeclaredMethod("method1", Object.class, Object.class);
        // 获取当前method1方法上声明的所有泛型变量类型
        TypeVariable&lt;Method>[] typeVariable = method1.getTypeParameters();
        // 遍历方法上声明的所有泛型变量类型列表
        for (TypeVariable&lt;Method> typeParameter : typeVariable) {
            System.out.println("标注在" + method1.getName() + "方法上声明的" + typeParameter.getName() + "泛型变量类型上的注解如下：");
            for (Annotation annotation : typeParameter.getAnnotations()) {
                System.out.println("annotation is :" + annotation);
                System.out.println("当前 annotation 的类型：" + annotation.getClass());
                if (annotation instanceof MyDefaultAnnotationReflect1) {
                    // 如果当前注解是 MyDefaultAnnotationReflect1 类型，则强转后获取注解中的属性值
                    MyDefaultAnnotationReflect1 myDefaultAnnotationReflect = (MyDefaultAnnotationReflect1) annotation;
                    System.out.println("该注解的value：" + myDefaultAnnotationReflect.value());
                }
                if (annotation instanceof MyDefaultAnnotationReflect1_1) {
                    // 如果当前注解是 MyDefaultAnnotationReflect1_1 类型，则强转后获取注解中的属性值
                    MyDefaultAnnotationReflect1_1 myDefaultAnnotationReflect1_1 = (MyDefaultAnnotationReflect1_1) annotation;
                    System.out.println("该注解的value：" + myDefaultAnnotationReflect1_1.value());
                }
            }
            System.out.println("------------------------------------------");
        }
    }

    // 解析标注在泛型类型的实际类型上的注解：对应ElementType.TYPE_USE
    @Test
    public void getAnnotationFromParameterizedType() throws NoSuchFieldException {
        // 获取map成员
        Field map = AnnotationReflect.class.getDeclaredField("map");
        // 获取该成员的类型
        Type genericType = map.getGenericType();
        if (genericType instanceof ParameterizedType) {
            System.out.println("标注在泛型类型" + genericType.getTypeName() + "上的注解如下：");
            // 获取该成员可能使用注解的类型，说白了就是一个成员的类型中如果标注了注解，就使用AnnotatedType接口来描述其类型，而不再使用Type接口描述
            AnnotatedType annotatedType = map.getAnnotatedType();
            // 如果类型是可能被注解标注的泛型类型
            if (annotatedType instanceof AnnotatedParameterizedType) {
                // 获取该泛型类型的真实类型
                AnnotatedType[] annotatedActualTypeArguments = ((AnnotatedParameterizedType) annotatedType).getAnnotatedActualTypeArguments();
                // 遍历可能被注解修饰的真实类型列表
                for (AnnotatedType annotatedActualTypeArgument : annotatedActualTypeArguments) {
                    System.out.println(annotatedActualTypeArgument.getType().getTypeName() + "类型上的注解：");
                    for (Annotation annotation : annotatedActualTypeArgument.getAnnotations()) {
                        System.out.println(annotation);
                    }
                }
            }
        }
    }

    // 解析标注在方法上的注解：对应ElementType.METHOD
    @Test
    public void getAnnotationFromMethod() throws NoSuchMethodException {
        Method method1 = AnnotationReflect.class.getDeclaredMethod("method1", Object.class, Object.class);
        System.out.println("标注在" + method1.getName() + "方法上的注解如下：");
        // 如果当前method1上存在 MyDefaultAnnotationReflect1 注解标注的话
        if (method1.isAnnotationPresent(MyDefaultAnnotationReflect1.class)) {
            // 获取method1上 MyDefaultAnnotationReflect1 类型的注解
            MyDefaultAnnotationReflect1 myDefaultAnnotationReflect1 = method1.getAnnotation(MyDefaultAnnotationReflect1.class);
            System.out.println("annotation is :" + myDefaultAnnotationReflect1);
            System.out.println("当前 annotation 的类型：" + myDefaultAnnotationReflect1.getClass());
            System.out.println("该注解的value：" + myDefaultAnnotationReflect1.value());
        }
        // 如果当前method1上存在 MyDefaultAnnotationReflect1_1 注解标注的话
        if (method1.isAnnotationPresent(MyDefaultAnnotationReflect1_1.class)) {
            // 获取method1上 MyDefaultAnnotationReflect1_1 类型的注解
            MyDefaultAnnotationReflect1_1 myDefaultAnnotationReflect1_1 = method1.getAnnotation(MyDefaultAnnotationReflect1_1.class);
            System.out.println("annotation is :" + myDefaultAnnotationReflect1_1);
            System.out.println("当前 annotation 的类型：" + myDefaultAnnotationReflect1_1.getClass());
            System.out.println("该注解的value：" + myDefaultAnnotationReflect1_1.value());
        }
    }

    // 解析标注在方法参数上的注解：对应ElementType.PARAMETER
    @Test
    public void getAnnotationFromMethodParameter() throws NoSuchMethodException {
        // 获取method1方法
        Method method1 = AnnotationReflect.class.getDeclaredMethod("method1", Object.class, Object.class);
        // 获取method1方法的参数列表（注意获取的是参数列表，而不是参数类型列表，比如String str是方法参数，则str是参数，String是参数类型）
        Parameter[] parameters = method1.getParameters();
        for (Parameter parameter : parameters) {
            System.out.println("标注在" + parameter.getName() + "参数上的注解如下：");
            // 如果当前parameter上存在 MyDefaultAnnotationReflect1_1 注解标注的话
            if (parameter.isAnnotationPresent(MyDefaultAnnotationReflect1.class)) {
                // 获取parameter上 MyDefaultAnnotationReflect1 类型的注解
                MyDefaultAnnotationReflect1 myDefaultAnnotationReflect1 = method1.getAnnotation(MyDefaultAnnotationReflect1.class);
                System.out.println("annotation is :" + myDefaultAnnotationReflect1);
                System.out.println("当前 annotation 的类型：" + myDefaultAnnotationReflect1.getClass());
                System.out.println("该注解的value：" + myDefaultAnnotationReflect1.value());
            }
            // 如果当前parameter上存在 MyDefaultAnnotationReflect1_1 注解标注的话
            if (parameter.isAnnotationPresent(MyDefaultAnnotationReflect1_1.class)) {
                // 获取parameter上 MyDefaultAnnotationReflect1_1 类型的注解
                MyDefaultAnnotationReflect1_1 myDefaultAnnotationReflect1_1 = method1.getAnnotation(MyDefaultAnnotationReflect1_1.class);
                System.out.println("annotation is :" + myDefaultAnnotationReflect1_1);
                System.out.println("当前 annotation 的类型：" + myDefaultAnnotationReflect1_1.getClass());
                System.out.println("该注解的value：" + myDefaultAnnotationReflect1_1.value());
            }
            System.out.println("------------------------------------------");
        }
    }
}
```

# 8. spring对注解的增强
## 8.1 获取注解上的注解
直接通过案例说明：  
<b>自定义注解A11</b>  
```java
package com.zeh.main.annotation.myannotation.spring;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface A11 {
    String value() default "A11";
}
```

<b>自定义注解B11，B11上标注了A11</b>
```java
package com.zeh.main.annotation.myannotation.spring;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
// 当前注解上标注了A11注解
@A11
public @interface B11 {
    String value() default "B11";
}
```

<b>定义目标类，目标类上只标注注解B11</b>
```java
package com.zeh.main.annotation.mytarget.spring;
import com.zeh.main.annotation.myannotation.spring.B11;
// 当前类上标注了B11注解并指定其中的注解成员值
@B11("ERIC")
public class AnnotationSpring {
}
```

<b>获取目标类 AnnotationSpring 上标注的注解，以及注解上标注的注解</b>
```java
package com.zeh.main.annotation.test;
import com.zeh.main.annotation.mytarget.spring.AnnotationSpring;
import org.junit.Test;
import java.lang.annotation.Annotation;
public class TestAnnotationSpring {
    // 获取类标注上的注解；获取类上的注解标注的注解
    @Test
    public void getAnnotationFromAnnotation() {
        // 获取标注在目标类上的注解列表
        Annotation[] annotations = AnnotationSpring.class.getAnnotations();
        // 遍历目标类上标注的注解列表
        for (Annotation annotation : annotations) {
            System.out.println("当前注解（目标类上标注的）：" + annotation);
            // 获取当前注解上标注的注解列表
            Annotation[] annotations1 = annotation.annotationType().getAnnotations();
            // 遍历注解上的注解列表
            for (Annotation annotation1 : annotations1) {
                System.out.println("当前注解" + annotation + "上标注的注解有：" + annotation1);
            }
        }
    }
}
```
<b>测试结果</b>
```
当前注解（目标类上标注的）：@com.zeh.main.annotation.myannotation.spring.B11(value=ERIC)
当前注解@com.zeh.main.annotation.myannotation.spring.B11(value=ERIC)上标注的注解有：@java.lang.annotation.Target(value=[TYPE])
当前注解@com.zeh.main.annotation.myannotation.spring.B11(value=ERIC)上标注的注解有：@java.lang.annotation.Retention(value=RUNTIME)
当前注解@com.zeh.main.annotation.myannotation.spring.B11(value=ERIC)上标注的注解有：@com.zeh.main.annotation.myannotation.spring.A11(value=A11)
```
上面获取注解的过程比较麻烦，spring提供了一个工具类AnnotatedElementUtils，能够方便的获取各种目标上的注解  
<b>使用AnnotatedElementUtils获取目标类上标注的注解</b>  
```java
// 通过spring提供的方法方便的获取目标类上的注解
@Test
public void getAnnotationFromSpring() {
    System.out.println(AnnotatedElementUtils.getMergedAnnotation(AnnotationSpring.class, B11.class));
    System.out.println(AnnotatedElementUtils.getMergedAnnotation(AnnotationSpring.class, A11.class));
}
```
<b>测试结果</b>
```
@com.zeh.main.annotation.myannotation.spring.B11(value=ERIC)
@com.zeh.main.annotation.myannotation.spring.A11(value=A11)
```

## 8.2 我们定义的注解有啥缺点？
通过上面的案例，我们发现，定义了一个注解B11，B11上标注了自定义的注解A11，然后使用B11标注了目标之后，通过spring的AnnotatedElementUtils能很方便的获取到标注在目标上的注解，以及注解上标注的指定注解。  
即：我们通过获取一个B11注解，也能方便的获取到标注在B11上的A11注解内容。  
但现在有一个问题：<font color="#a52a2a">如果想在目标类AnnotationSpring上通过修改B11注解类设置A11注解的值，是不能做到的</font>。  
造成这个现象的原因是：<font color="#a52a2a">Java中注解的定义是不能继承的，B11上标注了A11注解，但是B11注解并不能继承A11注解里面的成员。</font>  

## 8.3 spring通过@AliasFor注解解决了上述问题
@AliasFor注解用于给某个目标注解中的某个成员起别名。  
上案例：  
<b>修改上述B11注解</b>
```java
package com.zeh.main.annotation.myannotation.spring;
import org.springframework.core.annotation.AliasFor;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
// 当前注解上标注了A11注解
@A11
public @interface B11 {
    String value() default "B11";
    // 在B11注解中定义一个新成员，用于表示A11注解中的value成员的别名
    @AliasFor(annotation = A11.class, value = "value")
    String b11Value();
}
```

<b>修改标注目标AnnotationSpring</b>
```java
package com.zeh.main.annotation.mytarget.spring;
import com.zeh.main.annotation.myannotation.spring.B11;
// 当前类上标注了B11注解并指定其中的注解成员值
@B11(value = "ERIC", b11Value = "DAISY")
public class AnnotationSpring {
}
```

<b>测试程序</b>
```java
// 通过spring提供的方法方便的获取目标类上的注解
@Test
public void getAnnotationFromSpring() {
    System.out.println(AnnotatedElementUtils.getMergedAnnotation(AnnotationSpring.class, B11.class));
    System.out.println(AnnotatedElementUtils.getMergedAnnotation(AnnotationSpring.class, A11.class));
}
```

<b>测试结果</b>
```
@com.zeh.main.annotation.myannotation.spring.B11(value=ERIC, b11Value=DAISY)
@com.zeh.main.annotation.myannotation.spring.A11(value=DAISY)
```
上面通过在B11直接上增加一个新的注解成员b11Value，用spring提供的@AliasFor注解标注，让其成为A11注解中value成员的别名，这样一来b11Value成员的作用和A11中的value作用完全相同。  
相当于实现了注解定义的继承。  

<font color="#a52a2a">@AliasFor使用的前提是：@AliasFor的annotation参数指定的目标注解必须标注在当前注解上。</font>比如A11必须得标注在B11上。  

## 8.4 同一个注解中使用@AliasFor
<b>修改B11，给自己的value成员起一个别名</b>
```java
package com.zeh.main.annotation.myannotation.spring;
import org.springframework.core.annotation.AliasFor;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@B11
public @interface B11 {
    // 同一个注解中的两个成员之间互起别名，建议都带上default默认值的设置，并且在标注时二者只能选择一个成员去设置值，同时设置两个属性将报错
    // 同一个注解中，互为别名的2个成员必须互相使用@AliasFor标注对方
    @AliasFor(annotation = B11.class, value = "myOtherValue")
    String value() default "";
    // B11注解中使用@AliasFor为自己的value成员起别名
    @AliasFor(annotation = B11.class, value = "value")
    String myOtherValue() default "";
}
```

<b>修改标注目标</b>
```java
package com.zeh.main.annotation.mytarget.spring;
import com.zeh.main.annotation.myannotation.spring.B11;
// 只能设置B11注解里面互为别名的其中一个成员值，两个都设置解析将报错
@B11(myOtherValue = "DAISY")
public class AnnotationSpring {
}
```

<b>测试程序</b>
```java
// 通过spring提供的方法方便的获取目标类上的注解
@Test
public void getAnnotationFromSpring() {
    System.out.println(AnnotatedElementUtils.getMergedAnnotation(AnnotationSpring.class, B11.class));
}
```

<b>测试结果</b>
```
@com.zeh.main.annotation.myannotation.spring.B11(value=DAISY, myOtherValue=DAISY)
```

从输出结果中可以看到：互为别名的两个成员，随便设置一个值，2者的值都是相同的。  

## 8.5 @AliasFor不指定annotation属性值
<b>修改B11，不设置annotation属性值</b>
```java
package com.zeh.main.annotation.myannotation.spring;
import org.springframework.core.annotation.AliasFor;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@B11
public @interface B11 {
    // 不指定annotation属性值，则默认是@AliasFor修饰的成员所属的当前注解，即B11
    @AliasFor(value = "myOtherValue")
    String value() default "";
    @AliasFor(value = "value")
    String myOtherValue() default "";
}
```

<b>测试程序</b>
```java
// 通过spring提供的方法方便的获取目标类上的注解
@Test
public void getAnnotationFromSpring() {
    System.out.println(AnnotatedElementUtils.getMergedAnnotation(AnnotationSpring.class, B11.class));
}
```

<b>测试结果</b>
```
@com.zeh.main.annotation.myannotation.spring.B11(value=DAISY, myOtherValue=DAISY)
```
可以看出，当@AliasFor不设置annotation属性值时，其默认取得是当前标注的成员所属的注解。  

## 8.6 @AliasFor不指定value或者attribute属性值
当@AliasFor不指定value或者attribute属性值时，自动将@AliasFor修饰的成员作为它的value和attribute值。  
<b>修改B11，不设置value属性值和attribute属性值</b>
```java
package com.zeh.main.annotation.myannotation.spring;
import org.springframework.core.annotation.AliasFor;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
// 当前注解上标注了A11注解
@A11
public @interface B11 {
    String value() default "B11";
    // 不设置value和attribute属性值，则默认将b11Value作为它们的取值
    @AliasFor(annotation = A11.class)
    String b11Value();
}
```

<b>A11</b>
```java
package com.zeh.main.annotation.myannotation.spring;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface A11 {
    // A11的成员名字要和B11中使用@AliasFor修饰的成员名称相同
    String b11Value() default "A11";
}
```

<b>标注目标</b>
```java
package com.zeh.main.annotation.mytarget.spring;
import com.zeh.main.annotation.myannotation.spring.B11;
@B11(b11Value = "DAISY")
public class AnnotationSpring {
}
```

<b>测试程序</b>
```
@Test
public void getAnnotationFromSpring() {
    System.out.println(AnnotatedElementUtils.getMergedAnnotation(AnnotationSpring.class, B11.class));
    System.out.println(AnnotatedElementUtils.getMergedAnnotation(AnnotationSpring.class, A11.class));
}
```

<b>测试结果</b>
```
@com.zeh.main.annotation.myannotation.spring.B11(value=B11, b11Value=DAISY)
@com.zeh.main.annotation.myannotation.spring.A11(b11Value=DAISY)
```
可以看出，当@AliasFor不指定value或者attribute属性值时，自动将@AliasFor修饰的成员作为它的value和attribute值。  