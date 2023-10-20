---
layout:     post
title:      mybatis的foreach标签
subtitle:   测试mybatis的foreach标签
categories: [mybatis]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. mybatis实现的DAO
该DAO主要用来测试mybatis的foreach标签。       
全局配置文件和db.propertis将不再进行重复配置。   
zeh.mybatis.code02mybatis.dao.MybatisForeachDao
```java
package zeh.mybatis.code02mybatis.dao;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import zeh.mybatis.code00common.entity.ZehUser;
import zeh.mybatis.code00common.po.singleton.CommonPo;

import java.io.File;
import java.io.InputStream;
import java.util.List;
import java.util.Map;

// 测试mybatis的foreach标签
public class MybatisForeachDao {

    // 根据in关键字执行查询操作，主要观察${}和#{}的区别，本次使用#{}，传递的是一个list对象。
    public List<ZehUser> findUserByInForeach(List<ZehUser> paramList) throws Exception {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<ZehUser> list = sqlSession.selectList("myforeach.findUserByInForeach", paramList);
        sqlSession.close();
        return list;
    }


    // 根据in关键在执行查询操作，主要观察${}和#{}的区别。本次使用#{}，传递一个Map，map中引用一个list。
    public List<ZehUser> findUserByIn2Foreach(Map<String, List<String>> map) throws Exception {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<ZehUser> list = sqlSession.selectList("myforeach.findUserByIn2Foreach", map);
        sqlSession.close();
        return list;
    }

    // 根据in关键在执行查询操作，主要观察${}和#{}的区别
    // 本次使用#{}，但是同样接收的是一个大字符串，主要是在statementid里面进行分割处理。
    public List<ZehUser> findUserByIn4(Map<String, String> map) throws Exception {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<ZehUser> list = sqlSession.selectList("myforeach.findUserByIn4Foreach", map);
        sqlSession.close();
        return list;
    }

    // 直接传递简单类型
    public List<ZehUser> findUserByIn5(String[] parameter) throws Exception {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<ZehUser> list = sqlSession.selectList("myforeach.findUserByIn5Foreach", parameter);
        sqlSession.close();
        return list;
    }

    // 直接传递一个map，map中包装简单类型，测试foreach标签。
    public List<ZehUser> findUserByIn6(Map<String, String> map) throws Exception {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<ZehUser> list = sqlSession.selectList("myforeach.findUserByIn6Foreach", map);
        sqlSession.close();
        return list;
    }

    // 测试foreach强大的逻辑拼凑动态sql的能力。
    public List<CommonPo> findUserByIn7(Map<String, String> map) throws Exception {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("myforeach.findUserByIn7Foreach", map);
        sqlSession.close();
        return list;
    }

    // 传入list<Map>复杂条件，观察foreach标签的逻辑拼凑能力。
    // 切记：foreach的collection属性本质上只能接收list或者数组。
    public List<CommonPo> findUserByIn8(Map<String, Object> listMap) throws Exception {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("myforeach.findUserByIn8Foreach", listMap);
        sqlSession.close();
        return list;
    }


    // foreach循环map中的list的map
    public List<CommonPo> findUserByForeach(Map<String, List<Map<String, String>>> paramMap) throws Exception {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("myforeach.findUserByForeachForeach", paramMap);
        sqlSession.close();
        return list;
    }


    // foreach 在 insert 操作中的循环作用（批量插入）
    public void insertZehUserByForeach(List<CommonPo> userPos) throws Exception {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        sqlSession.insert("myforeach.insertZehUserByForeach", userPos);
        sqlSession.commit();
        System.out.println("insert操作成功提交");
        sqlSession.close();
    }
}

```
上述案例最后一个方法 insertZehUserByForeach，其主要测试 foreach 在 insert 操作中的循环作用（批量插入）. 注意如下：     
```youtrack
foreach原则上只能在一个完整的sql语句中执行，如果foreach拼接的是多个sql语句（;隔开），则需要做特殊处理。
oracle：
//每条sql以分号结束，end也是分号结束
<delete id = "delete">
begin
delete from table1 where id = 1;
delete from table2 where id = 2;
end;
<delete>
<p>
Mysql:
数据连接开启多条语句处理
在url后面加入allowMultiQueries=true
url: jdbc:mysql://127.0.0.1:3306/user?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&allowMultiQueries=true`
<p>
//每条sql以分号结束
<delete id="delete">
delete from table1 where id = 1;
delete from table2 where id = 2;
<delete>
<p>
只有这样，才能正确处理。
```

# 2. resource下配置mapper.xml
mybatis/mapper/mybatis-first-foreach-mapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="myforeach">

    <!-- 1.根据in查询，主要观察#{}和${}在in中对于引号的区别 ，本次对应的是#{}，对于in操作而言，使用#{}复杂，因为#{}会自动为传入的string类型的参数加上''，需要遍历去拼装。 -->
    <!-- 2.此 statementId 接收的是list，所以collection必须是list。 -->
    <!-- 3.mybatis巧妙的利用foreach标签遍历list列表或者数组，然后动态的拼接出指定格式的sql片段。 -->
    <!-- 4.注意，foreach标签很灵活，其中的collection属性就是接收一个列表对象、数组或者map对象的，当然，collection属性也支持表达式。 -->
    <!-- 5.如果传递进来的就是一个大字符串，那么collection可以写成：collection="mystr.split(',')"。 -->
    <!-- 即：mybatis的标签中也提供了很多内置函数，其中split(',')就是mybatis提供的字符串分割函数，其参数是分割符号，返回值是是一个列表容器。 -->
    <!-- 6.foreach标签一共支持三种类型，分别为list，array，map三种。-->
    <!-- 7.foreach标签属性详解：-->
    <!-- (1)、item：foreach循环体中的具体对象。支持属性的点路径访问,比如item.age,item.age等，注意，点路径访问和直接传入pojo是一样的原理，通过XXX反射getXXX或者isXXX-->
    <!--      的XXX来获得对应的值（而不是直接反射对象中的属性）。说明：item在list和array中是其中包含的对象；在map中是其对应的value。 该参数为必选。-->
    <!-- (2)、foreach：作为foreach的输入对象，list对象默认使用list作为键，数组对象默认使用array作为键，mybatis将输入参数默认封装成map对象。 该参数为必选。-->
    <!--      意味着mybatis默认将list,array都封装成map对象来传入，其中键都有默认的取值，这也是为什么collection要默认取值list,array的原因。-->
    <!--      当然在作为入参时可以使用@Param()来为参数指定别名，指定别名后，list,array将失效，collection中必须使用别名来取值。-->
    <!--      除了直接使用list,array等，还有一种参数作为对象的属性传入的，比如pojo.ids作为参数传入，其中pojo对象的ids是个list对象。-->
    <!--      或者是map作为对象包含了一个list传进去，则直接根据map对应的key取出list等。-->
    <!-- collection 总结：collection属性到底取值为什么，具体要看对哪个对象做循环。-->
    <!-- collection 也是通过反射入参的getXXX、isXXX或者直接根据map的key值去遍历map元祖通过get进行取值的。 -->
    <!-- 如果是pojo则根据isXXX或者getXXX的XXX进行方法的反射从而获取值；如果不存在getXXX或者isXXX方法，则直接反射XXX的属性值；如果XXX属性也不存在，则报错。 -->
    <!-- (3)、separator：元素之间的分隔符，例如在in()的时候，separator=","会自动在元素中间用“,“隔开，避免手动输入逗号导致sql错误，如in(1,2,)这样。该参数可选。-->
    <!-- (4)、open：foreach代码的开始符号，一般是(和close=")"合用。常用在in(),values()时。该参数可选。-->
    <!-- (5)、close：foreach代码的关闭符号，一般是)和open="("合用。常用在in(),values()时。该参数可选。-->
    <!-- (6)、index：在list和数组中,index是元素的序号，在map中，index是元素的key，该参数可选。-->
    <!-- 8.parameterType，直接指定list，其实不用指定也会默认根据实际传入的类型自动确认。但是resultType为list时，指定的是其中元素的类型。-->
    <select id="findUserByInForeach" parameterType="java.util.List"
            resultType="zeh.mybatis.code00common.entity.ZehUser">
        <!-- SELECT * FROM ZEH_USER WHERE NAME IN (#{value}) -->
        SELECT * FROM ZEH_USER
        <if test="_parameter != null">
            WHERE NAME IN
            <foreach collection="list" item="item" index="index" open="(" close=")" separator=",">
                #{item.name}
            </foreach>
        </if>
    </select>

    <!-- 1.此statementid接收的是一个map，且该map中key为myParamMap，对应的value是一个list，所以此处collection为map中的key即myParamMap。-->
    <!-- 2.实际此处的collection属性接收的是一个list，不过通过map方式传递进来的则不再使用默认的list命名，而是用自己维护的map中对应的key。 -->
    <!-- 3.同时下面不指定parameterType，则默认此处传递进来的参数是map类型，此处的parameterType默认就是map类型。 -->
    <!-- 4.再次强调：collection属性接收3种类型，list、array、map，且默认都是封装成对应的map对象进行的，默认的key值就是list、array、map等。-->
    <!-- 一旦自己手动封装了map的话，那么默认的key名称将失效。-->
    <select id="findUserByIn2Foreach" parameterType="java.util.Map"
            resultType="zeh.mybatis.code00common.entity.ZehUser">
        SELECT * FROM ZEH_USER
        <include refid="test123456"/>
    </select>
    <!-- 通用sql片段，通过include标签进行引用-->
    <sql id="test123456">
        WHERE 1=1
        <!-- 增加判断，当传入的map对象中的key不为空时（这个key是个list），才执行in操作，否则in操作将报错 -->
        <if test="myListMap!=null and !myListMap.isEmpty()">
            AND NAME IN
            <foreach collection="myListMap" item="item" index="index" open="(" close=")" separator=",">
                #{item}
            </foreach>
        </if>
    </sql>

    <!-- 1.此statementid接收的是字符串，主要是通过foreach进行分割，将大的字符串根据分割符分割为数组。 -->
    <!-- 2.分割时，collection属性指定的是mybatis的表达式，借助了mybatis标签内置的分割函数split()，该函数返回一个list容器。 -->
    <!-- 3.注意#{}会自动为字符串类型加上''。 -->
    <select id="findUserByIn4Foreach" resultType="zeh.mybatis.code00common.entity.ZehUser">
        <!-- select * from zeh_user where name in ( <foreach collection="gggg.split(',')"
            item="item" index="index" separator=","> #{item} </foreach> ) -->
        select * from
        <foreach collection="gggg.split('\\$\\$\\$\\$')" item="item"
                 open="(" close=") t" separator=" union ">
            <choose>
                <when test="item.indexOf('@@@@')>0">
                    select t.*,0 isExtend
                    <!-- 通过单循环截取并修改item的命名 -->
                    <foreach collection="item.split('@@@@')" item="item">
                        <include refid="handlerSubCorpIdWhere"/>
                    </foreach>
                </when>
                <otherwise>
                    select t.*,1 isExtend
                    <include refid="handlerSubCorpIdWhere"/>
                </otherwise>
            </choose>

        </foreach>
        where t.id='104' and rownum &lt; 3 order by t.id
    </select>
    <sql id="handlerSubCorpIdWhere">
        from zeh_user t
        where 1=1
        <if test="item != null and item != ''">
            and name =
            #{item}
        </if>
    </sql>

    <!-- 1.传递一个数组，验证foreach遍历数组的情况。 -->
    <!-- 2.foreach遍历标签中，除了collection属性和item属性必须存在，其他的属性都是非必须的。 -->
    <!-- 3.着重强调index属性：index表示foreach标签遍历的循环次数，从0开始；切记，取值一定是取的item。-->
    <!-- 4.而不是index，因为index取出来实际上是从0开始的下标。 -->
    <!-- 5.如果是单个传递数组对象，则 resultType 是数组中元素的类型，比如是字符串数组，则 resultType 为String。 -->
    <select id="findUserByIn5Foreach" resultType="zeh.mybatis.code00common.entity.ZehUser">
        select * from
        <foreach collection="array" item="item" open="(" close=") t" separator=" union " index="myindex">
            select * from zeh_user where name = #{item}
            <!-- 注意观察日志，看index输出是什么 -->
            or id = #{myindex}
        </foreach>
    </select>

    <!-- 1.传递一个map，map中全部是简单类型：验证后，发现foreach要想遍历map本身比较复杂。 -->
    <!-- 2.如果使用_parameter.keys只能取出map中所有的key，此时index和之前一样表示遍历的次数。 -->
    <!-- 3.如果使用_parameter.values只能取出map中所有的value，此时index和之前一样表示遍历的次数。 -->
    <!-- 4.如果使用_parameter.entrySet()则能取出所有的key和value，此时item表示的就是value，index表示的是key，一劳永逸！！！ -->
    <select id="findUserByIn6Foreach" resultType="zeh.mybatis.code00common.entity.ZehUser">
        select * from
        <!-- <foreach collection="_parameter.keys" item="value" open="(" close=")
            t" separator=" union " index="index"> select * from zeh_user where name =
            #{value} and id = #{index} </foreach> -->

        <!-- <foreach collection="_parameter.values" item="value" open="(" close=")
            t" separator=" union " index="index"> select * from zeh_user where name =
            #{value} and id = #{index} </foreach> -->
        <foreach collection="_parameter.entrySet()" item="item" open="("
                 close=") t" separator=" union " index="index">select * from zeh_user where
            name = #{item} or id = #{index}
        </foreach>
    </select>

    <!-- 1.验证foreach标签强大的逻辑处理能力： -->
    <!-- 2.传递的字符串是："ssg:1$$$$ddf:2$$$$jjk:3$$$$iio:4$$$$zhaoeh:5$$$$yurui:6"; -->
    <!-- and name="" -->
    <!-- and id="" -->
    <select id="findUserByIn7Foreach" resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from
        <foreach collection="corpIds.split('\\$\\$\\$\\$')" item="corpId"
                 open="(" close=") t" separator=" union ">
            select * from zeh_user
            where 1=1 and
            <foreach collection="corpId.split(':')" item="dataAuthFlag"
                     separator=" and ">
                <choose>
                    <!-- mybatis的choose-when-otherwise标签实现了if-else if-else的功能 -->
                    <when test="dataAuthFlag != null and dataAuthFlag.length() > 1">
                        <!-- 借助foreach可以实现只循环一次，要求：collection属性必须是list或者数组 -->
                        <!-- mybatis没有提供字符串转换成数组的函数，所以此处借助字符串的内置方法split()来转换成数组 -->
                        <!-- split()方法必须设置分割符，不能设置为''，如果是''将会按照''进行分割 -->
                        <!-- 应该结合业务传入一个分割符，确保一个字符串分割出来只有一个，从而达到实现单循环的目的 -->
                        <!-- 此处为何非要实现单循环呢？ -->
                        <!-- 解释：目的是为了使用foreach标签来对传入的字符串进行重命名。。。 -->
                        <!-- 即此处使用foreach单循环将dataAuthFlag这个item重新命名为了corpId。 -->
                        <!-- 因为有时候sql内部引用的其他公共的sql片段中是在使用corpId，我们不想改公共的，所以此处必须按照条件进行命名 -->
                        <foreach collection="dataAuthFlag.split('\\$\\$\\$\\$')"
                                 item="corpId">
                            name=#{corpId}
                        </foreach>
                    </when>
                    <otherwise>
                        <!-- 此处在这个条件中同样对截取后的字符串使用foreach单循环进行了重命名 -->
                        <foreach collection="dataAuthFlag.split('\\$\\$\\$\\$')"
                                 item="dataAuthFlag">
                            id=#{dataAuthFlag}
                        </foreach>
                    </otherwise>
                </choose>
            </foreach>
        </foreach>
    </select>

    <!-- 重点！！！ -->
    <!--传入一个List<Map>： list中每一个map存放：corpId,dataAuthFlag,dataType 。-->
    <!-- 结合foreach进行遍历。 -->
    <!-- 这个配置好好理解：主要是foreach如何遍历list中的map，浪费我几个小时，现在半夜了都。 -->
    <!-- 要获取入参list<Map>中的map对应的value值，则可以使用item.key来遍历获取每一个list中的元素（此时是item）的对应的key的value，即item.key。 -->
    <select id="findUserByIn8Foreach" resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from
        <foreach collection="listMap" item="items" open="(" close=") t"
                 separator=" union " index="myindex">
            select * from zeh_user
            where 1=1
            <!-- 注意：list遍历后，其中的每一个元素是一个map，然后此处使用foreach来遍历当前map的key和value -->
            <foreach collection="items.entrySet()" item="value" index="key"
                     separator="">
                <!-- mybatis在遍历map时，取出的key和value如果报错NumberFormatException则手动通过toString()函数进行转换 -->
                <!-- 这个问题困扰了很久，我本地不进行toString()转换是可以的，加上toString()报错，但是工作上不加报错，加上不报错，不知道是不是mybatis的版本原因 -->
                <!-- 建议加上toString() -->
                <if test="key.toString() == 'corpId' and value != null and value != ''">
                    <!-- foreach单循环换名 -->
                    <foreach collection="value.split('\\$\\$\\$\\$')" item="corpId">
                        and
                        name=#{corpId}
                    </foreach>
                </if>
                <if test="key == 'dataAuthFlag' and value != null and value != ''">
                    <!-- foreach单循环换名 -->
                    <foreach collection="value.split('\\$\\$\\$\\$')" item="dataAuthFlag">
                        <!-- and id=#{dataAuthFlag} -->
                    </foreach>
                    <if test="dataAuthFlag != null and dataAuthFlag != ''">
                        and id=#{dataAuthFlag}
                        and sex=#{dataType}
                        and age=#{age}
                    </if>
                </if>
                <if test="key == 'dataType' and value != null and value != ''">
                    <!-- foreach单循环换名 -->
                    <foreach collection="value.split('\\$\\$\\$\\$')" item="dataType">
                        <!-- and sex=#{dataType} -->
                    </foreach>
                </if>
            </foreach>
        </foreach>
    </select>

    <!-- 1.接收一个map,map中是list，list中又是一堆map。-->
    <!-- 2.不要着急，仔细分析。首先入参是map，而传入的是map中的list，所以使用map的自定义key指定collection。其次list中的item是个map，所以可以再次进行遍历。-->
    <select id="findUserByForeachForeach" resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        <foreach collection="myParamMap" item="map" index="index"
                 open="(" close=")" separator=" UNION ALL ">
            SELECT * FROM ZEH_USER T WHERE 1=1
            <foreach collection="map.entrySet()" item="value" index="key"
                     open="" close="" separator="">
                <if test="key == 'name'">
                    AND
                    T.NAME =
                    #{value}
                </if>
                <if test="key == 'sex'">
                    AND T.SEX = #{value}
                </if>
            </foreach>
        </foreach>
    </select>

    <!-- foreach 循环在insert中实现批量插入 -->
    <insert id="insertZehUserByForeach" parameterType="list">
        begin
        <foreach collection="list" item="user" separator=";" index="index" open="" close=";">
            insert into zeh_user(id,name,age,sex,hobby)
            values(#{user.id},#{user.name},#{user.age},#{user.sex},#{user.hobby})
        </foreach>
        end;
    </insert>
</mapper>
```

# 3. 运行
foreach原理和java的循环一样，就是重复执行某个完全相同的动作，然后将每次执行的结果进行处理。   
处理方式：   
（1）、经常将遍历中每次拿到的结果输出。   
（2）、经常将遍历中每次拿到的结果缓存起来。   
（3）、经常将遍历中每次拿到的结果进行拼接，拼接成一个大的字符串（mybatis的foreach就是这种，使用 separator 指定拼接符）。   

zeh.mybatis.code02mybatis.run.MybatisForeachDaoRun
```java
package zeh.mybatis.code02mybatis.run;

import org.junit.Before;
import org.junit.Test;
import zeh.mybatis.code00common.entity.ZehUser;
import zeh.mybatis.code00common.po.singleton.CommonPo;
import zeh.mybatis.code02mybatis.dao.MybatisForeachDao;

import java.util.*;

// 测试mybatis的foreach标签。
public class MybatisForeachDaoRun {
    private MybatisForeachDao foreachDao;

    //实例化dao对象
    @Before
    public void newInstanceFirstDao() {
        foreachDao = new MybatisForeachDao();
    }

    @Test
    public void findUserByInForeach() throws Exception {
        List<ZehUser> paramList = new ArrayList<ZehUser>();
        ZehUser ssg = new ZehUser();
        ssg.setName("ssg");
        ZehUser ddf = new ZehUser();
        ddf.setName("ddf");
        ZehUser jjk = new ZehUser();
        jjk.setName("jjk");
        ZehUser iio = new ZehUser();
        iio.setName("getMyName");
        ZehUser zhaoeh = new ZehUser();
        zhaoeh.setName("zhaoeh");
        ZehUser yurui = new ZehUser();
        yurui.setName("yurui");
        paramList.add(ssg);
        paramList.add(ddf);
        paramList.add(jjk);
        paramList.add(iio);
        paramList.add(zhaoeh);
        paramList.add(yurui);
        // 下面故意将param输入参数设置为null值，测试_parameter是否是Mybatis默认的输入参数对象，结论：_parameter参数就是mybatis默认的输入参数对象。
        // paramList = null;
        // 将 paramList 传递进去
        List<ZehUser> list = foreachDao.findUserByInForeach(paramList);
        Iterator<ZehUser> iterator = list.iterator();
        System.out.println("根据用户名称使用#{}查询到的users表的record如下：");
        while (iterator.hasNext()) {
            ZehUser zehUser = iterator.next();
            System.out.println("name = " + zehUser.getName() + ";age = " + zehUser.getAge() + ";sex = " + zehUser.getSex() + ";hobby = " + zehUser.getHobby());
        }
    }

    @Test
    public void findUserByIn2Foreach() throws Exception {
        List<String> param = new ArrayList<String>();
        Map<String, List<String>> paramMap = new HashMap<String, List<String>>();
        param.add("ssg");
        param.add("ddf");
        param.add("jjk");
        param.add("getMyName");
        param.add("zhaoeh");
        param.add("yurui");
        paramMap.put("myListMap", param);
        List<ZehUser> list = foreachDao.findUserByIn2Foreach(paramMap);
        Iterator<ZehUser> iterator = list.iterator();
        System.out.println("根据用户名称使用#{}查询到的users表的record如下：");
        while (iterator.hasNext()) {
            ZehUser zehUser = iterator.next();
            System.out.println("name = " + zehUser.getName() + ";age = " + zehUser.getAge() + ";sex = " + zehUser.getSex() + ";hobby = " + zehUser.getHobby());
        }
    }

    // 根据name在某个集合中，进行查询，主要验证#{}和${}的区别。
    // 本次仍旧使用#{}，而且不传递list或者array，或者map中封装的list，array等，本次就传递一个大的字符串，但是同样使用#{}进行处理，同样借助mybatis的foreach标签和collection属性指定组合规则，动态组成sql片段！
    @Test
    public void testFindUserByIn4() throws Exception {
        Map<String, String> paramMap = new HashMap<String, String>();
        // String sss = "'ssg','ddf','jjk','iio','zhaoeh','yurui'";
        // String sss = "ssg,ddf,jjk,iio,zhaoeh,yurui";

        /**
         * 下面这个大字符串的含义：
         *
         * 其实需要实现的sql是：
         *
         * select * from zeh_user where name = '' or name = '' or name = ''。。。
         *
         * 即查询zeh_user表中name为某个动态变化值的结果，这个name的参数根据字符串传递进去，不知道会有一个值。
         *
         * 所以，不建议使用or或者in，因为性能低导致索引失效。而采用union进行求并集去重的方式。 即：
         *
         * select * from zeh_user where name = ''
         *
         * union
         *
         * select * from zeh_user where name = ''
         *
         * union
         *
         * select * from zeh_user where name = ''
         *
         * ...
         *
         * 当然并集求完之后，后边可能还要对整个并集的过滤条件，所以要实现成如下的样子：
         *
         * select * from (
         *
         * select * from zeh_user where name = ''
         *
         * union
         *
         * select * from zeh_user where name = ''
         *
         * union
         *
         * select * from zeh_user where name = ''
         *
         * ... ) t
         *
         * where t.sex = .....
         *
         * 现在有了新需求，即传递的这个大字符串，按照$$$$分割成每一个name字段的值后，这个值可能会包含特殊字符“####”结尾，
         * 如果这个name包含这个特殊字符
         * ####，则查询时，先去掉这个特殊字符，然后所有按照该name查询出来的记录都应该携带一个字段isExtend且值为0
         * ；相反的其他的name条件查询出来的同样携带isExtend字段但是值为1
         * 。（注意，isExtend字段zeh_user表中没有，是需要我们自己组装的虚拟字段）！！！！
         */
        // String sss = "ssg$$$$ddf$$$$jjk####$$$$iio$$$$zhaoeh$$$$yurui";
        String sss = "2c9005e847aa52fb0147aa537dd50002$$$$364ae996eae24aebb3ac03fbeea89786@@@@$$$$afd6dbc5136b4aba8c30e30c2316b392";
        paramMap.put("gggg", sss);
        List<ZehUser> list = foreachDao.findUserByIn4(paramMap);
        Iterator<ZehUser> iterator = list.iterator();
        System.out.println("根据用户名称使用${}查询到的users表的record如下：");
        while (iterator.hasNext()) {
            ZehUser zehUser = iterator.next();
            System.out.println("name = " + zehUser.getName() + ";age = " + zehUser.getAge() + ";sex = " + zehUser.getSex() + ";hobby = " + zehUser.getHobby());
        }
    }

    // 直接传递一个数组，测试foreach标签。
    @Test
    public void testFindUserByIn5() throws Exception {
        String parameter[] = {"ssg", "ddf", "jjk", "iio", "zhaoeh"};
        List<ZehUser> list = foreachDao.findUserByIn5(parameter);
        Iterator<ZehUser> iterator = list.iterator();
        System.out.println("根据用户名称使用${}查询到的users表的record如下：");
        while (iterator.hasNext()) {
            ZehUser zehUser = iterator.next();
            System.out.println("name = " + zehUser.getName() + ";age = " + zehUser.getAge() + ";sex = " + zehUser.getSex() + ";hobby = " + zehUser.getHobby());
        }
    }

    // 传递一个简单类型的map：foreach标签解析map标签使用_parameter.entrySet()则一劳永逸！
    @Test
    public void testFindUserByIn6() throws Exception {
        Map<String, String> parameterMap = new HashMap<String, String>();
        parameterMap.put("01", "ssg");
        parameterMap.put("02", "ddf");
        parameterMap.put("03", "jjk");
        parameterMap.put("04", "iio");
        parameterMap.put("05", "getMyName");
        List<ZehUser> list = foreachDao.findUserByIn6(parameterMap);
        Iterator<ZehUser> iterator = list.iterator();
        System.out.println("根据用户名称使用${}查询到的users表的record如下：");
        while (iterator.hasNext()) {
            ZehUser zehUser = iterator.next();
            System.out.println("name = " + zehUser.getName() + ";age = " + zehUser.getAge() + ";sex = " + zehUser.getSex() + ";hobby = " + zehUser.getHobby());
        }
    }

    /**
     * 传递map，封装大字符串，利用foreach标签强大功能进行分割拼凑动态sql.
     * <p>
     * 需求是：传递一个大字符串，以$$$$分割，对应zeh_user表中的name字段；
     * <p>
     * 传递一个大字符串，以:分割，对应zeh_user表中的id字段。
     * <p>
     * 想要实现的sql是：
     * <p>
     * select * from zeh_user where 1=1 and name = "" and id = ""
     * <p>
     * union
     * <p>
     * select * from zeh_user where 1=1 and name = "" and id = ""
     * <p>
     * union
     * <p>
     * select * from zeh_user where 1=1 and name = "" and id = ""
     * <p>
     * union
     * <p>
     * select * from zeh_user where 1=1 and name = "" and id = ""
     * <p>
     * union并集的次数按照字符串只能中分割后的name字段使用foreach进行遍历。
     * <p>
     * 问题：mybatis的foreach标签虽然能够嵌套，也能够各自遍历对应的list或者数组，
     * 但是此处每次查询时都必须要保证name和id的关系是一一对应的。所以，只能将id字段以:的形式拼接在name那串长字符串中。
     *
     * @throws Exception
     */
    @Test
    public void testFindUserByIn7() throws Exception {
        Map<String, String> paramMap = new HashMap<String, String>();
        String corpIds = "ssg$$$$ddf$$$$jjk$$$$iio$$$$zhaoeh$$$$yurui$$$$";
        // String corpIds = "";
        // 对原始包含name的字符串进行拆分
        String[] corpIdArrays = corpIds.split("\\$\\$\\$\\$");
        corpIds = "";
        System.out.println(corpIdArrays.length);
        for (int i = 0; i < corpIdArrays.length; i++) {
            // 拆分后重组，重组的格式：ssg:1$$$$ddf:2$$$$jjk:3$$$$iio:4$$$$zhaoeh:5$$$$yurui:6
            corpIds = corpIds + corpIdArrays[i] + ":" + (i + 1) + "$$$$";
            System.out.println(corpIdArrays[i]);
        }
        System.out.println(corpIds);
        paramMap.put("corpIds", corpIds);
        List<CommonPo> list = foreachDao.findUserByIn7(paramMap);
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据用户名称使用${}查询到的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    // 难点！！！
    // 假设现在通过union对很多条sql语句取并集，然后每个sql语句where里面有三个条件，且这三个条件具备一一对应的关系，此时就应该使用List<Map>然后结合foreach标签去操作！
    // 三个要存储的key是：corpId,dataAuthFlag,dataType。
    // 这三个key必须是一个维度的。
    // 所以传递一个map，里面放置一个List<Map>对象。
    @Test
    public void testFindUserByIn8() throws Exception {
        Map<String, Object> parameterMap = new HashMap<String, Object>();
        // new一个list，其中放map
        List<Map<String, String>> listMap = new ArrayList<Map<String, String>>();
        String corpIds = "ssg$$$$ddf$$$$jjk$$$$iio$$$$zhaoeh$$$$yurui$$$$";
        //对原始包含name的字符串进行拆分，拆分后和对应的
        String[] corpIdArrays = corpIds.split("\\$\\$\\$\\$");
        for (int i = 0; i < corpIdArrays.length; i++) {
            // 每次进来要新建一个map对象，握草，日了狗了，2019-4-14在厦门排查这个问题搞了几个小时！！！！
            Map<String, String> paramMap = new HashMap<String, String>();
            String corpId = corpIdArrays[i];
            System.out.println(corpId);
            // 封装map
            paramMap.put("corpId", corpId);
            if (i == 3) {
                paramMap.put("corpId", null);
                paramMap.put("dataAuthFlag", Integer.toString(i + 1));
                paramMap.put("dataType", Integer.toString(i + 1) + "Test");
            }
            // 塞进list中
            listMap.add(paramMap);
        }
        System.out.println("=============:" + listMap);
        parameterMap.put("listMap", listMap);
        parameterMap.put("age", "22");
        List<CommonPo> list = foreachDao.findUserByIn8(parameterMap);
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据用户名称使用${}查询到的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    @Test
    public void testFindUserByForeach() throws Exception {
        List<Map<String, String>> param = new ArrayList<Map<String, String>>();
        Map<String, List<Map<String, String>>> paramMap = new HashMap<String, List<Map<String, String>>>();
        Map<String, String> map1 = new HashMap<String, String>();
        map1.put("name", "ssg");
        map1.put("sex", "女");
        Map<String, String> map2 = new HashMap<String, String>();
        map2.put("name", "jjk");
        map2.put("sex", "女");
        param.add(map1);
        param.add(map2);
        paramMap.put("myParamMap", param);
        List<CommonPo> list = foreachDao.findUserByForeach(paramMap);
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据用户名称使用#{}查询到的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    @Test
    public void testInsertZehUserByForeach() throws Exception {
        List<CommonPo> userPos = new ArrayList<>();
        CommonPo user1 = new CommonPo();
        CommonPo user2 = new CommonPo();
        user1.setId(110);
        user1.setName("批量1");
        user1.setAge(9);
        user1.setHobby("kanshu");
        user1.setSex("男");
        user2.setId(111);
        user2.setName("批量2");
        user2.setAge(12);
        user2.setHobby("kanshu");
        user2.setSex("男");
        userPos.add(user1);
        userPos.add(user2);
        foreachDao.insertZehUserByForeach(userPos);
    }
}
```