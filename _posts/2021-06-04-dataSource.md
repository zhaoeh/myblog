---
layout:     post
title:      dataSource
subtitle:   数据源
date:       2021-06-04
author:     zhaoeh
header-img: img/post-bg-myself7.jpg
catalog: true
tags:
    - DB/JDBC/事务
---

# 1. 什么是数据源？
<font color="#dc143c">在了解什么是数据源之前，先回顾之前的一个案例：</font>  
在使用前，为了避免数据库的各种信息硬编码，先手动实现一个通用的读取数据库配置的工具类：  
```java
package javase.demo26.db.util;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.util.Properties;
public class CommonJdbcLoadUtil {
    private static String DBDRIVER;
    private static String DBURL;
    private static String DBUSER;
    private static String DBPASS;
    private static final Properties PROPERTIES = new Properties();
    static {
        try {
            // idea中如果进行了模块化处理后，则所有的文件的路径实际上都不是相对原来的项目根路径了，而是相对于最顶层的项目路径了，因为原来myeclipse中的项目可能在idea中已经变成了模块了，所以要加上自己的模块的路径。
            PROPERTIES.load(new FileInputStream("myjavaSE/resource/jdbc.properties"));
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        DBDRIVER = PROPERTIES.getProperty("jdbc.driver", "oracle.jdbc.driver.OracleDriver");
        DBURL = PROPERTIES.getProperty("jdbc.url", "jdbc:oracle:thin:@localhost:1521:orcl");
        DBUSER = PROPERTIES.getProperty("jdbc.username", "oracle");
        DBPASS = PROPERTIES.getProperty("jdbc.password", "oracle1234");
    }
    public static String getDbdriver() {
        return DBDRIVER;
    }
    public static String getDburl() {
        return DBURL;
    }
    public static String getDbuser() {
        return DBUSER;
    }
    public static String getDbpass() {
        return DBPASS;
    }
}
```
resource目录下配置jdbc.properties文件用于指定数据库的信息：  
```
jdbc.driver=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@localhost:1521:orcl
jdbc.username=oracle
jdbc.password=oracle1234
```
从上面的案例可以看到，CommonJdbcLoadUtil这个工具类是用来专门读取配置文件中配置的数据库的各种信息的。  
实际上，在我们自己编写这个通用类之前，我们一般会将数据库的驱动程序、名称、密码等信息硬编码在代码中，硬编码导致数据库的配置信息很不优雅，看起来很乱。  
因此，搞一个专门的类和对应的配置文件，来专门保存和读取数据库的各种信息。  
获取了数据库的各种信息之后，才能进行驱动加载、创建连接、实例化statement等一系列动作来连接数据库，从而进行SQL的各种操作。  
我们把这种专门用来描述特定数据库的各种信息的配置文件和解析类称之为<font color="#dc143c">数据源</font>。  
<font color="#dc143c">数据源，就是描述数据的来源，它存储了建立数据库连接所需要的所有信息。直观点来说，就是通过硬编码或者配置文件等描述对应数据库的各种信息。</font>  
```
数据源一般用于描述建立数据库连接所需要的各种信息：
数据库名称
数据驱动
用户名
密码
...
通过数据源对数据库信息进行抽象描述，从而创建数据库连接，一般一个数据源对应一个数据库。
```

# 2. 获取数据库连接
通过数据源描述的数据库的各种信息，就可以与其描述的指定数据库创建数据库连接：  
```java
 // 1.加载驱动程序
Class.forName(CommonJdbcLoadUtil.getDbdriver());
// 2.连接oracle数据库服务器，返回连接对象。注意连接数据库服务器时用户名和密码一定要正确。
con = DriverManager.getConnection(CommonJdbcLoadUtil.getDburl(),
        CommonJdbcLoadUtil.getDbuser(),
        CommonJdbcLoadUtil.getDbpass());
```

# 3. 数据源和连接池的关系
实际上由于java发展的太快，最早的JDBC规范有很多不太合理的地方。  
截止到目前，我们知道了数据源实际上就是用来描述指定数据库的各种信息的。这些描述你可以硬编码到代码中，也可以统一存储到某个指定文件中然后使用特定工具类进行解析。总而言之，用于描述数据库信息的东西，我们就称它为数据源。  
有了数据源之后，我们就可以通过数据源去加载驱动，然后通过 DriverManager 去获取数据库连接。  
然而，随着JDBC的发展，逐渐显示出了弊端。比如，总是将数据库各种信息硬编码或者任由开发者去编写到某些指定文件中，看起来很乱，并没有规范；比如，执行一个SQL，总是需要通过DriverManager去获取数据库连接，执行完毕后然后释放连接。  
这一系列随之而来的弊端，导致了在JDBC2.0版本发布时，内置了一个接口：<font color="#dc143c">javax.sql.DataSource</font>  
SUN期望通过javax.sql.DataSource接口来统一描述数据源，并且提供一种全新获取数据库连接的方式（实际上它的底层和DriverManager获取Connection对象的方式相同）。  
<font color="#dc143c">因此，我们需要知道，JDBC2.0发布了javax.sql.DataSource接口来规范数据源，并且创建Connection对象，该接口需要各个厂商自己负责实现。</font>  

# 4. javax.sql.DataSource接口
<font color="#dc143c">最初的数据库连接方式：</font>  
```java
String URL = "jdbc:oracle:thin:@amrood:1521:EMP";
String USER = "username";
String PASS = "password"
Connection conn = DriverManager.getConnection(URL, USER, PASS);
```
可以看到，曾经获取数据库连接的代码还需要使用 DriverManager ,<font color="#dc143c">DriverManager.getConnection</font>是通过数据库驱动直接与数据库建立物理连接。  
<font color="#dc143c">建立数据库连接以及关闭连接属于耗费时间的事情</font>，如果业务每次进行SQL查询都使用此方式，将会产生较大的性能开销。  
因此，我们想着，既然线程对象都能够放在线程池中，那么数据库的Connection对象也能够放在一个池子中啊，这就是池化技术。所以，数据库连接池的设计开始出现了。  
线程池、连接池，这些池化技术的核心思想就是<font color="#dc143c">空间换时间</font>。  

鉴于JDBC发展的需要，因此SUN公司在JDBC2.0发布的时候，指定了一个新的接口规范，javax.sql.DataSource，用来规范化访问数据库的流程。  
SUN想通过javax.sql.DataSource的方式来替代原来的DriverManager获取连接的方式。  
当你想要获取数据库连接时，通过DataSource去获取即可，用完了也不用关闭连接对象等。  
然而SUN只是制定了该规范，至于究竟如何产生Connection对象，SUN并未做明确的要求，需要第三方自己去实现。  
因此，实际上DataSource只是描述数据源的规范化接口，它提供了快速获取Connection对象的规范，至于这个Connection对象是通过DriverManager去产生，还是直接通过数据库驱动去产生，是放置在连接池中，还是没有实现连接池，DataSource一律不作要求。  

javax.sql.DataSource接口：  
```java
public interface DataSource  extends CommonDataSource,Wrapper {
  Connection getConnection() throws SQLException;
  Connection getConnection(String username, String password) throws SQLException;
}
```
总结：
DataSource接口是SUN为了更加统一的描述数据源和方便实现连接池而推出的一种规范，需要各个第三方自己去实现。  
说白了，DataSource并不是连接池，它只是对数据源的规范描述。但是可以使用DataSource方便的实现连接池，因为可以使用DataSource来代替DriverManager来方便的获取数据库连接对象。  

注意：
DriverManager 只是JDBC1.0版本用来调用数据库驱动的工具包。  
JDBC2.0版本推出的DataSource规范之后，典型的像第三方 DruidDataSource 就没有依赖DriverManager，而是在自己实现类中直接调用了数据库驱动。  
这里只是重点强调，数据库连接不一定是使用DriverManager创建的（但实际上连接池中的连接对象还是从DriverManager或者类似组件中创建的）。  
使用javax.sql.DataSource的好处：  
有了DataSource后，数据库连接、用户名、密码等都有了统一的规范进行管理，它们作为DataSource属性的一部分，并且将数据库驱动名称填充，底层自动加载。  
