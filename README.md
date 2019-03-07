# mysql
在这记录下，mysql 相关的知识吧。知识需要积累，需要写下来，做输出。


1、mysql 基础增删改查
表建立、删除、备份


2、索引  索引的原理  B+ 树。 要理解他真实的数据结构
        可以从一个问题 来理解===》 比如一千万的数据。 他的树高度。（答案一般3层，最高4层）。
        这里面 涉及 很底层的知识。对于小白，这不是一两句能解释清楚的。我有时间在细细解释。
        问题： 一千万的数据，使用索引，问mysql这个B+树 的高度？？？
 
 索引的分类： 普通索引、唯一索引、主键索引、全文索引（mysql 做的不好，一般用ES等专门软件做全文搜索）
 
 全文索引
select * from `student` where match('name') against('聪'); 
            SELECT * FROM `student` WHERE MATCH(`name`) AGAINST('聪')

5.6版本之后InnoDB存储引擎开始支持全文索引，5.7版本之后通过使用ngram插件开始支持中文。之前仅支持英文，因为是通过空格作为分词的分隔符，对于中文来说是不合适的；MySQL允许在char、varchar、text类型上建立全文索引（https://www.jianshu.com/p/645402711dac）  

不知道现在什么情况了。。。。。。引文MySQL已经到8 了。
布尔全文搜索

like 是什么呢？是全文索引么？ 显然不是。。。。。

LIKE搜索的耗时随着记录数的增加而线性增长，但对于10万行记录以下的表（这里共100000*50个单词）搜索时间基本上能保持在1秒以内，所以like搜索的性能也不是特别差。由不同词汇量生成的文本对LIKE搜索的性能影响不大，不同词汇量对应的搜索时间基本上在一个很小的时间范围内变化。  
FULLTEXT搜索耗时也随表中记录数的增长而线性增加。对于10万行记录以下的表（这里共100000*50个单词）搜索时间基本上能保持在0.01秒以内。
(https://www.cnblogs.com/guifanbiji/p/6202195.html)

索引分类从大类上分有两种： Btree 、Hash

细分： Normal、Unique、FullText、Spatial   (mysql8 里面的)

http://www.cnblogs.com/lirunzhou/p/5883711.html
从物理存储角度：  聚簇索引、非聚簇索引  
从逻辑角度：      1、主键索引
                2、普通索引 或者 单列索引
                3、多列索引（复合索引）：最左前缀原则
                4、唯一索引 or 非唯一索引
                5、空间索引 ：空间索引是对空间数据类型的字段建立的索引
                            四种类型： GEOMETRY、POINT、LINESTRING、PLOYGON
                            SPATIAL关键字(空间索引)

mysql索引类型和索引方法 区别：：
               索引类型： normal，unique，full text、Spatial
               索引方法： B-Tree  、 Hash 、R-Tree

（在我的mycat如此显示。FullText、Spatial  没有显示 索引方法）
normal   ： BTree 、Hash  
unique   ： BTree 、Hash  
FullText ：  
Spatial  ：  


CREATE TABLE table_name[col_name data type] [unique|fulltext|spatial][index|key][index_name](col_name[length])[asc|desc]

例子： ALTER TABLE `baskball`.`qz_action` ADD INDEX `nice` USING BTREE (`qz_id`) comment '';



各种索引的特性 ：https://github.com/wen-fei/MySQLForOptimization  

BTree 索引特点：  
        以B+树的结构存储数据;
        能够加快数据的查询速度；
        更适合进行 “范围查找”  
        
 以下情况可以用到B树索引：  
         全值匹配查询 ：： 例如order_sn='6767654'查询指定订单号商品
         最左前缀的查询：：
         匹配列前缀查询：  比如  order_sn like "676%"  
         匹配范围查找： 
         精确匹配左前列 并 范围匹配 另一列  
 
 其自身限制：
        非左前缀，==》 无法使用索引  
        索引时，不能条做中间的列
        not in   和 <> 不能使用索引
        如果查询中某个列的范围查找，则其  右边 所有的列 都无法使用索引  ？？？？ select *  from table where a = 1 and  b > 1 and b < 10 and  c = 11 and d =18  
        其中 c、d 都不能使用索引。
        
        
hash 索引：
        基于hash的自身实现原理 ==> 只能精确匹配  等值查询  ==》 才能用到hash 
        实现原理  fun(colmun1,colmun2,colmun3,.....) = hash值。 索引里面存的就是这个hash值。
 
 限制::   
        hash 必须二次查找。 索引中的数据结构     fun(colmun1,colmun2,colmun3,.....) = hash值 <==映射==> 指针（应该是主键，主簇索引嘛）（数据的物理地址）
        所以==》 会全扫 这个 hash 池。但是 基本都在缓存中，速度很快，大部分情况下，对性能影响不大。（也许达到一定级别，会有瓶颈。比如几亿 hash 应该有几个G大。这时候的性能===》 必须重新设计了（上亿url 中查重问题））   
        hash 索引 无法用于排序（hash 值没法索引）  
        不支持 部分索引  也不支持 范围索引
        hash 可能产生冲突 且 不可避免  ；稀疏性小的列不能 进行 hash 索引。

使用索引的原因：
        减少存储引擎需要扫面的数量
        可以帮助我们进行排序，以免临时表  
        索引 可以把 随机索引 ==》 变为顺序索引
      
 索引弊端：
         增加了写操作的成本
         太多索引  ==》 会 增加  查询优化器  的选择时间
         

联合索引：
        索引顺讯原则：经常使用的列优先
                    稀疏性高的放左边（状态性的索引不要放左边）
                    宽度小的列优先（IO小）


覆盖索引  ：   （包含了所有需要查询的字段值得索引）  
                优点：
                可以优化缓存，减少磁盘IO操作
                可以减少随机IO，变随机IO操作为顺序IO操作（B树索引可以把随机IO变为顺序IO，顺序IO处理速度更快）
                可以避免对Innodb逐渐索引的二次查询（二级索引在叶子结点中保存的是行的主键值，查找到相应的主键后，还要通过主键进行二次查询，才能够获取行的数据）
                可以避免MyISAM表进行系统调用


无法使用覆盖索引的情况

存储引擎不支持覆盖索引
查询中使用了太多的列
使用了双%号的like查询



 mysql 的 GTID ===> 主从复制更快、MGR 等（https://www.cnblogs.com/f-ck-need-u/p/9164823.html）
        

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

6、数据库中会有一个information_schema 的数据。（http://help.wopus.org/mysql-manage/607.html）  
这个库，是mysql自带的。为访问数据库元数据，提供数据。
在MySQL中，把 information_schema 看作是一个数据库，确切说是信息数据库。其中保存着关于MySQL服务器所维护的所有其他数据库的信息。如数据库名，数据库的表，表栏的数据类型与访问权 限等。  
INFORMATION_SCHEMA中，有数个只读表。它们实际上是视图，而不是基本表，因此，你将无法看到与之相关的任何文件。  


7、mysql 语句 执行顺序：  
首先分析mysql 语句定义。就是有哪些固定格式：  
<SELECT clause> [<FROM clause>] [<WHERE clause>] [<GROUP BY clause>] [<HAVING clause>] [<ORDER BY clause>] [<LIMIT clause>]   
        
就是以上了。那么他的执行顺序是什么呢？  

from子句==》 where子句 ==》 group by 子句 ===》having子句  ==》 order by 子句 ===》select 子句 ==》limit 子句 ==》最终结果  

基本就是这个顺序。后一个字句  依赖前一个子句产生的结果。所以中间会有很多中间结果。 如果不存在某个子句，就直接跳过。

参考blog:(https://blog.csdn.net/u014044812/article/details/51004754)

 
执行 GROUP BY 子句, 把 tb_Grade 表按 "学生姓名" 列进行分组(注：这一步开始才可以使用select中的别名，他返回的是一个游标，而不是一个表，所以在where中不可以使用select中的别名，而having却可以使用，感谢网友  zyt1369  提出这个问题)
原文：https://blog.csdn.net/u014044812/article/details/51004754 

经过我查阅资料，关于mysql的update，insert into和delete是不能使用别名的

你好，这是因为在group by 这一步是第一步也是唯一一步可以使用SELECT列表中的列别名的步骤，它不返回有效的表，而是返回一个游标。所以你在having中可以使用 cal 而在where中是不可以使用 cal的，筛选结果的话因为 HAVING cal , 所以显示的 versions 应该都要的。(2年前)举报回复


8、根据mysql慢日志监控SQL语句执行效率（https://www.zhangshengrong.com/p/zAaOLMpNdb/）

首先你的mysql必须开启这个功能。==》 执行sql : show variables like '%slow%'; 出现如下变量： 
==》log_slow_admin_statements 、log_slow_slave_statements、slow_launch_time、slow_query_log、slow_query_log_file
===》slow_query_log 这个默认是 off (关闭的)。slow_query_log_file 慢查询日志存放磁盘位置

或者 到 my.cnf 文件里面配置。然后重启mysql.


===》 可以 使用mysql 性能监控工具  Innotop （https://www.zhangshengrong.com/p/YjNK83lXW2/）
























