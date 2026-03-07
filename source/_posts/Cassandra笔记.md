---
title: Cassandra笔记
date: 2026-01-01 08:05:00
tags:
  - "数据库"
  - "Cassandra"
  - "NoSQL"
  - "笔记"
categories:
  - "数据科学"
---

﻿[https://github.com/QInzhengk/Math-Model-and-Machine-Learning](https://github.com/QInzhengk/Math-Model-and-Machine-Learning)
@[TOC](Cassandra笔记)
## Cassandra常用操作
### 1、查看键空间

```sql
DESCRIBE keyspaces;
```
该命令用于展示casandra下的所有的keyspaces（类比mysql的show databases;），casandra的keyspaces和mysql的数据库概念相似，属于从逻辑上区分的物理隔离空间，基于各个keyspaces，管理各自的tables。

如果想进入到某个keyspace下，可以使用  `use [keyspace名称];`
### 2、查询所有table

```sql
desc tables;
```
### 3、清空表数据

```sql
truncate <表名称>;
```
### 4、cassandra支持插入数据时候设定一个过期时间，只需要在插入语句后面加上TTL关键字进行标识即可

```sql
update table using ttl 30 set uv = 6 where id = 'id'
```

## Cassandra数据模型
### 集群（Cluster）
Cassandra 数据库分布在几个一起操作的机器上。最外层容器被称为群集。对于故障处理，每个节点包含一个副本，如果发生故障，副本将复制。Cassandra 按照环形格式将节点排列在集群中，并为它们分配数据。

### 键空间 （Keyspace）
键空间是 Cassandra 中数据的最外层容器。Cassandra 中的一个键空间的基本属性是 - 

 - **复制因子** - 它是集群中将接收相同数据副本的计算机数。
 - **副本放置策略** - 它只是把副本放在介质中的策略。我们有简单策略（机架感知策略），旧网络拓扑策略（机架感知策略）和网络拓扑策略（数据中心共享策略）等策略。
 - **列族** - 键空间是一个或多个列族的列表的容器。列族又是一个行集合的容器。每行包含有序列。列族表示数据的结构。每个键空间至少有一个，通常是许多列族。

创建键空间的语法如下 -
```sql
CREATE KEYSPACE Keyspace name
WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 3};
```
## Cassandra 参考API
### 集群（Cluster）
这个类是驱动程序的主要入口点。它属于com.datastax.driver.core包。

**方法**

![在这里插入图片描述](/img/0605b8682652a0e389b18c585a3bdd6d.png)
### Cluster.Builder
此类用于实例化Cluster.Builder类。

**方法**

![在这里插入图片描述](/img/d051bd861caf47cd2dff6754619b0939.png)
### 会话
此接口保存与Cassandra群集的连接。使用此接口，可以执行CQL查询。它属于com.datastax.driver.core包。

**方法**

![在这里插入图片描述](/img/5e0821d350f00cf3e66cab9c7d68bb5c.png)
## Cassandra Cqlsh
默认情况下，Cassandra提供一个提示Cassandra查询语言shell（cqlsh），允许用户与它通信。使用此shell，您可以执行Cassandra查询语言（CQL）。

使用cqlsh，你可以

 - 定义模式， 
 - 插入数据， 
 - 执行查询。

### 启动cqlsh
使用命令cqlsh启动cqlsh，如下所示。它提供Cassandra cqlsh提示作为输出。

```sql
[hadoop@linux bin]$ cqlsh
Connected to Test Cluster at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 2.1.2 | CQL spec 3.2.0 | Native protocol v3]
Use HELP for help.
cqlsh>
```
Cqlsh - 如上所述，此命令用于启动cqlsh提示符。此外，它还支持更多的选项。下表说明了cqlsh的所有选项及其用法。

|选项	|用法|
| -- | -  |
|cqlsh --help	|显示有关cqlsh命令的选项的帮助主题。|
|cqlsh --version	|提供您正在使用的cqlsh的版本。|
|cqlsh --color	|指示shell使用彩色输出。|
|cqlsh --debug	|显示更多的调试信息。|
|cqlsh --execute      cql_statement|指示shell接受并执行CQL命令。|
|cqlsh --file= “**file name**”	|如果使用此选项，Cassandra将在给定文件中执行命令并退出。|
|cqlsh --no-color	|指示Cassandra不使用彩色输出。|
|cqlsh -u “**user name**”	|使用此选项，您可以验证用户。默认用户名为：cassandra。|
|**cqlsh-p “pass word”**	|使用此选项，您可以使用密码验证用户。默认密码为：cassandra。|

### Cqlsh命令
Cqlsh有几个命令，允许用户与它进行交互。命令如下所示。

### 记录的Shell命令
下面给出了Cqlsh记录的shell命令。这些是用于执行任务的命令，如显示帮助主题，退出cqlsh，描述等。

 - **HELP** -显示所有cqlsh命令的帮助主题。
 - **CAPTURE** -捕获命令的输出并将其添加到文件。
 - **CONSISTENCY** -显示当前一致性级别，或设置新的一致性级别。
 - **COPY** -将数据复制到Cassandra并从Cassandra复制数据。
 - **DESCRIBE** -描述Cassandra及其对象的当前集群。
 - **EXPAND** -纵向扩展查询的输出。
 - **EXIT** -使用此命令，可以终止cqlsh。
 - **PAGING** -启用或禁用查询分页。
 - **SHOW** -显示当前cqlsh会话的详细信息，如Cassandra版本，主机或数据类型假设。
 - **SOURCE** -执行包含CQL语句的文件。
 - **TRACING** -启用或禁用请求跟踪。

### CQL数据定义命令

 - **CREATE KEYSPACE** -在Cassandra中创建KeySpace。
 - **USE** -连接到已创建的KeySpace。
 - **ALTER KEYSPACE** -更改KeySpace的属性。
 - **DROP KEYSPACE** -删除KeySpace。
 - **CREATE TABLE** -在KeySpace中创建表。
 - **ALTER TABLE** -修改表的列属性。
 - **DROP TABLE** -删除表。
 - **TRUNCATE** -从表中删除所有数据。
 - **CREATE INDEX** -在表的单个列上定义新索引。
 - **DROP INDEX** -删除命名索引。

### CQL数据操作指令

 - **INSERT** -在表中添加行的列。
 - **UPDATE** -更新行的列。
 - **DELETE** -从表中删除数据。
 - **BATCH** -一次执行多个DML语句。

### CQL字句

 - **SELECT** -此子句从表中读取数据
 - **WHERE** -where子句与select一起使用以读取特定数据。
 - **ORDERBY** -orderby子句与select一起使用，以特定顺序读取特定数据。

## Cassandra Shell命令
除了CQL命令，Cassandra还提供了记录的shell命令。下面给出了Cassandra记录的shell命令。

### Help
HELP命令显示所有cqlsh命令的摘要和简要描述。下面给出了help命令的用法。

```sql
cqlsh> help

Documented shell commands:
===========================
CAPTURE COPY DESCRIBE EXPAND PAGING SOURCE
CONSISTENCY DESC EXIT HELP SHOW TRACING.

CQL help topics:
================
ALTER           CREATE_TABLE_OPTIONS       SELECT
ALTER_ADD       CREATE_TABLE_TYPES         SELECT_COLUMNFAMILY
ALTER_ALTER     CREATE_USER                SELECT_EXPR
ALTER_DROP      DELETE                     SELECT_LIMIT
ALTER_RENAME    DELETE_COLUMNS             SELECT_TABLE 
```
### Capture
此命令捕获命令的输出并将其添加到文件。例如，看看下面的代码，它将输出捕获到名为Outputfile的文件。

```sql
cqlsh> CAPTURE '/home/hadoop/CassandraProgs/Outputfile'
```

当我们在终端中键入任何命令时，输出将被给定的文件捕获。下面给出的是使用的命令和输出文件的快照。

```sql
cqlsh:tutorialspoint> select * from emp;
```

![文件](/img/b9a2896bbfd7eea6ac9200303f1bf891.png)

您可以使用以下命令关闭捕获。

```sql
cqlsh:tutorialspoint> capture off;
```

### Consistency
此命令显示当前的一致性级别，或设置新的一致性级别。

```sql
cqlsh:tutorialspoint> CONSISTENCY
Current consistency level is 1.
```

### Copy
此命令将数据从Cassandra复制到文件并从中复制。下面给出一个将名为emp的表复制到文件myfile的示例。

```sql
cqlsh:tutorialspoint> COPY emp (emp_id, emp_city, emp_name, emp_phone,emp_sal) TO ‘myfile’;
4 rows exported in 0.034 seconds.
```

如果您打开并验证给定的文件，您可以找到复制的数据，如下所示。

![文件2](/img/362aa25fc554cb545d8d597c6b492326.png)

### Describe
此命令描述Cassandra及其对象的当前集群。此命令的变体说明如下。

**Describe cluster** -此命令提供有关集群的信息。

```sql
cqlsh:tutorialspoint> describe cluster;

Cluster: Test Cluster
Partitioner: Murmur3Partitioner

Range ownership:
                  -658380912249644557 [127.0.0.1]
                  -2833890865268921414 [127.0.0.1]
                  -6792159006375935836 [127.0.0.1] 
```

**Describe Keyspaces** -此命令列出集群中的所有键空间。下面给出了这个命令的用法。

```sql
cqlsh:tutorialspoint> describe keyspaces;

system_traces system tp tutorialspoint
```

**Describe tables** -此命令列出了键空间中的所有表。下面给出了这个命令的用法。

```sql
cqlsh:tutorialspoint> describe tables;
emp
```

**Describe tables** -此命令提供表的描述。下面给出了这个命令的用法。

```sql
cqlsh:tutorialspoint> describe table emp;

CREATE TABLE tutorialspoint.emp (
   emp_id int PRIMARY KEY,
   emp_city text,
   emp_name text,
   emp_phone varint,
   emp_sal varint
) WITH bloom_filter_fp_chance = 0.01
   AND caching = '{"keys":"ALL", "rows_per_partition":"NONE"}'
   AND comment = ''
   AND compaction = {'min_threshold': '4', 'class':
   'org.apache.cassandra.db.compaction.SizeTieredCompactionStrategy',
   'max_threshold': '32'}
	
   AND compression = {'sstable_compression':
   'org.apache.cassandra.io.compress.LZ4Compressor'}
	
   AND dclocal_read_repair_chance = 0.1
   AND default_time_to_live = 0
   AND gc_grace_seconds = 864000
   AND max_index_interval = 2048
   AND memtable_flush_period_in_ms = 0
   AND min_index_interval = 128
   AND read_repair_chance = 0.0
   AND speculative_retry = '99.0PERCENTILE';
CREATE INDEX emp_emp_sal_idx ON tutorialspoint.emp (emp_sal);
```

### Describe tables
此命令用于描述用户定义的数据类型。下面给出了这个命令的用法。

```sql
cqlsh:tutorialspoint> describe type card_details;

CREATE TYPE tutorialspoint.card_details (
   num int,
   pin int,
   name text,
   cvv int,
   phone set<int>,
   mail text
);
```

### Describe Types
此命令列出所有用户定义的数据类型。下面给出了这个命令的用法。假设有两种用户定义的数据类型：card和card_details。

```sql
cqlsh:tutorialspoint> DESCRIBE TYPES;

card_details card
```

### Expand
此命令用于扩展输出。在使用此命令之前，您必须打开expand命令。下面给出了这个命令的用法。

```sql
cqlsh:tutorialspoint> expand on;
cqlsh:tutorialspoint> select * from emp;

@ Row 1
-----------+------------
    emp_id | 1
  emp_city | Hyderabad
  emp_name | ram
 emp_phone | 9848022338
   emp_sal | 50000
  
@ Row 2
-----------+------------
    emp_id | 2
  emp_city | Delhi
  emp_name | robin
 emp_phone | 9848022339
   emp_sal | 50000
  
@ Row 3
-----------+------------
    emp_id | 4
  emp_city | Pune
  emp_name | rajeev
 emp_phone | 9848022331
   emp_sal | 30000
  
@ Row 4
-----------+------------
    emp_id | 3
  emp_city | Chennai
  emp_name | rahman
 emp_phone | 9848022330
   emp_sal | 50000
(4 rows)
```

**注意**：您可以使用以下命令关闭展开选项。

```sql
cqlsh:tutorialspoint> expand off;
Disabled Expanded output.
```

### Exit
此命令用于终止cql shell。

### Show
此命令显示当前cqlsh会话的详细信息，如Cassandra版本，主机或数据类型假设。下面给出了这个命令的用法。

```sql
cqlsh:tutorialspoint> show host;
Connected to Test Cluster at 127.0.0.1:9042.

cqlsh:tutorialspoint> show version;
[cqlsh 5.0.1 | Cassandra 2.1.2 | CQL spec 3.2.0 | Native protocol v3]
```

### Source
使用此命令，可以在文件中执行命令。假设我们的输入文件如下：

源1
然后可以执行包含命令的文件，如下所示。

```sql
cqlsh:tutorialspoint> source '/home/hadoop/CassandraProgs/inputfile';

 emp_id |  emp_city | emp_name |  emp_phone | emp_sal
--------+-----------+----------+------------+---------
      1 | Hyderabad |   ram    | 9848022338 | 50000
      2 | Delhi     |   robin  | 9848022339 | 50000
      3 | Pune      |   rajeev | 9848022331 | 30000
      4 | Chennai   |   rahman | 9848022330 | 50000
(4 rows)
```
参考链接：[https://ke.qq.com/itdoc/cassandra/cassandra_create_keyspace.html](https://ke.qq.com/itdoc/cassandra/cassandra_create_keyspace.html)
