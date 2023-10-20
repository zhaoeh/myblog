---
layout:     post
title:      mybatis的if标签
subtitle:   测试mybatis的if标签
date:       2021-11-03
author:     zhaoeh
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - mybatis
---

# 1. mybatis实现的DAO
该DAO主要用来测试mybatis的if标签。       
全局配置文件和db.propertis将不再进行重复配置。   
zeh.mybatis.code02mybatis.dao.MybatisIfDao   
```java
package zeh.mybatis.code02mybatis.dao;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import zeh.mybatis.code00common.po.singleton.CommonPo;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.util.*;

// 测试mybatis的if标签
public class MybatisIfDao {
    
    // 测试if标签：输入参数为简单类型。
    public void findUserByIdIf() throws IOException {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("myif.findUserByIdIfString", "test");
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据id查询到的ZEH_USER表中的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
        sqlSession.close();
    }

    // 测试if标签：输入参数为list对象。
    public void findUserByIdIfList() throws IOException {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<String> allList = new ArrayList<>();
        allList.add("A");
        allList.add("B");
        allList.add("C");
        allList.add("D");
        List<CommonPo> list = sqlSession.selectList("myif.findUserByIdIfList", allList);
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据id查询到的ZEH_USER表中的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
        sqlSession.close();
    }


    // 测试if标签：输入参数为数组对象。
    public void findUserByIdIfArray() throws IOException {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        int[] array = new int[]{1, 2, 3, 4};
        List<CommonPo> list = sqlSession.selectList("myif.findUserByIdIfArray", array);
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据id查询到的ZEH_USER表中的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
        sqlSession.close();
    }

    // 测试if标签：输入参数为map对象。
    public void findUserByIdIfMap() throws IOException {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        Map<String, Object> testMap = new HashMap<>();
        testMap.put("name", "赵二虎");
        List<CommonPo> list = sqlSession.selectList("myif.findUserByIdIfMap", testMap);
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据id查询到的ZEH_USER表中的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
        sqlSession.close();
    }

    // 测试if标签：输入参数为pojo对象。
    public void findUserByIdIfPo() throws IOException {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<String> allList = new ArrayList<>();
        allList.add("test");
        CommonPo commonPo = new CommonPo();
        commonPo.setId(1);
        commonPo.setName("eric");
        commonPo.setAllList(allList);
        List<CommonPo> list = sqlSession.selectList("myif.findUserByIdIfPo", commonPo);
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据id查询到的ZEH_USER表中的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
        sqlSession.close();
    }
}

```

# 2. resource下面配置mapper.xml
mybatis/mapper/mybatis-first-if-mapper.xml   
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="myif">

    <!-- 1.输入参数是 简单类型，mybatis 将简单类型直接赋值给_parameter参数或者value参数，所以可以直接通过_parameter参数或者value参数进行取值。-->
    <!-- 2.对于输入简单类型，则 if 中取值只能通过默认的_parameter参数或者默认的value这个参数别名,直接对_parameter参数或者value进行boolean判断即可。-->
    <!-- 3._parameter 默认参数是所有输入参数的对象，在本例子中，_parameter参数指的就是传递进来的简单类型值。 -->
    <!-- 4.value参数是mybatis对简单类型的封装，它是Object类型的。-->
    <!-- 4.简单类型中没有getXXX或者isXXX方法，所以不能使用_parameter对象反射出对应的getXXX或者isXXX方法。-->
    <!-- 5.简单类型不是list对象，所以不能使用list.方法名 反射出对应的方法。-->
    <!-- 6.简单类型不是array对象，所以不能反射array的属性或者方法。-->
    <!-- 7.简单类型如果是String，则可以使用_parameter参数调用对应的String类中的方法，比如equals()方法，但是不推荐使用，不稳定。-->
    <!-- 7.if标签后面跟的是boolean表达式（最终结果要么是true要么是false）。-->
    <select id="findUserByIdIfString" parameterType="java.lang.String"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from zeh_user a
        where 1 = 1
        <if test="_parameter != null">
            and a.name=#{qitamingcheng}
        </if>
        <if test="value != null">
            and a.name='value'
        </if>
        <if test="value == null">
            and a.name='value22'
        </if>
        <if test="_parameter == 'test'.toString()">
            and a.name = 'equals'
        </if>
        <if test="_parameter.equals('test')">
            and a.name = 'equals22'
        </if>
    </select>

    <!-- 1.输入参数是list，mybatis 将list对象包装为map对象，其中key 名称默认为 list，根据key名称get出map对象的value即list列表。-->
    <!-- 2.对于输入list对象，则 if 中取值直接反射 list 对象中的方法获取对应的值即可。-->
    <!-- 3._parameter 默认参数是所有输入参数的对象，在本案例中，_parameter参数指的是包装后的以list为key的map对象。如果需要调用list对象的方法，则直接使用list调用方法即可，使用_parameter将找不到方法。 -->
    <!-- 4.分析：_parameter参数：map("list",list); list：取出_parameter中的指定key即list对应的value，即取出的是对应的list对象。-->
    <select id="findUserByIdIfList" parameterType="java.util.List"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from zeh_user a
        where 1 = 1
        <if test="_parameter != null">
            and a.name = 'list'
        </if>
        <if test="list.size() > 0">
            and a.name = 'size'
        </if>
        <if test="list.size() == 0">
            and a.name = 'zhaoeh'
        </if>
        <if test="list.get(0) != null">
            and a.name = 'get(1)'
        </if>
        <if test="_parameter.size() > 1">
            and a.name = 'listsize'
        </if>
        <!-- list中的每一个元素是String 类型，完全按照 String 类型进行取值。-->
        <!-- mybatis比较字符串相等时,如果参数不是数字，会报类型转换错误。-->
        <!-- 原因：ognl语法认为''括起来的字符是char类型，所以'A'被认为是char类型会尝试转换为整数，所以转换报错。-->
        <!-- (1)、改为'A'.toString()后进行 == 比较。-->
        <!-- (2)、将'A'进行转义：&quot;A&quot;。-->
        <!-- (3)、改为 test='list.get(2)== "C"'，将""放在字符串里面，将test表达式改用''。-->
        <if test="list.get(0) == 'A'.toString()">
            and a.name = 'A'
        </if>
        <if test="list.get(1)== 'B'.toString()">
            and a.name = 'B'
        </if>
        <if test='list.get(2)== "C"'>
            and a.name = 'C'
        </if>
    </select>

    <!-- 1.输入参数是 array，mybatis 将list对象包装为map对象，其中key 名称默认为 array，根据key名称get出map对象的value即array数组。-->
    <!-- 2.对于输入array对象，则 if 中取值直接反射 array 对象中的[]或者length属性获取对应的值即可。-->
    <!-- 3._parameter 默认参数是所有输入参数的对象，在本案例中，_parameter参数指的是包装后的以array为key的map对象。如果需要调用array对象的方法，则直接使用array调用方法即可，使用_parameter将找不到方法。 -->
    <!-- 4.分析：_parameter参数：map("array",array); list：取出_parameter中的指定key即array对应的value，即取出的是对应的array对象。-->
    <select id="findUserByIdIfArray" parameterType="java.util.List"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from zeh_user a
        where 1 = 1
        <if test="_parameter != null">
            and a.name = 'array'
        </if>
        <if test="array != null">
            and a.name = 'testArray'
        </if>
        <if test="array.length > 0">
            and a.name = 'arrayLength'
        </if>
        <if test="array.length == 0">
            and a.name = 'arrayLength2'
        </if>
        <if test="array[1] == 2">
            and a.name = 'arrayLength3'
        </if>
        <if test="_parameter.size() > 0">
            and a.name = 'parameterSize'
        </if>
    </select>

    <!-- 1.输入参数是 map，mybatis 不进行默认key的包装，而是直接根据key名称get出map对象中对应的参数值。所以只需要指定代码中设置的key名称即可。-->
    <!-- 2._parameter 默认参数是所有输入参数的对象，在本案例中，_parameter参数指的就是输入的map对象。 -->
    <select id="findUserByIdIfMap" parameterType="java.util.Map"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from zeh_user a
        where 1 = 1
        <if test="_parameter != null">
            and a.name = 'map'
        </if>
        <if test="_parameter.name != null">
            and a.name = 'myMapName'
        </if>
        <if test="name != null">
            and a.name = 'myMapName2'
        </if>
        <if test="_parameter.size() > 0">
            and a.name = 'myMapName3'
        </if>
        <if test="name != null and name != ''">
            and a.name = 'myMapName4'
        </if>
        <if test="_parameter.name != ''">
            and a.name = 'myMapName5'
        </if>
    </select>

    <!-- 1.输入参数是 pojo，mybatis 不进行默认key的包装，对于 if 中参数取值必须使用getXXX或者isXXX中的XXX去反射对应的方法进行取值。如果不存在getXXX或者isXXX将报错。-->
    <!-- 2._parameter 默认参数是所有输入参数的父对象，即输入参数向上转型成Object类型后的默认对象名。 -->
    <!-- 3._parameter 表示statementid的任何输入对象，可以通过_parameter根据ONGL表达式去获取属性值，也可以直接获取不用加_parameter。-->
    <!-- 4._parameter只能通过OGNL表达式获取setXXX、getXXX或者isXXX中的XXX（一般情况下都是属性值），其他的东西一律不能.出来。-->
    <!-- 5.实际上pojo的_parameter对象中只存储了传入的对象的所有setter()，getter()，isxxx()对应的xxx，通过反射对应的get()，is()去获取值。-->
    <!-- 6.pojo的if标签的test中可以指定任何boolean表达式，但是不能执行传入对象的目标方法，因为无法获取传入对象的方法（如4所描述）-->
    <!-- 7.pojo的if标签可以直接指定传入对象的一个boolean方法，注意在传入的pojo中该boolean方法必须符合规范，即isXXX()，或者getXXX()。-->
    <!-- 可以通过_parameter对象.这个方法后面的xxx，mybatis会自动根据xxx反射对应的方法，从而拿到boolean值（很多时候这个方法的xxx并不是属性）。-->
    <!-- 当然如果是第一层pojo，也可以不用加_parameter参数。-->
    <!-- 8.if标签也可以直接指定 true 或者 false-->
    <!-- 9.对于pojo中的list或者array对象，也是通过getXXX或者isXXX进行反射获取的，此时将不再包装成默认的list或者array的map对象。-->
    <!--对于输入对象中，isxxx()或者getxxx()，后面的xxx不允许重复，如果有isZhao()和getZhao(),则下面根据zhao去反射方法就会报错，不知道找哪一个。 -->
    <select id="findUserByIdIfPo" parameterType="zeh.mybatis.code00common.po.singleton.CommonPo"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from zeh_user a
        where 1 = 1
        <if test="_parameter.name != null">
            and a.name = #{name}
        </if>
        <if test="true">
            and a.name = 2
        </if>
        <if test="_parameter.zhao">
            and a.name = 'zhao'
        </if>
        <if test="yui">
            and a.name = 'yui'
        </if>
        <if test="_parameter.allList.size() > 0">
            and a.name = 'size'
        </if>
        <if test="_parameter.allList.get(0) != null">
            and a.name = 'get0'
        </if>
        <if test="true">
            and a.id in (select id from zeh_user b where 1=1 and b.id = #{id})
        </if>

    </select>

    <!-- 总结 mybatis 的 if 标签取值： -->
    <!-- 对于 list 和 array 会自动包装成以 list 或者 array 为key的map对象进行传输，通过默认的key遍历反射get()方法进行取值。list或者array可以直接反射对应的方法。 -->
    <!-- 对于 简单类型、map 和 pojo 则不进行包装，直接根据参数值、指定的key反射get()方法进行取值或者反射 getXXX() 或者 isXXX()方法获取对应的值。-->

</mapper>
```

# 3. 运行
zeh.mybatis.code02mybatis.run.MybatisIfDaoRun
```java
package zeh.mybatis.code02mybatis.run;

import org.junit.Before;
import org.junit.Test;
import zeh.mybatis.code02mybatis.dao.MybatisIfDao;

import java.io.IOException;

// 测试mybatis的if标签。
public class MybatisIfDaoRun {
    private MybatisIfDao ifDao;

    @Before
    public void newInstanceFirstDao() {
        ifDao = new MybatisIfDao();
    }

    @Test
    public void testFindUserByIdIf() throws IOException {
        ifDao.findUserByIdIf();
    }

    @Test
    public void testFindUserByIdIfList() throws IOException {
        ifDao.findUserByIdIfList();
    }

    @Test
    public void testFindUserByIdIfArray() throws IOException {
        ifDao.findUserByIdIfArray();
    }

    @Test
    public void testFindUserByIdIfMap() throws IOException {
        ifDao.findUserByIdIfMap();
    }

    @Test
    public void testFindUserByIdIfPo() throws IOException {
        ifDao.findUserByIdIfPo();
    }
}
```