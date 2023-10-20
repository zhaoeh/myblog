---
layout:     post
title:      java 中的泛型
subtitle:   generic表示java中的泛型，泛型实际上很难
categories: [Java初级进阶]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

参考链接：  
[理解java中的泛型](https://blog.csdn.net/u011240877/article/details/53545041)  
[java中的泛型](https://blog.csdn.net/sunxianghuang/article/details/51982979)  

# 1. 泛型
## 1.1 什么是泛型？
当定义一个Class类、定义一个Interface接口、或者定义一个方法Method时，对于这些类、接口和方法中需要使用到的变量不去明确声明其变量的数据类型，而是采用泛型去声明其类型。  
泛型就是一种泛指的数据类型，用<T>来表示。其中的T可以是任何你习惯使用的字母。  
其本质就是定义一种通用的数据类型，用一种参数变量来表示某种通用的类型，即参数化类型；用一个变量来表示类型。  
用泛型声明的类、接口或者方法，想要使用这些类、接口和方法时，建议在使用时明确地指定其中泛型变量的真实类型，否则编译器将自动擦除泛型使用Object替换。  
<b>“泛型” 意味着编写的代码可以被不同类型的对象所重用。</b>  

# 1.2 泛型的作用
<font color="#FF0000"> 口诀：一保护，两避免。 </font>  

保护数据安全性，避免客户端误操作、避免类转换异常。  
jdk1.5之后提供的泛型，目的是提供<b>编译时类型检查</b>，尤其是消除了集合类使用时的ClassCastException。  

## 1.3 泛型的安全警告
对于使用泛型声明的接口、类和方法，在实例化对象时如果未指定泛型对应的具体的数据类型，将会出现安全警告信息。  
此时泛型将被擦除，使用Object类型接收。  
擦除后和直接使用Object接收没有任何区别，所以使用了泛型就一定要指定具体的泛型类型。  
如果擦除泛型后使用Object类型进行接收，则客户端误操作后会报类转换异常。  
但是如果显式指定了泛型类型的具体类型，则在设置内容时一旦类型不符则编译报错，即通过提前暴露的机制避免了在运行时发生类转换异常。  

## 1.4 泛型只在编译阶段有效
泛型的出现就是为了避免类型转换异常，在java编译器对源码进行编译时就进行校验，一旦存在类型转换异常则编译器报错。  
在深入学习泛型前，必须首先指出，java中引入的泛型，只是在源码中声明的，以方便java编译器对泛型的设置进行检查，一旦不符合规范则编译器报错。  
jdk1.5之后为了加入泛型，java编译器也做了很多的改造适配工作。  
当java编译器完成了对源码中泛型的检查后，编译成字节码时会擦除掉所有和泛型相关的内容，所有的泛型变量统一使用Object替换，然后在获取泛型变量修饰的方法返回值时，编译器会自动替我们进行强制转换。  
<b>也就是说，泛型实际上只是编译器提供的一个语法糖，编译后的字节码文件中完全没有泛型的任何相关内容。</b>  

下面看个例子：  
定义一个泛型方法  
```java
package com.zeh.main.generic;
public class MyGeneric {
    public <K> K getName(K kk) {
        return kk;
    }
}
```
调用上面的泛型方法  
```java
package com.zeh.main.generic.test;
import com.zeh.main.generic.MyGeneric;
public class TestMyGeneric {
    public static void main(String[] args) {
        MyGeneric myGeneric = new MyGeneric();
        String result = myGeneric.getName("123");
        System.out.println(result);
    }
}
```
下面使用jad对编译后的MyGeneric.class和TestMyGeneric.class进行反编译，观察编译器将这两个玩意儿最终处理成了啥样  
反编译MyGeneric.class  
```java
package com.zeh.main.generic;
public class MyGeneric
{
    public MyGeneric()
    {
    }
    public Object getName(Object kk)
    {
        return kk;
    }
}
```
反编译TestMyGeneric.class  
```java
package com.zeh.main.generic.test;
import com.zeh.main.generic.MyGeneric;
import java.io.PrintStream;
public class TestMyGeneric
{
    public TestMyGeneric()
    {
    }
    public static void main(String args[])
    {
        MyGeneric myGeneric = new MyGeneric();
        String result = (String)myGeneric.getName("123");
        System.out.println(result);
    }
}
```
通过jad反编译class文件，可以看出，java编译器在将泛型编译成class文件后，删除了所有和泛型相关定义，泛型变量直接使用了Object进行替换。  
在获取泛型返回值时，也直接将Object类型和动态设置进去的真实类型进行了强转。  
<b>因此，java中的泛型实际上是伪泛型，其底层利用编译器对类型之间进行各种强转来实现。 </b>    

## 1.5 java中的类型以及泛型中相关概念
java中所有的类型只有5种：  
1.  泛型（参数化）类型：java中使用接口ParameterizedType表示。  
2.  泛型数组类型：java中使用接口GenericArrayType表示。  
3.  通配符类型类型：java中使用接口WildcardType表示。  
4.  泛型变量类型：java中使用接口TypeVariable表示。  
5.  类类型：java中使用类Class表示。  

<b>泛型相关的几个概念：</b>  
1.  类类型  
    ```java
    public class MyGenericDemo01;
    ```
    其中class定义的MyGenericDemo01就是类类型，在java中class，interface，enum，数组等都是类类型。  
2.  泛型变量类型：  
    ```java
    public class MyGenericDemo02<T1, T2 extends Integer, T3 extends GenericDemo02I1>;
    ```
    其中的T1,T2,T3就是泛型变量类型。泛型变量类型使用前需要在类上或者方法上进行声明。  
3.  泛型类型：  
    就拿上面声明的泛型类MyGenericDemo02来说明。  
    ```java
    MyGenericDemo02<T1,T2,T3> demo;
    ```
    1>.  泛型类型： <b>MyGenericDemo02<T1,T2,T3></b>就是泛型类型。  
    2>.  原始类型：针对泛型类型而言，只有泛型类型才有原始类型， <b>MyGenericDemo02</b>就是原始类型。  
    3>.  实际类型： <b><></b>中的内容叫做实际类型。  
    4>.  实际类型可以取值5种类型中的任意一种。    
    （1）、当实际类型取值为类类型时，我们称此时实际类型为<b>具体类型</b>。  
    （2）、实际类型也可以为<b>泛型变量类型</b>。  
    （3）、实际类型也可以为<b>通配符类型</b>。  
    （4）、实际类型也可以为<b>反省类型</b>。  
    （5）、实际类型也可以为<b>泛型数组类型</b>。  
    如上描述，泛型类型<>中的实际类型可以取值为其他5种类型的任意一种，这样一来泛型类型组合起来是比较复杂的。如下：  
    ```java
    List<Map<List<?>, Map<String,?>>> list;
    List<Map<?,Integer>[]> list1;
    ```
4.  通配符类型：  
    java中使用?表示通配符类型。通配符必须作为泛型类型的实际类型存在，结合泛型类型一起使用，不能单独用于修饰变量。  
    <b>List<?></b>，其中的?就是通配符类型。  
5.  泛型数组类型：  
    数组中的元素是泛型类型的数组，就是泛型数组类型。  
    ```java
    List<String> list[];
    List<?> list[];
    ```
    
# 2. 泛型变量类型的声明
## 2.1 声明泛型
java中用于声明泛型变量的地方总共有3处：接口、类、方法。  
接口名称后使用<>声明泛型变量。  
类名称后使用<>声明泛型。  
方法返回值前使用<>声明泛型。  

<font color="#FF0000"> 口诀：接口类方法，泛型声明处 </font>  

在类中声明泛型，叫做泛型类。  
在接口中声明泛型，叫做泛型接口。  
在方法中声明泛型，叫做泛型方法。  

## 2.2 泛型接口
```
[接口修饰符] interface 接口名称<泛型类型名称>{
}
```
示例如下：  
```java
public interface IGeneric<T> {
}
```

## 2.3 泛型类
```
[类修饰符] class 类名称<泛型类型名称>{
}
```
示例如下：  
```java
 public class MyGeneric<T> {
 }
```

## 2.4 泛型方法（包括泛型构造方法）
1.  泛型构造方法 
    ```
    [方法修饰符] <泛型类型名称> 构造方法名称(){
    }
    ```
    示例如下：  
    ```java
    public <S> MyGeneric(S ss){
    }
    ```
2.  泛型方法
    ```
    [方法修饰符] <泛型类型名称> 返回值类型 方法名称(){
    }
    ```
    示例如下：  
    ```java
    public <K> K getName(K s){
        return s;
    }
    ```

# 3. 泛型成员
泛型成员是在泛型类的基础之上指定的，成员本身不能声明泛型，只能使用已经声明的泛型类型。  
如下案例：  
```java
public class MyGeneric<T> {
    // 使用类中声明的泛型变量去定义成员
    private T name;
    // 构造方法1：构造方法自己声明了泛型变量<S>，则在其方法内，可以使用该泛型变量S去定义其他变量的类型
    public <S> MyGeneric(S ss) {
        S s = null;
    }
    // 构造方法2：构造方法自己声明了泛型变量<S>，类本身也声明了泛型变量<T>。
    // 则构造方法内部既可以使用自己声明的泛型变量S，也可以使用类中声明的泛型变量T。
    public <S> MyGeneric(S ss, T tt) {
        T t = null;
        S s = null;
    }
    // 普通方法1：同理，普通方法声明了自己的泛型变量<K>，则在其方法内，可以使用该泛型变量去定义其他变量的类型。
    public <K> K getName(K kk) {
        K k = (K) new Object();
        return k;
    }
    // 普通方法2：普通方法自己声明了泛型变量<K>，类本身也声明了泛型变量<T>。
    // 则在其方法内部既可以使用自己声明的泛型变量S，也可以使用类中声明的泛型变量T。
    public <K> T getName(K kk, T tt) {
        T t = (T) new Object();
        return t;
    }
}
```

# 4. 泛型的具体类型指定
前面在接口、类和方法上面声明了泛型，实际上相当于定义了一个泛型变量，这个泛型变量是需要在真正使用时指定其具体的类型的。  
1.  实例化泛型类对象时指定泛型变量的具体类型（对于基本类型必须指定其对应的包装类型）。  
2.  在子类实现接口或者继承父类（抽象类）时，指定父类泛型变量的具体类型。  

# 5. 泛型对象在引用传递中的限制
引用传递传递的是引用地址，对于泛型对象而言，应该也能够传递引用。  
普通java对象传递引用时，可以传递相同类型的引用，也可以向上转型，但是泛型对象不同。  
对于普通java对象而言，person对象就是Object类的子类对象，所以person在引用传递时，被调用方可以使用Object接收。  
但是对于泛型java对象而言，Person<String>就不是Person<Object>的子类，因此不能向上转型，此时引用传递编译器会报错。

解决方案：  
1.  被调用方不使用泛型类型去接收泛型对象，此时会直接擦除泛型类型。  
2.  被调用方使用通配符类型?来接受任意的泛型对象，但是不能设置泛型对象的属性值（null值除外）。  
3.  被调用方指定和调用方一致的泛型类型。  

# 6. 通配符类型
## 6.1 什么是通配符类型？
java泛型中的通配符用?表示，这玩意儿看起来挺玄乎，实际上它就是一种类型，表示任意类型，不清楚的类型。  
由于泛型对象在引用传递中的限制，所以引出了通配符的概念。  
通配符类型只能依赖泛型类型生存，而不能单独使用。  
比如：  
```java
private ? name;
```
单独使用?修饰成员，编译器报错。    

```java
private List<?> list;
```
通配符类型作为泛型类型的实际类型，结合泛型类型使用，这种才是通配符打开的唯一正确方式，就如上面的List<?> 。  
下面我们说的通配符类型，指的就是结合泛型类型的这种方式，类似List\<?\>。

当一个对象被通配符类型修饰时，表示该泛型对象可以接受一个实际类型为任意类型的泛型对象，如下：  
```java
public List<?> list;
```
上面的list对象就是通配符类型的，意味着这个对象可以被传入任意实际类型的list对象，比如：  
```java
list = new ArrayList<String>();list = new ArrayList<Integer>();
```

当一个变量被通配符修饰时，该变量不能设置具体的值，只能设置null值。因为该变量的实际类型是通配符类型，表示不清楚的类型，所以向其中设置具体值时，是会编译报错的，如下：  
```java
// list使用通配符修饰
List<?> list = new ArrayList<String>();

// 向list中设置具体值0，编译报错。因为list容器中的实际类型此时是通配符类型，表示不清楚的类型，因此不能设置具体值。
list.add(0);

 // 向list中设置null值，编译正常。因为null值就表示不清楚的值，和通配符的含义相同。
list.add(null);
```
要向其中设置具体值，则应该将通配符类型强转成泛型类型（实际类型为具体类型）。如下：  
```java
// 通配符类型修饰，不能设置具体值
List<?> list = new ArrayList<>(); 

// 将其强转成泛型类型（实际类型为具体类型）
List<String> myList = (List<String>)list; 
```

通配符使用场景：修饰一个成员、一个方法变量、方法参数、方法返回值时，我们不清楚修饰的这些目标的具体成员到底是什么，这个时候就可以使用泛型的通配符类型进行修饰。  
<b>通配符的出现实际上就是应付泛型对象在引用传递中的限制，通配符类型修饰的泛型对象可以接受实际类型为任意类型的泛型对象。</b>  

## 6.2 受限泛型
通配符类型修饰泛型对象时，表示该对象可以接受实际类型为任意类型的泛型对象。  
但是也可以设置通配符类型的上限和下限。  
指定通配符类型的上限，即通配符类型可以接受的实际类型必须是Number类型或者其子类型：  
```java
List<? extends Number>
```
指定通配符类型的下限，即通配符类型可以接受的实际类型必须是Integer类型或者其父类型：  
```java
List<? super Integer>
```

## 6.3 通配符案例
通配符案例:  
```java
package com.zeh.main.generic.basemygeneric;
/**
 * 因为通配符类型必须集合泛型类型，所以先声明一个泛型类
 */
class Generic<T> { // 声明泛型变量类型，表示该类是一个泛型类
    // 该类中的成员、方法参数、返回值都使用泛型变量类型进行修饰
    private T name;
    public T getName() {
        return name;
    }
    public void setName(T name) {
        this.name = name;
    }
    public T test(T t) {
        System.out.println("t is :" + t);
        return getName();
    }
}
public class MyGeneric {
    // 使用上面的泛型类型修饰此处的成员（泛型类型的实际类型是通配符类型）
    private Generic<?> generic;
    public Generic<?> getGeneric() {
        return generic;
    }
    public void setGeneric(Generic<?> generic) {
        this.generic = generic;
    }
    // 同理，泛型类型（实际类型是通配符类型）修饰方法参数、返回值
    public Generic<?> help(Generic<?> generic) {
        System.out.println("generic is :" + generic);
        return this.getGeneric();
    }
    public static void main(String[] args) {
        // 实例化MyGeneric对象，使用通配符接收，意思是new出来的MyGeneric对象的实际类型可以是任意类型
        MyGeneric myGeneric = new MyGeneric();
        // 分别实例化多个不同实际类型的Generic对象
        Generic<Integer> genericInteger = new Generic<Integer>();
        Generic<String> genericString = new Generic<String>();
        // 给MyGeneric中的泛型成员设置了不同的泛型类型变量
        myGeneric.setGeneric(genericInteger);
        Generic<Integer> result1 = (Generic<Integer>) myGeneric.help(genericInteger);
        result1.test(123);
        myGeneric.setGeneric(genericString);
        Generic<String> result2 = (Generic<String>) myGeneric.help(genericString);
        result2.test("123");
        // 通配符类型和一般泛型类型之间强转
        Generic<?> generic = new Generic<>();
        Generic<String> generic1 = (Generic<String>) generic;
        generic1.setName("123");
        generic1.test("123");
    }
}
```

# 7. 案例详解：java中5种类型
## 7.1 类类型
代码：  
```java
package com.zeh.main.generic.mygeneric;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.Type;

// 步骤1：声明一个接口表示类类型
interface GenericDemo01I1 {
}

// 1.   java中类型之一：类类型
// 2.   类类型应该是我们最常见的类型，定义一个接口、一个类、一个枚举、一个数组等，都是声明了类类型。
// 当然也可以直接使用Jdk内置的类类型或者其他第三方提供的类类型。
// 3.   java中任何类型允许修饰的目标只有3种：
// （1）、修饰成员变量（方法变量也可以修饰，只是无法通过反射获取方法变量）
// （2）、修饰方法参数
// （3）、修饰方法返回值
// 4.   步骤：
// （1）、   声明类型
// （2）、   获取第1步声明的类型信息
// （3）、   使用声明的类型修饰成员变量、方法参数、方法返回值
// （4）、   获取第3步成员变量、方法参数、方法返回值的类型信息
public class MyGenericDemo01 { //步骤1：声明一个类表示类类型
    // 步骤3：使用声明的类类型或者已有的类类型修饰成员变量
    private GenericDemo01I1 genericDemo01I1;
    private String name;
    private MyGenericDemo01 myGenericDemo01;
    // 步骤3：使用类类型修饰方法返回值和方法参数
    public MyGenericDemo01 method1(String str, MyGenericDemo01 myGenericDemo01, GenericDemo01I1 genericDemo01I1, int i) {
        return new MyGenericDemo01();
    }
   
    // 打印声明的类类型信息
    private static void printType(Class<?> clazz) {
        System.out.println("参数类型名称：" + clazz.getTypeName());
        System.out.println("参数类名：" + clazz.getName());
        System.out.println("---------------------------------------------");
    }

    // 打印目标方法的参数类型信息
    private static void printMethodPramType(String targetClassName, String targetMethodName, Class<?>... parameterTypes) throws ClassNotFoundException, NoSuchMethodException {
        Class<?> targetClass = Class.forName(targetClassName);
        Method targetMethod = targetClass.getDeclaredMethod(targetMethodName, parameterTypes);
        System.out.println(targetMethod.getName() + "方法的所有参数类型信息：");
        // 获取当前方法的所有参数类型信息，所有参数类型都是类类型的，类类型在java中用 Class 表示
        Type[] genericParameterTypes = targetMethod.getGenericParameterTypes();
        for (Type genericParameterType : genericParameterTypes) {
            if (genericParameterType instanceof Class) {
                Class clazz = (Class) genericParameterType;
                System.out.println("参数类型名称：" + clazz.getTypeName());
                System.out.println("参数类名：" + clazz.getName());
            }
            System.out.println("---------------------------------------------");
        }
    }

    // 打印目标方法的返回值类型信息
    private static void printMethodReturnType(String targetClassName, String targetMethodName, Class<?>... parameterTypes) throws ClassNotFoundException, NoSuchMethodException {
        Class<?> targetClass = Class.forName(targetClassName);
        Method targetMethod = targetClass.getDeclaredMethod(targetMethodName, parameterTypes);
        // 获取当前方法的返回值类型，是类类型，在java中用 Class 表示
        System.out.println(targetMethod.getName() + "方法返回值类型信息：");
        Type genericReturnType = targetMethod.getGenericReturnType();
        if (genericReturnType instanceof Class) {
            Class clazz = (Class) genericReturnType;
            System.out.println("返回值类型名称：" + clazz.getTypeName());
            System.out.println("返回值类名：" + clazz.getName());
        }
        System.out.println("---------------------------------------------");
    }

    // 打印目标成员的类型信息
    private static void printFieldType(String targetClassName, String targetFiledName) throws ClassNotFoundException, NoSuchFieldException {
        Class<?> targetClass = Class.forName(targetClassName);
        Field targetField = targetClass.getDeclaredField(targetFiledName);
        System.out.println(targetField.getName() + "成员的类型信息：");
        // 获取当前成员的类型信息
        Type fieldType = targetField.getGenericType();
        // 如果当前成员类型是类类型，在java中使用 Class 表示
        if (fieldType instanceof Class) {
            Class clazz = (Class) fieldType;
            System.out.println("参数类型名称：" + clazz.getTypeName());
            System.out.println("参数类名：" + clazz.getName());
        }
        System.out.println("---------------------------------------------");
    }
    
    public static void main(String[] args) throws NoSuchMethodException, ClassNotFoundException, NoSuchFieldException {
        // 步骤2：打印声明的类型信息
        System.out.println("===================获取声明的类类型信息===================");
        printType(String.class);
        printType(MyGenericDemo01.class);
        printType(GenericDemo01I1.class);
        // 步骤4：打印方法参数类型、返回值类型和成员类型
        System.out.println("===================获取目标方法上所有参数类型信息===================");
        printMethodPramType("com.zeh.main.generic.mygeneric.MyGenericDemo01", "method1", String.class, MyGenericDemo01.class, GenericDemo01I1.class, int.class);
        System.out.println("===================获取目标方法上返回值类型信息===================");
        printMethodReturnType("com.zeh.main.generic.mygeneric.MyGenericDemo01", "method1", String.class, MyGenericDemo01.class, GenericDemo01I1.class, int.class);
        System.out.println("===================获取目标成员上的类型信息===================");
        printFieldType("com.zeh.main.generic.mygeneric.MyGenericDemo01", "genericDemo01I1");
        printFieldType("com.zeh.main.generic.mygeneric.MyGenericDemo01", "name");
        printFieldType("com.zeh.main.generic.mygeneric.MyGenericDemo01", "myGenericDemo01");
    }
}
```

测试结果：  
```
 ===================获取声明的类类型信息===================
参数类型名称：java.lang.String
参数类名：java.lang.String
---------------------------------------------
参数类型名称：com.zeh.main.generic.mygeneric.MyGenericDemo01
参数类名：com.zeh.main.generic.mygeneric.MyGenericDemo01
---------------------------------------------
参数类型名称：com.zeh.main.generic.mygeneric.GenericDemo01I1
参数类名：com.zeh.main.generic.mygeneric.GenericDemo01I1
---------------------------------------------
===================获取目标方法上所有参数类型信息===================
method1方法的所有参数类型信息：
参数类型名称：java.lang.String
参数类名：java.lang.String
---------------------------------------------
参数类型名称：com.zeh.main.generic.mygeneric.MyGenericDemo01
参数类名：com.zeh.main.generic.mygeneric.MyGenericDemo01
---------------------------------------------
参数类型名称：com.zeh.main.generic.mygeneric.GenericDemo01I1
参数类名：com.zeh.main.generic.mygeneric.GenericDemo01I1
---------------------------------------------
参数类型名称：int
参数类名：int
---------------------------------------------
===================获取目标方法上返回值类型信息===================
method1方法返回值类型信息：
返回值类型名称：com.zeh.main.generic.mygeneric.MyGenericDemo01
返回值类名：com.zeh.main.generic.mygeneric.MyGenericDemo01
---------------------------------------------
===================获取目标成员上的类型信息===================
genericDemo01I1成员的类型信息：
参数类型名称：com.zeh.main.generic.mygeneric.GenericDemo01I1
参数类名：com.zeh.main.generic.mygeneric.GenericDemo01I1
---------------------------------------------
name成员的类型信息：
参数类型名称：java.lang.String
参数类名：java.lang.String
---------------------------------------------
myGenericDemo01成员的类型信息：
参数类型名称：com.zeh.main.generic.mygeneric.MyGenericDemo01
参数类名：com.zeh.main.generic.mygeneric.MyGenericDemo01
---------------------------------------------
```

## 7.2 泛型变量类型
代码：  
```java
package com.zeh.main.generic.mygeneric;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.Type;
import java.lang.reflect.TypeVariable;
interface GenericDemo02I1 {
}
interface GenericDemo02I2 {
}

// 1.   java中类型之二：泛型变量类型
// 2.   泛型变量类型的声明允许在下面2个地方进行：
// （1）、 在类（或者接口）上声明泛型变量类型：泛型类
// （2）、  在方法上声明泛型变量类型：泛型方法
// 3.   java中任何类型允许修饰的目标只有3种：
// （1）、修饰成员变量（方法变量也可以修饰，只是无法通过反射获取方法变量）
// （2）、修饰方法参数
// （3）、修饰方法返回值
// 4.   步骤：
// （1）、   声明类型
// （2）、   获取第1步声明的类型信息
// （3）、   使用声明的类型修饰成员变量、方法参数、方法返回值
// （4）、   获取第3步成员变量、方法参数、方法返回值的类型信息
public class MyGenericDemo02<T1, T2 extends Integer, T3 extends GenericDemo02I1 & GenericDemo02I2> {// 步骤1：在类上声明泛型变量类型
    // 步骤3：使用声明的泛型变量类型修饰成员变量（T1,T2,T3是当前类上声明的泛型变量类型，因此可以使用它们修饰当前类中的成员变量）
    private T1 myT1;
    private T2 myT2;
    private T3 myT3;
    // 步骤3：使用声明的类类型修饰成员变量（MyGenericDemo02是定义好的一个类，即相当于声明了类类型）
    private MyGenericDemo02 myGenericDemo02;

    // 泛型方法
    // 泛型方法中共有3处和类型相关：
    // 1.   方法参数的类型，即(T11 t1, T22 t2, T33 t3, String s)中的T11,T22,T33,String
    // 2.   方法返回值的类型，即T33
    // 3.   方法中声明的泛型变量类型，即<T11, T22 extends Integer, T33 extends GenericDemo02I1 & GenericDemo02I2>中的T1,T2,T3
    // 上述3者之间完全独立。
    // 步骤3：使用声明的泛型变量类型T33修饰方法返回值；使用T11,T22,T33和类类型String修饰方法参数
    public <T11, T22 extends Integer, T33 extends GenericDemo02I1 & GenericDemo02I2> T33 method1(T11 t1, T22 t2, T33 t3, String s) {
        // 步骤1：在方法上声明泛型变量类型
        return t3;
    }
  
    // 获取目标类上的指定方法对象
    private static Method getMethod(Class<MyGenericDemo02> clazz, String methodName) {
        // 获取 MyGenericDemo02 中定义的所有方法
        Method[] methods = MyGenericDemo02.class.getDeclaredMethods();
        // 找到method1方法
        Method method1 = null;
        for (Method method : methods) {
            if (methodName.equals(method.getName())) {
                method1 = method;
                break;
            }
        }
        return method1;
    }

    // 打印目标类 MyGenericDemo02 上声明的泛型变量类型列表
    private static void printClassTypeParameters(Class<MyGenericDemo02> clazz) {
        System.out.println(clazz.getName() + "类上声明的泛型变量类型列表：");
        // 获取当前类上的所有泛型变量类型列表，泛型变量类型在java中使用 TypeVariable 表示
        TypeVariable<Class<MyGenericDemo02>>[] classTypeParameters = clazz.getTypeParameters();
        for (TypeVariable<Class<MyGenericDemo02>> typeParameter : classTypeParameters) {
            System.out.println("该泛型变量类型的名称：" + typeParameter.getName());
            System.out.println("声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：" + typeParameter.getGenericDeclaration());
            Type[] bounds = typeParameter.getBounds();
            System.out.println("该泛型变量类型的上边界数量：" + bounds.length);
            System.out.println("该泛型变量类型的上边界清单：");
            for (Type bound : bounds) {
                System.out.println(bound.getTypeName());
            }
            System.out.println("---------------------------------------------");
        }
    }

    // 打印目标方法上声明的泛型变量类型列表
    private static void printMethodTypeParameters(Method method) {
        // 获取当前方法上声明的所有泛型变量类型列表，泛型变量类型在java中使用 TypeVariable 表示
        System.out.println(method.getName() + "方法声明的泛型变量类型列表：");
        TypeVariable<Method>[] typeVariables = method.getTypeParameters();
        for (TypeVariable<Method> typeVariable : typeVariables) {
            System.out.println("泛型变量类型的名称：" + typeVariable.getName());
            System.out.println("声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：" + typeVariable.getGenericDeclaration());
            Type[] bounds = typeVariable.getBounds();
            System.out.println("该泛型变量类型的上边界数量：" + bounds.length);
            System.out.println("该泛型变量类型的上边界清单：");
            for (Type bound : bounds) {
                System.out.println(bound.getTypeName());
            }
            System.out.println("---------------------------------------------");
        }
    }

    // 打印目标成员的类型信息
    private static void printFieldType(Field field) {
        System.out.println(field.getName() + "成员的类型信息：");
        // 获取当前成员的类型信息
        Type fieldType = field.getGenericType();
        // 如果当前成员是泛型变量类型，在java中使用 TypeVariable 表示
        if (fieldType instanceof TypeVariable) {
            TypeVariable typeVariable = (TypeVariable) fieldType;
            System.out.println("泛型变量类型的名称：" + typeVariable.getName());
            System.out.println("声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：" + typeVariable.getGenericDeclaration());
            Type[] bounds = typeVariable.getBounds();
            System.out.println("该泛型变量类型的上边界数量：" + bounds.length);
            System.out.println("该泛型变量类型的上边界清单：");
            for (Type bound : bounds) {
                System.out.println(bound.getTypeName());
            }
            // 如果当前成员类型是类类型，在java中使用 Class 表示
        } else if (fieldType instanceof Class) {
            Class clazz = (Class) fieldType;
            System.out.println("参数类型名称：" + clazz.getTypeName());
            System.out.println("参数类名：" + clazz.getName());
        }
        System.out.println("---------------------------------------------");
    }

    // 打印目标方法的参数类型信息
    private static void printMethodPramType(Method method) {
        System.out.println(method.getName() + "方法的所有参数类型信息：");
        // 获取当前方法的所有参数类型信息
        Type[] genericParameterTypes = method.getGenericParameterTypes();
        for (Type genericParameterType : genericParameterTypes) {
            // 前3个参数类型都是泛型变量类型的，在java中使用 TypeVariable 表示
            if (genericParameterType instanceof TypeVariable) {
                TypeVariable typeVariable = (TypeVariable) genericParameterType;
                System.out.println("泛型变量类型的名称：" + typeVariable.getName());
                System.out.println("声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：" + typeVariable.getGenericDeclaration());
                Type[] bounds = typeVariable.getBounds();
                System.out.println("该泛型变量类型的上边界数量：" + bounds.length);
                System.out.println("该泛型变量类型的上边界清单：");
                for (Type bound : bounds) {
                    System.out.println(bound.getTypeName());
                }
            }
            // 最后一个参数是类类型的，类类型在java中用 Class 表示
            // 注意，类型这个概念，最后一个参数是String s，即变量s的数据类型是String类型，String是一种数据类型，它在java中就是用Class对象来描述的一种的类型
            else if (genericParameterType instanceof Class) {
                Class clazz = (Class) genericParameterType;
                System.out.println("参数类型名称：" + clazz.getTypeName());
                System.out.println("参数类名：" + clazz.getName());
            }
            System.out.println("---------------------------------------------");
        }
    }

    // 打印目标方法的返回值类型信息
    private static void printMethodReturnType(Method method) {
        // 获取当前方法的返回值，也是一个泛型变量类型
        System.out.println(method.getName() + "方法返回值类型信息：");
        Type genericReturnType = method.getGenericReturnType();
        if (genericReturnType instanceof TypeVariable) {
            TypeVariable typeVariable = (TypeVariable) genericReturnType;
            System.out.println("泛型变量类型的名称：" + typeVariable.getName());
            System.out.println("声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：" + typeVariable.getGenericDeclaration());
            Type[] bounds = typeVariable.getBounds();
            System.out.println("该泛型变量类型的上边界数量：" + bounds.length);
            System.out.println("该泛型变量类型的上边界清单：");
            for (Type bound : bounds) {
                System.out.println(bound.getTypeName());
            }
        } else if (genericReturnType instanceof Class) {
            Class clazz = (Class) genericReturnType;
            System.out.println("返回值类型名称：" + clazz.getTypeName());
            System.out.println("返回值类名：" + clazz.getName());
        }
        System.out.println("---------------------------------------------");
    }
    public static void main(String[] args) throws NoSuchFieldException {
        // 获取目标类 MyGenericDemo02 上的目标方法 method1
        Method method = getMethod(MyGenericDemo02.class, "method1");
        System.out.println("===================获取目标类上声明的泛型变量类型列表===================");
        // 步骤2：打印目标类 MyGenericDemo02 上声明的泛型变量类型列表
        printClassTypeParameters(MyGenericDemo02.class);
        System.out.println("===================获取目标方法上声明的泛型变量类型列表===================");
        // 步骤2：打印目标类 MyGenericDemo02 上的目标方法 method1 上声明的泛型变量类型列表
        printMethodTypeParameters(method);
        System.out.println("===================获取目标方法上所有参数类型信息===================");
        // 步骤4：打印目标方法上的方法参数类型信息
        printMethodPramType(method);
        System.out.println("===================获取目标方法上返回值类型信息===================");
        // 步骤4：打印目标方法上的返回值类型信息
        printMethodReturnType(method);
        System.out.println("===================获取目标成员上的类型信息===================");
        // 步骤4：打印目标成员变量的类型信息
        Field myT1 = MyGenericDemo02.class.getDeclaredField("myT1");
        Field myT2 = MyGenericDemo02.class.getDeclaredField("myT2");
        Field myT3 = MyGenericDemo02.class.getDeclaredField("myT3");
        Field myGenericDemo02 = MyGenericDemo02.class.getDeclaredField("myGenericDemo02");
        printFieldType(myT1);
        printFieldType(myT2);
        printFieldType(myT3);
        printFieldType(myGenericDemo02);
    }
}
```
测试结果：  
```
===================获取目标类上声明的泛型变量类型列表===================
com.zeh.main.generic.mygeneric.MyGenericDemo02类上声明的泛型变量类型列表：
该泛型变量类型的名称：T1
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：class com.zeh.main.generic.mygeneric.MyGenericDemo02
该泛型变量类型的上边界数量：1
该泛型变量类型的上边界清单：
java.lang.Object
---------------------------------------------
该泛型变量类型的名称：T2
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：class com.zeh.main.generic.mygeneric.MyGenericDemo02
该泛型变量类型的上边界数量：1
该泛型变量类型的上边界清单：
java.lang.Integer
---------------------------------------------
该泛型变量类型的名称：T3
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：class com.zeh.main.generic.mygeneric.MyGenericDemo02
该泛型变量类型的上边界数量：2
该泛型变量类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo02I1
com.zeh.main.generic.mygeneric.GenericDemo02I2
---------------------------------------------
===================获取目标方法上声明的泛型变量类型列表===================
method1方法声明的泛型变量类型列表：
泛型变量类型的名称：T11
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：public com.zeh.main.generic.mygeneric.GenericDemo02I1 com.zeh.main.generic.mygeneric.MyGenericDemo02.method1(java.lang.Object,java.lang.Integer,com.zeh.main.generic.mygeneric.GenericDemo02I1,java.lang.String)
该泛型变量类型的上边界数量：1
该泛型变量类型的上边界清单：
java.lang.Object
---------------------------------------------
泛型变量类型的名称：T22
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：public com.zeh.main.generic.mygeneric.GenericDemo02I1 com.zeh.main.generic.mygeneric.MyGenericDemo02.method1(java.lang.Object,java.lang.Integer,com.zeh.main.generic.mygeneric.GenericDemo02I1,java.lang.String)
该泛型变量类型的上边界数量：1
该泛型变量类型的上边界清单：
java.lang.Integer
---------------------------------------------
泛型变量类型的名称：T33
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：public com.zeh.main.generic.mygeneric.GenericDemo02I1 com.zeh.main.generic.mygeneric.MyGenericDemo02.method1(java.lang.Object,java.lang.Integer,com.zeh.main.generic.mygeneric.GenericDemo02I1,java.lang.String)
该泛型变量类型的上边界数量：2
该泛型变量类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo02I1
com.zeh.main.generic.mygeneric.GenericDemo02I2
---------------------------------------------
===================获取目标方法上所有参数类型信息===================
method1方法的所有参数类型信息：
泛型变量类型的名称：T11
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：public com.zeh.main.generic.mygeneric.GenericDemo02I1 com.zeh.main.generic.mygeneric.MyGenericDemo02.method1(java.lang.Object,java.lang.Integer,com.zeh.main.generic.mygeneric.GenericDemo02I1,java.lang.String)
该泛型变量类型的上边界数量：1
该泛型变量类型的上边界清单：
java.lang.Object
---------------------------------------------
泛型变量类型的名称：T22
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：public com.zeh.main.generic.mygeneric.GenericDemo02I1 com.zeh.main.generic.mygeneric.MyGenericDemo02.method1(java.lang.Object,java.lang.Integer,com.zeh.main.generic.mygeneric.GenericDemo02I1,java.lang.String)
该泛型变量类型的上边界数量：1
该泛型变量类型的上边界清单：
java.lang.Integer
---------------------------------------------
泛型变量类型的名称：T33
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：public com.zeh.main.generic.mygeneric.GenericDemo02I1 com.zeh.main.generic.mygeneric.MyGenericDemo02.method1(java.lang.Object,java.lang.Integer,com.zeh.main.generic.mygeneric.GenericDemo02I1,java.lang.String)
该泛型变量类型的上边界数量：2
该泛型变量类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo02I1
com.zeh.main.generic.mygeneric.GenericDemo02I2
---------------------------------------------
参数类型名称：java.lang.String
参数类名：java.lang.String
---------------------------------------------
===================获取目标方法上返回值类型信息===================
method1方法返回值类型信息：
泛型变量类型的名称：T33
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：public com.zeh.main.generic.mygeneric.GenericDemo02I1 com.zeh.main.generic.mygeneric.MyGenericDemo02.method1(java.lang.Object,java.lang.Integer,com.zeh.main.generic.mygeneric.GenericDemo02I1,java.lang.String)
该泛型变量类型的上边界数量：2
该泛型变量类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo02I1
com.zeh.main.generic.mygeneric.GenericDemo02I2
---------------------------------------------
===================获取目标成员上的类型信息===================
myT1成员的类型信息：
泛型变量类型的名称：T1
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：class com.zeh.main.generic.mygeneric.MyGenericDemo02
该泛型变量类型的上边界数量：1
该泛型变量类型的上边界清单：
java.lang.Object
---------------------------------------------
myT2成员的类型信息：
泛型变量类型的名称：T2
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：class com.zeh.main.generic.mygeneric.MyGenericDemo02
该泛型变量类型的上边界数量：1
该泛型变量类型的上边界清单：
java.lang.Integer
---------------------------------------------
myT3成员的类型信息：
泛型变量类型的名称：T3
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：class com.zeh.main.generic.mygeneric.MyGenericDemo02
该泛型变量类型的上边界数量：2
该泛型变量类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo02I1
com.zeh.main.generic.mygeneric.GenericDemo02I2
---------------------------------------------
myGenericDemo02成员的类型信息：
参数类型名称：com.zeh.main.generic.mygeneric.MyGenericDemo02
参数类名：com.zeh.main.generic.mygeneric.MyGenericDemo02
---------------------------------------------
```

## 7.3 泛型类型
代码：  
```java
package com.zeh.main.generic.mygeneric;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.lang.reflect.TypeVariable;
interface GenericDemo03I1 {
}
interface GenericDemo03I2 {
}
class GenericDemo03 implements GenericDemo03I1, GenericDemo03I2 {
}

// 泛型类搞一个子类。
//泛型类和泛型接口定义好后，子类可以继承或者实现。
// 1.   子类在继承或者实现时如果不显式指定父类或者父接口的泛型类型的实际类型，则默认泛型将擦除，使用Object。
// 2.   如果子类指定了父类或者父接口的泛型类型的实际类型，则使用实际类型进行接收。
// 实际类型取值有：
// 如果子类自己声明了泛型变量类型，则父类或者父接口的泛型类型的实际类型可以指定为子类声明的泛型变量类型，也可以指定为具体类型。
// 如果子类没有声明泛型变量类型，则父类或者父接口的泛型类型的实际类型只能指定为具体类型。
class MyGenericDemo03Extend<E extends GenericDemo03I1 & GenericDemo03I2> extends MyGenericDemo03<E> {
}

// 1.   java中类型之三：泛型类型
// 2.   泛型类型实际上就是我们Demo2中在类上定义了泛型变量类型后的类，这个类就是泛型类，也叫做泛型类型。
// 3.   java中任何类型允许修饰的目标只有3种：
// （1）、修饰成员变量（方法变量也可以修饰，只是无法通过反射获取方法变量）
// （2）、修饰方法参数
// （3）、修饰方法返回值
// 4.   步骤：
// （1）、   声明类型
// （2）、   获取第1步声明的类型信息
// （3）、   使用声明的类型修饰成员变量、方法参数、方法返回值
// （4）、   获取第3步成员变量、方法参数、方法返回值的类型信息
public class MyGenericDemo03<T extends GenericDemo03I1 & GenericDemo03I2> {   // 在类上声明一个泛型变量类型，声明好之后这个类就是泛型类型
    public class Class1 {
        private MyGenericDemo03<T> myGenericDemo03;
        public MyGenericDemo03<T> method1(MyGenericDemo03<T> myGenericDemo03) {
            // 对myGenericDemo03做一些操作
            return myGenericDemo03;
        }
        public MyGenericDemo03<GenericDemo03> method2(MyGenericDemo03<GenericDemo03> myGenericDemo03) {
            // 对myGenericDemo03做一些操作
            return myGenericDemo03;
        }
    }

    // 打印声明的泛型类型信息，即 MyGenericDemo03<T extends GenericDemo03I1 & GenericDemo03I2> 类
    private static void printType(Class<?> targetClass) {
        System.out.println("--------声明的泛型类型---------");
        // 获取父类的详细类型信息，MyGenericDemo03Extend的父类是MyGenericDemo03，是个泛型类型，泛型类型在java中用 ParameterizedType 表示
        Type genericType = targetClass.getGenericSuperclass();
        System.out.println("genericType is :" + genericType.getClass());
        Class<?> targetSuperClass = targetClass.getSuperclass();
        if (genericType instanceof ParameterizedType) {
            System.out.println(targetSuperClass.getName() + "类型是泛型类型！！！");
            ParameterizedType parameterizedType = (ParameterizedType) genericType;
            // 获取成员类型的原始类型
            Type rawType = parameterizedType.getRawType();
            System.out.println(targetSuperClass.getName() + "类型的原始类型：" + rawType);
            // 获取成员类型的所属类型
            Type ownerType = parameterizedType.getOwnerType();
            System.out.println(targetSuperClass.getName() + "类型的所属类型：" + ownerType);
            // 获取成员类型的实际类型列表，因为List<T>是泛型类型，所以它有实际类型列表
            Type[] actualTypes = parameterizedType.getActualTypeArguments();
            for (Type actualType : actualTypes) {
                // 如果实际类型是泛型变量类型，在java中使用 TypeVariable 表示
                if (actualType instanceof TypeVariable) {
                    System.out.println("这个实际参数类型是个泛型变量类型！！！");
                    TypeVariable<Class<MyGenericDemo03>> oneActualType = (TypeVariable) actualType;
                    System.out.println("该泛型变量类型的名称：" + oneActualType.getName());
                    System.out.println("声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：" + oneActualType.getGenericDeclaration());
                    Type[] bounds = oneActualType.getBounds();
                    System.out.println("该泛型变量类型的上边界数量：" + bounds.length);
                    System.out.println("该泛型变量类型的上边界清单：");
                    for (Type bound : bounds) {
                        System.out.println(bound.getTypeName());
                    }
                }
                // 如果第一个实际类型是类类型（原始类型或者具体类型）
                else if (actualType instanceof Class) {
                    Class clazz = (Class) actualType;
                    System.out.println("参数类型名称：" + clazz.getTypeName());
                    System.out.println("参数类名：" + clazz.getName());
                }
            }
        }
    }
    private static void printFieldType(String fieldName) throws NoSuchFieldException {
        Field field = MyGenericDemo03.Class1.class.getDeclaredField(fieldName);
        // 获取成员的类型信息
        Type fieldType = field.getGenericType();
        System.out.println("--------------------" + fieldName + "成员类型信息--------------------");
        // field成员类型是泛型类型，因此取出，泛型类型在java中用 ParameterizedType 表示
        if (fieldType instanceof ParameterizedType) {
            System.out.println(fieldName + "成员类型是泛型类型！！！");
            ParameterizedType parameterizedType = (ParameterizedType) fieldType;
            // 获取成员类型的原始类型
            Type rawType = parameterizedType.getRawType();
            System.out.println(fieldName + "成员类型的原始类型：" + rawType);
            // 获取成员类型的所属类型
            Type ownerType = parameterizedType.getOwnerType();
            System.out.println(fieldName + "成员类型的所属类型：" + ownerType);
            // 获取成员类型的实际类型列表，因为List<T>是泛型类型，所以它有实际类型列表
            Type[] actualTypes = parameterizedType.getActualTypeArguments();
            // 第一个实际类型是T，T是泛型变量类型，在java中使用 TypeVariable 表示
            Type actualType = actualTypes[0];
            System.out.println(fieldName + "成员类型的第一个实际类型所表示的类：" + actualType.getClass());
            // 如果第一个实际类型是泛型变量类型
            if (actualType instanceof TypeVariable) {
                System.out.println("这个实际参数类型是个泛型变量类型！！！");
                TypeVariable<Class<MyGenericDemo03>> oneActualType = (TypeVariable) actualType;
                System.out.println("该泛型变量类型的名称：" + oneActualType.getName());
                System.out.println("声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：" + oneActualType.getGenericDeclaration());
                Type[] bounds = oneActualType.getBounds();
                System.out.println("该泛型变量类型的上边界数量：" + bounds.length);
                System.out.println("该泛型变量类型的上边界清单：");
                for (Type bound : bounds) {
                    System.out.println(bound.getTypeName());
                }
            }
            // 如果第一个实际类型是类类型（原始类型或者具体类型）
            else if (actualType instanceof Class) {
                Class clazz = (Class) actualType;
                System.out.println("参数类型名称：" + clazz.getTypeName());
                System.out.println("参数类名：" + clazz.getName());
            }
        }
    }
    private static void printMethodParamType(String methodName) throws NoSuchMethodException {
        // 获取method方法
        Method method = MyGenericDemo03.Class1.class.getDeclaredMethod(methodName, MyGenericDemo03.class);
        // 获取method方法的所有参数类型信息
        Type[] genericParameterTypes = method.getGenericParameterTypes();
        Type genericParameterType1 = genericParameterTypes[0];
        System.out.println("--------------------" + methodName + "方法参数类型信息--------------------");
        // method方法参数列表中有一个类型是泛型类型，因此取出，泛型类型在java中用 ParameterizedType 表示
        if (genericParameterType1 instanceof ParameterizedType) {
            System.out.println(methodName + "方法第一个参数类型是泛型类型！！！");
            ParameterizedType parameterizedType = (ParameterizedType) genericParameterType1;
            // 获取method方法第一个参数类型的原始类型
            Type rawType = parameterizedType.getRawType();
            System.out.println(methodName + "方法第一个参数类型的原始类型：" + rawType);
            // 获取method方法第一个参数类型的所属类型
            Type ownerType = parameterizedType.getOwnerType();
            System.out.println(methodName + "方法第一个参数类型的所属类型：" + ownerType);
            // 获取method方法第一个参数类型的实际类型列表，因为List<T>是泛型类型，所以它有实际类型列表
            Type[] actualTypes = parameterizedType.getActualTypeArguments();
            // 第一个实际类型是T，T是泛型变量类型，在java中使用 TypeVariable 表示
            Type actualType = actualTypes[0];
            System.out.println(methodName + "方法第一个参数类型的第一个实际类型所表示的类：" + actualType.getClass());
            // 如果第一个实际类型是泛型变量类型
            if (actualType instanceof TypeVariable) {
                System.out.println("这个实际参数类型是个泛型变量类型！！！");
                TypeVariable<Class<MyGenericDemo03>> oneActualType = (TypeVariable) actualType;
                System.out.println("该泛型变量类型的名称：" + oneActualType.getName());
                System.out.println("声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：" + oneActualType.getGenericDeclaration());
                Type[] bounds = oneActualType.getBounds();
                System.out.println("该泛型变量类型的上边界数量：" + bounds.length);
                System.out.println("该泛型变量类型的上边界清单：");
                for (Type bound : bounds) {
                    System.out.println(bound.getTypeName());
                }
            }
            // 如果第一个实际类型是类类型（原始类型或者具体类型）
            else if (actualType instanceof Class) {
                Class clazz = (Class) actualType;
                System.out.println("参数类型名称：" + clazz.getTypeName());
                System.out.println("参数类名：" + clazz.getName());
            }
        }
    }
    private static void printMethodReturnType(String methodName) throws NoSuchMethodException {
        // 获取method方法
        Method method = MyGenericDemo03.Class1.class.getDeclaredMethod(methodName, MyGenericDemo03.class);
        System.out.println("--------------------" + methodName + "方法返回值类型信息--------------------");
        // method1方法返回值类型是泛型类型的，泛型类型在java中用 ParameterizedType 表示
        // getGenericReturnType方法会返回当前方法的返回值类型信息，如果返回值类型是泛型类型的，
        Type returnType = method.getGenericReturnType();
        // 如果返回值类型是泛型类型
        if (returnType instanceof ParameterizedType) {
            System.out.println(methodName + "方法返回值类型是泛型类型！！！");
            ParameterizedType parameterizedType = (ParameterizedType) returnType;
            // 获取method1方法返回值类型的原始类型
            Type rawType = parameterizedType.getRawType();
            System.out.println(methodName + "方法返回值类型的原始类型：" + rawType);
            // 获取method1方法返回值类型的所属类型
            Type ownerType = parameterizedType.getOwnerType();
            System.out.println(methodName + "方法返回值类型的所属类型：" + ownerType);
            // 获取method1方法返回值类型的实际类型列表，因为List<T>是泛型类型，所以它有实际类型列表
            Type[] actualTypes = parameterizedType.getActualTypeArguments();
            // 第一个实际类型是T，T是泛型变量类型，在java中使用 TypeVariable 表示
            Type actualType = actualTypes[0];
            System.out.println(methodName + "方法返回值类型的第一个实际类型所表示的类：" + actualType.getClass());
            // 如果第一个实际类型是泛型变量类型
            if (actualType instanceof TypeVariable) {
                System.out.println("这个实际参数类型是个泛型变量类型！！！");
                TypeVariable<Class<MyGenericDemo03>> oneActualType = (TypeVariable) actualType;
                System.out.println("该泛型变量类型的名称：" + oneActualType.getName());
                System.out.println("声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：" + oneActualType.getGenericDeclaration());
                Type[] bounds = oneActualType.getBounds();
                System.out.println("该泛型变量类型的上边界数量：" + bounds.length);
                System.out.println("该泛型变量类型的上边界清单：");
                for (Type bound : bounds) {
                    System.out.println(bound.getTypeName());
                }
            }
            // 如果第一个实际类型是类类型（原始类型或者具体类型）
            else if (actualType instanceof Class) {
                Class clazz = (Class) actualType;
                System.out.println("参数类型名称：" + clazz.getTypeName());
                System.out.println("参数类名：" + clazz.getName());
            }
        }
    }
    public static void main(String[] args) throws NoSuchMethodException, NoSuchFieldException {
        // 步骤2：打印声明的类型信息
        System.out.println("===================获取声明的类类型信息===================");
        printType(MyGenericDemo03Extend.class);
        System.out.println("===================获取声明的类类型信息===================");
        // 自己实现子类 MyGenericDemo03Extend，自己实现子类时指定了子类中的泛型类型的实际类型为具体类型GenericDemo03
        // 但此时该子类的父类 MyGenericDemo03 中的泛型类型的实际类型此时还是泛型变量类型E
        MyGenericDemo03Extend<GenericDemo03> myGenericDemo03Extend = new MyGenericDemo03Extend<GenericDemo03>();
        printType(myGenericDemo03Extend.getClass());
        System.out.println("===================获取声明的类类型信息===================");
        // 匿名内部类实现子类，匿名内部类实现子类时，直接指定了父类MyGenericDemo03中的泛型类型的实际类型为具体类型 GenericDemo03
        MyGenericDemo03<GenericDemo03> myGenericDemo03 = new MyGenericDemo03<GenericDemo03>() {
        };
        printType(myGenericDemo03.getClass());
        // 步骤4：打印方法参数类型、返回值类型和成员类型
        System.out.println("===================获取目标方法上所有参数类型信息===================");
        printMethodParamType("method1");
        System.out.println("===================获取目标方法上所有参数类型信息===================");
        printMethodParamType("method2");
        System.out.println("===================获取目标方法上返回值类型信息===================");
        printMethodReturnType("method1");
        System.out.println("===================获取目标方法上返回值类型信息===================");
        printMethodReturnType("method2");
        System.out.println("===================获取目标成员上的类型信息===================");
        printFieldType("myGenericDemo03");
    }
}
```
测试结果：  
```
===================获取声明的类类型信息===================
--------声明的泛型类型---------
genericType is :class sun.reflect.generics.reflectiveObjects.ParameterizedTypeImpl
com.zeh.main.generic.mygeneric.MyGenericDemo03类型是泛型类型！！！
com.zeh.main.generic.mygeneric.MyGenericDemo03类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo03
com.zeh.main.generic.mygeneric.MyGenericDemo03类型的所属类型：null
这个实际参数类型是个泛型变量类型！！！
该泛型变量类型的名称：E
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：class com.zeh.main.generic.mygeneric.MyGenericDemo03Extend
该泛型变量类型的上边界数量：2
该泛型变量类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo03I1
com.zeh.main.generic.mygeneric.GenericDemo03I2
===================获取声明的类类型信息===================
--------声明的泛型类型---------
genericType is :class sun.reflect.generics.reflectiveObjects.ParameterizedTypeImpl
com.zeh.main.generic.mygeneric.MyGenericDemo03类型是泛型类型！！！
com.zeh.main.generic.mygeneric.MyGenericDemo03类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo03
com.zeh.main.generic.mygeneric.MyGenericDemo03类型的所属类型：null
这个实际参数类型是个泛型变量类型！！！
该泛型变量类型的名称：E
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：class com.zeh.main.generic.mygeneric.MyGenericDemo03Extend
该泛型变量类型的上边界数量：2
该泛型变量类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo03I1
com.zeh.main.generic.mygeneric.GenericDemo03I2
===================获取声明的类类型信息===================
--------声明的泛型类型---------
genericType is :class sun.reflect.generics.reflectiveObjects.ParameterizedTypeImpl
com.zeh.main.generic.mygeneric.MyGenericDemo03类型是泛型类型！！！
com.zeh.main.generic.mygeneric.MyGenericDemo03类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo03
com.zeh.main.generic.mygeneric.MyGenericDemo03类型的所属类型：null
参数类型名称：com.zeh.main.generic.mygeneric.GenericDemo03
参数类名：com.zeh.main.generic.mygeneric.GenericDemo03
===================获取目标方法上所有参数类型信息===================
--------------------method1方法参数类型信息--------------------
method1方法第一个参数类型是泛型类型！！！
method1方法第一个参数类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo03
method1方法第一个参数类型的所属类型：null
method1方法第一个参数类型的第一个实际类型所表示的类：class sun.reflect.generics.reflectiveObjects.TypeVariableImpl
这个实际参数类型是个泛型变量类型！！！
该泛型变量类型的名称：T
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：class com.zeh.main.generic.mygeneric.MyGenericDemo03
该泛型变量类型的上边界数量：2
该泛型变量类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo03I1
com.zeh.main.generic.mygeneric.GenericDemo03I2
===================获取目标方法上所有参数类型信息===================
--------------------method2方法参数类型信息--------------------
method2方法第一个参数类型是泛型类型！！！
method2方法第一个参数类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo03
method2方法第一个参数类型的所属类型：null
method2方法第一个参数类型的第一个实际类型所表示的类：class java.lang.Class
参数类型名称：com.zeh.main.generic.mygeneric.GenericDemo03
参数类名：com.zeh.main.generic.mygeneric.GenericDemo03
===================获取目标方法上返回值类型信息===================
--------------------method1方法返回值类型信息--------------------
method1方法返回值类型是泛型类型！！！
method1方法返回值类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo03
method1方法返回值类型的所属类型：null
method1方法返回值类型的第一个实际类型所表示的类：class sun.reflect.generics.reflectiveObjects.TypeVariableImpl
这个实际参数类型是个泛型变量类型！！！
该泛型变量类型的名称：T
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：class com.zeh.main.generic.mygeneric.MyGenericDemo03
该泛型变量类型的上边界数量：2
该泛型变量类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo03I1
com.zeh.main.generic.mygeneric.GenericDemo03I2
===================获取目标方法上返回值类型信息===================
--------------------method2方法返回值类型信息--------------------
method2方法返回值类型是泛型类型！！！
method2方法返回值类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo03
method2方法返回值类型的所属类型：null
method2方法返回值类型的第一个实际类型所表示的类：class java.lang.Class
参数类型名称：com.zeh.main.generic.mygeneric.GenericDemo03
参数类名：com.zeh.main.generic.mygeneric.GenericDemo03
===================获取目标成员上的类型信息===================
--------------------myGenericDemo03成员类型信息--------------------
myGenericDemo03成员类型是泛型类型！！！
myGenericDemo03成员类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo03
myGenericDemo03成员类型的所属类型：null
myGenericDemo03成员类型的第一个实际类型所表示的类：class sun.reflect.generics.reflectiveObjects.TypeVariableImpl
这个实际参数类型是个泛型变量类型！！！
该泛型变量类型的名称：T
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：class com.zeh.main.generic.mygeneric.MyGenericDemo03
该泛型变量类型的上边界数量：2
该泛型变量类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo03I1
com.zeh.main.generic.mygeneric.GenericDemo03I2
```

## 7.4 通配符类型
代码：  
```java
package com.zeh.main.generic.mygeneric;
import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.lang.reflect.WildcardType;
import java.util.List;
import java.util.Map;
interface GenericDemo04I1 {
}
interface GenericDemo04I2 {
}

// 1.   java中类型之四：通配符类型
// 2.   前面学习泛型类型时，我们知道泛型类型一般表示为 List<String>或者List<T>，其中List叫做原始类型，<>中的叫做实际类型；实际类型可以是具体类型比如String，也可以是声明的泛型变量类型比如T。
// 在这里我们要新增一个类型，即实际类型也可以是通配符类型?。比如List<?>。
// 通配符和前面学习的类类型、泛型变量类型不同，类类型和泛型变量类型都可以单独作为一种类型，而且在使用前必须声明。
// 而通配符是java编译器默认为我们创建的，用?表示，直接使用，没法声明；
// 并且通配符类型必须作为泛型类型的实际类型一起使用，而不能单独使用，比如下面：
// private ? name;
// 这种直接尝试使用?去修饰成员，编译器是禁止的。
// 可以这样理解，通配符类型?是对泛型变量类型的一个特例，表示可以接受任何实际类型。
// 因此，本案例实际上和Demo03很类似，都是通过泛型类型去修饰成员、方法参数和返回值，只不过泛型类型的实际参数变成了通配符类型而已。
// 3.   java中任何类型允许修饰的目标只有3种：
// （1）、修饰成员变量（方法变量也可以修饰，只是无法通过反射获取方法变量）
// （2）、修饰方法参数
// （3）、修饰方法返回值
// 4.   步骤：
// （1）、   声明类型（通配符类型不用声明）
// （2）、   获取第1步声明的类型信息（通配符类型不能获取声明信息）
// （3）、   使用声明的类型修饰成员变量、方法参数、方法返回值
// （4）、   获取第3步成员变量、方法参数、方法返回值的类型信息
public class MyGenericDemo04<T extends GenericDemo04I1 & GenericDemo04I2> {
    // 使用泛型类型修饰成员变量，只是和前面相比，该泛型类型的实际类型不是具体类型、也不是泛型变量类型，而是通配符类型
    private MyGenericDemo04<?> myGenericDemo04;
    public MyGenericDemo04<? extends GenericDemo04I1> method1(MyGenericDemo04<?> myGenericDemo04, List<?> list, Map<? extends GenericDemo04I1, ? super GenericDemo04I2> map) {
        return myGenericDemo04;
    }

    // 打印方法参数类型列表
    private static void printMethodParamType(String methodName) throws NoSuchMethodException {
        // 获取当前method方法
        Method method = MyGenericDemo04.class.getDeclaredMethod(methodName, MyGenericDemo04.class, List.class, Map.class);
        // 获取当前method方法的所有参数类型信息
        Type[] genericParameterTypes = method.getGenericParameterTypes();
        System.out.println("--------------------" + methodName + "方法参数类型信息--------------------");
        // 遍历当前method的所有参数类型列表
        for (Type genericParameterType : genericParameterTypes) {
            // 当前method方法参数类型列表中都是泛型类型，泛型类型在java中用 ParameterizedType 表示
            if (genericParameterType instanceof ParameterizedType) {
                System.out.println(methodName + "方法" + genericParameterType.getTypeName() + "类型是泛型类型！！！");
                // 将当前参数类型强制转换成 ParameterizedType 类型
                ParameterizedType parameterizedType = (ParameterizedType) genericParameterType;
                // 获取当前参数类型的原始类型
                Type rawType = parameterizedType.getRawType();
                System.out.println(methodName + "方法" + genericParameterType.getTypeName() + "类型的原始类型：" + rawType);
                // 获取当前参数类型的所属类型
                Type ownerType = parameterizedType.getOwnerType();
                System.out.println(methodName + "方法" + genericParameterType.getTypeName() + "类型的所属类型：" + ownerType);
                // 获取当前参数类型的实际类型列表，因为method1方法的2个参数类型分别是MyGenericDemo04<?>和List<?>，都是泛型类型，因此它们都有实际类型
                Type[] actualTypes = parameterizedType.getActualTypeArguments();
                for (Type actualType : actualTypes) {
                    // 如果实际类型是通配符类型，在java中使用 WildcardType 表示
                    if (actualType instanceof WildcardType) {
                        System.out.println(actualType + "实际参数类型是个通配符类型！！！");
                        // 强转当前实际类型为 WildcardType 通配符类型
                        WildcardType currentActualType = (WildcardType) actualType;
                        // 获取通配符的名称 输出是 ?
                        System.out.println("该通配符类型的名称：" + currentActualType.getTypeName());
                        Type[] upperBounds = currentActualType.getUpperBounds();
                        System.out.println("该通配符类型的上边界数量：" + upperBounds.length);
                        System.out.println("该通配符类型的上边界清单：");
                        for (Type bound : upperBounds) {
                            System.out.println(bound.getTypeName());
                        }
                        Type[] lowerBounds = currentActualType.getLowerBounds();
                        System.out.println("该通配符类型的下边界数量：" + lowerBounds.length);
                        System.out.println("该通配符类型的下边界清单：");
                        for (Type bound : lowerBounds) {
                            System.out.println(bound.getTypeName());
                        }
                    }
                }
            }
            System.out.println("---------------------------------------------");
        }
    }
    private static void printMethodReturnType(String methodName) throws NoSuchMethodException {
        // 获取当前method方法
        Method method = MyGenericDemo04.class.getDeclaredMethod(methodName, MyGenericDemo04.class, List.class, Map.class);
        System.out.println("--------------------" + methodName + "方法返回值类型信息--------------------");
        // 当前method方法返回值类型是泛型类型的，泛型类型在java中用 ParameterizedType 表示
        // getGenericReturnType方法会返回当前方法的返回值类型信息
        Type returnType = method.getGenericReturnType();
        // 如果返回值类型是泛型类型
        if (returnType instanceof ParameterizedType) {
            System.out.println(methodName + "方法返回值类型是泛型类型！！！");
            ParameterizedType parameterizedType = (ParameterizedType) returnType;
            // 获取当前method方法返回值类型的原始类型
            Type rawType = parameterizedType.getRawType();
            System.out.println(methodName + "方法返回值类型的原始类型：" + rawType);
            // 获取当前method方法返回值类型的所属类型
            Type ownerType = parameterizedType.getOwnerType();
            System.out.println(methodName + "方法返回值类型的所属类型：" + ownerType);
            // 获取当前method方法返回值类型的实际类型列表，因为MyGenericDemo04<? extends GenericDemo04I1>是泛型类型，所以它有实际类型列表
            Type[] actualTypes = parameterizedType.getActualTypeArguments();
            for (Type actualType : actualTypes) {
                System.out.println(methodName + "方法返回值类型当前实际类型所表示的类：" + actualType.getClass());
                // 如果实际类型是通配符类型，在java中使用 WildcardType 表示
                if (actualType instanceof WildcardType) {
                    System.out.println(actualType + "实际参数类型是个通配符类型！！！");
                    // 强转当前实际类型为 WildcardType 通配符类型
                    WildcardType currentActualType = (WildcardType) actualType;
                    // 获取通配符的名称 输出是 ?
                    System.out.println("该通配符类型的名称：" + currentActualType.getTypeName());
                    Type[] upperBounds = currentActualType.getUpperBounds();
                    System.out.println("该通配符类型的上边界数量：" + upperBounds.length);
                    System.out.println("该通配符类型的上边界清单：");
                    for (Type bound : upperBounds) {
                        System.out.println(bound.getTypeName());
                    }
                    Type[] lowerBounds = currentActualType.getLowerBounds();
                    System.out.println("该通配符类型的下边界数量：" + lowerBounds.length);
                    System.out.println("该通配符类型的下边界清单：");
                    for (Type bound : lowerBounds) {
                        System.out.println(bound.getTypeName());
                    }
                }
            }
        }
    }
    public static void main(String[] args) throws NoSuchMethodException {
        // 步骤4：打印方法参数类型、返回值类型和成员类型
        System.out.println("===================获取目标方法上所有参数类型信息===================");
        printMethodParamType("method1");
        System.out.println("===================获取目标方法上返回值类型信息===================");
        printMethodReturnType("method1");
    }
}
```
测试结果：  
```
===================获取目标方法上所有参数类型信息===================
--------------------method1方法参数类型信息--------------------
method1方法com.zeh.main.generic.mygeneric.MyGenericDemo04<?>类型是泛型类型！！！
method1方法com.zeh.main.generic.mygeneric.MyGenericDemo04<?>类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo04
method1方法com.zeh.main.generic.mygeneric.MyGenericDemo04<?>类型的所属类型：null
?实际参数类型是个通配符类型！！！
该通配符类型的名称：?
该通配符类型的上边界数量：1
该通配符类型的上边界清单：
java.lang.Object
该通配符类型的下边界数量：0
该通配符类型的下边界清单：
---------------------------------------------
method1方法java.util.List<?>类型是泛型类型！！！
method1方法java.util.List<?>类型的原始类型：interface java.util.List
method1方法java.util.List<?>类型的所属类型：null
?实际参数类型是个通配符类型！！！
该通配符类型的名称：?
该通配符类型的上边界数量：1
该通配符类型的上边界清单：
java.lang.Object
该通配符类型的下边界数量：0
该通配符类型的下边界清单：
---------------------------------------------
method1方法java.util.Map<? extends com.zeh.main.generic.mygeneric.GenericDemo04I1, ? super com.zeh.main.generic.mygeneric.GenericDemo04I2>类型是泛型类型！！！
method1方法java.util.Map<? extends com.zeh.main.generic.mygeneric.GenericDemo04I1, ? super com.zeh.main.generic.mygeneric.GenericDemo04I2>类型的原始类型：interface java.util.Map
method1方法java.util.Map<? extends com.zeh.main.generic.mygeneric.GenericDemo04I1, ? super com.zeh.main.generic.mygeneric.GenericDemo04I2>类型的所属类型：null
? extends com.zeh.main.generic.mygeneric.GenericDemo04I1实际参数类型是个通配符类型！！！
该通配符类型的名称：? extends com.zeh.main.generic.mygeneric.GenericDemo04I1
该通配符类型的上边界数量：1
该通配符类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo04I1
该通配符类型的下边界数量：0
该通配符类型的下边界清单：
? super com.zeh.main.generic.mygeneric.GenericDemo04I2实际参数类型是个通配符类型！！！
该通配符类型的名称：? super com.zeh.main.generic.mygeneric.GenericDemo04I2
该通配符类型的上边界数量：1
该通配符类型的上边界清单：
java.lang.Object
该通配符类型的下边界数量：1
该通配符类型的下边界清单：
com.zeh.main.generic.mygeneric.GenericDemo04I2
---------------------------------------------
===================获取目标方法上返回值类型信息===================
--------------------method1方法返回值类型信息--------------------
method1方法返回值类型是泛型类型！！！
method1方法返回值类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo04
method1方法返回值类型的所属类型：null
method1方法返回值类型当前实际类型所表示的类：class sun.reflect.generics.reflectiveObjects.WildcardTypeImpl
? extends com.zeh.main.generic.mygeneric.GenericDemo04I1实际参数类型是个通配符类型！！！
该通配符类型的名称：? extends com.zeh.main.generic.mygeneric.GenericDemo04I1
该通配符类型的上边界数量：1
该通配符类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo04I1
该通配符类型的下边界数量：0
该通配符类型的下边界清单：
```

## 7.5 泛型数组类型
代码：  
```java
package com.zeh.main.generic.mygeneric;
import java.lang.reflect.Field;
import java.lang.reflect.GenericArrayType;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.lang.reflect.TypeVariable;
import java.lang.reflect.WildcardType;
interface GenericDemo05I1 {
}
interface GenericDemo05I2 {
}
class GenericDemo05 implements GenericDemo05I1, GenericDemo05I2 {
}

// 1.   java中类型之五：泛型数组类型
// 2.   泛型数组类型：数组中的元素是泛型类型的数组，就是泛型数组类型。所以说泛型数组的本质还是泛型类型，因此泛型数组类型不用声明，只需要声明对应的泛型类型即可。
// 3.   java中任何类型允许修饰的目标只有3种：
// （1）、修饰成员变量（方法变量也可以修饰，只是无法通过反射获取方法变量）
// （2）、修饰方法参数
// （3）、修饰方法返回值
// 4.   步骤：
// （1）、   声明类型
// （2）、   获取第1步声明的类型信息
// （3）、   使用声明的类型修饰成员变量、方法参数、方法返回值
// （4）、   获取第3步成员变量、方法参数、方法返回值的类型信息
public class MyGenericDemo05<T extends GenericDemo05I1 & GenericDemo05I2> {
    // 我们使用前面的泛型类型，来一个比较全面的案例：获取成员的类型就够了，方法参数类型和返回值类型和前面的案例大同小异。
    // 泛型数组类型修饰成员，实际类型是泛型变量类型
    private MyGenericDemo05<T> m1[];
    // 泛型数组类型修饰成员，实际类型是类类型
    private MyGenericDemo05<GenericDemo05> m2[];
    // 泛型数组类型修饰成员，实际类型是通配符类型
    private MyGenericDemo05<?> m3[];
    private static void printFieldType(String fieldName) throws NoSuchFieldException {
        System.out.println("--------------------" + fieldName + "成员类型信息--------------------");
        Field field = MyGenericDemo05.class.getDeclaredField(fieldName);
        // 获取成员的类型信息
        Type fieldType = field.getGenericType();
        System.out.println(fieldName + "成员的类型：" + fieldType.getClass());
        // 当前field成员类型是泛型数组类型，在java中用 GenericArrayType 表示
        if (fieldType instanceof GenericArrayType) {
            // 将当前成员类型强转成 GenericArrayType 类型
            GenericArrayType genericArrayType = (GenericArrayType) fieldType;
            // 获取泛型数组类型的实际类型，此处的实际类型是MyGenericDemo05<T>、MyGenericDemo05<GenericDemo05>和MyGenericDemo05<?>，都是泛型类型
            Type genericType = genericArrayType.getGenericComponentType();
            System.out.println("当前泛型数组的实际类型是：" + genericType.getClass());
            if (genericType instanceof ParameterizedType) {
                System.out.println(fieldName + "成员类型是泛型类型！！！");
                ParameterizedType parameterizedType = (ParameterizedType) genericType;
                // 获取成员类型的原始类型
                Type rawType = parameterizedType.getRawType();
                System.out.println(fieldName + "成员类型的原始类型：" + rawType);
                // 获取成员类型的所属类型
                Type ownerType = parameterizedType.getOwnerType();
                System.out.println(fieldName + "成员类型的所属类型：" + ownerType);
                // 获取成员类型的实际类型列表，因为List<T>是泛型类型，所以它有实际类型列表
                Type[] actualTypes = parameterizedType.getActualTypeArguments();
                for (Type actualType : actualTypes) {
                    System.out.println(fieldName + "成员类型的当前实际类型所表示的类：" + actualType.getClass());
                    // 如果当前实际类型是泛型变量类型
                    if (actualType instanceof TypeVariable) {
                        System.out.println("这个实际参数类型是个泛型变量类型！！！");
                        TypeVariable<Class<MyGenericDemo03>> oneActualType = (TypeVariable) actualType;
                        System.out.println("该泛型变量类型的名称：" + oneActualType.getName());
                        System.out.println("声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：" + oneActualType.getGenericDeclaration());
                        Type[] bounds = oneActualType.getBounds();
                        System.out.println("该泛型变量类型的上边界数量：" + bounds.length);
                        System.out.println("该泛型变量类型的上边界清单：");
                        for (Type bound : bounds) {
                            System.out.println(bound.getTypeName());
                        }
                    }
                    // 如果当前实际类型是类类型（原始类型或者具体类型）
                    else if (actualType instanceof Class) {
                        Class clazz = (Class) actualType;
                        System.out.println("参数类型名称：" + clazz.getTypeName());
                        System.out.println("参数类名：" + clazz.getName());
                    }
                    // 如果当前实际类型是通配符类型
                    else if (actualType instanceof WildcardType) {
                        System.out.println(actualType + "实际参数类型是个通配符类型！！！");
                        // 强转当前实际类型为 WildcardType 通配符类型
                        WildcardType currentActualType = (WildcardType) actualType;
                        // 获取通配符的名称 输出是 ?
                        System.out.println("该通配符类型的名称：" + currentActualType.getTypeName());
                        Type[] upperBounds = currentActualType.getUpperBounds();
                        System.out.println("该通配符类型的上边界数量：" + upperBounds.length);
                        System.out.println("该通配符类型的上边界清单：");
                        for (Type bound : upperBounds) {
                            System.out.println(bound.getTypeName());
                        }
                        Type[] lowerBounds = currentActualType.getLowerBounds();
                        System.out.println("该通配符类型的下边界数量：" + lowerBounds.length);
                        System.out.println("该通配符类型的下边界清单：");
                        for (Type bound : lowerBounds) {
                            System.out.println(bound.getTypeName());
                        }
                    }
                }
            }
        }
    }
    public static void main(String[] args) throws NoSuchFieldException {
        printFieldType("m1");
        printFieldType("m2");
        printFieldType("m3");
    }
}
```
测试结果：  
```
--------------------m1成员类型信息--------------------
m1成员的类型：class sun.reflect.generics.reflectiveObjects.GenericArrayTypeImpl
当前泛型数组的实际类型是：class sun.reflect.generics.reflectiveObjects.ParameterizedTypeImpl
m1成员类型是泛型类型！！！
m1成员类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo05
m1成员类型的所属类型：null
m1成员类型的当前实际类型所表示的类：class sun.reflect.generics.reflectiveObjects.TypeVariableImpl
这个实际参数类型是个泛型变量类型！！！
该泛型变量类型的名称：T
声明该泛型变量类型的原始类型（即该泛型变量在哪声明）：class com.zeh.main.generic.mygeneric.MyGenericDemo05
该泛型变量类型的上边界数量：2
该泛型变量类型的上边界清单：
com.zeh.main.generic.mygeneric.GenericDemo05I1
com.zeh.main.generic.mygeneric.GenericDemo05I2
--------------------m2成员类型信息--------------------
m2成员的类型：class sun.reflect.generics.reflectiveObjects.GenericArrayTypeImpl
当前泛型数组的实际类型是：class sun.reflect.generics.reflectiveObjects.ParameterizedTypeImpl
m2成员类型是泛型类型！！！
m2成员类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo05
m2成员类型的所属类型：null
m2成员类型的当前实际类型所表示的类：class java.lang.Class
参数类型名称：com.zeh.main.generic.mygeneric.GenericDemo05
参数类名：com.zeh.main.generic.mygeneric.GenericDemo05
--------------------m3成员类型信息--------------------
m3成员的类型：class sun.reflect.generics.reflectiveObjects.GenericArrayTypeImpl
当前泛型数组的实际类型是：class sun.reflect.generics.reflectiveObjects.ParameterizedTypeImpl
m3成员类型是泛型类型！！！
m3成员类型的原始类型：class com.zeh.main.generic.mygeneric.MyGenericDemo05
m3成员类型的所属类型：null
m3成员类型的当前实际类型所表示的类：class sun.reflect.generics.reflectiveObjects.WildcardTypeImpl
?实际参数类型是个通配符类型！！！
该通配符类型的名称：?
该通配符类型的上边界数量：1
该通配符类型的上边界清单：
java.lang.Object
该通配符类型的下边界数量：0
该通配符类型的下边界清单：
```




