---
layout:     post
title:      mybatis和原生spring整合
subtitle:   mybatis集成到原生spring中，很多自己的特性都通过spring替代了
date:       2021-11-10
author:     zhaoeh
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - mybatis
---

# 1. 配置mybatis全局配置文件
和spring整合后，mybatis全局配置文件中的很多配置都交给spring去管理了。   
mybatis_spring/sqlmap/sqlmap-config.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- 注意：本来mybatis的全局配置中的事务数据源等，现在都交给spring进行统一整合管理即可。-->
    <!-- 别名定义 -->
    <typeAliases>
        <!-- 批量别名定义 指定包名，mybatis自动扫描包中的po类，自动定义别名，别名就是类名（首字母大写或小写都可以） -->
        <package name="zeh.mybatis.spring.po"/>
    </typeAliases>

    <!-- 加载 映射文件 -->
    <mappers>
        <!-- 方式1：加载单个mapper.xml，mapper.xml的位置可以自定义 -->
        <mapper resource="mybatis_spring/mapper/mapper.xml"/>

        <!-- 批量加载mapper.xml 指定mapper接口的包名 -->
        <!-- mybatis自动扫描包下边所有mapper接口进行加载  -->
        <!-- 遵循一些规范：需要将mapper接口类名和mapper.xml映射文件名称保持一致 -->
        <!-- 且在一个目录 中 上边规范的前提是：使用的是mapper代理方法 -->
        <package name="zeh.mybatis.code09mybatisspring.mapper"/>

        <!-- 和spring整合后，使用mapper扫描器，上述不需要配置了，但是上述的规范依然要遵守。 -->
        <!-- 注意：实际上sqlmap-config.xml中的扫描是为了加载mapper.xml文件从而加载，而spring配置中是为了生成mapper代理对象。 -->
        <!-- 这两者其实不同，但是spring的mapper扫描器范围更大，自动包含了sqlmap-config中的扫描器。 -->
    </mappers>

</configuration>

```

# 2. 配置数据源配置
mybatis_spring/db.properties
```properties
# oracle 数据库连接配置
jdbc.driver=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@localhost:1521:orcl
jdbc.username=oracle
jdbc.password=oracle

```

# 3. 配置mapper.xml
这个mapper.xml是配置在resource里面的，需要在全局配置只能怪显式引用。   
mybatis_spring/mapper/mapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- namespace命名空间，作用就是对sql进行分类化管理，理解sql隔离 注意：使用mapper代理方法开发，namespace有特殊重要的作用 -->
<mapper namespace="mybatisSpring">

    <!-- 在 映射文件中配置很多sql语句 -->
    <!-- 需求：通过id查询用户表的记录 -->
    <!-- 通过 select执行数据库查询 -->
    <!-- id：标识 映射文件中的 sql 将sql语句封装到mappedStatement对象中，所以将id称为statement的id -->
    <!-- parameterType：指定输入 参数的类型，这里指定int型 #{}表示一个占位符号 #{id}：其中的id表示接收输入 的参数，参数名称就是id，如果输入 -->
    <!-- 参数是简单类型，#{}中的参数名可以任意，可以value或其它名称 -->
    <!-- resultType：指定sql输出结果 的所映射的java对象类型，select指定resultType表示将单条记录映射成的java对象。 -->
    <select id="findUserByIdBySpring" parameterType="int" resultType="zeh.mybatis.spring.po.User">
		select * from users where id=#{value}
	</select>

    <insert id="insertUserBySpring">
        insert into users (ID, NAME, AGE, SEX, HOBBY)
        values (1 ,'大西洋', 22, '男', '打球')
    </insert>

    <insert id="insertUserABySpring">
        insert into users_a (ID, NAME, AGE, SEX, HOBBY)
        values (1 ,'userA', 22, '男', '打球')
    </insert>

    <insert id="insertUserBBySpring">
        insert into users_b (ID, NAME, AGE, SEX, HOBBY)
        values (1 ,'userB', 22, '男', '打球')
    </insert>

</mapper>
```

# 4. 配置spring配置文件
mybatis_spring/spring/applicationContext.xml   
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-3.2.xsd
		http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-3.2.xsd ">

    <!-- spring主要配置： -->
    <!-- 数据源 -->
    <!-- 注入sqlSessionFactory的bean实例 -->
    <!-- 注入dao对象 -->
    <!-- 自动扫描并实例化mapper代理对象 -->
    <!-- ...总的来说，spring可以通过ioc注入一切用户想要注入的bean实体对象； -->

    <!-- 加载db.properties配置文件（不再使用mybatis自身提供的加载配置） 加载的路径可以使用通配符加载多个db配置，即加载多个数据源
        如："classpath:/datasource/*.properties" -->
    <context:property-placeholder location="classpath:mybatis_spring/db.properties"/>

    <!-- spring注解扫描器-->
    <context:component-scan base-package="zeh.mybatis.code09mybatisspring"/>

    <!-- 使用spring管理数据源。 -->
    <!-- 什么是数据源？ -->
    <!-- 数据源就是数据库连接池，即将数据库的很多操作步骤简化封装成了数据源对象。 -->
    <!-- 该对象中保存了连接数据库的各个属性并且会自动实例化Connection对象进行池化。 -->
    <!-- 所以说，数据源本身是一个封装了很多数据库属性的对象 数据源是需要实现的，由不同的方式实现，比如dbcp,c3p0,odbc等。 -->
    <!-- 实现过程就是将数据库的指定配置实例化成Connection对象进行存储。 -->
    <!-- 数据源，使用dbcp；class属性是dbcp的数据源实现。 -->
    <bean id="dbcpdataSource" class="org.apache.commons.dbcp.BasicDataSource"
          destroy-method="close">
        <!--JDBC连接四大配置参数 -->
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <!-- 数据源连接池参数 -->
        <property name="maxActive" value="10"/>
        <property name="maxIdle" value="5"/>
    </bean>

    <!-- 使用spring管理mybatis事务：不论是采用spring的声明式事务还是编程式事务，要对事务进行管理，必须进行如下配置。-->
    <!-- @Transactional注解只是对事务的一种标识，真正通过反射管理事务的正是 DataSourceTransactionManager，所以一定要配置事务管理器，否则事务不生效。-->
    <bean name="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dbcpdataSource"></property>
    </bean>
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>

    <!-- 配置SqlSessionFactoryBean -->
    <!-- 使用spring管理mybatis的sqlSessionFactory配置-->
    <!-- Spring管理mybatis的工厂bean 通过spring的ioc注入sqlSessionFactoryBean对象，class路径是从spring-mybatis的整合jar包中找到的 -->
    <!-- 此处就是spring和mybatis的整合，借助的是整合包 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 加载mybatis的配置全局配置文件SqlMapConfig.xml文件，configLocation是整个包中提供的成员属性 -->
        <property name="configLocation" value="mybatis_spring/sqlmap/sqlmap-config.xml"/>
        <!-- 数据源， dataSource是成员属性名称 -->
        <property name="dataSource" ref="dbcpdataSource"/>
    </bean>

    <!-- mybatis采用原始方式开发dao模式，需要自己编写dao接口和dao实现类 -->
    <bean id="userDao" class="zeh.mybatis.code09mybatisspring.dao.MarshallUserDaoImpl">
        <!-- 必须向dao实现类中注入sqlSessionFactory对象 -->
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>

    <!-- mapper配置 MapperFactoryBean： -->
    <!-- 根据mapper接口生成代理对象，交给spring进行bean的实例化 -->
    <!-- 此种方式的缺陷：真正开发中有很多mapper接口，name如果都按照这种方式单一性实例化mapper代理，那么将很不方便 -->
    <!-- 所以，将使用批量扫描，自动扫描目标包下面的mapper接口并且实例化代理对象 -->
    <!-- mapper代理的方式，只定义了接口（自动生成的实现），使用spring框架要想得到代理实现，需要其提供的MapperFactoryBean类得到mapper代理对象-->
    <bean id="userMapperTest" class="org.mybatis.spring.mapper.MapperFactoryBean">
        <!-- mapperInterface属性指定mapper接口 -->
        <property name="mapperInterface" value="zeh.mybatis.code09mybatisspring.mapper.MarshallUserMapper"/>
        <!-- sqlSessionFactory属性引用上面实例化好的sqlSessionFactory对象，从而得到selSession对象 -->
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>

    <!-- mapper批量扫描，从mapper包中扫描出mapper接口，自动创建mapper代理对象并且在spring容器中注册 -->
    <!-- 遵循规范：将mapper.java和mapper.xml映射文件名称保持一致，且在一个目录中 。 -->
    <!-- 自动扫描出来的mapper的bean的id为mapper类名（首字母小写） -->
    <!-- MapperScannerConfigurer这个bean专门用来扫描mybatis的mapper包；然后存储动态生成的mapper代理对象。 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 指定扫描的包名 如果扫描多个包，每个包中间使用半角逗号分隔；不能使用通配符 -->
        <property name="basePackage" value="zeh.mybatis.spring.mapper"/>
        <!-- 注意下面的name属性在MapperScannerConfigurer这个bean中是String类型的，而不是引用类型，所有要使用value指定，而不能用ref。 -->
        <!-- 但是MapperScannerConfigurer中也有sqlSessionFactory这个对象的依赖，但是不要注入这个对象，而是注入sqlSessionFactoryBeanName。 -->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>

</beans>
```
# 5. java代码开发
## 5.1 mybatis和spring整合后，依然使用原始方式开发DAO
原始方式开发DAO，mybatis的全局配置文件中依然需要手动依赖mapper.xml。   
zeh.mybatis.code09mybatisspring.dao.MarshallUserDao   
```java
package zeh.mybatis.code09mybatisspring.dao;

import zeh.mybatis.code00common.po.more.MarshallUser;

/**
 * mybatis原始方式实现dao
 *
 * @author zhaoeh
 */
public interface MarshallUserDao {

    // 根据id查询用户信息
    public MarshallUser findUserById(int id) throws Exception;

    //插入user信息
    public int insertUserUseSpring() throws Exception;

    public int insertUserA() throws Exception;

    public int insertUserB() throws Exception;
}

```
zeh.mybatis.code09mybatisspring.dao.MarshallUserDaoImpl
```java
package zeh.mybatis.code09mybatisspring.dao;

import org.apache.ibatis.session.SqlSession;
import org.mybatis.spring.support.SqlSessionDaoSupport;
import zeh.mybatis.code00common.po.more.MarshallUser;

/**
 * mybatis原始方式开发dao的实现类。
 * <p>
 * 重点：整合后dao实现类继承SqlSessionDaoSupport类，能够自动实现sqlSessionFactory对象的注入。(前提是该dao必须交给spring的ioc进行实例化)
 *
 * @类名称： UserDaoImpl
 * @作者： zhaoerhu
 * @创建时间： 2019-4-13
 * @修改时间： 2019-4-13上午12:13:31
 * @备注：尊重每一行代码！
 */
public class MarshallUserDaoImpl extends SqlSessionDaoSupport implements MarshallUserDao {

    /**
     * 和spring整合后将不再需要自己定义构造方法来注入sqlSession对象了，而是继承spring-
     * mybatis整合包中的SqlSessionDaoSupport类
     * ，通过该类则spring可以自动注入sqlSession对象到当前的dao实现类中。
     */
//    private SqlSessionFactory sqlSessionFactory;
//
//    public UserDaoImpl() {
//    }
//
//    public UserDaoImpl(SqlSessionFactory sqlSessionFactory) {
//        this.sqlSessionFactory = sqlSessionFactory;
//    }
    @Override
    public MarshallUser findUserById(int id) throws Exception {
        // 继承SqlSessionDaoSupport，通过this.getSqlSession()得到sqlSessoin。
        SqlSession sqlSession = this.getSqlSession();

        MarshallUser user = sqlSession.selectOne("mybatisSpring.findUserByIdBySpring", id);

        return user;
    }


    @Override
    public int insertUserUseSpring() throws Exception {
        SqlSession sqlSession = this.getSqlSession();
        return sqlSession.insert("mybatisSpring.insertUserBySpring");
    }

    @Override
    public int insertUserA() throws Exception {
        SqlSession sqlSession = this.getSqlSession();
        return sqlSession.insert("mybatisSpring.insertUserABySpring");
    }

    @Override
    public int insertUserB() throws Exception {
        SqlSession sqlSession = this.getSqlSession();
        return sqlSession.insert("mybatisSpring.insertUserBBySpring");
    }

}
```

zeh.mybatis.code09mybatisspring.run.MarshallUserDaoImplRun
```java
package zeh.mybatis.code09mybatisspring.run;

import org.junit.Before;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import zeh.mybatis.code00common.po.more.MarshallUser;
import zeh.mybatis.code09mybatisspring.dao.MarshallUserDao;

/**
 * junit测试mybatis原始开发方式。
 * <p>
 * 如果运行报错: Failed to read candidate component class.
 * 解决方案：
 * 查看编译环境用的是1.8，将1.8降为1.7，问题解决，服务启动正常,也就是说spring 3.2不支持1.8编译环境，解决办法就是降为1.7编译环境
 * spring官网说了，要使用java8，只支持spring 4.X以上版本，而spring的使用最低java要求java5及以上，如果出现例外，那就例外说了，比如一开
 * 始spring 3.1就可以在java8上编译。
 *
 * @类名称： UserDaoImplTest
 * @作者： zhaoerhu
 * @创建时间： 2019-4-13
 * @修改时间： 2019-4-13上午12:14:43
 * @备注：尊重每一行代码！
 */
public class MarshallUserDaoImplRun {

    private ApplicationContext applicationContext;

    // 在setUp这个方法得到spring容器applicationContext对象
    @Before
    public void setUp() throws Exception {
        //通过ClassPathXmlApplicationContext构造方法加载spring配置
        applicationContext = new ClassPathXmlApplicationContext("classpath:mybatis_spring/spring/applicationContext.xml");
    }

    @Test
    public void testFindUserById() throws Exception {
        //参数1: name/id 参数2(可选): 可以指定生成对象类型,如果不填此参数,需进行强转 两种方式都可以获取!
        MarshallUserDao marshallUserDao = applicationContext.getBean("userDao", MarshallUserDao.class);// 通过spring容器获取dao的bean实例对象
        // 调用userDao的方法
        MarshallUser marshallUser = marshallUserDao.findUserById(1);
        System.out.println(marshallUser);
    }
}

```

## 5.2 mybatis和spring整合后使用代理模式开发DAO
zeh.mybatis.code09mybatisspring.mapper.MarshallItemsMapper
```java
package zeh.mybatis.code09mybatisspring.mapper;

import org.apache.ibatis.annotations.Param;
import zeh.mybatis.code00common.po.more.MarshallItems;
import zeh.mybatis.code09mybatisspring.pojo.SmellItemsExample;

import java.util.List;

public interface MarshallItemsMapper {
    int countByExample(SmellItemsExample example);

    int deleteByExample(SmellItemsExample example);

    int deleteByPrimaryKey(Integer id);

    int insert(MarshallItems record);

    int insertSelective(MarshallItems record);

    List<MarshallItems> selectByExampleWithBLOBs(SmellItemsExample example);

    List<MarshallItems> selectByExample(SmellItemsExample example);

    MarshallItems selectByPrimaryKey(Integer id);

    int updateByExampleSelective(@Param("record") MarshallItems record, @Param("example") SmellItemsExample example);

    int updateByExampleWithBLOBs(@Param("record") MarshallItems record, @Param("example") SmellItemsExample example);

    int updateByExample(@Param("record") MarshallItems record, @Param("example") SmellItemsExample example);

    int updateByPrimaryKeySelective(MarshallItems record);

    int updateByPrimaryKeyWithBLOBs(MarshallItems record);

    int updateByPrimaryKey(MarshallItems record);
}
```
对应的mapper.xml，和mapper接口在同一个包里。   
zeh/mybatis/code09mybatisspring/mapper/MarshallItemsMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="zeh.mybatis.code09mybatisspring.mapper.MarshallItemsMapper">

    <resultMap id="BaseResultMap" type="zeh.mybatis.code09mybatisspring.pojo.SmellItems">
        <id column="id" property="id" jdbcType="INTEGER"/>
        <result column="name" property="name" jdbcType="VARCHAR"/>
        <result column="price" property="price" jdbcType="REAL"/>
        <result column="pic" property="pic" jdbcType="VARCHAR"/>
        <result column="createtime" property="createtime" jdbcType="TIMESTAMP"/>
    </resultMap>

    <resultMap id="ResultMapWithBLOBs" type="zeh.mybatis.code09mybatisspring.pojo.SmellItems"
               extends="BaseResultMap">
        <result column="detail" property="detail" jdbcType="LONGVARCHAR"/>
    </resultMap>

    <sql id="Example_Where_Clause">
        <where>
            <foreach collection="oredCriteria" item="criteria" separator="or">
                <if test="criteria.valid">
                    <trim prefix="(" suffix=")" prefixOverrides="and">
                        <foreach collection="criteria.criteria" item="criterion">
                            <choose>
                                <when test="criterion.noValue">
                                    and ${criterion.condition}
                                </when>
                                <when test="criterion.singleValue">
                                    and ${criterion.condition} #{criterion.value}
                                </when>
                                <when test="criterion.betweenValue">
                                    and ${criterion.condition} #{criterion.value}
                                    and
                                    #{criterion.secondValue}
                                </when>
                                <when test="criterion.listValue">
                                    and ${criterion.condition}
                                    <foreach collection="criterion.value" item="listItem"
                                             open="(" close=")" separator=",">
                                        #{listItem}
                                    </foreach>
                                </when>
                            </choose>
                        </foreach>
                    </trim>
                </if>
            </foreach>
        </where>
    </sql>
    <sql id="Update_By_Example_Where_Clause">
        <where>
            <foreach collection="example.oredCriteria" item="criteria"
                     separator="or">
                <if test="criteria.valid">
                    <trim prefix="(" suffix=")" prefixOverrides="and">
                        <foreach collection="criteria.criteria" item="criterion">
                            <choose>
                                <when test="criterion.noValue">
                                    and ${criterion.condition}
                                </when>
                                <when test="criterion.singleValue">
                                    and ${criterion.condition} #{criterion.value}
                                </when>
                                <when test="criterion.betweenValue">
                                    and ${criterion.condition} #{criterion.value}
                                    and
                                    #{criterion.secondValue}
                                </when>
                                <when test="criterion.listValue">
                                    and ${criterion.condition}
                                    <foreach collection="criterion.value" item="listItem"
                                             open="(" close=")" separator=",">
                                        #{listItem}
                                    </foreach>
                                </when>
                            </choose>
                        </foreach>
                    </trim>
                </if>
            </foreach>
        </where>
    </sql>
    <sql id="Base_Column_List">
		id, name, price, pic, createtime
	</sql>
    <sql id="Blob_Column_List">
		detail
	</sql>
    <select id="selectByExampleWithBLOBs" resultMap="ResultMapWithBLOBs"
            parameterType="zeh.mybatis.code09mybatisspring.pojo.SmellItemsExample">
        select
        <if test="distinct">
            distinct
        </if>
        <include refid="Base_Column_List"/>
        ,
        <include refid="Blob_Column_List"/>
        from items
        <if test="_parameter != null">
            <include refid="Example_Where_Clause"/>
        </if>
        <if test="orderByClause != null">
            order by ${orderByClause}
        </if>
    </select>
    <select id="selectByExample" resultMap="BaseResultMap"
            parameterType="zeh.mybatis.code09mybatisspring.pojo.SmellItemsExample">
        select
        <if test="distinct">
            distinct
        </if>
        <include refid="Base_Column_List"/>
        from items
        <if test="_parameter != null">
            <include refid="Example_Where_Clause"/>
        </if>
        <if test="orderByClause != null">
            order by ${orderByClause}
        </if>
    </select>
    <select id="selectByPrimaryKey" resultMap="ResultMapWithBLOBs"
            parameterType="java.lang.Integer">
        select
        <include refid="Base_Column_List"/>
        ,
        <include refid="Blob_Column_List"/>
        from items
        where id = #{id,jdbcType=INTEGER}
    </select>
    <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer">
		delete from items
		where id = #{id,jdbcType=INTEGER}
	</delete>
    <delete id="deleteByExample" parameterType="zeh.mybatis.code09mybatisspring.pojo.SmellItemsExample">
        delete from items
        <if test="_parameter != null">
            <include refid="Example_Where_Clause"/>
        </if>
    </delete>
    <insert id="insert" parameterType="zeh.mybatis.code09mybatisspring.pojo.SmellItems">
		insert into items (id, name,
		price,
		pic, createtime, detail
		)
		values (#{id,jdbcType=INTEGER},
		#{name,jdbcType=VARCHAR},
		#{price,jdbcType=REAL},
		#{pic,jdbcType=VARCHAR}, #{createtime,jdbcType=TIMESTAMP},
		#{detail,jdbcType=LONGVARCHAR}
		)
	</insert>
    <insert id="insertSelective" parameterType="zeh.mybatis.code09mybatisspring.pojo.SmellItems">
        insert into items
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="id != null">
                id,
            </if>
            <if test="name != null">
                name,
            </if>
            <if test="price != null">
                price,
            </if>
            <if test="pic != null">
                pic,
            </if>
            <if test="createtime != null">
                createtime,
            </if>
            <if test="detail != null">
                detail,
            </if>
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="id != null">
                #{id,jdbcType=INTEGER},
            </if>
            <if test="name != null">
                #{name,jdbcType=VARCHAR},
            </if>
            <if test="price != null">
                #{price,jdbcType=REAL},
            </if>
            <if test="pic != null">
                #{pic,jdbcType=VARCHAR},
            </if>
            <if test="createtime != null">
                #{createtime,jdbcType=TIMESTAMP},
            </if>
            <if test="detail != null">
                #{detail,jdbcType=LONGVARCHAR},
            </if>
        </trim>
    </insert>
    <select id="countByExample" parameterType="zeh.mybatis.code09mybatisspring.pojo.SmellItemsExample"
            resultType="java.lang.Integer">
        select count(*) from items
        <if test="_parameter != null">
            <include refid="Example_Where_Clause"/>
        </if>
    </select>
    <update id="updateByExampleSelective" parameterType="map">
        update items
        <set>
            <if test="record.id != null">
                id = #{record.id,jdbcType=INTEGER},
            </if>
            <if test="record.name != null">
                name = #{record.name,jdbcType=VARCHAR},
            </if>
            <if test="record.price != null">
                price = #{record.price,jdbcType=REAL},
            </if>
            <if test="record.pic != null">
                pic = #{record.pic,jdbcType=VARCHAR},
            </if>
            <if test="record.createtime != null">
                createtime = #{record.createtime,jdbcType=TIMESTAMP},
            </if>
            <if test="record.detail != null">
                detail = #{record.detail,jdbcType=LONGVARCHAR},
            </if>
        </set>
        <if test="_parameter != null">
            <include refid="Update_By_Example_Where_Clause"/>
        </if>
    </update>
    <update id="updateByExampleWithBLOBs" parameterType="map">
        update items
        set id = #{record.id,jdbcType=INTEGER},
        name =
        #{record.name,jdbcType=VARCHAR},
        price = #{record.price,jdbcType=REAL},
        pic = #{record.pic,jdbcType=VARCHAR},
        createtime =
        #{record.createtime,jdbcType=TIMESTAMP},
        detail =
        #{record.detail,jdbcType=LONGVARCHAR}
        <if test="_parameter != null">
            <include refid="Update_By_Example_Where_Clause"/>
        </if>
    </update>
    <update id="updateByExample" parameterType="map">
        update items
        set id = #{record.id,jdbcType=INTEGER},
        name =
        #{record.name,jdbcType=VARCHAR},
        price = #{record.price,jdbcType=REAL},
        pic = #{record.pic,jdbcType=VARCHAR},
        createtime =
        #{record.createtime,jdbcType=TIMESTAMP}
        <if test="_parameter != null">
            <include refid="Update_By_Example_Where_Clause"/>
        </if>
    </update>
    <update id="updateByPrimaryKeySelective" parameterType="zeh.mybatis.code09mybatisspring.pojo.SmellItems">
        update items
        <set>
            <if test="name != null">
                name = #{name,jdbcType=VARCHAR},
            </if>
            <if test="price != null">
                price = #{price,jdbcType=REAL},
            </if>
            <if test="pic != null">
                pic = #{pic,jdbcType=VARCHAR},
            </if>
            <if test="createtime != null">
                createtime = #{createtime,jdbcType=TIMESTAMP},
            </if>
            <if test="detail != null">
                detail = #{detail,jdbcType=LONGVARCHAR},
            </if>
        </set>
        where id = #{id,jdbcType=INTEGER}
    </update>
    <update id="updateByPrimaryKeyWithBLOBs" parameterType="zeh.mybatis.code09mybatisspring.pojo.SmellItems">
		update
		items
		set name = #{name,jdbcType=VARCHAR},
		price =
		#{price,jdbcType=REAL},
		pic = #{pic,jdbcType=VARCHAR},
		createtime =
		#{createtime,jdbcType=TIMESTAMP},
		detail =
		#{detail,jdbcType=LONGVARCHAR}
		where id = #{id,jdbcType=INTEGER}
	</update>

    <update id="updateByPrimaryKey" parameterType="zeh.mybatis.code09mybatisspring.pojo.SmellItems">
		update items
		set
		name = #{name,jdbcType=VARCHAR},
		price = #{price,jdbcType=REAL},
		pic =
		#{pic,jdbcType=VARCHAR},
		createtime = #{createtime,jdbcType=TIMESTAMP}
		where id = #{id,jdbcType=INTEGER}
	</update>
</mapper>
```
zeh.mybatis.code09mybatisspring.run.MarshallItemsMapperRun
```java
package zeh.mybatis.code09mybatisspring.run;

import org.junit.Before;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import zeh.mybatis.code00common.po.more.MarshallItems;
import zeh.mybatis.code09mybatisspring.mapper.MarshallItemsMapper;
import zeh.mybatis.code09mybatisspring.pojo.SmellItemsExample;

import java.util.List;

/**
 * 应用层入口
 *
 * @author zhaoeh
 */
public class MarshallItemsMapperRun {

    private ApplicationContext applicationContext;

    private MarshallItemsMapper itemsMapper;

    // 在setUp这个方法通知spring容器加载spring配置文件，从而得到spring对象获取bean实例
    @Before
    public void setUp() throws Exception {
        applicationContext = new ClassPathXmlApplicationContext("classpath:mybatis_spring/spring/applicationContext.xml");
        //参数1: name/id 参数2(可选): 可以指定生成对象类型,如果不填此参数,需进行强转 两种方式都可以获取!
        itemsMapper = applicationContext.getBean("itemsMapper", MarshallItemsMapper.class);
    }

    // 根据主键删除
    @Test
    public void testDeleteByPrimaryKey() {

    }

    // 插入
    @Test
    public void testInsert() {
        // 构造 items对象
        MarshallItems items = new MarshallItems();
        items.setName("手机");
        items.setPrice(999f);
        itemsMapper.insert(items);
    }

    // 自定义条件查询
    @Test
    public void testSelectByExample() {
        SmellItemsExample itemsExample = new SmellItemsExample();
        // 通过criteria构造查询条件
        SmellItemsExample.Criteria criteria = itemsExample.createCriteria();
        criteria.andNameEqualTo("笔记本3");
        // 可能返回多条记录
        List<MarshallItems> list = itemsMapper.selectByExample(itemsExample);

        System.out.println(list);

    }

    // 根据主键查询
    @Test
    public void testSelectByPrimaryKey() {
        MarshallItems items = itemsMapper.selectByPrimaryKey(1);
        System.out.println(items);
    }

    // 更新数据
    @Test
    public void testUpdateByPrimaryKey() {

        // 对所有字段进行更新，需要先查询出来再更新
        MarshallItems items = itemsMapper.selectByPrimaryKey(1);
        items.setName("水杯");
        //使用updateByPrimaryKey或者updateByPrimaryKeySelective方法时，只需要传入需要更新的实体对象，更新条件就是传入实体对象的主键。
        //所以传入的实体对象如果不存在主键值，则必须手动set进去，否则将不会更新任何数据。
        itemsMapper.updateByPrimaryKey(items);
        // 如果传入字段不空为才更新，在批量更新中使用此方法，不需要先查询再更新
        // itemsMapper.updateByPrimaryKeySelective(record);

    }

}

```

案例二：   
zeh.mybatis.code09mybatisspring.mapper.MarshallUserMapper
```java
package zeh.mybatis.code09mybatisspring.mapper;

import zeh.mybatis.code00common.po.more.MarshallUser;

import java.util.List;

/**
 * <p>
 * Title: UserMapper
 * </p>
 * <p>
 * Description: mapper接口，相当 于dao接口，用户管理
 * </p>
 * <p>
 * Company: www.itcast.com
 * </p>
 *
 * @author 传智.燕青
 * @version 1.0
 * @date 2015-4-22下午2:45:12
 */
public interface MarshallUserMapper {

    // 根据id查询用户信息
    public List<MarshallUser> findUserById(int id) throws Exception;

}

```
对应的mapper.xml，和Mapper接口在同一个包里。   
zeh/mybatis/code09mybatisspring/mapper/MarshallUserMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- namespace命名空间，作用就是对sql进行分类化管理，理解sql隔离 -->
<!-- 注意：使用mapper代理方法开发，namespace有特殊重要的作用，namespace等于mapper接口地址 -->
<mapper namespace="zeh.mybatis.code09mybatisspring.mapper.MarshallUserMapper">

    <!-- 本mapper比较简单，只是执行一个statement id，目的就是为了验证mybatis和spring的集成 -->
    <select id="findUserById" parameterType="int" resultType="marshallUser">
		select
		* from users where id=#{value}
	</select>

</mapper>
```

zeh.mybatis.code09mybatisspring.run.MarshallUserMapperRun
```java
package zeh.mybatis.code09mybatisspring.run;

import org.junit.Before;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import zeh.mybatis.code00common.po.more.MarshallUser;
import zeh.mybatis.code09mybatisspring.mapper.MarshallUserMapper;

import java.util.Iterator;
import java.util.List;

/**
 * junit测试mapper代理开发方式。
 *
 * @类名称： UserMapperTest
 * @作者： zhaoerhu
 * @创建时间： 2019-4-13
 * @修改时间： 2019-4-13上午12:25:39
 * @备注：尊重每一行代码！
 */
public class MarshallUserMapperRun {

    private ApplicationContext applicationContext;

    @Before
    public void setUp() throws Exception {
        //通过ClassPathXmlApplicationContext构造方法加载spring配置，实例化spring对象，从而得到指定的bean对象
        this.applicationContext = new ClassPathXmlApplicationContext("classpath:mybatis_spring/spring/applicationContext.xml");
    }

    @Test
    public void testFindUserById() throws Exception {
        /**
         * 如果通过spring自动扫描mapper后，通过ioc注入的bean实例对象默认是mapper接口对应的名称（首字母小写）。
         *
         * 参数1: name/id 参数2(可选): 可以指定生成对象类型,如果不填此参数,需进行强转 两种方式都可以获取!
         */
        MarshallUserMapper marshallUserMapper = applicationContext.getBean("marshallUserMapper", MarshallUserMapper.class);
        // 查询结果集返回多条记录，使用list集合接收
        List<MarshallUser> list = marshallUserMapper.findUserById(1);

        // 采用Iterator标准集合输出list
        Iterator<MarshallUser> iterator = list.iterator();
        System.out.println("根据id查询多条user结果如下所示：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
        System.out.println("根据id查询多条user结果总数目为：" + list.size());
    }
}

```