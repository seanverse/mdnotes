# 高性能MySQL读书笔记（High performance MySQL V3)


### Chapter 1 MySQL 架构与历史

##### MySQL 逻辑架构

              客户端
           连接/线程处理              
    查询                解析器       
    缓存

                        优化器
        
            存储引擎

1. 连接管理与安全性
    每个客户端都会在服务器进程中拥有一个线程，这个连接的查询只会在这个单独的线程中执行，服务器会负责缓存线程。连接基于用户密码认证，也可使用SSL连接，还可以使用X.509证书认证。
2. 优化与执行
    MySQL会解析查询，并创建内部数据结构(解析树)，然后对其进行各种优化，包括重写查询、决定表的读取顺序，以及选择合适的索引等。用户可以通过特殊的关键字提示(hint)优化器，影响它的决策过程。对于Select语句，在解析查询之前，服务器会先检查查询缓存(Query Cache)。
3. 并发控制
   
4. 锁
   共享锁(shared lock)=读写锁、排他锁(exclusive lock)=写锁
   锁粒度： 表锁(table lock)如MyISAM 行级锁(row lock)如InnoDB和XtraDB
5. 事务
   ACID 原子性（atomicity), 一致性(consistency),隔离性(isolation),持久性(durability)
   * MySQL默认是自动提交,可以设置: SHOW VARIABLES LIKE 'AUTOCOMMIT' SET AUTOOMMIT=1, 为0时要求显式的COMMIT提交或者ROLLBACK
   * 设置隔离等级，SET SESSION TRANSACTION ISOLATION READ COMMITTED

6. 隔离级别
    隔离级别|脏读可能性|不可重复可能性|幻读可能性|加锁读|备注
    -|-|-|-|-|-
    READ UNCOMMITTED|YES|YES|YES|NO|
    READ COMMITTED|NO|YES|YES|NO| 大部分DB的默认，读不到其他事务未提交的记录
    REPEATABLE READ|NO|NO|YES|NO|MySQL默认，存在幻读问题，可能会读到其他事务新插入的记录，但是InnoDB和XtraDB通过MVCC解决了这个问题。
    SERIALIZABLE|NO|NO|NO|YES|
7. 死锁：两个或多个事务在同一资源上相互占用，并请求对方占用的资源。
8. 多版本并发控制(MVCC)
   一般有乐观(optimistic)并发控制和悲观(perimistic)并发控制。InnoDB的MVCC通过每行记录的保存两个隐藏的列来实现。一是保存了行的创建时间，一是保存了过期时间(或删除时间)
   * SELECT 只查找行的系统版本号<=事务的系统版本号，可以确保读取事务之前存的以及本事务变更的。行的删除版本要么未定义，要么大于当前事务版本号，可以确保读取到的行在事务开始之前未被删除。
   * INSERT 每一行保存当前系统版本号作为行版本号
   * DELETE 删除行标识记录当前系统版本号
   * UPDATE 插入一行新记录，旧记录作行删除标识
9. MySQL的存储引擎
   * SHOW TABLE STATUS LIKE 'user'\G ， ROW_format: Dynamic,Fixed,Compressed, Engine:存储引擎
   * INNDB 存储引擎：INNDB基于聚簇索引建立的，所以它的二级索引中必须包含主键列，所以主键列很大的话，会造成其他索引也很大。Oracle的MySQL Enterprise Backup或Percona开源的XtraBackup都可以热备份。MySQL其他引擎不支持热备份。
    **InnoDB是目前唯一支持外键的内置存储引擎。**
   * MyISAM存储引擎：5.2之前的默认，提供包括全文索引，压缩，空间函数(GIS)等，但不支持事务和行级锁，崩溃后无法安全恢复。对于只读数据或数据量小可以忍受repair操作时选用。存储分为数据文件.myd和索引文件.myi。MyISAM表如果是变长行，支持256TB数据，因为指向数据记录的长度是6字节，但可以通过MAX_ROWS和AVG_ROW_LENGTH选项来调整，调整后会重建整个表和表的索引。
        > * MyISAM对整张表加锁，读时共享锁，写时加排他锁；
        > * 修复: REPAIR TABLE mytable,当Mysql服务器已经关闭，可以通过myisamchk命令行进行表检查和修复。
        > * 索引特性: 即使是BLOB和TEXT等长字段，也可以基于前500个字符创建索引，还支持全文分词索引，可以支持复杂的查询。
        > * 延迟更新索引键，如果指定 DELAY_KEY_WRITE选项，每次修改执行完成会写到内存中的键缓存区，只在清理缓存区或者关闭表时才会将对应的索引块写入到磁盘。极大提高了性能，但在异常时会造成索引损坏，需要执行修复操作。延迟更新索引键的特性，可以全局设置，也可以为单个表设置。
        > * MyISAM压缩表， 适合表在导入数据以后，不会再修改操作，(要修改时先解除压缩，改完再次压缩 myisampack),可以减少空间和I/O,压缩表也支持索引但索引也是只读的。
        > * MyISAM最大问题是表锁的问题。
    * Archive引擎：只支持INSERT和SELECT操作。它会缓存所有的写并利用zlib对插入的行进行压缩，支持行级锁和专用缓冲区，可以实现高并发的插入，不支持事务。
    * CSV引擎:通用，作为数据交换有用。
    * Federated/FederatedX引擎: 作为其他MySQL服务器的一个远程代理用。
    * Memory引擎:HEAP表，速度快，重启以后只会保留表结构，数据丢失。可以用于内存查找，缓存中间数据等，支持Hash索引，查找非常快，表级锁，并发写入性能较低，每行都是定长的,指定了varchar也会转换为char.
    * 其他引擎: NDB集群引擎，TokuDB大数据存储引擎，面向列的存储引擎(InfiniDB等)。Aria:用于代替MyISAM,MariaDB包含了该引擎，解决了崩溃安全恢复问题，MyISAM只能缓存索引，它可以缓存数据。
    * 引擎的选择考虑点：事务，备份，崩溃恢复，其他特性
    * 转换表的引擎:
        > * ALTER TABLE mytable ENGINE=InnoDB;
        > * 导出与导入: 使用mysqldump将数据导出到文件，修改sql文件的ENGINE。注意，mysqldump会createtable前加上droptable
        > * create table innodb_table like myisam_table; alter table innodb_table engine=InnoDB; insert into innodb_table select * from myisam_table;
        > * Percona toolkit提供了一个工具 pt_online-schema-change工具
10. INFORMATION_SCHEMA
> - **INFORMATION_SCHEMA**提供了对数据库元数据的访问，MySQL服务器信息，如数据库或表的名称，列的数据类型，访问权限等。 有时也把这些信息叫做数据字典或系统目录。
> 每个数据库实例都会有一个 INFORMATION_SCHEMA 库，保存的是本实例下其他所有库的信息。
 
> - INFORMATION_SCHEMA数据库包含多个只读表。 它们实际上是视图，而不是基础表，所以没有与它们关联的文件，并且你不能在它们上设置触发器。此外，数据库目录下也没有该库的目录。

> - 虽然可以使用USE语句将INFORMATION_SCHEMA选择为缺省数据库，但只能读取表的内容，不能对它们执行INSERT，UPDATE或DELETE操作。
> - 每个MySQL用户都可以访问 INFORMATION_SCHEMA，但是只能看到自己有权限的那些行。

 ### Chapter 3 服务器性能剖析
 ##### 3.3 剖析MySQL查询
 ###### 剖析服务器负载
 1. 捕获MySQL的查询到日志文件中
    设置 **long_query_time=0**可以抓到所有的查询。Percona Server对日志有更增强的功能，如查询执行计划，锁，I/O等。然后用pt-query-digest从慢查询中生成分析报告。
 2. 使用show profile:
    > * 先开启，set profiling=1; 当一条查询提交给服务器时，此工具会记录剖析信息到一张临时表(存在 information_shema.profiling表中)
    > * 然后执行 show profiles; 查看查询执行时间。、、、、、、、、、、、、、、、、、、、、、
 3. SHOW STATUS 返回一些计数器
    > * FLUSH STATUS; SELECT * FROM sakila.nice_but_slower_file_list;
        show status where Variable_name LIKE 'Handler%' OR Variable_name LIKE 'Create%';  
        > ** SHOW TABLE STATUS **

 ### Chapter 4 Schema与数据类型优化
 ##### 优化数据类型   
    > * 更小的通常更好
    > * 尽量避免NULL, sql脚本的影响以及索引、索引统计更复杂
1. 整数类型
   TINYINT(8位), SMALLINT(16位), MEDIUMINT(24位), INT(32位), BIGINT(64位) 
   整数类型具有可选的UnSIGNED,无符号时可以正数的上限提升一倍。有无符号性能是一样的。
   MySQL 可以为整数类型指定宽度，如INT(11)，大多数它没有意义，它不会限制值的合法范围，只是在一些环境只影响显示字符的个数。对于存储和计算来说，INT(1)和INT(20)是相同的。
2. 实数类型
   MySQL支持精确类型**DECIMAL**也支持不精确类型**FLOAT(4个字节)**、**DOUBLE(8个字节)**。DECIMAL（18,9)表示小数点两边各存9个数字。DECIMAL只是存储类型。所有的实数类型在内部都会用DOUBLE来作为浮点数计算类型。鉴于DECIMAL精确计算带来的代价问题，可以用BIGINT来存浮点数，实际使用时进行处理。
3. 字符串类型
   VARCHAR存储变长字符串，更省空间。如果ROW_FORMAT=FIXED，依然每一行是定长的。VARCHAR需要使用1或2个额外字节记录字符串的长度。当长度<=255使用一个字符，否则2个字节。
   CHAR存储定长，不容易产生内存碎片。会删除串的空格(VARCHAR不会)。Memory引擎只支持CHAR类型，但是Percona Server的Memory引擎支持变长的行。
   BINARY和VARBINARY，它们存储的是二进制字符串(字节码)。BINARY填充的是\0（零字节)，而CHAR填充空格，BINARY在检索时也不会去掉填充值。
4. BLOB和TEXT类型
    为存储很大的数据而设计的字符串数据类型。分别采用二进制和字符方式存储。MySQL把每个BLOB和TEXT值当作一个独立的对象处理。TEXT类型有字符串和排序规则。BLOB和TEXT不能以全部值作为排序，可以用: **ORDER BY SUBSTRING(column, length).**
    TINYTEXT,SMALLTEXT,TEXT,MEDIUMTEXT,LONGTEXT,TEXT等同是SMALLTEXT.
    TINYBLOB,SMALLBLOB,BLOB,MEDIUMBLOB,LONGBLOB,BLOG等同是SMALLBLOB.
5. 使用枚举类型ENUM代替字符串类型,内部存储为整数。MySQL会建立一个数字-字符串影射的'查找表'
    >如 Create talbe enum_test(e ENUM('fish','apple','dog')) NOT NULL
    > INSERT INTO enum_test(e) values('fish'),('dog'),('apple');
    > select e+0 from enum_test; 出来的是数字。
    > select e from enum_test; 出来的是字符串。
6. 日期和时间类型
    MySQL能存储的最小到秒，而MariaDB支持微秒。MySQL能使用微秒进行临时计算。
    DATETIME:从1001年到9999年，精度为秒。8个字节空间，使用YYYYMMDDHHMMSS的整数来存储。与时区无关。
    TIMESTAMP:表示从1970/01/01 0点以来的秒数(UTC),使用4个字节存储。它只能表示从1970-2038.显示的值依赖时区。列默认NOT NULL.
7. 位数据类型
    BIT:可以支持BIT(1)--BIT(64) MySQL把BIT当成字符串类型，检索BIT(1)返回的是二进制0或1的字符串。
    SET: 如果需要保存很多True/false值时用，MySQL提供FIND_IN_SET（）和FIELD()这样的函数。
8. 当处理UIID值时应移除**-**符号，或者用UNHEX()函数转换UUID值为16字符的数字，并存储在一个BINARY(16)的列中。检索时可以通过HEX()函数来格式化为十六进制。

**三大范式通俗解释：**
* (1) 简单归纳：
- 第一范式（1NF）：字段不可分；
- 第二范式（2NF）：有主键，非主键字段依赖主键；
- 第三范式（3NF）：非主键字段不能相互依赖。
* (2)解释：
- 1NF：原子性。 字段不可再分,否则就不是关系数据库;；
- 2NF：唯一性 。一个表只说明一个事物,要有主键；
    > 例: 表：学号、课程号、姓名、学分; 这个表明显说明了两个事务:学生信息, 课程信息;由于非主键字段必须依赖主键，这里学分依赖课程号，姓名依赖与学号，所以不符合二范式。
- 3NF：每列都与主键有直接关系，不存在传递依赖(要有外键)。
    >   表：学号、姓名、 年龄、 所在学院、学院联系电话、学院联系电话 **存在依赖传递: (学号) → (所在学院) → (学院地点, 学院电话)**

> **反范式是通过增加冗余数据或数据分组来提高数据库读性能的过程。冗余字段，缓存表，汇总表等等**

### Chapter 5 创键高性能的索引
 ##### 5.5.2 更新索引编译信息
 1. 索引优化性能
 2. 反范式优化性能
 ##### 5.5.2 更新索引编译信息
 1. MySQL的查询优化器会通过存储引擎获得可能可使用的索引和可能的扫描行数，如果扫描行数不精确或执行计划太多复杂，优化器会使用索引编译信息来估算扫描行数。
 2. 可能通过运行**ANALYZE TABLE**来重新生成统计信息。重新生成会造成表锁。
 3. Memory引擎不存储索引统计，MyISAM将索引统计信息存储在磁盘中。InnoDB不存在磁盘中，而且通过随机访问进行评估并将其存储在内存中。
 4. **SHOW INDEX FROM sakila.actor\G** 可以查看表的索引信息。在**INFORMATION_SCHEMA.STATISTICS**表可也可以查阅。
##### 5.5.2 减少索引和数据的碎片
1. 表的数据存储也可能碎片。行碎片，行间碎片，剩余空间碎片。
2. 通过执行OPTIMIZE TABLE 或者导入导出的方式来整理数据。

##### 查询执行计划成本
MySQL使用基于成本的优化器，它尝试预测一个查询使用某种执行计划时的成本，并选择其中成本最小的一个。在MySQL可以通过查询当前会话的last_query_cost的值来得到其计算当前查询的成本。

mysql> select * from t_message limit 10;
...省略结果集

mysql> show status like 'last_query_cost';
+-----------------+-------------+
| Variable_name   | Value       |
+-----------------+-------------+
| Last_query_cost | 6391.799000 |
+-----------------+-------------+

### Chapter 7 MySQL高级特性
##### 7.1 分区表
1. 每一个分区表都有一个使用#分隔命名的表文件。MySQL分区表没有全局索引，Oracle可以灵活设定。MySQL创建表时使用**PARTITION BY**子句定义每个分区存放的数据。查询优化器会自动处理分区数据。**SHOW VARIABLES LIKE '%partition%';  查询支持分区**
2. 分区表的限制和优化
- 一个表最多只能有1024个分区，分区表达式须是整数
- 如果分区字段中有主键或者唯一索引的列，那么所有主键列和唯一索引列都必须包含进来
- 分区表中无法使用外键约束
- WHERE 条件中带入分区列，有时候看似多余也要带入上。
- 分区列的NULL值会使分区过滤无效。分区列和索引列尽量是匹配的。

3. 4种常见的分区表示例：
- RANGE分区
```
create table teacher  
(id varchar(20) not null ,  
name varchar(20),  
age varchar(20),  
birthdate date not null,  
salary int  
)  
partition by range(year(birthdate))  
(  
partition p1 values less than (1970),  
partition p2 values less than (1990),  
partition p3 values less than maxvalue  
);  
或者创建表再分区：
ALTER TABLE teacher   
partition by range(year(birthdate))  
(  
partition p1 values less than (1970),  
partition p2 values less than (1990),  
partition p3 values less than maxvalue  
);  
```
- LiST分区
```
create table student  
 (id varchar(20) not null ,  
 studentno int(20) not null,  
 name varchar(20),  
 age varchar(20)  
 )  
 partition by list(studentno)  
 (  
 partition p1 values in (1,2,3,4),  
 partition p2 values in  (5,6,7,8),  
 partition p3 values in (9,10,11)  
 );  
 ```
 - HASH分区
 ```
create table user (  
  id int(20) not null,  
  role varchar(20) not null,  
  description varchar(50)   
)  
partition by hash(id)   
partitions 10;
 ```
 - Key分区,KEY分区只支持计算一列或多列
 ```
create table role( id int(20) not null,name varchar(20) not null)  
partition by linear key(id)  
partitions 10;  
 ```
4. 分区表管理
- 对指定表添加分区
 > alter table user add partition(partition p4 values less than MAXVALUE);
- 删除分区
>alter table student drop partition p1;   
- 创建子分区
```
create table role_subp(id int(20) not null,name int(20) not null)  
partition by list(id)  
subpartition by hash(name)  
subpartitions 3  
(  
  partition p1 values in(10),  
  partition p2 values in(20)  
)
```
- 合并分区
```
 alter table user  
reorganize partition p1,p3 into  
(partition p1 values less than (1000));
```
##### 7.3 外键约束
1. InnoDB是目前 MySQL 内置唯一支持外键的存储引擎。
2. 好处是外键一般会比在应用程序中检查一致性的性能要高很多。
3. 性能影响大。但是使得查询需要额外访问一些别的表，也就需要额外的锁。在向子表写入一条记录，外键约束会让InnoDB检查对应的父表的记录。也就需要对父表对应记录进行加锁操作。

##### 7.4 在MySQL内部存储代码
1.  MySQL允许通过触发器、存储过程、函数的形式来存储代码
2.  事件是5.1引入的，类似于Linux的定时任务，指定某个时候或每间隔执行一段Sql代码.事件在一个独立线程(与处理连接的线程没有关系)中执行，不接收参数，没有返回值，可以在日志中查看或者在表**INFORMATION_SCHEMA.EVENTS**中看到各个事件的状态。

##### 7.5 游标
MySQL只在服务器端提供只读的，单向的游标，只能在存储过程或底层API中用，不支持客户端游标。

##### 7.6 Prepared statement 变量化
1. 使用 **?**表示。
2. 在服务器只需要解析一次SQL语句，某些优化器工作也只需要执行一次，因为它会缓存一部分执行计划，客户端在预执行后只需要发送参数。相对更安全，减少了SQL注入的风险。
3. 优化：
- 如果可能的话，简化嵌套循环的关联，并将外关联转化成内关联。
- 尽量移除COUNT(),min(),max()优化关联顺序。
4. 支持SQL接口使用绑定变量
```
SET @sql := 'select actor_id,first_name from sakila.actor where first_name=?';
prepare stmt_fetch_actor form @sql;
set @actor_name := 'Penelope';
excute stmt_fetch_actor using @actor_name;
```
##### 7.7 用户自定义函数
1. http://www.mysqludf.org
2. MySQL是一个多线程环境，因此,udf要确保是线程安全的。
1. 支持各种样的插件，或在启动时处理或在后台执行任务。可以无须修改MySQL的源码扩展它的功能。
2. 存储过程插件，在存储过程运行后再处理一次运行结果。
3. 后台插件，实现自己的网络监听、执行自己的定期任务
4. 可以提供一个新的内存INFORMATION_SCHEMA表的插件.
5. 全文解析插件
6. 审计插件，在执行的固定点调用，因此可以生成事件日志
7. 认证插件，例如实现LDAP认证

##### 7.10 全文索引
1. MySQL 5.7.6开始，MySQL内置了ngram全文检索插件，用来支持中文分词，并且对MyISAM和InnoDB引擎有效。
##### 7.11 XA事务
1. MySQL支持XA事务。 InnoDB和二进制日志也是需要使用XA事务来协调的，从而保证系统崩溃的时候，数据能够一致性恢复。使用mysql复制时，sync_binlog设置为1，这时存储引擎和二进制日志才是真正同步的。
##### 7.12 查询缓存
1. 查询缓存的参数配置: query_cache_type:OFF,ON,DEMAND(查询语句是写时SQL_CAHCE的语句才会放入查询缓存); query_cache_size:1024字节的整数倍，...
2. 在高并发压力环境下，会导致系统性能下降，可以改用在应用程序处使用memcached等方案。使用时不要设置太大内存。