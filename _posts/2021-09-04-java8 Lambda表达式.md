---
layout:     post
title:      java8 Lambda表达式
subtitle:   Lambda表达式方便的为函数式接口创建对象
categories: [JAVA8]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 什么是Lambda表达式？
我们一再说明，在java8之前，我们从编码层面上也能够很方便的为一个java接口创建对象，而不用显式的编写该接口的实现子类。   
这种方式就是匿名内部类的方式。   
比如Runnable接口。   
但匿名内部类的方式实际上也依旧臃肿，繁琐，不简洁，从编码角度来看，根本无法体现“函数式编程”的特征。   
因此，java8升级了jdk源码，java编译器和解释器等，以让jdk源码中出现了一种新的接口类别----函数式接口。   
因为函数式接口中有且仅有一个抽象方法，这样一来，java编译器和解释器就能够很精准的根据函数式接口中的抽象方法定义，去做一些推断。   
java8一直提倡函数式编程，将原来java中在方法间通过传递java对象的方式转化为传递函数的方式。   
而函数实际上是一段可执行的逻辑代码。   

其实，函数式编程在java中的本质原理并没有发生任何变化，只是从编译器、解释器和jdk层面提供的一些语法层面的改动而已，方便开发者去使用函数式编程的方式进行代码编写。   
java8使用Lambda表达式来表示一个“函数”，其本质就是一段可执行的逻辑代码，Lambda表达式从语法上来看简介干练，不拖泥带水，究其原因就是因为它其实是函数式接口的一个匿名实现类对象而已。   
正式因为java8提供了函数式接口，其中只有一个抽象方法，因此，java编译器和解释器才能替外表简介的Lambda表达式做很多补充工作。   
比如自动推断Lambda表达式的参数类型---类型推断。   

因此，我们知道了Lambda的底层原理，便清楚了，它不过是java8为函数式接口提供的一个看起来像一个“函数式逻辑片段”的表达式而已，但它实际上通过编译器和解释器执行后，本质还是一个接口的实现类对象而已。

# 2. Lambda表达式的语法规则
（1）java8的编译器和解释器从语法层面支持Lambda表达式，因此只要使用java8及其以上版本，就可以使用Lambda表达式的语法。   
（2）lambda表达式是一段可以传递的代码逻辑，它的核心思想是将面向对象中的传递数据变成传递行为，即将行为数据化。      
（3）Lambda表达式只能针对函数式接口使用，为函数式接口创建一个匿名对象，对于其他普通类和普通接口没法使用Lambda表达式。   
（4）实际上在java8之前，通过匿名内部类的方式直接实现一个接口的抽象方法，也是将行为进行数据化的实现。   

语法规范：   
Lambda表达式核心语法三段式，参数，箭头，语句。
```
(参数) -> {语句}
```   
即：   
```
（...）-> { ...}
```

# 3. Lambda语法详解 
()->{}中，各个部分的含义如下：   
（1）()是要实现的函数式接口的抽象方法的形式参数。   
如果只有1个参数，可以不用()；如果没有参数或者有多个参数，则必须使用()。   
可以显式指定参数类型，显式指定时必须明确的使用()括起来；也可以不指定参数类型，java编译器和解释器将自动根据函数式接口的抽象方法参数进行类型推断。   
当存在多个参数时，使用()括起来，各个参数之间使用,分隔。   

（2）-> 表示Lambda的固定写法。

（3）{}中是Lambda要实现的函数式接口的抽象方法的方法体。   
只有一行语句时，可以不加{}。   
有返回值时，当{}中的实现只有一行语句时，可不写{}，也不写return。   
当然，也可以显式编写return，但是只要显式编写return，就不能省略{}，方法体逻辑必须包含在{}中。   
不用指定返回值类型，编译器和解释器根据函数式接口抽象方法自动进行类型推断。   

形式化的语法描述大概如下：   
```
expression = (variable) -> {action}
```
variable: 这是一个变量,一个占位符。像x,y,z,可以是多个变量。   
action: 这里我称它为action, 这是我们实现的代码逻辑部分,它可以是一行代码也可以是一个代码片段。   
可以看到Java中lambda表达式的格式：参数、箭头、以及动作实现，当一个动作实现无法用一行代码完成，可以编写 一段代码用{}包裹起来。   

# 4. Lambda表达式注意事项
（1）通常都会把lambda表达式内部变量的名字起得短一些。这样能使代码更简短，放在同一行。所以，在上述代码中，变量名选用a、b或者x、y会比even、odd要好。   
（2）Lambda虽然本质是匿名内部类对象，但它不能是一个匿名对象，相反，它必须得是一个具名对象，否则将无法直接通过一个Lambda表达式去调用它实现的方法。   

下面看一个案例：
```java
package zeh.myjavase.code42java8.demo02;

import org.junit.Test;

public class LambdaRun {
    @Test
    public void testYuanShi() {
        //原始方式：通过匿名内部类方便的实现接口或者父类的实现子类，注意匿名内部类同时也是一个匿名对象，往往只会使用一次。
        //当然也可以通过Eat类型去接收匿名对象使之成为具名对象。
        new TestDemo02() {
            @Override
            public void eat(String name) {
                System.out.println(name + "在吃东西...");
            }

            @Override
            public void test() {
                System.out.println("吃饭");
            }
        }.eat("赵二虎");
    }

    @Test
    public void testLambda() {
        EatDemo02 ea1 = (name) -> System.out.println(name + "在吃东西");
        ea1.eat("Lambda表达式1");
        EatDemo02 ea2 = name -> System.out.println(name + "在吃东西");
        ea2.eat("Lambda表达式2");

        // ()中的参数也可以显式指定类型，但必须使用()括起来
        EatDemo02 ea3 = (String name) -> System.out.println(name + "在吃东西");
        ea3.eat("Lambda表达式3");

        //Lambda表达式如何确定这个表达式实现的就是某个对应的接口呢？
        //答案：Lambda表达式必须具名，即不能直接使用Lambda表达式直接调用目标方法，因为这样找不到。
        //只有显式的使用对应接口的类型去接收lambda表达式，这样才能确定Lambda表达式实现的对应的函数式接口。从而确定关系。
        // 像下面这种直接使用Lambda表达式去调用eat()方法，编译报错。
        // 因为Lambda本质是个具名的java对象，而不能作为匿名java对象。
        (name) -> System.out.println(name + "吃").eat();

        //实现的接口方法存在多个参数，则使用()括起来，放入形式参数名称。
        //当实现的接口方法体有return返回值时，Lambda表达式中不用return，当然也可以用return，会根据接口中定义的抽象方法的返回值类型自动确认返回类型。
        SumDemo02 sum = (a, b) -> a + b;
        System.out.println("result:" + sum.sum(12, 34));

        SumDemo02 sum1 = (c, d) -> {
            return c + d;
        };
        System.out.println("result2:" + sum1.sum(111, 22));

        //lambda表达式中可以应用类成员和局部变量（在lambda表达式内部会将这些变量隐式转换成final的）
        int t = 100;
        SumDemo02 sum2 = (c, d) -> {
            return c + d + t;
        };
        System.out.println("result3:" + sum2.sum(222, 444));
    }
}
```


