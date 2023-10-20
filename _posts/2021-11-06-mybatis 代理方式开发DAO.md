---
layout:     post
title:      mybatis代理方式开发DAO
subtitle:   mybatis代理方式开发的DAO接口不用自己提供实现类，但需要遵守一定的规范
categories: [mybatis]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 什么是mybatis代理开发？
mybatis的mapper代理接口开发，简化了开发流程，不再需要开发者编写dao接口实现类。   
开发者只需要按照mapper代理接口开发的流程编写Dao接口和mapper.xml文件即可，至于Dao接口的实现类，mybatis底层会通过动态代理的方式自动为Dao接口动态生成Dao实现类，从而达到简化开发的目的。   
mapper代理方式实现dao模式，mapper接口，相当于dao接口。只需要定义mapper接口即可：按照mybatis的规范进行接口的编写即可。   
## 1.1 mapper代理开发规范如下：
1.在mapper.xml中配置的namespace 必须等于对应的 mapper接口的权限定名地址，而mybatis原始方式开发需要编写Dao接口、Dao实现类、mapper.xml文件，且mapper.xml中的namespace任意指定。   
2.mapper.java接口中的方法名必须和mapper.xml中statementId一致。   
3.mapper.java接口中的方法输入参数类型和mapper.xml中statement的parameterType指定的类型一致。   
4.mapper.java接口中的方法返回值类型和mapper.xml中statement的resultType指定的类型一致。   
5.如果要在sqlMapConfig.xml文件中通过class属性加载映射文件的话，还需要mapper.java接口和mapper.xml配置文件同名且放在同一个包路径下；否则，单独使用代理开发是不需要遵守第5点的。   
## 1.2 mapper代理方式的灵活性体现在：
因为底层采用动态代理，mapper接口实现类是动态生成的，所以可以显式传递多个输入参数，而不受限于原始方式的固定参数（原始方式只能传递一个简单类型或者一个pojo\list\map等）。   
当使用代理方式或者注解方式(注解方式是代理方式的简化)传递多个输入参数的时候，有如下几种方式：   
（1）、使用pojo、list、map等对象对多个参数进行封装，适用于原始方式、mapper代理、注解方式；parameterType可以指定也可以不指定。   
（2）、如果传入参数全部为简单类型，可以通过参数下标使用#{}进行获取，下标从0开始。parameterType不能指定。--基本不用。   
（3）、如果传入参数为简单类型和对象类型，可以使用注解@Param()为参数指定别名，从而通过#{}或者${}使用OGNL表达式获取，此时获取时#{}或者${}中的参数必须是@Param()注解中设置的参数别名。注意这种方式可以传递任何类型，比如多个简单类型，多个简单类型和pojo等。parameterType可指定可不指定。    
## 1.3 mybatis的取参方式
mybatis的取参方式：（1）#{}；（2）${}。   
## 1.4 总结：
(1),使用原始方式开发mapper只能传入单个对象，parameterType可指定可不指定。    
(2),使用代理方式和注解方式可以传入多个简单类型，parameterType可指定可不指定，只能按照#{}从下标为0开始获取下标确定对应的参数；   
传入单个参数和原始方式一样；   
也支持传入多个参数使用@Param()指定参数别名，然后使用#{}${}按照别名获取--使用频率非常高。   

# 2. mybatis代理开发-用户DAO
## 2.1 定义mapper接口
zeh.mybatis.code04proxy.dao.UserDaoProxy
```java
package zeh.mybatis.code04proxy.dao;

import org.apache.ibatis.annotations.Param;
import zeh.mybatis.code00common.po.singleton.CommonPo;
import zeh.mybatis.code00common.po.singleton.UserCustomVo;
import zeh.mybatis.code00common.po.singleton.UserQueryVo;

import java.util.List;

/**
 * mapper代理方式开发dao接口(mapper接口)
 * <p>
 * mybatis代理方式开发dao接口，不要使用泛型定义接口，直接指定类型。
 * <p>
 * mybatis的原始方式开发 dao 接口没法显式接收多个输入参数；但是使用代理开发方式可以显式接收多个输入参数（需要使用@Param()注解标注各个参数的别名）。
 *
 * @类名称： UserDaoProxy
 * @作者： zhaoerhu
 * @创建时间： 2019-4-3
 * @修改时间： 2019-4-3上午11:22:03
 * @备注：尊重每一行代码！
 */
public interface UserDaoProxy {

    // 用户信息综合查询，接收两个参数：一个简单类型，一个pojo类型。
    // 说明：多个参数时，简单类型和pojo一起上送，不一定每个参数都得使用@Param()注解指定参数别名。
    // 原则为：只要是目标statementId中使用到的参数，必须使用@Param()指定别名，否则上送一堆参数，statementId不知道取哪一个参数。
    // 只要使用了@Param()直接指定了别名，则statementid中的parameterType可指定可不指定，无所谓了。
    public List<UserCustomVo> findUserListProxy(@Param("aaa") String test, @Param("zhao") UserQueryVo userQueryVo) throws Exception;

    // 用户信息综合查询总数，简单类型和pojo一起上送，必须为statementid使用的参数指定@Param()别名。
    public int findUserCountProxy(String param1, @Param("eric") UserQueryVo userQueryVo) throws Exception;

    // 根据id查询用户信息,传递多个简单 参数，使用@Param()注解指定别名。
    public CommonPo findUserByIdProxy(@Param("哈哈哈哈") String param1, @Param(value = "daisy") int id) throws Exception;

    // 根据id查询用户信息，使用resultMap输出
    public CommonPo findUserByIdResultMap(int id) throws Exception;

    // 根据用户名列查询用户列表，只传递一个参数
    public List<CommonPo> findUserByNameProxy(String name) throws Exception;

    // 插入用户
    public void insertUser(String param1, CommonPo user) throws Exception;

    // 删除用户
    public void deleteUser(String param1, int id) throws Exception;
}

```
## 2.2 resource下定义mapper.xml
mybatis/mapper/mybatis-proxy-mapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- mybatis接口开发Dao模式 mapper.xml映射文件很重要。 -->
<!-- namespace命名空间，作用就是对sql进行分类化管理，理解sql隔离。 -->
<!-- 注意：使用mapper代理方法开发，namespace有特殊重要的作用，namespace等于mapper接口的全限定名。 -->
<!-- mapper代理方式开发dao遵循的规范： -->
<!-- 1.在mapper.xml中namespace等于mapper接口地址。 -->
<!-- 2.mapper.java接口中的方法名和mapper.xml中statement的id一致。 -->
<!-- 3.mapper.java接口中的方法输入参数类型和mapper.xml中statement的parameterType指定的类型一致。 -->
<!-- 4.mapper.java接口中的方法返回值类型和mapper.xml中statement的resultType指定的类型一致。 -->
<!-- 5.如果要在sqlmapconfig.xml文件中通过class属性加载映射文件的话，还需要mapper.java接口和mapper.xml配置文件同名且放在同一个包路径下。 -->
<!-- 否则，单独使用代理开发是不需要遵守第5点的。 -->
<mapper namespace="zeh.mybatis.code04proxy.dao.UserDaoProxy">
    <!-- 开启本mapper的namespace下的二级缓存，二级缓存以namespace进行区分 -->
    <cache/>

    <!-- 定义sql片段 -->
    <!-- sql片段：将完整的sql拆分成子语句，提高sql语句的可复用性 经验：尽量基于单表定义sql片段，这样sql片段的可重用性才高。 -->
    <!-- 在sql片段中不要包括where，而是放在引用sql片段的地方使用where。 -->
    <sql id="query_user_where">
        <!-- if标签定义判断条件，即当条件成立时该执行怎样的sql片段 test中定义的成员是引用sql片段的地方指定的输入类型po中的成员变量 -->
        <!-- 该例中引用该sql片段的statement id的输入类型是mybatis.po.UserQueryVo UserQueryVo中引用了userCustomVo，userCustomVo继承了UserPo -->
        <if test="eric.userCustom != null">
            <if test="eric.userCustom.sex!=null and eric.userCustom.sex!=''">
                <!-- 开始拼接sql片段，引用该sql片段的statement id使用的sql语句是select * from users -->
                <!-- 所以，此处拼接的是users.sex= #{userCustom.sex},通过OGNL表达式拿出输入vo对象中的成员链 如果是第一个and会被自动去掉（前提是在调用处使用where） -->
                and users.sex = #{eric.userCustom.sex}
            </if>
            <if test="eric.userCustom.name != null and eric.userCustom.name != ''">
                <!-- 通过OGNL取得输入vo对象中的成员链，${属性.属性...}表示拼接sql串 -->
                and users.name like '%${eric.userCustom.name}%'
            </if>
            <if test="eric.ids != null">
                <!-- UserQueryVo中必须存在ids成员，ids是集合类型，所以支持foreach标签遍历初始化集合中的元素 。 -->
                <!-- collection:指定输入参数vo对象中的集合成员 -->
                <!-- 如果是单个list，则collection是list；如果是数组，则是array；如果是map，则是map的key；如果是pojo，则是pojo中的list属性名 -->
                <!-- item:给初始化集合ids中的每个元素定义的变量名，sql中会通过OGNL调用 -->
                <!-- open:要拼接的sql片段的开始字符串 -->
                <!-- close:要拼接的sql片段的结束字符串 -->
                <!-- separator:要拼接的字符串的连接符号 -->
                <!-- 遍历组合该sql： and (id=1 or id=3) -->
                <foreach collection="eric.ids" item="users_id" open="and ("
                         close=")" separator="or">
                    <!-- 每次遍历只需要组装下面的核心sql，开始、结束字符串和连接符号foreach标签已经指定 -->
                    id=#{users_id}
                </foreach>

                <!-- 同样的方式遍历组合该sql： or id in(1,3) -->
                <foreach collection="eric.ids" item="users_id" open="or id in("
                         close=")" separator=",">
                    #{users_id}
                </foreach>
            </if>
        </if>
    </sql>

    <!-- 定义resultMap -->
    <!-- 注意： 使用resultType进行输出映射，只有查询出来的列名和pojo中的属性名一致，该列才可以映射成功。 -->
    <!-- 如果查询出来的列名和pojo的属性名不一致，通过定义一个resultMap对列名和pojo属性名之间作一个映射关系。 -->
    <!-- 即resultMap比resultType更高级，是对resultType的再包装 。 -->
    <!-- 假如执行目标sql:SELECT id id_,username username_ FROM USER，该查询字段是别名，和po对象中的User成员不同， -->
    <!-- 所以使用resultType不能成功映射。 -->
    <!-- 通过resultMap指定查询的列名和目标po对象中属性成员的映射关系。 -->
    <!-- type：resultMap最终映射的java对象（po对象）类型，可以使用别名，此处使用别名userPo -->
    <!-- id：resultMap的标识id -->
    <resultMap type="commonPo" id="userResultMap">
        <!-- id：表示查询结果集中唯一标识 ，即主键，如果主键是组合列，则id写多个映射。 -->
        <!-- column：查询出来的列名 property：type指定的pojo类型中的成员名称。 -->
        <!-- property：结果即对应的pojo中的对应属性成员的名称。 -->
        <!-- 最终resultMap对column和property作一个映射关系 （对应关系），表示将sql查询出来的字段映射到指定的pojo类属性上 -->
        <id column="id_" property="id"/>

        <!-- result：对普通列的映射定义 -->
        <!-- column：sql查询出来的列名，普通列 -->
        <!-- property：type指定的pojo类型中的属性名 -->
        <!-- 最终resultMap对column和property作一个映射关系 （对应关系） -->
        <result column="username_" property="username"/>
    </resultMap>


    <!-- 根据sql片段查询user表满足条件的record parameterType：指定输入类型，这里是经过再次包装的vo类 resultType：输出结果类型 -->
    <!-- 注意：mybatis的查询语句将返回结果集封装成指定的vo或者po对象，之后需要在代码中解封装即可 使用resultType进行输出映射 -->
    <!-- 只有查询出来的列名和pojo中的属性名一致，该列才可以映射成功，即该列才会被封装到po或者vo对象的指定属性中。 -->
    <!-- 如果查询出来的列名和pojo中的属性名全部不一致，则不会创建pojo对象进行封装。 -->
    <!-- 只要查询出来的列名和pojo中的属性有一个一致，则会创建pojo对象封装一致的列与属性。 -->
    <select id="findUserListProxy"
            parameterType="zeh.mybatis.code00common.po.singleton.UserQueryVo"
            resultType="zeh.mybatis.code00common.po.singleton.UserCustomVo">
        select users.* from
        <include refid="12345"></include>
        <!-- mybatis的标签可以用在任何地方，和代码中的if等逻辑含义一样，不一定非得用在where中 -->
        <!-- where标签可以自动去掉引用的sql片段中的第一个and -->
        <!-- 且一般都是在statement id中通过where标签拼接where条件，而不要在sql片段中硬编码 -->
        <!-- 亦或者更常见的做法是在此处加上 where 1 = 1 这个通用查询条件。 -->
        <where>
            <!-- 通过include标签引用指定的sql片段id，如果refid指定的sql片段id不在本mapper文件中，需要前边加namespace -->
            <!--<include refid="query_user_where"></include>-->
            <!-- <include refid="query_user_where"/> -->
            <!-- 这里还可以继续引用其他的sql片段 -->
            <!-- 此处传入的是多个参数，都用@Param()注解指定了别名，所以一旦使用@Param()指定别名了，则在mapper中所有的参数都得带上该别名 -->
            <!--比如，原来userCustom是指定的parameterType的vo中的，直接使用即可，但是该vo有了别名就必须得带上别名，否则找不到该参数-->
            <if test="zhao.userCustom != null">
                <if test="zhao.userCustom.sex!=null and zhao.userCustom.sex!=''">
                    <!-- 开始拼接sql片段，引用该sql片段的statement id使用的sql语句是select * from users -->
                    <!-- 所以，此处拼接的是users.sex= #{userCustom.sex},通过OGNL表达式拿出输入vo对象中的成员链 如果是第一个and会被自动去掉（前提是在调用处使用where） -->
                    and users.sex = #{zhao.userCustom.sex}
                </if>
                <if test="zhao.userCustom.name != null and zhao.userCustom.name != ''">
                    <!-- 通过OGNL取得输入vo对象中的成员链，${属性.属性...}表示拼接sql串 -->
                    and users.name like '%${zhao.userCustom.name}%'
                </if>
                <if test="zhao.ids != null">
                    <!-- UserQueryVo中必须存在ids成员，ids是集合类型，所以支持foreach标签遍历初始化集合中的元素 。 -->
                    <!-- collection:指定输入参数vo对象中的集合成员 -->
                    <!-- 如果是单个list，则collection是list；如果是数组，则是array；如果是map，则是map的key；如果是pojo，则是pojo中的list属性名 -->
                    <!-- 如果使用了@Param()为传入的对象指定了别名的话，则不能再使用上面的默认名称，要使用显式指定的别名 -->
                    <!-- item:给初始化集合ids中的每个元素定义的变量名，sql中会通过OGNL调用 -->
                    <!-- open:要拼接的sql片段的开始字符串 -->
                    <!-- close:要拼接的sql片段的结束字符串 -->
                    <!-- separator:要拼接的字符串的连接符号 -->
                    <!-- 遍历组合该sql： and (id=1 or id=3) -->
                    <foreach collection="zhao.ids" item="users_id" open="and ("
                             close=")" separator="or">
                        <!-- 每次遍历只需要组装下面的核心sql，开始、结束字符串和连接符号foreach标签已经指定 -->
                        id=#{users_id}
                    </foreach>

                    <!-- 同样的方式遍历组合该sql： or id in(1,3) -->
                    <foreach collection="zhao.ids" item="users_id" open="or id in("
                             close=")" separator=",">
                        #{users_id}
                    </foreach>
                </if>
            </if>
            <!-- 同样 aaa 也是@Param()注解指定的参数别名 -->
            <if test="aaa != null  and aaa != ''">
                or users.name like '%一%'
            </if>
        </where>
    </select>

    <sql id="12345">
        <if test="zhao.userCustom.name != null and zhao.userCustom.name != ''">
            users
        </if>
    </sql>

    <!-- 根据sql片段查询user表满足条件的record数量 parameterType：指定输入类型和findUserList一样，都是vo类mybatis.po.UserQueryVo
        resultType：输出结果类型 -->
    <select id="findUserCountProxy"
            parameterType="zeh.mybatis.code00common.po.singleton.UserQueryVo"
            resultType="int">
        select count(1) from users
        <!-- where可以自动去掉条件中的第一个and -->
        <where>
            <!-- 通过include标签引用指定的sql片段id，如果refid指定的sql片段id不在本mapper文件中，需要前边加namespace -->
            <include refid="query_user_where"></include>
            <!-- 这里还可以继续引用其他的sql片段 -->
        </where>
    </select>

    <!-- 在 映射文件中配置很多sql语句 -->
    <!-- 需求：通过id查询用户表的记录 -->
    <!-- 通过 select执行数据库查询 id：标识 映射文件中的 sql 将sql语句封装到mappedStatement对象中，所以将id称为statement的id
        parameterType：指定输入 参数的类型，这里指定int型 resultType：指定sql输出结果 的所映射的java对象类型，select指定resultType表示将单条记录映射成的java对象，此处使用别名。
        #{}表示一个占位符号 #{id}：其中的id表示接收输入 的参数，参数名称就是id，如果输入 参数是简单类型，#{}中的参数名可以任意，可以value或其它名称 -->
    <!-- 如果接口使用了@Param()指定了对应参数的别名，则该语句中用到的参数名必须和设置的别名完全一致 -->
    <select id="findUserByIdProxy" parameterType="int" resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
		select * from users where id=#{daisy}
	</select>

    <!-- 使用resultMap进行输出映射 resultMap：指定定义的resultMap的id，如果这个resultMap在其它的mapper文件，前边需要加namespace -->
    <select id="findUserByIdResultMap" parameterType="int"
            resultMap="userResultMap">
		SELECT id id_,name name_ FROM USERS WHERE id=#{value}
	</select>


    <!-- 根据用户名称模糊查询用户信息，可能返回多条 resultType：指定就是单条记录所映射的java对象 类型 ${}:表示拼接sql串，将接收到参数的内容不加任何修饰拼接在sql中。
        使用${}拼接sql，引起 sql注入 ${value}：接收输入 参数的内容，如果传入类型是简单类型，${}中只能使用value -->
    <select id="findUserByNameProxy" parameterType="java.lang.String"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        <!-- select * from users where name like '%${value}%' -->

        <!-- 下面方式直接使用${}取值是不对的，因为${}取出来的值是不加任何修饰的，对于字符串连引号也不会加。 -->
        <!-- 所以如果采用${}取值的话需要手动加上单引号 。 -->
        <!-- select * from users where name = ${value} -->

        <!-- 使用如下方式，将${}传进来的值如果是字符串类型的话手动加上单引号即可 -->
        <!-- 手动加上单引号 -->
        select * from users where name = '${value}'

        <!-- 采用字符串拼接的方式也是错误的 -->
        <!-- select * from users where name ='' || ${value} -->
    </select>

    <!-- 添加用户 parameterType：指定输入 参数类型是pojo（包括 用户信息） #{}中指定pojo的属性名，接收到pojo对象的属性值，mybatis通过OGNL获取对象的属性值 -->
    <insert id="insertUser" parameterType="zeh.mybatis.code00common.po.singleton.CommonPo">
        <!-- 将插入数据的主键返回，返回到user对象中 SELECT LAST_INSERT_ID()：得到刚insert进去记录的主键值，只适用与自增主键
            keyProperty：将查询到主键值设置到parameterType指定的对象的哪个属性 order：SELECT LAST_INSERT_ID()执行顺序，相对于insert语句来说它的执行顺序
            resultType：指定SELECT LAST_INSERT_ID()的结果类型 -->
        <selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
            SELECT
            LAST_INSERT_ID()
        </selectKey>
        insert into users(id,name,age,sex,hobby)
        values(#{id},#{name},#{age},#{sex},#{hobby})
        <!-- 使用mysql的uuid（）生成主键 执行过程： 首先通过uuid()得到主键，将主键设置到user对象的id属性中 其次在insert执行时，从user对象中取出id属性值 -->
        <!-- <selectKey keyProperty="id" order="BEFORE" resultType="java.lang.String">
            SELECT uuid() </selectKey> insert into user(id,username,birthday,sex,address)
            value(#{id},#{username},#{birthday},#{sex},#{address}) -->
    </insert>

    <!-- 删除 用户 根据id删除用户，需要输入 id值 -->
    <delete id="deleteUser" parameterType="java.lang.Integer">
		delete from users where id=#{id}
	</delete>

    <!-- 根据id更新用户 分析： 需要传入用户的id 需要传入用户的更新信息 parameterType指定user对象，包括 id和更新信息，注意：id必须存在
        #{id}：从输入 user对象中获取id属性值 -->
    <update id="updateUser" parameterType="zeh.mybatis.code00common.po.singleton.CommonPo">
		update users set
		name=#{name},age=#{age},sex=#{sex},hobby=#{hobby}
		where id=#{id}
	</update>
</mapper>
```
## 2.3 运行
zeh.mybatis.code04proxy.run.UserDaoProxyRun
```java
package zeh.mybatis.code04proxy.run;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
import zeh.mybatis.code00common.po.singleton.CommonPo;
import zeh.mybatis.code00common.po.singleton.UserCustomVo;
import zeh.mybatis.code00common.po.singleton.UserQueryVo;
import zeh.mybatis.code04proxy.dao.UserDaoProxy;

import java.io.File;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

/**
 * 使用Junit测试mapper代理实现的dao模式。
 * <p>
 * mapper代理方式实现dao模式-mapper代理方式实现dao模式只需要定义dao接口即可。
 * 具体mapper接口底层使用的是selectOne()还是selectList()取决于调用接口时的返回值类型。
 * 如果返回是一个简单类型或者map或者pojo则调用selectOne()接口。
 * 如果是一个List<Pojo>或者List<Map<String,Object>>则默认调用selectList()接口。
 *
 * @类名称： UserDaoProxyJunitRun
 * @作者： zhaoerhu
 * @创建时间： 2019-4-3
 * @修改时间： 2019-4-3上午10:49:06
 * @备注：尊重每一行代码！
 */
public class UserDaoProxyRun {

    private SqlSessionFactory sqlSessionFactory;

    // 创建sqlSessionFactory:此方法是在执行testFindUserById之前执行
    @Before
    public void setUp() throws Exception {
        // mybatis配置文件
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        // 得到配置文件流
        InputStream inputStream = Resources.getResourceAsStream(resource);
        // 创建会话工厂，传入mybatis的配置文件信息
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    }

    /**
     * 功能描述: 用户信息的综合查询，传递一个简单类型和一个pojo，mapper代理方式可以传递多个参数到 statementId 中。
     *
     * @params: []
     * @return: void
     * @author: zhaoerhu
     * @date: 2019/7/20 17:47
     */
    @Test
    public void testFindUserList() throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建UserMapper对象，mybatis自动生成mapper代理对象（通过反射和动态代理实现）
        UserDaoProxy userMapper = sqlSession.getMapper(UserDaoProxy.class);
        // 创建包装对象，设置查询条件
        UserCustomVo userCustom = new UserCustomVo();
        // 由于这里使用动态sql，如果不设置某个值，条件不会拼接在sql中
        userCustom.setSex("男");
        userCustom.setName("赵二虎");
        // 传入多个id
        List<Integer> ids = new ArrayList<Integer>();
        ids.add(1);
        ids.add(3);
        // 将ids通过userQueryVo传入statement中
        UserQueryVo userQueryVo = new UserQueryVo();
        userQueryVo.setIds(ids);
        userQueryVo.setUserCustom(userCustom);
        // 调用userMapper的方法，传递两个参数：一个简单类型，一个pojo类型。
        List<UserCustomVo> list = userMapper.findUserListProxy("zhaoerhu", userQueryVo);
        //采用标准输出iterator输出list集合
        Iterator<UserCustomVo> iterator = list.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    /**
     * 功能描述: 查询指定条件的记录总数。传递一个简单类型和一个pojo。
     *
     * @params: []
     * @return: void
     * @author: zhaoerhu
     * @date: 2019/7/20 17:47
     */
    @Test
    public void testFindUserCount() throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建UserMapper对象，mybatis自动生成mapper代理对象
        UserDaoProxy userMapper = sqlSession.getMapper(UserDaoProxy.class);
        // 创建包装对象，设置查询条件
        UserQueryVo userQueryVo = new UserQueryVo();
        UserCustomVo userCustom = new UserCustomVo();
        userCustom.setSex("1");
        userCustom.setName("张三丰");
        userQueryVo.setUserCustom(userCustom);
        // 调用userMapper的方法
        int count = userMapper.findUserCountProxy("测试", userQueryVo);
        System.out.println(count);
    }

    /**
     * 功能描述: 根据Id查询记录。传递多个简单类型，分别使用@Param()指定别名。
     *
     * @params: []
     * @return: void
     * @author: zhaoerhu
     * @date: 2019/7/20 17:54
     */
    @Test
    public void testFindUserById() throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建UserMapper对象，mybatis自动生成mapper代理对象
        UserDaoProxy userMapper = sqlSession.getMapper(UserDaoProxy.class);
        // 调用userMapper的方法
        CommonPo commonPo = userMapper.findUserByIdProxy("wo cao", 1);
        System.out.println(commonPo);
    }

    /**
     * 根据id查询记录并使用resultMap进行结果列映射。
     *
     * @throws Exception
     */
    @Test
    public void testFindUserByIdResultMap() throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建UserMapper对象，mybatis自动生成mapper代理对象
        UserDaoProxy userMapper = sqlSession.getMapper(UserDaoProxy.class);
        // 调用userMapper的方法
        CommonPo commonPo = userMapper.findUserByIdResultMap(1);
        System.out.println(commonPo);
    }

    /**
     * 功能描述: 根据用户名称查询用户信息
     *
     * @params: []
     * @return: void
     * @author: zhaoerhu
     * @date: 2019/7/20 20:57
     */
    @Test
    public void testFindUserByName() throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建UserMapper对象，mybatis自动生成mapper代理对象
        UserDaoProxy userMapper = sqlSession.getMapper(UserDaoProxy.class);
        // 调用userMapper的方法
        List<CommonPo> list = userMapper.findUserByNameProxy("赵二虎");
        sqlSession.close();
        System.out.println(list);
    }
}

```

# 3. mybatis代理开发-级联DAO
## 3.1 定义mapper接口
zeh.mybatis.code04proxy.dao.OrdersCustomDaoProxy
```java
package zeh.mybatis.code04proxy.dao;

import zeh.mybatis.code00common.po.more.MarshallOrders;
import zeh.mybatis.code00common.po.more.MarshallOrdersCustom;
import zeh.mybatis.code00common.po.singleton.CommonPo;

import java.util.List;

/**
 * mapper代理方式开发 mapper接口即dao接口。
 * <p>
 * mybatis代理方式开发dao接口，不要使用泛型定义接口，直接指定类型。
 *
 * @类名称： OrdersCustomDaoProxy
 * @作者： zhaoerhu
 * @创建时间： 2019-4-3
 * @修改时间： 2019-4-3上午11:22:36
 * @备注：尊重每一行代码！
 */
public interface OrdersCustomDaoProxy {

    // 查询订单关联查询用户信息，属于一对一的查询。
    public List<MarshallOrdersCustom> findOrdersUser() throws Exception;

    // 查询订单关联查询用户，属于一对一的查询。使用resultMap。
    public List<MarshallOrders> findOrdersUserResultMap() throws Exception;

    // 查询订单(关联用户)及订单明细，属于一对多的查询。使用resultMap。
    public List<MarshallOrders> findOrdersAndOrderDetailResultMap() throws Exception;

    // 查询用户购买商品信息，多对多查询。使用resultMap。
    public List<CommonPo> findUserAndItemsResultMap() throws Exception;

    // 查询订单关联查询用户，用户信息是延迟加载
    public List<MarshallOrders> findOrdersUserLazyLoading() throws Exception;

}

```
## 3.2 resource下定义mapper.xml
mybatis/mapper/mybatis-proxy-mappingfind-mapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- 本文件是mapper代理方式开发DAO，完成所有一对一、一对多、多对多的关联查询。 -->
<mapper namespace="zeh.mybatis.code04proxy.dao.OrdersCustomDaoProxy">

    <!-- resultMap对关联查询的结果集进行高级映射。 -->

    <!-- 一对一查询结果映射：使用 association 完成关联查询，将关联查询信息映射到pojo对象中。 -->
    <!-- 订单查询关联用户的resultMap 将整个查询的结果映射到demo00.mybatis.model.common.mapping.po.Orders中 -->
    <!-- 通过专门定义resultMap映射，使用 association 标签直接将一对一查询的用户结果映射到订单对象的用户属性中。 -->
    <!-- id：该resultMap的表示id，必须整个mapperz.xml中唯一。 -->
    <resultMap type="zeh.mybatis.code00common.po.more.MarshallOrders"
               id="OrdersUserResultMap">
        <!-- 配置映射的订单信息 -->
        <!-- id：指定查询列中的唯 一标识，订单信息的中的唯 一标识，如果有多个列组成唯一标识，配置多个id 。 -->
        <!-- column：订单信息的唯 一标识 列 。 -->
        <!-- property：订单信息的唯 一标识 列所映射到Orders中哪个属性。 -->
        <id column="id" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="number" property="number"/>
        <result column="createtime" property="createtime"/>
        <result column="note" property="note"/>

        <!-- 配置映射关联的用户信息 -->
        <!-- association：用于映射关联查询单个对象的信息 ，即表示进行关联查询单条记录。 -->
        <!-- property：要将关联查询的用户信息映射到Orders中哪个属性 ，即表示关联查询的结果存储在Orders的哪个属性中。 -->
        <!-- javaType：表示关联查询的结果类型。 -->
        <association property="user"
                     javaType="zeh.mybatis.code00common.po.more.MarshallUser">
            <!-- id：表示查询结果集中唯一标识 ，即主键，如果主键是组合列，则id写多个映射。 -->
            <!-- column：查询出来的列名 property：type指定的pojo类型中的成员名称。 -->
            <!-- property：结果即对应的pojo中的对应属性成员的名称。 -->
            <!-- 最终resultMap对column和property作一个映射关系 （对应关系），表示将sql查询出来的字段映射到指定的pojo类属性上 -->
            <id column="user_id" property="id"/>
            <!-- result：对普通列的映射定义 -->
            <!-- column：sql查询出来的列名，普通列 -->
            <!-- property：type指定的pojo类型中的属性名 -->
            <!-- 最终resultMap对column和property作一个映射关系 （对应关系） -->
            <result column="username" property="username"/>
            <result column="sex" property="sex"/>
            <result column="address" property="address"/>
        </association>

    </resultMap>

    <!-- 一对多查询结果映射：使用collection完成关联查询，将关联查询信息映射到集合对象。 -->
    <!-- 订单及订单明细的resultMap 使用extends继承，不用在中配置订单信息和用户信息的映射 -->
    <!-- 对于跨mapper的resultMap的继承或者引用，需要在对应的目标resultMap的id名前添加目标mapper的namespace作为前缀进行引用 -->
    <resultMap type="zeh.mybatis.code00common.po.more.MarshallOrders"
               id="OrdersAndOrderDetailResultMap" extends="OrdersUserResultMap">
        <!-- 订单信息 -->
        <!-- <id column="id" property="id" /> <result column="user_id" property="userId"
            /> <result column="number" property="number" /> <result column="createtime"
            property="createtime" /> <result column="note" property="note" /> -->
        <!-- 用户信息 -->
        <!-- <association property="user" javaType="demo00.mybatis.model.common.mapping.po.User">
            <id column="user_id" property="id" /> <result column="username" property="username"
            /> <result column="sex" property="sex" /> <result column="address" property="address"
            /> </association> -->
        <!-- 上面的订单信息和用户信息的映射完全不用配置，resultMap支持继承，使用extends继承，不用再次配置订单信息和用户信息的映射。 -->

        <!-- 订单明细信息 一个订单关联查询出了多条明细，要使用collection进行映射。 -->
        <!-- collection：对关联查询到多条记录映射到集合对象中 。 -->
        <!-- property：将关联查询到多条记录映射到demo00.mybatis.model.common.mapping.po.Orders哪个属性 -->
        <!-- ofType：指定映射到list集合属性中pojo的类型，即关联查询出来的结果类型。 -->
        <!-- 注意，collection标签和association标签很类似，association标签指定一对一，collection标签指定一对多的结果映射。 -->
        <collection property="orderdetails"
                    ofType="zeh.mybatis.code00common.po.more.MarshallOrderDetail">
            <!-- id：订单明细唯 一标识 -->
            <!-- property:要将订单明细的唯 一标识 映射到demo00.mybatis.model.common.mapping.po.Orderdetail"的哪个属性 -->
            <id column="orderdetail_id" property="id"/>
            <result column="items_id" property="itemsId"/>
            <result column="items_num" property="itemsNum"/>
            <result column="orders_id" property="ordersId"/>
        </collection>

    </resultMap>

    <!-- 多对多查询结果映射： -->
    <!-- 查询用户及购买的商品 -->
    <resultMap type="zeh.mybatis.code00common.po.more.MarshallUser"
               id="UserAndItemsResultMap">
        <!-- 用户信息 -->
        <id column="user_id" property="id"/>
        <result column="username" property="username"/>
        <result column="sex" property="sex"/>
        <result column="address" property="address"/>

        <!-- 订单信息 一个用户对应多个订单，使用collection映射 -->
        <collection property="ordersList"
                    ofType="zeh.mybatis.code00common.po.more.MarshallOrders">
            <id column="id" property="id"/>
            <result column="user_id" property="userId"/>
            <result column="number" property="number"/>
            <result column="createtime" property="createtime"/>
            <result column="note" property="note"/>

            <!-- 订单明细 一个订单包括 多个明细 -->
            <collection property="orderdetails"
                        ofType="zeh.mybatis.code00common.po.more.MarshallOrderDetail">
                <id column="orderdetail_id" property="id"/>
                <result column="items_id" property="itemsId"/>
                <result column="items_num" property="itemsNum"/>
                <result column="orders_id" property="ordersId"/>

                <!-- 商品信息 一个订单明细对应一个商品 -->
                <association property="items"
                             javaType="zeh.mybatis.code00common.po.more.MarshallItems">
                    <id column="items_id" property="id"/>
                    <result column="items_name" property="name"/>
                    <result column="items_detail" property="detail"/>
                    <result column="items_price" property="price"/>
                </association>
            </collection>
        </collection>

    </resultMap>


    <!-- 一对一查询：查询所有订单信息，关联查询下单用户信息。 -->
    <!-- 通过订单关联查询用户信息：查询条件从订单表 ZEH_ORDERS 而来，关联查询ZEH_MYUSER 表。 -->
    <!-- 注意：因为一个订单信息只会是一个人下的订单，所以从查询订单信息出发关联查询用户信息为一对一查询。 -->
    <!-- 如果从用户信息出发查询用户下的订 单信息则为一对多查询，因为一个用户可以下多个订单。 -->
    <!-- 方式一：定义专门的订单用户PO即OrdersCustom，其中OrdersCustom通过继承Orders订单信息实现用户信息的扩展，使用resultType进行映射。 -->
    <!-- 方式一实现的一对一关联查询，需要定义专门的PO类映射查询结果，不需要结果对应的PO类之间相互依赖，也不需要定义复杂的resultMap。 -->
    <!-- 方式一定义了专门的po类作为输出类型，其中定义了sql查询结果集所有的字段。此方法较为简单，企业中使用普遍。 -->
    <select id="findOrdersUser" resultType="zeh.mybatis.code00common.po.more.MarshallOrdersCustom">
		select a.*, b.username,
		b.sex, b.address
		from zeh_orders a, zeh_myuser b
		where a.user_id = b.id
	</select>

    <!-- 一对一查询：查询所有订单信息，关联查询下单用户信息。 -->
    <!-- 方式二：使用resultmap ，定义专门的resultMap用于映射一对一查询结果 -->
    <!-- 方式二实现需要在Orders类中加入User属性，user属性中用于存储关联查询的用户信息。 -->
    <!-- 因为订单关联查询用户是一对一关系，所以这里使 用单个User对象存储关联查询的用户信息。 -->
    <select id="findOrdersUserResultMap" resultMap="OrdersUserResultMap">
		select a.*,
		b.username, b.sex, b.address
		from zeh_orders a, zeh_myuser b
		where
		a.user_id = b.id
	</select>


    <!-- 一对多查询：查询订单关联查询用户及订单明细，使用resultmap -->
    <!-- 注意如果使用resultType的话同样需要单独定义一个实体类POJO进行映射。 -->
    <select id="findOrdersAndOrderDetailResultMap" resultMap="OrdersAndOrderDetailResultMap">
		SELECT
		a.*,
		b.username,
		b.sex,
		b.address,
		c.id orderdetail_id,
		c.items_id,
		c.items_num,
		c.orders_id
		FROM zeh_orders a, zeh_myuser b,
		zeh_orderdetail c
		WHERE a.user_id = b.id
		AND c.orders_id = a.id
	</select>

    <!-- 多对多查询：查询用户及购买的商品信息，使用resultmap -->
    <!-- 注意多对多查询往往是多个表之间的关系，而且至少三张表，其中通过一个中间表来关联。 -->
    <!-- 本例中，orderdetail明细表就是那张中间表，关联 orders订单表和 items商品表。 -->
    <select id="findUserAndItemsResultMap" resultMap="UserAndItemsResultMap">
		SELECT a.*,
		b.username,
		b.address,
		c.id orderdetail_id,
		c.items_id,
		c.items_num,
		d.name items_name,
		d.detail items_detail
		FROM zeh_orders a, zeh_myuser
		b, zeh_orderdetail c, zeh_items d
		WHERE a.user_id = b.id
		AND a.id =
		c.orders_id
		AND c.items_id = d.id
	</select>

    <!-- 延迟加载的resultMap -->
    <resultMap type="zeh.mybatis.code00common.po.more.MarshallOrders"
               id="OrdersUserLazyLoadingResultMap">
        <!--对订单信息进行映射配置 -->
        <id column="id" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="number" property="number"/>
        <result column="createtime" property="createtime"/>
        <result column="note" property="note"/>
        <!-- 实现对用户信息进行延迟加载 select：指定延迟加载需要执行的statement的id（是根据user_id查询用户信息的statement）
            要使用userMapper.xml中findUserById完成根据用户id(user_id)用户信息的查询，如果findUserById不在本mapper中需要前边加namespace
            column：订单信息中关联用户信息查询的列，是user_id 关联查询的sql理解为： SELECT orders.*, (SELECT username
            FROM USER WHERE orders.user_id = user.id)username, (SELECT sex FROM USER
            WHERE orders.user_id = user.id)sex FROM orders -->
        <association property="user"
                     javaType="zeh.mybatis.code00common.po.more.MarshallUser"
                     select="cn.itcast.mybatis.mapper.UserMapper.findUserById"
                     column="user_id">
            <!-- 实现对用户信息进行延迟加载 -->
        </association>

    </resultMap>

    <!-- 查询订单关联查询用户，用户信息需要延迟加载 -->
    <select id="findOrdersUserLazyLoading" resultMap="OrdersUserLazyLoadingResultMap">
		SELECT * FROM
		orders
	</select>
</mapper>
```
## 3.3 运行
zeh.mybatis.code04proxy.run.OrdersCustomDaoProxyRun
```java
package zeh.mybatis.code04proxy.run;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
import zeh.mybatis.code00common.po.more.MarshallOrders;
import zeh.mybatis.code00common.po.more.MarshallOrdersCustom;
import zeh.mybatis.code04proxy.dao.OrdersCustomDaoProxy;

import java.io.File;
import java.io.InputStream;
import java.util.Iterator;
import java.util.List;

public class OrdersCustomDaoProxyRun {
    private SqlSessionFactory sqlSessionFactory;

    // 此方法是在执行testFindUserById之前执行
    @Before
    public void setUp() throws Exception {
        // mybatis配置文件
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        // 得到配置文件流
        InputStream inputStream = Resources.getResourceAsStream(resource);
        // 创建会话工厂，传入mybatis的配置文件信息
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    }

    /**
     * 一对一查询：测试findOrdersUser方法。
     *
     * @throws Exception
     */
    @Test
    public void testFindOrdersUser() throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        OrdersCustomDaoProxy userMapper = sqlSession.getMapper(OrdersCustomDaoProxy.class);
        List<MarshallOrdersCustom> list = userMapper.findOrdersUser();
        Iterator<MarshallOrdersCustom> iterator = list.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    /**
     * 一对一查询：测试findOrdersUserResultMap方法。
     *
     * @throws Exception
     */
    @Test
    public void testFindOrdersUserResultMap() throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        OrdersCustomDaoProxy userMapper = sqlSession
                .getMapper(OrdersCustomDaoProxy.class);
        List<MarshallOrders> list = userMapper.findOrdersUserResultMap();
        Iterator<MarshallOrders> iterator = list.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    /**
     * 一对多查询：测试findOrdersAndOrderDetailResultMap方法。
     *
     * @throws Exception
     */
    @Test
    public void testFindOrdersAndOrderDetailResultMap() throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        OrdersCustomDaoProxy userMapper = sqlSession
                .getMapper(OrdersCustomDaoProxy.class);
        List<MarshallOrders> list = userMapper.findOrdersAndOrderDetailResultMap();
        Iterator<MarshallOrders> iterator = list.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

}
```