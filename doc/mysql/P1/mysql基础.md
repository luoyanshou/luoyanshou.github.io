[TOC]

# Mysql 基础

## Mysql的并发性

共享锁（shared lock，读锁）：共享性，相互不阻塞

排他锁（exclusive lock，写锁）：排他性，一个写锁会阻塞其他的写锁和读锁

## 事务

>  事务的特性「ACID」

1. 原子性（atomicity）一个事务必须被视为一个不可分割的最小工作单元，整个事务中所有操作要么全部提交成功，要么全部失败回滚，对于一个事务来说，不可能只执行其中的一部分操作
2. 一致性（consistency）数据库总是从一个一致性的状态转换到另外一个一致性的状态
3. 隔离性（isolation）一个事务所做的修改在最终提交以前，对其他事务是不可见的
4. 持久性（durability）一旦事务提交，则其所做的修改就会永久保存到数据库中

> 隔离级别

1.  READ UNCOMMITTED（未提交读），事务中的修改，即使没有提交，对其他事务也都是可见的，事务可以读取未提交的数据，也被称为脏读（Dirty Read），这个级别会导致很多问题

2. READ COMMITTED（提交读），大多数数据库系统的默认隔离级别，一个事务开始时，只能“看见”已经提交的事务所做的修改，一个事务从开始直到提交之前，所做的任何修改对其他事务都是不可见的，也叫不可重复读（nonrepeatable read），有可能出现幻读（Phantom Read），指的是当某个事务在读取某个范围内的记录时，另外一个事务又在该范围内插入了新的记录，当之前的事务再次读取该范围的记录时，会产生幻行（Phantom Row）

3.  REPEATABLE READ（可重复读），通过InnoDB和XtraDB存储引擎，是MySQL的默认事务隔离级别

4.  SERIALIZABLE（可串行化）最高级别，通过强制事务串行执行，避免了幻读问题，会在读取的每一行数据上都加锁，可能导致大量的超时和锁争用的问题

   
> 死锁

  指两个或多个事务在同一资源上相互占用，并请求锁定对方占用的资源，从而导致恶性循环的现象

> 事务日志

  存储引擎在修改表的数据时只需要修改其内存拷贝，再把该修改行为记录到持久在硬盘上的事务日志中，而不用每次都将修改的数据本身持久到磁盘。事务日志持久以后，内存中被修改的数据在后台可以慢慢地刷回到磁盘，称为预写式日志（Write-Ahead Logging）

## 多版本并发控制

1. 多版本并发控制（MVCC）是行级锁的一个变种，但是它在很多情况下避免了加锁操作，因此开销更低。虽然实现机制有所不同，但大都实现了非阻塞的读操作，写操作也只锁定必要的行
2. MVCC的实现，是通过保存数据在某个时间点的快照来实现的，有乐观和悲观两种，只在REPEATABLE READ和READ COMMITTED两个隔离级别下工作

## MySQL的存储引擎

> `SHOW ENGINES`可查看当前版本支持的各种存储引擎

### InnoDB

InnoDB是一个健壮的事务型存储引擎，这种存储引擎已经被很多互联网公司使用，为用户操作非常大的数据存储提供了一个强大的解决方案。我的电脑上安装的 MySQL 5.6.13 版，InnoDB就是作为默认的存储引擎。InnoDB还引入了行级锁定和外键约束，在以下场合下，使用InnoDB是最理想的选择：

- 更新密集的表。InnoDB存储引擎特别适合处理多重并发的更新请求。
- 事务。InnoDB存储引擎是支持事务的标准MySQL存储引擎。
- 自动灾难恢复。与其它存储引擎不同，InnoDB表能够自动从灾难中恢复。
- 外键约束。MySQL支持外键的存储引擎只有InnoDB。
- 支持自动增加列AUTO_INCREMENT属性。
- 从5.7开始innodb存储引擎成为默认的存储引擎。

一般来说，如果需要事务支持，并且有较高的并发读取频率，InnoDB是不错的选择。

### MyISAM

MyISAM表是独立于操作系统的，这说明可以轻松地将其从Windows服务器移植到Linux服务器；每当我们建立一个MyISAM引擎的表时，就会在本地磁盘上建立三个文件，文件名就是表名。例如，我建立了一个MyISAM引擎的tb_Demo表，那么就会生成以下三个文件：

- tb_demo.frm，存储表定义。
- tb_demo.MYD，存储数据。
- tb_demo.MYI，存储索引。

MyISAM表无法处理事务，这就意味着有事务处理需求的表，不能使用MyISAM存储引擎。MyISAM存储引擎特别适合在以下几种情况下使用：

1. 选择密集型的表。MyISAM存储引擎在筛选大量数据时非常迅速，这是它最突出的优点。
2. 插入密集型的表。MyISAM的并发插入特性允许同时选择和插入数据。例如：MyISAM存储引擎很适合管理邮件或Web服务器日志数据。

### MRG_MYISAM

MRG_MyISAM存储引擎是一组MyISAM表的组合，老版本叫 MERGE 其实是一回事儿，这些MyISAM表结构必须完全相同，尽管其使用不如其它引擎突出，但是在某些情况下非常有用。说白了，Merge表就是几个相同MyISAM表的聚合器；Merge表中并没有数据，对Merge类型的表可以进行查询、更新、删除操作，这些操作实际上是对内部的MyISAM表进行操作。

Merge存储引擎的使用场景。对于服务器日志这种信息，一般常用的存储策略是将数据分成很多表，每个名称与特定的时间端相关。例如：可以用12个相同的表来存储服务器日志数据，每个表用对应各个月份的名字来命名。当有必要基于所有12个日志表的数据来生成报表，这意味着需要编写并更新多表查询，以反映这些表中的信息。与其编写这些可能出现错误的查询，不如将这些表合并起来使用一条查询，之后再删除Merge表，而不影响原来的数据，删除Merge表只是删除Merge表的定义，对内部的表没有任何影响。

- ENGINE=MERGE，指明使用MERGE引擎，其实是跟MRG_MyISAM一回事儿，也是对的，在MySQL 5.7已经看不到MERGE了。
- UNION=(t1, t2)，指明了MERGE表中挂接了些哪表，可以通过alter table的方式修改UNION的值，以实现增删MERGE表子表的功能。比如：

```
alter table tb_merge engine=merge union(tb_log1) insert_method=last;
```

- INSERT_METHOD=LAST，INSERT_METHOD指明插入方式，取值可以是：0 不允许插入；FIRST 插入到UNION中的第一个表； LAST 插入到UNION中的最后一个表。
- MERGE表及构成MERGE数据表结构的各成员数据表必须具有完全一样的结构。每一个成员数据表的数据列必须按照同样的顺序定义同样的名字和类型，索引也必须按照同样的顺序和同样的方式定义。

### MEMORY

使用MySQL Memory存储引擎的出发点是速度。为得到最快的响应时间，采用的逻辑存储介质是系统内存。虽然在内存中存储表数据确实会提供很高的性能，但当mysqld守护进程崩溃时，所有的Memory数据都会丢失。获得速度的同时也带来了一些缺陷。它要求存储在Memory数据表里的数据使用的是长度不变的格式，这意味着不能使用BLOB和TEXT这样的长度可变的数据类型，VARCHAR是一种长度可变的类型，但因为它在MySQL内部当做长度固定不变的CHAR类型，所以可以使用。

一般在以下几种情况下使用Memory存储引擎：

- 目标数据较小，而且被非常频繁地访问。在内存中存放数据，所以会造成内存的使用，可以通过参数max_heap_table_size控制Memory表的大小，设置此参数，就可以限制Memory表的最大大小。
- 如果数据是临时的，而且要求必须立即可用，那么就可以存放在内存表中。
- 存储在Memory表中的数据如果突然丢失，不会对应用服务产生实质的负面影响。
- Memory同时支持散列索引和B树索引。B树索引的优于散列索引的是，可以使用部分查询和通配查询，也可以使用<、>和>=等操作符方便数据挖掘。散列索引进行“相等比较”非常快，但是对“范围比较”的速度就慢多了，因此散列索引值适合使用在=和<>的操作符中，不适合在<或>操作符中，也同样不适合用在order by子句中。

### CSV

CSV 存储引擎是基于 CSV 格式文件存储数据。

- CSV 存储引擎因为自身文件格式的原因，所有列必须强制指定 NOT NULL 。
- CSV 引擎也不支持索引，不支持分区。
- CSV 存储引擎也会包含一个存储表结构的 .frm 文件，还会创建一个 .csv 存储数据的文件，还会创建一个同名的元信息文件，该文件的扩展名为 .CSM ，用来保存表的状态及表中保存的数据量。
- 每个数据行占用一个文本行。

因为 csv 文件本身就可以被Office等软件直接编辑，保不齐就有不按规则出牌的情况，如果出现csv 文件中的内容损坏了的情况，也可以使用 CHECK TABLE 或者 REPAIR TABLE 命令检查和修复

### ARCHIVE

Archive是归档的意思，在归档之后很多的高级功能就不再支持了，仅仅支持最基本的插入和查询两种功能。在MySQL 5.5版以前，Archive是不支持索引，但是在MySQL 5.5以后的版本中就开始支持索引了。Archive拥有很好的压缩机制，它使用zlib压缩库，在记录被请求时会实时压缩，所以它经常被用来当做仓库使用。

### BLACKHOLE

黑洞存储引擎，所有插入的数据并不会保存，BLACKHOLE 引擎表永远保持为空，写入的任何数据都会消失，

### PERFORMANCE_SCHEMA

主要用于收集数据库服务器性能参数。MySQL用户是不能创建存储引擎为PERFORMANCE_SCHEMA的表，一般用于记录binlog做复制的中继。在这里有官方的一些介绍[MySQL Performance Schema](https://dev.mysql.com/doc/refman/5.6/en/performance-schema.html)

### FEDERATED

主要用于访问其它远程MySQL服务器一个代理，它通过创建一个到远程MySQL服务器的客户端连接，并将查询传输到远程服务器执行，而后完成数据存取；在MariaDB的上实现是FederatedX

### Mysql常用引擎对比

| 特性                                                   | InnoDB | MyISAM | MEMORY | ARCHIVE |
| ------------------------------------------------------ | ------ | ------ | ------ | ------- |
| 存储限制(Storage limits)                               | 64TB   | No     | YES    | No      |
| 支持事物(Transactions)                                 | Yes    | No     | No     | No      |
| 锁机制(Locking granularity)                            | 行锁   | 表锁   | 表锁   | 行锁    |
| B树索引(B-tree indexes)                                | Yes    | Yes    | Yes    | No      |
| T树索引(T-tree indexes)                                | No     | No     | No     | No      |
| 哈希索引(Hash indexes)                                 | Yes    | No     | Yes    | No      |
| 全文索引(Full-text indexes)                            | Yes    | Yes    | No     | No      |
| 集群索引(Clustered indexes)                            | Yes    | No     | No     | No      |
| 数据缓存(Data caches)                                  | Yes    | No     | N/A    | No      |
| 索引缓存(Index caches)                                 | Yes    | Yes    | N/A    | No      |
| 数据可压缩(Compressed data)                            | Yes    | Yes    | No     | Yes     |
| 加密传输(Encrypted data)                            | Yes    | Yes    | Yes    | Yes     |
| 集群数据库支持(Cluster databases support)              | No     | No     | No     | No      |
| 复制支持(Replication support)                       | Yes    | No     | No     | Yes     |
| 外键支持(Foreign key support)                          | Yes    | No     | No     | No      |
| 存储空间消耗(Storage Cost)                             | 高     | 低     | N/A    | 非常低  |
| 内存消耗(Memory Cost)                                  | 高     | 低     | N/A    | 低      |
| 数据字典更新(Update statistics for data dictionary)    | Yes    | Yes    | Yes    | Yes     |
| 备份/时间点恢复(backup/point-in-time recovery)      | Yes    | Yes    | Yes    | Yes     |
| 多版本并发控制(Multi-Version Concurrency Control/MVCC) | Yes    | No     | No     | No      |
| 批量数据写入效率(Bulk insert speed)                    | 慢     | 快     | 快     | 非常快  |
| 地理信息数据类型(Geospatial datatype support)          | Yes    | Yes    | No     | Yes     |
| 地理信息索引(Geospatial indexing support)           | Yes    | Yes    | No     | Yes     |


