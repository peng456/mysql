# mysql
在这记录下，mysql 相关的知识吧。知识需要积累，需要写下来，做输出。


1、mysql 基础增删改查
表建立、删除、备份


2、索引  索引的原理  B+ 树。 要理解他真实的数据结构
        可以从一个问题 来理解===》 比如一千万的数据。 他的树高度。（答案一般3层，最高4层）。
        这里面 涉及 很底层的知识。对于小白，这不是一两句能解释清楚的。我有时间在细细解释。
        问题： 一千万的数据，使用索引，问mysql这个B+树 的高度？？？

3、锁  
      行级锁：
      页锁：
      表锁：
      
      实现锁的原理： MVVC 
      
      问题： 什么情况下，会导致表锁？

4、事务   不支持嵌套功能
ACID 四个特性

SET AUTOCOMMIT=0；

autocommit
select * from table for update;
autocommit=0;  

事务的隔离级别  
serializable(序列化)  
repeatable read 可重读  
read committed 提交后读  
read uncommitted 未提交读

mysql 默认隔离级别 repeatable read （确实是,我在本机的mysql 上测试是repeatable read）

那么如何修改呢？TRANSACTION ISOLATION LEVEL  这个可以修改。

5、伪事务（https://www.cnblogs.com/wade-luffy/p/6042712.html#_label1_1）
  1、用表锁定==》代替事务
    LOCK TABLES table_name lock_type,......  
    UNLOCK TABLES
  2、应用表锁实现伪事务

5、数据库中会有一个information_schema 的数据。（http://help.wopus.org/mysql-manage/607.html）  
这个库，是mysql自带的。为访问数据库元数据，提供数据。
在MySQL中，把 information_schema 看作是一个数据库，确切说是信息数据库。其中保存着关于MySQL服务器所维护的所有其他数据库的信息。如数据库名，数据库的表，表栏的数据类型与访问权 限等。  
INFORMATION_SCHEMA中，有数个只读表。它们实际上是视图，而不是基本表，因此，你将无法看到与之相关的任何文件。  





















