---
layout:     post
title:      玩转JdbcTemplate
subtitle:   spring对JDBC的操作也封装了一个jdbcTemplate工具
categories: [DB/JDBC/事务]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 什么是JdbcTemplate？
前面学习了Commons DbUtils，是用来简化jdbc传统的操作步骤的。  
实际上spring针对原生jdbc也封装了一个模板对象，就是JdbcTemplate。  
java中操作db最原始的方式就是纯jdbc了，每次操作db都需要加载数据库驱动、获取连接、获取PreparedStatement、执行sql、关闭PreparedStatement、关闭连接等，操作还是比较繁琐的，spring中提供了一个模块，对jdbc操作进行了封装，使其更简单。  
这个模块的核心对象就是JdbcTemplate，JdbcTemplate是spring对jdbc的封装，目的是使jdbc更加易于使用。  

# 2. JdbcTemplate源码解析
1.  类定义  
```java
public class JdbcTemplate extends JdbcAccessor implements JdbcOperations
``` 
2.  属性字段  
```java
private static final String RETURN_RESULT_SET_PREFIX = "#result-set-";
private static final String RETURN_UPDATE_COUNT_PREFIX = "#update-count-";
private NativeJdbcExtractor nativeJdbcExtractor;
private boolean ignoreWarnings = true;
private int fetchSize = 0;
private int maxRows = 0;
private int queryTimeout = 0;
private boolean skipResultsProcessing = false;
private boolean skipUndeclaredResults = false;
private boolean resultsMapCaseInsensitive = false;
```
3.  构造方法  
```java
public JdbcTemplate() {
}
public JdbcTemplate(DataSource dataSource) {
    this.setDataSource(dataSource);
    this.afterPropertiesSet();
}
public JdbcTemplate(DataSource dataSource, boolean lazyInit) {
    this.setDataSource(dataSource);
    this.setLazyInit(lazyInit);
    this.afterPropertiesSet();
}
```
4.  setter/getter  
```java
public void setNativeJdbcExtractor(NativeJdbcExtractor extractor) {
    this.nativeJdbcExtractor = extractor;
}
public NativeJdbcExtractor getNativeJdbcExtractor() {
    return this.nativeJdbcExtractor;
}
public void setIgnoreWarnings(boolean ignoreWarnings) {
    this.ignoreWarnings = ignoreWarnings;
}
public boolean isIgnoreWarnings() {
    return this.ignoreWarnings;
}
public void setFetchSize(int fetchSize) {
    this.fetchSize = fetchSize;
}
public int getFetchSize() {
    return this.fetchSize;
}
public void setMaxRows(int maxRows) {
    this.maxRows = maxRows;
}
public int getMaxRows() {
    return this.maxRows;
}
public void setQueryTimeout(int queryTimeout) {
    this.queryTimeout = queryTimeout;
}
public int getQueryTimeout() {
    return this.queryTimeout;
}
public void setSkipResultsProcessing(boolean skipResultsProcessing) {
    this.skipResultsProcessing = skipResultsProcessing;
}
public boolean isSkipResultsProcessing() {
    return this.skipResultsProcessing;
}
public void setSkipUndeclaredResults(boolean skipUndeclaredResults) {
    this.skipUndeclaredResults = skipUndeclaredResults;
}
public boolean isSkipUndeclaredResults() {
    return this.skipUndeclaredResults;
}
public void setResultsMapCaseInsensitive(boolean resultsMapCaseInsensitive) {
    this.resultsMapCaseInsensitive = resultsMapCaseInsensitive;
}
public boolean isResultsMapCaseInsensitive() {
    return this.resultsMapCaseInsensitive;
}
```

# 3. 如何使用JdbcTemplate？
1.  使用JdbcTemplate需要DataSource数据源的支持。  
 spring提供的jdbcTemplate对象是对jdbc技术的封装，通过该对象可以更加方便的操作数据库，相当于DBUtils里面的QueryRunner对象。  
 使用JdbcTemplate对象，需要数据源的支持。  
 通过JdbcTemplate源码可以知道，构造方法总共有3个：  
```java
public JdbcTemplate() {
}
public JdbcTemplate(DataSource dataSource) {
    this.setDataSource(dataSource);
    this.afterPropertiesSet();
}
public JdbcTemplate(DataSource dataSource, boolean lazyInit) {
    this.setDataSource(dataSource);
    this.setLazyInit(lazyInit);
    this.afterPropertiesSet();
}
```
第1个是空构造方法；  
第2个是有一个DataSource参数的构造方法；  
第3个是有两个参数的构造方法，分别是DataSource和lazyInit。  
除了第1个空构造方法外，剩下两个构造方法都需要传入DataSource对象。  
也就是说，我们在实例化JdbcTemplate对象时，并不一定非得传入DataSource对象。  
但实际上，JdbcTemplate方法里面去绑定数据源的方式，只提供了一种，如下：  
```java
public void setDataSource(DataSource dataSource)
```
<font color="#dc143c">而并没有提供其他类似jdbc之类的单独去绑定driverClassName、url、username、password之类的方法。由此可见，使用JdbcTemplate是离不开DataSource数据源的支持的。这也侧面验证了遵守jdbc2.0规范的重要性。</font>  

2.   使用JdbcTemplate的方式  
（1）直接硬编码，即直接new，此时的JdbcTemplate对象并没有交给Spring管理。  
```java
public class JdbcTemplateTest {
    public static void main(String[] args) {
        // 硬编码方式1
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(new BasicDataSource());
        // 硬编码方式2
        JdbcTemplate jdbcTemplate1 = new JdbcTemplate(new BasicDataSource());
        // 硬编码方式3
        JdbcTemplate jdbcTemplate2 = new JdbcTemplate(new BasicDataSource(),Boolean.FALSE);
    }
}
```

（2）spring和jdbcTemplate整合，交给spring管理，通过注解或者xml配置进行注入。  
依赖关系是：业务Dao对象依赖JdbcTemplate，JdbcTemplate依赖DataSource。  
业务Dao需要自己手动定义jdbcTemplate作为成员和对应的setter方法，如下：  
```java
public class PersonDaoImpl implements PersonDao{
    private JdbcTemplate jdbcTemplate;
    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
}
```
通过xml配置依赖关系如下：  
```xml
<!-- 配置dbcp数据源-->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
    destroy-method="close">
    <!-- 通过property属性注入依赖成员 -->
    <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver" />
    <property name="url">
        <value>jdbc:oracle:thin:@localhost:1521:orcl</value>
    </property>
    <property name="username" value="oracle" />
    <property name="password" value="oracle" />
</bean>
<!-- 配置jdbcTemplate -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>
<!-- 配置dao -->
<bean id="personDao" class="demo08.myspring.model.jdbc.dao.impl.PersonDaoImpl"
      depends-on="jdbcTemplate" init-method="initDatabase">
    <property name="jdbcTemplate">
        <ref bean="jdbcTemplate" />
    </property>
</bean>
```

（3）spring和jdbcTemplate整合，交给spring管理，引入JdbcDaoSupport更加方便的直接管理JdbcTemplate对象。  
通过（2）可以看出，自己编写一个业务Dao，交给Spring管理，需要有如下依赖关系：  
<font color="#dc143c">业务Dao依赖JdbcTemplate对象；  
JdbcTemplate对象依赖DataSource对象。</font>  
这种依赖层次有点多，如果能够直接在业务Dao中注入DataSource对象就很方便了。  
spring提供了一个Support类来实现这种便捷，这个类是 <font color="#dc143c">JdbcDaoSupport</font> 。  
业务Dao必须继承JdbcDaoSupport类，才能持有JdbcTemplate对象，这样就不用自己在Dao类中添加JdbcTemplate变量了：  
```java
public class PersonDaoImpl extends JdbcDaoSupport implements PersonDao
```  
通过xml配置依赖关系如下：  
```xml
<!-- 配置dbcp数据源-->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
    destroy-method="close">
    <!-- 通过property属性注入依赖成员 -->
    <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver" />
    <property name="url">
        <value>jdbc:oracle:thin:@localhost:1521:orcl</value>
    </property>
    <property name="username" value="oracle" />
    <property name="password" value="oracle" />
</bean>
<!-- 配置dao -->
<bean id="personDao" class="demo08.myspring.model.jdbc.dao.impl.PersonDaoImpl"
      depends-on="dataSource" init-method="initDatabase">
    <property name="dataSource">
        <ref bean="dataSource" />
    </property>
</bean>
```

# 4. 通过硬编码操作JdbcTemplate的API
我们直接通过硬编码的方式去构建JdbcTemplate。  
 ## 4.1 JdbcTemplate使用步骤
 1.  创建数据源DataSource。  
 2.  创建JdbcTemplate对象，new JdbcTemplate(dataSource);  
 3.  调用JdbcTemplate的方法操作db，如增删改查。  
 ```java
package lurenjia_jdbcTemplate.study01;
import org.apache.commons.dbcp.BasicDataSource;
import javax.sql.DataSource;
/**
 * @Date 2021/9/23 19:25
 */
public class DataSourceBuild {
    public static DataSource getDataSource() {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName("oracle.jdbc.driver.OracleDriver");
        dataSource.setUrl("jdbc:oracle:thin:@localhost:1521:orcl");
        dataSource.setUsername("oracle");
        dataSource.setPassword("oracle123");
        return dataSource;
    }
}
```
```java
package lurenjia_jdbcTemplate.study01;
import org.junit.Test;
import org.springframework.jdbc.core.JdbcTemplate;
import javax.sql.DataSource;
import java.util.List;
import java.util.Map;
/**
 * @Date 2021/9/23 18:59
 */
public class JdbcTemplateTest {
    @Test
    public void test() {
        // 1.创建数据源DataSource
        DataSource dataSource = DataSourceBuild.getDataSource();
        // 2.创建jdbcTemplate对象，new JdbcTemplate(dataSource)
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        // 3.调用JdbcTemplate的方法操作db，如增删改查
        List<Map<String, Object>> allMaps = jdbcTemplate.queryForList("select * from users");
        System.out.println(allMaps);
    }
}
```
输出：  
```
[{ID=12345, NAME=TestInsert, AGE=19, SEX=男, HOBBY=编程}, {ID=12333, NAME=Mybatis, AGE=19, SEX=男, HOBBY=Mybatis系列}, {ID=12334, NAME=mySql, AGE=21, SEX=女, HOBBY=唱歌，跳舞}]
```
上面查询返回了oracle中person表中所有的记录数据，返回了一个集合List列表，集合中每一个元素是一个Map，Map表示一行记录，key为列名，value为列对应的值。  
是不是觉得特别的方便，只需要<font color="#dc143c">jdbcTemplate.queryForList("select * from person");</font>这么简单的一行代码，数据就被获取到了。  
下面继续探索更强大更好用的功能。  

## 4.2 增加、删除、修改操作
<font color="#dc143c">JdbcTemplate中以update开头的方法，用来执行增、删、改操作</font> ，下面看几个常用的。</br>  
<font color="#dc143c"><b><u>无参情况</u></b></font>  
<b>Api</b>  
```java
public int update(final String sql)
```
<b>案例</b>  
```java
@Test
public void test1() {
    DataSource dataSource = DataSourceBuild.getDataSource();
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    int updateRows = jdbcTemplate.update("insert into users(id,name,age,sex,hobby) values ('12345','TestInsert',19,'男','编程')");
    System.out.println("影响行数：" + updateRows);
}
```
<b>运行</b>  
```
影响行数：1
```

<font color="#dc143c"><b><u>有参情况1</u></b></font>  
<b>Api</b>  
```java
public int update(String sql, Object... args)  
```
<b>案例</b>  
<font color="#dc143c">sql中使用?作为占位符</font>  
```java
@Test
public void test2() {
    DataSource dataSource = DataSourceBuild.getDataSource();
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    int updateRows = jdbcTemplate.update("insert into users(id,name,age,sex,hobby) values (?,?,?,?,?)", "12333", "Mybatis", 19, "男", "Mybatis系列");
    System.out.println("影响行数：" + updateRows);
}
```
<b>运行</b>  
```
影响行数：1
```

<font color="#dc143c"><b><u>有参情况2</u></b></font>  
<b>Api</b>  
```java
public int update(String sql, PreparedStatementSetter pss)
```
通过PreparedStatementSetter来设置参数，是一个函数式接口，内部有个setValues方法会传递一个PreparedStatement参数，我们可以通过这个参数手动的设置参数的值。  
<b>案例</b>  
```java
@Test
public void test3(){
    DataSource dataSource = DataSourceBuild.getDataSource();
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    int updateRows = jdbcTemplate.update("insert into users(id,name,age,sex,hobby) values (?,?,?,?,?)", new PreparedStatementSetter() {
        @Override
        public void setValues(PreparedStatement preparedStatement) throws SQLException {
            preparedStatement.setString(1,"12334");
            preparedStatement.setString(2,"mySql");
            preparedStatement.setInt(3,21);
            preparedStatement.setString(4,"女");
            preparedStatement.setString(5,"唱歌，跳舞");
        }
    });
    System.out.println("影响行数：" + updateRows);
}
```
<b>运行</b>  
```
影响行数：1
```

## 4.3 获取自增列的值
<b>Api</b>  
```java
public int update(PreparedStatementCreator psc, final KeyHolder generatedKeyHolder)
```
<b>案例</b>  
```java
@Test
public void test4() {
    DataSource dataSource = DataSourceBuild.getDataSource();
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    final String sql = "insert into users(id,name,age,sex,hobby) values (?,?,?,?,?)";
    KeyHolder keyHolder = new GeneratedKeyHolder();
    int rowCount = jdbcTemplate.update(new PreparedStatementCreator() {
        @Override
        public PreparedStatement createPreparedStatement(Connection connection) throws SQLException {
            // 手动创建PreparedStatement，注意第二个参数：Statement.RETURN_GENERATED_KEYS
            PreparedStatement preparedStatement = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
            preparedStatement.setString(1, "12335");
            preparedStatement.setString(2, "mySql");
            preparedStatement.setInt(3, 21);
            preparedStatement.setString(4, "女");
            preparedStatement.setString(5, "唱歌，跳舞");
            return preparedStatement;
        }
    }, keyHolder);
    System.out.println("新记录id：" + keyHolder.getKey().intValue());
}
```

## 4.4 批量增删改操作
<b>Api</b>  
```java
public int[] batchUpdate(final String[] sql)
public int[] batchUpdate(String sql, List<Object[]> batchArgs)
public int[] batchUpdate(String sql, List<Object[]> batchArgs, int[] argTypes)
```
<b>案例</b>  
```java
@Test
public void test5() {
    DataSource dataSource = DataSourceBuild.getDataSource();
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    final String sql = "insert into users(id,name,age,sex,hobby) values (?,?,?,?,?)";
    List<Object[]> list = Arrays.asList(
            new Object[]{"13001","Mybatis", 19, "1", "Mybatis系列"},
            new Object[]{"13002","Mybatis", 19, "1", "Mybatis系列"},
            new Object[]{"13003","Mybatis", 19, "1", "Mybatis系列"},
            new Object[]{"13004","Mybatis", 19, "1", "Mybatis系列"}
    );
    int[] updateRows = jdbcTemplate.batchUpdate("insert into users(id,name,age,sex,hobby) values (?,?,?,?,?)", list);
    for(int updateRow : updateRows){
        System.out.println(updateRow);
    }
}
```

## 4.4 查询操作
<font color="#dc143c"><b><u>查询一列单行</u></b></font>  
<b>Api</b>  
```java
// sql：执行的sql，如果有参数，使用参数占位符?
// requiredType：返回的一列数据对应的java类型，比如String
// args：?占位符对应的参数列表
public <T> T queryForObject(String sql, Class<T> requiredType, Object... args)
```
<b>案例</b>  
```java
@Test
public void test6() {
    DataSource dataSource = DataSourceBuild.getDataSource();
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    String name = jdbcTemplate.queryForObject("select name from users where id = ?", String.class, "13001");
    System.out.println(name);
}
```
<b>运行</b>  
```
Mybatis
```
<b>使用注意：</b>  
<font color="#dc143c">如果queryForObject中sql查询无结果时，会报错</font>  
比如id=0的记录不存在  
```java
@Test
public void test7() {
    DataSource dataSource = DataSourceBuild.getDataSource();
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    String name = jdbcTemplate.queryForObject("select name from users where id = ?", String.class, "0");
    System.out.println(name);
}
```
运行，会弹出一个异常 <font color="#dc143c">EmptyResultDataAccessException</font>，期望返回一条记录，但实际上却没有找到记录，和期望结果不符，所以报错了。  
```
org.springframework.dao.EmptyResultDataAccessException: Incorrect result size: expected 1, actual 0
    at org.springframework.dao.support.DataAccessUtils.requiredSingleResult(DataAccessUtils.java:71)
    at org.springframework.jdbc.core.JdbcTemplate.queryForObject(JdbcTemplate.java:730)
    at org.springframework.jdbc.core.JdbcTemplate.queryForObject(JdbcTemplate.java:749)
```  
这种如何解决呢，需要用到查询多行的方式来解决了，即下面要说到的<font color="#dc143c">queryForList</font>相关的方法，无结果的时候返回一个空的List，我们可以在这个空的list上面做文章。  
<font color="#dc143c"><b><u>查询一列多行</u></b></font>  
<b>Api</b>  

# 5. 通过spring整合JdbcTemplate去操作API
我们通过业务Dao继承JdbcDaoSupport类的方式，直接将JdbcTemplate交给Spring管理。  
代码结构：  
![][jdbcTemplate代码结构]
resource资源配置文件：  
![][resource结构]  
<font color="#dc143c"><b>定义pojo Person类：</b></font>  
```java
package demo08.myspring.model.jdbc.pojo;
import java.util.Date;
// POJO对象实体-属于Model层，也是DAO层。简单的Person bean。
// 一般自定义的bean如果是有状态的bean，则不要交给spring容器负责实例化，因为默认实例化的对象是单例，存在非线程安全。
// 建议：自定义的bean作为方法变量进行传递，这种bean尤其见于这些用来保存状态并传输的pojo对象。
public class Person {
    private int id;
    private String name;
    private String sex;
    private int age;
    private Date birthday;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public Date getBirthday() {
        return birthday;
    }
    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
}
```
<font color="#dc143c"><b>定义Service接口：</b></font>  
```java
package demo08.myspring.model.jdbc.service;
import demo08.myspring.model.jdbc.pojo.Person;
import java.util.List;
public interface PersonService {
    public abstract String name(Integer id);
    public abstract void insert(Person per);
    public abstract int count();
    public abstract List<Person> list();
}
```
<font color="#dc143c"><b>Service接口实现类：</b></font>  
```java
package demo08.myspring.model.jdbc.service.impl;
import java.util.List;
import demo08.myspring.model.jdbc.dao.PersonDao;
import demo08.myspring.model.jdbc.pojo.Person;
import demo08.myspring.model.jdbc.service.PersonService;
// Service层除了保存dao对象和无状态的bean对象外，不要保存有状态的bean。
public class PersonServiceImpl implements PersonService {
    private PersonDao personDao;
    public PersonDao getPersonDao() {
        return personDao;
    }
    public void setPersonDao(PersonDao personDao) {
        this.personDao = personDao;
    }
    @Override
    public String name(Integer id) {
        return personDao.getPersonName(id);
    }
    @Override
    public void insert(Person per) {
        this.personDao.insertPerson(per);
    }
    @Override
    public int count() {
        return this.personDao.getPersonCount();
    }
    @Override
    public List<Person> list() {
        return this.personDao.listPersons();
    }
}
```
<font color="#dc143c"><b>定义Dao接口：</b></font>  
```java
package demo08.myspring.model.jdbc.dao;
import java.util.List;
import demo08.myspring.model.jdbc.pojo.Person;
// 定义DAO层接口，DAO层主要负责处理数据。
public interface PersonDao {
    // 根据id获得person的姓名
    public abstract String getPersonName(int id);
    // 添加Person对象
    public abstract void insertPerson(Person per);
    // 返回person的总数目
    public abstract int getPersonCount();
    // 返回所有的person对象
    public abstract List<Person> listPersons();
}
```
<font color="#dc143c"><b>定义Dao实现类：通过继承JdbcDaoSupport的方式整个JdbcTemplate</b></font>  
```java
package demo08.myspring.model.jdbc.dao.impl;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Map;
import org.springframework.jdbc.core.support.JdbcDaoSupport;
import demo08.myspring.model.jdbc.dao.PersonDao;
import demo08.myspring.model.jdbc.pojo.Person;
// spring整合JDBC。
// DAO层接口的实现类：定义的dao实现必须继承JdbcDaoSupport，因为要获取JdbcTemplate模板。
// spring管理jdbc后一切操作通过jdbcTemplate去操作。
// 注意：spring创建的JdbcTemplate模板对象每次返回的都是单例对象，但是使用ThreadLocal来保证Connection对象的线程安全。
// spring整合mybatis或者hibernate后将不再使用JdbcTemplate对象，而是直接使用对mybatis和hibernate的封装对象。
// 在spring中，所有的bean都是默认单例的，不要在service和dao中定义成员变量，避免非线程安全。
public class PersonDaoImpl extends JdbcDaoSupport implements PersonDao {
    // 扩展一个初始化方法，用于创建数据表。
    // spring将jdbc的操作封装成了jdbcTemplate模板对象。
    // 该方法在将当前Dao交给spring容器管理时，通过在配置文件中指定init-method="initDatabase"进行初始化调用。
    public void initDatabase() {
        // 拼接sql语句
        String createTableSql = "create table table_person(id number(12),name varchar(21),age number(12),sex varchar(12),birthday Date)";
        try {
            this.getJdbcTemplate().execute(createTableSql);
        } catch (Exception e) {
            // 一旦抛出异常直接忽略，一般是因为数据表已将存在了。
            ;
        }
    }
    // 插入一条记录
    public void insertPerson(Person per) {
        String sql = "insert into table_person(id,name,sex,age,birthday) values(?,?,?,?,?)";
        this.getJdbcTemplate().update(
                sql,
                new Object[]{per.getId(), per.getName(), per.getSex(),
                        per.getAge(), per.getBirthday()});
    }
    // 根据id号查询姓名。
    public String getPersonName(int id) {
        String sql = "select name from table_person where id = " + id;
        return (String) this.getJdbcTemplate()
                .queryForObject(sql, String.class);
    }
    // 查询总数。
    public int getPersonCount() {
        String sql = "SELECT COUNT(1) FROM TABLE_PERSON";
        return this.getJdbcTemplate().queryForInt(sql);
    }
    public List<Person> listPersons() {
        String sql = "select id,name,sex,age,birthday from table_person";
        List<Map<String, Object>> list = this.getJdbcTemplate().queryForList(
                sql);
        List<Person> personList = new ArrayList<Person>();// 实例化list容器
        for (Map<String, Object> row : list) {
            // 封装成person对象
            Person per = new Person();
            per.setId((Integer) row.get("id"));
            per.setName((String) row.get("name"));
            per.setAge((Integer) row.get("age"));
            per.setSex((String) row.get("sex"));
            per.setBirthday((Date) row.get("birthday"));
            personList.add(per);
        }
        // 返回person的list集合
        return personList;
    }
}
```
<font color="#dc143c"><b>配置如下：</b></font>  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:redis="http://www.springframework.org/schema/redis"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.1.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-3.1.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-3.1.xsd
        http://www.springframework.org/schema/cache
        http://www.springframework.org/schema/cache/spring-cache-3.1.xsd
        http://www.springframework.org/schema/redis http://www.springframework.org/schema/redis/spring-redis-1.0.xsd">
    <!-- spring整合jdbc，一切操作通过spring的JdbcTemplate对象操作 -->
    <!-- 配置dbcp数据源，此处配置的是dbcp数据源的实现,通过Spring自带的beanFactory进行bean的实例化 -->
    <!-- 执行完bean dataSource的所有方法后执行destroy-method属性指定的方法 -->
    <!-- 此处实例化的是dbcp这个数据源对象 -->
    <!-- 销毁连接池对象时执行close关闭方法 -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close">
        <!-- 通过property属性注入依赖成员 -->
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver" />
        <property name="url">
            <value>jdbc:oracle:thin:@localhost:1521:orcl</value>
        </property>
        <property name="username" value="oracle" />
        <property name="password" value="oracle" />
    </bean>
    <!-- 一般对于dao对象的实例化采用xml配置的方式 -->
    <!-- 实例化PersonDaoImpl这个bean时，必须先实例化dataSource数据源这个bean，PersonDaoImpl对象实例化好后首先执行的方法是initDatabase() -->
    <bean id="personDaoImpl" class="demo08.myspring.model.jdbc.dao.impl.PersonDaoImpl"
        depends-on="dataSource" init-method="initDatabase">
        <property name="dataSource">
            <ref bean="dataSource" />
        </property>
    </bean>
    <!-- 实例化service对象 -->
    <bean id="personServiceImpl"
        class="demo08.myspring.model.jdbc.service.impl.PersonServiceImpl">
        <property name="personDao" ref="personDaoImpl" />
    </bean>
</beans>
```
<font color="#dc143c"><b>测试运行：</b></font>  

```java
package demo08.myspring.model.jdbc.run;
import java.util.Date;
import org.junit.Before;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import demo00.myspring.common.initconfig.LoadSpringConfigFactory;
import demo08.myspring.model.jdbc.pojo.Person;
import demo08.myspring.model.jdbc.service.PersonService;
public class PersonServiceJunit {
    private ApplicationContext applicationContext;
    private PersonService personService;
    @Before
    public void loadApplicationContext() {
        // 加载spring配置
        applicationContext = LoadSpringConfigFactory.getApplicationContext("spring/demo08-applicationContext-dao.xml");
        // 参数1: bean name/bean id
        // 参数2(可选): 可以指定生成对象类型,如果不填此参数,需进行强转 两种方式都可以获取!
        // PersonDao personDao = applicationContext.getBean("personDaoImpl", PersonDao.class);
        // 不指定参数2则需要强制转换
        personService = (PersonService) applicationContext.getBean("personServiceImpl");
    }
    // 通过JdbcTemplate对象插入数据成功。
    @Test
    public void testInsert() {
        Person per = new Person();
        per.setId(12);
        per.setName("赵二虎");
        per.setSex("男");
        per.setAge(25);
        per.setBirthday(new Date());
        personService.insert(per);
        System.out.println("插入表数据成功...");
    }
    // 查询数据表的总条数。
    @Test
    public void testCount() {
        Integer count = this.personService.count();
        System.out.println("总数为：" + count);
    }
    // 根据id查询目标姓名。
    @Test
    public void queryName() {
        String name = this.personService.name(12);
        if (name == null) {
            System.out.println("目标任务不存在...");
        } else {
            System.out.println("查询到的姓名是:" + name);
        }
    }
}
```

  [jdbcTemplate代码结构]: data:image/png;base64,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
  [resource结构]: data:image/png;base64,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


