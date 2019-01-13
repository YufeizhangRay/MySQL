# MySQL  
  
- [索引](#索引)  
  - [索引是什么](#索引是什么)  
  - [为什么要用索引](#为什么要用索引)  
  - [索引结构为什么采用B+Tree](#索引结构为什么采用btree)  
  - [MySQL B+Tree索引体现形式](#mysql-btree索引体现形式)  
  - [索引知识点](#索引知识点)  
  - [索引使用注意点](#索引使用注意点)  
- [MySQL插拔式的存储引擎](#mysql插拔式的存储引擎)  
  - [MySQL存储引擎介绍](#mysql存储引擎介绍)  
  - [CSV存储引擎](#csv存储引擎)  
  - [Archive存储引擎](#archive存储引擎)  
  - [Memory存储引擎](#memory存储引擎)  
  - [MyISAM](#myisam)  
  - [InnoDB](#innodb)  
  - [对比](#对比)    
- [MySQL体系结构及运行机理](#mysql体系结构及运行机理)  
  - [MySQL体系结构](#mysql体系结构)
- [MySQL查询优化](#mysql查询优化)  
  - [客户端/服务端通信](#客户端服务端通信)  
  - [查询状态](#查询状态)  
  - [查询缓存](#查询缓存)  
  - [查询优化处理](#查询优化处理)  
  - [查询执行引擎](#查询执行引擎)  
  - [返回客户端](#返回客户端)  
- [慢SQL](#慢sql)  
  - [如何定位慢SQL](#如何定位慢sql)  
  - [慢查询日志分析](#慢查询日志分析)  
  - [慢查询日志分析工具](#慢查询日志分析工具)  
- [事务](#事务)  
  - [什么是事务](#什么是事务)  
  - [事务的ACID特性](#事务的acid特性)  
  - [事务并发带来的的问题](#事务并发带来的的问题)  
  - [事务的四种隔离级别](#事务的四种隔离级别)  
  - [InnoDB引擎对事务隔离的支持程度](#innodb引擎对事务隔离的支持程度)  
- [数据库的锁](#数据库的锁)  
  - [表锁和行锁](#表锁和行锁)  
  - [InnoDB锁类型](#innodb锁类型)  
  - [利用锁解决并发问题](#利用锁解决并发问题)  
  - [死锁](#死锁)  
  - [死锁的避免](#死锁的避免)  
- [MVCC](#MVCC)  
  - [MVCC是什么](#mvcc是什么)  
  - [MVCC版本控制案例](#mvcc版本控制案例)  
- [Undo Log](#undo-log)  
  - [Undo Log是什么](#undo-log是什么)    
  - [当前读、快照读](#当前读快照读)  
- [Redo Log](#redo-log)  
  - [Redo Log是什么](#redo-log是什么)  
  - [Redo Log补充](#redo-log补充)  
- [MySQL配置优化](#mysql配置优化)  
  - [服务器参数](#服务器参数)  
  - [寻找配置文件](#寻找配置文件)  
  - [全局配置文件配置](#全局配置文件配置)  
  - [内存参数配置](#内存参数配置)  
  - [其他参数配置](#其他参数配置)  
- [数据库表设计](#数据库表设计)  
  - [三大范式](#三大范式)  
  - [完全满足范式的缺点](#完全满足范式的缺点)  
  
## MySQL性能优化分析   
  
### 索引  
  
正确的创建合适的索引是提升数据库查询性能的基础。  

#### 索引是什么  
索引是为了加速对表中数据行的检索而创建的一种分散存储的数据结构(硬盘级)。  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E7%B4%A2%E5%BC%95.jpeg)  
  
#### 为什么要用索引  
>索引能极大的减少存储引擎需要扫描的数据量。  
>索引可以把随机IO变成顺序IO。  
>索引可以帮助我们在进行分组、排序等操作时，避免使用临时表。  

#### 索引结构为什么采用B+Tree  
二叉查找树 Binary Search Tree  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E4%BA%8C%E5%8F%89%E6%A0%91.jpeg)  
  
平衡二叉查找树 Balanced Binary Search Tree  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E7%9B%B8%E5%AF%B9%E5%B9%B3%E8%A1%A1%E4%BA%8C%E5%8F%89%E6%A0%91.jpeg)  
  
二叉树和平衡二叉树的缺点：  
  
树结构太深  
因为每个节点都存有数据，数据处的(高)深度决定着他的IO操作次数，IO操作耗时大。  
  
数据存储太小  
每一个磁盘块(节点/页，单位4kb)保存的数据量(远远不足4k)太小了  
没有很好的利用操作磁盘IO的数据交换特性，也没有利用好磁盘IO的预读能力(空间局部性原理，预读8k、12k等)，从而带来频繁的IO操作。  
  
多路平衡查找树 B-Tree  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E7%BB%9D%E5%AF%B9%E5%B9%B3%E8%A1%A1%E4%BA%8C%E5%8F%89%E6%A0%91.jpeg)  
  
平衡查找树的分支数量与关键字大小有关，可以大致认为：磁盘块的容量/关键字大小 = 平衡树分支数量。  
  
加强版的多路平衡查找树 Mysql的B+Tree  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/B%2Btree.jpeg)  
  
B+Tree与B-Tree的区别  
>1.B+节点关键字搜索采用闭合区间  
2.B+非叶节点不保存数据相关信息，只保存关键字和子节点的引用  
3.B+关键字对应的数据保存在叶子节点中  
4.B+叶子节点是顺序排列的，并且相邻节点具有顺序引用的关系  
  
为什么选择B+Tree  
B+树是B-树的变种(PLUS版)多路绝对平衡查找树，他拥有B-树的优势  
B+树扫库、表能力更强  
B+树的磁盘读写能力更强  
B+树的排序能力更强   
B+树的查询效率更加稳定  
  
#### Mysql B+Tree索引体现形式  
Myisam  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/Myisam%E7%B4%A2%E5%BC%95.jpeg)  
  
数据保存在MYD文件，索引文件为MYI。  
  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/Myisam%E5%8F%8C%E7%B4%A2%E5%BC%95.jpeg)  
  
Innodb  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/InnoDB%E7%B4%A2%E5%BC%95.jpeg)  
  
未指定索引的情况下InnoDB会自动生成隐式索引。  
  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/InnoDB%E5%8F%8C%E7%B4%A2%E5%BC%95.jpeg)  
  
这样设计的好处就是在数据迁移的时候辅助索引可以不做作相应的指向改变。  
对于InnoDB的辅助索引，它的叶子节点存储的是索引值和指向主键索引的位置。  
  
Myisam VS Innodb  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E7%B4%A2%E5%BC%95%E5%AF%B9%E6%AF%94.jpeg)  
  
#### 索引知识点  
列的离散性  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E7%B4%A2%E5%BC%95%E7%A6%BB%E6%95%A3%E6%80%A7.jpeg)  
  
离散性低的索引会造成选择性差，无法寻找合适的分支，数库会使用全局扫描。类似男女这种字段如果简历索引则要简历位图索引。  
  
最左匹配原则  
对索引中关键字进行计算(对比)，一定是从左往右依次进行(每一位)，且不可跳过。
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E6%9C%80%E5%B7%A6%E5%8C%B9%E9%85%8D.jpeg)  
  
联合索引  
单列索引：节点中关键字[name]  
联合索引：节点中关键字[name,phoneNum]  
单列索引是特殊的联合索引  
  
联合索引列选择原则  
>1.经常用的列优先 【最左匹配原则】  
2.选择性(离散度)高的列优先【离散度高原则】  
3.宽度小的列优先【最少空间原则】  
  
覆盖索引  
如果索引包含所有满足查询需要的数据的索引成为覆盖索引(Covering Index)，也就是平时所说的不需要回表操作。  
使用explain，可以通过输出的extra列来判断，对于一个索引覆盖查询，显示为using index，MySQL查询优化器在执行查询前会决定是否有索引覆盖查询。  
覆盖索引可减少数据库IO，将随机IO变为顺序IO，可提高查询性能。  

#### 索引使用注意点  
索引列的数据长度能少则少。  
索引一定不是越多越好，越全越好，一定是建合适的。  
匹配列前缀可用到索引 like 9999%，like %9999%、like %9999用不到索引; Where 条件中 not in 和 <>操作无法使用索引;  
匹配范围值，order by 也可用到索引;   
多用指定列查询，只返回自己想到的数据列，少用select *;   
联合索引中如果不是按照索引最左列开始查找，无法使用索引;   
联合索引中精确匹配最左前列并范围匹配另外一列可以用到索引;   
联合索引中如果查询中有某个列的范围查询，则其右边的所有列都无法使用索引;  
  
### Mysql插拔式的存储引擎  
  
#### Mysql存储引擎介绍  
>1.插拔式的插件方式。  
2.存储引擎是指定在表之上的，即一个库中的每一个表都可以指定专用的存储引擎。  
3.不管表采用什么样的存储引擎，都会在数据区，产生对应的一个frm文件(表结构定义描述文件)。  
  
#### CSV存储引擎  
数据存储以CSV文件  
特点:  
不能定义索引、列定义必须为NOT NULL、不能设置自增列 -->不适用大表或者数据的在线处理  
CSV数据的存储用,隔开，可直接编辑CSV文件进行数据的编排 -->数据安全性低  
注:编辑之后，要生效使用flush table XXX 命令  
  
应用场景:  
数据的快速导出导入 表格直接转换成CSV  
  
#### Archive存储引擎  
压缩协议进行数据的存储  
数据存储为ARZ文件格式  
  
特点:  
只支持insert和select两种操作  
只允许自增ID列建立索引  
行级锁  
不支持事务  
数据占用磁盘少  
  
应用场景:  
日志系统  
大量的设备数据采集  
  
#### Memory存储引擎  
数据都是存储在内存中，IO效率要比其他引擎高很多。  
服务重启数据丢失，内存数据表默认只有16M。  
  
特点:  
支持hash索引，B tree索引，默认hash(查找复杂度0(1))  
字段长度都是固定长度varchar(32)=char(32)  
不支持大数据存储类型字段如 blog，text  
表级锁  
  
应用场景:  
等值查找热度较高数据  
查询结果内存中的计算，大多数都是采用这种存储引擎  
作为临时表存储需计算的数据  

#### Myisam  
Mysql5.5版本之前的默认存储引擎  
较多的系统表也还是使用这个存储引擎  
系统临时表也会用到Myisam存储引擎  
  
特点:  
select count(*) from table 无需进行数据的扫描  
数据(MYD)和索引(MYI)分开存储  
表级锁  
不支持事务  
  
#### Innodb  
Mysql5.5及以后版本的默认存储引擎  
  
Key Advantages:  
Its DML operations follow the ACID model [事务ACID]  
Row-level locking[行级锁]  
InnoDB tables arrange your data on disk to optimize queries based on primary keys[聚集索引(主键索引)方式进行数据存储]  
To maintain data integrity, InnoDB supports FOREIGN KEY constraints[支持外键关系保证数据完整性]  
  
#### 对比  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E5%BC%95%E6%93%8E%E5%AF%B9%E6%AF%94.jpeg)  
  
### MySQL体系结构及运行机理  
  
#### MySQL体系结构  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E6%9E%B6%E6%9E%84.jpeg)  
  
Client Connectors  
接入方 支持协议很多  
Management Serveices & Utilities 系统管理和控制工具，mysqldump、 mysql复制集群、分区管理等  
  
Connection Pool  
连接池:管理缓冲用户连接、用户名、密码、权限校验、线程处理等需要缓存的需求  
  
SQL Interface  
SQL接口:接受用户的SQL命令，并且返回用户需要查询的结果  
  
Parser  
解析器，SQL命令传递到解析器的时候会被解析器验证和解析。解析器是由Lex和YACC实现的  
  
Optimizer  
查询优化器，SQL语句在查询之前会使用查询优化器对查询进行优化  

Cache和Buffer(高速缓存区)  
查询缓存，如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据  
  
pluggable storage Engines 插件式存储引擎。存储引擎是MySql中具体的与文件打交道的子系统  
  
file system  
文件系统，数据、日志(redo，undo)、索引、错误日志、查询记录、慢查询等  
  
### MySQL查询优化  
  
#### MySQL查询执行路径  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E6%9F%A5%E8%AF%A2%E6%B5%81%E7%A8%8B.jpeg)  
  
>1.mysql 客户端/服务端通信  
2.查询缓存  
3.查询优化处理  
4.查询执行引擎  
5.返回客户端  
  
#### 客户端/服务端通信  
  
Mysql客户端与服务端的通信方式是“半双工”;  
  
全双工:双向通信，发送同时也可以接收  
半双工:双向通信，同时只能接收或者是发送，无法同时做操作  
单工:只能单一方向传送  
  
半双工通信: 在任何一个时刻，要么是有服务器向客户端发送数据，要么是客户端向服务端发送数据，这两个动作不能同时发生。所以我们无法也无需将一个消息切成小块进 行传输。  
  
特点和限制: 客户端一旦开始发送消息，另一端要接收完整个消息才能响应。  
客户端一旦开始接收数据没法停下来发送指令。  
  
#### 查询状态  
对于一个mysql连接，或者说一个线程，时刻都有一个状态来标识这个连接正在做什么  
查看命令 show full processlist / show processlist  
https://dev.mysql.com/doc/refman/5.7/en/general-thread-states.html (状态全集)  
  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E6%9F%A5%E8%AF%A2%E5%8F%82%E6%95%B0.jpeg)  
  
Sleep  
线程正在等待客户端发送数据  
Query  
连接线程正在执行查询  
Locked  
线程正在等待表锁的释放  
Sorting result  
线程正在对结果进行排序  
Sending data  
向请求端返回数据  
  
可通过kill {id} 的方式进行连接的杀掉  
  
#### 查询缓存  
工作原理:  
>缓存SELECT操作的结果集和SQL语句;  
新的SELECT语句，先去查询缓存，判断是否存在可用的记录集;  
  
判断标准:  
与缓存的SQL语句，是否完全一样，区分大小写(简单认为存储了一个key-value结构，key为sql，value为sql查询结果集)。  
  
query_cache_type  
>值:0 -– 不启用查询缓存，默认值;  
值:1 -– 启用查询缓存，只要符合查询缓存的要求，客户端的查询语句和记录集都可以缓存起来，供其他客户端使用，加上SQL_NO_CACHE将不缓存。  
值:2 -– 启用查询缓存，只要查询语句中添加了参数:SQL_CACHE，且符合查询缓存的要求，客户端的查询语句和记录集，则可以缓存起来，供其他客户端使用。  
  
query_cache_size  
>允许设置query_cache_size的值最小为40K，默认1M，推荐设置为:64M/128M;   
query_cache_limit 限制查询缓存区最大能缓存的查询记录集，默认设置为1M  
  
`show status like 'Qcache%' `命令可查看缓存情况。  
  
不会缓存的情况  
>1.当查询语句中有一些不确定的数据时，则不会被缓存。如包含函数NOW()，CURRENT_DATE()等类似的函数，或者用户自定义的函数，存储函数，用户变量等都不会被缓存。  
2.当查询的结果大于query_cache_limit设置的值时，结果不会被缓存。  
3.对于InnoDB引擎来说，当一个语句在事务中修改了某个表，那么在这个事务提交之前，所有与这个表相关的查询都无法被缓存。因此长时间执行事务，会大大降低缓存命中率。  
4.查询的表是系统表  
5.查询语句不涉及到表  
  
为什么mysql默认关闭了缓存开启  
>1.在查询之前必须先检查是否命中缓存,浪费计算资源。  
2.如果这个查询可以被缓存，那么执行完成后，MySQL发现查询缓存中没有这个查询，则会将结果存入查询缓存，这会带来额外的系统消耗。  
3.针对表进行写入或更新数据时，将对应表的所有缓存都设置失效。  
4.如果查询缓存很大或者碎片很多时，这个操作可能带来很大的系统消耗。  

缓存适用业务场景  
以读为主的业务，数据生成之后就不常改变的业务  
比如门户类、新闻类、报表类、论坛类等  
  
#### 查询优化处理  
查询优化处理的三个阶段:  
>1.解析sql通过lex词法分析,yacc语法分析将sql语句解析成解析树。  
https://www.ibm.com/developerworks/cn/linux/sdk/lex/  
2.预处理阶段 根据mysql的语法的规则进一步检查解析树的合法性，如：检查数据的表和列是否存在，解析名字和别名的设置，还会进行权限的验证。  
3.查询优化器 优化器的主要作用就是找到最优的执行计划。  
  
查询优化器如何找到最优执行计划  
• 使用等价变化规则  
>5=5 and a>5 改写成 a>5  
a<b and a=5 改写成 b>5 and a=5  
  
• 优化count 、min、max等函数  
>min函数只需找索引最左边  
max函数只需找索引最右边  
myisam引擎count(*)  
  
• 覆盖索引扫描  
• 子查询优化  
• 提前终止查询  
>用了limit关键字或者使用不存在的条件
• IN的优化
>先进性排序，再采用二分查找的方式
...  
  
Mysql的查询优化器基于成本计算的原则，会尝试各种执行计划，以数据抽样的方式进行试验(随机的读取一个4K的数据块进行分析)。  
  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E6%9F%A5%E8%AF%A2%E4%BC%98%E5%8C%96.jpeg)  
  
id  
select查询的序列号，标识执行的顺序  
>1.id相同，执行顺序由上至下。  
2.id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行。  
3.id相同又不同即两种情况同时存在，id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行。  
  
select_type  
查询的类型，主要是用于区分普通查询、联合查询、子查询等  
>SIMPLE:简单的select查询，查询中不包含子查询或者union  
PRIMARY:查询中包含子部分，最外层查询则被标记为primary  
SUBQUERY/MATERIALIZED:  
SUBQUERY表示在select或where列表中包含了子查询  
MATERIALIZED表示where后面in条件的子查询  
UNION:若第二个select出现在union之后，则被标记为union   
UNION RESULT:从union表获取结果的select  
  
table  
查询涉及到的表  
直接显示表名或者表的别名  
<unionM,N> 由ID为M,N 查询union产生的结果  
<subqueryN> 由ID为N查询生产的结果  
  
type  
访问类型，sql查询优化中一个很重要的指标，结果值从好到坏依次是:   
system > const > eq_ref > ref > range > index > ALL  
>system:表只有一行记录(等于系统表)，const类型的特例，基本不会出现，可以忽略不计。  
const:表示通过索引一次就找到了，const用于比较primary key或者unique索引。  
eq_ref:唯一索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描。  
ref:非唯一性索引扫描，返回匹配某个单独值的所有行，本质是也是一种索引访问。  
range:只检索给定范围的行，使用一个索引来选择行。  
index:Full Index Scan，索引全表扫描，把索引从头到尾扫一遍。  
ALL:Full Table Scan，遍历全表以找到匹配的行。  
  
possible_keys  
查询过程中有可能用到的索引  
  
key  
实际使用的索引，如果为NULL，则没有使用索引  
  
rows  
根据表统计信息或者索引选用情况，大致估算出找到所需的记录所需要读取的行数  
  
filtered  
表示返回结果的行数占需读取行数的百分比，filtered的值越大越好  
  
Extra  
十分重要的额外信息  
>1.Using filesort : mysql对数据使用一个外部的文件内容进行了排序，而不是按照表内的索引进行排序读取。  
2.Using temporary: 使用临时表保存中间结果，也就是说mysql在对查询结果排序时使用了临时表，常见于order by 或 group by。  
3.Using index:表示相应的select操作中使用了覆盖索引(Covering Index)，避免了访问表的数据行，效率高。  
4.Using where : 使用了where过滤条件。  
5.select tables optimized away: 基于索引优化MIN/MAX操作或者MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段在进行计算，查询执行计划生成的阶段即可完成优化。  
  
#### 查询执行引擎  
调用插件式的存储引擎的原子API的功能进行执行计划的执行。  
  
#### 返回客户端  
>1.有需要做缓存的，执行缓存操作。  
2.增量的返回结果：开始生成第一条结果时，mysql就开始往请求方逐步返回数据。  
好处: mysql服务器无须保存过多的数据，浪费内存；用户体验好，马上就拿到了数据。  
  
### 慢SQL
  
#### 如何定位慢SQL   
>1.业务驱动  
2.测试驱动  
3.慢查询日志  
```  
show variables like 'slow_query_log'  
set global slow_query_log = on  
set global slow_query_log_file = '/var/lib/mysql/test-slow.log'   
set global log_queries_not_using_indexes = on  
set global long_query_time = 0.1 (秒)  
```
  
#### 慢查询日志分析  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E6%85%A2%E6%9F%A5%E8%AF%A2%E6%97%A5%E5%BF%97.jpeg)  
  
```
Time :日志记录的时间  
User@Host:执行的用户及主机  
Query_time:查询耗费时间  
Lock_time 锁表时间  
Rows_sent 发送给请求方的记录条数  
Rows_examined 语句扫描的记录条数  
SET timestamp 语句执行的时间点  
select .... 执行的具体语句  
```
  
#### 慢查询日志分析工具  
```
mysqldumpslow -t 10 -s at /var/lib/mysql/test-slow.log  
```
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E6%85%A2%E6%9F%A5%E8%AF%A2%E5%B7%A5%E5%85%B7.jpeg)  
  
其他工具  
>mysqlsla  
pt-query-digest  
  
### 事务  
  
#### 什么是事务  
数据库操作的最小工作单元，是作为单个逻辑工作单元执行的一系列操作;  
事务是一组不可再分割的操作集合(工作逻辑单元);  
  
典型事务场景(转账):  
```
update user_account set balance = balance - 1000 where userID = 3;   
update user_account set balance = balance + 1000 where userID = 1;  
```
  
mysql中如何开启事务:  
```
begin / start transaction             -- 手工
commit / rollback                     -- 事务提交或回滚
set session autocommit = on/off;      -- 设定事务是否自动开启

JDBC 编程: 
connection.setAutoCommit(boolean);  
  
Spring 事务AOP编程: 
expression=execution(cn.zyf.dao.*.*(..))  
```
  
#### 事务的ACID特性
>原子性(Atomicity) 最小的工作单元，整个工作单元要么一起提交成功，要么全部失败回滚。  
一致性(Consistency) 事务中操作的数据及状态改变是一致的，即写入资料的结果必须完全符合预设的规则，不会因为出现系统意外等原因导致状态的不一致。  
隔离性(Isolation) 一个事务所操作的数据在提交之前，对其他事务的可见性设定(一般设定为不可见)。  
持久性(Durability) 事务所做的修改就会永久保存，不会因为系统意外导致数据的丢失。  
  
#### 事务并发带来的的问题  
脏读  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E8%84%8F%E8%AF%BB.jpeg)  
  
不可重复读  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E4%B8%8D%E5%8F%AF%E9%87%8D%E5%A4%8D%E8%AF%BB.jpeg)  
  
幻读  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E5%B9%BB%E8%AF%BB.jpeg)  
  
#### 事务的四种隔离级别  
SQL92 ANSI/ISO标准:  
http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt  

>Read Uncommitted(未提交读) --未解决并发问题   
事务未提交对其他事务也是可见的，脏读(dirty read)  
Read Committed(提交读) --解决脏读问题  
一个事务开始之后，只能看到自己提交的事务所做的修改，不可重复读(nonrepeatable read)  
Repeatable Read (可重复读) --解决不可重复读问题   
在同一个事务中多次读取同样的数据结果是一样的，这种隔离级别未定义解决幻读的问题  
Serializable(串行化) --解决所有问题 最高的隔离级别，通过强制事务的串行执行  
  
查看mysql的设置的事务隔离级别
select global.@@tx_isolation; select @@tx_isolation;  
  
#### InnoDB引擎对事务隔离的支持程度  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/InnoDB%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB.jpeg)  
  
### 数据库的锁  
  
#### 表锁和行锁  

锁是用于管理不同事务对共享资源的并发访问  
表锁与行锁的区别:   
>锁定粒度:表锁 > 行锁  
加锁效率:表锁 > 行锁  
冲突概率:表锁 > 行锁  
并发性能:表锁 < 行锁  
  
InnoDB存储引擎支持行锁和表锁(另类的行锁)  
  
#### InnoDB锁类型  
共享锁(行锁):Shared Locks  
排它锁(行锁):Exclusive Locks  
意向锁共享锁(表锁):IntentionShared Locks  
意向锁排它锁(表锁):Intention Exclusive Locks  
自增锁:AUTO-INC Locks  
  
行锁的算法  
记录锁 Record Locks  
间隙锁 Gap Locks  
临键锁 Next-key Locks  
  
共享锁:  
又称为读锁，简称S锁，顾名思义，共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改；  
  
加锁释锁方式:  
select * from users WHERE id=1 LOCK IN SHARE MODE;   
commit/rollback  
  
排他锁:  
又称为写锁，简称X锁，排他锁不能与其他锁并存，如一个事务获取了一个数据行的排他锁，其他事务就不能再获取该行的锁(共享锁、排他锁)，只有该获取了排他锁的事务是可以对数据行进行读取和修改，(其他事务要读取数据可来自于快照)。  
  
加锁释锁方式:  
delete / update / insert 默认加上X锁  
SELECT * FROM table_name WHERE ... FOR UPDATE  
commit/rollback  
  
InnoDB行锁究竟锁住了什么  
  
InnoDB的行锁是通过给索引上的索引项加锁来实现的。  
  
只有通过索引条件进行数据检索，InnoDB才使用行级锁，否则，InnoDB 将使用表锁(锁住索引的所有记录)。  
二级索引(辅助索引)去更新数据，会把二级索引和聚集索引都上锁。  
  
意向共享锁(IS)  
表示事务准备给数据行加入共享锁，即一个数据行加共享锁前必须先取得该表的IS锁，意向共享锁之间是可以相互兼容的。  
  
意向排它锁(IX)  
表示事务准备给数据行加入排他锁，即一个数据行加排他锁前必须先取得该表的IX锁，意向排它锁之间是可以相互兼容的。  
  
意向锁(IS、IX)是InnoDB数据操作之前自动加的，不需要用户干预。  
  
意义:  
当事务想去进行锁表时，可以先判断意向锁是否存在，存在时则可快速返回该表不能启用表锁。  
  
自增锁:AUTO-INC Locks  
针对自增列自增长的一个特殊的表级别锁  
show variables like 'innodb_autoinc_lock_mode';   
默认取值1，代表连续，事务未提交ID永久丢失  
  

临键锁Next-key locks:  
锁住记录+区间(左开右闭)  
当sql执行按照索引进行数据的检索时,查询条件为范围查找(between and、<、>等)并有数 据命中则此时SQL语句加上的锁为Next-key locks，锁住索引的记录+区间(左开右闭)  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E4%B8%B4%E9%94%AE%E9%94%81.jpeg)  
  
间隙锁Gap locks:  
锁住数据不存在的区间(左开右开)  
当sql执行按照索引进行数据的检索时，查询条件的数据不存在，这时SQL语句加上的锁即为 Gap locks，锁住索引不存在的区间(左开右开)  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E9%97%B4%E9%9A%99%E9%94%81.jpeg)  
  
Gap只在RR事务隔离级别存在。  
  
记录锁Record locks:  
锁住具体的索引项  
当sql执行按照唯一性(Primary key、Unique key)索引进行数据的检索时，查询条件等值匹 配且查询的数据是存在，这时SQL语句加上的锁即为记录锁Record locks，锁住具体的索引项  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E8%AE%B0%E5%BD%95%E9%94%81.jpeg)  
  
#### 利用锁解决并发问题  
解决脏读  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E8%A7%A3%E5%86%B3%E8%84%8F%E8%AF%BB.jpeg)  
  
解决不可重复读  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E8%A7%A3%E5%86%B3%E4%B8%8D%E5%8F%AF%E9%87%8D%E5%A4%8D%E8%AF%BB.jpeg)  
  
解决幻读  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/%E8%A7%A3%E5%86%B3%E5%B9%BB%E8%AF%BB.jpeg)  
  
#### 死锁
多个并发事务(2个或者以上);  
每个事务都持有锁(或者是已经在等待锁);  
每个事务都需要再继续持有锁;  
事务之间产生加锁的循环等待，形成死锁。  

#### 死锁的避免
1.类似的业务逻辑以固定的顺序访问表和行。  
2.大事务拆小。大事务更倾向于死锁，如果业务允许，将大事务拆小。  
3.在同一个事务中，尽可能做到一次锁定所需要的所有资源，减少死锁概率。  
4.降低隔离级别，如果业务允许，将隔离级别调低也是较好的选择  
5.为表添加合理的索引。可以看到如果不走索引将会为表的每一行记录添 加上锁(或者说是表锁)  

### MVCC  
```  
ex1:tx1: set session autocommit=off;  
update users set lastUpdate=now() where id =1;  
在未做commit/rollback操作之前  
在其他的事务我们能不能进行对应数据的查询(特别是加上了X锁的数据)  
tx2: select * from users where id > 1;  
select * from users where id = 1;  
```
```
ex2:tx1: begin  
select * from users where id =1 ;
tx2: begin
tx1: update users set lastUpdate=now() where id =1;
select * from users where id =1;  
```
  
这两个案例从结果上来看是一致的，底层实现是一样的吗？跟MVCC有什么关系么？  
  
#### MVCC是什么  
MVCC:Multiversion concurrency control (多版本并发控制)  
  
普通话解释: 并发访问(读或写)数据库时，对正在事务内处理的数据做多版本的管理。以达到用来避免写操作的堵塞，从而引发读操作的并发问题。  
  
MVCC插入逻辑流程  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/MVCC%E6%8F%92%E5%85%A5.jpeg)  
  
MVCC删除逻辑流程  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/MVCC%E5%88%A0%E9%99%A4.jpeg)  
  
MVCC修改逻辑流程  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/MVCC%E4%BF%AE%E6%94%B9.jpeg)  
  
MVCC查询逻辑流程  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/MVCC%E6%9F%A5%E8%AF%A2.jpeg)  
    
#### MVCC版本控制案例 
数据准备:  
```
insert into teacher(name,age) value ('seven',18) ;   
insert into teacher(name,age) value ('qing',20) ;  
```
```
tx1:
begin;                                    ----------1
select * from users ;                     ----------2
commit;
```
```
tx2:
begin;                                    ----------3 
update teacher set age =28 where id =1;   ----------4
commit;
```
  
控制案例1 (1,2,3,4,2)  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/MVCC%E6%A1%88%E4%BE%8B1.jpeg)  
  
控制案例2 (3,4,1,2)  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/MVCC%E6%A1%88%E4%BE%8B2.jpeg)  
  
### Undo Log  
  
#### Undo Log是什么  
undo意为取消，以撤销操作为目的，返回指定某个状态的操作。  
undo log指事务开始之前，在操作任何数据之前，首先将需操作的数据备份到一个地方 (Undo Log)。  
  
UndoLog是为了实现事务的原子性而出现的产物。  
  
Undo Log实现事务原子性:  
事务处理过程中如果出现了错误或者用户执行了ROLLBACK语句,Mysql可以利用Undo Log中的备份将数据恢复到事务开始之前的状态。  
  
UndoLog在MySQL InnoDB存储引擎中用来实现多版本并发控制。  
  
Undo log实现多版本并发控制:  
事务未提交之前，Undo保存了未提交之前的版本数据，Undo 中的数据可作为数据旧版本快照供其他并发事务进行快照读。  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/Undo.jpeg)  
  
#### 当前读、快照读  
快照读:  
SQL读取的数据是快照版本，也就是历史版本，普通的SELECT就是快照读。  
InnoDB快照读，数据的读取将由 cache(原本数据) + undo(事务修改过的数据) 两部分组成。  
  
当前读:  
SQL读取的数据是最新版本。通过锁机制来保证读取的数据无法通过其他事务进行修改。  
UPDATE、DELETE、INSERT、SELECT ... LOCK IN SHARE MODE、SELECT ... FOR UPDATE都是当前读。  
  
### Redo Log  
  
#### Redo Log是什么  
Redo，顾名思义就是重做。以恢复操作为目的，重现操作。  
Redo log指事务中操作的任何数据，将最新的数据备份到一个地方 (Redo Log)。  

不是随着事务的提交才写入的，而是在事务的执行过程中，便开始写入redo中。具体的落盘策略可以进行配置。  
  
RedoLog是为了实现事务的持久性而出现的产物。  
  
Redo Log实现事务持久性: 防止在发生故障的时间点，尚有脏页未写入磁盘，在重启mysql服务的时候，根据redo log进行重做，从而达到事务的未入磁盘数据进行持久化这一特性。  
![](https://github.com/YufeizhangRay/image/blob/master/MySQL/Redo.jpeg)  
  
#### Redo Log补充  
指定Redolog记录在{datadir}/ib_logfile1&ib_logfile2，可通过innodb_log_group_home_dir配置指定目录存储。  
  
一旦事务成功提交且数据持久化落盘之后，此时Redo log中的对应事务数据记录就失去了意义，所以Redo log的写入是日志文件循环写入的。  
>指定Redo log日志文件组中的数量 innodb_log_files_in_group 默认为2。  
指定Redo log每一个日志文件最大存储量innodb_log_file_size 默认48M。  
指定Redo log在cache/buffer中的buffer池大小innodb_log_buffer_size 默认16M。  
  
Redo buffer 持久化Redo log的策略， Innodb_flush_log_at_trx_commit:  
>取值 0 每秒提交 Redo buffer --> Redo log OS cache -->flush cache to disk[可能丢失一秒内的事务数据]。  
取值 1 默认值，每次事务提交执行Redo buffer --> Redo log OS cache -->flush cache to disk[最安全，性能最差的方式]。  
取值 2 每次事务提交执行Redo buffer --> Redo log OS cache 再每一秒执行 ->flush cache to disk操作。  
  
### MySQL配置优化  
  
#### 服务器参数  
基于参数的作用域    
全局参数  
set global autocommit = ON/OFF;  
  
会话参数(会话参数不单独设置则会采用全局参数)  
set session autocommit = ON/OFF;  
  
注意:
全局参数的设定对于已经存在的会话无法生效  
会话参数的设定随着会话的销毁而失效  
全局类的统一配置建议配置在默认配置文件中，否则重启服务会导致配置失效  
  
#### 寻找配置文件  
mysql --help 寻找配置文件的位置和加载顺序  
```  
Default options are read from the following files in the given order:  
/etc/my.cnf 
/etc/mysql/my.cnf 
/usr/etc/my.cnf 
~/.my.cnf  
```
```
mysql --help | grep -A 1 'Default options are read from the following files in the given order'
```
  
#### 全局配置文件配置  
最大连接数配置  
max_connections  
  
系统句柄数配置  
/etc/security/limits.conf   
ulimit -a  
  
mysql句柄数配置  
/usr/lib/systemd/system/mysqld.service  
  
```
port = 3306
socket = /tmp/mysql.sock 
basedir = /usr/local/mysql 
datadir = /data/mysql
pid-file = /data/mysql/mysql.pid 
user = mysql
bind-address = 0.0.0.0

max_connections=2000 
lower_case_table_names = 0 #表名区分大小写 
server-id = 1
tmp_table_size=16M
transaction_isolation = REPEATABLE-READ 
ready_only=1
...
```
   
#### 内存参数配置  
每一个connection内存参数配置:  
sort_buffer_size connection排序缓冲区大小  
建议256K(默认值)-> 2M之内  
当查询语句中有需要文件排序功能时，马上为connection分配配置的内存大小  
  
join_buffer_size connection关联查询缓冲区大小  
建议256K(默认值)-> 1M之内 当查询语句中有关联查询时，马上分配配置大小的内存用这个关联查询，所以有可能在一个查询语句中会分配很多个关联查询缓冲区。  
  
上述配置4000连接占用内存:  
4000*(0.256M+0.256M) = 2G  
  
Innodb_buffer_pool_size  
innodb buffer/cache的大小(默认128M)  
  
Innodb_buffer_pool  
>数据缓存  
索引缓存  
缓冲数据  
内部结构  
  
大的缓冲池可以减小多次磁盘I/O访问相同的表数据以提高性能。  
  
参考计算公式:  
Innodb_buffer_pool_size = (总物理内存 - 系统运行所用 - connection 所用)* 90%   
  
#### 其他参数配置  
wait_timeout  
服务器关闭非交互连接之前等待活动的秒数。  
  
innodb_open_files  
限制Innodb能打开的表的个数。  
  
innodb_write_io_threads  
innodb_read_io_threads  
innodb使用后台线程处理innodb缓冲区数据页上的读写 I/O(输入输出)请求。  

innodb_lock_wait_timeout  
InnoDB事务在被回滚之前可以等待一个锁定的超时秒数。  
  
https://www.cnblogs.com/wyy123/p/6092976.html 常见配置的帖子  
  
### 数据库表设计  
  
#### 三大范式  
第一范式(1NF):  
字段具有原子性，不可再分。所有关系型数据库系统都满足第一范式，数据库表中的字段都是单一属性的，不可再分。  
  
第二范式(2NF):  
要求实体的属性完全依赖于主键。所谓完全依赖是指不能存在仅依赖主键一部分的属性，如果存在，那么这个属性和主关键字的这一部分应该分离出来形成一个新的实体， 新实体与原实体之间是一对多的关系。为实现区分通常需要为表加上一个列，以存储各个实例的惟一标识。简而言之，第二范式就是属性完全依赖主键。  
  
第三范式(3NF):  
满足第三范式(3NF) 必须先满足第二范式(2NF)。 简而言之，第三范式(3NF)要求一个数据库表中不包含已在其它表中已包含的非主键信息。  
  
简单一点:  
>1.每一列只有一个单一的值，不可再拆分。  
2.每一行都有主键能进行区分。  
3.每一个表都不包含其他表已经包含的非主键信息。  
  
#### 完全满足范式的缺点  
充分的满足第一范式设计将为表建立太量的列。  
数据从磁盘到缓冲区，缓冲区脏页到磁盘进行持久的过程中，列的数量过多会导致性能下降。过多的列影响转换和持久的性能。
  
过分的满足第三范式化造成了太多的表关联。  
表的关联操作将带来额外的内存和性能开销。  
  
使用InnoDB引擎的外键关系进行数据的完整性保证。  
外键表中数据的修改会导致Innodb引擎对外键约束进行检查，就带来了额外的开销。  
