---
layout:     post
title:      JDBC批处理
subtitle:   JDBC批量操作DB
date:       2021-06-03
author:     zhaoeh
header-img: img/post-bg-myself7.jpg
catalog: true
tags:
    - DB/JDBC/事务
---

# 1. JDBC批处理操作
批量处理允许您将相关的SQL语句分组到批处理中，并通过对数据库的一次调用提交它们。  
当您一次向数据库发送多个SQL语句时，可以减少连接数据库的开销，从而提高性能。  

## 1.1 使用Statement对象进行批处理操作
以下是使用语句对象的批处理的典型步骤序列：  
1.  使用createStatement（）方法创建Statement对象。  
2.  使用setAutoCommit（）将auto-commit设置为false 。  
3.  使用addBatch（）方法在创建的语句对象上添加您喜欢的SQL语句到批处理中。  
4.  在创建的语句对象上使用executeBatch（）方法执行所有SQL语句。  
5.  最后，使用commit（）方法提交所有更改。  
```java
// Create statement object
Statement stmt = conn.createStatement();
​
// Set auto-commit to false
conn.setAutoCommit(false);
​
// Create SQL statement
String SQL = "INSERT INTO Employees (id, first, last, age) " +
             "VALUES(200,'Zia', 'Ali', 30)";
// Add above SQL statement in the batch.
stmt.addBatch(SQL);
​
// Create one more SQL statement
String SQL = "INSERT INTO Employees (id, first, last, age) " +
             "VALUES(201,'Raj', 'Kumar', 35)";
// Add above SQL statement in the batch.
stmt.addBatch(SQL);
​
// Create one more SQL statement
String SQL = "UPDATE Employees SET age = 35 " +
             "WHERE id = 100";
// Add above SQL statement in the batch.
stmt.addBatch(SQL);
​
// Create an int[] to hold returned values
int[] count = stmt.executeBatch();
​
//Explicitly commit statements to apply changes
conn.commit();
```
 ##  1.2 PrepareStatement对象进行批处理
 1.  使用占位符创建SQL语句。  
 2.  使用prepareStatement（） 方法创建PrepareStatement对象。  
 3.  使用setAutoCommit（）将auto-commit设置为false 。  
 4.  使用addBatch（）方法在创建的语句对象上添加您喜欢的SQL语句到批处理中。  
 5.  在创建的语句对象上使用executeBatch（）方法执行所有SQL语句。  
 6.  最后，使用commit（）方法提交所有更改。  
```java
// Create SQL statement
String SQL = "INSERT INTO Employees (id, first, last, age) " +
             "VALUES(?, ?, ?, ?)";
​
// Create PrepareStatement object
PreparedStatemen pstmt = conn.prepareStatement(SQL);
​
//Set auto-commit to false
conn.setAutoCommit(false);
​
// Set the variables
pstmt.setInt( 1, 400 );
pstmt.setString( 2, "Pappu" );
pstmt.setString( 3, "Singh" );
pstmt.setInt( 4, 33 );
// Add it to the batch
pstmt.addBatch();
​
// Set the variables
pstmt.setInt( 1, 401 );
pstmt.setString( 2, "Pawan" );
pstmt.setString( 3, "Singh" );
pstmt.setInt( 4, 31 );
// Add it to the batch
pstmt.addBatch();
​
//add more batches
​
//Create an int[] to hold returned values
int[] count = stmt.executeBatch();
​
//Explicitly commit statements to apply changes
conn.commit();
```
## 1.3 通过循环控制批处理
批处理的特性，往往会被我们应用到完全相同的SQL语句（只是参数不同）中，然后通过循环来迭代提交，如下案例：  
```java
String SQL = "INSERT INTO Employees (id, first, last, age) " + "VALUES(?, ?, ?, ?)";
while(resultSet.next()){
    pstmt.setInt( 1, resultSet.getInt(1) );
    pstmt.setString( 2, resultSet.getString(2) );
    pstmt.setString( 3, "resultSet.getString(3) );
    pstmt.setInt( 4, resultSet.getInt(4) );
    pstmt.addBatch();
}
```
上述案例在模拟一个数据复制的过程，resultSet是从源表中查询出来的数据结果集，然后遍历这个结果集，迭代设置不同的insert参数，每次构建好SQL语句后，进行addBatch操作。  
最终再统一进行executeBatch。  

# 2. 为什么jdbc批处理能够提升性能？
通过前面的知识可以知道，一个数据库连接可以执行一条sql语句，然后关闭连接释放资源；也可以执行多条sql语句，然后关闭连接释放资源。  
jdbc发送的任何sql语句，在数据库里都是一次交互，即一条sql命令就是一次交互，10条sql语句就是10次交互。  
而每一次交互，数据库都需要读取该命令，解析该命令，然后执行该命令，最后释放该命令，也就是说jdbc和数据库的交互越频繁，性能消耗越大。  
因此，jdbc2.0提出了一种全新的批处理方式：一次性将多条sql语句先准备好，然后统一提交给数据库，这样jdbc就只和数据库进行了一次交互，消耗比较小。  

批处理需要注意：  
如果批处理操作没有在同一个事务中，性能会比较低。批处理一次性将一批SQL命令上送给数据库，只和数据库进行一次交互，但是如果采用数据库默认的事务，则这一批SQL命令中每一条都是一个单独的事务，即每一条SQL命令都会开启事务，执行，然后提交事务。  
所以说，尽管批处理只和数据库进行了一次交互，但是在数据库方面将这一批SQL划分为单个的事务进行，还是比较影响性能。  
比较优秀的做法是：<b>将批处理的所有SQL语句放在同一个事务中，这样一来同时满足了两点，一是多条SQL成批上送只和数据库进行一次交互；二是数据库收到批量SQL后发现在同一个事务中，只会进行一次事务开启和提交，这样就能极大的提升性能。  