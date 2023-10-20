---
layout:     post
title:      mybatis原始方式实现DAO
subtitle:   原始方式实现DAO意味着mybatis除了编写mapper接口，还应该实现mapper接口
categories: [mybatis]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. 原始方式实现DAO-基础操作
## 1.1 定义BaseDao接口
mybatis原始方式定义DAO接口。   
原始方式只是简单的引入了mybatis的简单步骤而已，并没有应用mybatis的高级应用。   
原始方式开发的接口使用泛型定义，增加dao接口的高可用性。   
zeh.mybatis.code03yuanshi.dao.IBaseDao
```java
package zeh.mybatis.code03yuanshi.dao;

import zeh.mybatis.code00common.po.singleton.CommonPo;

import java.util.List;
import java.util.Map;

// 2019-3-6下午2:56:50
public interface IBaseDao<T> {

    // 根据用户名模糊查询用户列表，使用 ${} 获取参数
    public List<T> findUserByNameForLikeBy$(String name) throws Exception;

    // 根据用户名模糊查询用户列表，使用 #{} 获取参数
    public List<CommonPo> testFindUserByNameForLikeByJing(String name) throws Exception;

    // 根据users表的name字段查询user信息，使用${}进行等值查询。
    public List<CommonPo> findUserByName3(String name) throws Exception;

    // 根据外部指定sql进行查询操作
    public List<T> findUser(String sql) throws Exception;

    // 根据用户名的in操作查询用户列表，对应${}，使用${}接收原生不动的字符串。
    public List<CommonPo> findUserByIn1(String name) throws Exception;

    public List<CommonPo> findUserByString2Date(String name) throws Exception;

    // 添加用户信息
    public void insertUser(T user) throws Exception;

    // 删除用户信息
    public void deleteUser(int id) throws Exception;

    // 更新用户信息
    public void updateUser(T user) throws Exception;

    // 执行任意sql
    public List<T> query(String sql) throws Exception;

    public List<CommonPo> findUserByName4(String name) throws Exception;

    public List<CommonPo> findUserByName5(String name) throws Exception;

    public List<CommonPo> findUserByName6(String name) throws Exception;

    public List<CommonPo> findUserByName7(Map<String, String> params) throws Exception;

    public List<CommonPo> findUserByName8(Map<String, String> params) throws Exception;

}

```
## 1.2 自己实现BaseDao接口
原始方式必须自己实现dao接口的实现类.   
1、sqlSessionFactory建议使用单例模式进行管理，Spring和mybatis整合后sqlSessionFactory就是按照单例模式进行管理的，避免sqlSessionFactory被频繁创建，影响系统性能。   
2、原始方式实现dao模式的弊端：   
dao接口中定义的方法中依旧存在大量的模板方法步骤。   
调用sqlSession中的方法时将输入参数硬编码，并且调用sqlSession的方法时使用object接收参数，就是参数传递错误也不报错。   
但是相对于原生态的dao模式的mybatis编程，将sqlSessionFactory对象进行单例管理了，并且减少了部分重复性的编码。   
其实mybatis和spring整合之后这些问题都不用考虑，spring负责创建mybatis的sqlSession对象，并且默认是单例， 但是底层通过ThreadLocal来保证session中Connection的线程安全。   
3、mybatis直接操作sql的接口全部封装在sqlSession对象中，全部的接口定义如下：   
从mybatis底层定义的接口来看，实际上采用原始方式进行mapper开发是不能传递多个输入参数的，要么传递简单类型，要么封装到pojo、list或者map对象中去。   
原始方式不论是传递简单类型还是对象类型，都只能传递一个；既可以指定 parameterType 也可以不指定（不指定时mybatis将自动根据传入的参数类型匹配）。   
4、mybatis原始接口的所有方法如下：   
```java
public abstract Object selectOne(String s);
public abstract Object selectOne(String s, Object obj);
public abstract List selectList(String s);
public abstract List selectList(String s, Object obj);
public abstract List selectList(String s, Object obj, RowBounds rowbounds);
public abstract Map selectMap(String s, String s1);//s1是查询返回的map对象中指定的key名称
public abstract Map selectMap(String s, Object obj, Strings1);//s1是查询返回的map对象中指定的key名称
public abstract Map selectMap(String s, Object obj, String s1, RowBoundsrowbounds);//s1是查询返回的map对象中指定的key名称
public abstract void select(String s, Object obj, ResultHandlerresulthandler);
public abstract void select(String s, ResultHandler resulthandler);
public abstract void select(String s, Object obj, RowBounds rowbounds,ResultHandler resulthandler);
public abstract int insert(String s);
public abstract int insert(String s, Object obj);
public abstract int update(String s);
public abstract int update(String s, Object obj);
public abstract int delete(String s);
public abstract int delete(String s, Object obj);
public abstract void commit();
public abstract void commit(boolean flag);
public abstract void rollback();
public abstract void rollback(boolean flag);
public abstract List flushStatements();
public abstract void close();
public abstract void clearCache();
public abstract Configuration getConfiguration();
public abstract Object getMapper(Class class1);
public abstract Connection getConnection();
```

下面看案例：   
zeh.mybatis.code03yuanshi.dao.impl.BaseDaoImpl
```java
package zeh.mybatis.code03yuanshi.dao.impl;

import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import zeh.mybatis.code00common.po.singleton.CommonPo;
import zeh.mybatis.code03yuanshi.dao.IBaseDao;

import java.util.List;
import java.util.Map;

public class BaseDaoImpl implements IBaseDao<CommonPo> {

    // 类和类之间就是个引用链的关系。
    // 这种引用链对于当前类来讲被描述成引用，即A中引用了B；对于A的调用方来讲被描述成注入或者抽取（注入就是调用set方法设置属性值，抽取就是调用get方法获取属性值）
    // 需要向dao实现类中注入SqlSessionFactory，SqlSessionFactory被设计成成员变量的形式，通过构造方法注入实例化对象，这样保证了其单例模式的特性。
    // 但是sqlSession对象内部有成员变量，所以在并发环境下sqlSession是非线程安全的，建议sqlSession不要使用成员变量的保存形式（ 即不要使用单例模式），而是采用方法变量的形式，保证线程安全性（方法变量是线程私有的虚拟机栈内存）。
    // spring整合后不用考虑sqlSession的安全性问题，因为spring在实例化sqlSession对象的时候对里面的成员加入了ThreadLocal实现了栈封闭。
    private SqlSessionFactory sqlSessionFactory;

    // 这里通过构造方法注入sqlSessionFactory单例。
    public BaseDaoImpl(SqlSessionFactory sqlSessionFactory) {
        this.sqlSessionFactory = sqlSessionFactory;
    }

    // 根据用户名称模糊查询users表中的record，使用${}进行like匹配查询。
    @Override
    public List<CommonPo> findUserByNameForLikeBy$(String name) throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("yuanshi.findUserByNameYuanShiForLikeBy$", name);
        sqlSession.close();
        return list;
    }

    // 根据用户名称模糊查询users表中的record，使用#{}进行like匹配查询。
    @Override
    public List<CommonPo> testFindUserByNameForLikeByJing(String name) throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("yuanshi.testFindUserByNameForLikeByJing", name);
        sqlSession.close();
        return list;
    }

    // 根据用户名称模糊查询users表中的record，使用${}进行等值查询。
    // 使用${}进行等值查询，传递进去的参数如果是String将不会自动加上''，所以需要手动加上''；这样会导致，一旦oracle服务器表中该字段要求的是String类型，而${}方式传递进去的就是一个数字型，oracle 服 务 端 会进行了隐式函数转换，则建立在该字段上的索引将失效，导致全表扫 描 。
    // 解决办法：
    //（1）、在应用代码层面，使用#{}传递参数，则String会自动加上''，不会导致oracle服务端隐式转换；
    //（2）、在应用代码层面，使用${}传递参数，但是手动加上''，不会导致oracle服务端隐式转换；
    //（3）、在oracle层面，对该字段建立的索引建立函数索引，则即便对该字段使用了转换函数，也不会导致索引失效，就不会全表扫描了。
    @Override
    public List<CommonPo> findUserByName3(String name) throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("yuanshi.findUserByName3YuanShi", name);
        sqlSession.close();
        return list;
    }




    // 根据用户名称查询users表中的record，使用#{}进行等值查询
    @Override
    public List<CommonPo> findUserByName4(String name) throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("yuanshi.findUserByName4YuanShi", name);
        sqlSession.close();
        return list;
    }

    // 根据用户名称查询users表中的record，使用${}进行等值查询，验证sql注入；
    // ${}不会预编译和转义，传入原生参数，引起sql注入。
    @Override
    public List<CommonPo> findUserByName5(String name) throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("yuanshi.findUserByName5YuanShi", name);
        sqlSession.close();
        return list;
    }

    // 根据用户名称查询users表中的record，使用#{}进行等值查询，验证sql注入；
    // #{}会预编译和sql转义，不会引起sql注入。
    @Override
    public List<CommonPo> findUserByName7(Map<String, String> params)
            throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("yuanshi.findUserByName7YuanShi", params);
        sqlSession.close();
        return list;
    }

    // mapper配置了statementtype属性，测试不同的属性控制的范围。
    @Override
    public List<CommonPo> findUserByName8(Map<String, String> params)
            throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("yuanshi.findUserByName8YuanShi", params);
        sqlSession.close();
        return list;
    }

    // 根据用户名称查询users表中的record，使用#{}进行等值查询，验证sql注入；
    // #{}会预编译和sql转义，不会引起sql注入。
    @Override
    public List<CommonPo> findUserByName6(String name) throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("yuanshi.findUserByName6YuanShi", name);
        sqlSession.close();
        return list;
    }

    // 根据in关键在执行查询操作，主要观察${}和#{}的区别
    // 本次使用${}，接收的是一个字符串。
    @Override
    public List<CommonPo> findUserByIn1(String name) throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();

        List<CommonPo> list = sqlSession.selectList("yuanshi.findUserByIn1YuanShi", name);

        sqlSession.close();

        return list;
    }


    @Override
    public List<CommonPo> findUserByString2Date(String name) throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("yuanshi.findUserByString2DateYuanShi", name);
        sqlSession.close();
        return list;
    }

    // 向users表中增加新的记录
    @Override
    public void insertUser(CommonPo user) throws Exception {
        // 重新构造sqlSession对象
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        // 通过sqlSession对象操作Executor执行器，向数据库发送sql；注入输入参数user
        sqlSession.insert("yuanshi.insertUserYuanShi", user);
        // 提交事务
        sqlSession.commit();
        // 关闭sqlSession
        sqlSession.close();
    }

    // 根据id删除record，或者id is null的record
    @Override
    public void deleteUser(int id) throws Exception {
        // 重新实例化sqlSession对象
        SqlSession sqlSession = this.sqlSessionFactory.openSession();

        // sqlSession对象操作底层Executor执行器发送sql
        sqlSession.delete("yuanshi.deleteUserYuanShi", id);

        // delete操作需要提交事务
        sqlSession.commit();

        // 关闭sqlSession会话
        sqlSession.close();
    }

    @Override
    public List<CommonPo> findUser(String sql) throws Exception {
        // 重新实例化方法内部的sqlSession对象
        SqlSession sqlSession = this.sqlSessionFactory.openSession();

        List<CommonPo> list = sqlSession.selectList("yuanshi.findUserYuanShi", sql);

        // 关闭sqlSession，select操作不用提交事务
        sqlSession.close();

        return list;
    }

    // 全量sql
    public List<CommonPo> query(String sql) throws Exception {
        // 重新实例化方法内部的sqlSession对象
        SqlSession sqlSession = this.sqlSessionFactory.openSession();

        List<CommonPo> list = sqlSession.selectList("yuanshi.selectTestYuanShi", sql);

        // 关闭sqlSession
        sqlSession.close();

        return list;
    }

    @Override
    public void updateUser(CommonPo user) throws Exception {
        // 重新实例化sqlSession对象
        SqlSession sqlSession = this.sqlSessionFactory.openSession();

        // 传入user对象，指定更新操作
        sqlSession.update("yuanshi.updateUserYuanShi", user);
        // delete操作需要提交事务
        sqlSession.commit();
        // 关闭sqlSession会话
        sqlSession.close();
    }

}
```
## 1.3 resource下定义mapper.xml
mybatis/mapper/mybatis-yuanshi-base-mapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- mybatis原始方式开发DAO -->
<!-- namespace命名空间，作用就是对sql进行分类化管理，理解sql隔离 -->
<!-- 下面方式不使用mybatis的代理方法开发，所以namespace可以随便起 -->
<!-- 注意：如果使用mapper代理方法开发，namespace有特殊重要的作用 -->
<mapper namespace="yuanshi">

    <!-- 1.根据用户名称模糊查询用户信息，可能返回多条。 -->
    <!-- 2.resultType：指定就是单条记录所映射的java对象类型，可以是任何java类型，包括简单类型和pojo（如果是list的话只需要指定其中元素的类型即可）。-->
    <!-- 3.${}:表示拼接sql串，将接收到参数的内容不加任何修饰拼接在sql中。不加任何修饰，意味着传进来是什么就是什么，对于字符串连单引号都不会加。 -->
    <!-- 4.使用${}拼接sql，引起 sql注入。因为${}的方式传入的参数不会被预编译而是和sql一起发送给数据库，由数据库统一进行编译。 -->
    <!-- 5.${value}：接收输入参数的内容，如果传入类型是简单类型，${}中只能使用value。 -->
    <!-- 6.像本例中这种直接在sql中编写字符串的，要么使用${}这种形式拼接字符串；要么就只能使用#{}进行整体占位。 -->
    <!-- 7.本例先使用${}进行原封不动的拼接（注意${}不会为字符串加上''）。 -->
    <select id="findUserByNameYuanShiForLikeBy$" parameterType="java.lang.String"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from zeh_user where name like '%${value}%'
        <!-- 下面方式直接使用${}取值是不对的，因为${}取出来的值是不加任何修饰的，对于字符串连引号也不会加，所以如果采用${}取值的话需要手动加上单引号 -->
        <!-- select * from zeh_user where name = ${value} -->

        <!-- 使用${}必须加上单引号 -->
        <!-- select * from zeh_user where name = '${value}' -->
    </select>

    <!-- 1.要想使用#{}占位符号的话，可以使用如下形式，且在代码中传递的时候必须手动加上%% -->
    <select id="testFindUserByNameForLikeByJing" parameterType="java.lang.String"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
		SELECT * FROM ZEH_USER WHERE name LIKE #{value}
	</select>

    <!-- 1.下面方式直接使用${}取值是不对的，因为${}取出来的值是不加任何修饰的，对于字符串连引号也不会加，所以如果采用${}取值的话需要手动加上单引号。 -->
    <!-- 2.使用${}必须加上单引号 ，如果不手动加''号且oracle服务端不做类型转换处理的话，mybatis将报错，提示标识符无效。 -->
    <!-- 3.一旦oracle服务器表中该字段要求的是String类型，而${}方式传递进去的就是一个数字型，则oracle默认会隐式转换。 -->
    <!-- 4.若oracle服务端通过类型转换函数对name字段进行了隐式转换，则oracle中该表的该字段建立的索引将失效，导致全表扫描。 -->
    <!-- 5.解决办法，在应用层使用#{}传递参数，将自动加上''；或者使用如下的，对${}手动加上''; -->
    <!-- 或者在数据库层面将该字段的索引建立成函数索引后，索引将不会失效。 -->
    <select id="findUserByName3YuanShi" parameterType="java.lang.String"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from zeh_user where name = '${value}'
    </select>

    <!-- 1.使用#{}不用手动加''，mybatis会根据类型判断是否需要自动加上''。 -->
    <select id="findUserByName4YuanShi" parameterType="string"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
		select * from zeh_user where name = #{value}
	</select>

    <!-- 1.使用${}进行等值查询，改变传入的name参数手动拼接字符串，使得数据库将参数当做sql命令执行，验证sql注入。 -->
    <!-- 2.结论：${}传入原生参数，可能导致sql注入！ -->
    <select id="findUserByName5YuanShi" parameterType="string"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
		select * from zeh_user where name = ${value}
	</select>


    <!-- 1.使用#{}进行等值查询，#{}会预编译和转义，不会将参数当做sql命令执行。 -->
    <!-- 2.结论：#{}不会引起sql注入！ -->
    <select id="findUserByName6YuanShi" parameterType="java.lang.String"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
		select * from zeh_user where name = #{value}
	</select>


    <!-- 1.观察同时使用#{}和${}，看这个statementid使用的是预编译还是非预编译？ -->
    <!-- 2.主要明确：预编译到底是对传入的参数进行的预编译还是对sql语句进行的预编译呢？搞清楚。 -->
    <!-- 3.结论：只要一个statementid中使用了哪怕一个${}，那么整个sql语句就都不会预编译，而是采用拼接参数的形式。 -->
    <!-- 4.预编译是对sql语句进行预编译，但是如果一个statementid中只要使用了${}，则整个sql语句都不会预编译。 -->
    <!-- 5.那么对于使用了#{}的占位符这种情况下是什么时候编译的？ -->
    <!-- 解答：#{}会将传入参数预编译成一个占位符号，#{}接收输入参数，类型可以是简单类型，pojo、hashmap、list等；-->
    <!-- #{}会对sql语句和输入参数进行预编译，将传入的参数预编译成一个占位符?，然后通过设置参数的形式设置完参数后将sql发送给数据。 -->
    <select id="findUserByName7YuanShi" parameterType="hashmap"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
		select * from zeh_user where name = ${key1} and hobby = #{key2}
	</select>


    <!-- 1.配置statementType为STATEMENT，将取消默认的PREPARED预编译模式，sql中获取参数只能使用${}获取，一个#{}都不能有。 -->
    <!-- 2._parameter是mybatis默认用来表示输入参数的对象： -->
    <!-- 比如输入参数是map，则表示当前输入的map对象、是String则表示是当前的string对象、是list则表示是当前输入的list对象等等 -->
    <select id="findUserByName8YuanShi" parameterType="hashmap"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo"
            statementType="STATEMENT">
        <if test="_parameter != null">
            select * from zeh_user
            where name = ${key1} and hobby =
            ${key2}
        </if>
    </select>


    <!-- 1.根据in查询，主要观察#{}和${}在in中对于引号的区别 ，本次对应的是${}，对于in操作而言，使用${}明显更简单。此statementid接收的是字符串 -->
    <select id="findUserByIn1YuanShi" parameterType="java.lang.String"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
		select * from zeh_user where name in (${value})
	</select>


    <!-- 1.根据in查询，主要观察#{}和${}在in中对于引号的区别 ，本次对应的是#{}，对于in操作而言，使用#{}复杂，因为需要对每个元素自动拼接''； -->
    <!-- 2.此statementid接收的是list，所以collection必须是list。 -->
    <!-- 3.mybatis巧妙的利用foreach标签遍历list列表或者数组，然后动态的拼接出指定格式的sql片段。 -->
    <!-- 4.注意，foreach标签很灵活，其中的collection属性就是接收一个列表对象或者数组的，当然，collection属性也支持表达式。 -->
    <!-- 5.如果传递进来的就是一个大字符串，那么collection可以写成：collection="mystr.split(',')"。 -->
    <!-- 即：mybatis的标签中也提供了很多内置函数，其中split(',')就是mybatis提供的字符串分割函数，其参数是分割符号，返回值是是一个列表容器。 -->
    <select id="findUserByIn2YuanShi" parameterType="list"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        <!-- select * from zeh_user where name in (#{value}) -->
        select * from zeh_user
        <if test="_parameter != null">
            where name in
            <foreach collection="list" item="item" index="index" open="("
                     close=")" separator=",">
                #{item}
            </foreach>
        </if>
    </select>


    <!-- 1.根据in查询，主要观察#{}和${}在in中对于引号的区别 ，本次对应的是#{}，对于in操作而言，使用#{}复杂，因为需要对每个元素自动拼接''； -->
    <!-- 2.此statementid接收的是一个map，且该map中key为myParamMap，对应的value是一个list，所以此处collection为map中的key即myParamMap； -->
    <!-- 3.但是实际此处的collection属性接收的是一个list，不过通过map方式传递进来的则不再使用默认的list命名，而是用map中对应的key。 -->
    <!-- 4.同时下面不指定parameterType，则默认传递进来的参数是map类型，此处的parameterType默认就是map类型。 -->
    <select id="findUserByIn3YuanShi" resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from zeh_user where
        <include refid="test123456"/>
    </select>
    <sql id="test123456">
        (1=1
        <!-- 增加判断，当传入的map对象中的key不为空时（这个key是个list），才执行in操作，否则in操作将报错 -->
        <if test="myParamMap!=null and !myParamMap.isEmpty()">
            and name in
            <foreach collection="myParamMap" item="item" index="index"
                     open="(" close=")" separator=",">
                #{item}
            </foreach>
        </if>
        )
    </sql>


    <!-- 1.根据in查询，主要观察#{}和${}在in中对于引号的区别 ，本次使用的#{}，对于in操作而言，使用#{}比较复杂了。 -->
    <!-- 2.此statementid接收的依旧是字符串，主要是通过foreach进行分割，将大的字符串根据分割符分割为数组。 -->
    <!-- 3.分割时，collection属性指定的是mybatis的表达式，借助了mybatis标签内置的分割函数split()，该函数返回一个list容器。 -->
    <!-- 4.注意#{}会自动为字符串类型加上''。 -->
    <select id="findUserByIn4YuanShi" resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
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
    <!-- 3.着重强调index属性：index表示foreach标签遍历的循环次数，从0开始；切记，取值一定是取的item， -->
    <!-- 4.而不是index，因为index取出来实际上是从0开始的下标 -->
    <!-- 5.如果是单个传递数组对象，则parameterType是数组中元素的类型，比如是字符串数组，则parameterType为String -->
    <select id="findUserByIn5YuanShi" resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from
        <foreach collection="array" item="item" open="(" close=") t"
                 separator=" union " index="myindex">
            select * from zeh_user where name
            =
            #{item}
            <!-- 注意观察日志，看index输出是什么 -->
            or id = #{myindex}
        </foreach>
    </select>


    <!-- 1.传递一个map，map中全部是简单类型：验证后，发现foreach要想遍历map本身比较复杂。 -->
    <!-- 2.如果使用_parameter.keys只能取出map中所有的key，此时index和之前一样表示遍历的次数； -->
    <!-- 3.如果使用_parameter.values只能取出map中所有的value，此时index和之前一样表示遍历的次数； -->
    <!-- 4.如果使用_parameter.entrySet()则能取出所有的key和value，此时item表示的就是value，index表示的是key，一劳永逸！！！ -->
    <select id="findUserByIn6YuanShi" resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from
        <!-- <foreach collection="_parameter.keys" item="value" open="(" close=")
            t" separator=" union " index="index"> select * from zeh_user where name =
            #{value} and id = #{index} </foreach> -->

        <!-- <foreach collection="_parameter.values" item="value" open="(" close=")
            t" separator=" union " index="index"> select * from zeh_user where name =
            #{value} and id = #{index} </foreach> -->
        <foreach collection="_parameter.entrySet()" item="value" open="("
                 close=") t" separator=" union " index="key">select * from zeh_user where
            name = #{key} and id = #{value}
        </foreach>
    </select>


    <!-- 1.验证foreach标签强大的逻辑处理能力： -->
    <!-- 2.传递的字符串是："ssg:1$$$$ddf:2$$$$jjk:3$$$$iio:4$$$$zhaoeh:5$$$$yurui:6"; -->
    <!-- and name="" -->
    <!-- and id="" -->
    <select id="findUserByIn7YuanShi" resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
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
    <!--传入一个List<Map>： list中每一个map存放：corpId,dataAuthFlag,dataType -->
    <!-- 结合foreach进行遍历 -->
    <!-- 这个配置好好理解：主要是foreach如何遍历list中的map，浪费我几个小时，现在半夜了都 -->
    <!-- 要获取入参list<Map>中的map对应的value值，则可以使用item.key来遍历获取每一个list中的元素（此时是item）的对应的key的value，即item.key -->
    <select id="findUserByIn8YuanShi" resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
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


    <!-- 对于string转成date，使用${}和#{}是一样的 -->
    <select id="findUserByString2DateYuanShi" parameterType="java.lang.String"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        <!-- select * from zeh_user where 1=1 and to_date(BUSSDATE,'yyyyMMdd')
            >= to_date(#{value},'yyyyMMdd') -->
        select * from zeh_user where 1=1 and to_date(BUSSDATE,'yyyyMMdd') >=
        to_date(${value},'yyyyMMdd')
    </select>


    <!-- 添加用户 -->
    <!-- parameterType：指定输入 参数类型是pojo（包括 用户信息）。 -->
    <!-- #{}中指定pojo的属性名，接收到pojo对象的属性值。 -->
    <!-- 不论是#{}还是${}，mybatis都是通过OGNL表达式获取对象的属性值，即属性名.属性名.属性名...的方式。 -->
    <insert id="insertUserYuanShi" parameterType="zeh.mybatis.code00common.po.singleton.CommonPo">
        <!-- 将插入数据后的主键返回，返回到user对象中： -->
        <!-- SELECT LAST_INSERT_ID()函数：是mysql的内置函数，返回auto_increment自增列新记录id值， -->
        <!-- 得到刚insert进去记录的主键值，只适用于自增主键 。 -->
        <!-- selectKey标签作用：获取一个值然后设置到parameterType指定的某个属性上。 -->
        <!-- 注意：selectKey只是获取一个key值然后设置到指定的属性中去，这个key一般是通过各个数据库返回的主键， -->
        <!-- 也可以自己指定statement语句实现一个子sql来返回一个值设置到属性中去。反正就是为了获取一个值设置到parameterType的属性中。 -->
        <!-- selectKey标签一般都是包含在insert,update,delete标签中的。 -->
        <!-- 顺序必须在insert操作之后，才能得到刚才insert进去的自增主键的值，即order="AFTER" -->

        <!-- keyProperty：将查询到主键值设置到parameterType指定的对象的哪个属性 -->
        <!-- order：SELECT LAST_INSERT_ID()执行顺序，相对于insert语句来说它的执行顺序，即在insert操作之后执行该函数 -->
        <!-- 由于mysql的自增原理执行完insert语句之后才将主键生成，所以这里 selectKey的执行顺序为after。 -->
        <!-- resultType：指定SELECT LAST_INSERT_ID()的结果类型，即返回的主键是什么类型 -->
        <!-- 本例中的users表没有设置主键，所以下属方案暂时注释掉 -->
        <!-- <selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
            SELECT LAST_INSERT_ID() </selectKey> -->

        <!-- mybatis对null值转换报错的处理： -->
        <!-- 如果mybatis在执行insert操作时候传入的参数有null值的话（注意只有null值，空字符可以正常插入，不会报类型转换错误），那么下面 -->
        <!-- 使用OGNL表达式读取参数的时候，mybatis会报错，即类型转换错误：Invalid column type:1111。 -->
        <!-- 原因是，对于值为null的字段，如果不指定jdbcType的话，mybatis自动将null类型转换成Other，Other对于oracle是不识别的类型，所以对于null -->
        <!-- 必须显式指定jdbcType，对于其他的类型mybatis自动做了转换！否则mybatis将报无效的列类型。 -->

        <!-- 还有一种解决方式，直接在sqlmap-config.xml全局配置文件中配置settings参数，jdbcTypeForNull=VARCHAR -->
        <!-- mybatis的OGNL表达式的底层原理（反射）： -->
        <!-- 使用#{}占位，采取OGNL表达式获取（get）到po对象中的指定成员值；OGNL表达式底层采用反射方式 -->
        <!-- insert into users(id,name,age,sex,hobby) values(#{id},#{name},#{age},#{sex,jdbcType=VARCHAR},#{hobby}) -->
        insert into zeh_user(id,name,age,sex,hobby)
        values(#{id},#{name},#{age},#{sex},#{hobby})

        <!-- 先通过uuid()查询到主键，将主键 输入 到sql语句中。 -->
        <!-- 执行uuid()语句顺序相对于insert语句之前执行。 -->
        <!-- 使用mysql的uuid（）生成主键 -->
        <!-- 执行过程： -->
        <!-- 首先通过uuid()得到主键，将主键设置到user对象的id属性中 -->
        <!-- 其次在insert执行时，从user对象中取出id属性值 -->
        <!-- <selectKey keyProperty="id" order="BEFORE" resultType="java.lang.String">
            SELECT uuid() </selectKey> insert into zeh_user(id,username,birthday,sex,address)
            value(#{id},#{username},#{birthday},#{sex},#{address}) -->
    </insert>


    <!-- 删除 用户 -->
    <!-- 根据id删除用户，需要输入 id值，或者id is null的record -->
    <delete id="deleteUserYuanShi" parameterType="java.lang.Integer">
		delete from zeh_user where
		id=#{id} or id is null
	</delete>


    <!-- 根据指定sql查询数据 -->
    <select id="findUserYuanShi" parameterType="string"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
		${value}
	</select>

    <select id="selectTestYuanShi" parameterType="string" resultType="map"
            statementType="STATEMENT">
		${value}
	</select>


    <!-- 根据id更新用户 -->
    <!-- 分析： -->
    <!-- 需要传入用户的id -->
    <!-- 需要传入用户的更新信息 -->
    <!-- parameterType指定user对象，包括 id和更新信息，注意：id必须存在 -->
    <!-- #{id}：从输入 user对象中获取id属性值 -->
    <!-- 任何一个sql语句，使用mybatis都必须分析出其中的输入参数和输出参数到底是什么， -->
    <!-- 应该怎么配置，是简单java类型，还是pojo自定义对象还是hashMap等集合 -->
    <update id="updateUserYuanShi" parameterType="zeh.mybatis.code00common.po.singleton.CommonPo">
        <if test="_parameter != null">
            update zeh_user
            <!-- set 标签和set关键字一样，如果需要对set的字段进行if等标签处理并且去掉最后一个,，则用set标签代替set关键字；否则直接使用set关键字即可。 -->
            <set>
                <if test="name != null">
                    <!-- 获取参数和直接指定值可以同时存在，直接指定值肯定就不获取参数了。 -->
                    name=#{name},age=#{age},sex=#{sex},hobby='吃饭喝酒'
                </if>
            </set>
            where id=#{id}
        </if>
    </update>

    <select id="findUserByForeachYuanShi" resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        <foreach collection="myParamMap" item="map" index="index"
                 open="(" close=")" separator=" UNION ALL ">
            select * from zeh_user t where 1=1
            <foreach collection="map.entrySet()" item="value" index="key"
                     open="" close="" separator="">
                <if test="key == 'name'">
                    and
                    t.name =
                    #{value}
                </if>
                <if test="key == 'sex'">
                    and t.sex = #{value}
                </if>
            </foreach>
        </foreach>
    </select>


    <!-- 全量sql，使用$直接将输入参数原封不动的进行拼接 -->
    <!-- 添加 statementType="STATEMENT"，解决替换表名问题。缺点：此种方式无法预编译。以下类似语句与此处含义相同。同时使用$符号代替# -->
    <insert id="insertTestYuanShi" parameterType="string" statementType="STATEMENT">
		${value}
	</insert>
    <delete id="deleteTestYuanShi" parameterType="string" statementType="STATEMENT">
		${value}
	</delete>
    <update id="updateTestYuanShi" parameterType="string" statementType="STATEMENT">
		${value}
	</update>

</mapper>
```
## 1.4 运行
zeh.mybatis.code03yuanshi.run.BaseDaoRun
```java
package zeh.mybatis.code03yuanshi.run;


import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
import zeh.mybatis.code00common.po.singleton.CommonPo;
import zeh.mybatis.code03yuanshi.dao.IBaseDao;
import zeh.mybatis.code03yuanshi.dao.impl.BaseDaoImpl;

import java.io.File;
import java.io.InputStream;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

// mybatis 原始方式实现 dao 模式的基本操作。
// 原始方式就是编写dao接口且要实现dao接口，实现的Dao需要我们使用sqlSession去操作CRUD操作。
public class BaseDaoRun {
    // 将sqlSessionFactory定义为成员变量，目的就是单例化。
    // 使用单例模式管理sqlSessionFactory实例，在代码中方便操作。
    // sqlSessionFactory是无状态的bean，作为成员变量不存在多线程安全隐患。
    private SqlSessionFactory sqlSessionFactory;

    // 解析sqlMapConfig.xml，创建sqlSessionFactory。
    // 此方法是在所有@Test标注的方法之前自动执行。
    @Before
    public void setUp() throws Exception {
        // 定义mybatis配置文件相对classpath路径
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        // 定义配置文件输入流
        InputStream inputStream = Resources.getResourceAsStream(resource);

        // 通过SqlSessionFactoryBuilder创建会话工厂实例，实例化SqlSessionFactory对象。
        // 将SqlSessionFactoryBuilder当成一个工具类使用即可，不需要使用单例管理SqlSessionFactoryBuilder，但是使用单例管理SqlSessionFactory对象。
        // 在需要创建SqlSessionFactory时候，只需要new一次SqlSessionFactoryBuilder即可。
        this.sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    }

    // 根据users表的name字段查询user信息，使用${}进行like匹配查询。
    @Test
    public void testFindUserByNameForLikeBy$() throws Exception {
        // 创建UserDao的对象，注入sqlSessionFactory对象。
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        // 调用UserDao的方法，注入输入参数。
        // mybatis 返回list的话，如果list为空，会默认实例化一个空的list返回，不会返回null值。
        // 如果返回结果是一个pojo，没查询到数据则返回一个null值。
        List<CommonPo> list = userDao.findUserByNameForLikeBy$("虎");
        System.out.println(list == null);
        if (!list.isEmpty()) {
            // 采用标准iterator输出集合输出List
            Iterator<CommonPo> iterator = list.iterator();
            System.out.println("根据用户名称模糊查询到的users表的record如下：");
            while (iterator.hasNext()) {
                System.out.println(iterator.next());
            }
        } else {
            System.out.println("没有符合条件的数据...");
        }
    }

    // 根据users表的name字段查询user信息，使用#{}进行like匹配查询。
    @Test
    public void testFindUserByNameForLikeByJing() throws Exception {
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        List<CommonPo> list = userDao.testFindUserByNameForLikeByJing("%虎%");
        if (list.isEmpty()) {
            System.out.println("没有查询到数据");
        } else {
            Iterator<CommonPo> iterator = list.iterator();
            System.out.println("根据用户名称模糊查询到的users表的record如下：");
            while (iterator.hasNext()) {
                System.out.println(iterator.next());
            }
        }
    }

    // 根据users表的name字段查询user信息，使用${}进行等值查询。
    @Test
    public void testFindUserByName3() throws Exception {
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        List<CommonPo> list = userDao.findUserByName3("赵二虎");
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据用户名称查询到的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    // 根据users表的name字段查询user信息，使用#{}进行等值查询
    @Test
    public void testFindUserByName4() throws Exception {
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        List<CommonPo> list = userDao.findUserByName4("赵二虎");
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据用户名称查询到的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    // 根据users表的name字段查询user信息，使用${}进行等值查询，验证${}不会预编译和转义，会原生发给数据库，然后导致sql注入。
    @Test
    public void testFindUserByName5() throws Exception {
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        List<CommonPo> list = userDao.findUserByName5("'赵二虎' or '1' = '1'");
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据用户名称查询到的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    // 根据users表的name字段查询user信息，使用#{}进行等值查询，验证#{}会预编译并且会自动进行sql中的转义
    // 将输入参数当做一个纯的字符串参数而不是转换成sql命令执行，所以#{}就不会查出record，即不会导致sql注入。
    @Test
    public void testFindUserByName6() throws Exception {
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        // 调用UserDao的方法，注入输入参数，完整的sql输出后如下，会自动为单引号进行转义
        // select * from users where name = '''赵二虎'' or ''1'' = ''1''';
        List<CommonPo> list = userDao.findUserByName6("'赵二虎' or '1' = '1'");

        if (list != null && list.size() > 0) {
            Iterator<CommonPo> iterator = list.iterator();
            System.out.println("根据用户名称查询到的users表的record如下：");
            while (iterator.hasNext()) {
                System.out.println(iterator.next());
            }
        } else {
            System.out.println("没有符合条件的数据!");
        }

    }

    // 根据users表的name字段查询user信息。
    // 同一个statementid同时使用${}和#{}，观察mybatis到底对这个statementid使用的是预编译模式还非预编译模式呢？
    // 结论：只要一个statementid中使用了哪怕一个${}，那么整个sql语句就都不会预编译，而是采用拼接参数的形式。
    @Test
    public void testFindUserByName7() throws Exception {
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        // 封装map
        Map<String, String> params = new HashMap<String, String>();
        params.put("key1", "'赵二虎' or '1' = '1'");// key1使用${}传递
        params.put("key2", "发呆");// key2使用#{}传递
        // 调用UserDao的方法，注入输入参数，完整的sql输出后如下，会自动为单引号进行转义
        // select * from users where name = '''赵二虎'' or ''1'' = ''1''';
        List<CommonPo> list = userDao.findUserByName7(params);
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据用户名称模糊查询到的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    // 测试mapper配置了 statementType 属性后，每种属性的控制范围。
    // 结论：
    // statementType 三种模式范围：
    // PREPARED(预编译模式)：控制范围最大，默认的预编译模式，既可以使用#{}也可以使用${}；
    // STATEMENT（非预编译模式）：范围次之，只能显式指定，指定后默认的PREPARED模式将失效，只能通过${}取值，不能使用#{}；
    @Test
    public void testFindUserByName8() throws Exception {
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        // 封装map
        Map<String, String> params = new HashMap<String, String>();
        params.put("key1", "'赵二虎' or '1' = '1'");// key1使用${}传递
        params.put("key2", "'发呆'");// key2使用#{}传递
        // 调用UserDao的方法，注入输入参数，完整的sql输出后如下，会自动为单引号进行转义
        // select * from users where name = '''赵二虎'' or ''1'' = ''1''';
        List<CommonPo> list = userDao.findUserByName8(null);
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据用户名称模糊查询到的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    // 根据name在某个集合中，进行查询，主要验证#{}和${}的区别。
    // 本次使用${}，对于in操作而言，使用${}很简单，因为${}不会自动为字符串加上''，所以传递进去的参数直接就是一个字符串；但是会导致sql注入！
    @Test
    public void testFindUserByIn1() throws Exception {
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        List<CommonPo> list = userDao.findUserByIn1("'ssg','ddf','jjk','iio','zhaoeh','yurui'");
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据用户名称使用${}查询到的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    // 使用${}获取字符串类型的日期，然后转换进行日期比较。
    @Test
    public void testFindUserByString2Date() throws Exception {
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        List<CommonPo> list = userDao.findUserByString2Date("20180708");
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据bussdate日期查询到的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    // 向users表中插入数据
    @Test
    public void testInsertUser() throws Exception {
        // 创建UserDao的对象，注入sqlSessionFactory对象
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        // 封装需要insert的user对象
        CommonPo user = new CommonPo();
        user.setId(04);
        user.setName("于瑞");
        user.setAge(18);
        user.setSex("");// 验证传入空字符，mybatis会不会报类型转换错误？结论：传入参数为空字符不会报类型转换错误，只有传入参数是null值的时候才报
        user.setHobby("发呆");
        // 调用dao接口中封装的insert方法
        userDao.insertUser(user);
        // 打印执行成功信息
        System.out.println("insert操作成功提交");

    }

    // 根据users表中的id字段删除users表中的record
    @Test
    public void testDeleteUser() throws Exception {
        // 创建UserDao的对象，注入sqlSessionFactory对象
        IBaseDao<?> userDao = new BaseDaoImpl(sqlSessionFactory);
        userDao.deleteUser(2);
        System.out.println("delete操作成功提交");

    }

    // 根据传入的原生sql指定查询操作
    @Test
    public void testFindUser() throws Exception {
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        String sql = "select * from users";
        if (!sql.trim().toLowerCase().startsWith("select")) {
            System.out.println("此处只能执行查询语句...");
            System.out.println("sql请以select开头...");
        }
        List<CommonPo> list = userDao.findUser(sql);
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据指定sql查询的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    // 根据原生sql执行查询操作
    @Test
    public void testQuery() throws Exception {
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        String sql = "select * from users";
        if (!sql.trim().toLowerCase().startsWith("select")) {
            System.out.println("此处只能执行查询语句...");
            System.out.println("sql请以select开头...");
        }
        List<CommonPo> list = (List<CommonPo>) userDao.query(sql);
        Iterator<CommonPo> iterator = list.iterator();
        Map<?, ?> map = (Map<?, ?>) list.get(0);
        Iterator<?> it = map.keySet().iterator();
        while (it.hasNext()) {
            System.out.println(it.next());
        }

        System.out.println("根据指定sql查询的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    // 更新users表中的数据行
    @Test
    public void testUpdateUser() throws Exception {
        IBaseDao<CommonPo> userDao = new BaseDaoImpl(sqlSessionFactory);
        /* 更新操作前进行对象的封装 */
        CommonPo user = new CommonPo();
        user.setId(4);
        user.setAge(29);
        user.setHobby("吃饭");
        user.setSex("女");
        user.setName("更新");
        /* service层调用dao层 */
        userDao.updateUser(user);
        System.out.println("update操作成功提交");
    }
}

```

# 2. 原始方式实现DAO-mybatis的一级缓存和二级缓存
## 2.1 定义mapper接口
zeh.mybatis.code03yuanshi.dao.IMybatisCacheDao
```java
package zeh.mybatis.code03yuanshi.dao;

import java.util.List;

// 2022/11/23 23:06
public interface IMybatisCacheDao<T> {

    // 根据id查询用户信息，测试mybatis默认开启的一级缓存，sqlSession级别的缓存
    public List<T> findUserByIdCache1(int id) throws Exception;


    // 根据id查询用户信息，测试二级缓存，mapper级别的缓存，根据namespace划分
    public List<T> findUserByIdCache2(int id) throws Exception;


    // 根据id查询用户信息，测试二级缓存，mapper级别的缓存，根据namespace划分，针对具体的 statementId 手动配置是否使用二级缓存。
    public List<T> findUserByIdCache2ForStatementIdConfig(int id) throws Exception;

    // 测试ehcache缓存
    List<T> findUserByIdEhCache(int id) throws Exception;

    // 全量查询users表信息
    public List<T> findUserAll() throws Exception;

    // 分页结合缓存
    public List<T> findUserAllByCache1() throws Exception;
}
```
## 2.2 实现mapper接口
zeh.mybatis.code03yuanshi.dao.impl.MybatisCacheDaoImpl
```java
package zeh.mybatis.code03yuanshi.dao.impl;

import org.apache.ibatis.session.RowBounds;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import zeh.mybatis.code00common.po.singleton.CommonPo;
import zeh.mybatis.code03yuanshi.dao.IMybatisCacheDao;

import java.util.Iterator;
import java.util.List;

// 2022/11/23 23:06
public class MybatisCacheDaoImpl implements IMybatisCacheDao<CommonPo> {

    private SqlSessionFactory sqlSessionFactory;

    public MybatisCacheDaoImpl(SqlSessionFactory sqlSessionFactory) {
        this.sqlSessionFactory = sqlSessionFactory;
    }

    // 根据id查询users表，方法内部实例化sqlSession对象
    // 测试 mybatis 默认开启的一级缓存
    @Override
    public List<CommonPo> findUserByIdCache1(int id) throws Exception {

        SqlSession sqlSession = this.sqlSessionFactory.openSession();

        //同一个sqlSession，第一次查询直接查询数据表
        List<CommonPo> list1 = sqlSession.selectList("cache.findUserByIdCache1YuanShi", id);

        //采用标准iterator输出集合输出List1，第一次的查询结果
        Iterator<CommonPo> iterator1 = list1.iterator();
        System.out.println("根据用户id查询的users表的record如下：");
        while (iterator1.hasNext()) {
            System.out.println(iterator1.next());
        }

        /*
         * 同一个 sqlSession 在一次事务中对相同的sql进行第二次查询。
         * 注意，只有当第一次查询的结果后续没有被同一个sqlSession指定的commit操作（insert,delete,update）时，上次查询的结果才会依旧保存在一级缓存中，否则一级缓存中缓存的数据将被清空。
         * 如果上次查询的数据在缓存中被清空了，则第二次还是会查询数据表，而不能从同一个sqlSession中的一级缓存中取出数据。
         * 同一个sqlSession，第二次查询直接使用一级缓存数据进行返回，不再查询数据表。
         * 一旦同一个sqlSession被close掉，则该session中的一级缓存结果被清空（由此可见一级缓存的范围局限性很大）。
         */
        List<CommonPo> list2 = sqlSession.selectList("cache.findUserByIdCache1YuanShi", id);

        //标准输出第二次的查询结果，观察结果，第二次将会不打印sql语句，说明是直接从一级缓存中拿出数据的
        Iterator<CommonPo> iterator2 = list2.iterator();
        System.out.println("根据用户id查询的users表的record如下：");
        while (iterator2.hasNext()) {
            System.out.println(iterator2.next());
        }

        //第三次查询。
        List<CommonPo> list3 = sqlSession.selectList("cache.findUserByIdCache1YuanShi", id);

        //标准输出第三次的查询结果，观察结果，第二次将会不打印sql语句，说明是直接从一级缓存中拿出数据的
        Iterator<CommonPo> iterator3 = list3.iterator();
        System.out.println("根据用户id查询的users表的record如下：");
        while (iterator3.hasNext()) {
            System.out.println(iterator3.next());
        }
        // 关闭sqlSession，所有缓存将被清空
        sqlSession.close();
        return list1;
    }

    // 根据id查询users表，方法内部实例化sqlSession对象。
    // 测试二级缓存，每一次调用同一个mapper下的同一个select的 statementId 都会保存到二级缓存中去。
    @Override
    public List<CommonPo> findUserByIdCache2(int id) throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("cache.findUserByIdCach2YuanShi", id);
        // 关闭sqlSession,二级缓存中sqlSession必须关闭，否则第一次的sql结果将无法写入到二级缓存中去（一级缓存不关闭不影响缓存写入，因为范围不一样）
        sqlSession.close();
        return list;
    }

    // 二级缓存开启的前提下，针对具体的 statementId 手动配置是否使用二级缓存（如果不配置，则默认所有开启二级缓存的 namespace 下的所有 statementId 都默认使用二级缓存）。
    @Override
    public List<CommonPo> findUserByIdCache2ForStatementIdConfig(int id) throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("cache.findUserByIdCach2YuanShiForStatementIdConfig", id);
        // 关闭sqlSession,二级缓存中sqlSession必须关闭，否则第一次的sql结果将无法写入到二级缓存中去（一级缓存不关闭不影响缓存写入，因为范围不一样）
        sqlSession.close();
        return list;
    }

    // 根据id查询users表，方法内部实例化sqlSession对象。
    // 测试EhCache缓存。
    @Override
    public List<CommonPo> findUserByIdEhCache(int id) throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("cache.findUserByIdEhCachYuanShi", id);
        // 关闭sqlSession
        sqlSession.close();
        return list;
    }

    // RowBounds逻辑分页全量查询
    @Override
    public List<CommonPo> findUserAll() throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();

        RowBounds rb = new RowBounds(4, 3);// 设置RowBounds分页查询参数
        List<CommonPo> list = sqlSession.selectList("cache.findUserAllYuanShi", null, rb);

        sqlSession.close();

        return list;
    }

    // mybatis的 RowBounds 逻辑分页结合一级缓存进行优化
    // 结论：逻辑分页每次传入的offset和limit不同，mybatis将认为是两个不同的statementid而不会使用缓存查询。
    @Override
    public List<CommonPo> findUserAllByCache1() throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();

        //尽管是在同一个sqlSession中，但是每次传递的分页参数不同
        List<CommonPo> list = sqlSession.selectList("cache.findUserAllYuanShi", null, new RowBounds(4, 3));
        List<CommonPo> list2 = sqlSession.selectList("cache.findUserAllYuanShi", null, new RowBounds(4, 8));

        //采用标准iterator输出集合输出List
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("全量查询的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
        Iterator<CommonPo> iterator2 = list2.iterator();
        System.out.println("全量查询的users表的record如下：");
        while (iterator2.hasNext()) {
            System.out.println(iterator2.next());
        }
        sqlSession.close();
        return list;
    }
}
```
## 2.3 resource下定义mapper.xml
mybatis/mapper/mybatis-yuanshi-cache-mapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cache">
    <!-- 开启本mapper的namespace下的二级缓存，二级缓存以namespace进行区分 ；即二级缓存是namespace级别的。 -->
    <!-- 注意此处才是开启二级缓存的配置，但是此处的配置取决于全局的配置开关是否打开（注意全局的二级缓存配置开关默认是打开的）。 -->
    <!-- 如果sqlmapconfig.xml中全局的二级缓存配置手动关闭，则即便此处开启了二级缓存也不生效。 -->
    <!-- 设置二级缓存推荐使用的是 ehcache 框架，下面的type是ehcache和mybatis整合包中的Cache接口的实现类 -->
    <!-- mybatis开放了Cache接口供第三方缓存框架实现，如果需要自定义缓存框架则需要自己实现Cache接口 -->
    <!-- Cache接口：org.apache.ibatis.cache.Cache -->
    <!-- mbatis和ehcache整合包中的实现类：org.mybatis.caches.ehcache.EhcacheCache -->
    <!-- 根据需求调整参数 -->
    <!-- 整合EhCache -->
    <!-- 如果使用<cache/>配置，则mybatis默认的二级缓存的Cache接口的实现类是：org.apache.ibatis.cache.impl.PerpetualCache。 -->
    <!-- 即mybatis自己实现的二级缓存，一般开发中都使用第三方缓存框架。 -->
    <!-- <cache /> -->
    <cache type="org.mybatis.caches.ehcache.EhcacheCache">
        <property name="timeToIdleSeconds" value="3600"/>
        <property name="timeToLiveSeconds" value="3600"/>
        <!-- 同ehcache参数maxElementsInMemory -->
        <property name="maxEntriesLocalHeap" value="1000"/>
        <!-- 同ehcache参数maxElementsOnDisk -->
        <property name="maxEntriesLocalDisk" value="10000000"/>
        <property name="memoryStoreEvictionPolicy" value="LRU"/>
    </cache>


    <!-- 测试mybatis默认开启的一级缓存 -->
    <select id="findUserByIdCache1YuanShi" parameterType="int"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        <choose>
            <when test="_parameter == 1">
                select * from zeh_user where id=#{value}
            </when>
            <otherwise>
                select * from zeh_user where id=10
            </otherwise>
        </choose>
    </select>

    <!-- 测试 mybatis 需要手动开启的二级缓存 -->
    <select id="findUserByIdCach2YuanShi" parameterType="int"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
		select * from zeh_user where id=#{value}
	</select>

    <!-- 测试 mybatis 需要手动开启的二级缓存；针对该statementId手动配置是否需要使用二级缓存。 -->
    <!-- useCache="false" 表示该 statementId 不使用二级缓存，注意对一级缓存不生效，一级缓存总是默认开启并且总是所有 statementId 默认使用。 -->
    <!-- 如果不配置该参数，在二级缓存开启的情况下，该配置默认为true，即该 statementId 默认会使用二级缓存。 -->
    <select id="findUserByIdCach2YuanShiForStatementIdConfig" parameterType="int"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo" useCache="false">
		select * from zeh_user where id=#{value}
	</select>

    <!-- 测试mybatis的二级缓存，使用第三方缓存框架 EhCache 代替 mybatis 默认的二级缓存实现。 -->
    <select id="findUserByIdEhCachYuanShi" parameterType="int"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo" useCache="true">
		select * from zeh_user where id=#{value}
	</select>

    <!-- 全量查询users表，测试RowBounds逻辑分页功能 -->
    <select id="findUserAllYuanShi" resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        <!-- 测试_parameter参数，该参数表示传入进来的查询参数对象 -->
        <if test="_parameter == null">
            select * from zeh_user
        </if>
    </select>

</mapper>
```
## 2.4 运行
zeh.mybatis.code03yuanshi.run.MybatisCacheDaoRun
```java
package zeh.mybatis.code03yuanshi.run;


import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
import zeh.mybatis.code00common.po.singleton.CommonPo;
import zeh.mybatis.code03yuanshi.dao.IMybatisCacheDao;
import zeh.mybatis.code03yuanshi.dao.impl.MybatisCacheDaoImpl;

import java.io.File;
import java.io.InputStream;
import java.util.Iterator;
import java.util.List;

// 使用Junit测试-mybatis 原始方式测试一级缓存和二级缓存。
public class MybatisCacheDaoRun {
    private SqlSessionFactory sqlSessionFactory;

    @Before
    public void setUp() throws Exception {
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        this.sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    }

    // 测试mybatis默认开启的一级缓存。
    @Test
    public void testCache1() throws Exception {
        // 创建 IMybatisCacheDao 对象
        IMybatisCacheDao<CommonPo> mybatisCacheDao = new MybatisCacheDaoImpl(sqlSessionFactory);
        //调用一级缓存，只要使用的是同一个sqlSession实例且两次查询中间没有进行过commit操作，则后续的相同查询会直接从当前的sqlSession的第一次的一级缓存中进行查询
        mybatisCacheDao.findUserByIdCache1(1);
    }

    // 测试mybatis的二级缓存
    // 需要手动开启，配置两个地方：
    // 一个是sqlMapConfig.xml中配置全局的二级缓存开关。
    // 另一个需要在相对应的mapper.xml的namespace下使用<cache/>标签开启本mapper的二级缓存。
    // 还有就是二级缓存存储的查询结果对应的pojo类必须实现序列化接口，表示该查询结果可以被缓存的。
    // 二级缓存是跨sqlSession的，它的作用范围是mapper级别的，即namespace级别的，以namespace进行区分。
    // 在main应用程序中，缓存框架的有效时间将不会生效，因为main执行完就结束了，进程都销毁了，要想观察缓存的时效性就必须在web容器中进行观察。 
    // 所以本案例需要在代码中每次都连续查询两次才能观察出二级缓存的价值。
    @Test
    public void testCache2() throws Exception {
        IMybatisCacheDao<CommonPo> mybatisCacheDao = new MybatisCacheDaoImpl(sqlSessionFactory);

        /*
         * 调用 IMybatisCacheDao 的方法，每次都实例化新的sqlSession，但是执行的是同一个namespace下的同一个 statementId。
         * 第一次的查询结果将被缓存在该mapper下的二级缓存中；
         * 第一次查询将发送sql直接查询数据库，第二次将从缓存中查询。
         */
        List<CommonPo> list1 = mybatisCacheDao.findUserByIdCache2(1);

        Iterator<CommonPo> iterator1 = list1.iterator();
        System.out.println("（第一次）根据用户id查询的users表的record如下：");
        while (iterator1.hasNext()) {
            System.out.println(iterator1.next());
        }

        //第二次查询：重新实例化sqlSession 对象。
        List<CommonPo> list2 = mybatisCacheDao.findUserByIdCache2(1);

        Iterator<CommonPo> iterator2 = list2.iterator();
        System.out.println("（第二次）根据用户id查询的users表的record如下：");
        while (iterator2.hasNext()) {
            System.out.println(iterator2.next());
        }
    }

    // 测试 mybatis 的二级缓存。
    // 在二级缓存开启的情况下（sqlMapConfig.xml和mapper.xml下都配置了二级缓存开启开关），还可以针对mapper中具体的 statementId 手动指定该 statementId 是否需要使用二级缓存，提高了二级缓存配置的灵活性。
    // 在 mapper.xml 中对应的 statementId 中配置 useCache="false" 则该 statementId 拒绝使用二级缓存。如果不配置，则默认为true。
    @Test
    public void testCache2ForStatementIdConfig() throws Exception {
        IMybatisCacheDao<CommonPo> mybatisCacheDao = new MybatisCacheDaoImpl(sqlSessionFactory);
        //第一次查询，直接发送sql到数据库。
        List<CommonPo> list1 = mybatisCacheDao.findUserByIdCache2ForStatementIdConfig(1);
        Iterator<CommonPo> iterator1 = list1.iterator();
        System.out.println("（第一次）根据用户id查询的users表的record如下：");
        while (iterator1.hasNext()) {
            System.out.println(iterator1.next());
        }

        //第二次查询，尽管对应的 mapper 配置了二级缓存的开启，但是该 statementId 手动配置了不使用二级缓存，所以二级缓存不生效。
        List<CommonPo> list2 = mybatisCacheDao.findUserByIdCache2ForStatementIdConfig(1);
        Iterator<CommonPo> iterator2 = list2.iterator();
        System.out.println("（第二次）根据用户id查询的users表的record如下：");
        while (iterator2.hasNext()) {
            System.out.println(iterator2.next());
        }
    }


    // 测试EhCache：不再使用 mybatis 默认提供的二级缓存实现，而是使用第三方提供的 ehCache 分布式缓存插件作为二级缓存。
    // 需要导入 EhCache 的jar，并且在 mapper 中集成 EhCache。
    @Test
    public void testEhCache() throws Exception {
        IMybatisCacheDao<CommonPo> mybatisCacheDao = new MybatisCacheDaoImpl(sqlSessionFactory);
        //第一次查询，向数据库发送sql进行查询。
        List<CommonPo> list1 = mybatisCacheDao.findUserByIdEhCache(1);
        Iterator<CommonPo> iterator1 = list1.iterator();
        System.out.println("（第一次）根据用户id查询的users表的record如下：");
        while (iterator1.hasNext()) {
            System.out.println(iterator1.next());
        }

        //第二次查询，直接从缓存框架EHCache中进行查询，不再向数据库发送sql。
        List<CommonPo> list2 = mybatisCacheDao.findUserByIdEhCache(1);
        Iterator<CommonPo> iterator2 = list2.iterator();
        System.out.println("（第二次）根据用户id查询的users表的record如下：");
        while (iterator2.hasNext()) {
            System.out.println(iterator2.next());
        }
    }

    // 全量获取mybatis查询的数据。
    // 检测mybatis自带的逻辑分页功能RowBounds
    // 逻辑分页除了跨数据库外，几乎一无是处，性能比较低，每次都是全量查询到ResultSet，然后再使用代码进行分页展示到前端（每次分页都全量查询，只是为了进行分页展示，假分页）。
    @Test
    public void testFindUserAll() throws Exception {
        IMybatisCacheDao<CommonPo> mybatisCacheDao = new MybatisCacheDaoImpl(sqlSessionFactory);
        List<CommonPo> list = mybatisCacheDao.findUserAll();
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("全量查询的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    // 逻辑分页 RowBounds 结合一级缓存实现性能提升
    // 结论：逻辑分页每次传入的offset和limit不同，mybatis将认为是两个不同的 statementId 而不会查询缓存
    // 所以逻辑分页mybatis的处理除了跨数据库外，性能是一塌糊涂。结合二级缓存也是一样的结果，因为执行的sql不同。
    @Test
    public void testFindUserAllByCache1() throws Exception {
        IMybatisCacheDao<CommonPo> mybatisCacheDao = new MybatisCacheDaoImpl(sqlSessionFactory);
        mybatisCacheDao.findUserAllByCache1();
    }
}
```
# 3. 原始方式实现DAO-pageHelper物理分页插件
## 3.1 定义mapper接口
zeh.mybatis.code03yuanshi.dao.IPageHelperDao
```java
package zeh.mybatis.code03yuanshi.dao;

import java.util.List;

// IPageHelperDao
public interface IPageHelperDao<T> {

    // 根据id查询用户信息
    public List<T> findUserByIdForPageHelper(String param) throws Exception;
}
```
## 3.2 实现mapper接口
zeh.mybatis.code03yuanshi.dao.impl.PageHelperDaoImpl
```java
package zeh.mybatis.code03yuanshi.dao.impl;

import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import zeh.mybatis.code00common.po.singleton.CommonPo;
import zeh.mybatis.code03yuanshi.dao.IPageHelperDao;

import java.util.List;

// PageHelperDaoImpl
public class PageHelperDaoImpl implements IPageHelperDao<CommonPo> {

    private SqlSessionFactory sqlSessionFactory;

    public PageHelperDaoImpl(SqlSessionFactory sqlSessionFactory) {
        this.sqlSessionFactory = sqlSessionFactory;
    }


    //根据id查询users表，方法内部实例化sqlSession对象。
    @Override
    public List<CommonPo> findUserByIdForPageHelper(String id) throws Exception {
        SqlSession sqlSession = this.sqlSessionFactory.openSession();
        List<CommonPo> list = sqlSession.selectList("pagehelper.findUserByIdForPageHelper", id);
        // 关闭sqlSession
        sqlSession.close();
        return list;
    }
}

```
## 3.3 resource下配置mapper.xml
注意，pageHelper插件已经在mybatis的全局配置文件中进行过配置了。   
mybatis/mapper/mybatis-yuanshi-pagehelper-mapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="pagehelper">
    <cache type="org.mybatis.caches.ehcache.EhcacheCache">
        <property name="timeToIdleSeconds" value="3600"/>
        <property name="timeToLiveSeconds" value="3600"/>
        <!-- 同ehcache参数maxElementsInMemory -->
        <property name="maxEntriesLocalHeap" value="1000"/>
        <!-- 同ehcache参数maxElementsOnDisk -->
        <property name="maxEntriesLocalDisk" value="10000000"/>
        <property name="memoryStoreEvictionPolicy" value="LRU"/>
    </cache>

    <select id="findUserByIdForPageHelper" parameterType="String"
            resultType="zeh.mybatis.code00common.po.singleton.CommonPo">
        select * from zeh_user where 1=1
        <if test="value != null">
            and id=#{value}
        </if>
    </select>

</mapper>
```

## 3.4 运行
千古总结：能够物理分页的都是一条sql直接干到底的（如果对sql结果集做加减法或者数据来源为多个模块手动拼接的数据时，一定无法使用物理分页，需要自己编写手动分页对结果集进行分页处理）   

zeh.mybatis.code03yuanshi.run.PageHelperDaoRun
```java
package zeh.mybatis.code03yuanshi.run;

import com.github.pagehelper.Page;
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
import zeh.mybatis.code00common.po.singleton.CommonPo;
import zeh.mybatis.code00common.utils.PageInfoLimitUtil;
import zeh.mybatis.code03yuanshi.dao.IPageHelperDao;
import zeh.mybatis.code03yuanshi.dao.impl.PageHelperDaoImpl;

import java.io.File;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.stream.Collectors;

// 测试 pageHelper 物理分页插件
public class PageHelperDaoRun {
    // 将 sqlSessionFactory 定义为成员变量，目的就是单例化。
    // 使用单例模式管理 sqlSessionFactory 实例，让代码中方便操作；否则若作为方法内部变量则需要每次都实例化一个 sqlSessionFactory 出来，很麻烦。
    // sqlSessionFactory 是无状态的bean，作为成员变量不存在多线程安全隐患。
    private SqlSessionFactory sqlSessionFactory;

    // 解析sqlMapConfig.xml，创建sqlSessionFactory。
    // 此方法是在执行testFindUserById之前执行。
    @Before
    public void setUp() throws Exception {
        // 定义mybatis配置文件相对 classpath 路径
        String resource = "mybatis/sqlmap" + File.separator + "sqlmap-config.xml";
        // 定义配置文件输入流
        InputStream inputStream = Resources.getResourceAsStream(resource);

        // 通过 SqlSessionFactoryBuilder 创建会话工厂实例，实例化 SqlSessionFactory 对象。
        // 将 SqlSessionFactoryBuilder 当成一个工具类使用即可，不需要使用单例管理 SqlSessionFactoryBuilder ，但是使用单例管理 SqlSessionFactory 对象。
        // 在需要创建 SqlSessionFactory 时，只需要new一次 SqlSessionFactoryBuilder 即可。
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        this.sqlSessionFactory = sqlSessionFactoryBuilder.build(inputStream);
    }

    // 测试 pageHelper 物理分页插件的使用。
    // 在执行sql前，加入PageHelper物理分页插件，每次都执行指定分页参数的查询sql，而不是全量查询，因此提升查询性能。
    // 此处将PageHelper.startPage(2, 5);写在Service层不合理，应该写在DAO层，后面紧跟mybatis的查询。
    @Test
    public void testPageHelperShiYong() throws Exception {
        // 实例化 IPageHelperDao 对象
        IPageHelperDao<CommonPo> pageHelperDao = new PageHelperDaoImpl(sqlSessionFactory);

        // 通过 PageHelper 物理分页插件进行 sql 拦截分页。
        // 第一个参数是当前页面索引，即当前查询第几页。
        // 第二个参数是当前页面应该显示的record条数，即当前页面展示的记录数。
        // 在要执行的 statementId 前面加上该分页语句，则该条 statementId 绑定的sql将会被分页拦截器拦截并拼装成带有分页参数的sql语句。
        PageHelper.startPage(2, 5);

        // 调用 IPageHelperDao 的方法。
        // 上面加上PageHelper分页参数后，会拦截sql拼装成分页sql后再执行。
        // 如果想要获取分页后的sql结果，则按照下面第一种方式。
        // 如果不仅仅想要获取单纯的结果list，还想获取分页的其他信息，则要强转成Page对象。
        // PageHelper返回的Page对象是list集合的子类，将原来的list强转成Page后会将ThreadLocal保存的page的其他属性一同塞进集合里。
//         List<CommonPo> list = pageHelperDao.findUserById(1);
        Page<CommonPo> list = (Page<CommonPo>) pageHelperDao.findUserByIdForPageHelper("3");

        //上面返回的list已经不仅仅只包含查询结果，也包含众多分页信息 ，下面将从执行后的page对象中获取page的其他属性信息
        System.out.println("-----" + list.getCountColumn());
        System.out.println("--开始行（从哪一行开始）---" + list.getStartRow());
        System.out.println("--结束行（从哪一行开始结束）---" + list.getEndRow());
        System.out.println("--排序列---" + list.getOrderBy());
        System.out.println("--当前页面索引---" + list.getPageNum());
        System.out.println("--总共显示了几页---" + list.getPages());
        System.out.println("--当前页的页面大小（显式多少行）---" + list.getPageSize());
        System.out.println("--开始行（不包含当前行）---" + list.getStartRow());
        System.out.println("--执行count的总条数---" + list.getTotal());// 获取分页sql执行count(1)的查询总量结果，如果count(1)查询标志设置为false，则此值永远为1。
        System.out.println("-----" + list.getPageSizeZero());
        System.out.println("-----" + list.getReasonable());
        System.out.println("--取得page的所有属性和查询结果集合---" + list.getResult());

        //采用标准iterator输出集合输出List
        Iterator<CommonPo> iterator = list.iterator();
        System.out.println("根据用户id查询的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next().getName());
        }
    }


    // 直接使用 PageHelper.startPage(1, 3, Boolean.TRUE); 返回的page对象接收执行后的sql结果集。
    // 使用 pageHelper 返回的page对象，实际中并不常用。
    // 实际中往往是直接使用dao接口返回的结果集，然后对其转型成page对象，因为这样代码的结果集不依赖pageHelper。
    @Test
    public void testPageHelperByOrder() throws Exception {
        IPageHelperDao<CommonPo> pageHelperDao = new PageHelperDaoImpl(sqlSessionFactory);

        // 该版本的pageHelper的startPage还有第三个参数，即设置是否将拦截sql生成count(1)的总量查询：
        // 如果设置为true，则会生成总量查询语句,当拦截后的count(1)为0时，不会生成select * from 详情查询sql；只有当count(1)大于0，才会生成select * from 详情查询sql。
        // 如果设置为false，则不会生成count(1)总量查询sql，查询总数也不会设置到结果集page或者pageInfo对象中（因为根本就没有执行查询总数的sql，count总默认为1），而且每次都会执行详情sql。
        // 默认为true。
        // PageHelper.startPage(2, 5)返回的page对象没有存储真正的分页查询结果，只是保存了对真正sql处理后的结果集的一个引用而已。
        // 在真正的 sql 执行前，此处返回的 page 对象是没有真正的属性信息的；只有当真正的查询sql执行完毕才会回调该方法将结果的引用设置进去。
        // startPage()方法被重载了很多个，可以设置排序字段，可以指定是否生成count(1)的查询sql等。
        Page<CommonPo> page = PageHelper.startPage(1, 3, Boolean.TRUE);
        //pageHelper设置排序列，注意pageHelper也可以对指定列设置orderBy，此时是拦截sql，先全量查询进行全部排序，然后再进行分页截取。
        PageHelper.orderBy("name");
        //拦截sql前，page 对象没有任何属性信息。
        System.out.println("sql执行前的page对象：" + page.getTotal());
        pageHelperDao.findUserByIdForPageHelper("3");
        //在sql执行后
        System.out.println("sql执行后的page对象：" + page.getTotal());

        // 采用标准iterator输出集合输出List
        Iterator<CommonPo> iterator = page.iterator();
        System.out.println("根据用户id查询的users表的record如下：");
        while (iterator.hasNext()) {
            System.out.println(iterator.next().getName());
        }
    }

    // 使用pageInfo对象封装结果集。
    // 对pageHelper查询的结果集做加法。
    // PageHelper只能拦截一个sql语句，如果存在多个sql执行，需要在每个sql执行前加上PageHelper进行拦截分页。
    // 只对结果集做加法，则pageInfo中的total永远是count(1)查询出来的总数，而不是做完加法后的真正总数。
    @Test
    public void testPageHelperByByJiaFa() throws Exception {
        IPageHelperDao<CommonPo> pageHelperDao = new PageHelperDaoImpl(sqlSessionFactory);
        PageHelper.startPage(1, 5, Boolean.TRUE);
        List<CommonPo> list = pageHelperDao.findUserByIdForPageHelper(null);
        System.out.println("list:" + list);
        System.out.println("list 的size:" + list.size());
        list.add(new CommonPo());//手动改装 list，进行加法，观察pageInfo的总数信息。
        list.add(new CommonPo());
        list.add(new CommonPo());

        //注意，PageInfo比Page作用更强大，都是pageHelper包提供的结果集封装对象。
        PageInfo<CommonPo> pageInfo = new PageInfo(list);
        System.out.println("封装list对象后的pageInfo总数:" + pageInfo.getTotal());
        System.out.println("封装list对象后的pageInfo size:" + pageInfo.getSize());
    }

    // 对pageHelper查询的结果集做减法。
    // 如果对list做过滤操作，然后通过pageInfo对象包装，则pageInfo中的属性是完全按照list数量来的，即做加法总数始终是count()，而做减法总数就是实际上转换的list数量。
    // 只要查询的list之后做了过滤（不论在哪一步做减法），则最后封装的pageInfo对象中的total就和list是一致绑定的。
    // 这样导致的问题是，数据库原始的count(1)将不会再绑定到pageInfo上，导致每次查询时，页面选择了多少条记录，则total总是查询出来的总条数，导致没有下一页。
    @Test
    public void testPageHelperByJianFa() throws Exception {
        IPageHelperDao<CommonPo> pageHelperDao = new PageHelperDaoImpl(sqlSessionFactory);
        PageHelper.startPage(1, 5, Boolean.TRUE);
        List<CommonPo> list = pageHelperDao.findUserByIdForPageHelper(null);
        System.out.println("list:" + list);
        System.out.println("list 的size:" + list.size());
        list.add(new CommonPo());//手动改装 list，进行加法，观察pageInfo的总数信息。
        list.add(new CommonPo());
        list.add(new CommonPo());

        list = list.stream().filter(e -> e.getId() == 3).collect(Collectors.toList());
        //做完减法再做加法
        list.add(new CommonPo());//手动改装list1，进行加法或者减法，观察pageInfo的总数信息。
        list.add(new CommonPo());
        list.add(new CommonPo());

        //注意，PageInfo比Page作用更强大，都是pageHelper包提供的结果集封装对象。
        PageInfo<CommonPo> pageInfo = new PageInfo(list);
        System.out.println("封装list对象后的pageInfo总数:" + pageInfo.getTotal());
        System.out.println("封装list对象后的pageInfo size:" + pageInfo.getSize());
    }

    // 使用PageInfo包装结果集，不可以强转。
    // 查询结果可强转成Page对象，这是因为PageHelper在拦截sql后实际上返回的是自己包内的page对象，因为Page对象是List的子类，通过向上转型为list对象了。
    // 所以可以再向下转型强转成Page对象。
    // 但是，不能够直接强转成PageInfo对象。PageInfo对象式独立的，和List没任何关系。
    // 所以结果集强转为 pageInfo 报转换异常。
    @Test
    public void testPageHelperByPageDiffPageInfo() throws Exception {
        IPageHelperDao<CommonPo> pageHelperDao = new PageHelperDaoImpl(sqlSessionFactory);
        PageHelper.startPage(1, 5, Boolean.TRUE);
        PageInfo<CommonPo> pageInfo = (PageInfo<CommonPo>) pageHelperDao.findUserByIdForPageHelper(null);
        System.out.println("pageInfo 总数：" + pageInfo.getTotal());
    }

    // 使用pageInfo包装结果集，直接 setList 将结果集设置进去。
    // 不能手动set结果集到pageInfo对象中，只能通过有参构造方式进行pageInfo对象的实例化，否则手动set进去的list造成pageInfo对象的总数始终为0。
    @Test
    public void testPageHelperBySetList() throws Exception {
        IPageHelperDao<CommonPo> pageHelperDao = new PageHelperDaoImpl(sqlSessionFactory);
        PageHelper.startPage(1, 5, Boolean.TRUE);
        List list = pageHelperDao.findUserByIdForPageHelper(null);
        PageInfo pageInfo1 = new PageInfo();
        pageInfo1.setList(list);
        System.out.println("list 大小：" + list.size());
        System.out.println("pageInfo1 总数：" + pageInfo1.getTotal());
    }

    // 使用pageInfo包装结果集，先使用 PageInfo 构造对结果集进行包装，然后再手动 setList (set一个新的list)。
    // 上述方式完全错误，set list之后会覆盖查询出来的list，尽管总数 total 不变，但是list 完全被改变了。
    // 只要使用构造包装了list，则total永远是包装时设置的真实总数。
    @Test
    public void testPageHelperByGouZaoAndSetList() throws Exception {
        IPageHelperDao<CommonPo> pageHelperDao = new PageHelperDaoImpl(sqlSessionFactory);
        PageHelper.startPage(1, 5, Boolean.TRUE);
        List<CommonPo> list = pageHelperDao.findUserByIdForPageHelper(null);

        //先使用带参数的PageInfo进行构造，再手动set List。
        PageInfo pageInfo = new PageInfo(list);
        List<CommonPo> addList = new ArrayList<>();
        addList.add(new CommonPo());
        addList.add(new CommonPo());
        addList.add(new CommonPo());
        pageInfo.setList(addList);
        System.out.println("list 大小：" + list.size());
        System.out.println("pageInfo 总数：" + pageInfo.getTotal());
    }

    // 使用pageInfo包装结果集，先使用 PageInfo 构造对结果集进行包装，然后获取出list，修改后再手动set 进去。
    // total永远不变，但是list的内容改变了。
    // 只要使用构造包装了list，则total永远是包装时设置的真实总数。
    // 只能通过有参构造方式进行pageInfo对象的实例化。
    // 总结：通过各种方式测试，得出结果-----
    // 使用pageHelper进行自动分页，不要对结果list进行加法或者减法操作（可以更改属性值）。
    // 一旦进行了加法或者减法则page对象或者pageInfo对象中实际存储的list和total是无论如何没法自动匹配的，这样导致分页失效。
    // 最简单的方案是：
    // 对有逻辑处理的结果集应该尽量编写在同一个sql中，统一交给sql进行逻辑处理，而不要通过应用代码进行处理。
    // 如果实在不行，则需要借助PageInfo对象进行手动分页参数设置了。
    @Test
    public void testPageHelperByGetListAndSetList() throws Exception {
        IPageHelperDao<CommonPo> pageHelperDao = new PageHelperDaoImpl(sqlSessionFactory);
        PageHelper.startPage(1, 5, Boolean.TRUE);
        List list = pageHelperDao.findUserByIdForPageHelper(null);
        //构造方法包装list
        PageInfo pageInfo = new PageInfo(list);
        //包装后获取list
        List<CommonPo> getList = pageInfo.getList();
        //修改获取出的list
        getList.add(new CommonPo());
        getList.add(new CommonPo());
        getList.add(new CommonPo());
        getList.add(new CommonPo());
        getList.add(new CommonPo());
        getList.add(new CommonPo());
        getList.add(new CommonPo());
        getList.add(new CommonPo());
        getList.add(new CommonPo());
        //重新手动setList
        pageInfo.setList(list);
        System.out.println("list 大小：" + list.size());
        System.out.println("pageInfo 总数：" + pageInfo.getTotal());
    }


    // 因为要对原始sql查询出来的结果集做加法或者做减法，所以此处需要手动封装PageInfo的各个参数值。
    // 相当于手动对list结果集进行分页。
    @Test
    public void testFindUserByIdShouDongFenYe() throws Exception {
        //模拟前端分页参数
        int pageNumWeb = 1;
        int pageSizeWeb = 5;

        IPageHelperDao<CommonPo> pageHelperDao = new PageHelperDaoImpl(sqlSessionFactory);
        System.out.println("********************测试升级版结果集对象pageInfo**********************");
        //手动设置分页参数后，则不需要使用自动分页了，因为使用的话是会影响结果集合的。
//        PageHelper.startPage(pageNumWeb, pageSizeWeb, Boolean.TRUE);
        //构建最终的结果集合，随便做加法和减法
        List<CommonPo> list1 = pageHelperDao.findUserByIdForPageHelper(null);
        System.out.println("list1:" + list1);
        System.out.println("list1的size:" + list1.size());
        list1.add(new CommonPo());//手动改装list1，进行加法或者减法，观察pageInfo的总数信息。
        list1.add(new CommonPo());
        list1.add(new CommonPo());
        list1.add(new CommonPo());
        list1.add(new CommonPo());

        //对自定义的list结果集进行PageInfo封装
        //分析：
        //1、如果使用pageHelper进行自动分页，则对list不论在哪一步做了过滤（做减法），然后做了其他加减法，total，size始终和加工后的list一致，导致分页失效。
        //如果没做任何减法，只做加法，则total始终为分页查出来的count，但是list的size变了。
        //2、如果不使用pageHelper进行自动分页（采取手动分页），则total，size始终的加工后的list保持一致，即查询的total就是全量的。
        PageInfo<CommonPo> realPageInfo = new PageInfo(list1);
        PageInfoLimitUtil.pageInfoLimit(list1, realPageInfo, pageNumWeb, pageSizeWeb);
        System.out.println("手动分页后的pageInfo总数:" + realPageInfo.getTotal());
        System.out.println("手动分页后的pageInfo size:" + realPageInfo.getSize());
    }
}

```